# Díl 01 — Základy: instalace, první session a klávesové zkratky

> **Co se naučíš:** `npm install -g @anthropic-ai/claude-code`, `claude update`, `/init`, `/clear`,
> `@` mentions, bash mód `!`, line break `\`, `Esc`, `Ctrl+R`, Plan mode, auto-accept („yolo mode").
>
> **Na čem:** na naší spustitelné aplikaci `task-library/` (PHP-OOP + Bootstrap + Docker).

---

## 1. Instalace Claude Code

`npm install -g @anthropic-ai/claude-code`. To pořád platí. V roce 2026 máš dvě cesty:

```bash
# Varianta A — přes npm (jakýkoliv OS, potřebuješ Node.js 18+)
npm install -g @anthropic-ai/claude-code

# Varianta B — instalační skript (macOS / Linux / WSL)
curl -fsSL https://claude.ai/install.sh | sh

# Ověření
claude --version
```

> 💡 **Tvůj setup (Pokud používáš WSL2/Ubuntu):** Doporučuju variantu A nebo B přímo ve WSL, ne ve Windows. Claude Code pak vidí tvůj linuxový filesystém `~/projekty/` nativně.

### `claude update`

```bash
claude update      # aktualizace na nejnovější verzi
```

Claude Code vychází **prakticky každý týden**, takže `claude update` pouštěj klidně jednou týdně. Pokud něco z cheatsheetu nesedí, většinou je důvod ten, že máš starou verzi — `claude update` to často vyřeší.

---

## 2. Spuštění naší pískoviště aplikace

Než pustíme Claude Code, rozběhneme codebase, na které budeme pracovat:

```bash
cd t420-01-zaklady-cld/task-library
docker compose up --build
```

Otevři http://localhost:8080 — uvidíš „Knihovnu úkolů" s progress barem a seznamem úkolů. Funguje přidávání, odškrtávání i mazání.

Architektura (ať víš, co Claude uvidí):
```
task-library/
├── autoload.php              # ruční PSR-4 autoloader (žádný Composer)
├── public/index.php          # front controller
├── src/
│   ├── Core/                 # Router, Database
│   ├── Controller/           # TaskController
│   ├── Service/              # TaskService (business logika)
│   ├── Repository/           # TaskRepository (přístup k DB)
│   └── Model/                # Task (entita)
├── templates/tasks.php       # Bootstrap 5 frontend
└── docker-compose.yml        # nginx + PHP-FPM
```

---

## 3. První session + `/init`

Spusť Claude Code **v kořeni dílu** (ne v `task-library/`) — odtud Claude uvidí `README.md`, `DEMO.md` i celou aplikaci:

```bash
cd t420-01-zaklady-cld
claude
```

> ℹ️ Aplikaci (`docker compose`) startuješ z `task-library/` v samostatném terminálu, ale **Claude spouštíš z kořene dílu**. Proto jsou cesty v promptech níže relativní ke kořeni — tj. `@task-library/src/...`.

První věc, kterou v novém projektu uděláš, je `/init`:

```
> /init
```

**Co `/init` dělá:** Claude projde tvůj projekt a vygeneruje (nebo aktualizuje) soubor **`CLAUDE.md`** — paměť projektu. Ten se načítá na začátku každé session a zůstává v kontextu pořád. Je to místo, kam patří popis stacku, příkazy, konvence a „co nedělat".

> V našem projektu už `CLAUDE.md` přiložený mám (mrkni na něj). Můžeš ho buď nechat, nebo zkusit `/init` a porovnat, co Claude vygeneruje sám. Dobré cvičení: nech Claude `/init` a pak ho popros, ať tam doplní pravidlo „business logika patří do Service, ne do Controlleru".

### Vyzkoušej: nech Claude přečíst codebase

```
> Vysvětli mi architekturu téhle aplikace. Jak teče request od HTTP až po databázi?
```

Claude si přečte `index.php`, router, controller, service, repository a popíše ti tok. Tohle je nejlepší způsob, jak naskočit do cizí codebase.

---

## 4. `@` — mentions souborů a složek

`@` = Mention files/folders. Místo kopírování obsahu souboru do promptu na něj **odkážeš** přes `@`:

```
> Vysvětli mi, co dělá @task-library/src/Service/TaskService.php
```

```
> Projdi celou složku @task-library/src/Repository/ a zkontroluj, jestli jsou všechny SQL dotazy přes prepared statements
```

```
> Porovnej @task-library/src/Controller/TaskController.php se @task-library/src/Service/TaskService.php — neporušuje něco vrstvení?
```

**Proč `@` místo copy-paste:** Claude soubor načte přes nástroj, lépe ho tokenizuje a hlavně v audit logu je vidět konkrétní cesta. Pro celé složky je to nenahraditelné.

Můžeš odkázat i víc věcí naráz:
```
> Mrkni na @task-library/src/Model/Task.php a @task-library/src/Repository/TaskRepository.php a navrhni, jak přidat pole `priority`
```

---

## 5. Bash mód `!` — spouštění příkazů

`!` = Bash mode prefix. Když řádek začneš `!`, Claude Code ho spustí jako shell příkaz (po tvém schválení) a výstup vidíš rovnou:

```
> !cd task-library && docker compose ps
```

```
> !find task-library -name "*.php" -not -path "*/data/*" -exec php -l {} \;
```

Tohle je super pro rychlé ověření stavu, aniž bys opouštěl session. Praktický příklad na naší aplikaci — ověř, že kód prošel lintem:

```
> !php -l task-library/src/Service/TaskService.php
```

---

## 6. Line break `\` a další klávesy

| Klávesa | Co dělá | Praktické použití |
|---|---|---|
| `\` + Enter | Line break — víceřádkový prompt | Když píšeš delší zadání s víc body |
| `Esc` | Interrupt Claude | Claude se vydal špatným směrem → přerušíš ho, aniž bys zabil session |
| `Esc` `Esc` | History navigation | Vrátíš se k předchozím promptům |
| `Ctrl+R` | Full output / context | Rozbalí zkrácený výstup (např. dlouhý diff) |
| `Ctrl+V` | Paste image | Vložíš screenshot (např. chybu z prohlížeče) |

**Praktická ukázka `Esc`:** Zadej Claude velký refaktor a v půlce ho přeruš `Esc`. Uvidíš, že session žije dál a můžeš upřesnit zadání — nemusíš začínat znovu.

**Praktická ukázka `Ctrl+V`:** Rozbij něco v UI (např. v `task-library/templates/tasks.php` umaž `</div>`), vyfoť rozbitou stránku z prohlížeče a vlož screenshot přes `Ctrl+V`:
```
> [vložený screenshot] Tahle stránka se rozbila, najdi proč
```

---

## 7. Plan mode vs. yolo mode — dva režimy práce

Tohle je klíčový koncept. Claude Code má režimy, jak moc se tě ptá před akcí:

### Plan mode (`Shift+Tab` `Tab`)

`Shift+Tab+Tab` = Plan mode. Claude **nejdřív navrhne plán** a teprve po tvém odsouhlasení něco udělá. Ideální na složité, vícekrokové změny:

```
[přepneš do Plan mode]
> Přidej úkolům pole `priority` (low/medium/high) — od migrace DB, přes Model, Repository, Service, až po zobrazení v Bootstrap UI
```

Claude ti vrátí plán („1. upravím schéma v Database.php, 2. přidám pole do Task modelu, 3. …") a počká. Ty plán schválíš nebo upravíš. **Žádný kód se nezmění, dokud plán neschválíš.**

> 🎯 **Doporučení pro CTO/seniory:** Na cokoliv napříč více soubory používej Plan mode. Ušetří ti to „Claude něco udělal, ale ne to, co jsem chtěl".

### Auto-accept / „yolo mode" (`Shift+Tab`)

`Shift+Tab` = Auto-accept ("yolo mode"). Claude přestane čekat na schválení každé editace a jede sám. Rychlé, ale **používej s rozmyslem** — hodí se na rutinu, kde Claude nemůže nic rozbít (formátování, generování boilerplate v izolované složce).

> ⚠️ Yolo mode v ostrém repu bez commitnutého stavu = recept na průšvih. Pravidlo: **nejdřív `git commit`, pak yolo.**

---

## 8. `/clear` — čistý kontext

`/clear` = Clear history. Když dokončíš jeden úkol a jdeš na nesouvisející druhý, **vyčisti kontext**:

```
> /clear
```

**Proč:** Claude si nese celou historii session v kontextu. Když 20 minut řešíš databázi a pak skočíš na CSS, stará historie jen plete a žere kontextové okno. `/clear` = čistý štít. Nová session na nové téma.

---

## Shrnutí dílu 01

Teď umíš: nainstalovat a aktualizovat Claude Code, spustit první session, vygenerovat `CLAUDE.md` přes `/init`, odkazovat soubory přes `@`, spouštět shell přes `!`, přerušit Claude přes `Esc`, rozbalit výstup přes `Ctrl+R`, vložit screenshot přes `Ctrl+V`, a hlavně rozdíl mezi **Plan mode** (rozmyslet napřed) a **yolo mode** (jet sám). A `/clear` mezi úkoly.

V dílu 02 půjdeme na slash příkazy do hloubky a napíšeme si vlastní příkaz.

---

## 🤓 7 zajímavostí o tomto projektu

1. **Žádný Composer, žádný framework — a je to schválně.** Celá „infrastruktura" (autoloader, router, správa DB) má dohromady pár desítek řádků vlastního kódu. Uvidíš tak, co za tebe frameworky normálně dělají — a že PSR-4 autoloader je ve skutečnosti jedna funkce v `autoload.php`.

2. **Databáze se vytvoří sama.** Soubor `data/tasks.sqlite` v repu nenajdeš — vznikne při úplně prvním requestu. `Database::connection()` je singleton, který si při prvním zavolání vytvoří schéma a naseeduje 3 ukázkové úkoly. Smažeš-li DB, appka si ji při dalším načtení stránky postaví znovu.

3. **DI „chudého muže".** Žádný dependency injection kontejner — každá vrstva má svou závislost jako default hodnotu v konstruktoru (`TaskService $service = new TaskService()`). V produkci se řetěz Controller → Service → Repository složí sám, v testech závislost prostě předáš ručně.

4. **Router umí jediný parametr: `{id}`.** Zápis `{id}` v cestě se překládá na regex `(?P<id>\d+)` — nic víc. Žádné wildcardy, žádné anotace; všechny routy jsou vypsané jako tabulka v `public/index.php`. Schválně minimalistické, ať je celý routing pochopitelný na jedno přečtení.

5. **Šablony jsou `extract()` + `require`.** Žádný Twig ani Blade — `TaskController::render()` rozbalí pole dat do lokálních proměnných a inkluduje obyčejný PHP soubor. Přesně takhle fungovaly šablonovací enginy pod kapotou, než dostaly vlastní syntaxi.

6. **Audit log je append-only.** `AuditLogRepository` má jen `append()` a `all()` — update ani delete neexistují, protože auditní stopa, do které se dá zpětně sahat, ztrácí důvěryhodnost. A `task_id` je schválně bez cizího klíče, aby záznam přežil i smazání úkolu, o kterém vypovídá.

7. **Dva kontejnery na jednu mini aplikaci.** nginx a PHP-FPM běží odděleně a baví se spolu přes FastCGI — stejné uspořádání jako na produkčních serverech, jen v malém. Díky bind-mountu se změny v kódu projeví hned, bez rebuildu image.

---

## ✅ Test dílu 01

Odpovědi jsou schované pod otázkami — nejdřív si odpověz sám.

**1. Jakým příkazem aktualizuješ Claude Code a jak často to má smysl dělat?**

<details><summary>Odpověď</summary>

`claude update`. Claude Code vychází prakticky každý týden, takže má smysl aktualizovat klidně jednou týdně — spousta nesrovnalostí mezi návody a realitou je způsobená starou verzí.
</details>

**2. Co dělá `/init` a jaký soubor vytváří?**

<details><summary>Odpověď</summary>

Projde projekt a vygeneruje/aktualizuje **`CLAUDE.md`** — paměť projektu, která se načítá na začátku každé session a zůstává v kontextu. Patří do ní stack, příkazy, konvence a „co nedělat".
</details>

**3. Proč je lepší použít `@task-library/src/Service/TaskService.php` než zkopírovat obsah souboru do promptu?**

<details><summary>Odpověď</summary>

Claude soubor načte přes nástroj (lepší tokenizace), cesta se objeví v audit logu, a u celých složek je `@` jediná rozumná cesta. Copy-paste je náchylnější k chybám a zaplevelí prompt.
</details>

**4. Jaký je rozdíl mezi Plan mode (`Shift+Tab+Tab`) a yolo mode (`Shift+Tab`)?**

<details><summary>Odpověď</summary>

**Plan mode** = Claude nejdřív navrhne plán a počká na schválení; žádný kód se nezmění, dokud plán neodsouhlasíš. Ideální na vícekrokové změny. **Yolo mode (auto-accept)** = Claude nečeká na schválení každé editace a jede sám; rychlé, ale rizikové — používat jen na rutinu a po commitu.
</details>

**5. Začneš `!` na začátku řádku. Co se stane?**

<details><summary>Odpověď</summary>

Je to **bash mode prefix** — Claude Code zbytek řádku spustí jako shell příkaz (po schválení) a ukáže ti výstup, aniž bys opustil session. Např. `!cd task-library && docker compose ps`.
</details>

**6. Pracoval jsi 20 minut na databázové vrstvě a teď chceš řešit CSS. Co uděláš s kontextem a proč?**

<details><summary>Odpověď</summary>

Použiješ `/clear`. Stará historie (databáze) by jen plnila kontextové okno a pletla Claude při nesouvisejícím úkolu (CSS). `/clear` dá čistý štít pro nové téma.
</details>

**7. Claude se při generování vydal špatným směrem. Jaká klávesa ho zastaví, aniž bys zabil session?**

<details><summary>Odpověď</summary>

`Esc` (Interrupt Claude). Session žije dál, jen upřesníš zadání. (`Esc` `Esc` tě navíc pustí do historie promptů.)
</details>

→ Pokračuj na [Díl 02 — Slash příkazy a vlastní příkazy](../t420-02-slash-prikazy-cld/README.md)
