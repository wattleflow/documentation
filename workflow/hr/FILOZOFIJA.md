# Wattleflow inženjerska filozofija

## Prema znanstveno utemeljenoj metodologiji programskog inženjerstva

**Verzija:** Draft v0.4.1
**Status:** živi dokument; engleska inačica nastaje pri prelasku na v1.0
**Izvorni jezik:** hrvatski

> *„Arhitektura nije dokumentacija softvera. Arhitektura je znanstveni temelj iz
> kojeg softver proizlazi."*

---

## Predgovor

Wattleflow je nastao iz prakse, kao rad pojedinca, gdje su manjak vremena i
praktične potrebe diktirali tempo i pristup. Ova filozofija označava svjestan
zaokret: od implementacijom vođenog razvoja prema znanstveno utemeljenom,
multidisciplinarnom inženjerstvu. To nije naknadna ambicija nego izvorna
namjera, o čemu svjedoče raniji radovi autora, posebice istraživanje na polju
informacijskih znanosti ([RESEARCH.md](https://github.com/wattleflow/core/blob/ion/docs/research/RESEARCH.md)): teza da su informacije vrijednost sa
značenjem i kontekstom, a njihova zaštita strukturna pretpostavka pouzdanog
sustava — ne dodatak.

## Sažetak

Ovaj dokument uspostavlja inženjersku filozofiju na kojoj počiva Wattleflow.
Cilj nije dokumentirati niz odluka, nego postaviti rigor i metodologiju za
projektiranje informacijskih sustava utemeljenu na znanstvenim načelima,
međunarodno priznatim standardima i praktičnim potrebama inženjeringa
suvremenih, AI-potpomognutih sustava.

Vodeća hipoteza glasi: **arhitektura ne smije nastati kao nusprodukt
implementacije.** Arhitektura stoji na inženjerskim načelima, a načela
proizlaze iz utemeljenog znanstvenog pristupa i strukovnog znanja. Dopuštena
je — i očekivana — iteracija između arhitekture i implementacije, ali unutar kaskade 
upravljanja opisane u Doktrini: implementacijsko iskustvo
smije revidirati arhitekturu isključivo kroz dokumentiranu odluku (zapis odluke - DR),
nikad prešutnom izmjenom koda.

Ovaj je dokument stoga **anti-manifest**: manifest deklarira vrijednosti bez
tereta dokaza; anti-manifest deklarira teret dokaza kao vrijednost. Dvije
stvari dokument čini eksplicitnima i obvezujućima za svu ostalu
dokumentaciju:

1. **Filozofija je kišobran** nad razvojem — najviša instanca iz koje
   proizlaze policyji, principi i metode (sekcija *Doktrina*).
2. **Kritičko razmišljanje je temeljna doktrina** koju Wattleflow ne samo
   usvaja nego i propagira — ugrađuje ga u način rada umjesto da ga prepusti
   slučaju (sekcija *Kritičko razmišljanje*).

Vodeći operativni princip je **mjerljivost**: svaka doktrinarna tvrdnja mora
imati opovrgljiv oblik (sekcija *Hipoteze*). Arhitektura i funkcionalnosti
izvode se iz zahtjeva koji žive u FR.md (funkcionalni) i NFR.md
(nefunkcionalni); temelji metodologije razrađeni su u Svesku I
(METHODOLOGY.md).

## Sadržaj (svesci)

| Svezak | Naslov | Lokacija |
|---|---|---|
| I | Temelji (Foundations) | METHODOLOGY.md |
| II | Arhitektonska načela | *(planirano)* |
| III | Decision Records (zapisi odluka) | documentation/dr/ *(aktivan)* |
| IV | Dizajn sustava | *(planirano)* |
| V | Razvojni standardi i funkcionalnosti | FR.md i NFR.md *(aktivan)* |
| VI | Osiguranje kvalitete | *(planirano)* |
| VII | AI-potpomognuto programsko inženjerstvo | *(planirano)* |
| VIII | Upravljanje, usklađenost i informacijska znanost | *(planirano)* |

---

## Motivacija

Suvremeni razvoj softvera često pati od inverzije prioriteta. Tipičan
projekt razvija se slijedom:

```
Poslovni zahtjev
      ↓
Korisnička priča (user story)
      ↓
Implementacija
      ↓
Arhitektura  (rekonstruirana naknadno, iz koda)
```

U ovom slijedu arhitektura nastaje gomilanjem — kao talog implementacijskih
odluka koje nitko nije donio eksplicitno. Posljedice su poznate:
arhitektonski drift, nedosljedan dizajn, rastući tehnički dug, slaba
održivost, smanjena objašnjivost i ograničena obrazovna vrijednost.

Ova inverzija nije povijesna nužnost. Iterativni i inkrementalni razvoj
prethodi agilnom pokretu desetljećima, izvorni „vodopadni" model bio je
izričito iterativan i upozoravao da jednosmjerni prolaz „poziva na neuspjeh"
(P-15), a povratna petlja kao znanstveni instrument izvire iz statističke
kontrole procesa — gdje se vrti nad *teorijom* (hipoteza → mjerenje →zaključak), 
a ne nad backlogom (P-01). Iskustvo bez teorije ne uči ništa(P-02).
Wattleflow zato ne odbacuje iteraciju nego joj vraća znanstveni sadržaj.

Wattleflow usvaja znanstvena i inženjerska načela te poslovne ciljeve kao
**ulaze**; zahtjevi se iz njih **izvode**; mjerenja zatvaraju petlju:

```
Znanstvena načela + Inženjerska načela + Poslovni ciljevi
      ↓
Funkcionalni i nefunkcionalni zahtjevi  (FR.md, NFR.md)
      ↓
Arhitektura  ⇄  Implementacija   (iteracija pod kaskadom; revizija samo kroz DR)
      ↓
Testiranje + Konformnost (wem_lint) + Operativne metrike
      ↓
Empirijski dokaz → vraća se u načela
```

Arhitektura i dokumentacija time postaju prvorazredni inženjerski artefakti
i **epistemološka građa**: zapis znanja, ne dnevnik rada (P-14). Poslovni
ciljevi su ulaz koji oblikuje zahtjeve — ne artefakt naknadno podmetnut
ispod njih. Potpuni model sljedivosti i ontologija zahtjeva razrađeni su u
Svesku I (§4.1, §9).

---

## Doktrina: filozofija kao kišobran

Filozofija je najviša instanca razvojnog ciklusa — kišobran iz kojeg
proizlaze policyji, principi i metode. Njezin je diskurs po naravi
**epistemološki**: bavi se time - koje znanje smatramo utemeljenim, kako ga
stječemo, provjeravamo i revidiramo.

Iz epistemološkog diskursa proizlazi **ontološka perspektiva** — što u
domeni i u prostoru zahtjeva *postoji* (domenska ontologija: Workflow,
Document, Strategy, …; ontologija zahtjeva: poslovni zahtjev, pravilo,
FR/NFR, ograničenje). Ontologija nije temelj *ispod* filozofije nego njezin
**proizvod**: filozofski diskurs odlučuje koji entiteti i odnosi postoje i
po kojem se kriteriju to znanje brani.

```
      Filozofija      (kišobran — epistemološki diskurs)
           │
           ├──────────►  Ontološka perspektiva  (što postoji: domena + zahtjevi)
           │
           ▼
       Policyji       (obvezujuća pravila — CLAUDE.md, sigurnosna politika)
           │
           ▼
       Principi       (inženjerska i znanstvena načela — Parnas, SOLID, DRY, …)
           │
           ▼
       Metode         (metodologije, standardi, alati — PEP, ISO, SBOM, wem_lint)
```

**Pravilo subordinacije:** niži sloj nikad ne nadjačava viši. Policy se ne
uvodi mimo filozofije, princip se ne bira mimo policyja, metoda se ne usvaja
mimo principa. Svako odstupanje je ili izvod iz višeg sloja ili
**dokumentirana revizija** tog sloja — a jedini legalan oblik revizije je
zapis odluke (DR). Kaskada djeluje u oba smjera: odozgo propisuje, odozdo se revidira
dokazima (primjer: usvajanje wheel RECORD/SBOM mehanizma umjesto vlastitog
digest-registra — nova spoznaja o postojećem standardu profinila je metodu,
uz očuvanu sljedivost do principa Occam/DRY i do epistemičke jezgre
filozofije).

**Epistemička jezgra povjerenja.** Na filozofijskoj razini vrijedi:
*povjerenje se ne pretpostavlja nego dokazuje* — poseban slučaj opće
doktrine da tvrdnja bez svjedočanstva ne obvezuje. Imenovana sigurnosna
politika koja iz toga slijedi (zero-trust arhitektura, granice povjerenja,
integritet lanca opskrbe) živi sloj niže, u policyjima, i revidira se poput
svake politike — kroz DR. Time filozofija ne propisuje sigurnosnu
tehnologiju nego epistemički standard koji svaka sigurnosna tehnologija
mora zadovoljiti.

## Kritičko razmišljanje — temeljna doktrina

Motor epistemološkog diskursa je kritičko razmišljanje: sustavno
preispitivanje pretpostavki, zahtijevanje opravdanja i spremnost na reviziju
u svjetlu dokaza. Koncept je u znanosti dobro poznat; ovdje se navodi
eksplicitno jer ga Wattleflow ne samo usvaja nego i **propagira** — ugrađuje
u način rada. Kritičko razmišljanje nije stav nego **provediva praksa**:

* Nijedna odluka ne poziva se na autoritet ili naviku — mora preživjeti
  prosudbu: znanstveno načelo, priznati standard, matematičko rezoniranje
  ili empirijski dokaz. **Osobna preferencija nije opravdanje.**
* **DR je artefakt kritičkog razmišljanja** [44][45][46] — bilježi kontekst,
  razmotrene alternative, cijenu i svjedočanstvo, pa je odluka provjerljiva
  i opoziva. Odluka koja ne navodi svjedočanstvo deklarira se kao
  aspiracija, ne kao dokazana.
* Pravilo subordinacije („promjena samo kroz dokumentiranu reviziju") jest
  kritičko razmišljanje **institucionalizirano** — sprječava drift po
  inerciji.
* Vrijedi jednako za čovjeka i za AI-agenta: kod nedoumice se pita, izlaže
  trošak i alternative, a ne pogađa (POLICY.md §5; Svezak I §8).
* Vrijedi i za samu filozofiju: doktrinarna tvrdnja koja ne navodi što bi je
  oborilo koristi autoritet — što ova doktrina zabranjuje. Zato svaka
  nosiva tvrdnja ima opovrgljiv oblik (sljedeća sekcija).

**Znanje raste — i taj rast vozi diskurs.** Teoretsko znanje nije statično:
kroz epistemološku analizu i nova saznanja ono se širi, ulazi natrag u
filozofski diskurs i kaskadno preoblikuje policyje, principe i metode. To je
operativni oblik povratne petlje iz Motivacije: mjerenja i nove spoznaje
nisu kraj lanca nego njegov novi ulaz. Doktrina zahtijeva da svaki takav
pomak bude zabilježen (DR/FR-NFR) s obrazloženjem — čime rast znanja ostaje
sljediv, a ne proizvoljan.

## Hipoteze

Doktrina mjerljivosti primjenjuje se najprije na samu filozofiju. Nosive
tvrdnje imaju opovrgljiv oblik; svaka navodi mjeru i svjedočanstvo koje bi
je oborilo. Mjerenja žive u analizi konformnosti (wem_lint, C-snimke) i u
radu o kvantifikaciji arhitektonske granice.

**H1 — Arhitektura-prvo pod upravljanjem smanjuje arhitektonski drift.**
*Mjera:* slaganje deklarirane i detektirane particije sustava
(normalizirana uzajamna informacija) praćeno kroz verzije.
*Obara je:* sustavni pad slaganja unatoč provedbi kaskade, ili jednako
slaganje u usporedivim sustavima bez upravljanja.

**H2 — Provedeni semiotički registar smanjuje semantičku entropiju
sustava.** Kontrolirani vokabular (registar imenovanja, ORG-02/03) je
semiotički stabilizator (P-08): fiksira odnos znaka i uloge.
*Mjera:* trend povreda imenovanja po verziji; broj znakova s višestrukom
semantikom (nulta točka: jedan TypeVar `T` s četiri uloge, zabilježeno u
analizi jezgre).
*Obara je:* stagnacija ili rast višeznačnosti pod provedbom registra.

**H3 — Verzionirani kriterij čuva valjanost instrumenta.** Instrument
mjerenja korodira i tiho (P-10); verzioniranje kriterija (registar + alat +
platforma) čini razilaženje proxyja i konstrukta vidljivim.
*Mjera:* latencija otkrivanja incidenata valjanosti (nulta točka: ABS-03
incident otkriven unutar dva mjerna ciklusa, ne godinama).
*Obara je:* incident valjanosti koji verzionirani sustav propusti dulje od
neverzioniranog benchmarka, ili nalazi dviju verzija kriterija koji se ne
mogu pomiriti deklariranim napomenama.

Popis nije zatvoren: nova doktrinarna tvrdnja ulazi u dokument tek s
pripadajućom hipotezom ili s eksplicitnom deklaracijom da je konceptualna,
a ne empirijska.

---

## Iznad programskog inženjerstva

Veliki informacijski sustavi postoje na sjecištu **računarstva**,
**programskog inženjerstva** i **informacijske znanosti**. Wattleflow
namjerno integrira sve tri — s posebnim naglaskom na treću, povijesno
potisnutu iz razvojne prakse premda joj je temelj: teorija informacije
daje mjeru nesigurnosti i granice prijenosa (P-06), kibernetika povratnu
petlju i regulaciju te zakon nužne raznolikosti (P-05), semiotika odnos
znaka i značenja u organizacijskom kontekstu (P-08), a infološka tradicija
razliku podatka i informacije — informacija je funkcija podatka, predznanja
i vremena interpretacije (P-07). Svaka disciplina i njezin
doprinos razrađeni su u Svesku I (§3).

```
                 Računarstvo
                      ▲
                      │
Informacijska ◄───────┼───────► Programsko
   znanost            │        inženjerstvo
                      ▼
                 Wattleflow
```

## Ontologija prije implementacije

Implementacija slijedi ontologiju, ne obrnuto. Wattleflow definira
eksplicitnu **domensku ontologiju** (Workflow, Document, Strategy, Driver,
Repository, Connection, Processor, Pipeline, Blackboard, Memento) i zasebnu
**ontologiju zahtjeva**. Obje su razrađene u Svesku I (§4, §4.1) — i obje
su, po Doktrini, proizvod epistemološkog diskursa: mijenjaju se kad se
promijeni znanje, kroz DR.

## Znanstveni temelji

Svaka značajna arhitektonska odluka sljediva je do utemeljenog znanja:
teorije dizajna (Parnas [1], SOLID, GRASP, GoF, Dijkstra, Law of Demeter,
Stable Dependencies/Abstractions), inženjerskih načela (KISS, DRY, YAGNI,
Open–Closed, Liskov, Principle of Least Astonishment, Occamova britva),
teorije sustava (Simon [2], Conway, Gall, Lehman, Brooks [29]) i teorije
mjerenja (reprezentacijski uvjet i tipovi skala, P-09). Katalog i primjena
su u Svesku I (§5); nosive tvrdnje s podrijetlom i ključevima vodi registar
postulata (`POSTULATE.md`).

## Arhitektura kao obrazovni artefakt

Svrha arhitekture nije samo proizvesti softver nego prenijeti inženjersko
znanje: zašto komponente postoje, zašto su odgovornosti razdvojene, zašto
ovisnosti teku u jednom smjeru, zašto su konvencije imenovanja važne i
zašto su odabrani određeni obrasci. Arhitektura je stoga obrazovni resurs
(Svezak I §6) — a dokumentacija je sustav s više publika (arhitekt,
implementator, tester, sigurnosni analitičar, integrator, poslovni
korisnik, revizor), od kojih svaka treba svoj artefakt: jedan format za sve
publike krši zakon nužne raznolikosti (P-05). Dublje: arhitektura je
eksternalizirani dio teorije koju tim drži o sustavu, pa je obrazovna
vrijednost arhitekture upravo prijenos te teorije (P-16).

## Arhitektonska samoopisivost

Arhitektura mora biti razumljiva bez čitanja implementacije: ime klase,
njezino nasljeđivanje, mjesto u paketu i ovisnosti komuniciraju sloj,
domenu, odgovornost i životni ciklus. Imena moraju biti **domenski
kvalificirana i uloga-eksplicitna** — gole generičke imenice (`Manager`,
`Helper`, `Parser`, `Piece`, `Sheet`) su zabranjene; kvalificirani oblici
(`DriverS3UriParser`, `SheetNestingPlacement`) su ispravni. Ovo je
semiotička politika (P-08): ime je znak čiji odnos prema ulozi registar
fiksira, a lint provodi. Provediva gramatika je NFR-ORG-02 (NFR.md);
obrazloženje u Svesku I §7.

## AI-potpomognuto programsko inženjerstvo

Veliki jezični modeli uvode novog potrošača arhitekture. Povijesno je
arhitektura služila ljudima; danas mora podržati i strojno zaključivanje —
generiranje dokumentacije, reverzni inženjering, analizu ovisnosti,
validaciju, refaktoriranje i generiranje koda. Semantička dosljednost
koristi i ljudima i strojevima; doktrina kritičkog razmišljanja vrijedi za
oba (kod nedoumice: pitati, izložiti trošak i alternative — ne pogađati).
Razrada u Svesku I §8.

## Decision Records (DR)

Praksa zapisa odluka (naslijeđeni naziv: ADR) ima jasnu genealogiju: odluka kao prvorazredni objekt
arhitekture i „isparavanje znanja" kad odluke nemaju zapis [45], bogati
predložak zapisa [44], te minimalni format koji je praksu učinio održivom
[46]. Wattleflow usvaja prošireni predložak (Status / Kontekst / Odluka /
Ugovor / Cijena / Svjedočanstvo / Registar) — bogatstvo Tyree–Akerman
linije s disciplinom troška: polja postoje jer ih alat i proces troše, ne
radi potpunosti. Polje *Svjedočanstvo* razdvaja dokazano od aspiracije;
polje *Registar* veže odluku uz strojno provjeriv kriterij, čime zapis odluke
prestaje biti samo zapis i postaje izvor pravila konformnosti.

Unutar Wattleflowa DR-ovi imaju četiri komplementarna cilja:

* **Poslovna usklađenost** — odluke ostaju sljedive do poslovnih ciljeva.
* **Reverzni inženjering** — arhitektura se može rekonstruirati iz zapisa.
* **Arhitektonsko upravljanje** — dosljednost kroz dugoročnu evoluciju.
* **Znanstveno opravdanje** — svaka veća odluka opravdana je priznatim
  načelima, teorijama, matematičkim rezoniranjem ili standardima.

Numeracija: serija po projektu s prefiksom — `DR-COR` (core), `DR-WFL`
(workflow), `DR-PRC` (processors), `DR-CAD` (cad). Oznaka je time globalno
jedinstvena bez središnjeg brojača, a odluka pripada seriji onog projekta
čiji artefakt mijenja. 

Format zapisa je zamjenjiva
pod-metoda doktrinarne funkcije evidencije odluka (Svezak I, §8.1) —
funkcija je obvezna, forma se revidira dokazima. Razrada u Svesku III.

## Funkcionalni zahtjevi

ISO/IEC/IEEE 29148 definira funkcionalne zahtjeve kao one koji opisuju
ponašanje sustava, usluge koje mora pružiti, ulaze i izlaze, te reakcije na
vanjske događaje. U Wattleflowu je ova ideja pretočena u `FR.md`, gdje
funkcionalni zahtjevi dobivaju jedinstvene identifikatore, kriterije
prihvaćanja i sljedivost prema poslovnim ciljevima i arhitekturi.

Glavne discipline iz 29148 koje projekcija u ovom dokumentu prati su:

* `Stakeholder needs and expectations` — funkcionalni zahtjevi polaze od
  interesa dionika i poslovnih ciljeva, ne od tehnologije.
* `Elicitation` i `Analysis` — zahtjevi se prikupljaju, analiziraju, razjašnjavaju
  i prioritetiziraju prije arhitektonskih odluka.
* `Documentation` — svaki FR treba imati jedinstveni ID, opis, rationale,
  acceptance criteria i traceability reference. To je u skladu s registrom
  u `FR.md`.
* `Verification` i `Validation` — funkcionalni zahtjevi se verificiraju kao
  ispravni, potpuni i provjerljivi, te validiraju s potrebama korisnika.
* `Traceability` i `Requirements management` — promjene se prate i mapiraju
  kroz cijeli životni ciklus zahtjeva.

Ovaj pristup potvrđuje odluku dokumenta: funkcionalnosti se vode u
posebnom registru (`FR.md`), a arhitektura ih slijedi kao izvedene, ne obrnuto.

## Nefunkcionalni zahtjevi

Arhitektonske odluke generiraju **mjerljive** nefunkcionalne zahtjeve,
svaki usidren na ISO/IEC 25010, identificiran kao `NFR-<KATEGORIJA>-NN`,
provođen kroz standarde kodiranja, arhitektonske preglede, statičku analizu
i CI/CD kontrole. Mjera konformnosti je vektorska — po dimenzijama, bez
skalarne ukupne ocjene, jer ponderirani zbroj preko nominalnih i ordinalnih
skala nije definirana operacija (P-09). Registar se vodi u NFR.md.

## Usklađenost sa standardima

Umjesto zamjene standarda, Wattleflow ih integrira: PEP, ISO/IEC/IEEE
42010, ISO/IEC 25010/25012/12207, OMG SBVR i OCL, NIST OSCAL, W3C
RDF/OWL/PROV-O te (budući) SPDX/CycloneDX. Standard ulazi kroz epistemičku
prosudbu, ne po inerciji; što koji pokriva navedeno je u Svesku I (§10).

## Vizija

Dugoročni cilj Wattleflowa nije pružiti još jedan workflow framework, nego
pokazati da se suvremeni informacijski sustavi mogu razvijati metodologijom
koja spaja znanstveno rezoniranje, programsko inženjerstvo, informacijsku
znanost, međunarodno priznate standarde i praktičnu implementaciju.

Rezultirajući framework treba biti tehnički rigorozan, obrazovan,
objašnjiv, održiv, usklađen, AI-kompatibilan i arhitektonski samoopisiv.

U ovoj filozofiji arhitektura nije dokumentacija softvera.

**Arhitektura je znanstveni temelj iz kojeg softver proizlazi.**

---

## Reference

Ovaj dokument ne održava vlastitu tablicu literature (P-20: prikaz nije
izvor istine). Ključevi [n] upućuju na konsolidiranu tablicu
`documentation/LITERATURE.md`; oznake P-NN na registar postulata
`documentation/POSTULATE.md` (tvrdnja s podrijetlom, primjenom i ključem).

Standard citiranja: numerički ključevi (IEEE stil), jedinstveni preko svih
doktrinarnih dokumenata i rada, uz **append-only** pravilo (novi ključ
isključivo na kraj tablice — umetanje bi prenumeriralo citate u svim
dokumentima). Evolucijski put prema stabilnim identifikatorima
(literatura.yaml, brojevi kao generirani prikaz) evidentiran je kao DR
kandidat.
