# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Malá PHP-OOP aplikace bez frameworku — slouží jako pískoviště pro tutoriál Claude Code.

## Stack
- PHP 8.3 (čisté OOP, žádný framework, žádný Composer)
- SQLite (PDO)
- Bootstrap 5 (přes CDN)
- Docker: nginx + PHP-FPM

## Architektura
- Ruční PSR-4 autoloader (`autoload.php`), namespace `App\` → `src/`. Nová třída musí ležet v `src/` podle namespace, jinak ji nic nenačte.
- Front controller: `public/index.php`
- Vrstvení: **Controller → Service → Repository → (PDO)**
- Šablony: `templates/*.php` (prosté PHP, žádný šablonovací engine). `TaskController::render()` udělá `extract($data)` a `require` šablonu.

Tok requestu (chování je rozprostřené přes víc souborů):

```
nginx (web) → public/index.php → Router → TaskController → TaskService → TaskRepository → Database (PDO/SQLite)
```

Co není vidět z jednoho souboru:
- **Routy se registrují jako tabulka v `public/index.php`** (`$router->add(...)`), ne anotacemi. `Router` umí jen literální cesty a jeden parametr `{id}` (regex `(?P<id>\d+)`). Novou cestu přidáváš do `index.php`.
- **`Database::connection()` je singleton, který si při prvním volání sám vytvoří schéma a naseeduje 3 úkoly** (`migrate()`). DB soubor `data/tasks.sqlite` vzniká za běhu, je gitignorovaný a v Dockeru musí být zapisovatelný pro `www-data` (řeší Dockerfile).
- **DI „chudého muže"** — žádný kontejner. Každá vyšší vrstva má závislost jako default v konstruktoru (`new TaskService()`, `new TaskRepository()`); pro testy lze závislost předat ručně.

## Příkazy
- Start: `docker compose up --build`
- Aplikace běží na: http://localhost:8080
- Lint všech PHP: `find . -name "*.php" -not -path "./data/*" -exec php -l {} \;`
- Lint jednoho souboru: `php -l src/Service/TaskService.php`

## Testy
- Testovací framework zatím **není**. `TaskService::progress()` je v README dílu 01 označená jako kandidát na testy a `/review` v dílu 02.

## Konvence
- `declare(strict_types=1)` v každém PHP souboru
- Třídy jsou `final`, vlastnosti `readonly` kde to dává smysl
- Repository je jediné místo, kde se sahá na DB — Controller na ni nikdy nesahá přímo
- Komentáře a UI texty jsou česky

## Don't
- Nepřidávej Composer ani framework — celý smysl je čisté OOP
- Neměň `data/tasks.sqlite` ručně (vytváří se automaticky při startu)
- Nedávej business logiku do Controlleru — patří do Service
