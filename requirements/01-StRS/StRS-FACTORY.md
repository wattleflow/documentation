<link rel="stylesheet" href="../../requirements/styles/wattleflow.css">

# StRS-FACTORY — Workflow Factory: sastavljanje toka iz konfiguracije

| | |
|---|---|
| **Verzija** | v0.0.6 |
| **Razred** | Specifikacija zahtjeva dionika (StRS) — nosi narativ i poslovna pravila; ne opisuje korake |
| **Norma** | ISO/IEC/IEEE 29148:2018 — StRS; zahtjevi sustava (SyRS) su u djeci |
| **Predmet** | `concrete/factory.py`: `WorkflowFactory`, `WorkflowFactoryException` |
| **Djeca (SyRS)** | odrediti (§6) |
| **Povezano** | [`StRS-WORKFLOW`](StRS-WORKFLOW.md) — tok koji Workflow Factory sastavlja · [`StRS-MANAGER`](StRS-MANAGER.md) — upravitelji koje Workflow Factory puni · [`StRS-PROCESSOR`](StRS-PROCESSOR.md), [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md), [`StRS-REPOSITORY`](StRS-REPOSITORY.md), [`StRS-PIPELINE`](StRS-PIPELINE.md), [`StRS-DRIVER`](StRS-DRIVER.md), [`StRS-CONNECTION`](StRS-CONNECTION.md), [`StRS-STRATEGY`](StRS-STRATEGY.md), [`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md) — komponente koje Workflow Factory stvara |
| **Dijagrami** | inline (§2.2, §2.4, §2.5) — pogled izveden iz koda, ne izvor istine ([D-13](../../DOCTRINE.md#d-13)) |

## 1. Uvod

### 1.1 Svrha

Ovaj zapis objašnjava čemu služi Workflow Factory, tko ga koristi, kako iz konfiguracije nastaje gotov tok i što se događa kad konfiguracija nije ispravna.

### 1.2 Opseg

**U opsegu:** registar vrsta komponenti, čitanje konfiguracije, stvaranje konekcija, drivera, procesora, pipelineova, Blackboarda i repozitorija sa strategijama, njihova registracija i povezivanje, uključivanje kontrolnih točaka i praćenja rada, postavke okruženja te greške koje Workflow Factory javlja.

**Izvan opsega:** što stvorene komponente rade ([`StRS-WORKFLOW`](StRS-WORKFLOW.md) i zapisi pojedinih komponenata), kako se konfiguracija čita iz datoteke i što točno monitor mjeri. Tehnički detalji izvedbe nalaze se u zahtjevima sustava (SyRS).

### 1.3 Publika

Zapis je pisan za široku publiku, od poslovnog korisnika do arhitekta sustava. Za čitanje nije potrebno poznavanje programiranja ni UML-a: svaki dijagram prati kratko objašnjenje. Tehnička razrada za programere nalazi se u zahtjevima sustava (SyRS).

### 1.4 Pojmovi

| pojam | naziv u dijagramima | značenje |
|---|---|---|
| Workflow Factory | Workflow Factory | dio sustava koji iz konfiguracije stvara sve dijelove toka i sastavlja gotov workflow; isti naziv koriste svi zapisi |
| workflow | Workflow | gotov tok podataka ([`StRS-WORKFLOW`](StRS-WORKFLOW.md)) |
| konfiguracija | configuration | zapis (npr. YAML datoteka) koji opisuje što tok sadrži i kako je podešen |
| čitač konfiguracije | Config adapter | dio koji Workflow Factoryju daje sadržaj konfiguracije |
| vrsta komponente | type | ime pod kojim je u konfiguraciji naveden razred konekcije, drivera, procesora i sl. |
| registar vrsta | registry | popis imena i razreda koje Workflow Factory smije stvarati |
| unos | entry | opis jedne komponente u konfiguraciji: ime, vrsta, postavke |
| upravitelj | Manager | evidencija objekata jedne vrste po imenu ([`StRS-MANAGER`](StRS-MANAGER.md)) |
| konekcija, driver, procesor, pipeline, Blackboard, repozitorij, strategija | Connection, Driver, Processor, Pipeline, Blackboard, Repository, Strategy | komponente toka opisane u svojim zapisima |
| kontrolna točka | checkpoint | zabilježen napredak procesora iz kojeg se prekinuta obrada može nastaviti; čuva se u pohrani snimki ([`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md)) |
| monitor | Monitor | dio koji mjeri rad toka; neobvezan |
| postavke bilježenja | audit settings | razina, izlaz i oblik zapisnika koje komponenta nasljeđuje od toka ili ih navede sama |
| pokretač | Runner | dio sustava ili osoba koja traži sastavljanje toka, npr. raspoređivač |
| StRS | — | specifikacija zahtjeva dionika: što dionici trebaju i zašto; ovaj zapis |
| SyRS | — | specifikacija zahtjeva sustava: što sustav mora raditi i kako se to provjerava |

## 2. Opis sustava

### 2.1 Narativ

**Problem** Tok podataka sastoji se od mnogo dijelova koji moraju biti stvoreni, podešeni i povezani ispravnim redoslijedom. Kad bi svaki tok to radio ručno u kodu, promjena toka tražila bi izmjenu koda, a pogrešne postavke otkrile bi se tek usred rada.

**Rješenje** Workflow Factory iz konfiguracije stvara sve dijelove toka, upisuje ih u upravitelje, povezuje ih i predaje gotov workflow. Koje komponente tok ima, zadaje se imenom vrste i postavkama u konfiguraciji, pa se tok mijenja izmjenom konfiguracije, ne koda.

**Smjer** Workflow Factory ne zna konkretne komponente: zna samo registar vrsta. Programer u njega upisuje vrste, a konfiguracija ih imenuje. Workflow Factory radi samo pri sastavljanju i poslije toga ne sudjeluje u radu toka. Neispravnu konfiguraciju prijavljuje odmah, uz odjeljak i ime unosa, a ne usred izvedbe.

### 2.2 Kontekst

Pokretač (npr. raspoređivač) traži od Workflow Factoryja workflow i predaje mu čitač konfiguracije. Workflow Factory iz registra vrsta uzima razrede koje konfiguracija imenuje, stvara konekcije, zatim drivere, pa procesore s pipelineovima, Blackboardom i repozitorijima te ih upisuje u upravitelje. Driver koji komponenta treba dohvaća od upravitelja drivera po imenu i predaje ga toj komponenti. Na kraju stvara workflow i, ako je zadano, priključuje mu monitor. Pokretač dobiva gotov workflow ili grešku.

```plantuml
@startuml context-diagram-uml
!include requirements/styles/wattleflow.puml

title "Workflow Factory: Data flow"
caption "Context Diagram"

actor "Runner" as Runner
actor "Programmer" as Programmer
component "Config adapter" as Config
component "Managers" as Managers
component "Components\n(Connection, Driver, Processor,\nPipeline, Blackboard, Repository)" as Components
component "Workflow" as Workflow
component "Monitor\n(optional)" as Monitor

rectangle "Workflow Factory" as Boundary <<system boundary>> {
  component "Workflow Factory" as Factory
}

Programmer ..> Factory : registers types
Runner ..> Factory : asks for Workflow
Config --> Factory : configuration
Factory ..> Components : creates, wires
Factory ..> Managers : fills; gets drivers\nby name
Factory ..> Workflow : assembles
Factory ..> Monitor : creates, attaches
Factory --> Runner : Workflow / error
@enduml
```

### 2.3 Dionici

| akter | uloga | zadaća |
|---|---|---|
| Programer | razvija i održava vrste komponenti | upisuje nove vrste u registar da ih konfiguracija može imenovati |
| Autor konfiguracije | opisuje tok | piše konfiguraciju: koje komponente, kojim redoslijedom i s kojim postavkama |
| Pokretač | traži tok | traži od Workflow Factoryja workflow i prima ga ili grešku |
| Čitač konfiguracije | daje konfiguraciju | vraća dio konfiguracije koji Workflow Factory traži |
| Komponente toka | stvaraju se | primaju postavke i vezu s ostalim komponentama |

### 2.4 Korisnički slučajevi

#### UC1 — Registriraj vrstu komponente

| | |
|---|---|
| **Akter** | Programer |
| **Okidač** | Konfiguracija treba imenovati novu vrstu komponente |
| **Preduvjet** | Vrsta komponente postoji kao razred |
| **Glavni tijek** | 1. Programer upisuje u registar ime i razred. 2. Workflow Factory ga pamti. |
| **Alternativni tok** | A1. Upisuje se nešto što nije razred: upis se odbija, greška Workflow Factoryja. A2. Ime je već upisano za drugi razred: novi razred zamjenjuje stari, zapisuje se upozorenje. |
| **Ishod** | Konfiguracija može imenovati tu vrstu |

#### UC2 — Sastavi workflow iz konfiguracije

| | |
|---|---|
| **Akter** | Pokretač (primarni), Čitač konfiguracije (podupirući) |
| **Okidač** | Pokretač treba tok koji konfiguracija opisuje |
| **Preduvjet** | Pokretač ima čitač konfiguracije i zna koji dio konfiguracije sadrži tok; sve vrste koje konfiguracija imenuje su registrirane (UC1) |
| **Glavni tijek** | 1. Workflow Factory nalazi tok u konfiguraciji (po imenu, ili jedini). 2. Postavlja postavke okruženja i razinu bilježenja. 3. Stvara konekcije i upisuje ih u upravitelja konekcija. 4. Stvara drivere i upisuje ih u upravitelja drivera; driver koji navodi konekciju dobiva upravitelja konekcija. 5. Za svaki procesor stvara procesor (s driverom, ako ga navodi), registrira mu pipelineove i Blackboard sa strategijom stvaranja, a Blackboardu registrira repozitorije sa strategijama pisanja i čitanja; driver koji procesor ili repozitorij navodi dohvaća od upravitelja drivera po imenu. Procesor upisuje u upravitelja procesora. 6. Stvara workflow s trima upraviteljima. 7. Ako je zadano, priključuje mu monitor. |
| **Alternativni tok** | A1. Čitač konfiguracije nije ispravne vrste, ne zna se dio konfiguracije ili traženi tok ne postoji: greška Workflow Factoryja. A2. Ime vrste nedostaje ili nije registrirano: greška Workflow Factoryja s prijedlogom sličnih imena. A3. Tok nema procesora, procesor nema Blackboard sa strategijom stvaranja ili unos nije ispravan: greška Workflow Factoryja. A4. Komponenta traži driver koji nije naveden ili nije registriran: greška Workflow Factoryja. A5. Unos ima ključ koji Workflow Factory ne koristi: ključ se odbacuje uz upozorenje i uputu da ga se stavi u postavke komponente. A6. Komponenta pri stvaranju javi vlastitu grešku: greška se prosljeđuje onome tko je Workflow Factory pozvao. |
| **Ishod** | Gotov workflow s upraviteljima popunjenim komponentama, ili greška bez workflowa |

Workflow Factory drivere stvara i predaje komponentama, ali ih ne spaja (§6).

#### UC3 — Uključi kontrolne točke procesora

| | |
|---|---|
| **Akter** | Autor konfiguracije (primarni), Pokretač (podupirući) |
| **Okidač** | Autor želi da se prekinuta obrada može nastaviti |
| **Preduvjet** | Konfiguracija toka ima odjeljak za kontrolne točke |
| **Glavni tijek** | 1. Workflow Factory čita odjeljak: vrstu pohrane (memorija, datoteka ili registrirana vrsta), njezine postavke i razmak između točaka. 2. Workflow Factory stvara pohranu. 3. Svakom procesoru predaje pohranu, ključ (njegovo ime) i razmak. |
| **Alternativni tok** | A1. Odjeljak nije ispravan, vrsta pohrane nije poznata ili se ne može stvoriti, ili ključ nije dopušten: greška Workflow Factoryja. Bez odjeljka kontrolnih točaka nema. |
| **Ishod** | Procesori mogu zapisivati i čitati kontrolne točke |

#### UC4 — Uključi praćenje rada

| | |
|---|---|
| **Akter** | Autor konfiguracije (primarni), Pokretač (podupirući) |
| **Okidač** | Autor želi mjeriti rad toka |
| **Preduvjet** | Konfiguracija ima odjeljak za praćenje s razinom koja nije isključena |
| **Glavni tijek** | 1. Workflow Factory čita razinu, pragove, ograničenja, razmak mjerenja i izlaze. 2. Workflow Factory stvara monitor; izlaz koji navodi driver dobiva ga od upravitelja drivera. 3. Workflow Factory priključuje monitor workflowu. |
| **Alternativni tok** | A1. Odjeljak nije ispravan ili izlaz nema vrstu: greška Workflow Factoryja. A2. Odjeljak ima nepoznate ključeve: odbacuju se uz upozorenje. A3. Odjeljka nema ili je razina isključena: monitor se ne stvara. |
| **Ishod** | Workflow s monitorom, ili bez njega |

```plantuml
@startuml use-case-diagram
!include requirements/styles/wattleflow.puml

title "Workflow Factory: Data flow"
caption "Use Cases"

left to right direction
skinparam nodesep 10
skinparam ranksep 20

actor Programmer
actor Runner
actor "Config adapter" as Config
actor "Configuration author" as Author

rectangle "Workflow Factory" as Boundary <<system boundary>> {
    usecase "Registriraj vrstu\nkomponente" as UC1
    usecase "Sastavi workflow\niz konfiguracije" as UC2
    usecase "Uključi kontrolne točke\nprocesora" as UC3
    usecase "Uključi praćenje rada" as UC4
}

Programmer -- UC1
Runner -- UC2
UC2 -- Config
Author -- UC3
Author -- UC4
UC3 ..> UC2 : <<extend>>
UC4 ..> UC2 : <<extend>>
@enduml
```

### 2.5 Tok rada i praćenje grešaka

Workflow Factory nema stanja: svaki poziv sastavlja novi workflow. Dijagram prikazuje normalni tok (od zahtjeva do gotovog workflowa) i alternativne tokove u kojima se sastavljanje prekida greškom.

```plantuml
@startuml activity-diagram
!include requirements/styles/wattleflow.puml

title "Workflow Factory: Data flow"
caption "Flow Chart"

start
:Runner asks for a Workflow;
if (Config adapter valid\nand section given?) then (no)
  :<b><color:red>WorkflowFactoryException</color></b>;
  :error returned to Runner;
  stop
endif
if (workflow found in configuration?) then (no)
  :<b><color:red>WorkflowFactoryException</color></b>;
  :error returned to Runner;
  stop
endif
:environment and logging level set;
:create connections, then drivers;
if (every type registered\nand every driver found?) then (no)
  :<b><color:red>WorkflowFactoryException</color></b>\n(section, entry, cause);
  :error returned to Runner;
  stop
endif
if (at least one processor,\nentries valid?) then (no)
  :<b><color:red>WorkflowFactoryException</color></b>;
  :error returned to Runner;
  stop
endif
:create processors with Pipelines,\nBlackboard and Repositories;
if (a component fails\nwhile being created?) then (yes)
  :error of the component;
  :error returned to Runner;
  stop
endif
:create Workflow from three Managers;
if (monitoring switched on?) then (yes)
  if (monitoring section valid?) then (no)
    :<b><color:red>WorkflowFactoryException</color></b>;
    :error returned to Runner;
    stop
  endif
  :Monitor attached;
endif
:Workflow returned to Runner;
stop
@enduml
```

#### Praćenje grešaka

| greška ili ishod | kada nastaje | posljedica |
|---|---|---|
| `WorkflowFactoryException` | čitač konfiguracije nije ispravne vrste; nije zadan dio konfiguracije; tok nije pronađen; vrsta nedostaje ili nije registrirana; upisano nije razred; tok nema procesora ili unos nije ispravan; traženi driver nije naveden ili registriran; odjeljak kontrolnih točaka ili praćenja nije ispravan | sastavljanje se prekida i workflow se ne vraća; poruka navodi odjeljak i ime unosa, a izvorni uzrok ostaje priložen; pokretač prima grešku |
| greška komponente | komponenta pri stvaranju javi vlastitu grešku | prosljeđuje se pokretaču kakva jest |
| upozorenje | unos ima ključ koji se odbacuje; odjeljak praćenja ima nepoznate ključeve; ime vrste zamijenjeno drugim razredom | sastavljanje se nastavlja; događaj se bilježi |

Workflow Factory bilježi početak i završetak sastavljanja (s brojem konekcija, drivera i procesora) te svaku grešku konfiguracije.

### 2.6 Pretpostavke i ograničenja

- Workflow Factory ne zna konkretne komponente: zna samo registar vrsta i što je svaka vrsta zatražila.
- Workflow Factory radi samo pri sastavljanju i poslije ne sudjeluje u radu toka.
- Redoslijed je određen: konekcije, driveri, procesori, jer driver traži konekciju, a procesor i repozitorij traže driver.
- Workflow Factory drivere ne spaja; komponente ih dobivaju u stanju u kojem su stvoreni.
- Pri prekidu sastavljanja gotov workflow se ne vraća; što je dotad stvoreno, ne ulazi ni u kakav tok.
- Postavke okruženja koje konfiguracija zadaje (putanje vanjskih alata, vremenska zona) postavljaju se za cijeli proces.
- Dodatne argumente komponentama Workflow Factory ne određuje sam: može ih dodati samo pozivatelj kroz priključak, a Workflow Factory ne zna što znače.

## 3. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-FCT-01** | Tok se sastavlja samo iz konfiguracije; vrste komponenti navode se imenom, a ime mora biti registrirano. |
| **BR-FCT-02** | U registar se može upisati samo razred; zamjena imena drugim razredom bilježi se upozorenjem. |
| **BR-FCT-03** | Nedostajuće ili neregistrirano ime vrste prekida sastavljanje; poruka predlaže slična imena i navodi registrirana. |
| **BR-FCT-04** | Čitač konfiguracije mora biti ispravne vrste i mora biti zadan dio konfiguracije u kojem je tok; inače se tok ne sastavlja. |
| **BR-FCT-05** | Ako konfiguracija ima više tokova, traženi se bira po imenu, a jedini se uzima sam; ako traženog nema, tok se ne sastavlja. |
| **BR-FCT-06** | Redoslijed stvaranja: konekcije, driveri, procesori. |
| **BR-FCT-07** | Driver koji navede ime konekcije dobiva upravitelja konekcija, da je sam nađe pri spajanju. |
| **BR-FCT-08** | Komponenta koja traži driver mora ga imenom navesti u konfiguraciji, a driver mora biti registriran; Workflow Factory ga dohvaća od upravitelja drivera i predaje komponenti. Inače se tok ne sastavlja. |
| **BR-FCT-09** | Tok mora imati barem jedan procesor. |
| **BR-FCT-10** | Svaki procesor dobiva svoje pipelineove i obvezno svoj Blackboard sa strategijom stvaranja; Blackboard dobiva svoje repozitorije. Workflow Factory ih registrira (pipelineove i Blackboard kod procesora, repozitorije kod Blackboarda). |
| **BR-FCT-11** | Repozitorij mora imati strategiju pisanja, a strategija čitanja nije obvezna. |
| **BR-FCT-12** | Postavke komponente dolaze iz njezina odjeljka u konfiguraciji; ključ koji Workflow Factory ne koristi odbacuje se uz upozorenje i uputu da ga se stavi u postavke komponente. Iznimka je Blackboard, kojem se i ključevi uz unos predaju kao postavke. |
| **BR-FCT-13** | Postavke bilježenja komponenta nasljeđuje od toka, osim ako ih navede sama. |
| **BR-FCT-14** | Kontrolne točke uključuju se odjeljkom u konfiguraciji toka; ključ točke je ime procesora; neispravan odjeljak prekida sastavljanje. |
| **BR-FCT-15** | Monitor se stvara samo ako je praćenje zadano i razina nije isključena; inače ga nema. |
| **BR-FCT-16** | Postavke okruženja zadane u konfiguraciji postavljaju se prije stvaranja komponenti. |
| **BR-FCT-17** | Neuspjelo sastavljanje ne vraća workflow. Greška koju otkrije sam Workflow Factory nosi odjeljak, ime unosa i izvorni uzrok; greška koju javi komponenta prosljeđuje se kakva jest, jer pravila komponente posjeduje komponenta ([`NFRQ-ORG-15`](../03-NFRQ/NFRQ-ORG-15-separation-of-concerns.md)). |

## 4. Nefunkcionalni zahtjevi

Kakvoće proizlaze iz narativa i koda; za njih još nema poveznice na zapise zahtjeva na kakvoću (§6).

| kakvoća | što znači za Workflow Factory |
|---|---|
| podesivost | tok se mijenja izmjenom konfiguracije, ne koda (BR-FCT-01) |
| rano otkrivanje grešaka | neispravna konfiguracija javlja se pri sastavljanju, a ne usred izvedbe (BR-FCT-17) |
| razdvajanje odgovornosti | Workflow Factory provjerava samo ono što komponenta deklarira; pravila i greške komponente pripadaju komponenti ([`NFRQ-ORG-15`](../03-NFRQ/NFRQ-ORG-15-separation-of-concerns.md)) |
| razumljivost grešaka | poruka navodi odjeljak i ime unosa te predlaže slična imena vrsta (BR-FCT-03) |
| odvojenost | Workflow Factory ne zna konkretne komponente, pa se one dodaju bez njegove izmjene (BR-FCT-02) |
| praćenje | Workflow Factory bilježi sastavljanje i greške konfiguracije |

## 5. Verifikacija

### 5.1 Kriteriji provjere

Ovaj zapis nema vlastite kriterije provjere. Poslovna pravila provjeravaju se kriterijima i testovima u zahtjevima sustava (SyRS).

### 5.2 Sljedivost prema zahtjevima sustava (SyRS)

Matrica se popunjava kad se odrede djeca (§6).

## 6. Otvoreno

1. Djeca ovog zapisa (SyRS) nisu određena. Oznake `BR-FAC-*` i naziv sloja u naslovima dijagrama („Data flow", kao u [`StRS-WORKFLOW`](StRS-WORKFLOW.md)) su prijedlog.
2. Odlomak *Problem* u §2.1 je prijedlog autora zapisa i treba ga potvrditi.
3. **Tko spaja drivere.** Workflow Factory drivere stvara i predaje komponentama, ali ih ne spaja, a generički driver se pri čitanju ili pisanju ne spaja sam ([`BR-DRV-04`](StRS-DRIVER.md#3-poslovna-pravila)). Nije određeno tko drivere spaja prije rada toka.
4. Pipelineove procesoru registrira Workflow Factory (BR-FCT-10); [`StRS-PROCESSOR`](StRS-PROCESSOR.md) i [`StRS-PIPELINE`](StRS-PIPELINE.md) su usklađeni. Ostaje odlučiti je li registracija pipelineova ondje zaseban korisnički slučaj.
5. **Autor konfiguracije.** Uloga je izvedena iz načina na koji konfiguracija služi (UC3, UC4); treba potvrditi da je to zasebna uloga i tko je.
6. **Pokretač.** Samo raspoređivač je potvrđen kao pozivatelj Workflow Factoryja; ostale pokretače treba odrediti.
7. **Čitanje konfiguracije.** Čitač konfiguracije nema zapis; treba odlučiti treba li ga zasebni StRS.
8. **Postavke okruženja.** Konfiguracija pri sastavljanju mijenja okruženje procesa (putanje vanjskih alata, vremenska zona). Treba potvrditi da je to željeno za svaki tok, posebno kad se u istom procesu sastavlja više tokova.
9. **Neuspjeh usred sastavljanja.** Dotad stvorene komponente (npr. otvorene konekcije) ne vraćaju se nikome. Treba odrediti tko ih i kada oslobađa.
10. Pozivatelj mora znati unaprijed koji dio konfiguracije sadrži tok; nije određeno tko to određuje.
11. Kakvoće iz §4 treba povezati s postojećim NFRQ zapisima ili otvoriti nove.

## 7. Reference

| oznaka | izvor |
|---|---|
| ISO/IEC/IEEE 29148:2018 | *Systems and software engineering — Life cycle processes — Requirements engineering* |
| [`StRS-WORKFLOW`](StRS-WORKFLOW.md) | tok koji Workflow Factory sastavlja |
| [`StRS-MANAGER`](StRS-MANAGER.md) | upravitelji koje Workflow Factory puni |
| [`StRS-PROCESSOR`](StRS-PROCESSOR.md) | procesor |
| [`StRS-BLACKBOARD`](StRS-BLACKBOARD.md) | Blackboard |
| [`StRS-REPOSITORY`](StRS-REPOSITORY.md) | repozitorij |
| [`StRS-PIPELINE`](StRS-PIPELINE.md) | pipeline |
| [`StRS-STRATEGY`](StRS-STRATEGY.md) | strategije |
| [`StRS-DRIVER`](StRS-DRIVER.md) | driver |
| [`StRS-CONNECTION`](StRS-CONNECTION.md) | konekcija |
| [`StRS-MEMENTO-STORE`](StRS-MEMENTO-STORE.md) | pohrana snimki |
| `concrete/factory.py` | modul, predmet zapisa |

## Povijest promjena

| Verzija | Datum | Promjena |
|---|---|---|
| v0.0.6 | 2026-10-11 | Naziv uloge *pokretač* u dijagramima: *Runner* umjesto *Initiator*. |
| v0.0.5 | 2026-10-11 | Odluka 2026-10-11 (razdvajanje odgovornosti, NFRQ-ORG-15): BR-FCT-17 razlikuje grešku koju otkrije Workflow Factory (s odjeljkom, imenom unosa i uzrokom) od greške komponente (prosljeđuje se kakva jest); time je usklađen s UC2 A6 i tablicom grešaka. Kakvoća razdvajanja odgovornosti u §4. |
| v0.0.4 | 2026-10-11 | Poveznice na StRS-MEMENTO-STORE. |
| v0.0.3 | 2026-10-11 | Povezani zapisi (StRS-DRIVER, StRS-MANAGER, StRS-REPOSITORY, StRS-PROCESSOR, StRS-PIPELINE, StRS-BLACKBOARD) usklađeni s ovim kodom; stavka o ispravcima uklonjena iz *Otvoreno*. |
| v0.0.2 | 2026-10-11 | Nazivlje usklađeno s ostalim zapisima: posvuda *Workflow Factory* (umjesto *tvornica workflowa*); zastarjela stavka o nazivlju uklonjena. Prema kodu: driver komponentama dohvaća Workflow Factory od upravitelja drivera (BR-FCT-08, UC2, kontekst); Blackboard je obvezan za svaki procesor (BR-FCT-10, A3); Blackboard prima i ključeve uz unos (BR-FCT-12); izlaz monitora dobiva driver od upravitelja (UC4); Workflow Factory drivere ne spaja (§2.6). *Otvoreno*: oznaka zapisa više nije prijedlog; dodana pitanja o tome tko spaja drivere i o ispravcima StRS-DRIVER, StRS-MANAGER i StRS-REPOSITORY. |
| v0.0.1 | 2026-10-11 | Prvi zapis, izveden iz koda `concrete/factory.py`. |