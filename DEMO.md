# DEMO.md — Jak tohle prezentovat někomu, kdo nezná AI ani Claude

> Scénář živé ukázky. Cílem **není** naučit diváka příkazy Claude Code (na to je `README.md`).
> Cílem je, aby člověk, který nikdy neslyšel o AI ani o Claude, **pochopil, co se děje, a řekl „aha, to dává smysl"**.
> Počítej s ~20–30 minutami. Vše se dá zkrátit — sekce jsou označené 🟢 nutné / 🟡 když je čas.

---

## 0. Než přijde divák — příprava (udělej 10 minut předem)

Nic z tohohle nedělej živě před publikem, byla by to nuda a může se to zaseknout.

```bash
cd t420-01-zaklady-cld/task-library
docker compose up --build        # počkej, až naběhne
```

Checklist připravenosti:

- [ ] V prohlížeči otevřená a funkční stránka **http://localhost:8080** (vidíš „Seznam úkolů", progress bar, seznam).
- [ ] Vyzkoušej ručně přidat a odškrtnout úkol — ať víš, že appka žije.
- [ ] Otevřený **terminál v kořeni dílu `t420-01-zaklady-cld/`** (odtud budeš spouštět `claude`), dost velké písmo (Ctrl/Cmd + `+`), tmavé pozadí. Aplikace přitom běží z předchozího kroku (`docker compose` v `task-library/`) — klidně v jiném okně/záložce.
- [ ] Claude Code **nainstalovaný a přihlášený** (`claude --version` projde). Přihlašování nikdy nedělej živě.
- [ ] `git status` je čistý (kdyby divák chtěl vidět „jak se to vrátí zpět").
- [ ] Zavři si notifikace, Slack, e-mail. Sdílíš obrazovku.
- [ ] Připrav si **jednu rozbitou věc do zálohy** (viz sekce 6) pro efekt „on to opraví".

> 💡 Zlaté pravidlo dema: **měj otevřené jen dvě okna** — prohlížeč s aplikací a terminál. Nic víc. Přepínání mezi deseti okny diváka ztratí.

---

## 1. Rámování: co vůbec uvidí (2 min, 🟢 nutné)

Než cokoliv spustíš, řekni tři věty. Nezačínej slovem „AI" — začni tím, co divák zná.

> „Ukážu ti program, se kterým se **píše jako s kolegou přes chat** — jenže ten kolega umí číst a upravovat soubory v počítači a spouštět příkazy. Napíšu mu česky, co chci, on to udělá a ukáže mi výsledek. Budeme to zkoušet na téhle malé webové appce na úkoly."

Tři pojmy, které musí zaznít **v řeči diváka**, ne v žargonu:

| Pojem (žargon) | Jak to říct člověku, co nezná AI |
|---|---|
| AI / LLM | „Program, který rozumí normální řeči a umí psát text a kód. Neťukám příkazy, píšu větu." |
| Claude | „Konkrétní taková AI od firmy Anthropic. Jako je Chrome jeden konkrétní prohlížeč." |
| Claude Code | „Verze Claude, která **sedí přímo v mém počítači** a smí sahat na soubory a spouštět příkazy — ne jen chatovat v okně na webu." |

**Čeho se vyvaruj:** slova „model", „token", „kontextové okno", „LLM", „agent". Pro první dojem jsou to zabijáci pozornosti. Přijdou později, když se zeptá.

---

## 2. Nejdřív ukaž „na co" — samotnou aplikaci (2 min, 🟢 nutné)

Divák musí nejdřív vidět **normální program**, aby ocenil, že do něj AI zasahuje.

1. Přepni do prohlížeče na http://localhost:8080.
2. Ukaž myší: „Tohle je obyčejná webová aplikace na úkoly. Nahoře **progress bar**, dole **seznam úkolů**. Můžu přidat úkol, odškrtnout, smazat."
3. Klidně jeden přidej a odškrtni — ať je jasné, že je to živé, ne obrázek.

Jedna věta o tom, co je „pod kapotou", ale bez detailů:

> „Je to napsané v PHP, běží to v Dockeru. Nemusíš tomu rozumět — důležité je, že je to **skutečný projekt z víc souborů**, ne hračka na jedné stránce."

> 🎯 Pointa téhle sekce: divák si zapamatuje „appka na úkoly, funguje". Teď má s čím porovnávat.

---

## 3. První kouzlo: AI si přečte cizí projekt (3 min, 🟢 nutné)

Tohle je nejlepší úvodní „wow" — je neohrozitelné (nic nemění) a ukazuje hlavní hodnotu.

Spusť Claude v terminálu **v kořeni dílu** (ne v `task-library/`):

```bash
cd t420-01-zaklady-cld
claude
```

> ℹ️ Claude spouštíš z **kořene dílu `t420-01-zaklady-cld/`** (viz sekce 0). Proto jsou cesty v promptech níže relativní ke kořeni — tj. `@task-library/src/...`.

Řekni: „Teď mu nedám žádný příkaz. Napíšu mu **česky větu**, jako bych psal kolegovi." A napiš:

```
Vysvětli mi, co je tohle za aplikaci a jak funguje. Piš jednoduše, jako pro netechnického člověka.
```

Co komentovat, **zatímco Claude pracuje** (nenech ticho):

- „Vidíš, jak si sám **otevírá soubory**? Nikdo mu neřekl které. On se rozhodl, že se podívá do `index.php`, pak do dalších."
- „Nekopíroval jsem mu nic. On si ten projekt čte sám, jako by ho dostal nový člověk v práci."

Až doběhne, přečti nahlas 2–3 věty z jeho odpovědi. Pak pointa:

> „Představ si, že nastoupíš do práce na cizí projekt. Tohle je nový kolega, kterému za 15 vteřin řekneš ‚vysvětli mi to' — a on ti to vysvětlí."

---

## 4. Odkaz na konkrétní soubor přes `@` (2 min, 🟡 když je čas)

Ukaž, že mu můžeš cíleně říct, na co se má koukat — pomocí `@`.

```
Vysvětli mi jednoduše, co dělá @task-library/src/Service/TaskService.php
```

Řekni: „Ten zavináč je jako **přetáhnout soubor do chatu**. Říkám mu ‚koukni přesně sem'." Když se objeví nabídka souborů při psaní `@`, ukaž, že si vybírá ze skutečných souborů projektu.

> Nespěchej sem, pokud publikum tápe už u sekce 3. Tohle je „hezké mít", ne must-have.

---

## 5. Vrchol dema: nech ho něco SKUTEČNĚ změnit (5 min, 🟢 nutné)

Tady se láme chleba — divák uvidí, že to není jen chytré vyhledávání, ale že to **dělá práci**. Použij **Plan mode**, protože je to zároveň nejlepší ukázka bezpečnosti.

1. Řekni: „Teď ho poprosím, ať aplikaci **vylepší**. Ale nechci, aby hned bušil do kódu — nejdřív ať mi **řekne plán**. Na to má speciální režim."
2. Přepni do Plan mode: **`Shift+Tab`, `Tab`** (řekni nahlas „přepínám do režimu, kdy nejdřív plánuje, nesahá na nic").
3. Zadej úkol — vyber jeden, drž se malého a viditelného:

   ```
   Přidej k úkolům možnost priority (nízká, střední, vysoká) a u vysokých ať se v seznamu zobrazí červený štítek.
   ```

4. Claude vrátí **plán** („1. upravím databázi, 2. přidám pole do modelu, …"). Přečti ho nahlas a řekni pointu:

   > „Všimni si — **zatím nezměnil ani písmenko**. Ptá se mě, jestli s tím souhlasím. Tohle je jako když ti řemeslník řekne postup dřív, než začne bourat."

5. Plán **schval**. Teď Claude edituje soubory. Komentuj:
   - „Vidíš ty zelené a červené řádky? To jsou **konkrétní změny v souborech**, které navrhuje. Každou mi ukazuje."
6. Až doběhne, **přepni do prohlížeče a dej refresh (F5)**. Ukaž novou funkci naživo.

> 🎯 Pointa: „Řekl jsem mu česky větu → on rozmyslel plán → počkal na moje ‚ano' → změnil pět souborů → funguje to v prohlížeči. To celé za dvě minuty."

> ⚠️ **Pozor na kolizi s README.** Stejný příklad (přidání pole „priority") používá i `README.md` v sekci Plan mode. Pokud prezentuješ obojí, nespouštěj demo z už změněného stavu — mezi ukázkami vrať projekt zpět: `git checkout -- task-library/` (příp. zvol jiné vylepšení, třeba „přidej k úkolům datum splnění a zvýrazni úkoly po termínu").

**Pokud se výsledek nepovede na první dobrou** (může se stát — je to živě): neber to jako průšvih, **udělej z toho ukázku**. Napiš mu „Nefunguje mi to takhle: [popiš] — oprav to." a nech ho opravit. Divák uvidí, že se s tím **normálně konverzuje**, jako s kolegou. To je věrohodnější než dokonalé demo.

---

## 6. Záložní trik: „on to opraví" (3 min, 🟡 když je čas / když selže sekce 5)

Silná pointa pro netechnické publikum: AI umí **najít a spravit chybu**.

1. Předem (nebo živě, pokud si věříš) rozbij šablonu — třeba v `task-library/templates/tasks.php` smaž jeden `</div>`.
2. Ukaž rozbitou stránku v prohlížeči.
3. **Vyfoť ji / udělej screenshot** a vlož do Claude přes `Ctrl+V`:

   ```
   [vložený screenshot] Tahle stránka se mi rozbila. Najdi proč a oprav to.
   ```

4. Řekni: „Dal jsem mu **obrázek** rozbité stránky. On z něj pozná problém, najde ho v kódu a opraví." Refresh → funguje.

> Tohle publikum miluje, protože „poslat screenshot a říct ‚oprav to'" je něco, co si dokáže představit každý.

---

## 7. Ukaž, že je to bezpečné a vratné (2 min, 🟡 když je čas)

Netechnický divák má strach „a co když to rozbije počítač". Ukaž dvě jistoty:

- **Ptá se před akcí.** „Viděl jsi, že u plánu i u příkazů čeká na moje potvrzení. Nedělá věci potají."
- **Jde to vrátit.** V terminálu (bash mód `!`) ukaž, že změny jsou ve verzování:

  ```
  !git status
  ```

  „Každá změna je zaznamenaná. Jedním příkazem vrátím projekt do stavu před demem, jako by se nic nestalo."

> Pokud publikum není technické vůbec, stačí věta: „Všechno, co udělá, se dá vrátit jedním krokem zpět. Nic není nevratné."

---

## 8. Uzavření: co si má odnést (2 min, 🟢 nutné)

Shrň to do tří vět, v řeči diváka, **bez žargonu**:

1. „Není to psaní příkazů — je to **konverzace česky** s něčím, co umí sahat na projekt."
2. „Nejdřív si to **přečte a rozmyslí**, teprve po mém odsouhlasení mění kód — a všechno jde vrátit."
3. „Zvládne to, co by jinak dělal programátor: pochopit cizí projekt, přidat funkci, najít chybu."

Pak nabídni další krok podle publika:
- Zvědavý laik → „Zkus si to sám, napíšeš mu jednu větu."
- Manažer → „Tohle zrychluje práci vývojáře, ne že ho nahrazuje — pořád rozhoduje člověk (viděls to schvalování)."
- Vývojář → odkaž na `README.md`, kde jsou všechny příkazy a zkratky.

---

## Časté otázky publika a jak na ně (odpovídej lidsky, ne technicky)

| Otázka diváka | Krátká odpověď bez žargonu |
|---|---|
| „Ono to přemýšlí?" | „Ne jako člověk. Napodobuje, jak lidé řeší úlohy, protože se to učilo z ohromného množství textu a kódu. Ale výsledky vypadají, jako by přemýšlelo." |
| „Vezme to práci programátorům?" | „Zatím spíš jako kalkulačka účetním — zrychlí rutinu, ale rozhodnutí a kontrola zůstávají na člověku. Viděls, že jsem každý krok schvaloval." |
| „Kde ten kód bere? Nekrade ho?" | „Napíše ho na míru tvému projektu. Učilo se z veřejného kódu, podobně jako se člověk učí programovat z tutoriálů." |
| „Může to smazat/rozbít věci?" | „Ptá se před akcí a všechno je verzované — jde vrátit. Proto se vážné věci pouští v ‚plánovacím' režimu, co jsem ukázal." |
| „Rozumí i jinak než anglicky?" | „Ano, celé demo jsem psal česky. Rozumí běžné řeči, ne příkazům." |
| „Kolik to stojí / kde to běží?" | „Placená služba od firmy Anthropic, počítá se u nich na jejich serverech. Na první osahání stačí volný účet." |
| „Vždycky to má pravdu?" | „Ne. Občas se splete nebo si domyslí. Proto to člověk kontroluje a schvaluje — není to automat, je to pomocník." |

---

## Čeho se při prezentaci vyvarovat

- ❌ **Nezačínej instalací a přihlašováním.** Nudné a rizikové živě. Měj hotovo.
- ❌ **Nečti mu odpovědi celé nahlas.** Vyber 2–3 věty, zbytek shrň.
- ❌ **Nepřepínej deset oken.** Jen prohlížeč + terminál.
- ❌ **Neomlouvej se za pomalost.** Když Claude přemýšlí, komentuj, co dělá („teď si čte soubory") — mlčení působí jako zásek.
- ❌ **Netvrz, že je neomylný.** Když se splete, udělej z toho přednost: „vidíš, konverzuje se s tím dál, jako s kolegou".
- ❌ **Nezahlť žargonem.** Token/model/kontext/agent zmiňuj jen, když se někdo výslovně zeptá.

---

## Bleskový scénář na 5 minut (když máš málo času)

1. Ukaž appku v prohlížeči (30 s).
2. `cd t420-01-zaklady-cld && claude` → „Vysvětli mi jednoduše, co je tohle za aplikaci." (90 s).
3. Plan mode → „Přidej úkolům prioritu s červeným štítkem u vysokých." → schval → refresh v prohlížeči (2,5 min).
4. Věta na závěr: „Napsal jsem česky větu, on rozmyslel plán, počkal na moje ano, změnil kód — a funguje to." (30 s).
