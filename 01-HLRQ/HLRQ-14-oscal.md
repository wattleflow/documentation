# HLRQ-14 — Sposobnost strojno provjerljive usklađenosti (OSCAL)

> **Razred `HLRQ` nije u vokabularu** (`CLAUDE.md` §3.6 — „u uporabi, ali nije u registru");
> uvođenje traži DR (D-12). Kategorija `OSCAL` **jest**, od
> [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md).

| | |
|---|---|
| **Status** | Djelomično provedeno (2026-08-22) — mehanizam radi, `BR-OSCAL-11` **proveden** kroz tri komponentne baze ([`FRQ-OSCAL-14.13`](../02-FRQ/FRQ-OSCAL-14.13-component-bases.md)); vrata su `strict=False` i **inertna** dok se ne odluči tko predaje politiku (§7 t.2) |
| **Odluka** | [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md) — kategorija i broj sposobnosti |
| **Razred** | Zahtjev visoke razine — nosi narativ i poslovna pravila; ne opisuje korake |
| **Distribucija** | `blackwattle` — OSCAL nije zaseban paket (provjereno 2026-08-22); uvozno zatvorenje sloja je `stdlib ∪ wattleflow`, bez third-party ovisnosti |
| **Djeca** | trinaest zapisa `FRQ-OSCAL-14.1…14.13` — §4 |
| **Podloga** | [pregled zapisa](../blackwattle/OSCAL.md) · `CLAUDE.md` §6.1 (mjesto provedbe) · `HLRQ-13` §6 (prvi potrošač) |
| **Sljedivost** | `NFRQ-SEC-01`…`NFRQ-SEC-06` (§6) · `NFRQ-ORG-04` (ne uvodi se primitiv) · `NFRQ-ORG-05`, `NFRQ-ORG-08` |

## 1. Narativ

Framework mora imati komponente, usklađene sa sigurnosnim kontrolama. Kaako bi dokumentacija 
za izradu frameworka bila **provjerljiva**, framework mora imati funkcionalnost evidencije komponenti 
te kako zadovoljavaju koju OSCAL kontrolu, kao i što se dogadja prestankom.
Bududci da je OSCAL je NIST-ov jezik u kojem je ta tvrdnja **strojno čitljiva**, 
koristimo ga u svrhu provjerljivosti, umjesto vjerovanja u kod.

**Zašto.** Komponenta koja pristupa vanjskom sustavu (baza, red poruka, model, SFTP) dodiruje
kontrole pristupa, kredencija, prijenosa i kriptografije. Ako svaka komponenta sama za sebe
tvrdi što provodi, tvrdnje se ne mogu ni zbrojiti ni opovrgnuti. Sposobnost koju ovaj zahtjev
pokriva daje **jedan baseline** (ASD ISM Essential Eight), **jedan oblik deklaracije**
(`OSCAL_CONTROLS`) i **jedna vrata** (`declared ⊆ baseline`) — pa je usklađenost svojstvo
koje se moze kvantizirati.

**Mehanizam**: distribucija je objavljena, modeli, učitavanje, razrješavanje profila,
prijevod taksonomije i gate su izvedeni i pokriveni zapisima `FRQ-OSCAL-14.1…14.13`. 
Isporučeni ASD ISM katalog nosi **1130 kontrola u 564 grupe**; tri E8 baselinea biraju 
**46 / 87 / 123** kontrole.

**Provedba**: `wattleflow` framework kompoenente, poput `blackwattle` ima 
**module s deklarinim** `OSCAL_CONTROLS` (to su konekcije, driveri i procesori).
Svaka komponenta mora **primjenjivati dekorater** — trojka  
(`connections/`, `drivers/`, `processors/`). 

<!-- Preostalih 11 nosi deklaraciju koju **ništa ne provjeravaju**: dio ih to i piše u komentaru, 
pozivajući se na tvrdnju da OSCAL paket nije deployan — tvrdnju koja je od 2026-08-21 ispravljena kao netočna (`CLAUDE.md` §6.1). 
Deklaracija bez provedbe je aspiracija, ne kontrola (D-05). -->

## 2. Mjesto u dekompoziciji

Usklađenost cjevovoda su **ograničenje nad komponentama koje ga čine**. 
Ova se sposobnost dijeli na četiri sloja, svaki s vlastitim zapisom:

| sloj | briga | predmet | zapisi |
|---|---|---|---|
| **Model** | vjeran prikaz OSCAL dokumenta | `Catalog`, `Control`, `Group`, `Profile`, vrijednosni objekti, baze | 14.1–14.6 |
| **Ulaz** | granica prema disku i pakiranju | loaderi, `resources/`, javna površina | 14.7, 14.12 |
| **Izbor** | od baselinea do popisa kontrola | registar, resolver, crosswalk | 14.8, 14.9, 14.10 |
| **Provedba** | vrata nad komponentom | `OSCALPolicy` + dekorateri | 14.11 |

Granica je namjerna: **model ne provodi politiku, a politika ne parsira dokumente**. Dekorateri
(`@oscal_connection`, `@oscal_driver`, `@oscal_processor`) žive u `wattleflow-workflow`, jer
provedba pripada sloju koji komponente i gradi.

## 3. Što ova sposobnost nije

| nije | zašto |
|---|---|
| ocjena sigurnosti | gate kaže govori li komponenta o kontrolama u opsegu, ne je li sigurna (`FRQ-OSCAL-14.11` §1) |
| skalarna mjera usklađenosti | izlaz je skup prekršaja, ne postotak (D-09) |
| izvor kontrola | ISM katalog je **vendoriran artefakt** ASD-a; ovaj paket ga isporučuje, ne piše |
| novi domenski primitiv | `Catalog`, `Profile`, `Control` nisu framework primitivi (`NFRQ-ORG-04`, `DR-WFL-013` t.4) |

## 4. Opseg

| oznaka | predmet | dokument |
|---|---|---|
| `FRQ-OSCAL-14.1` | `Catalog` — agregat kontrola | [14.1](../02-FRQ/FRQ-OSCAL-14.1-catalog.md) ✅ |
| `FRQ-OSCAL-14.2` | `Control` — jedinica po kojoj se sudi | [14.2](../02-FRQ/FRQ-OSCAL-14.2-control.md) ✅ |
| `FRQ-OSCAL-14.3` | `Group` — hijerarhija kataloga | [14.3](../02-FRQ/FRQ-OSCAL-14.3-group.md) ✅ |
| `FRQ-OSCAL-14.4` | `Profile` i selektori — baseline | [14.4](../02-FRQ/FRQ-OSCAL-14.4-profile.md) ✅ |
| `FRQ-OSCAL-14.5` | vrijednosni objekti | [14.5](../02-FRQ/FRQ-OSCAL-14.5-value-objects.md) ✅ |
| `FRQ-OSCAL-14.6` | baze modela i njihovi ugovori | [14.6](../02-FRQ/FRQ-OSCAL-14.6-bases.md) ✅ |
| `FRQ-OSCAL-14.7` | učitavanje — granica prema disku | [14.7](../02-FRQ/FRQ-OSCAL-14.7-loaders.md) ✅ |
| `FRQ-OSCAL-14.8` | registar — ravni indeks kontrola | [14.8](../02-FRQ/FRQ-OSCAL-14.8-registry.md) ✅ |
| `FRQ-OSCAL-14.9` | razrješavanje profila u katalog | [14.9](../02-FRQ/FRQ-OSCAL-14.9-resolver.md) ✅ |
| `FRQ-OSCAL-14.10` | crosswalk — prijevod taksonomije | [14.10](../02-FRQ/FRQ-OSCAL-14.10-crosswalk.md) ⚠ mapiranja bez sign-offa |
| `FRQ-OSCAL-14.11` | gate `declared ⊆ baseline` | [14.11](../02-FRQ/FRQ-OSCAL-14.11-policy.md) ✅ |
| `FRQ-OSCAL-14.12` | javna površina i vendorirani artefakti | [14.12](../02-FRQ/FRQ-OSCAL-14.12-package-surface.md) ⚠ verzija ima tri izvora |
| `FRQ-OSCAL-14.13` | komponentne baze pod vratima (`blackwattle`) | [14.13](../02-FRQ/FRQ-OSCAL-14.13-component-bases.md) ✅ |

**Izvan opsega, kandidati:** `component-definition` (deklaracija komponente kao OSCAL dokument, a
ne kao ClassVar), `assessment-results` (rezultat provjere kao OSCAL dokument), katalozi izvan ASD
ISM-a (NIST SP 800-53, ISO 27001, CIS) — vidi §7 t.5.

## 5. Poslovna pravila

Vrijede za cijelu sposobnost; djeca ih nasljeđuju i pozivaju se na oznaku.

| oznaka | pravilo |
|---|---|
| **BR-OSCAL-01** | Baseline putuje **s paketom**, ne s okolinom: katalog i profili su vendorirani artefakti, pa provjera ne ovisi o mreži ni o konfiguraciji. |
| **BR-OSCAL-02** | Komponenta deklarira kontrole koje zadovoljava **u vlastitoj klasi** (`OSCAL_CONTROLS`), ne u konfiguraciji — deklaracija se ne smije mijenjati izvana. |
| **BR-OSCAL-03** | Vrata propuštaju komponentu ako i samo ako je **svaka** deklarirana kontrola u aktivnom baselineu (`declared ⊆ baseline`). |
| **BR-OSCAL-04** | Deklaracija **ne smije tvrditi kontrolu koju klasa ne provodi**; suvišna deklaracija je netočna izjava, ne opreznost. |
| **BR-OSCAL-05** | Komponenta smije deklarirati u stranoj taksonomiji; prijevod je **kurirani artefakt**, a nemapirani `id` **pada na vratima**, nikad ne nestaje. |
| **BR-OSCAL-06** | Prekršaj vrata **zaustavlja uporabu komponente** i imenuje troje: komponentu, baseline i sporne `id`-eve. |
| **BR-OSCAL-07** | Katalog i profil su **nepromjenjivi** nakon učitavanja; razriješeni baseline je novi artefakt, a izvorni ostaje netaknut. |
| **BR-OSCAL-08** | Razrješavanje **ne izmišlja**: `id` koji profil traži a izvorni katalog nema je kvar, ne prazan rezultat (zadano `strict`). |
| **BR-OSCAL-09** | Prijevod bez compliance sign-offa **nije dokaz usklađenosti** — mehanizam smije raditi, tvrdnja ne smije stajati. |
| **BR-OSCAL-10** | Kontrola koju nijedna komponenta ne deklarira **nije pokrivena**; pokrivenost je pitanje baselinea, ne komponente. |
| **BR-OSCAL-11** | Svaka komponenta koja deklarira `OSCAL_CONTROLS` **mora nositi dekorater** svoje uloge (`@oscal_connection`, `@oscal_driver`, `@oscal_processor`) — **osim ako joj je predak već dekoriran**: vrata se nasljeđuju, a dvostruko omatanje je kvar (§7 t.2). Deklaracija bez dekoratera je tvrdnja koju ništa ne provjerava. |
| **BR-OSCAL-12** | Framework vodi **evidenciju komponenti i kontrola** koje zadovoljavaju, i ima definiran ishod **prestanka**: kad komponenta prestane zadovoljavati kontrolu, to je vidljivo, ne tiho. |

`BR-OSCAL-04` je preuzet iz `HLRQ-13` §6 t.2 i podignut na razinu sposobnosti; `BR-OSCAL-09`
formalizira status crosswalk artefakta; `BR-OSCAL-10` imenuje pokrivenost, koju danas ništa ne
mjeri (§7 t.4).

> **`BR-OSCAL-11` i `BR-OSCAL-12` nemaju izvedbu** (2026-08-21). `BR-OSCAL-11` se danas ne može
> provesti — primjena dekoratera na komponentu koja deklarira kontrole **ruši je** pri prvom
> prijelazu FSM-a, jer joj nitko ne predaje aktivnu politiku (§7 t.2; dokaz u §8).
> `BR-OSCAL-12` nema nositelja: evidencija komponenta→kontrola ne postoji ni kao struktura ni
> kao artefakt (§7 t.4). Obveze su zapisane, izvedba je dug (D-05).

## 6. Nefunkcionalni zahtjevi

| NFR | što nalaže | posljedica za ovu sposobnost |
|---|---|---|
| `NFRQ-SEC-01` blast radius | ograniči dosežljivost iz kompromitirane komponente | jedan policy po baselineu; prekršaj imenuje `uuid` profila, pa se opseg vidi. Registar je u memoriji i po procesu — nema dijeljene točke kvara |
| `NFRQ-SEC-02` napadna površina | javno sučelje minimalno, `__all__` eksplicitan | 23 imena u paketnom `__all__`; loaderi su jedina točka koja otvara datoteku; baze modela namjerno nisu izvezene |
| `NFRQ-SEC-03` supply-chain i lokalnost | closure ⊆ tier distribucije | uvozno zatvorenje je `stdlib ∪ wattleflow` — **paket smije uvoziti clean core potrošač**; `MANIFEST.in` je kontrolna točka pakiranja |
| `NFRQ-SEC-04` radna točka detekcije | nalaz mora biti radnja, ne istraga | poruka prekršaja nosi komponentu, baseline i sortirani popis `id`-eva |
| `NFRQ-SEC-05` model protivnika | disciplina ulaganja | vrata odbijaju deklaraciju izvan opsega; ne procjenjuju rizik i ne rangiraju prijetnje |
| `NFRQ-SEC-06` povjerljivost zapisa | bez tajni u zapisu | ova sposobnost ne dodiruje kredencijale — deklaracija je popis `id`-eva |
| `NFRQ-ORG-04` sposobnost vs primitiv | cross-cutting sposobnost je helper, ne novi primitiv | ontologija se ne proširuje (§3) |
| `NFRQ-ORG-05` enkapsulacija | samoreferencirajuće metode kroz `cls` | deserijalizacija, obilazak i orezivanje su članovi klasa; u paketu nema nijedne funkcije na razini modula osim javne `resolve` |
| `NFRQ-ORG-08` deduplikacija | pravilo živi na jednom mjestu | mapiranje ključeva, prazan popis i prolaz kroz čvorove postoje jednom (`FRQ-OSCAL-14.6`) |

## 7. Otvoreno

Odluke koje ovaj dokument ne donosi; svaka traži DR (D-03).

1. ~~Nedeklarirana ovisnost.~~ **Otpalo 2026-08-22.** OSCAL sloj je vendiran u
   `blackwattle`, pa ovisnosti o zasebnoj distribuciji nema — ni ovdje ni u jezgri.
   Uvoz ostaje eager — `wattleflow.oscal` je unutar tiera `stdlib ∪ wattleflow`, pa ne krši
   `NFRQ-SEC-03`. **Preostaje operativno:** izdanje workflowa s novom ovisnošću i instalacija u
   zatečenim okruženjima; do tada `wattleflow.connections` se uvozi, ali razrješavanje imena
   (`__getattr__`) puca bez instalirane OSCAL distribucije.
2. **`BR-OSCAL-11` je proveden, `BR-OSCAL-03` još nije.** Vrata nosi tri komponentne baze
   ([`FRQ-OSCAL-14.13`](../02-FRQ/FRQ-OSCAL-14.13-component-bases.md)), a svih 15 komponenti koje
   deklariraju `OSCAL_CONTROLS` (12 konekcija — kafka nosi dvije — 2 drivera, 1 procesor) ih
   nasljeđuje, uz `strict=False`. Posljedica koju treba držati na oku: dok nitko ne predaje
   `oscal_policy=`, provjera se preskače — vrata su postavljena, ali ne provode ništa. Sa `strict=True` iste bi komponente pale pri prvom prijelazu, jer
   **nijedno mjesto u stablima `core`, `workflow`, `processors` i `examples` ne predaje
   `oscal_policy=`** (§8). Preostaje odluka *tko predaje aktivnu politiku*:

   | put | što traži | cijena |
   |---|---|---|
   | tvornica ubrizgava politiku | `WorkflowFactory` predaje `oscal_policy=` svakoj komponenti, iz konfiguracije | dira `wattleflow-workflow` (`CLAUDE.md` §2.5); traži izvor aktivnog profila u konfiguraciji |
   | zadana politika procesa | dekorater poseže za registriranom politikom kad kwarg izostane | nova funkcionalnost u `wattleflow-workflow` i skriveno globalno stanje |

   **Odlučeno 2026-08-21, potvrđeno pregledom koda — dekorater je namjerno proizvoljan** i
   primjenjuje se prema pravilu, na komponente koje OSCAL stvarno koriste; `OpenSearchConnection`
   je bila uzor primjene. Prisila na razini **generičkih baza frameworka** (`concrete/`) odbijena
   je i ostaje odbijena — vidi ograničenje uvoza ispod.

   ~~Nedostatak: propust se ne vidi ničim.~~ **Riješeno** —
   [`FRQ-OSCAL-14.13`](../02-FRQ/FRQ-OSCAL-14.13-component-bases.md): tri baze u `blackwattle`
   (`OSCALConnection`, `OSCALDriver`, `OSCALProcessor`) nose dekorater, a konkretne klase ih
   nasljeđuju. Vrata su time svojstvo hijerarhije, a ne pamćenja; dekorater po klasi ostaje
   dostupan za slučaj izvan te tri uloge. Uz to, dekorater sada provjerava **prije** konstrukcije
   ([`DR-WFL-014`](../04-DR/DR-WFL-014-oscal-gate-before-construction.md)), pa neusklađena
   konekcija više ne stigne otvoriti vezu.

   **Kad se tome vrati — zatvoren je samo put kroz dekorater, ne i sama prisila.** Razlika je
   provjerena izvršavanjem (2026-08-21):

   | put | stanje |
   |---|---|
   | `@oscal_*` dekorater unutar `concrete/` | **zatvoren.** `concrete/driver.py` koji uvozi `wattleflow.decorators.oscal` ruši uvoz paketa: `ImportError: cannot import name 'Wattleflow' from partially initialized module 'wattleflow.concrete'`. Uzrok je smjer ovisnosti — `wattleflow.oscal` uzvodno uvozi `wattleflow.concrete` (`LazyIterator`, `Wattleflow`). Isto vrijedi za `concrete/base.py`. |
   | `GenericConnectionOSCAL` s **ugrađenom** provjerom | **otvoren, bez ijedne ispravke.** Sloj ne uvozi `wattleflow.oscal` uopće: politika stiže kao argument i zove se po ugovoru (`Protocol` s `verify`), a iznimka je workflow-native. Prototip: uvoz OSCAL-a `False`, ciklusa nema. |

   Prototip je potvrdio i **redoslijed**, koji je za konekcije ključan: `GenericConnection.__init__`
   na kraju zove `ensure_created()`, dakle **otvara vezu**. Provjera zato mora prethoditi
   `super().__init__()` — tako neusklađena komponenta padne prije nego išta otvori (trag:
   `['gate(Violating)']`, bez `ensure_created`). Provjera nakon `super()` stigla bi prekasno.

   **Jedna ispravka koda ipak je uvjet, i drugdje je nego što se činilo.**
   `concrete/connection.py:245` — `__del__` na polusagrađenom objektu zove `self.warning()`, a
   `_logger` još ne postoji: `AttributeError: '…' object has no attribute '_logger'`. Zatečeni kvar
   koji se danas rijetko vidi jer konstrukcija rijetko pada; s vratima u konstruktoru **svako
   odbijanje** ispisuje „Exception ignored in `__del__`" uz pravu iznimku. Popravak je uvjet za
   uredan ispis odbijanja, ne za samu prisilu. Izmjena je `DR-WFL` razine (`concrete/`).

   **Ograničenje otkriveno pri izvedbi:** dekorirati i pretka i potomka je kvar, ne redundancija.
   Vanjski omotač popne `oscal_policy` iz kwargsa pa ga unutarnji ne vidi; pod `strict=True`
   unutarnji guard odbija ispravno konfiguriranu komponentu. Zato `ConnectionHuggingFace` vrata
   **nasljeđuje** od `ProxyConnection` (`FRQ-OSCAL-14.11` §11 t.7).
3. **Crosswalk mapiranja su `proposed`** i čekaju compliance sign-off; `sc-8` i `sc-13` prevode se
   u kontrole **izvan** Essential Eight opsega, pa protiv E8 baselinea ispravno padaju. Do
   sign-offa vrijedi `BR-OSCAL-09`.
4. **Pokrivenost i evidencija (`BR-OSCAL-10`, `BR-OSCAL-12`).** Nitko ne pita je li svaka kontrola
   iz baselinea pokrivena barem jednom komponentom, niti postoji evidencija *koja komponenta
   zadovoljava koju kontrolu*. `OSCAL_CONTROLS` je raspršen po klasama i čita se samo u trenutku
   provjere; poslije ne ostaje zapis. **Prestanak** — komponenta koja prestane zadovoljavati
   kontrolu — nema definiran ishod jer nema stanja koje bi se promijenilo. Kandidat je OSCAL
   `component-definition` kao artefakt (t.5), ne još jedan ClassVar.
5. **Smije li specijalizacija deklarirati manje od pretka — odgođeno namjerno** (odluka autora
   2026-08-21: iznimke se ostavljaju za kraj, pristup se prvo provjerava empirijski). Zadano je
   `controls_strategy="merge"`, dakle unija kroz MRO: potomak **ne može suziti** deklaraciju
   (`('ism-0445',)` → pet kontrola). Empirijska provjera koja prethodi odluci je ujedno građa za
   `BR-OSCAL-12`: za svaku od 15 klasa utvrditi **koju kontrolu stvarno provodi**, pa usporediti s
   deklariranim. Danas je 14 od 15 deklaracija **identičan literal** `("ac-3","ia-5","sc-8","sc-13")`
   — dok se to ne provjeri po komponenti, ne zna se je li `merge` problem ili nije.
6. **Katalozi izvan ASD ISM-a** (NIST SP 800-53, ISO 27001, CIS) i OSCAL dokumenti izvan
   catalog/profile (`component-definition`, `assessment-results`) — otvoreno pitanje iz
   `CLAUDE.md`; model ih danas ne pokriva.
7. **Prazna deklaracija prolazi vrata** (`∅ ⊆ baseline`): gate ne razlikuje „nema što deklarirati"
   od „zaboravljeno" (`FRQ-OSCAL-14.11` §11 t.1). Kandidat za proširenje `BR-OSCAL-02`.
8. **Vrijednosni objekti nemaju core pattern** — `CLAUDE.md` §2.5 traži da svaka klasa naslijedi
   odgovarajući pattern, a `wattleflow.core` nema ugovor za nepromjenjiv vrijednosni objekt
   (`FRQ-OSCAL-14.6` §11 t.1). Razina `DR-COR`.
9. **Verzija distribucije ima tri izvora istine** (`_version.py` `0.0.0.1`, `pyproject` `0.0.5`,
   PyPI `0.0.4`) — `FRQ-OSCAL-14.12` §11 t.1.
10. **Razred `HLRQ` nije u vokabularu** — vrijedi za ovaj zapis kao i za `HLRQ-13`
   (`DR-WFL-013` §Otvoreno t.2).

## 8. Svjedočanstvo

Alat: skripte `fr_evidence.py`, `prune_oracle.py`, `harness.py` (radno okruženje), `grep` nad stablom `blackwattle`; kriterij: §8 svakog
zapisa `FRQ-OSCAL-14.*`; platforma: CPython 3.11.15 (Linux/WSL2), `wattleflow/oscal` radno stablo
2026-08-21, ASD ISM izdanje 2026-03-24.

**Ponašanje vrata pod dekoraterom** (`gate_probe.py`; sintetička komponenta s pravim
`TRANSITIONS`/`ConnectionState` iz `concrete/connection.py`, ISM E8 ML1 + NIST→ISM crosswalk):

| slučaj | ishod |
|---|---|
| deklarira kontrole, **bez** `oscal_policy` kwarga | `OSCALPolicyError: … oscal_policy kwarg is required …` — prijelaz se ne dogodi |
| dekorirana, ne deklarira kontrole | prolaz (opt-out grana) |
| NIST deklaracija + politika **bez** crosswalka | pada na `['ac-3','ia-5','sc-13','sc-8']` |
| NIST deklaracija + politika **s** crosswalkom | pada na prevedenim `['ism-0469','ism-1080']` — izvan E8 opsega |
| deklaracija u ISM taksonomiji, unutar baselinea | prolaz |
| komponenta bez `_fsm` | dekorater je **no-op** — nema što čuvati |
| `controls_strategy="merge"` (zadano) | podklasa nasljeđuje roditeljske: `('ism-0445',)` → `('ac-3','ia-5','ism-0445','sc-13','sc-8')` |

Pretraga `oscal_policy=` po stablima `core`, `workflow`, `processors`, `examples`: **nula pojava**.

**Slijepe pjege (D-11).** (a) Skripte nisu u repozitoriju — test framework nije odabran
(`CLAUDE.md` §4); „provjereno" znači *izvršeno 2026-08-21*, ne *ponovljivo iz stabla*. (b) Put
`@oscal_connection` → `OSCAL_CONTROLS` → `verify` izvršen je nad **sintetičkom** komponentom, ne nad
zatečenom konekcijom — te traže third-party drivere kojih u ovom okruženju nema. (c) Isporučeni ISM katalog ne koristi
ugniježđene kontrole, `params` ni `links`; te su grane provjerene samo sintetički.
