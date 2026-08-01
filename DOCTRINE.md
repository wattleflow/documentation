# Wattleflow doktrina — registar normi

## Uvod

Ovaj dokument je strojno čitljiv i ljudima pristupačan registar normi. Njegova
uloga nije da opisuje istinu, nego da formalizira obvezujuće norme koje se
provode i evaluiraju kroz odluke (DR), testove i revizije.

> Norma je preskriptivni iskaz koji obvezuje: nije istinita ni lažna, nego je
> na snazi ili nije. Njegovo kršenje mora biti nalaz.

Arhitektura referenciranja je doktrina (D) → postulati (P) → literatura [n].
Opravdanje navodi P-oznake i/ili reference literature kada postulati ne postoje.
Proza u [PHILOSOPHY.md](PHILOSOPHY.md) i [METHODOLOGY.md](METHODOLOGY.md)
citira norme D-oznakama i obrazlaže ih; normativni teret nosi ovaj registar.

Upravljanje je jednostavno: svaki članak se dodaje, mijenja, suspendira ili
ukida isključivo kroz DR.

## Meta-informacije

```yaml
registry: wattleflow-doktrina
registry_version: "0.1.0"
source_of_truth: true
statuses: [na-snazi, suspendirana, ukinuta]
slojevi: [filozofija, policy, princip, metoda]
enactment: >
  v0.1 je kodifikacija zatečenih normi iz PHILOSOPHY.md v0.4.1 i
  METHODOLOGY.md v0.3.1 — ne uvodi nove obveze. Formalno donošenje cijelog
  registra: DR-018 (kandidat). Pojedini članci navode raniji DR gdje je
  norma već bila donesena zasebno.
```

## Članci doktrine

### D-01 — Model odlučivanja

- Sloj: filozofija
- Status: na-snazi
- Izvor: DR-018 (kandidat)

**Iskaz**

Model odlučivanja je deklariran i verzioniran. Trenutno: jedan autor, vezan
istim epistemičkim standardom kao svaka odluka (Temelji, Svjedočanstvo —
autoritet autora nije opravdanje). Prijelaz na konsenzusni model zajednice
provodi se dokumentiranom revizijom ovog članka (DR), ne prešutno.

**Opravdanje**

- Postulati: P-02
- Literatura: 56, 57
- Bilješka: Evolucijski put ima dokumentirane presedane koji se tada usvajaju
  kao pod-metode: PEP 13 (BDFL → upravljačko vijeće, proveden kroz dokumentiranu
  odluku core developera, prosinac 2018.) [57] i IETF rough consensus (konsenzus
  kao odsustvo neadresiranih prigovora, ne preglasavanje — RFC 7282) [56],
  srodan standardu svjedočanstva.

**Provedba**

DR proces; polje Temelji obvezno i za autorove odluke.

**Evaluacija**

Udio DR-ova s popunjenim Temeljima; obvezna revizija članka pri prvom vanjskom
suradniku.

### D-02 — Kaskada slojeva

- Sloj: filozofija
- Status: na-snazi
- Izvor: DR-018 (kandidat)

**Iskaz**

Niži sloj ne nadjačava viši: politika se ne uvodi mimo filozofije, princip mimo
politike, metoda mimo principa. Svako odstupanje je izvod iz višeg sloja ili
dokumentirana revizija tog sloja.

**Opravdanje**

- Postulati: P-14
- Literatura: 44, 45

**Provedba**

DR proces; revizija dokumenata pri sintezama (Bilješke o sintezi).

**Evaluacija**

Nalaz: metoda koja propisuje neodlučeno (presedan: akronimsko lint pravilo,
suspendirano do DR-015).

### D-03 — Promjena kroz DR

- Sloj: filozofija
- Status: na-snazi
- Izvor: DR-018 (kandidat)

**Iskaz**

Prijelaz stanja (donošenje, izmjena, ukidanje politike, ugovora, kriterija ili
članka doktrine) ide isključivo kroz zapis odluke (DR); prešutna akrecija nije
legalan put promjene.

**Opravdanje**

- Postulati: P-14
- Literatura: 44, 45, 46

**Provedba**

DR proces; registri odbijaju izmjene bez DR reference (Održavanje sekcije).

**Evaluacija**

Broj izmjena registara bez DR reference (cilj: nula).

### D-04 — Iteracija pod kaskadom

- Sloj: filozofija
- Status: na-snazi
- Izvor: DR-018 (kandidat)

**Iskaz**

Iteracija (uključujući arhitektura ↔ implementacija) dopuštena je isključivo pod
kaskadom: implementacijsko iskustvo revidira više slojeve samo kroz DR.
Povratna petlja vrti se nad teorijom (hipoteza → mjerenje → zaključak), ne nad
popisom posla.

**Opravdanje**

- Postulati: P-01, P-15
- Literatura: 47, 48

**Provedba**

DR proces; motivacijska petlja [PHILOSOPHY.md](PHILOSOPHY.md).

**Evaluacija**

Arhitektonske promjene bez pripadnog DR-a = nalaz (H1 srodno).

### D-05 — Tvrdnje bez svjedočanstva

- Sloj: filozofija
- Status: na-snazi
- Izvor: DR-018 (kandidat)

**Iskaz**

Tvrdnja bez svjedočanstva vodi se kao aspiracija i tako se označava; odsutnost
prigovora nije svjedočanstvo. Osobna preferencija i autoritet nisu opravdanje.

**Opravdanje**

- Postulati: P-02

**Provedba**

Polja Svjedočanstvo/Temelji u DR; oznaka aspiracije u registrima.

**Evaluacija**

Udio odluka i članaka s deklariranim statusom dokaza.

### D-06 — Opovrgljivost tvrdnje

- Sloj: filozofija
- Status: na-snazi
- Izvor: DR-018 (kandidat)

**Iskaz**

Nosiva doktrinarna tvrdnja ima opovrgljiv oblik (hipotezu: mjera + što bi je
oborilo) ili eksplicitnu deklaraciju da je konceptualna.

**Opravdanje**

- Postulati: P-02
- Bilješka: operacionalizacija kritičkog razmišljanja na samu doktrinu

**Provedba**

Sekcija Hipoteze u [PHILOSOPHY.md](PHILOSOPHY.md); pravilo ulaska novih tvrdnji.

**Evaluacija**

Tvrdnje bez hipoteze i bez deklaracije = nalaz.

### D-07 — Povjerenje se dokazuje

- Sloj: filozofija
- Status: na-snazi
- Izvor: DR-018 (kandidat)

**Iskaz**

Povjerenje se ne pretpostavlja nego dokazuje. Imenovane sigurnosne politike
(zero-trust i srodne) izvode se iz ovog načela na policy sloju i revidiraju kao
svaka politika.

**Opravdanje**

- Postulati: P-17
- Literatura: 23

**Provedba**

Policy sloj (NFR-ORG-06 i srodni); clean-core i granice povjerenja u registru.

**Evaluacija**

Ovisnosti bez deklariranog čvora povjerenja = nalaz.

### D-08 — Mjera i valjanost

- Sloj: metoda
- Status: na-snazi
- Izvor: DR-018 (kandidat)

**Iskaz**

Mjera ne ulazi u metodologiju bez statusa po protokolu valjanosti V1–V6;
minimalno su obvezni tip skale i dopuštene agregacije (V2), sadržajna
pokrivenost (V3) i granica dijagnostičko/upravljačko (V6). Do prolaska V1 i V4
mjera se iskazuje kao konformna, ne deskriptivna.

**Opravdanje**

- Postulati: P-09
- Literatura: 15, 16, 17, 18, 19

**Provedba**

[METHODOLOGY.md](METHODOLOGY.md) §3.1/§6.1; redak Valjanost mjere u anatomiji.

**Evaluacija**

Mjere u upotrebi bez statusa = nalaz; H4-DQI kao prvi test.

### D-09 — Vektorska konformnost

- Sloj: metoda
- Status: na-snazi
- Izvor: DR-014; DR-018 (kodifikacija)

**Iskaz**

Konformnost se iskazuje vektorski po dimenzijama; skalarna ukupna ocjena ne
postoji. Upravljačka uporaba (gate) dopuštena je samo na binarnim pravilima;
indeksi su isključivo dijagnostički.

**Opravdanje**

- Postulati: P-09, P-10
- Literatura: 16, 22, 35

**Provedba**

wem_lint (vektorski izlaz po konstrukciji); DQI (vektor primaran).

**Evaluacija**

Pokušaj uvođenja skalara ili gatea na indeksu = nalaz; H3 signal.

### D-10 — Reproducibilnost nalaza

- Sloj: metoda
- Status: na-snazi
- Izvor: DR-014

**Iskaz**

Nalaz je reproducibilan samo uz trojku (verzija alata, verzija kriterija,
verzija platforme); zelene konformnosti arhiviraju se kao C-snimke s trojkom,
datumom i popisom deklariranih iznimaka.

**Opravdanje**

- Postulati: P-10
- Literatura: 22
- Bilješka: empirijska nulta točka: vektori 0.2.0 i 0.3.0 nad istim kodom;
  ABS-03 incident

**Provedba**

wem_lint changelog konvencija; docs/conformance/.

**Evaluacija**

Nalaz bez trojke = nereproducibilan = nalaz; H3.

### D-11 — Deklariranje slijepe pjege

- Sloj: metoda
- Status: na-snazi
- Izvor: DR-018 (kandidat)

**Iskaz**

Slijepe točke instrumenta i procesa deklariraju se, nikad ne presućuju;
izuzeća su vidljiva u svakom nalazu, a njihov nestanak bez DR-a je nalaz.

**Opravdanje**

- Postulati: P-18
- Literatura: 31, 32

**Provedba**

EXC-01 i STA-03 mehanizmi u wem_lint; blind_spots sekcije registara.

**Evaluacija**

Tiho nestala deklarirana iznimka = nalaz.

### D-12 — Kontrolirani vokabulari

- Sloj: metoda
- Status: na-snazi
- Izvor: DR-018 (kandidat)

**Iskaz**

Kontrolirani vokabulari obvezuju u svojim domenama: identifikatori koda
(naming_registry), pojmovi diskursa (rječnik), reference (literatura,
append-only), tvrdnje (postulati), norme (doktrina). Sukob značenja razrješava
se DR-om, ne prešutnim izborom u tekstu.

**Opravdanje**

- Postulati: P-08
- Literatura: 38, 51

**Provedba**

Pet registara s Održavanje sekcijama; lint (kod); H2 signal (diskurs).

**Evaluacija**

Trend povreda vokabulara po verziji (H2).

### D-13 — Prikaz nije izvor istine

- Sloj: metoda
- Status: na-snazi
- Izvor: DR-018 (kandidat)

**Iskaz**

Prikaz (dijagram, pogled, generirani dokument) nikad nije izvor istine:
generira se iz izvora ili se prema njemu verificira, s deklariranim gledištem i
publikom.

**Opravdanje**

- Postulati: P-20, P-08
- Bilješka: ISO/IEC/IEEE 42010 (pogled/gledište) kao pod-metoda

**Provedba**

Rječnik.yaml → RJECNIK.md smjer; budući generirani prikazi doktrine.

**Evaluacija**

Ručno održavana kopija izvora = nalaz (presedan: tablica literature u
filozofiji, uklonjena).

### D-14 — Evidencija odluka

- Sloj: filozofija
- Status: na-snazi
- Izvor: DR-016 (nazivlje); DR-018 (kodifikacija funkcije)

**Iskaz**

Evidencija odluka je obvezna funkcija (sljedivost, alternative, cijena,
Temelji, Svjedočanstvo, revizibilna povijest uključujući povučene odluke);
format zapisa (DR) je zamjenjiva pod-metoda i revidira se dokazima.

**Opravdanje**

- Postulati: P-14, P-16
- Literatura: 44, 45, 46

**Provedba**

DR predložak; [METHODOLOGY.md](METHODOLOGY.md) §8.1 (funkcijski kriteriji
nasljednika).

**Evaluacija**

Odluke bez zapisa = nalaz; kandidat nasljednik: graf odluka (RDF/PROV-O).

### D-15 — Politika na snazi

- Sloj: policy
- Status: na-snazi
- Izvor: DR-013 (presedan); DR-018 (kodifikacija)

**Iskaz**

Politika na snazi deklarira provedbeni mehanizam i evaluacijski signal;
politika bez oba vodi se kao aspiracija. Politika se donosi opravdavajućim
lancem (informacije → analiza → zaključak → preporuka → odluka) i evaluira u
petlji.

**Opravdanje**

- Postulati: P-01, P-10
- Literatura: 22
- Bilješka: presedan potpunog ciklusa: politika verzija Pythona (DR-013)

**Provedba**

DR polja Temelji + Registar; provedba kroz kriterije/alate.

**Evaluacija**

Politike bez evaluacijskog signala = aspiracije (popis se održava).

### D-16 — Dokumentacija kao sustav publika

- Sloj: princip
- Status: na-snazi
- Izvor: DR-018 (kandidat)

**Iskaz**

Dokumentacija je sustav s više publika; svaka publika ima svoj artefakt, a
jedan format za sve publike nije legalan cilj.

**Opravdanje**

- Postulati: P-05, P-07, P-16
- Literatura: 30, 39, 53

**Provedba**

Matrica publika × artefakt (kandidat); postojeća podjela registara i svezaka.

**Evaluacija**

Publika bez artefakta = deklarirana rupa, ne presućena.

### D-17 — Standard kao pod-metoda

- Sloj: princip
- Status: na-snazi
- Izvor: DR-018 (kandidat)

**Iskaz**

Standard se usvaja kao pod-metoda kroz epistemičku prosudbu i bilježi se;
usvajanje po inerciji nije legalno. Granice se spajaju kompozicijom (ugovor +
provjera na granici), ne konkatenacijom.

**Opravdanje**

- Postulati: P-21, P-03, P-17
- Bilješka: Occam/DRY za pod-metode; registar + lint kao mehanizam kompozicije

**Provedba**

[METHODOLOGY.md](METHODOLOGY.md) §8 tablica pod-metoda; clean-core filtar;
obitelji baza u registru.

**Evaluacija**

Ovisnost ili standard bez zapisa prosudbe = nalaz.