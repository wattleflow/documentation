# Wattleflow inženjerska metodologija (WEM)

## Svezak I — Temelji

**Verzija:** Draft v0.4 — *prijedlog*; dokumentirana odluka o izmjeni nije donesena (D-03)
**Zadnja izmjena:** 2026-10-01
**Nadređeni dokument:** [PHILOSOPHY.md](PHILOSOPHY.md) — filozofija je kišobran, metodologija je iz nje izvedena
**Pratioci:** [NFR registar](requirements/03-NFRQ/NFRQ-000-INDEX.md), [DOCTRINE.md](DOCTRINE.md), [POSTULATE.md](POSTULATE.md), [dictionary.yaml](dictionary.yaml)
**Svesci:** zapisan je samo Svezak I; ostali nisu zapisani (§11 t.1)

> Metodologija nije zbirka praksi. Ona je ponovljiv test kojim se tvrdnje o sustavu provjeravaju u skladu s filozofijom i doktrinom.

## Povijest izmjena

| Datum | Verzija | Izmjena |
|---|---|---|
| 2026-10-01 | v0.4 | Preoblikovano po nalazima revizije 2026-10-01 (nezapisana u `06-ANALYSIS/`). Tekst sažet. §4: dodani zahtjev dionika, prijelazni zahtjev i vrsta poslovnog pravila (*prijedlog*). §6: dodan ulazni sloj strateške analize (*prijedlog*); narativ o ispravku v0.3 premješten u ovu tablicu. §7: dodan BABOK. §8: uloga razdvojena od imena klase. §10: oznaka `NFRQ-<KAT>-<NN>`. §5: uklonjeno krivo upućivanje „Welford, §3"; putanja `documentation/POSTULATE.md` → `POSTULATE.md`. §5.1 i §7.1 spušteni na treću razinu. Dodan §11 Otvoreno. |
| 2026-08-24 | v0.3.2 | Dokument preseljen u korijen stabla (bio symlink, pa su relativne poveznice radile samo iz `workflow/hr/`). Ispravljene oznake odluka i putanje registara; uklonjene bilješke o sintezi koje su upućivale na sekcije kojih u dokumentu nema (§3b, §3.1, §6.1, §8.1, §11). |
| 2026-07-28 | v0.3.1 | Usklađenje s `PHILOSOPHY.md` v0.4.1: nazivlje dosljedno; oznaka otvorene odluke →  uveden sloj P-oznaka (§1, §4, §6, §9); pratioci dopunjeni registrima. |
| 2026-07-28 | v0.3 | Razriješen smjer odluka ↔ FR/NFR: v0.2 je postavila „odluka prije FR/NFR" apsolutno, u sukobu s §4 (`realises`) i praksom (`NFRQ-ORG-02` prethodio je odluci o korijenu i bio joj kriterij); ispravljeno na primarne/izvedene zahtjeve (§6). Evidencija odluka redefinirana kao **funkcija** čija je forma zamjenjiva pod-metoda (§7.1). Uveden protokol valjanosti V1–V6 (§5.1) i hipoteza H4-DQI. Anatomija (§2) dobila retke *Hipoteza* i *Valjanost mjere*. Akronimsko pravilo suspendirano na WARNING do (§8). |
| 2026-07-12 | v0.2 | Restrukturirano; ispravljena inverzija kišobrana; doktrina „ponovljivi znanstveni test"; standardi kao pod-metode. |
| 2026-06-30 | v0.1 | Prva inačica. |

---

> *„Filozofija postavlja pitanje zašto; metodologija je ponovljivi test kojim tražimo odgovor."*

---

## Predgovor — položaj u doktrini

Ovo je Svezak I Wattleflow doktrine. Filozofija (`PHILOSOPHY.md`) je kišobran; metodologija stoji ispod njega.

Filozofija odgovara na pitanje *zašto* i što vrijedi kao znanje. Metodologija daje ponovljiv postupak kojim se to provjerava, mjeri i održava: pravila razvoja frameworka i metode kojima se ta pravila provjeravaju. Kao u znanosti, ne dokazuje tvrdnje, nego ih čini provjerljivima i ponovljivima.

Odnos slojeva je jednosmjeran: niži ne nadjačava viši. Odstupanje je izvod iz višeg sloja ili dokumentirana revizija (§7.1).

---

## 1. Metodologija kao ponovljivi znanstveni test

Metodologija je alat za odgovor na jasno postavljeno pitanje. Iz toga slijedi šest načela.

1. **Polazi od pitanja.** Mjeri, izlaže ili razrješava nejasnoću, otvoreno pitanje ili informacijsku prazninu. Bez pitanja ostaje procedura.
2. **Ponovljiva je.** Isti ulaz pod istim uvjetima daje isti rezultat. Postupak je opisan tako da ga čovjek ili stroj može reproducirati i osporiti (oborivost, *falsifiability*). Nalaz vrijedi samo uz trojku: verzija alata + verzija kriterija/registra + verzija platforme (D-10). Bez nje se rezultat ne može ponoviti, samo prepričati.
3. **Radije promatra nego zadire.** Mjerenje ne mijenja predmet ni ne zaustavlja tok; rezultat putuje s njim kao metapodatak. Razlog je P-10: pokazatelj koji postane cilj upravljanja prestaje mjeriti ono što je mjerio. Zadana uporaba mjere je dijagnostička. Upravljačka uporaba (*gate*) dopuštena je uz izričitu deklaraciju i ograničen opseg (§5.1, V6); nedeklarirani gate pretvara dijagnostiku u metu.
4. **Ne izmišlja toplu vodu.** Priznati standardi preuzimaju se kao pod-metode (Occam, DRY; §7).
5. **Mjeri i prosuđuje.** Kvaliteta se provjerava brojem (indeks, prag, varijanca) i prosudbom (koja su pravila prekršena, gdje, s kojim neto učinkom). Oba su valjani ishodi ako prilože dokaz vlastite valjanosti (§5.1).
6. **Ostavlja trag.** Rezultat se veže na pitanje, na standard koji mu je pod-metoda i na načelo koje ga opravdava (§6). Ako hrani doktrinarnu hipotezu, veže se i na nju (§2).

Zajednička ideja: metodologija je instrument kritičkog razmišljanja. Tvrdnja preživljava opravdanje, ne autoritet ni naviku. Isto vrijedi za mjere: mjera bez provjere valjanosti je tvrdnja na autoritet formule.

---

## 2. Anatomija metodologije

Svaka konkretna metodologija opisuje se istim skeletom, radi ponovljivosti i sljedivosti.

| Element | Sadržaj |
|---|---|
| **Znanstveno pitanje** | Što tražimo? Koju nejasnoću ili prazninu izlažemo? |
| **Hipoteza (ako postoji)** | Koju doktrinarnu hipotezu iz `PHILOSOPHY.md` test hrani (H1–H4…) i što bi je oborilo. |
| **Kontrolne točke / opseg** | Gdje i kada se mjeri; jedinica promatranja. |
| **Pod-metode (standardi)** | Koji priznati standardi se integriraju (ISO, PEP, …). |
| **Mjera (kvantitativno)** | Formula, tip skale, dopuštene agregacije, raspon, prag; numerička stabilnost. |
| **Valjanost mjere** | Status po §5.1 (V1–V6): dokazano, deklarirano, aspiracija. |
| **Prosudba (kvalitativno)** | Prekršena pravila; interpretacija; neto učinak. |
| **Ishod i sljedivost** | Rezultat (verdikt, metrika, vektor) + poveznica na pitanje, hipotezu, standard i načelo. |

Nove metodologije dodaju se po ovom obrascu; jezgra frameworka ostaje nepromijenjena (plug-in pristup).

---

## 3. Tri komplementarne discipline

Veliki informacijski sustavi ne opisuju se samo programskim inženjerstvom. Wattleflow integrira tri discipline (razrada „Iznad programskog inženjerstva", `PHILOSOPHY.md`):

```
                 Računarstvo
                       ▲
                       │
Informacijska ◄────────┼────────► Programsko
   znanost             │           inženjerstvo
                       ▼
                  Wattleflow
```

| Disciplina | Predmet | Teorijski temelj |
|---|---|---|
| Informacijska znanost | ontologija, taksonomija, metapodaci, provenijencija, semantička interoperabilnost, životni ciklus informacije | teorija informacije (P-06), kibernetika (P-05), semiotika (P-08), infološka tradicija (P-07) |
| Programsko inženjerstvo | arhitektura, životni ciklus, osiguranje kvalitete, testiranje, evolucija, obrasci dizajna | §5 |
| Računarstvo | algoritmi, složenost, formalni jezici, automati, teorija grafova, optimizacija | §5 |

---

## 4. Ontologija zahtjeva

Domenska ontologija opisuje *od čega se sustav sastoji*. Ortogonalna ontologija zahtjeva opisuje *što sustav mora zadovoljiti*. Zahtjevi su slojeviti po razini apstrakcije (BABOK v3; ISO/IEC/IEEE 29148).

| Koncept | Sloj | Definicija | Relacija | Izvor |
|---|---|---|---|---|
| Poslovni zahtjev | poslovni | cilj ili ishod poduzeća (*zašto*), neovisan o rješenju | `derivedFrom` problem / prilika / direktiva | BABOK, ISO 29148 |
| Zahtjev dionika *(prijedlog)* | dionici | potreba pojedinog dionika koju rješenje mora zadovoljiti; most između poslovnog zahtjeva i rješenja | `derivedFrom` poslovni zahtjev | BABOK |
| Poslovno pravilo | ortogonalno | politika ili definicija u vlasništvu biznisa; *definicijsko* (aletičko, ne može se prekršiti) ili *bihevioralno* (deontičko, može se prekršiti, nosi razinu provođenja) | `generates` / `constrains` FR i NFR | OMG SBVR |
| Funkcionalni zahtjev (FR) | rješenje | što sustav radi (ponašanje, sposobnost) | `realises` poslovni zahtjev i zahtjev dionika; `enforces` poslovno pravilo | ISO 29148 |
| Nefunkcionalni zahtjev (NFR) | rješenje | koliko dobro sustav radi (kvaliteta, ograničenje); **primaran** (iz ciljeva i načela) ili **izveden** (iz odluke, §6) | `realises` poslovni zahtjev i zahtjev dionika; usidren na model kvalitete | ISO/IEC 25010 |
| Prijelazni zahtjev *(prijedlog)* | prijelaz | što treba da se iz zatečenog stanja dođe u ciljno (migracija, obrnuto inženjerstvo zapisa, obuka); prestaje vrijediti po prijelazu | `derivedFrom` analiza jaza (§6) | BABOK |
| Ograničenje (Constraint) | polisemično | *zahtjevsko* sužava prostor rješenja (tehnologija, regulativa, budžet); *modelsko* je formalni boolean uvjet na element dizajna | zahtjevsko ~ vrsta NFR-a; modelsko `formalises` pravilo ili NFR | ISO 29148 / OMG OCL |

BABOK je zbirka znanja, ne standard; stupac je stoga *Izvor*. Preuzet je kao pod-metoda (§7).

Posljedice:

- FR i NFR su artefakti razine rješenja. Sljeduju se prema gore do zahtjeva dionika i poslovnih zahtjeva.
- Poslovno pravilo je zaseban artefakt: izvor i ograničenje zahtjeva. Definicijsko se preslikava u modelsko ograničenje (OCL invarijanta). Bihevioralno nosi deontičku modalnost i provodi se *enforcement* logikom.
- *Prijedlog:* svako pravilo `BR-<NN>-<nn>` u HLRQ-u deklarira vrstu (definicijsko / bihevioralno) i, za bihevioralno, razinu provođenja. Zatečeni `BR-WFL-01…02`, `BR-PTN-01…06`, `BR-PRC-01`, `BR-DRV-01` to ne nose (§11 t.4).
- Zahtjev dionika i prijelazni zahtjev nemaju oznaku ni registar. Ulaze uz dokumentiranu izmjenu (D-12); do tada su *prijedlog*.

---

## 5. Znanstveni temelji

Arhitektonske odluke moraju biti sljedive do priznatog inženjerskog znanja, ne intuicije.

| Područje | Izvori |
|---|---|
| Teorija dizajna | SOLID, GRASP, GoF, Information Hiding (Parnas [1]), Separation of Concerns (Dijkstra), Law of Demeter, Stable Dependencies / Stable Abstractions |
| Inženjerska načela | KISS, DRY, YAGNI, Principle of Least Astonishment, Open–Closed [69], Liskov Substitution [67][68], Occamova britva |
| Teorija sustava | Simon (P-12), Conway, Gall, Lehman, Brooks (P-13); kibernetika: povratna petlja, regulacija, zakon nužne raznolikosti (P-05); sinteza naspram analize (P-03, P-04) |
| Teorija mjerenja | reprezentacijski uvjet i tipovi skala (P-09) [15][16]; aksiomatsko vrednovanje metrika [17][18]; valjanost i vanjski kriterij [19]; erozija mjere pod upravljanjem (P-10); višekriterijska usporedba umjesto nelegalnog skalara [35]. Nosive tvrdnje s podrijetlom vodi [POSTULATE.md](POSTULATE.md) |
| Matematičko rezoniranje | od više valjanih rješenja prednost ima najjednostavnije koje zadovoljava zahtjeve; numerička stabilnost i složenost algoritma su prvorazredni kriteriji (Welford; §11 t.2) |

### 5.1 Valjanost mjere kao opći zahtjev

Nijedna mjera ne ulazi u metodologiju bez statusa po protokolu V1–V6. Protokol je generički; status po fazama vodi dokument pripadne metode, ne ovaj tekst.

| Faza | Pitanje | Što faza traži |
|---|---|---|
| **V1 Reprezentacijski uvjet** | Postoji li empirijska relacija neovisna o formuli? | Definiran uređaj „A je bolji od B po dimenziji d" i dokaz da ga mjera čuva (homomorfizam) [15][19]. |
| **V2 Tip skale i dopuštene operacije** | Što se s brojevima smije raditi? | Deklariran tip skale i dopuštene agregacije; zbroj preko nesumjerljivih dimenzija je indeks, ne mjerenje [16][35]. |
| **V3 Sadržajna pokrivenost** | Mjere li pravila ono što tvrde? | Preslikavanje pravilo → dimenzija; dimenzija bez pravila iskazuje se kao *nemjerena*, nikad kao savršena. |
| **V4 Prediktivna valjanost** | Znači li broj išta izvan sebe? | Povezanost s vanjskim kriterijem na uzorku različitom od kalibracijskog [19]. |
| **V5 Osjetljivost i stabilnost** | Ovisi li zaključak o proizvoljnom? | Perturbacija parametara; gdje poredak ovisi o njima, odluka se vraća na vektor i Pareto analizu [35]. |
| **V6 Erozija pod upravljanjem** | Što kad mjera postane cilj? | Deklarirana granica: indeks je dijagnostički, gate stoji na binarnim pravilima [22]. |

Minimum za svaku mjeru: V2, V3 i V6. V1, V4 i V5 smiju ostati otvoreni, ali tada je mjera *konformna* (relativna na deklarirani kriterij), ne deskriptivna tvrdnja o svijetu. To je operacionalizacija doktrine: „osobna preferencija nije opravdanje" vrijedi i za formule.

Isto vrijedi za popise tema (PESTLE, SWOT; §6): oni daju pokrivenost (V3), ne vrijednost. Tema bez posljedice za zahtjev iskazuje se kao *nije primjenjivo* (D-11), ne prešućuje.

---

## 6. Sljedivost

Strateška analiza daje ulaze; iz njih se izvode **primarni zahtjevi**; odluke se donose *pod* tim zahtjevima i **generiraju izvedene zahtjeve**; operativne metrike zatvaraju petlju kao empirijski dokaz.

```
Strateška analiza i elicitacija  (BABOK; §7)                ← prijedlog
   analiza dionika · PESTLE (vanjski čimbenici) · SWOT (unutarnji)
   analiza jaza as-is / to-be · analiza uzroka
                      ↓
Znanstvena teorija + Inženjersko načelo + Poslovni cilj
            (problem / prilika / direktiva)
                      ↓
     Zahtjev dionika  (§4)                                   ← prijedlog
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

**Ulazni sloj** (*prijedlog*). Svaki čimbenik strateške analize zapisuje se kao propozicija: „ako vrijedi P, sustav mora Q". Propozicija postaje ograničenje, poslovno pravilo ili NFR. Čimbenik bez posljedice za zahtjev izostavlja se; nerelevantan se označava *nije primjenjivo* (D-11). Prijelazni zahtjevi (§4) nastaju iz analize jaza.

**Smjer.** Primarni zahtjevi prethode odlukama i ograničavaju ih. Izvedeni zahtjevi nastaju iz odluka: zero-trust → `NFRQ-SEC-01/02/03`; clean-core → `NFRQ-ORG-01`. Primjer primarnog: `NFRQ-ORG-02` prethodio je odluci o korijenu i bio joj kriterij. Rečenica iz `PHILOSOPHY.md` („arhitektonske odluke generiraju mjerljive NFR-ove") odnosi se na izvedene.

**Oblik.** Sljedivost je graf, ne lanac. Čvorovi su načela, ciljevi, dionici, pravila, zahtjevi, odluke, artefakti i mjerenja. Bridovi su imenovane relacije: `derivedFrom`, `realises`, `constrains`, `generates`, `supersedes`. Dokumentacija zahtjeva i registri su serijalizacije čvorova. Vizualni prikaz grafa je legitiman primarni pogled (§7.1), ne izvor istine (D-13).

---

## 7. Integracija standarda kao pod-metoda

Metodologija dopunjuje standarde, ne zamjenjuje ih (§1 t.4). Usvajanje standarda je epistemička prosudba i bilježi se, ne događa po inerciji.

| Standard | Što pokriva | Pod-metoda za |
|---|---|---|
| Python PEP (osob. PEP 8) | stil jezika i konvencije imenovanja | samoopisivost (§8) |
| ISO/IEC/IEEE 42010 | opis arhitekture: gledišta i interesi | arhitektonsku dokumentaciju |
| ISO/IEC 25010 | model kvalitete proizvoda, sidro za NFR | NFR registar ([`requirements/03-NFRQ/`](requirements/03-NFRQ/NFRQ-000-INDEX.md)) |
| ISO/IEC 25012 | model kvalitete podataka | DQI dimenzije (`requirements/05-METHODS/dqi.md`) |
| ISO 8000-8 / 8000-61 | mjerenje i procesni model kvalitete podataka | DQI formula i kontrolne točke (isto) |
| ISO/IEC 12207 | procesi životnog ciklusa softvera | proces razvoja |
| ISO/IEC/IEEE 29148 | inženjerstvo zahtjeva | ontologija zahtjeva (§4) |
| IIBA BABOK v3 *(zbirka znanja; prijedlog, D-12)* | poslovna analiza: razine zahtjeva, dionici, elicitacija, strateška analiza | ontologija zahtjeva (§4), ulazni sloj sljedivosti (§6) |
| OMG SBVR | poslovni rječnik i pravila | ontologija zahtjeva (§4) |
| OMG OCL | formalna ograničenja na razini modela | modelska ograničenja |
| NIST OSCAL | strojno čitljive sigurnosne kontrole | compliance |
| W3C RDF / OWL / PROV-O | ontologija, semantika, provenijencija | informacijski sloj; graf sljedivosti (§6) |
| SPDX / CycloneDX *(budući)* | software bill of materials | supply-chain (`NFRQ-SEC-03`) |


**Pojasnjenje za SPDX/CycloneDX**

| | Cyclone | SPDX |
|---|---|---|
| Nositelj | OWASP, ECMA-424 | Linux Foundation, ISO/IEC 5962 |
| Težište |	sigurnost i lanac opskrbe | licence i usklađenost |
| Snaga| ranjivosti, VEX, ovisnosti | pravna provenijencija, ISO status|



### 7.1 Evidencija odluka kao funkcija; dokumentacija kao zamjenjiva pod-metoda

Doktrina zahtijeva **funkciju**: svaka značajna odluka evidentira se s kontekstom, alternativama, cijenom i svjedočanstvom, provjerljivo i opozivo. **Format evidencije je metoda, ne doktrina.** Trenutna pod-metoda je dokumentacija zahtjeva (HLRQ, FRQ, NFRQ), koja kontekst, odluku, cijenu i svjedočanstvo nosi u istom zapisu (genealogija [44][45][46]). Zatečena je, funkcionalna i zamjenjiva istom prosudbom kao svaki standard iz §7.

Kriteriji za nasljednika (funkcijski, ne formatski):

- sljedivost do načela i zahtjeva;
- zapis razmotrenih alternativa;
- razdvajanje dokazanog od aspiracije (Svjedočanstvo);
- vezanje na strojno provjeriv kriterij (Registar);
- revizibilna povijest, uključujući povučene odluke.

Kandidat-nasljednik: **odluka kao čvor grafa sljedivosti** (§6) s relacijama omogućuje / sprječava / nadomješta [45], serijaliziran strojno čitljivo (RDF / PROV-O iz §7). Zapis zahtjeva tada postaje generirani pogled na čvor, ne izvor istine. Prijelaz, kad za njega bude dokaza, ide kroz dokumentiranu reviziju ove sekcije.

---

## 8. Samoopisivost i imenovanje

Softver se sam objašnjava: uloga komponente čitljiva je iz imena, hijerarhije nasljeđivanja, mjesta u paketu i ovisnosti. Ime **klase** mora biti domenski kvalificirano i uloga-eksplicitno. Gole generičke imenice (`Manager`, `Helper`, `Parser`) kao ime klase su zabranjene; kvalificirani oblici (`DriverS3UriParser`, `KafkaReadProcessor`) su ispravni. Imenovanje je semiotička politika (P-08): ime je znak čiji odnos prema ulozi registar fiksira, a lint provodi. To je arhitektonsko pitanje, ne stilska preferencija. Provediva gramatika i kontrolirani rječnik: [`NFRQ-ORG-02`](requirements/03-NFRQ/NFRQ-ORG-02-class-nomenclature.md).

Pravilo vrijedi za ime klase, ne za **ulogu**. Uloge iz ontologije sloja (`Parser`, `Strategy`, `Document`, `Memento`) su primitivi, ne klase, i nisu obuhvaćene zabranom. Je li prefiks `Generic` (`GenericParser`) domenska kvalifikacija, nije odlučeno (§11 t.5).

> **Otvorena odluka (pending):** pisanje akronima (`CreatePDFDocument` po PEP 8 naspram zatečenog `CreatePdfDocument`). Do odluke je pripadno lint pravilo **suspendirano na WARNING s deklariranim waiverom**: smije upozoravati (upozorenja su građa za odluku), ne smije kažnjavati. Registar je draft i prilagođava se dokazima; provođenje jedne strane neodlučenog pitanja kršilo bi kaskadu.

---

## 9. AI-potpomognuto inženjerstvo i kritičko razmišljanje

Arhitektura podržava ljudsko i strojno zaključivanje: generiranje dokumentacije, analizu arhitekture, otkrivanje ovisnosti, obrnuto inženjerstvo, refaktoriranje, automatizirano testiranje. Semantička dosljednost (§8) koristi inženjerima i AI sustavima.

Kritičko razmišljanje vrijedi jednako za čovjeka i AI-agenta: nijedna odluka ne prolazi na autoritet ni naviku. Kod nedoumice agent pita, izlaže trošak i alternative, i ne pogađa (`ARCHITECTURE.md` §5). Metodologija je alat te discipline: ponovljiv test kojim se tvrdnja, ljudska ili strojna, provjerava, ne pretpostavlja.

---

## 10. Nefunkcionalni zahtjevi i konformnost

Zahtjevi su primarni (iz ciljeva i načela) ili izvedeni (iz odluka). Svaki NFR usidren je na karakteristiku iz ISO/IEC 25010, nosi oznaku `NFRQ-<KAT>-<NN>` (prijedlog), kriterije prihvaćanja, metodu verifikacije i sljedivost prema gore (§6). Registar: [`requirements/03-NFRQ/`](requirements/03-NFRQ/NFRQ-000-INDEX.md), zapis po zahtjevu, uz zajedničke definicije (`NFRQ-DEF-01`), povelju `[M]` (`NFRQ-DEF-02`) i Dodatak A o redoslijedu faceta (`NFRQ-APX-01`). Opseg kategorija vodi indeks registra, ne ovaj tekst (D-13).

Verifikacija konformnosti je **statička provjera nad izvornim kodom**, po anatomiji iz §2:

- kriterij je verzioniran odvojeno od alata;
- nalaz je reproducibilan samo uz trojku (alat, kriterij, platforma) (D-10);
- zelena konformnost arhivira se kao **C-snimka**;
- izlaz je vektorski, bez skalarne ocjene (§5.1);
- pokreće se na zahtjev; u CI-ju je kontrola kvalitete uz ogradu iz §1 t.3: gate stoji na binarnim pravilima, indeksi ostaju dijagnostički.

Alat, prekidači, opseg i ključevi registra: `tools/README.md`. Obveze koje verdikt nameće: `CONFORMANCE.md` §9. Ovdje se ne prepričavaju.

---

## 11. Otvoreno

1. **Svesci.** Zapisan je samo Svezak I. Postoje li ili su planirani drugi svesci, nije zapisano.
2. **Welford.** Ranije upućivanje „§3" bilo je krivo (§3 opisuje discipline). Ispravno odredište nije utvrđeno; kandidat je dokument DQI metode (`requirements/05-METHODS/dqi.md`), neprovjereno.
3. **Rječnik.** Pratioci navode `dictionary.yaml`; vještina `wattleflow-docs` §7 navodi `tools/dictionary.json` (vlasnik `wattleflow-tools-wem`). Odnos dviju datoteka nije zapisan.
4. **Ontologija zahtjeva (§4).** Zahtjev dionika, prijelazni zahtjev i vrsta poslovnog pravila su *prijedlog* i čekaju dokumentirana izmjena (D-03, D-12). Zatečeni `BR-WFL-01…02`, `BR-PTN-01…06`, `BR-PRC-01`, `BR-DRV-01` (`HLRQ-01` §5) ne deklariraju vrstu ni razinu provođenja.
5. **Imenovanje (§8).** Je li prefiks `Generic` domenska kvalifikacija ili gola imenica s prefiksom, nije odlučeno. Ovisi o `NFRQ-ORG-02` i dokumentiranoj odluci.
6. **Akronimi.** pending (§8).
7. **Ulazni sloj sljedivosti (§6) i BABOK (§7).** Prijedlog; čeka dokumentiranu izmjenu (D-12). PMESII nije uključen: prikladniji je za analizu sustava i prijetnji (`NFRQ-SEC-*`) nego za poslovno okruženje.
8. **Numerirane reference.** Oznake [1], [15]–[19], [22], [35], [44]–[46], [67]–[69] nemaju popis literature u ovom dokumentu. Mjesto razrješenja nije navedeno.
9. **Usklađenost s mlađim odlukama.** Odluke su referirane po vještini `wattleflow-docs` i `HLRQ-01-GENERIC-LAYER`, ne po zasebnim zapisima. Neprovjereno.

---

## Završna riječ

Metodologija je most između filozofije i koda. Filozofija postavlja pitanje i mjerilo znanja; metodologija daje ponovljivi test kojim se do odgovora dolazi i kojim se on brani. Softver ne smije samo raditi: mora biti razumljiv, objašnjiv, održiv, auditabilan i obrazovan, a svaka tvrdnja o njemu, uključujući svaku mjeru, mora preživjeti provjeru.

---

## Bilješka o sintezi

Trag racionalizacije (P-14) vodi tablica *Povijest izmjena*; pojedinačne intervencije nose git i dokumentacija. Prikaz koji se raziđe s izvorom je nalaz, ne povijest (D-13).