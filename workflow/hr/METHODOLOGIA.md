# Wattleflow inženjerska metodologija (WEM)

## Svezak I — Temelji

**Verzija:** Draft v0.3.1  
**Zadnja izmjena:** 2026-07-28  
**Izvorni jezik:** hrvatski  
**Nadređeni dokument:** [PHILOSOPHY.md](PHILOSOPHY.md) — filozofija je kišobran; ova metodologija je iz nje izvedena  
**Pratioci:** [NFR.md](NFR.md), [DOCTRINE.md](DOCTRINE.md), [POSTULATE.md](POSTULATE.md), [dictionary.yaml](dictionary.yaml)

> Ova metodologija nije zbirka praksi. Ona je ponovljiv test kojim se tvrdnje o sustavu provjeravaju u skladu s filozofijom i doktrinom.

## Povijest izmjena

| Datum | Verzija | Izmjena |
|---|---|---|
| 2026-07-28 | v0.3.1 | Usklađenje s PHILOSOPHY.,md v0.4.1: nazivlje dosljedno DR (normativni tekst; povijesni zapisi netaknuti), oznaka otvorene odluke DR-WFL-004 → DR-WFL-004, uveden sloj P-oznaka (§1, §4, §6, §9) po arhitekturi filozofija/metodologija → postulati → literatura, pratioci dopunjeni registrima (POSTULATI, rjecnik). |
| 2026-07-28 | v0.3 | Sinteza i usklađenje s PHILOSOPHY.,md v0.4. Razriješen smjer ADR ↔ FR/NFR (primarni/izvedeni zahtjevi, §7). Evidencija odluka redefinirana kao **funkcija** čija je ADR forma zamjenjiva pod-metoda (§8.1). DQI dobio protokol valjanosti V1–V6 i status konformnog indeksa (§3.1); dodana hipoteza H4-DQI. §6 dopunjen teorijom mjerenja. Dodan wem_lint kao drugi referentni primjer (§3b). Anatomija (§2) dobila redak *Hipoteza*. Akronimsko pravilo suspendirano na WARNING do DR-WFL-004 (§9). §11 ažuriran na stvarno stanje alata (dva izdanja, runtime provjere, verzionirani kriterij, C-snimke). |
| 2026-07-12 | v0.2 | Restrukturirano; ispravljena inverzija kišobrana; doktrina „ponovljivi znanstveni test"; referentni primjer DQI; standardi kao pod-metode; propagirano kritičko razmišljanje. |
| 2026-06-30 | v0.1 | Prva inačica. |

---

> *„Filozofija postavlja pitanje zašto; metodologija je ponovljivi test kojim tražimo
> odgovor."*

---

## Predgovor — položaj u doktrini

Ovaj dokument je **Svezak I** Wattleflow doktrine. Njegov položaj u hijerarhiji je
eksplicitan i podređen (`PHILOSOPHY.,md`, „Doktrina: filozofija kao kišobran"):

- **Filozofija** je najviša instanca — epistemološki diskurs iz kojeg proizlaze ontološka
  perspektiva te policyji, principi i metode.
- **Ova metodologija** je iz filozofije **izvedena** i ne uspostavlja je, nego je
  operacionalizira: implementira policyje (obvezujuća pravila) koji hrane razvoj i
  održavanje frameworka, i pruža ponovljive metode kojima se policyji i principi
  provjeravaju.

Filozofija kaže *zašto* i *što vrijedi kao znanje*; metodologija daje *ponovljivi postupak*
kojim se to dokazuje, mjeri i održava. Niži sloj ne nadjačava viši; svako odstupanje je
izvod iz višeg sloja ili dokumentirana revizija (§8.1).

---

## 1. Doktrina metodologije: ponovljivi znanstveni test

**Metodologija definira ponovljivi princip znanstvenog testa zasnovan na znanstvenim
pitanjima.** Za svako pitanje metodologija je alat za traženje odgovora.

Načela:

1. **Polazi od pitanja.** Svaka metodologija odgovara na jasno formulirano znanstveno
   pitanje — mjeri, izlaže ili razrješava ambiguitet, nejasnoću, otvoreno pitanje ili
   informacijsku prazninu. Bez pitanja nema metodologije, samo procedure.
2. **Ponovljiva je.** Isti ulaz pod istim uvjetima daje isti rezultat; postupak je opisan
   tako da ga drugi (čovjek ili stroj) može reproducirati i osporiti (falsifikacija).
   Reproducibilnost uključuje i **verziju kriterija**: nalaz je reproducibilan samo uz
   trojku (verzija alata, verzija kriterija/registra, verzija platforme).
3. **Opservacijska prije nego intruzivna.** Gdje je moguće, mjerenje ne mijenja predmet
   mjerenja i ne zaustavlja tok — rezultat se pridružuje kao metapodatak i putuje s njim.
   Teorijsko sidro: pokazatelj koji se koristi za upravljanje prestaje mjeriti ono što je
   mjerio (P-10); dijagnostička uporaba preživljava, upravljačka korodira. Zato je zadana
   uporaba svake mjere dijagnostička, a upravljačka (gate) mora biti eksplicitno
   deklarirana i ograničena (v. §3.1, V6).
4. **Integrira postojeće metode kao pod-metode.** Metodologija ne izmišlja toplu vodu:
   priznate standarde preuzima kao specifičnu pod-metodu za testiranje i provjeru
   (Occam/DRY; §8).
5. **Kvantitativno i kvalitativno.** Kvaliteta se provjerava i mjerom (indeks, prag,
   varijanca) i prosudbom (koja su pravila prekršena, u kojoj točki, kakav je neto
   učinak). Oba su valjani ishodi testa — ali nijedan ne prolazi bez dokaza vlastite
   valjanosti (§6.1).
6. **Sljediva je.** Rezultat se veže natrag na pitanje, na standard-pod-metodu i na
   načelo/policy koji ga opravdava (§7); gdje test hrani doktrinarnu hipotezu, veže se i
   na nju (§2).

Ovo je doktrina koju Wattleflow propagira: metodologija je instrument kritičkog
razmišljanja (`PHILOSOPHY.,md`) — sredstvo kojim tvrdnja preživljava opravdanje umjesto da
prolazi na autoritet ili naviku. To vrijedi i za same mjere: **mjera koja nije prošla
provjeru valjanosti je tvrdnja na autoritet formule.**

---

## 2. Anatomija jedne metodologije

Svaka konkretna metodologija opisuje se istim skeletom, radi ponovljivosti i sljedivosti:

| Element | Sadržaj |
|---|---|
| **Znanstveno pitanje** | Što točno tražimo? Koji ambiguitet / prazninu izlažemo? |
| **Hipoteza (ako postoji)** | Koju doktrinarnu hipotezu iz `PHILOSOPHY.,md` test hrani (H1–H4…); što bi je oborilo. |
| **Kontrolne točke / opseg** | Gdje i kada se mjeri; što je jedinica promatranja. |
| **Pod-metode (standardi)** | Koji priznati standardi/metode se integriraju (ISO, PEP, …). |
| **Mjera (kvantitativno)** | Formula, tip skale, dopuštene agregacije, raspon, prag; numerička stabilnost. |
| **Valjanost mjere** | Status po protokolu iz §6.1 (V1–V6): što je dokazano, što je deklarirano, što je aspiracija. |
| **Prosudba (kvalitativno)** | Koja su pravila prekršena; interpretacija; neto učinak. |
| **Ishod i sljedivost** | Rezultat (verdikt/metrika/vektor) + poveznica na pitanje, hipotezu, standard i načelo. |

Nove metodologije dodaju se po ovom obrascu; jezgra frameworka ostaje nepromijenjena
(plug-in pristup).

---

## 3. Referentni primjer A — mjerenje kvalitete podataka (DQI)

Kanonski primjer: **`pipelines/quality/dqi.py`** *(putanju uskladiti s kodom — v0.2 je
navodila i `quality.py`; jedan od zapisa je zastario)*. Ilustrira anatomiju iz §2.

- **Znanstveno pitanje:** kolika je kvaliteta podataka i kakav je neto učinak pipelinea na
  nju? (Prije mjerenja kvaliteta je informacijska praznina.)
- **Hipoteza:** **H4-DQI** — *vektor dimenzija kvalitete predviđa operativne troškove
  kvalitete podataka* (ručne ispravke, incidente, downstream odbijanja). Obara je izostanak
  povezanosti s vanjskim kriterijem na uzorku različitom od kalibracijskog.
- **Kontrolne točke:** `T1` (sirovi ulaz — inherentna kvaliteta izvora), `T2…`
  (međurezultati), `T3` (finalni podaci — kvaliteta proizvoda). Razlika po dimenzijama
  `d(T3) − d(T1)` daje neto učinak; stopa `|T3|/|T1|` zadržavanje zapisa.
- **Pod-metode:** ISO/IEC 25012 (model dimenzija), ISO 8000-8 (mjera kao omjer zapisa koji
  zadovoljavaju pravilo), ISO 8000-61 (procesni model; odvajanje kvalitete izvora od
  kvalitete proizvoda).
- **Mjera:** **primaran rezultat je vektor dimenzija** `(d₁ … d₆)` — potpunost, valjanost,
  dosljednost, točnost, aktualnost, jedinstvenost — svaka u `[0, 1]` kao omjer zapisa
  (omjerna skala po dimenziji). **DQI = Σ wᵢ·dᵢ je deklarirani zbirni indeks**, konforman,
  ne deskriptivan: težine su izbor, ne empirijska činjenica; promijeni težine — promijeni
  se broj. Svaki iskaz DQI-ja nosi deklaraciju težina i verziju konfiguracije. Agregacija
  prosjeka/varijance Welfordovim on-line algoritmom (O(1) memorije, numerički stabilan,
  Besselova korekcija).
- **Valjanost:** status po §3.1; do prolaska V1–V4 DQI se iskazuje kao *konformni indeks s
  deklariranim težinama*, ne kao mjera kvalitete po sebi.
- **Prosudba:** `failed_rules` (koja pravila, gdje), usporedba po kontrolnim točkama,
  interpretacija neto učinka i zadržavanja.
- **Ishod i sljedivost:** rezultat putuje kao metapodatak uz zapis
  (`facade.document.update_metadata`); mjerenje je opservacijsko. Modul je agnostičan na
  standard: promjenom YAML konfiguracije isti pipeline mjeri prema različitim okvirima;
  nove dimenzije su plug-in funkcije.

## 3.1 Protokol valjanosti mjere (V1–V6) — primjena na DQI

Kritičko razmišljanje primijenjeno na vlastitu metriku: valjanost se ne pretpostavlja,
nego testira, fazama:

| Faza | Pitanje | Postupak | Status za DQI |
|---|---|---|---|
| **V1 Reprezentacijski uvjet** | Postoji li empirijska relacija neovisna o formuli? | Za svaku dimenziju definirati uređaj „skup A kvalitetniji od skupa B po dimenziji d" (npr. ekspertna prosudba na parovima uzoraka) i provjeriti da mjera čuva uređaj (homomorfizam) [15][19]. | *otvoreno* |
| **V2 Tip skale i dopuštene operacije** | Što se s brojevima smije raditi? | Dimenzije su omjeri zapisa → omjerna skala po dimenziji; zbroj preko dimenzija nije mjerenje nego indeks [16]. Dopušteno: usporedba po dimenziji, razlika po kontrolnim točkama, vektorska/Pareto usporedba [35]. | *deklarirano* |
| **V3 Sadržajna pokrivenost** | Mjere li pravila dimenziju? | Preslikavanje pravilo→dimenzija (ISO 25012); dimenzija bez ijednog pravila iskazuje se kao *nemjerena*, nikad kao 1.0. | *provedivo odmah* |
| **V4 Prediktivna valjanost** | Znači li broj išta izvan sebe? | Povezanost s vanjskim kriterijem (trošak ispravaka, incidenti, odbijanja) na uzorku različitom od kalibracijskog [19]. Ovo je test hipoteze H4-DQI. | *otvoreno* |
| **V5 Osjetljivost i stabilnost** | Ovisi li zaključak o proizvoljnom? | Perturbacija težina (±); stabilnost po kontrolnim točkama i granulaciji. Gdje god poredak alternativa ovisi o težinama, odluka se vraća na vektor i Pareto analizu [35]. | *provedivo odmah* |
| **V6 Erozija pod upravljanjem** | Što se dogodi kad mjera postane cilj? | DQI indeks je isključivo dijagnostički [22]; quality gate smije stajati samo na binarnim pravilima (`failed_rules`), nikad na indeksu — inače se optimizira broj, ne kvaliteta. | *propisano* |

Isti protokol vrijedi za svaku buduću mjeru; DQI mu je prvi kandidat, wem_lint vektor
drugi (za njega su V2, V3 i V6 zadovoljeni konstrukcijom: binarna konformnost, sljedivost
pravilo→NFR, vektor bez skalara).

# 3b. Referentni primjer B — konformnost jezgre (wem_lint)

Drugi kanonski primjer, komplementaran DQI-ju: kvalitativno-konformna grana (§1 t.5) nad
kodom umjesto podacima.

- **Znanstveno pitanje:** je li sloj sučelja konforman deklariranom NFR skupu?
- **Hipoteze:** hrani H2 (semantička entropija — trend povreda imenovanja) i H3 (valjanost
  instrumenta — verzionirani kriterij).
- **Kontrolne točke:** modul (uvozi, side-effecti), klasa (apstraktnost, stanje), TypeVar
  (vokabular uloga), fasada paketa (`__all__`).
- **Pod-metode:** PEP 8, statička analiza (AST), runtime introspekcija (`inspect`),
  kontrolirani rječnik (`naming_registry.yaml`).
- **Mjera:** vektor nalaza po dimenzijama (IMP/SFX/ABS/STA/TYP/FAC/HDR/EXC) s brojačima po
  (pravilo, strogost); **ukupna ocjena ne postoji** — nominalna konformnost ne dopušta
  ponderirani zbroj [16].
- **Prosudba:** deklarirane iznimke (waiveri, DR reference), trajno vidljive u vektoru;
  njihov nestanak bez odluke je nalaz.
- **Ishod i sljedivost:** zeleni vektor arhivira se kao **C-snimka** s trojkom (verzija
  alata, verzija kriterija, verzija platforme); svaki nalaz sljediv je do NFR-a i do
  odluke koja pravilo opravdava.

---

## 4. Tri komplementarne discipline

Veliki informacijski sustavi ne opisuju se adekvatno samo programskim inženjerstvom.
Wattleflow integrira tri discipline (razrada „Iznad programskog inženjerstva",
`PHILOSOPHY.,md`):

```
                 Računarstvo
                       ▲
                       │
Informacijska ◄────────┼────────► Programsko
   znanost             │           inženjerstvo
                       ▼
                  Wattleflow
```

- **Računarstvo** — algoritmi, složenost, formalni jezici, automati, teorija grafova,
  optimizacija.
- **Programsko inženjerstvo** — arhitektura, životni ciklus, osiguranje kvalitete,
  testiranje, evolucija, obrasci dizajna.
- **Informacijska znanost** — opis same informacije: ontologija, taksonomija, metapodaci,
  provenijencija, semantička interoperabilnost, životni ciklus informacije; teorijski
  temelji u teoriji informacije (P-06), kibernetici (P-05), semiotici (P-08) i
  infološkoj tradiciji (P-07).

DQI (§3) je sjecište sve tri: mjerni algoritam (računarstvo), kontrolne točke u toku
(programsko inženjerstvo), model kvalitete podataka (informacijska znanost). Isto vrijedi
za wem_lint (§3b): AST/introspekcija, konformnost sloja sučelja, kontrolirani rječnik kao
semiotički stabilizator.

---

## 5. Ontologija zahtjeva

Domenska ontologija (Workflow, Processor, Pipeline, Driver, Repository, Blackboard,
Strategy, Connection, Document, Memento) je proizvod filozofskog diskursa i opisuje *od
čega se sustav sastoji*. Uz nju stoji ortogonalna **ontologija zahtjeva** — *što sustav
mora zadovoljiti*. Zahtjevi su slojeviti prema razini apstrakcije (BABOK; ISO/IEC/IEEE
29148).

| Koncept | Sloj | Definicija | Relacija | Standard |
|---|---|---|---|---|
| Poslovni zahtjev | poslovni | cilj ili ishod poduzeća (*zašto*), neovisan o rješenju | `derivedFrom` problem / prilika / direktiva | BABOK, ISO 29148 |
| Poslovno pravilo | ortogonalno | politika ili definicija u vlasništvu biznisa; *definicijsko* (aletičko — ne može se prekršiti) ili *bihevioralno* (deontičko — može se prekršiti, nosi razinu provođenja) | `generates` / `constrains` FR i NFR | OMG SBVR |
| Funkcionalni zahtjev (FR) | rješenje | što sustav radi (ponašanje, sposobnost) | `realises` poslovni zahtjev; `enforces` poslovno pravilo | ISO 29148 |
| Nefunkcionalni zahtjev (NFR) | rješenje | koliko dobro sustav radi (kvaliteta, ograničenje) | `realises` poslovni zahtjev; usidren na model kvalitete; **primaran** (iz ciljeva/načela) ili **izveden** (iz odluke, §7) | ISO/IEC 25010 |
| Ograničenje (Constraint) | (polisemično) | *zahtjevsko* sužava prostor rješenja (tehnologija, regulativa, budžet); *modelsko* je formalni boolean uvjet na element dizajna | zahtjevsko ~ vrsta NFR-a; modelsko `formalises` pravilo ili NFR | ISO 29148 / OMG OCL |

Posljedice: FR i NFR su izvedeni artefakti razine rješenja koji se sljeduju prema gore do
poslovnih zahtjeva; poslovno pravilo je zaseban artefakt — izvor i ograničenje zahtjeva;
definicijsko pravilo čisto se preslikava u modelsko ograničenje (OCL invarijanta),
bihevioralno nosi deontičku modalnost i ostvaruje se kroz enforcement logiku.

---

## 6. Znanstveni temelji

Arhitektonske odluke moraju biti sljedive do priznatog inženjerskog znanja, ne intuicije:

- **Teorija dizajna:** SOLID, GRASP, GoF, Information Hiding (Parnas [1]), Separation of
  Concerns (Dijkstra), Law of Demeter, Stable Dependencies/Abstractions.
- **Inženjerska načela:** KISS, DRY, YAGNI, Principle of Least Astonishment, Open–Closed,
  Liskov Substitution, Occamova britva.
- **Teorija sustava:** Simon (P-12), Conway, Gall, Lehman, Brooks (P-13); kibernetika —
  povratna petlja i regulacija te zakon nužne raznolikosti (P-05); sinteza naspram
  analize (P-03, P-04).
- **Teorija mjerenja:** reprezentacijski uvjet i tipovi skala (P-09) [15][16];
  aksiomatsko vrednovanje metrika [17][18]; valjanost i vanjski kriterij [19]; erozija
  mjere pod upravljanjem (P-10); višekriterijska usporedba umjesto nelegalnog skalara
  [35]. Nosive tvrdnje s podrijetlom vodi registar postulata (documentation/POSTULATE.md).
- **Matematičko rezoniranje:** kad postoji više valjanih rješenja, prednost ima
  najjednostavnije koje zadovoljava zahtjeve; numerička stabilnost i složenost algoritma
  su prvorazredni kriteriji (Welford, §3).

## 6.1 Valjanost mjere kao opći zahtjev

Nijedna mjera ne ulazi u metodologiju bez statusa po protokolu V1–V6 (§3.1). Minimalni
uvjeti za *bilo koju* mjeru: deklariran tip skale i dopuštene agregacije (V2), sadržajna
pokrivenost (V3), propisana dijagnostička/upravljačka granica (V6). V1, V4 i V5 smiju biti
otvoreni, ali tada se mjera iskazuje kao *konformna* (relativna na deklarirani kriterij),
ne kao deskriptivna tvrdnja o svijetu. Ovo pravilo je operacionalizacija doktrine: „osobna
preferencija nije opravdanje" vrijedi i za formule.

---

## 7. Sljedivost

Znanstvena i inženjerska načela te poslovni ciljevi su **ulazi**; iz njih se izvode
**primarni zahtjevi**; odluke se donose *pod* tim zahtjevima i **generiraju izvedene
zahtjeve**; operativne metrike zatvaraju petlju kao empirijski dokaz.

```
Znanstvena teorija + Inženjersko načelo + Poslovni cilj
            (problem / prilika / direktiva)
                      ↓
     Poslovno pravilo (SBVR)  i  PRIMARNI FR/NFR
        (kriterij koji odluke moraju zadovoljiti)
                      ↓
     Arhitektonska ODLUKA  (evidentirana; §8.1)
        │
        ├── ograničena je primarnim zahtjevima (odluka se brani zahtjevom)
        └── generira IZVEDENE NFR-ove (odluka stvara nove obveze)
                      ↓
                Dizajn sustava
                      ↓
                Implementacija
                      ↓
        Testiranje / metodologije (§1–§3b)
                      ↓
            Operativne metrike i C-snimke
                      ↓
   (empirijski dokaz — vraća se u načela; epistemički rast)
```

Razrješenje smjera (ispravak v0.3): v0.2 je postavila „ADR prije FR/NFR" apsolutno, čime
je nastao sukob s §5 (`realises` poslovni zahtjev) i s praksom (NFR-ORG-02 prethodio je
odluci o korijenu i bio joj kriterij). Ispravno je **oboje, razdvojeno**: primarni
zahtjevi prethode odlukama i ograničavaju ih; izvedeni zahtjevi nastaju iz odluka
(zero-trust → NFR-ORG-06; clean-core → NFR-ORG-01). Rečenica iz `PHILOSOPHY.,md`
(„arhitektonske odluke generiraju mjerljive NFR-ove") odnosi se na izvedene.

Sljedivost je po naravi **graf**, ne lanac: čvorovi su načela, ciljevi, pravila, zahtjevi,
odluke, artefakti i mjerenja; bridovi su imenovane relacije (`derivedFrom`, `realises`,
`constrains`, `generates`, `supersedes`). Tekstualni zapisi (DR datoteke, registri) su
serijalizacije čvorova; vizualni prikaz grafa je legitiman — i poželjan — primarni pogled
(v. §8.1).

---

## 8. Integracija standarda kao pod-metoda

Metodologija dopunjuje, ne zamjenjuje standarde: priznati standard preuzima se kao
integrabilna pod-metoda za testiranje i provjeru (§1 t.4). Standard se ne usvaja po
inerciji — usvajanje je epistemička prosudba i bilježi se.

| Standard | Što pokriva | Pod-metoda za |
|---|---|---|
| Python PEP (osob. PEP 8) | stil jezika i konvencije imenovanja | samoopisivost (§9) |
| ISO/IEC/IEEE 42010 | opis arhitekture — gledišta i interesi | arhitektonsku dokumentaciju |
| ISO/IEC 25010 | model kvalitete proizvoda — sidro za NFR | NFR registar (`NFR.md`) |
| ISO/IEC 25012 | model kvalitete podataka | DQI dimenzije (§3) |
| ISO 8000-8 / 8000-61 | mjerenje i procesni model kvalitete podataka | DQI formula i kontrolne točke (§3) |
| ISO/IEC 12207 | procesi životnog ciklusa softvera | proces razvoja |
| ISO/IEC/IEEE 29148 | inženjerstvo zahtjeva | ontologija zahtjeva (§5) |
| OMG SBVR | poslovni rječnik i pravila | ontologija zahtjeva (§5) |
| OMG OCL | formalna ograničenja na razini modela | modelska ograničenja |
| NIST OSCAL | strojno čitljive sigurnosne kontrole | compliance |
| W3C RDF / OWL / PROV-O | ontologija, semantika, provenijencija | informacijski sloj; graf sljedivosti (§7) |
| SPDX / CycloneDX *(budući)* | software bill of materials | supply-chain (NFR-ORG-06) |

## 8.1 Evidencija odluka kao funkcija; DR kao zamjenjiva pod-metoda

Doktrina (filozofija) zahtijeva **funkciju**: svaka značajna odluka mora biti evidentirana
s kontekstom, alternativama, cijenom i svjedočanstvom — provjerljiva i opoziva.
**Format te evidencije nije doktrina nego metoda.** Trenutna pod-metoda je DR
(naslijeđeni naziv: ADR; prošireni predložak: Status / Kontekst / Odluka / Ugovor / Cijena / Svjedočanstvo /
Registar; genealogija [44][45][46]) — zatečena, funkcionalna, i zamjenjiva istom
epistemičkom prosudbom kao svaki standard iz §8.

Kriteriji koje svaki nasljednik mora zadovoljiti (funkcijski, ne formatski):
sljedivost do načela i zahtjeva; zapis razmotrenih alternativa; razdvajanje dokazanog od
aspiracije (Svjedočanstvo); vezanje na strojno provjeriv kriterij (Registar); revizibilna
povijest uključujući povučene odluke. Kandidat-nasljednik s prednostima za vizualni
prikaz: **odluka kao čvor grafa sljedivosti** (§7) s relacijama
omogućuje/sprječava/nadomješta [45], serijaliziran strojno čitljivo (RDF/PROV-O iz §8
pokriva točno ovo); DR datoteka tada postaje generirani pogled na čvor, a ne izvor
istine. Prijelaz, kad za njega bude dokaza, ide kroz dokumentiranu reviziju ove sekcije.

---

## 9. Samoopisivost i imenovanje

Softver se mora sam objašnjavati: uloga komponente čitljiva je iz imena, hijerarhije
nasljeđivanja, mjesta u paketu i ovisnosti. Imena moraju biti **domenski kvalificirana i
uloga-eksplicitna** — gole generičke imenice (`Manager`, `Helper`, `Parser`) su
zabranjene; kvalificirani oblici (`DriverS3UriParser`, `KafkaReadProcessor`) su ispravni.
Imenovanje je semiotička politika (P-08): ime je znak čiji odnos prema ulozi registar
fiksira, a lint provodi — arhitektonsko pitanje, ne stilska preferencija. Provediva
gramatika i kontrolirani rječnik su u **NFR-ORG-02** (`NFR.md`).

> **Otvorena odluka (DR-WFL-004, pending):** pisanje akronima (`CreatePDFDocument` po PEP 8
> vs zatečeno `CreatePdfDocument`). Do odluke: pripadno lint pravilo **suspendirano je na
> WARNING s deklariranim waiverom** — smije upozoravati (upozorenja su empirijska građa za
> odluku), ne smije kažnjavati. Registar je draft i prilagođava se empirijskim dokazima;
> provođenje jedne strane neodlučenog pitanja kršilo bi kaskadu (metoda ne propisuje mimo
> odluke).

---

## 10. AI-potpomognuto inženjerstvo i propagacija kritičkog razmišljanja

Suvremena arhitektura podržava i ljudsko i strojno zaključivanje: generiranje
dokumentacije, analizu arhitekture, otkrivanje ovisnosti, reverzni inženjering,
refaktoriranje, automatizirano testiranje. Semantička dosljednost (§9) koristi i
inženjerima i AI sustavima.

**Kritičko razmišljanje vrijedi jednako za čovjeka i za AI-agenta:** nijedna odluka ne
prolazi na autoritet ili naviku; kod nedoumice agent pita, izlaže trošak i alternative, i
ne pogađa (CLAUDE.md §5). Metodologija je alat te discipline — ponovljiv test kojim se
tvrdnja (ljudska ili strojno generirana) provjerava, a ne pretpostavlja.

---

## 11. Nefunkcionalni zahtjevi i konformnost

Zahtjevi su primarni (iz ciljeva i načela) ili izvedeni (iz odluka); svaki NFR usidren je
na karakteristiku iz **ISO/IEC 25010**, nosi identifikator (`NFR-<KATEGORIJA>-NN`),
kriterije prihvaćanja, metodu verifikacije i sljedivost prema gore (§7). Registar se vodi
u **`NFR.md`** (trenutno `NFR-ORG-01` … `NFR-ORG-06`, uz Dodatak A — Facet Ordering).

Verifikacija konformnosti izvedena je kao **wem_lint** (§3b), u dva izdanja s odvojenim
kriterijima: *core izdanje* (sloj sučelja: statička analiza AST-om + runtime provjere
apstraktnosti introspekcijom; ugrađeni kriterij s JSON nadjačavanjem) i *workflow izdanje*
(registar `naming_registry.yaml`: domene, clean-core opseg, obitelji baza, vokabular
uloga). Kriterij je **verzioniran odvojeno od alata**; nalaz je reproducibilan samo uz
trojku (alat, kriterij, platforma), a zelene konformnosti arhiviraju se kao **C-snimke**
(`docs/conformance/`; C0 = prva zelena konformnost jezgre). Izlaz je vektorski, bez
skalarne ocjene (§6.1). Pokretljiv na zahtjev; kad CI postoji — kontrola kvalitete, uz
ogradu iz §1 t.3: gate stoji na binarnim pravilima, indeksi ostaju dijagnostički.

---

## Završna riječ

Metodologija je most između filozofije i koda: filozofija postavlja pitanje i mjerilo
znanja, metodologija daje ponovljivi test kojim se do odgovora dolazi i kojim se on brani.
Softver ne smije samo raditi — mora biti razumljiv, objašnjiv, održiv, auditabilan i
obrazovan, a svaka tvrdnja o njemu, uključujući svaku mjeru, mora preživjeti provjeru.

---

## Bilješka o sintezi (v0.3 → v0.3.1)

1. **Nazivlje dosljedno DR** u normativnom tekstu (§3b, §7, §8.1, §9,
   zaglavlje); povijesni zapisi (Povijest izmjena, Bilješka v0.2→v0.3, citat
   v0.2 formulacije u §7) namjerno zadržavaju ADR — opisuju stare verzije.
2. **DR-WFL-004** kao oznaka otvorene odluke o akronimima (bila DR-WFL-004), radi
   konzistentnosti s rječnikom i filozofijom.
3. **Sloj P-oznaka**: tvrdnje koje postoje kao postulati citiraju P-NN
   (P-03, P-04, P-05, P-06, P-07, P-08, P-09, P-10, P-12, P-13); izravni
   ključevi [n] ostaju za operativne protokole i katalog (V1–V6 tablica,
   aksiomatski okviri [17][18], vanjski kriterij [19], Pareto [35]).
4. Nadređeni dokument ažuriran na v0.4.1; pratioci dopunjeni s
   POSTULATI.md i rjecnik.yaml.
5. Otvoreno i dalje: putanja DQI modula (dqi.py vs quality.py) — čeka
   provjeru u kodu.

## Bilješka o sintezi (v0.2 → v0.3)

Za analizu autora — intervencije redom težine:

1. **§7 razriješen smjer ADR ↔ FR/NFR**: primarni zahtjevi prethode odlukama i
   ograničavaju ih; izvedeni NFR-ovi nastaju iz odluka. Uklonjen apsolutni „ADR prije
   FR/NFR" iz v0.2 koji je proturječio §5 i praksi.
2. **§8.1 nova**: evidencija odluka definirana kao doktrinarna *funkcija*; ADR
   degradiran u zamjenjivu pod-metodu s funkcijskim kriterijima nasljednika; graf odluka
   (RDF/PROV-O) imenovan kao kandidat — provodi autorovu rezervu prema ADR-u i sklonost
   vizualnom prikazu bez kršenja kaskade.
3. **§3.1 nova — protokol valjanosti V1–V6**; DQI redefiniran: vektor primaran, skalar
   konformni deklarirani indeks; dodana hipoteza H4-DQI (kandidat za PHILOSOPHY.,md
   sekciju Hipoteze). §6.1 poopćuje protokol na sve mjere.
4. **§3b nova — wem_lint kao drugi referentni primjer** (kvalitativno-konformna grana).
5. **§6 dopunjen teorijom mjerenja** [15][16][17][18][19][22][35] i kibernetikom/
   sintezom [30][37][49].
6. **§9 akronimi**: pravilo suspendirano na WARNING do DR-WFL-004; obrazložen sukob
   metoda-vs-odluka.
7. **§2 anatomija**: dodani retci *Hipoteza* i *Valjanost mjere*.
8. **§11 ažuriran** na stvarno stanje alata (dva izdanja, runtime ABS provjere,
   verzionirani kriterij, C-snimke); redak „Zamjenjuje" uklonjen iz zaglavlja (osobni
   podsjetnik autora, ne sadržaj dokumenta).
9. **§1 t.2 i t.3 dopunjeni**: reproducibilnost uključuje verziju kriterija; opservacijsko
   načelo dobilo Campbellovo sidro [22].
10. Referentni ključevi vezani na konsolidiranu tablicu literature (1–52); putanja DQI
    modula označena za provjeru (dqi.py vs quality.py).