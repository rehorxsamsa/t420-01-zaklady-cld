# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Co to je

Díl 01 výukové série o Claude Code („Základy"). Dvě části:

- **`README.md`** — vlastní text dílu (instalace, první session, `/init`, `@` mentions, bash mód `!`, Plan vs. yolo mode, `/clear`). Když upravuješ obsah tutoriálu, edituješ tenhle soubor; je psaný česky a obsahuje na konci sekci „Test dílu 01" s odpověďmi v `<details>`.
- **`task-library/`** — spustitelná pískoviště aplikace, na které tutoriál demonstruje příkazy. Má vlastní `CLAUDE.md` s detaily — přečti ho před prací uvnitř.

README na aplikaci přímo odkazuje (cesty k souborům, příklady promptů). Když přejmenuješ nebo přesuneš soubor v `task-library/`, zkontroluj, jestli na něj README nereferuje.

## Aplikace task-library — architektura

Čisté PHP 8.3 OOP, **žádný framework, žádný Composer**, SQLite přes PDO, nginx + PHP-FPM v Dockeru.

Tok requestu (vyžaduje pochopení přes víc souborů):

```
nginx (web)  →  public/index.php  →  Router  →  TaskController  →  TaskService  →  TaskRepository  →  Database (PDO)
                (front controller   (regex      (orchestrace,      (business      (jediný přístup    (singleton,
                 + tabulka rout)     {id})       render šablony)     logika)        k DB)              auto-migrace)
```

Klíčové, co není vidět z jednoho souboru:

- **Routing je tabulka v `public/index.php`** — routy se registrují tam (`$router->add(...)`), ne anotacemi. Router umí jen literální cesty a jeden parametr `{id}` (regex `(?P<id>\d+)`). Novou cestu přidáváš do `index.php`.
- **DI „chudého muže"** — vrstvy se neskládají žádným kontejnerem. Každá vyšší vrstva má závislost jako default v konstruktoru (`new TaskService()`, `new TaskRepository()`). Pro testy lze závislost předat ručně.
- **`Database::connection()` je singleton, který při prvním volání sám vytvoří schéma a naseeduje 3 úkoly** (`migrate()`). DB soubor `data/tasks.sqlite` se generuje za běhu, je gitignorovaný a v Dockeru musí být zapisovatelný pro `www-data` (řeší Dockerfile).
- **Šablony jsou prosté PHP** — `TaskController::render()` udělá `extract($data)` a `require templates/<x>.php`. Žádný šablonovací engine.
- **Autoloader je ruční PSR-4** (`autoload.php`): `App\` → `src/`. Nová třída musí ležet v `src/` podle namespace, jinak ji nic nenačte.

## Příkazy

Vše se pouští z `task-library/`:

```bash
docker compose up --build        # nastartuje app (PHP-FPM) + web (nginx) → http://localhost:8080
docker compose down

# Lint všech PHP souborů (přeskoč generovanou DB)
find . -name "*.php" -not -path "./data/*" -exec php -l {} \;
php -l src/Service/TaskService.php   # lint jednoho souboru
```

Testovací framework zatím není (`progress()` v `TaskService` je v README označená jako kandidát na testy v dílu 02).

## Konvence (platí v task-library)

- `declare(strict_types=1)` v každém PHP souboru; třídy `final`, `readonly` kde dává smysl.
- **Vrstvení se nesmí porušit:** Controller nikdy nesahá na DB ani na Repository přímo — jde přes Service. Repository je jediné místo s SQL.
- **Business logika patří do Service, ne do Controlleru.**
- Komentáře a UI texty jsou česky.
- Nepřidávej Composer ani framework — celý smysl pískoviště je čisté OOP.
