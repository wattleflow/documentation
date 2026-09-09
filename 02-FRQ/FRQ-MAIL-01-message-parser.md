# FRQ-MAIL-01 — Čitanje poštanske poruke

> **Oznaka je provizorna, i broj ne referira roditelja.** Obrazac `FR-<KAT>-<NN>.<M>` traži broj
> nadređenog HLRQ-a. Ovaj zahtjev ga nema jer opisuje funkcionalnost koju **dva različita procesa
> koriste s različitim ciljem** (§2). Opcije i njihova cijena: §12.

| | |
|---|---|
| **Status** | Provedeno u kodu, zapisano 2026-08-28 — obrnuto inženjerstvo zatečenog, provjereno nad radnim primjerima |
| **Norme** | [`NFRQ-ORG-09`](../03-NFRQ/NFRQ-ORG-09-external-standard-EN.md) · [`NFRQ-ORG-01`](../03-NFRQ/NFRQ-ORG-01-helper-locality-EN.md) · [`NFRQ-ORG-04`](../03-NFRQ/NFRQ-ORG-04-crosscutting-capability-EN.md) · [`NFRQ-ORG-05`](../03-NFRQ/NFRQ-ORG-05-self-referencing-helpers-EN.md) · [`NFRQ-SEC-03`](../03-NFRQ/NFRQ-SEC-03-supply-chain-locality-EN.md) |
| **Standard** | RFC 5322, uz toleranciju na RFC 2822 §4.3; MIME; RFC 2047, RFC 2231, RFC 5321 §4.4 |

## 1. Opseg

Sustav mora iz **spremnika poruke**, kakav proizvede poštanski klijent, učiniti dostupnim ono
što proces dalje treba: **strane** (tko šalje, tko prima), **vremena** (kada je poslana, kada
isporučena), **čitljiv tekst** poruke, **priložene datoteke** i **identitet** poruke.

Dvije obveze određuju opseg:

1. **Proces ne smije znati u kojem je spremniku poruka stigla.** Isti skup podataka mora biti
   dostupan bez obzira dolazi li poruka kao RFC datoteka ili kao binarni izvoz klijenta.
2. **Proces ne smije određivati što je poruka.** Koja je informacija ciljna razlikuje se od
   procesa do procesa; zahtjev je da su svi podaci koje standard nosi **dostupni**, a ne da su
   unaprijed izabrani za jednog potrošača.

Što se s pročitanim radi — arhiviranje, smanjenje osobnih podataka, anotacija — **nije** predmet
ovog zahtjeva.

## 2. Procesi koji ga koriste

| proces (`examples/workflows/`) | njegov cilj | koju informaciju iz poruke troši |
|---|---|---|
| `06_fetch_emails` | arhivirati izvornik pod imenom po vremenu slanja | vrijeme slanja, predmet |
| `04_pii_reduction` | smanjiti osobne podatke i složiti dokazni snop | tekst, strane, privitci, digest |

Isti zahtjev, dva nadređena procesa — sljedivost je **graf, ne lanac** (`METHODOLOGY.md` §6).

## 3. Akteri

Nijedan nije definiran ovim zahtjevom, a bez svakoga proces ne može početi:

| oznaka | tko / što | unosi u proces |
|---|---|---|
| **A1** | onaj tko pokreće prolaz nad izvorišnom lokacijom | opseg (uzorak, rekurzija, vremenski raspon) i trenutak pokretanja |
| **A2** | poštanski klijent **izvan sustava** | sam spremnik i stupanj njegove usklađenosti sa standardom; sustav na to ne utječe i ne smije pretpostaviti da je potpuna |
| **A3** | čitač binarnog spremnika, vanjska ovisnost izvan povjerljive jezgre | sadržaj poruke koja nikad nije prošla poštanski prijenos |
| **A4** | okvir koji drži stranu izvora | otvaranje i zatvaranje izvora, reviziju pokušaja, jedinstveni razred kvara |

Spremište, dokument i radna ploča **nisu akteri** — ovaj zahtjev ih ne dotiče.

## 4. Trigeri

| oznaka | trigger |
|---|---|
| **EV01** | konfiguracija je učitana i prolaz nad izvorišnom lokacijom je pokrenut |
| **EV02** | A1 predaje spremnik koji je nastao poštanskim prijenosom |
| **EV03** | A1 predaje spremnik koji poštanski prijenos nije prošao (skica, kopija u poslanima) |
| **EV04** | potrošač traži podatak — strane, vrijeme, tekst ili privitke |

## 5. Preduvjeti

1. Spremnik postoji i pripada ovom zahtjevu (prepoznat po **sadržaju**, ne po sufiksu — `BR-04`).
2. Za `EV03`: A3 je dostupan. Njegov izostanak je kvar **prije** obrade, ne usred nje.

## 6. Normalan tok

| korak | radnja |
|---|---|
| 1 | A1 predaje spremnik (`EV02` ili `EV03`) |
| 2 | Sadržaj spremnika se čita; zaglavlja se svode na **jedan** imenovani skup, neovisno o putu |
| 3 | Strane se razlažu na ime i adresu; ponovljene adrese se svode na jednu (`BR-02`) |
| 4 | Vremena se razlažu na trenutak **i podatak je li vremenska zona poznata** (`BR-03`) |
| 5 | Tekst se izlaže čitljivim, bez izvršnog sadržaja (`BR-05`) |
| 6 | Privitci se izlažu sa sadržajem i njegovim otiskom (`BR-06`) |
| 7 | Podaci iz koraka 3–6 računaju se **tek na zahtjev** (`EV04`), ne pri čitanju |

## 7. Alternativni tokovi

| uvjet | ponašanje | ishod |
|---|---|---|
| spremnik ne pripada ovom zahtjevu | preskače se uz zapis | drugi čitač ga preuzima |
| binarni spremnik nosi tuđi sufiks | prepoznaje se po sadržaju i čita kao RFC (`BR-04`) | poruka se ne gubi zbog imena datoteke |
| spremnik bez traga prijenosa (`EV03`) | čitaju se podaci koje je klijent izlučio | strane i vremena su siromašniji, **ne prazni** |
| jedan privitak nema dekodiv sadržaj | izostavlja se, obrada se nastavlja | jedan kvar ne zaustavlja poruku (`BR-06`) |
| vremenska zona nije poznata | bilježi se kao nepoznata (`BR-03`) | usporedba vremena ostaje valjana |
| poruka nosi samo oblikovani tekst | oblikovanje i izvršni sadržaj se uklanjaju (`BR-05`) | tekst je siguran za daljnju obradu |
| spremnik se ne da pročitati | kvar se prijavljuje kao jedinstveni razred (A4) | prolaz se ne prekida na jednoj poruci |

## 8. Poslovna pravila

> **Nisu ratificirana.** `BR-nn` žive u nadređenom HLRQ-u, kojeg ovaj zahtjev nema (§12). Niže su
> pravila **očitana iz zatečenog ponašanja i primjera** — kandidati, ne norma (D-05).

| oznaka | pravilo | zašto |
|---|---|---|
| `BR-01` | Izvornik se nikad ne mijenja | dokazna vrijednost; arhiviranje je preslika, ne ponovni zapis |
| `BR-02` | Ista strana navedena više puta broji se jednom | dvostruko brojanje primatelja iskrivljuje svaku daljnju obradu |
| `BR-03` | Nepoznata vremenska zona ne smije se prikazati kao poznata | miješanje takvih vrijednosti ruši usporedbu unutar arhive |
| `BR-04` | Spremnik se prepoznaje po sadržaju, ne po nazivu datoteke | naziv dodjeljuje klijent (A2) i nije pouzdan |
| `BR-05` | Tekst poruke ne smije nositi izvršni sadržaj | tekst ide dalje u obradu i prikaz |
| `BR-06` | Jedan neispravan privitak ne smije zaustaviti obradu poruke | poruka nosi vrijednost i bez njega |

## 9. Rezultat

Strane, vremena, tekst, privitci i identitet poruke dostupni su procesu **u istom obliku bez
obzira na spremnik**, i mogu se predati dalje kao imenovani podaci uz dokument.

Neuspjeh čitanja jedne poruke ne prekida prolaz.

## 10. Kriteriji prihvaćanja i verifikacija

| | kriterij | verifikacija |
|---|---|---|
| 1 | Oba puta iz §6 daju **istovjetan skup imenovanih podataka** | **izvedeno** 2026-08-28 nad RFC uzorkom i stvarnim binarnim spremnikom |
| 2 | Svaki podatak koji standard nosi dostupan je procesu; ništa nije unaprijed izabrano za jednog potrošača | pregled |
| 3 | Nova ciljna informacija ne traži izmjenu ovog zahtjeva ni njegove izvedbe | pregled; AST pokazatelj **nije automatiziran** (D-11) |
| 4 | Zahtjev ne otvara i ne zatvara izvor | pregled |
| 5 | `BR-03` drži: zona poznata i nepoznata razlikuju se u izlazu | **izvedeno** nad uzorkom s `-0000` |
| 6 | `BR-05` drži: izvršni sadržaj ne dolazi u tekst | **izvedeno** nad uzorkom sa skriptom u tijelu |

**Testova nema** — deklarirana rupa (`DR-WFL-023`). Alternativni tok `EV03` **nije provjeren**:
uzorak spremnika bez traga prijenosa ne postoji.

## 11. Dijagram

[`FRQ-MAIL-01-message-parser.puml`](FRQ-MAIL-01-message-parser.puml) — activity; gledište: put
jedne poruke od spremnika do dostupnih podataka; publika: analitičar i implementator. Slika nije
generirana. Pogled, ne izvor istine (D-13).

## 12. Otvoreno

- **Broj ne referira roditelja.** Cijena po opciji: (a) uvesti HLRQ za obradu pošte i numerirati
  pod njim — veže višekratnu funkcionalnost uz jedan proces, a služi dvama; (b) svrstati uz
  generički sloj — nije generički, nego specijalizacija; (c) priznati da višekratna
  funkcionalnost nema jedan broj roditelja — traži izmjenu obrasca oznake.
- **`BR-01…BR-06` nisu ratificirana** — bez HLRQ-a nemaju gdje živjeti (§8).
- **Zaštitna oznaka u predmetu** ulazi u naziv arhivirane datoteke. Shema nije odlučena.
- **Imenovanje izlazne datoteke** izvedeno je na strani čitanja, a pripada procesu koji zapisuje.
- Kategorija `MAIL` (zaglavlje).

## 13. Sljedivost prema izvedbi

Za dizajn i strukturu klasa vrijedi class diagram, ne ovaj zapis. Ovdje samo poveznica:
`processors/src/wattleflow/helpers/parsers/mail.py`, dosegnuto kroz `IParser` ugovor.
