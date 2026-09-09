# FRQ-AUD-01 — Put audit zapisa od komponente do spremišta

> **Oznaka je provizorna.** Kategorija `AUD` nije u vokabularu FR registra; uvođenje traži DR
> (D-12). Audit je *cross-cutting* sposobnost, ne ontološki primitiv, pa pripada osi sposobnosti
> i nema nadređeni HLRQ ([`DR-WFL-022`](../04-DR/DR-WFL-022-class-role-categories.md) t.4).

| | |
|---|---|
| **Status** | Provedeno u kodu, zapisano 2026-08-28 — obrnuto inženjerstvo zatečenog |
| **Predmet** | Obitelj `logger` temeljnog sloja: sabirnik zapisa (`ILogger` + `IObserver`), asinkroni handler, kontekstni filtar, tablica formata |
| **Izvedba** | `workflow/src/wattleflow/helpers/audit.py` (283 linije, `wattleflow-workflow` v0.0.1.11) |
| **Sljedivost** | `NFRQ-OBS-01/02/03`, `NFRQ-SEC-06`, `NFRQ-ORG-07` · `DR-WFL-008`, `DR-WFL-018`, `DR-WFL-021` · D-07 |
| **Sestrinski** | `NFRQ-OBS-02` (ključevi zapisa) · `DR-WFL-021` (gdje na pozivnom lancu zapis nastaje) |

## 1. Predmet

Zapis nastaje u komponenti kao **par**: događaj i imenovana polja. Ovaj zahtjev opisuje **put**
tog para do spremišta — što ga sastavlja, tko ga prima i u kojem obliku. *Što* zapis nosi uređuje
`NFRQ-OBS-02`, *gdje na pozivnom lancu* nastaje `DR-WFL-021`.

Sabirnik je terminalna karika kooperativnog `__init__` lanca: troši logging argumente i guta
ostatak, pa do `object.__init__` ne stiže ništa.

## 2. Sudionici

| uloga | što radi | zatečeno |
|---|---|---|
| **emitent** | potomak temeljne klase; poziva `debug` · `info` · `warning` · `error` · `critical` · `fatal` · `exception` s imenovanim poljima | da |
| **sabirnik** | razdvaja logging argumente od podatkovnih, serijalizira podatkovna u tekst, predaje stdlib loggeru | da |
| **handler** | formatira i upisuje u spremište | da — **jedan**, zadani stream handler |

Spremište nije sudionik koji išta pokreće.

## 3. Put zapisa

1. Emitent poziva razinu s imenovanim poljima; `stacklevel` se postavlja na 3 da zapis pokaže
   pozivatelja, ne sabirnik.
2. Sabirnik odvaja **kontrolne** argumente (`exc_info`, `stack_info`, `stacklevel`, `extra`) od
   **podatkovnih**. Kontrolni idu stdlib pozivu, podatkovni u tekst.
3. Podatkovna polja se serijaliziraju po vrijednosti:
   - skalar i `None` — doslovno `ključ=vrijednost`;
   - zbirka — **samo na razini INFO** skraćena na oznaku tipa i duljinu; na ostalim razinama ide
     `repr`;
   - ostalo — `repr` odrezan na 100 znakova.
4. Serijalizirana polja se **konkateniraju u poruku**. Iza ove točke pojedinačno polje više ne
   postoji — zapis je rečenica.
5. Stdlib logger predaje zapis svojim handlerima; handler formatira i upisuje.

**Rasprostiranje.** Logger je jedan **po klasi** (`getLogger(type(self).__name__)`), ne po
instanci. Zadani handler se gradi jednom po klasi, inače bi svaka instanca umnožila zapis.
Razina se primjenjuje kad god stigne eksplicitno — **zadnji eksplicitni pisac pobjeđuje** — jer
je logger dijeljen; izostavljena razina ne resetira tuđu postavku.

**Detekcija okvirnih objekata.** Zbirke nalik `DataFrame`u prepoznaju se pretragom `__mro__`, ne
`hasattr`om na instanci: sonda na živom objektu pokrenula bi `__getattr__` kuku — rdflib bi
emitirao upozorenje, lijeni proxy otvorio vezu samo da bi bio zapisan.

**Drugi ulaz.** Sabirnik je i promatrač: promatrani događaj ulazi u isti put kao INFO zapis s
`msg=Notify` i događajem kao imenovanim poljem. To je točka na koju bi se priključilo
prosljeđivanje vanjskim sustavima (`CLAUDE.md` §6.2).

## 4. Kriteriji prihvaćanja

| | kriterij | stanje |
|---|---|---|
| 1 | Ulazna površina ne postaje zapisna: `**kwargs` se ne prosljeđuje u logging poziv (`NFRQ-SEC-06` k.1) | mjereno, uz granicu populacije (§5) |
| 2 | Zapis pokazuje pozivatelja, ne sabirnik | drži `stacklevel` |
| 3 | Jedna instanca ne umnožava zapise svoje klase | drži gradnja handlera jednom po klasi |
| 4 | Serijalizacija ne dira instancu podatkovnog objekta | drži MRO sonda |
| 5 | Neuspjeh upisa ne prekida poslovni tok | drži stdlib `handleError`, **ne ovaj kod** |

## 5. Verifikacija

Kriterij 1 — `wem_lint --select OBS-02`, nalaz `audit-kwargs-splat`. Trojka (D-10): alat
`wem_lint 1.17.0` · kriterij `tools/dictionary.json` po distribuciji · platforma
`python 3.11.15 (Linux)` · stablo `src/wattleflow` po distribuciji · pravilo `OBS-02`.

> **Populacija pravila uža je od iskaza kriterija (D-11).** Pravilo prepoznaje samo pozive s
> primateljem `self`; zapis preko modulskog loggera ili tuđe instance nije u populaciji. Dva
> takva mjesta postoje — `workflow/src/wattleflow/concrete/workflow.py:185` i
> `processors/src/wattleflow/decorators/oscal/policy.py:60`. Zeleni vektor ne dokazuje nulu iz
> iskaza kriterija. Proširenje populacije mijenja kriterij i traži DR.

Kriteriji 2–5 nemaju test (`DR-WFL-023`); kriterij 5 posebno, jer ga danas drži stdlib
svojstvo koje ništa ne bi primijetilo da otpadne. Deklarirana rupa.

Run nije C-snimka: `OBS-01/02/03` još mjere po `DR-WFL-018`, ne po `DR-WFL-021`.

## 6. Nalazi (D-11)

| nalaz | posljedica |
|---|---|
| Sučelje loggera nasljeđuje *observable* ugovor, ali je pretplata promatrača `NotImplementedError` | deklarirani ugovor je inertan; emitirajuća strana radi, pretplatna ne. Mijenja core sučelje → `DR-COR` |
| Točka pretplate handlera nema **nijedno** pozivno mjesto u sva tri stabla; isto vrijedi za asinkroni handler | put s više odredišta je napisan, ali neizvršen |
| Kontekstni filtar čita polja koja **nitko ne postavlja**; format koji ih koristi nema potrošača | filtar je no-op, a taj format bi pao na nedostajućem polju |
| Članovi tablice formata miješaju `UPPER_SNAKE` i `PascalCase` | odstupanje od §2.3 |
| Skraćivanje zbirki vrijedi samo na INFO razini | ponašanje volumena; pripada `NFRQ-OBS-03`, ondje nije zapisano |

## 7. Aspiracija — put s više odredišta (D-05)

Framework predviđa više handlera prema spremištima različitog povjerenja (datoteka, Kafka, NiFi,
`Repository`), pri čemu bi zapis stizao **strukturiran**, svako polje klasificirano
(javno · identitet · mjera · klasificirano), a handler propuštao samo ono što mu deklarirana
sposobnost redakcije pokriva; neuspjeh upisa bio bi događaj za monitoring, a zapis nastao unutar
handlera ne bi se vraćao u put.

**Ništa od toga nije provedeno, i korak 4 iz §3 je razlog:** konkatenacija u rečenicu događa se
prije handlera, pa klasifikacija i redakcija po polju nisu izvedive (`DR-WFL-008` §Cijena). Ovo
nisu tri odvojena posla nego jedan — dok zapis do handlera ne stigne strukturiran, ostali koraci
nemaju predmet.

Dijagram: [`FRQ-AUD-01-audit-record-path.puml`](FRQ-AUD-01-audit-record-path.puml) — activity,
gledište: **ciljni** put; publika: programer i arhitekt. Slika nije generirana. Pogled, ne izvor
istine (D-13).

## 8. Otvoreno

- Kategorija `AUD` traži DR (zaglavlje).
- Klasifikacijska ljestvica iz §7 nije usvojena — kandidat za `NFRQ-SEC-06` ili zaseban DR.
- Čime handler **dokazuje** sposobnost (deklaracija vs provjera) — Q5c u analizi *CIA, audit i
  OSCAL* (2026-08-12); izvor nije u repozitoriju (D-11).
- Inertni observable ugovor i populacija pravila k.1 — vidi §5 i §6.
