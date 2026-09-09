# FRQ-WFL-15.9 — Generički workflow i tvornica

| | |
|---|---|
| **Status** | Provedeno u kodu, zapisano 2026-08-27 — obrnuto inženjerstvo zatečenog |
| **Odluka** | [`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) — kategorija `WFL`; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-15`](../01-HLRQ/HLRQ-15-generic-layer.md) — narativ, `BR-15-01…BR-15-09`, zajednički ugovor generičke klase (§4) |
| **Predmet** | `GenericWorkflow(Wattleflow, IOriginator, ABC)`, `WorkflowFactory`, `WorkflowFactoryLogger`, `WorkflowFactoryException` |
| **Sestrinski** | [`FRQ-PRC-15.3`](FRQ-PRC-15.3-processor.md) (tvornica ih gradi i uvezuje) · `FRQ-PTN-15.14` *(nenapisan)* (tri menadžera) |
| **Izvedba** | `workflow/src/wattleflow/concrete/workflow.py` (492 linije) |

## 1. Predmet

Workflow je **cjelina**, a tvornica je mjesto na kojem `BR-15-01` postaje istinit ili ne postaje:
zamjena bilo kojeg primitiva je izmjena konfiguracije samo ako postoji jedno mjesto koje ime iz
konfiguracije prevodi u klasu. To mjesto je `WorkflowFactory.resolve`.

Podjela je namjerno oštra:

| klasa | što je | zašto tako |
|---|---|---|
| `GenericWorkflow` | **objekt frameworka** — `Wattleflow` + `IOriginator` | drži tri menadžera i izvodi se |
| `WorkflowFactory` | **obična klasa**, ne `Wattleflow` | tvornica nije sudionik toka; gradi ga izvana |
| `WorkflowFactoryLogger` | `Wattleflow` bez tijela | tvornica ipak treba auditirati, pa posuđuje objekt koji to zna |

Tvornica gradi **odozdo prema gore**, redoslijedom ovisnosti: konekcije → driveri (koji dobivaju
menadžer konekcija) → procesori (koji dobivaju drivere) → i unutar procesora: pipelinei →
blackboard sa `strategy_create` → spremišta sa `strategy_write` i driverom.

**Konfiguracija se čita relativno.** `sections` je ključna putanja okolišnog bloka (npr.
`("infrastructure", "dev")`); svaki dohvat ide pod njom, pa sam adapter ostaje neopsežen i
jednostavan.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | Pozivatelj (skripta, notebook) | `adapter`, `sections`, opcionalno `workflow_name` |
| **A2** | `IConfig` adapter — YAML/JSON izvor | `find(*path, default=…)` |
| **A3** | `WorkflowFactory` — predmet ovog zahtjeva | registar imena → klasa |
| **A4** | `ConnectionManager` / `DriverManager` / `ProcessorManager` | registri izgrađenih primitiva |
| **A5** | `GenericWorkflow` — izgrađena cjelina | `execute()` |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | klasa se registrira → `register(name, class)` |
| **EV02** | A1 traži gradnju → `build(adapter=…, sections=…)` |
| **EV03** | ime iz konfiguracije se razrješava → `resolve(type_name)` |
| **EV04** | `runtime:` blok se primjenjuje → `_apply_runtime_env` |
| **EV05** | A1 pokreće → `workflow.execute()` |
| **EV06** | ime se ne razriješi → `WorkflowFactoryException` |

## 4. Preduvjeti

1. Svaka klasa koja se smije pojaviti u konfiguraciji je registrirana (`EV01`).
2. `adapter` implementira `IConfig`; `sections` nije `None`.
3. `workflows` blok postoji pod `sections`; ako ih je više, `workflow_name` bira jedan.
4. Svaki unos deklarira `type:` koje odgovara registriranom imenu (`BR-15-01`).

## 5. Normalan tok

1. **EV02** — `adapter` i `sections` se provjere; nedostatak jednog je kvar prije ijedne gradnje
   (`BR-15-02`).
2. `workflows` se dohvati pod `sections`. Lista s jednim unosom uzima se bez imena; s više unosa
   traži se `workflow_name`.
3. **EV03** — klasa workflowa se razriješi kroz `_resolve_section`, koji kvar obogaćuje **sekcijom
   i imenom unosa** — poruka kaže *gdje* je u konfiguraciji problem, ne samo *koje* ime fali.
4. **EV04** — `runtime:` blok se preslikava u varijable okoline: poznati ključevi kroz
   `_RUNTIME_KEY_TO_ENV` (`tika_server_jar` → `TIKA_SERVER_JAR`, `java_home` → `JAVA_HOME`…),
   ostatak iz `runtime.env` doslovno.
5. Globalne postavke zapisa (`level`, `handler`, `format`) čitaju se jednom i prosljeđuju svakom
   izgrađenom objektu; svaki unos ih smije nadjačati kroz `_audit`.
6. Gradnja redom: `_build_connections` → `_build_drivers` → `_build_processors`. Driver koji
   deklarira `connection_name` dobiva i `connection_manager`, pa svoju konekciju razrješava pri
   učitavanju.
7. Procesori se grade zajedno sa svojim pipelineima, blackboardom i spremištima; spremište dobiva
   razriješenu instancu drivera po imenu, ne ime.
8. **EV05** — `execute()` je apstraktan: specijalizacija odlučuje što „pokreni" znači.

## 6. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| `adapter` ne implementira `IConfig` | `WorkflowFactoryException` | gradnja ne počinje |
| `sections` je `None` | `WorkflowFactoryException` | konfiguracija bez opsega se odbija |
| workflow nije nađen po imenu | `WorkflowFactoryException` s imenom | — |
| `type:` nedostaje | poruka nabraja **koje** vrste unosa ga moraju imati | kvar konfiguracije je čitljiv |
| `type:` nije registriran | poruka nudi **do tri bliska imena** (`difflib`), popis prvih 20 registriranih i broj svih | tipfeler se ispravlja bez čitanja koda |
| kvar u pod-unosu | `_resolve_section` dodaje sekciju i `name` unosa, pa ponovno diže `from e` | uzrok očuvan (`BR-15-09`) |
| nema procesora u workflowu | `WorkflowFactoryException` | workflow bez posla se ne gradi |
| unos procesora nije rječnik | `WorkflowFactoryException` | — |
| `runtime` vrijednost je prazna ili `None` | preskače se | prazna postavka ne briše okolinu |

## 7. Rezultat

Iz jedne konfiguracijske datoteke nastaje uvezana cjelina: konekcije, driveri i procesori s
pipelineima, platnima i spremištima — svaki razriješen po imenu. Zamjena bilo kojeg dijela je
izmjena `type:` u konfiguraciji, bez dodirivanja koda (`BR-15-01`), a kvar konfiguracije zaustavlja
gradnju prije obrade (`BR-15-02`).

## 8. Kriteriji prihvaćanja

1. Svaki primitiv se razrješava po imenu iz jednog registra. ✅
2. Nerazriješeno ime zaustavlja gradnju i poruka imenuje sekciju, unos i bliske kandidate. ✅
3. Gradnja ide redoslijedom ovisnosti; driver dobiva menadžer konekcija kad ga treba. ✅
4. Globalne postavke zapisa se prosljeđuju svakom objektu i mogu se nadjačati po unosu. ✅
5. Tvornica nije objekt frameworka, ali auditira kroz namjenski objekt. ✅
6. Modul deklarira `__all__`; import closure je `stdlib ∪ wattleflow`. ✅
7. Zapisi tvornice su vidljivi na zadanoj konfiguraciji. ❌ — §11 t.1
8. Inline konfiguracijski ključevi rade jednako za sve vrste unosa. ❌ — §11 t.2
9. `__slots__` ne ponavlja slotove baze. ❌ — §11 t.3
10. Vrijednost iz `runtime:` ne ulazi u zapis (`NFRQ-SEC-06`). ❌ — §11 t.4

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-27) |
|---|---|---|
| 1 | pregled `_registry` i `resolve` | jedan `dict[str, type]`, jedan ulaz |
| 2 | pregled `resolve` / `_resolve_section` | `difflib.get_close_matches(…, n=3, cutoff=0.6)`; poruka nosi `section` i `name` |
| 3 | pregled `build` | tri poziva redom; `configuration.setdefault("connection_manager", connections)` |
| 4 | pregled `_audit` | `config.get(k, default[k])` po ključu |
| 5 | pregled | `class WorkflowFactory:` bez baze; `logger = WorkflowFactoryLogger(level="ERROR", …)` |
| 6 | pregled modula | `__all__` = 4 imena; uvozi `os`, `difflib`, `abc`, `logging`, `typing` + `wattleflow.*` |
| 7 | pregled razine modulskog loggera | `level="ERROR"` — `logger.info` i `logger.debug` nikad ne prolaze |
| 8 | `command grep -n '_settings(' concrete/workflow.py` | definiran jednom, pozvan **jednom** — samo za blackboard |
| 9 | usporedba `__slots__` | `GenericWorkflow` ponavlja `_level` i `_handler` iz `Audit.__slots__` |
| 10 | pregled `_apply_runtime_env` | `logger.debug(…, value=str(value))` za svaki ključ, uključujući `runtime.env` |

**Trojka reproducibilnosti (D-10):** alat — čitanje koda i `command grep`; kriterij — §8 gore;
platforma — `workflow` radno stablo 2026-08-27, CPython 3.11 (Linux/WSL2). **Mjereno stablo:**
`concrete/workflow.py` + `helpers/audit.py`.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica za ovaj zahtjev |
|---|---|
| `NFRQ-ORG-04` | `Workflow` je rezervirani primitiv; tvornica je pattern oko njega, ne novi primitiv |
| `NFRQ-SEC-01` | registar je jedina točka kroz koju ime iz konfiguracije postaje izvršni kod — jedno mjesto za nadzor |
| `NFRQ-SEC-02` | `STRUCTURAL` razdvaja ključeve koje tvornica troši od onih koje prosljeđuje; `PresetGate` odbija nepoznate |
| `NFRQ-SEC-03` | clean core tier; `_RUNTIME_KEY_TO_ENV` **imenuje** third-party alate (Tika, Spark) ali ih ne uvozi |
| `NFRQ-SEC-06` | `runtime:` vrijednosti se zapisuju doslovno — vidi §11 t.4 |
| `NFRQ-OBS-01` | jedan `INFO` po gradnji (broj konekcija, drivera, procesora) — koji se na zadanoj razini ne vidi (§11 t.1) |
| `NFRQ-ORG-08` | `_audit` i `_resolve_section` postoje upravo zato da se obrasci ne prepisuju po vrstama unosa |

## 11. Otvoreno

1. **Zapisi tvornice su nevidljivi.** Modulski `logger = WorkflowFactoryLogger(level="ERROR", …)`
   nastaje pri uvozu s razinom `ERROR`, pa `logger.info(msg=Event.Build, connections=…, drivers=…,
   processors=…)` i svaki `logger.debug` nikad ne izađu. Jedini `INFO` koji cijela gradnja
   proizvodi je time mrtav, a `NFRQ-OBS-01` ga mjeri kao postojeći. Razina bi trebala doći iz iste
   konfiguracije koja se upravo čita, ne biti fiksirana pri uvozu.
2. **`_settings` se primjenjuje samo na blackboard.** Metoda spaja `configuration:` blok s
   *inline* ključevima (onima izvan `STRUCTURAL`), ali je pozvana jednom. Konekcije, driveri,
   procesori i pipelinei čitaju `config.get("configuration", {})` izravno, pa inline ključ kod njih
   tiho nestaje. Ista konfiguracijska sintaksa znači dvije stvari ovisno o tome što se konfigurira.
3. **`GenericWorkflow.__slots__` ponavlja slotove baze.** `_level` i `_handler` su već u
   `Audit.__slots__`; ponovljena deklaracija stvara drugi descriptor koji zasjenjuje bazni i troši
   dodatni prostor po instanci. Ukloniti iz podklase.
4. **`runtime:` vrijednosti ulaze u zapis doslovno.** `_apply_runtime_env` zapisuje `value=str(value)`
   za svaki ključ, uključujući proizvoljan `runtime.env` blok. Tko ondje postavi token ili
   lozinku, dobiva je u audit tragu — `BR-15-08` i `NFRQ-SEC-06` traže suprotno. Uz to, upis u
   `os.environ` je **procesni** nuspojava iz konfiguracije: dva workflowa u istom procesu se
   pregaze.
5. **`strategy_create` se gradi bez audit postavki**, `strategy_write` s njima
   (`strategy_write(**audit)` naspram `strategy_class()`). Create-strategija time pada na zadane
   postavke zapisa dok sve oko nje nosi konfigurirane.
6. **Spremište uvijek dobiva `driver=`, i kad je `None`.** `GenericRepository` taj ključ ne
   očekuje, pa `None` završi u presetu; `RepositoryWithDriver` na `None` diže `ValueError`
   (`FRQ-REP-15.7` §11 t.4). Tvornica bi ključ trebala izostaviti kad drivera nema.
7. **Zatečeni pravopis `formating`** je u `STRUCTURAL` i u `_audit` kao prihvaćeni alias. Zajedno s
   `fmt` u blackboardu (`FRQ-BBD-15.1` §11) i `formating` u konekciji (`FRQ-CON-15.5` §11 t.6) to
   su tri zatečena naziva istog pojma bez zajedničkog popisa i bez roka povlačenja.
8. **Tipfeler u komentaru:** `# Worflow Class`.
