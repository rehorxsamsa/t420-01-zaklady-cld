# Architektura — Task Library

Referenční mini-aplikace (TODO list) v čistém **PHP 8.3 OOP** — bez frameworku, bez Composeru, bez šablonovacího enginu. Záměrně minimální plocha kódu s poctivým vrstvením; slouží jako pískoviště pro tutoriál Claude Code. Cílem tohoto dokumentu není opakovat, co je z kódu zřejmé, ale zachytit **invarianty, kompromisy a neintuitivní chování**, která z jednoho souboru nevyčteš.

## Stack

- **PHP 8.3**, `declare(strict_types=1)` všude, třídy `final`, `readonly` kde dává smysl.
- **SQLite / PDO** — souborová DB (`data/tasks.sqlite`), vzniká a migruje se za běhu.
- **Bootstrap 5 přes CDN** — žádný frontend build.
- **Docker** — nginx (:8080) jako edge, PHP-FPM (:9000, jen `expose`) jako backend přes FastCGI.

## Request flow

Klasické vrstvené MVC-ish uspořádání, ruční drátování. Řetěz na jeden request:

```
prohlížeč ──:8080──▶ nginx ──FastCGI :9000──▶ public/index.php ──▶ Router ──▶ TaskController ──▶ TaskService ──▶ TaskRepository ──▶ Database (PDO/SQLite)
                     (docker/nginx/            (front controller   (Core/       (Controller/       (Service/        (Repository/       (Core/Database.php
                      default.conf)             + routovací tabulka) Router.php)  TaskController.php) TaskService.php) TaskRepository.php) singleton+migrace+seed)
```

Napříč vrstvami putují dvě doménové entity: **`Task`** (`src/Model/Task.php`) a **`AuditEntry`** (`src/Model/AuditEntry.php`), obě s factory `fromRow()` mapující DB řádek na typovaný objekt.

## Vrstvy a invarianty

| Vrstva | Soubor | Odpovědnost | Tvrdá hranice |
|---|---|---|---|
| Front controller | `public/index.php` | registrace rout, dispatch | žádná doménová logika |
| Router | `src/Core/Router.php` | (metoda+cesta) → callable | nezná doménu |
| Controller | `src/Controller/TaskController.php` | čtení `$_POST`, volání Service, render/redirect | **nikdy** Repository ani PDO přímo |
| Service | `src/Service/TaskService.php` | business logika, validace, orchestrace auditu | žádné SQL |
| Repository | `src/Repository/*Repository.php` | veškeré SQL přes PDO | žádná business pravidla |
| Model | `src/Model/{Task,AuditEntry}.php` | držet data | nesahá na DB |
| Database | `src/Core/Database.php` | connection, migrace, seed | nezná doménu nad rámec schématu |

**Klíčový invariant:** Controller → Service → Repository, jednosměrně. SQL žije výhradně v Repository, business logika (validace, `progress()`, volba auditní akce) výhradně v Service. Porušení téhle hranice je to, co tutoriál v pozdějším dílu chytá přes `/review`.

## Netriviální rozhodnutí a gotchas

### Router: substituce, ne matcher
`dispatch()` lineárně projede routy, `{id}` nahradí za `(?P<id>\d+)` a matchne celou cestu. Důsledky, které stojí za vědomí:

- **Jediný podporovaný parametr je `{id}`** (číselný). Žádné wildcardy, žádné volitelné segmenty, žádné query-param routování.
- **Neexistuje 405** — neshoda metody spadne do stejné větve jako neznámá cesta a vrátí holé `404`. Route table se registruje v `index.php`, ne u controlleru.
- Handler dostává `?int $id` (null pro bezparametrické routy); přetypování na `int` řeší lambda v `index.php`.

### Database: procesní singleton s idempotentní migrací
`Database::connection()` je `static ?PDO` cachovaný **v rámci PHP-FPM workeru** (ne globálně napříč procesy). Nastavuje `ERRMODE_EXCEPTION` a `FETCH_ASSOC`. Při prvním volání v procesu spustí `migrate()`:

1. `CREATE TABLE IF NOT EXISTS` pro `tasks` i `audit_log` (idempotentní, běží na každém cold connectu — levné).
2. Seed 3 úkolů **jen když je `tasks` prázdná**.

Migrace je „schema-on-boot" bez verzování — jakákoli změna sloupce vyžaduje ruční zásah do existující DB, `IF NOT EXISTS` migraci sloupců neřeší.

Schéma `tasks`:

| sloupec | typ | pozn. |
|---|---|---|
| `id` | INTEGER PK AUTOINCREMENT | |
| `title` | TEXT NOT NULL | |
| `done` | INTEGER NOT NULL DEFAULT 0 | 0/1 → v PHP `bool` |
| `created_at` | TEXT NOT NULL | ISO 8601 (`date('c')`) |

Schéma `audit_log` — **append-only** stopa, zapisuje `TaskService` přes `AuditLogRepository`, záznamy se nikdy neupravují ani nemažou:

| sloupec | typ | pozn. |
|---|---|---|
| `id` | INTEGER PK AUTOINCREMENT | |
| `action` | TEXT NOT NULL | strojový klíč (`task.created`/`task.completed`/`task.reopened`/`task.deleted`) |
| `task_id` | INTEGER (nullable) | **bez FK** — záznam vědomě přežije smazání úkolu |
| `detail` | TEXT NOT NULL | lidsky čitelný český popis |
| `created_at` | TEXT NOT NULL | ISO 8601 |

Log se čte na `GET /audit` (`templates/audit.php`).

### Audit a atomicita — čtený stav
`TaskService::toggle()`/`remove()` nejdřív `find()` úkol (kvůli titulku a stavu pro audit), pak provedou write, pak zapíšou audit. Dvě vědomá omezení pro sandbox:

- **Write úkolu a zápis auditu nejsou v jedné transakci** — teoreticky můžou rozejít.
- **Auditní label u `toggle` se odvozuje z pre-write stavu** (`$task->done` přečteného před UPDATE). Samotný UPDATE je atomický (`done = 1 - done`), ale mezi read a write není zámek — pod konkurencí se label může rozejít se skutečností. Pro jednouživatelské pískoviště irelevantní, u produkce by šlo o TOCTOU.

Žádná CSRF ochrana na POST formulářích — mimo scope výukové aplikace.

### Model: „neměnný-ish"
`Task` má `readonly` jen `id` a `createdAt`; `title` a `done` jsou mutable. Není to plnohodnotná value object neměnnost — je to úmyslný kompromis (řádek jde načíst, upravit v paměti, ale identita a čas vzniku jsou fixní).

### DI „chudého muže"
Žádný kontejner. Závislosti jsou **defaulty v konstruktoru** (`TaskController(TaskService $s = new TaskService())`, dtto Service → dvě repository). V provozu se řetěz složí sám z `new TaskController()`; pro testy se závislost injektne ručně (`new TaskService($fakeRepo, $fakeAudit)`), takže vrstvy zůstávají testovatelné i bez kontejneru. Default `new` v signatuře se vyhodnocuje per-instance, ne staticky.

### Repository detaily
- `create()` po INSERT dělá `lastInsertId()` + re-`find()` (extra round-trip, ale vrací plně hydratovaný `Task` s `created_at`); fallback `?? throw`.
- `all()` řadí `ORDER BY id DESC` (nejnovější první) a mapuje přes `Task::fromRow()`.
- Všechny mutace přes prepared statements s pojmenovanými parametry.

### Šablony = prosté PHP
`TaskController::render()` udělá `extract($data)` a `require templates/<jméno>.php` — klíče pole se stanou lokálními proměnnými šablony. Žádný escaping engine (na výstup pozor ručně). Šablony: `templates/tasks.php`, `templates/audit.php`.

### Ruční PSR-4 autoloader
`autoload.php` mapuje prefix `App\` → `src/`. `public/index.php` načítá **jen** `autoload.php`, zbytek se dotahuje on-demand. Nová třída musí ležet v `src/` podle namespace, jinak se nenačte.

## Docker

Dvě služby (`docker-compose.yml`), obě s bind-mountem `./:/var/www/html` (změny zdrojáků bez rebuildu):

- **`app`** — `php:8.3-fpm-alpine` + `pdo_sqlite` (viz `docker/Dockerfile`), PHP-FPM :9000, ven se nepublikuje.
- **`web`** — `nginx:1.27-alpine`, `8080:80`, statiku servíruje sám, PHP předává FastCGI na `app:9000` (`docker/nginx/default.conf`).

**Gotcha s bind mountem:** mount překryje vše, co Dockerfile vytvořil uvnitř `/var/www/html`, včetně `data/` a jeho práv. Proto `data/` musí existovat a být zapisovatelný pro `www-data` (uid 82) **na hostitelské straně** mountu. DB soubor je gitignorovaný, generuje se za běhu.

## Příkazy

```bash
docker compose up -d --build      # poprvé / po změně Dockerfile
docker compose up -d              # dále
docker compose down

# PHP je jen v kontejneru — lint běží uvnitř:
docker compose exec -T app find . -name "*.php" -not -path "./data/*" -exec php -l {} \;
docker compose exec -T app php -l src/Service/TaskService.php
```

Aplikace: http://localhost:8080

## Stav testů

Testovací framework zatím **není**. `TaskService::progress()` (procento hotových úkolů, ošetřuje dělení nulou) je určený kandidát na první unit testy a `/review` v dílu 02 tutoriálu.
