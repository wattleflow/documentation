# Wattleflow inženjerska metodologija (WEM)

## Svezak I — Temelji

**Verzija:** Draft v0.3.2  
**Zadnja izmjena:** 2026-08-24  
**Nadređeni dokument:** [PHILOSOPHY.md](PHILOSOPHY.md) — filozofija je kišobran; ova metodologija je iz nje izvedena  
**Pratioci:** [NFR registar](03-NFRQ/0-NFRQ-EN.md), [DOCTRINE.md](DOCTRINE.md), [POSTULATE.md](POSTULATE.md), [dictionary.yaml](dictionary.yaml)

> Ova metodologija nije zbirka praksi. Ona je ponovljiv test kojim se tvrdnje o sustavu provjeravaju u skladu s filozofijom i doktrinom.

## Povijest izmjena

| Datum | Verzija | Izmjena |
|---|---|---|
| 2026-08-24 | v0.3.2 | Dokument preseljen u korijen stabla (bio symlink, pa su relativne poveznice radile samo iz `workflow/hr/`). Ispravljene oznake odluka i putanje registara; uklonjene bilješke o sintezi koje su upućivale na sekcije kojih u dokumentu nema (§3b, §3.1, §6.1, §8.1, §11). |
| 2026-07-28 | v0.3.1 | Usklađenje s `PHILOSOPHY.md` v0.4.1: nazivlje dosljedno `DR`; oznaka otvorene odluke `DR-015` → `DR-WFL-004`; uveden sloj P-oznaka (§1, §4, §6, §9); pratioci dopunjeni registrima. |
| 2026-07-28 | v0.3 | Razriješen smjer DR ↔ FR/NFR (primarni/izvedeni zahtjevi, §6). Evidencija odluka redefinirana kao **funkcija** čija je DR forma zamjenjiva pod-metoda (§7.1). Uveden protokol valjanosti V1–V6 (§5.1) i hipoteza H4-DQI. Anatomija (§2) dobila retke *Hipoteza* i *Valjanost mjere*. Akronimsko pravilo suspendirano na WARNING do `DR-WFL-004` (§8). |
| 2026-07-12 | v0.2 | Restrukturirano; ispravljena inverzija kišobrana; doktrina „ponovljivi znanstveni test"; standardi kao pod-metode. |
| 2026-06-30 | v0.1 | Prva inačica. |

---

> *„Filozofija postavlja pitanje zašto; metodologija je ponovljivi test kojim tražimo odgovor."*

---

## Predgovor — položaj u doktrini

Ovo je Svezak I (Volume I) Wattleflow doktrine. Njegov je položaj namjerno podređen: filozofija (PHILOSOPHY.md) je kišobran, metodologija stoji ispod njega.

Podjela je jednostavna. Filozofija odgovara na pitanje zašto i što uopće vrijedi kao znanje. Metodologija daje ponovljiv postupak (reproducible procedure) kojim se to provjerava, mjeri i održava — pravila kojima se vodi razvoj frameworka i metode kojima se ta pravila provjeravaju.

Uloga joj je ista kao u znanosti: ne dokazuje tvrdnje, nego ih čini provjerljivima i ponovljivima.

Odnos slojeva je jednosmjeran — niži ne nadjačava viši. Odstupanje je stoga ili izvod iz višeg sloja ili dokumentirana revizija (§7.1).

---
## 1. Metodologija kao ponovljivi znanstveni test
Svaka metodologija je alat za traženje odgovora na jasno postavljeno pitanje. Iz toga slijedi šest načela.

1. **Polazi od pitanja** . Metodologija mjeri, izlaže ili razrješava nejasnoću, otvoreno pitanje ili informacijsku prazninu. Bez pitanja ostaje samo procedura.
2. **Ponovljiva je**. Isti ulaz pod istim uvjetima daje isti rezultat, a postupak je opisan tako da ga drugi — čovjek ili stroj — može reproducirati i osporiti (falsifiability). Ponovljivost uključuje i verziju kriterija: nalaz vrijedi samo uz trojku verzija alata + verzija kriterija/registra + verzija platforme. Bez te trojke rezultat se ne može ponoviti, samo prepričati.
3. **Radije promatra nego zadire**. Gdje je moguće, mjerenje ne mijenja predmet mjerenja i ne zaustavlja tok; rezultat se pridružuje kao metapodatak (metadata) i putuje s njim.
Iza toga stoji poznato zapažanje (P-10): kad pokazatelj postane cilj upravljanja, prestaje mjeriti ono što je mjerio. Dijagnostička uporaba to preživljava, upravljačka korodira. Zato je zadana uporaba svake mjere dijagnostička. Upravljačka uporaba (gate) i dalje je dopuštena, ali uz izričitu deklaraciju i jasno ograničenje opsega (§5.1, V6) — nedeklarirani gate tiho pretvara dijagnostiku u metu i time gubi upravo ono što je mjerio.
4. **Ne izmišlja toplu vodu**. Priznati standardi preuzimaju se kao pod-metode za testiranje i provjeru (Occam/DRY; §7).
5. Mjeri i prosuđuje. Kvaliteta se provjerava brojem (indeks, prag, varijanca) i prosudbom (koja su pravila prekršena, u kojoj točki, kakav je neto učinak). Oba su valjani ishodi testa — uz uvjet da svaki priloži dokaz vlastite valjanosti (§5.1).
6. **Ostavlja trag**. Rezultat se veže natrag na pitanje, na standard koji mu služi kao pod-metoda i na načelo koje ga opravdava (§6). Ako test hrani doktrinarnu hipotezu, veže se i na nju (§2).
Sve to izvire iz jedne ideje: metodologija je instrument kritičkog razmišljanja — sredstvo kojim tvrdnja preživljava opravdanje umjesto da prolazi na autoritet ili naviku. Isto vrijedi i za same mjere: mjera koja nije prošla provjeru valjanosti je tvrdnja na autoritet formule.


---

## 2. Anatomija metodologije

Svaka se konkretna metodologija opisuje istim skeletom, radi ponovljivosti i sljedivosti (traceability):

| Element | Sadržaj |
|---|---|
| **Znanstveno pitanje** | Što točno tražimo? Koju nejasnoću ili prazninu izlažemo? |
| **Hipoteza (ako postoji)** | Koju doktrinarnu hipotezu iz `PHILOSOPHY.md` test hrani (H1–H4…) i što bi je oborilo. |
| **Kontrolne točke / opseg** | Gdje i kada se mjeri; što je jedinica promatranja. |
| **Pod-metode (standardi)** | Koji priznati standardi/metode se integriraju (ISO, PEP, …). |
| **Mjera (kvantitativno)** | Formula, tip skale, dopuštene agregacije, raspon, prag; numerička stabilnost. |
| **Valjanost mjere** | Status po protokolu iz §5.1 (V1–V6): što je dokazano, što je deklarirano, što je aspiracija. |
| **Prosudba (kvalitativno)** | Koja su pravila prekršena; interpretacija; neto učinak. |
| **Ishod i sljedivost** | Rezultat (verdikt/metrika/vektor) + poveznica na pitanje, hipotezu, standard i načelo. |

Nove metodologije dodaju se po ovom obrascu; jezgra frameworka ostaje nepromijenjena (plug-in pristup).

---

## 3. Tri komplementarne discipline

Veliki informacijski sustavi ne opisuju se adekvatno samo programskim inženjerstvom.
Wattleflow integrira tri discipline (razrada „Iznad programskog inženjerstva",
`PHILOSOPHY.md`):

```
                 Računarstvo
                       ▲
                       │
Informacijska ◄────────┼────────► Programsko
   znanost             │           inženjerstvo
                       ▼
                  Wattleflow
```

- **Informacijska znanost** — opis same informacije: ontologija, taksonomija, metapodaci,
  provenijencija, semantička interoperabilnost, životni ciklus informacije; teorijski
  temelji u teoriji informacije (P-06), kibernetici (P-05), semiotici (P-08) i
  infološkoj tradiciji (P-07).
- **Programsko inženjerstvo** — arhitektura, životni ciklus, osiguranje kvalitete,
  testiranje, evolucija, obrasci dizajna.
- **Računarstvo** — algoritmi, složenost, formalni jezici, automati, teorija grafova,
  optimizacija.


---

## 4. Ontologija zahtjeva

Domenska ontologija je proizvod filozofskog diskursa i opisuje *od čega se sustav sastoji*. Uz nju stoji ortogonalna **ontologija zahtjeva** — *što sustav mora zadovoljiti*. Zahtjevi su slojeviti prema razini apstrakcije (BABOK; ISO/IEC/IEEE
29148).

| Koncept | Sloj | Definicija | Relacija | Standard |
|---|---|---|---|---|
| Poslovni zahtjev | poslovni | cilj ili ishod poduzeća (*zašto*), neovisan o rješenju | `derivedFrom` problem / prilika / direktiva | BABOK, ISO 29148 |
| Poslovno pravilo | ortogonalno | politika ili definicija u vlasništvu biznisa; *definicijsko* (aletičko — ne može se prekršiti) ili *bihevioralno* (deontičko — može se prekršiti, nosi razinu provođenja) | `generates` / `constrains` FR i NFR | OMG SBVR |
| Funkcionalni zahtjev (FR) | rješenje | što sustav radi (ponašanje, sposobnost) | `realises` poslovni zahtjev; `enforces` poslovno pravilo | ISO 29148 |
| Nefunkcionalni zahtjev (NFR) | rješenje | koliko dobro sustav radi (kvaliteta, ograničenje) | `realises` poslovni zahtjev; usidren na model kvalitete; **primaran** (iz ciljeva/načela) ili **izveden** (iz odluke, §6) | ISO/IEC 25010 |
| Ograničenje (Constraint) | (polisemično) | *zahtjevsko* sužava prostor rješenja (tehnologija, regulativa, budžet); *modelsko* je formalni boolean uvjet na element dizajna | zahtjevsko ~ vrsta NFR-a; modelsko `formalises` pravilo ili NFR | ISO 29148 / OMG OCL |

Posljedice: FR i NFR su izvedeni artefakti razine rješenja koji se sljeduju prema gore do
poslovnih zahtjeva; poslovno pravilo je zaseban artefakt — izvor i ograničenje zahtjeva;
definicijsko pravilo čisto se preslikava u modelsko ograničenje (OCL invarijanta),
bihevioralno nosi deontičku modalnost i ostvaruje se kroz enforcement logiku.

---

## 5. Znanstveni temelji

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

## 5.1 Valjanost mjere kao opći zahtjev

Nijedna mjera ne ulazi u metodologiju bez statusa po protokolu V1–V6. Protokol je
**generički** — status po fazama vodi dokument pripadne metode, ne ovaj tekst.

| Faza | Pitanje | Što faza traži |
|---|---|---|
| **V1 Reprezentacijski uvjet** | Postoji li empirijska relacija neovisna o formuli? | Definiran uređaj „A je bolji od B po dimenziji d" i dokaz da ga mjera čuva (homomorfizam) [15][19]. |
| **V2 Tip skale i dopuštene operacije** | Što se s brojevima smije raditi? | Deklariran tip skale i dopuštene agregacije; zbroj preko nesumjerljivih dimenzija je indeks, ne mjerenje [16][35]. |
| **V3 Sadržajna pokrivenost** | Mjere li pravila ono što tvrde? | Preslikavanje pravilo → dimenzija; dimenzija bez pravila iskazuje se kao *nemjerena*, nikad kao savršena. |
| **V4 Prediktivna valjanost** | Znači li broj išta izvan sebe? | Povezanost s vanjskim kriterijem na uzorku različitom od kalibracijskog [19]. |
| **V5 Osjetljivost i stabilnost** | Ovisi li zaključak o proizvoljnom? | Perturbacija parametara; gdje poredak ovisi o njima, odluka se vraća na vektor i Pareto analizu [35]. |
| **V6 Erozija pod upravljanjem** | Što kad mjera postane cilj? | Deklarirana granica: indeks je dijagnostički, gate stoji na binarnim pravilima [22]. |

Minimalni uvjeti za *bilo koju* mjeru: deklariran tip skale i dopuštene agregacije (V2),
sadržajna pokrivenost (V3), propisana dijagnostička/upravljačka granica (V6). V1, V4 i V5
smiju biti otvoreni, ali tada se mjera iskazuje kao *konformna* (relativna na deklarirani
kriterij), ne kao deskriptivna tvrdnja o svijetu. Ovo pravilo je operacionalizacija
doktrine: „osobna preferencija nije opravdanje" vrijedi i za formule.

---

## 6. Sljedivost

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
     Arhitektonska ODLUKA  (evidentirana; §7.1)
        │
        ├── ograničena je primarnim zahtjevima (odluka se brani zahtjevom)
        └── generira IZVEDENE NFR-ove (odluka stvara nove obveze)
                      ↓
                Dizajn sustava
                      ↓
                Implementacija
                      ↓
        Testiranje / metodologije (§1–§3)
                      ↓
            Operativne metrike i C-snimke
                      ↓
   (empirijski dokaz — vraća se u načela; epistemički rast)
```

Razrješenje smjera (ispravak v0.3): v0.2 je postavila „DR prije FR/NFR" apsolutno, čime
je nastao sukob s §4 (`realises` poslovni zahtjev) i s praksom (NFR-ORG-02 prethodio je
odluci o korijenu i bio joj kriterij). Ispravno je **oboje, razdvojeno**: primarni
zahtjevi prethode odlukama i ograničavaju ih; izvedeni zahtjevi nastaju iz odluka
(zero-trust → `NFR-SEC-01/02/03`; clean-core → `NFR-ORG-01`). Rečenica iz `PHILOSOPHY.md`
(„arhitektonske odluke generiraju mjerljive NFR-ove") odnosi se na izvedene.

Sljedivost je po naravi **graf**, ne lanac: čvorovi su načela, ciljevi, pravila, zahtjevi,
odluke, artefakti i mjerenja; bridovi su imenovane relacije (`derivedFrom`, `realises`,
`constrains`, `generates`, `supersedes`). Tekstualni zapisi (DR datoteke, registri) su
serijalizacije čvorova; vizualni prikaz grafa je legitiman — i poželjan — primarni pogled
(v. §7.1).

---

## 7. Integracija standarda kao pod-metoda

Metodologija dopunjuje, ne zamjenjuje standarde: priznati standard preuzima se kao
integrabilna pod-metoda za testiranje i provjeru (§1 t.4). Standard se ne usvaja po
inerciji — usvajanje je epistemička prosudba i bilježi se.

| Standard | Što pokriva | Pod-metoda za |
|---|---|---|
| Python PEP (osob. PEP 8) | stil jezika i konvencije imenovanja | samoopisivost (§8) |
| ISO/IEC/IEEE 42010 | opis arhitekture — gledišta i interesi | arhitektonsku dokumentaciju |
| ISO/IEC 25010 | model kvalitete proizvoda — sidro za NFR | NFR registar ([`03-NFRQ/`](03-NFRQ/0-NFRQ-EN.md)) |
| ISO/IEC 25012 | model kvalitete podataka | DQI dimenzije ([`05-METHOD/dqi.md`](05-METHOD/dqi.md)) |
| ISO 8000-8 / 8000-61 | mjerenje i procesni model kvalitete podataka | DQI formula i kontrolne točke (isto) |
| ISO/IEC 12207 | procesi životnog ciklusa softvera | proces razvoja |
| ISO/IEC/IEEE 29148 | inženjerstvo zahtjeva | ontologija zahtjeva (§4) |
| OMG SBVR | poslovni rječnik i pravila | ontologija zahtjeva (§4) |
| OMG OCL | formalna ograničenja na razini modela | modelska ograničenja |
| NIST OSCAL | strojno čitljive sigurnosne kontrole | compliance |
| W3C RDF / OWL / PROV-O | ontologija, semantika, provenijencija | informacijski sloj; graf sljedivosti (§6) |
| SPDX / CycloneDX *(budući)* | software bill of materials | supply-chain (`NFR-SEC-03`) |

## 7.1 Evidencija odluka kao funkcija; DR kao zamjenjiva pod-metoda

Doktrina (filozofija) zahtijeva **funkciju**: svaka značajna odluka mora biti evidentirana
s kontekstom, alternativama, cijenom i svjedočanstvom — provjerljiva i opoziva.
**Format te evidencije nije doktrina nego metoda.** Trenutna pod-metoda je DR
(naslijeđeni naziv: ADR; prošireni predložak: Status / Kontekst / Odluka / Ugovor / Cijena / Svjedočanstvo /
Registar; genealogija [44][45][46]) — zatečena, funkcionalna, i zamjenjiva istom
epistemičkom prosudbom kao svaki standard iz §7.

Kriteriji koje svaki nasljednik mora zadovoljiti (funkcijski, ne formatski):
sljedivost do načela i zahtjeva; zapis razmotrenih alternativa; razdvajanje dokazanog od
aspiracije (Svjedočanstvo); vezanje na strojno provjeriv kriterij (Registar); revizibilna
povijest uključujući povučene odluke. Kandidat-nasljednik s prednostima za vizualni
prikaz: **odluka kao čvor grafa sljedivosti** (§6) s relacijama
omogućuje/sprječava/nadomješta [45], serijaliziran strojno čitljivo (RDF/PROV-O iz §7
pokriva točno ovo); DR datoteka tada postaje generirani pogled na čvor, a ne izvor
istine. Prijelaz, kad za njega bude dokaza, ide kroz dokumentiranu reviziju ove sekcije.

---

## 8. Samoopisivost i imenovanje

Softver se mora sam objašnjavati: uloga komponente čitljiva je iz imena, hijerarhije
nasljeđivanja, mjesta u paketu i ovisnosti. Imena moraju biti **domenski kvalificirana i
uloga-eksplicitna** — gole generičke imenice (`Manager`, `Helper`, `Parser`) su
zabranjene; kvalificirani oblici (`DriverS3UriParser`, `KafkaReadProcessor`) su ispravni.
Imenovanje je semiotička politika (P-08): ime je znak čiji odnos prema ulozi registar
fiksira, a lint provodi — arhitektonsko pitanje, ne stilska preferencija. Provediva
gramatika i kontrolirani rječnik su u **[`NFR-ORG-02`](03-NFRQ/NFR-ORG-02-class-nomenclature-EN.md)**.

> **Otvorena odluka (DR-WFL-004, pending):** pisanje akronima (`CreatePDFDocument` po PEP 8
> vs zatečeno `CreatePdfDocument`). Do odluke: pripadno lint pravilo **suspendirano je na
> WARNING s deklariranim waiverom** — smije upozoravati (upozorenja su empirijska građa za
> odluku), ne smije kažnjavati. Registar je draft i prilagođava se empirijskim dokazima;
> provođenje jedne strane neodlučenog pitanja kršilo bi kaskadu (metoda ne propisuje mimo
> odluke).

---

## 9. AI-potpomognuto inženjerstvo i propagacija kritičkog razmišljanja

Suvremena arhitektura podržava i ljudsko i strojno zaključivanje: generiranje
dokumentacije, analizu arhitekture, otkrivanje ovisnosti, reverzni inženjering,
refaktoriranje, automatizirano testiranje. Semantička dosljednost (§8) koristi i
inženjerima i AI sustavima.

**Kritičko razmišljanje vrijedi jednako za čovjeka i za AI-agenta:** nijedna odluka ne
prolazi na autoritet ili naviku; kod nedoumice agent pita, izlaže trošak i alternative, i
ne pogađa (`CLAUDE.md` §5). Metodologija je alat te discipline — ponovljiv test kojim se
tvrdnja (ljudska ili strojno generirana) provjerava, a ne pretpostavlja.

---

## 10. Nefunkcionalni zahtjevi i konformnost

Zahtjevi su primarni (iz ciljeva i načela) ili izvedeni (iz odluka); svaki NFR usidren je
na karakteristiku iz **ISO/IEC 25010**, nosi identifikator (`NFR-<KATEGORIJA>-NN`),
kriterije prihvaćanja, metodu verifikacije i sljedivost prema gore (§6). Registar se vodi
u **[`03-NFRQ/`](03-NFRQ/0-NFRQ-EN.md)** — zapis po zahtjevu, uz zajedničke definicije
(`NFR-DEF-01`), povelju `[M]` (`NFR-DEF-02`) i Dodatak A o redoslijedu faceta (`NFR-APX-01`).
Opseg kategorija vodi indeks registra, ne ovaj tekst (D-13).

Verifikacija konformnosti izvedena je kao **statička provjera nad izvornim kodom**, po istoj
anatomiji (§2): kriterij je **verzioniran odvojeno od alata**, nalaz je reproducibilan samo
uz trojku (alat, kriterij, platforma), a zelena konformnost arhivira se kao **C-snimka**.
Izlaz je vektorski, bez skalarne ocjene (§5.1). Pokretljiva na zahtjev; kad CI postoji —
kontrola kvalitete, uz ogradu iz §1 t.3: gate stoji na binarnim pravilima, indeksi ostaju
dijagnostički.

Alat, njegovi prekidači, opseg i ključevi registra opisani su u dokumentu alata
(`tools/README.md`); obveze koje verdikt nameće u `POLICY.md` §9. Ovdje se ne prepričavaju.

---

## Završna riječ

Metodologija je most između filozofije i koda: filozofija postavlja pitanje i mjerilo
znanja, metodologija daje ponovljivi test kojim se do odgovora dolazi i kojim se on brani.
Softver ne smije samo raditi — mora biti razumljiv, objašnjiv, održiv, auditabilan i
obrazovan, a svaka tvrdnja o njemu, uključujući svaku mjeru, mora preživjeti provjeru.

---

## Bilješka o sintezi

Trag racionalizacije (P-14) vodi tablica *Povijest izmjena*; pojedinačne intervencije nose
git i `DR-WFL` serija. Ranije bilješke o sintezi (v0.2 → v0.3 → v0.3.1) uklonjene su
2026-08-24 jer su upućivale na numeraciju sekcija koju ovaj dokument više ne nosi — prikaz
koji se razišao s izvorom je nalaz, ne povijest (D-13).
