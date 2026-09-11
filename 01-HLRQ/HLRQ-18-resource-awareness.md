# HLRQ-18 — Svijest workflowa o resursima

> **Oznaka je provizorna** koliko i razred `HLRQ` (`CLAUDE.md` §3.6). Kategorija djeteta `PTN` je u
> vokabularu ([`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md)).

| | |
|---|---|
| **Status** | **Prijedlog — nije provedeno** (2026-09-11). Zatečeno: nijedna komponenta ne zna granice resursa niti prati opterećenje |
| **Odluka** | Nijedan DR nije otvoren. **Odluke autora 2026-09-11:** manager resursa za radnu memoriju, pohranu, CPU i GPU; upozorenje čim opterećenje ili usko grlo postane vidljivo, bez zaustavljanja; osnovna izvedba u `wattleflow-workflow`, GPU s vanjskim knjižnicama u `blackwattle` |
| **Razred** | Zahtjev visoke razine — nosi narativ i poslovna pravila; ne opisuje korake |
| **Distribucija** | **osnovna izvedba** — radna memorija, pohrana, CPU i GPU kad ne traži vanjske knjižnice — u `wattleflow-workflow`, samo standardna knjižnica (`NFRQ-SEC-03`); **GPU s vanjskim knjižnicama** u `blackwattle` |
| **Djeca** | [`FRQ-PTN-18.1`](../02-FRQ/FRQ-PTN-18.1-resource-manager.md) · kandidati: kanal upozorenja prema vanjskom sustavu, učinkovitost po jedinici posla (§4) |
| **Dijagram** | [praćenje resursa](../02-FRQ/FRQ-PTN-18.1-resource-sequence.puml) (sekvencijski) — pogled, ne izvor istine (D-13) |
| **Podloga** | [nastanak više dokumenata](../06-ANALYSIS/2026-09-11-multi-document-creation.md) — mjerenja memorije i vremena · [radni oblik DataFrame](../06-ANALYSIS/2026-09-11-dataframe-working-form.md) |
| **Kaskada** | **P-09** (mjerenje tvrdi samo odnose koje mjera nosi) · **P-10** (pokazatelj korišten za upravljanje prestaje mjeriti — zato je prag upozorenje, a ne vrata) · D-09 · D-11 |
| **Sljedivost** | `NFRQ-OBS-01` · `NFRQ-OBS-03` · `NFRQ-OBS-04` · `NFRQ-DEF-02` · `NFRQ-SEC-02` · `NFRQ-SEC-03` · [`FRQ-MET-01`](../02-FRQ/FRQ-MET-01-metric-collection.md) · [`FRQ-PRC-15.22`](../02-FRQ/FRQ-PRC-15.22-document-flow.md) · [`FRQ-BBD-15.1`](../02-FRQ/FRQ-BBD-15.1-blackboard.md) |

## 1. Narativ

Workflow troši četiri vrste resursa: radnu memoriju, pohranu, procesor i — kad radi s jezičnim
modelima — grafički procesor. Načelo „obradi, pa flushaj u datoteku" drži radnu memoriju na razini
jedne jedinice posla, ali samo dok volumen nije red veličine iznad granice i dok se reference stvarno
otpuštaju; pohrana pritom preuzima teret koji memorija predaje.

**Zašto.** Workflow koji ne zna svoje granice vidi usko grlo tek kao pad procesa, prepunjen disk ili
neobjašnjeno usporenje. Workflow koji pri pokretanju **zna granice**, tijekom izvođenja **prati
opterećenje** i **upozori** korisnika čim usko grlo postane vidljivo pretvara neimenovan kvar u
imenovan nalaz — s resursom, izvorom granice i mjestom u toku.

**Zatečeno.** Ništa od ovoga ne postoji. Workflow drži managere konekcija, drivera i procesora;
resursa nema. Postoji modul metrika u helperima, a `FRQ-MET-01` vodi mjerenje kao neprovedeno — odnos
treba provjeriti.

## 2. Resursi i signal uskog grla

| resurs | granica | signal uskog grla | izvor bez vanjskih knjižnica | izvedba |
|---|---|---|---|---|
| radna memorija | granica kontejnera, inače memorija domaćina | udio granice; OOM događaji kontejnera | cgroup, procfs, `sysconf` | workflow |
| pohrana | slobodan prostor i inodi na putanjama zapisa | udio zauzeća; slobodni inodi | `shutil.disk_usage`, `statvfs` | workflow |
| CPU | kvota kontejnera, inače CPU-i dodijeljeni procesu | **prigušivanje** kontejnera; opterećenje prema dodijeljenim CPU-ima; CPU vrijeme po jedinici posla | cgroup, afinitet, `getloadavg`, `resource` | workflow |
| GPU | memorija uređaja | udio memorije uređaja; iskorištenost | **nema** u standardnoj knjižnici | `blackwattle` |

Izvori i lokalna mjerenja: [`FRQ-PTN-18.1`](../02-FRQ/FRQ-PTN-18.1-resource-manager.md) §1.

## 3. Mjesto u dekompoziciji

| sloj | briga | uloga |
|---|---|---|
| popis pri pokretanju | granica svakog resursa i njezin izvor | manager, uz managere konekcija, drivera i procesora |
| praćenje | uzorak na granici jedinice posla | manager kao promatrač procesora |
| upozorenje | prag ili signal uskog grla → korisnik | audit zapis (`NFRQ-OBS-01`) |
| rasterećenje | flush u datoteku, otpuštanje referenci | ploča i procesor — zatečeno |

## 4. Opseg

| oznaka | predmet | dokument |
|---|---|---|
| `FRQ-PTN-18.1` | manager resursa: popis, praćenje, pragovi, upozorenje | [FRQ-PTN-18.1](../02-FRQ/FRQ-PTN-18.1-resource-manager.md) |
| kandidat | kanal upozorenja prema vanjskom sustavu — SIEM je aspiracija (`CLAUDE.md` §6.2) | — |
| kandidat | učinkovitost: vrijeme po jedinici posla, izvedeno iz para početak/kraj (`FRQ-MET-01`) | — |

## 5. Poslovna pravila

| oznaka | pravilo |
|---|---|
| **BR-01** | Granica **svakog** resursa utvrđuje se **pri pokretanju workflowa**, iz **najužeg** izvora koji platforma daje: kontejner prije domaćina. Izvor se bilježi; granica koja se ne da utvrditi prijavljuje se kao nepoznata, ne pogađa se. |
| **BR-02** | Opterećenje se prati na **granicama jedinice posla** (ciklus, flush), ne po retku ni po bloku (`NFRQ-OBS-03`). |
| **BR-03** | Pragovi su **konfiguracija**, po resursu, izraženi kao udio utvrđene granice. |
| **BR-04** | **Upozorenje čim opterećenje ili usko grlo postane vidljivo** (odluka autora 2026-09-11): prelazak praga ili pojava signala uskog grla, npr. prigušivanje CPU-a. Workflow se **ne zaustavlja** (P-10). |
| **BR-05** | Svaki resurs je **zasebna mjera** troška; memorija i učinkovitost ne spajaju se u ocjenu (D-09). |
| **BR-06** | Rasterećenje vrijedi samo kad se reference **otpuste**: flush u datoteku bez otpuštanja ne smanjuje vrh. |
| **BR-07** | Manager **promatra i izvještava**; tijek workflowa ne mijenja, a vlastiti kvar mjerenja ne ruši workflow. |
| **BR-08** | **Nemjeren resurs prijavljuje se kao nemjeren**, nikad kao slobodan (`CLAUDE.md` §9) — npr. GPU bez proširenja iz `blackwattle`. |

## 6. Nefunkcionalni zahtjevi

| NFR | posljedica za ovu sposobnost |
|---|---|
| `NFRQ-SEC-03` | osnovna izvedba uvozi samo standardnu knjižnicu; vanjske knjižnice za GPU samo u `blackwattle`, uz odgođeni uvoz i test maskiranja |
| `NFRQ-SEC-02` | pragovi i putanje pohrane su deklarirani konfiguracijski ključevi; pokretanje vanjskog alata bila bi nova napadna površina (§7 t.7) |
| `NFRQ-OBS-01` | upozorenje = `WARNING`; `CRITICAL` znači da komponenta ne može nastaviti, a prelazak praga to ne znači |
| `NFRQ-OBS-03` | uzorak i zapis razmjerni jedinicama posla; sažetak jednom po prolazu |
| `NFRQ-OBS-04`, `NFRQ-DEF-02` | svaka mjera nosi tip skale, jedinicu i izvor; dijagnostika, ne vrata |
| memorija, učinkovitost | os kvalitete za njih još nema kategoriju (`DR-WFL-027`, worklist o kvaliteti informacija) |

## 7. Otvoreno

1. **Kategorija osi kvalitete** za memoriju i učinkovitost — mjerljive karakteristike troška resursa.
2. ~~Smije li kritični prag zaustaviti workflow.~~ **Odlučeno 2026-09-11 (autor):** upozorenje čim usko
   grlo postane vidljivo, bez zaustavljanja (`BR-04`).
3. **Prenosivost.** procfs, cgroup i afinitet postoje samo na Linuxu, `statvfs` na Unixu; samo mjerenje
   pohrane radi na svakoj platformi.
4. **Procesor drži zadnji dokument dok ne nastane sljedeći**, pa je vrh oko dva dokumenta i nakon
   flusha (provjereno čitanjem koda, nije izmjereno). Izmjena temeljne petlje traži `DR-WFL`.
5. **Volumen reda veličine iznad granice** traži komadanje, tok ili spremište sesije — izvan opsega.
6. **Granica po workflowu ili po procesoru.**
7. **GPU preko vanjskog alata.** Alat naredbenog retka proizvođača, pozvan kao pod-proces, ne mijenja
   uvoz (ostaje standardna knjižnica), ali ovisi o binarnoj datoteci izvan distribucije. Je li to
   „vanjska knjižnica" po odluci o smještaju — otvoreno.
8. **Broj CPU-a dostupnih procesu** daje standardna knjižnica tek od Pythona 3.13; pin je 3.11, pa do
   tada vrijede afinitet i kvota kontejnera.
9. **Trag u OSCAL sloju:** ASD ISM `ism-2091` — *„Resource limits are enforced for artificial
   intelligence models."* Preslikavanje na ovu sposobnost nije napravljeno.
