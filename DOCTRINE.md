# Wattleflow doktrina — registar normi

## Uvod

Ovaj dokument je strojno čitljiv i ljudima pristupačan registar normi. Njegova
uloga nije da opisuje istinu, nego da formalizira obvezujuće norme koje se
provode i evaluiraju.

> Norma je preskriptivni iskaz koji obvezuje: nije istinita ni lažna, nego je
> na snazi ili nije. Njezino kršenje mora biti nalaz.

Arhitektura referenciranja je doktrina (D) → postulati (P) → literatura [n].
Proza u [PHILOSOPHY.md](PHILOSOPHY.md) i [METHODOLOGY.md](METHODOLOGY.md)
citira norme D-oznakama i obrazlaže ih; normativni teret nosi ovaj registar.

**Status: registar je na početku životnog ciklusa i namjerno je malen.** Primat
u dokumentiranju ima sloj zahtjeva ([`01-HLRQ/`](01-HLRQ/HLRQ-000-EN.md),
[`02-FRQ/`](02-FRQ/FRQ-000-EN.md), [`03-NFRQ/`](03-NFRQ/NFRQ-000-EN.md)), jer je kod
zreliji od svojeg zapisa. Doktrina se dopunjuje kad taj sloj bude konsolidiran.

**Kriterij ulaska (v0.2).** Članak stoji u registru samo ako ima **stvaran
provedbeni mehanizam** i **stvaran evaluacijski signal** — mehanizam koji se
izvodi i signal koji netko čita. Norma bez oba je aspiracija (D-05) i ne vodi se
ovdje. Osam članaka iz v0.1 palo je na tom testu; popis je u §Povučeni članci.

## Meta-informacije

```yaml
registry: wattleflow-doktrina
registry_version: "0.2.0"   # 0.1.0: kodifikacija zatecenih normi
                            # 0.2.0: povuceni clanci bez mehanizma i signala
source_of_truth: true
statuses: [na-snazi, povucena]
slojevi: [filozofija, metoda]
entry_criteria:
  - provedbeni mehanizam koji se izvodi
  - evaluacijski signal koji se cita
  - clanak se citira izvan ovog registra
blind_spot: >
  Registar nema DR zapis ni u jednoj seriji. Do njega su clanci zatecena norma,
  ne dokazano donesena. Revizija DR serija je odgodena namjerno.
```

## Članci doktrine

### D-02 — Kaskada slojeva

- Sloj: filozofija
- Status: na-snazi

**Iskaz**

Niži sloj ne nadjačava viši: politika se ne uvodi mimo filozofije, metoda mimo
politike. Svako odstupanje je izvod iz višeg sloja ili dokumentirana revizija
tog sloja.

**Opravdanje**

- Postulati: P-14
- Literatura: 44, 45

**Provedba**

Pregled pri izmjeni doktrinarnog teksta; razilaženje se ne rješava u tekstu nego
prijavljuje kao nalaz u [`workflow/TODO.md`](workflow/TODO.md).

**Evaluacija**

Metoda koja propisuje neodlučeno = nalaz. Presedan: akronimsko lint pravilo stoji
na `WARNING` s deklariranim waiverom dok je `DR-WFL-004` otvoren, umjesto da
provede jednu stranu.

### D-03 — Promjena kroz DR

- Sloj: filozofija
- Status: na-snazi

**Iskaz**

Prijelaz stanja (donošenje, izmjena ili ukidanje politike, ugovora, kriterija ili
članka doktrine) ide kroz zapis odluke (DR); prešutna akrecija nije legalan put
promjene.

**Opravdanje**

- Postulati: P-14
- Literatura: 44, 45, 46

**Provedba**

Serije `DR-COR`, `DR-WFL`, `DR-PRC` s indeksima; zapisi u registrima zahtjeva
nose polje *Odluka* koje imenuje pripadni DR.

**Evaluacija**

Izmjena registra bez DR reference = nalaz. **Nije automatizirano** — otvoreni
nalazi se vode u [`workflow/TODO.md`](workflow/TODO.md) (npr. razlamanje registara
2026-08-24 provedeno je bez zapisa).

### D-05 — Tvrdnja bez svjedočanstva je aspiracija

- Sloj: filozofija
- Status: na-snazi

**Iskaz**

Tvrdnja bez svjedočanstva vodi se kao aspiracija i tako se označava; odsutnost
prigovora nije svjedočanstvo. Politika koja ne deklarira provedbeni mehanizam i
evaluacijski signal jednako je aspiracija. Osobna preferencija i autoritet nisu
opravdanje.

**Opravdanje**

- Postulati: P-02

**Provedba**

Polje *Svjedočanstvo* u DR predlošku; oznaka „aspiracija" u policyju i registrima
— danas je nose `POLICY.md` §6.2 (SIEM) i §6.4 (observability).

**Evaluacija**

Tvrdnja navedena kao stanje, bez svjedočanstva = nalaz.

### D-07 — Povjerenje se dokazuje

- Sloj: filozofija
- Status: na-snazi

**Iskaz**

Povjerenje se ne pretpostavlja nego dokazuje. Imenovane sigurnosne politike
(zero-trust i srodne) izvode se iz ovog načela na policy sloju i revidiraju kao
svaka politika.

**Opravdanje**

- Postulati: P-17
- Literatura: 23

**Provedba**

[`NFRQ-SEC-01/02/03`](03-NFRQ/NFRQ-000-EN.md) (nasljednici povučenog `NFRQ-ORG-06`);
`POLICY.md` §7; lint pravila `clean_core_imports` (**ERROR**) i
`distribution_manifest` (`WARNING`).

**Evaluacija**

Modul čiji import-closure izlazi iz tiera vlastite distribucije ruši build.

### D-09 — Vektorska konformnost

- Sloj: metoda
- Status: na-snazi
- Izvor: kodifikacija zatečene prakse; `DR-COR-014` je **predložen**, ne prihvaćen

**Iskaz**

Konformnost se iskazuje vektorski po dimenzijama; skalarna ukupna ocjena ne
postoji. Upravljačka uporaba (gate) dopuštena je samo na binarnim pravilima;
indeksi su isključivo dijagnostički.

**Opravdanje**

- Postulati: P-09, P-10
- Literatura: 16, 22, 35

**Provedba**

`wem_lint` po konstrukciji daje vektor — C-snimka nosi polje `vector`, a nigdje
ukupnu ocjenu.

**Evaluacija**

Pokušaj uvođenja skalara ili gatea na indeksu = nalaz.

### D-10 — Reproducibilnost nalaza

- Sloj: metoda
- Status: na-snazi
- Izvor: kodifikacija zatečene prakse; `DR-COR-014` je **predložen**, ne prihvaćen

**Iskaz**

Nalaz je reproducibilan samo uz trojku (verzija alata, verzija kriterija, verzija
platforme), uz mjereno stablo i pokrenuta pravila. Zelena konformnost arhivira se
kao C-snimka; run s greškama kao finding-vektor, nikad kao C-snimka.

**Opravdanje**

- Postulati: P-10
- Literatura: 22
- Bilješka: empirijska nulta točka — vektori dviju verzija kriterija nad istim
  kodom; incident tihe korozije instrumenta

**Provedba**

[`workflow/conformance/`](workflow/conformance/): svaka snimka nosi
`reproducibility_triple` (`tool`, `criterion`, `platform`) te `source` i
`rules_selected`.

**Evaluacija**

Nalaz bez trojke se ne prihvaća kao nalaz.

### D-11 — Deklariranje slijepe pjege

- Sloj: metoda
- Status: na-snazi

**Iskaz**

Slijepe točke instrumenta i procesa deklariraju se, nikad ne prešućuju; izuzeća
su vidljiva u svakom nalazu, a njihov tihi nestanak je nalaz. Nemjereno se ne
smije čitati kao čisto.

**Opravdanje**

- Postulati: P-18
- Literatura: 31, 32

**Provedba**

Blok `blind_spots` u svakoj C-snimci; sekcije „deklarirana slijepa pjega" u
registrima zahtjeva i u `workflow/TODO.md`.

**Evaluacija**

Deklaracija koja nestane bez zapisa = nalaz. Usporedba deklaracija između dviju
snimki **nije automatizirana** — i to je deklarirana slijepa pjega.

### D-12 — Kontrolirani vokabulari

- Sloj: metoda
- Status: na-snazi

**Iskaz**

Kontrolirani vokabulari obvezuju u svojim domenama: identifikatori koda
(`tools/dictionary.json`), pojmovi diskursa (`dictionary.yaml`), reference
(`LITERATURE.md`, append-only), tvrdnje (`POSTULATE.md`), norme (ovaj registar).
Sukob značenja razrješava DR, ne prešutan izbor u tekstu.

**Opravdanje**

- Postulati: P-08
- Literatura: 38, 51

**Provedba**

Lint čita `tools/dictionary.json` kao kriterij koda (`DR-WFL-020`); ostala četiri
registra nose sekciju *Održavanje*.

**Evaluacija**

Dva primjerka istog registra = nalaz. Presedan: `workflow/hr/LITERATURA.md` je
ključevima 56/57 dodjeljivao druge radove nego korijenski registar i povučen je
2026-08-24, zajedno s `workflow/hr/POSTULATI.md`.

### D-13 — Prikaz nije izvor istine

- Sloj: metoda
- Status: na-snazi

**Iskaz**

Prikaz (dijagram, pogled, indeks, generirani dokument) nikad nije izvor istine:
generira se iz izvora ili se prema njemu verificira, s deklariranim gledištem i
publikom. Brojevi nalaza se ne navode u prozi nego se referira snimka.

**Opravdanje**

- Postulati: P-20, P-08
- Bilješka: ISO/IEC/IEEE 42010 (pogled/gledište) kao pod-metoda

**Provedba**

Smjer `dictionary.yaml` → `DICTIONARY.md`; indeksi registara nose oznaku, iskaz i
poveznicu, a ne prepričavaju detalj.

**Evaluacija**

Ručno održavana kopija izvora = nalaz. Presedani: tablica literature u filozofiji
i brojke lint nalaza u registru zahtjeva — oboje uklonjeno.

---

## Povučeni članci (v0.1 → v0.2, 2026-08-24)

Brojevi se **ne recikliraju**. Povlačenje nije presuda o istinitosti tvrdnje nego
o tome da tvrdnja ne zadovoljava kriterij ulaska iz §Meta-informacije; gdje
sadržaj i dalje živi, naveden je nositelj.

| Bivši | Naslov | Razlog povlačenja | Gdje sadržaj živi |
|---|---|---|---|
| `D-01` | Model odlučivanja | Provedba je bila polje *Temelji*, kojeg u DR predlošku nema; evaluacija („udio DR-ova s popunjenim Temeljima") stoga nemjerljiva. Iskaz je uz to opis stanja, ne norma. | — |
| `D-04` | Iteracija pod kaskadom | Iskaz je konjunkcija `D-02` i `D-03`; vlastitog mehanizma nema. | `D-02`, `D-03` |
| `D-06` | Opovrgljivost tvrdnje | Provedba je bila sekcija *Hipoteze* u `PHILOSOPHY.md`; `H4-DQI` u nju nikad nije upisan, pa mehanizam ne stoji. | `PHILOSOPHY.md` §Hypotheses (praksa, ne norma) |
| `D-08` | Mjera i valjanost | Protokol `V1–V6` je metoda, ne norma; evaluacija se pozivala na `H4-DQI`, kandidata izvan registra. | `METHODOLOGY.md` §5.1, `NFRQ-DEF-02` |
| `D-14` | Evidencija odluka | Ista norma već stoji na metodološkom i policy sloju; ovdje je bila treći primjerak (kršenje `D-13`). Revizija DR forme je odgođena. | `METHODOLOGY.md` §7.1, `POLICY.md` §8 |
| `D-15` | Politika na snazi | Nosivi dio („bez mehanizma i signala = aspiracija") sada je klauzula u `D-05`; ostatak je duplikat. | `D-05` |
| `D-16` | Dokumentacija kao sustav publika | Provedba je bila „matrica publika × artefakt (kandidat)" — artefakt ne postoji. | `PHILOSOPHY.md` (kao teza) |
| `D-17` | Standard kao pod-metoda | Tablica pod-metoda je metodološki artefakt; evaluacija („standard bez zapisa prosudbe = nalaz") nema mjesta na kojem bi se zapis vodio. | `METHODOLOGY.md` §7 |

**Ispravljeno usput:** `D-11` je kao mehanizam navodio `EXC-01` i `STA-03` u
`wem_lint`; tih oznaka u alatu nema — stvarni nositelj je blok `blind_spots` u
C-snimci.

## Bilješke i deklarirane slijepe pjege (D-11)

1. **Registar nema DR zapis.** Ni kodifikacija v0.1 ni ovo povlačenje nemaju
   zapis. Do njega registar vrijedi kao zatečena norma — deklaracija, ne
   svjedočanstvo (D-05). Revizija DR serija **namjerno je odgođena** dok se sloj
   zahtjeva ne konsolidira.
2. **`D-09` i `D-10` stoje na praksi, ne na prihvaćenoj odluci.** `DR-COR-014` je
   u core indeksu *predložen*. Praksa je dokaziva (C-snimke postoje i nose
   trojku), pa članci ostaju — ali izvor je označen kao takav.
3. **Oznake su prefiksirane od 2026-08-24.** Raniji tekst nosio je neprefiksirane
   oznake (`DR-013/014/015/016/018`) iz vremena prije podjele na serije;
   neprefiksirana oznaka nije valjana (`POLICY.md` §8).

## Održavanje

Članak se dodaje, mijenja ili povlači kroz DR (D-03); do otvaranja zapisa izmjena
se vodi kao nalaz. Novi članak ulazi tek uz mehanizam i signal — dok ih nema,
tvrdnja pripada `PHILOSOPHY.md` ili `METHODOLOGY.md`, ne ovamo. Normativni teret
nosi ovaj registar i nigdje se ne prepisuje (D-13).
