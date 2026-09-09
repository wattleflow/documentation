# Wattleflow rječnik (kontrolirani rječnik doktrine)

| | |
|---|---|
| **Verzija** | Draft v0.1 |
| **Nadređeni dokumenti** | [`PHILOSOPHY.md`](PHILOSOPHY.md) (v0.4.2), [`METHODOLOGY.md`](METHODOLOGY.md) (v0.3.2) |
| **Srodni artefakti** | `tools/dictionary.json` (kontrolirani vokabular koda), [`LITERATURE.md`](LITERATURE.md) (ključevi 1–64) |
| **Izvor** | [`dictionary.yaml`](dictionary.yaml) — **ovo je generirani prikaz, ne izvor istine** (D-13) |
| **Upravljanje** | natuknica se dodaje, mijenja ili proglašava zastarjelom isključivo kroz zapis odluke (DR); v. §Održavanje |

## Svrha

Rječnik je **semiotički stabilizator diskursa** [38][51]: fiksira odnos naziva i
značenja za ljudski jezik doktrine, kao što `tools/dictionary.json` fiksira
vokabular koda. Cilj je smanjiti semantičku entropiju (hipoteza H2) — u
srodnim radovima ključni pojmovi (arhitektura, odluka, politika, mjera) ostaju
nedefinirani ili višeznačni, pa rasprava klizi. Ovdje svaki nosivi pojam ima:
definiciju, ontološki status, sloj kaskade kojem pripada i referencu gdje
postoji. Višeznačnosti se ne izbjegavaju nego **izrijekom razrješavaju**
(Dio V).

Pravilo prvenstva: u sukobu značenja, ovaj rječnik nadglasava kolokvijalnu
upotrebu u svim doktrinarnim dokumentima; sukob rječnika s PHILOSOPHY.md ili
METHODOLOGY.md je greška koja se razrješava DR-om, ne prešutno.

## Konvencije natuknice

**Pojam** *(engl. izvornik gdje je relevantan)* — definicija u jednoj do tri
rečenice. `[ontološki status | sloj | ključ reference]`

Ontološki statusi: **čin** (događaj, nepromjenjiv), **stanje** (na snazi,
evoluira), **znak** (reprezentacija), **artefakt** (zapis/alat), **svojstvo**
(mjerljiva karakteristika).

---

# Dio I — Ontološke kategorije (pentada i kaskada)

**Odluka** *(decision)* — evidentiran čin prijelaza: mijenja jedno ili više
stanja (politiku, arhitekturu, kriterij), nosi autora, kontekst, razmotrene
alternative, cijenu i temelje. Nepromjenjiva; kasnija odluka je nadomješta
(`supersedes`), nikad ne prepisuje. `[čin | sve razine | 44][45]`

**Zapis odluke (DR)** *(decision record)* — artefakt koji serijalizira
odluku; trenutna pod-metoda evidencije (METHODOLOGY §7.1). 
DR nosi polje `sloj` koje odluku smješta u kaskadu.
`[artefakt | metoda | 44][46]`

**Politika** *(policy)* — normativno stanje na snazi: što *mora* vrijediti.
Deontička je (može se prekršiti, pa se provodi); donosi je, mijenja i ukida
odluka. Svaka politika deklarira provedbeni mehanizam i evaluacijski signal —
politika bez oba vodi se kao aspiracija. Kvalifikator obavezan (razvojna,
sigurnosna, politika imenovanja) radi razlikovanja od *politics*. `[stanje |
policy sloj | SBVR; 22]`

**Arhitektura** *(architecture)* — tehničko stanje: temeljna svojstva i
struktura sustava u njegovu okruženju (granice, odgovornosti, ovisnosti,
ugovori). Nije skup odluka (odluke su njezina geneza i opravdanje) i nije
dijagram (dijagram je njezin prikaz). `[stanje | arhitektura | ISO 42010; 1][2]`

**Opis arhitekture** *(architecture description)* — artefakt koji arhitekturu
dokumentira; sadrži poglede, gledišta i obrazloženja. Razlikovati od
arhitekture same (stanja) — po ISO 42010. `[artefakt | metoda | ISO 42010]`

**Kriterij / metoda** *(criterion / method)* — mjerno stanje: čime se
provjerava konformnost ili kvaliteta (registar, protokol, alat s verzijom).
Kriterij je verzioniran odvojeno od predmeta mjerenja; promjena kriterija
mijenja nalaz bez promjene predmeta (konformnost). `[stanje | metoda | 22]`

**Prikaz / pogled** *(view)* — znak: projekcija nekog stanja za određenu
publiku, po deklariranom gledištu (ISO 42010 viewpoint). Generira se iz izvora
istine ili se prema njemu verificira; nikad nije izvor istine. Zastarijeva —
drift prikaza od stanja ista je klasa problema kao drift dokumentacije.
`[znak | metoda | ISO 42010; 38][51]`

**PHILOSOPHY** — najviša instanca kaskade: epistemološki diskurs koji
određuje što vrijedi kao znanje i kako se revidira; kišobran nad politikama,
principima i metodama. `[stanje (doktrinarno) | vrh kaskade | PHILOSOPHY.md]`

**Doktrina** — obvezujući, dokumentirani dio filozofije i njezinih izvedenica;
ono čega se ostala dokumentacija mora pridržavati. `[stanje | vrh kaskade |
PHILOSOPHY.md]`

**Princip** *(principle)* — priznato inženjersko ili znanstveno načelo usvojeno
kao osnova prosudbe (Parnas, SOLID, DRY, Occam …); bira se pod politikama,
opravdava metode. `[stanje | princip sloj | 1][2][29]`

**Kaskada (subordinacija slojeva)** — pravilo da niži sloj (metoda) ne
nadjačava viši (princip, politika, philosophy); prijelaz među stanjima ide
isključivo kroz odluku (DR). `[pravilo | svi slojevi | PHILOSOPHY.md]`

---

# Dio II — Epistemološki i mjerni pojmovi

**Hipoteza** — opovrgljiv oblik doktrinarne tvrdnje: tvrdnja + mjera + što bi
je oborilo. Doktrinarna tvrdnja bez hipoteze deklarira se kao konceptualna.
Aktivne: H1 (drift), H2 (semantička entropija), H3 (valjanost instrumenta),
H4-DQI (prediktivnost kvalitete). `[artefakt | philosophy | PHILOSOPHY.md]`

**Svjedočanstvo** *(evidence)* — dokazni status tvrdnje ili odluke: empirijski
nalaz, mjerenje, incident, standard ili eksplicitna deklaracija aspiracije.
Odsutnost prigovora nije svjedočanstvo. `[svojstvo | sve razine | DR polje]`

**Temelji** *(grounding)* — opravdavajući lanac odluke: informacije →
istraživanje/konzultacije → analiza → zaključak → preporuka. Polje DR zapisa,
komplementarno Svjedočanstvu (Temelji prethode odluci, Svjedočanstvo je prati
i slijedi). `[svojstvo | sve razine | DR polje]`

**Mjera stanja** — opisuje zatečenu strukturu artefakta; artefakt je dovoljan
izvor podataka. `[svojstvo | metoda | 11]`

**Mjera odluke** — procjenjuje isplativost izbora pod neizvjesnošću; traži
razdiobu nad budućim stanjima svijeta. `[svojstvo | metoda | 6]`

**Konformna mjera** — mjera relativna na deklarirani kriterij: kaže „usklađeno
s ovim skupom", ne „dobro po sebi"; promijeni kriterij — promijeni se broj.
Suprotnost: deskriptivna mjera. `[svojstvo | metoda | METHODOLOGY §5.1]`

**Valjanost mjere** — status po protokolu V1–V6: reprezentacijski uvjet, tip
skale, sadržajna pokrivenost, prediktivna valjanost, osjetljivost, erozija.
Mjera bez statusa je tvrdnja na autoritet formule. `[svojstvo | metoda |
15][16][19][22][35]`

**Erozija mjere (Goodhart/Campbell)** — pokazatelj korišten za upravljanje
prestaje mjeriti ono što je mjerio; zato je zadana uporaba mjere
dijagnostička, a upravljačka (gate) eksplicitno ograničena. `[pojava | metoda
| 22]`

**Vektorski nalaz** — rezultat po dimenzijama bez skalarne ukupne ocjene;
obavezan gdje su dimenzije nesumjerljive ili skale nominalne/ordinalne.
`[artefakt | metoda | 16][35]`

**Trojka reproducibilnosti** — (verzija alata, verzija kriterija, verzija
platforme); nalaz bez trojke nije reproducibilan. `[pravilo | metoda |
METHODOLOGY §1 t.2]`

**C-snimka** *(conformance snapshot)* — arhivirani zeleni vektor konformnosti
s trojkom, datumom i popisom deklariranih iznimaka; C0 = prva zelena
konformnost jezgre. `[artefakt | metoda | documentation/conformance/]`

**Semantička entropija** — višeznačnost znakova sustava (isti znak, više
uloga; ista uloga, više znakova); smanjuje se kontroliranim vokabularom.
Nulta točka: jedan TypeVar `T` s četiri semantike. `[pojava | sve razine |
36][38]`

**Slijepa pjega** *(blind spot)* — deklarirano nemjereno: ono što instrument
ili proces po konstrukciji ne vidi (izuzeti modul, strateški protivnik,
latencija signala). Deklarira se, nikad ne prešućuje. `[svojstvo | metoda |
PHILOSOPHY.md; 32]`

---

# Dio III — Artefakti i alati

**FR / NFR** — funkcionalni / nefunkcionalni zahtjev (METHODOLOGY §4).
NFR je **primaran** (izveden iz ciljeva i načela; prethodi odlukama kao
kriterij) ili **izveden** (generiran odlukom). `[artefakt | zahtjevi |
ISO 29148; ISO 25010]`

**Registar (`tools/dictionary.json`)** — strojno čitljivi kontrolirani vokabular
koda: domene, opseg, obitelji baza, uloge TypeVarova; TARGET vokabular čija
odstupanja lint prijavljuje. `[artefakt/kriterij | metoda | NFRQ-ORG-02 | NFRQ-ORG-03]`

**wem_lint** — alat konformnosti u dva izdanja (core: sloj sučelja; workflow:
registar); vektorski izlaz, verzionirani kriterij, izgrađen na frameworku koji
provjerava (samoreferentnost). `[alat | metoda | METHODOLOGY §10]`

**DQI** *(Data Quality Index)* — deklarirani zbirni indeks kvalitete podataka
nad vektorom dimenzija (ISO 25012 / 8000-8); konforman, ne deskriptivan;
primaran je vektor. `[artefakt | metoda | METHODOLOGY §10]`

**Wattleflow Core** — distribucija sučelja dizajn-paterna (čisti ugovori, bez
politika, stdlib-only). **Wattleflow Workflow** — zero-trust rješenje za
podatkovne tokove građeno na Coreu; nosi konkretne politike (concrete/ sloj).
`[artefakt | arhitektura | —]`

**WEM** — Wattleflow inženjerska metodologija (Svezak I = METHODOLOGY.md).
`[artefakt | metoda | —]`

---

# Dio IV — Akronimi

| Akronim | Značenje | Napomena |
|---|---|---|
| ABS/STA/TYP/IMP/SFX/FAC/HDR/EXC | dimenzije wem_lint nalaza (apstraktnost, stanje, tipovi, uvozi, side-effecti, fasada, zaglavlja, izuzeća) | vektor konformnosti |
| AST | *abstract syntax tree* | statička analiza |
| CIA | *confidentiality, integrity, availability* | tri različita grafa prijetnje |
| DR | *decision record* — zapis odluke | zamjenjuje ADR |
| DQI | *data quality index* | v. Dio III |
| EOL | *end of life* (verzija bez sigurnosnih zakrpa) | politika verzija (DR-013) |
| FR / NFR | funkcionalni / nefunkcionalni zahtjev | v. Dio III |
| LSP | *Liskov substitution principle* | princip |
| MDL | *minimum description length* | [13] |
| MRO | *method resolution order* (Python C3) | tehničko |
| NCD | *normalized compression distance* | [14] |
| NMI | *normalized mutual information* | [20]; mjera slaganja particija |
| ARI | *adjusted Rand index* | [21] |
| ORG | kategorija organizacijskih NFR-ova (`FR-ORG-NN`) | FR.md, NFRQ.md |
| PDSA | *plan–do–study–act* (Shewhart/Deming ciklus) | [47][48] |
| SBOM | *software bill of materials* | SPDX/CycloneDX; NFRQ-ORG-06 |
| SBVR | *Semantics of Business Vocabulary and Rules* (OMG) | ontologija zahtjeva |
| WEM | Wattleflow inženjerska metodologija | Svezak I |

Pisanje akronima u identifikatorima koda: **neodlučeno** (DR-WFL-004 pending);
lint upozorava, ne kažnjava, dok odluka ne padne.

---

# Dio V — Razriješene višeznačnosti

Ovdje se bilježe pojmovi koji u literaturi ili kolokvijalnoj upotrebi nose
više značenja; svaka natuknica propisuje kako se pojam koristi *ovdje*.

**„ADR"** — naziv sažima dvije kategorije (arhitektura = stanje; odluka =
čin) i sugerira da je svaka evidentirana odluka arhitektonska — što vlastiti
korpus opovrgava (većina zapisa su policy/metodske odluke). Ovdje: **DR**
(zapis odluke) s poljem `sloj`; „arhitektonska odluka" je DR čiji je sloj
arhitektura-ugovor. Identifikacija „arhitektura jest skup odluka" [45]
odbacuje se u korist: odluke su geneza i opravdanje arhitekture, ne
arhitektura sama.

**„Arhitektura" vs „opis arhitekture" vs „dijagram"** — tri kategorije
(stanje / artefakt / znak), po ISO 42010. Rečenica „arhitektura je zastarjela"
najčešće znači „prikaz je zastario"; ovdje se te tvrdnje ne smiju miješati.

**„Politika"** — ovdje uvijek *policy* (normativno stanje s provedbom i
evaluacijom), nikad *politics*; kvalifikator obavezan.

**„Mjera" vs „metrika" vs „indeks"** — *mjera* je homomorfizam iz empirijskog
u numerički sustav [15]; *metrika* se koristi samo kolokvijalno (bez tvrdnje
o zadovoljenom reprezentacijskom uvjetu); *indeks* je deklarirana agregacija
(npr. DQI) — konforman, ne mjerni rezultat po sebi.

**„Volatilnost"** — pokriva tri različita fenomena: stohastičku (zahtjevi,
egzogena), strukturnu (tehnologija, koncentracija dobavljača) i stratešku
(protivnik koji bira potez nakon arhitektonske odluke [32]); tvrdnje o
volatilnosti moraju imenovati fenomen.

**„Iteracija"** — ovdje: ponavljanje ciklusa *pod kaskadom* s revizijom kroz
DR; ne implicira agilni proces. Povratna petlja je znanstveni instrument
(PDSA nad teorijom [47][48]), ne procesna ideologija.

**„Zero-trust"** — imenovana sigurnosna politika (policy sloj); epistemička
jezgra iz koje slijedi („povjerenje se dokazuje") pripada filozofiji. Ne
koristiti kao naziv za filozofijsku razinu.

**„Modul"** — u Parnasovu smislu granica oko odluke koja se može promijeniti
[1], ne „datoteka" ni „paket"; gdje se misli na Python modul, reći „Python
modul".

**„Konformnost" vs „kvaliteta"** — konformnost je usklađenost s deklariranim
kriterijem (može se postići i lošim kriterijem); kvaliteta traži valjanost
kriterija (V1–V6). Zeleni vektor dokazuje konformnost, ne kvalitetu.

---

# Održavanje

1. **Izvor istine je `dictionary.yaml`, ne ovaj dokument.** Ovo je generirani
   prikaz i prema izvoru se verificira (D-13). Pet kontroliranih vokabulara,
   pet domena (D-12): pojmovi diskursa (`dictionary.yaml`), identifikatori koda
   (`tools/dictionary.json`), reference (`LITERATURE.md`), tvrdnje
   (`POSTULATE.md`), norme (`DOCTRINE.md`). Pojam koji postoji u više domena
   povezuje se, ne duplicira.
2. **Promjene:** natuknica se dodaje, mijenja ili proglašava zastarjelom
   isključivo DR-om; zastarjela natuknica ostaje s oznakom *(zastarjelo — v.
   DR-NNN)* radi čitljivosti starih dokumenata.
3. **Status natuknice:** bez oznake = usvojena; *(privremeno)* = u upotrebi
   bez odluke; *(zastarjelo)* = ne koristiti u novim tekstovima.
4. **Kriterij ulaska:** pojam ulazi kad je (a) nosiv za doktrinu, (b)
   višeznačan u literaturi, ili (c) korišten u ≥2 doktrinarna dokumenta.
   Rječnik nije enciklopedija — definicije domenskih klasa žive u docstringu
   i registru, ovdje samo pojmovi diskursa.
5. **Hipoteza koju hrani:** H2 (semantička entropija) — mjerljivi signal:
   broj pojmova s razriješenom višeznačnošću naspram broja neraz­riješenih
   upotreba u dokumentima (grep po zabranjenim oblicima).