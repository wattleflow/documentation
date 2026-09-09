# FRQ-OSCAL-14.11 — Provjera komponente prema baselineu

> **Kategorija `OSCAL` je u vokabularu** — [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md)
> (2026-08-21). Oznaka se od tada mijenja kroz DR, ne uređivanjem.

| | |
|---|---|
| **Status** | Prijedlog (2026-08-21) — obrnuto inženjerstvo zatečenog koda |
| **Odluka** | [`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md) — kategorija i broj sposobnosti; sam zahtjev nema vlastiti DR |
| **Nadređeni zahtjev** | [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) — narativ sposobnosti i poslovna pravila `BR-OSCAL-01…BR-OSCAL-12` |
| **Predmet** | `OSCALPolicy`, `OSCALPolicyError` |
| **Sestrinski** | [`14.4`](FRQ-OSCAL-14.4-profile.md) profil · [`14.10`](FRQ-OSCAL-14.10-crosswalk.md) crosswalk |
| **Izvedba** | `src/wattleflow/oscal/policy.py`; potrošači: `wattleflow.decorators.oscal` (`@oscal_connection`, `@oscal_driver`, `@oscal_processor`) |

## 1. Predmet

Policy je **vrata**: komponenta deklarira koje kontrole zadovoljava, a gate provjerava da svaka
deklarirana kontrola pripada aktivnom baselineu. Semantika je `declared ⊆ baseline` — ništa više
i ništa manje.

To što gate provjerava **nije** je li komponenta sigurna, nego govori li o kontrolama koje su u
opsegu. Deklaracija izvan baselinea znači da komponenta tvrdi nešto što baseline ne traži — a to
je ili kriva deklaracija ili krivi baseline.

## 2. Akteri

| oznaka | što je | ulazi u proces s |
|---|---|---|
| **A1** | `Profile` — aktivni baseline | `control_ids()` |
| **A2** | `Crosswalk` (opcionalan) | prijevod taksonomije |
| **A3** | Komponenta (konekcija, driver, procesor) | `OSCAL_CONTROLS` ClassVar |
| **A4** | Dekorater iz `wattleflow.decorators.oscal` | poziva `verify` |

## 3. Trigeri (događaji `EVnn`)

| oznaka | trigger |
|---|---|
| **EV01** | bootstrap gradi `OSCALPolicy(profile[, crosswalk])` |
| **EV02** | A4 pozove `verify(component_name, declared)` |

## 4. Preduvjeti

1. Profil je učitan i bira barem jednu kontrolu.
2. Ako komponenta deklarira u stranoj taksonomiji, crosswalk je predan pri gradnji policyja.

## 5. Normalan tok

1. EV01 — baseline se **zamrzava** u `frozenset` pri konstrukciji; kasnija izmjena profila
   (nemoguća — profil je nepromjenjiv) ne bi ga ni dosegla.
2. EV02 — deklarirani `id`-evi se prvo prevode (ako crosswalk postoji), inače uzimaju kakvi jesu.
3. Razlika `translated − required` je skup prekršaja. Prazna → prolaz (bez povratne vrijednosti).
4. Neprazna → `OSCALPolicyError` s imenom komponente, `uuid`-om baselinea i **sortiranim** popisom
   spornih `id`-eva.
5. Put od komponente do vrata vodi dekorater (`wattleflow.decorators.oscal`, distribucija
   `wattleflow-workflow`): omata `__init__`, izvuče `oscal_policy` iz kwargsa, razriješi
   `OSCAL_CONTROLS` kroz MRO i zamijeni `_fsm` čuvanim FSM-om. Vrata se otvaraju **pri prvom
   prijelazu**, ne pri uvozu ni pri konstrukciji. `BR-OSCAL-11` traži da taj dekorater nosi
   **svaka** komponenta koja deklarira kontrole.

## 6. Alternativni tokovi

| uvjet | ponašanje |
|---|---|
| deklaracija ⊆ baseline | prolaz, tiho |
| deklaracija sadrži `id` izvan baselinea | `OSCALPolicyError: <komponenta> declares controls outside baseline <uuid>: [<id>…]` |
| NIST deklaracija bez crosswalka | pada — `ac-3` nije ISM `id` |
| ista deklaracija s crosswalkom | prolazi ako su prijevodi u baselineu |
| prijevod izvan baselinea (`sc-8`, `sc-13` protiv E8) | pada, s **prevedenim** `id`-evima u poruci |
| **prazna deklaracija** | prolazi — vidi §11 t.1 |
| dekorirana komponenta, `oscal_policy` kwarg izostane | `OSCALPolicyError: … oscal_policy kwarg is required …`; prijelaz se ne dogodi — vidi §11 t.4 |
| dekorirana komponenta bez `_fsm` | dekorater je no-op; vrata se nikad ne otvore |
| podklasa s užom deklaracijom | `controls_strategy="merge"` (zadano) **unira** roditeljske kontrole — sužavanje deklaracije nasljeđivanjem ne radi |

## 7. Rezultat

Komponenta koja tvrdi nešto izvan opsega ne prolazi gate, a poruka nosi sve troje: tko, protiv
kojeg baselinea, i koji `id`-evi. `OSCALPolicyError` nasljeđuje `PermissionError` — odbijanje je
razred prava pristupa, ne razred podatkovne greške.

## 8. Kriteriji prihvaćanja

1. Baseline je `frozenset` iz `profile.control_ids()`, zamrznut pri konstrukciji. ✅
2. `verify` prolazi tiho kad je `declared ⊆ baseline`. ✅
3. Prekršaj daje `OSCALPolicyError` s imenom komponente, `uuid`-om i sortiranim popisom. ✅
4. Crosswalk se primjenjuje **prije** usporedbe. ✅
5. `OSCALPolicyError` je `PermissionError`. ✅
6. Policy ne drži stanje osim profila, crosswalka i zamrznutog skupa (`__slots__`). ✅
7. **Svaka komponenta koja deklarira `OSCAL_CONTROLS` je pod vratima** (`BR-OSCAL-11`) — kroz
   nasljeđenu bazu, ne kroz vlastiti dekorater ([`14.13`](FRQ-OSCAL-14.13-component-bases.md)). ✅
8. **Dekorater se ne ponavlja na klasi čiji je predak dekoriran** — vrata se nasljeđuju. ✅
9. **Provjera prethodi konstrukciji** — neusklađena komponenta ne izvrši ništa iz svojeg
   `__init__` ([`DR-WFL-014`](../04-DR/DR-WFL-014-oscal-gate-before-construction.md)). ✅
10. Komponenta pod vratima **dobiva aktivnu politiku** pri konstrukciji. ❌ — nijedno mjesto ne
    predaje `oscal_policy=`; vrata su zato `strict=False` i inertna (§11 t.5)

## 9. Verifikacija

| kriterij | metoda | rezultat (2026-08-21) |
|---|---|---|
| 1 | ML1 profil | `required` je `frozenset` od 46 `id`-eva; `profile_uuid` = `2d849506-…` |
| 2 | prve tri kontrole iz profila | prolaz, bez iznimke |
| 3 | `verify("comp", ["ism-9999999"])` | `OSCALPolicyError: comp declares controls outside baseline 2d849506-…: ['ism-9999999']` |
| 4 | `["ac-3","ia-5"]` bez i s crosswalkom | bez → `OSCALPolicyError`; s → prolaz |
| 4 | `["sc-8","sc-13"]` s crosswalkom | pada s **prevedenim** `['ism-0469', 'ism-1080']` |
| 5 | `issubclass(OSCALPolicyError, PermissionError)` | `True` |
| 6 | `OSCALPolicy.__slots__` | `('_profile', '_crosswalk', '_required')` |
| 7 | uvoz svake klase koja deklarira ili je dekorirana, uz podmetnute stubove third-party knjižnica (`verify_gates.py`) | 18 klasa, **0 grešaka**; 15 pod vratima **kroz bazu**, 3 baze nose vlastiti dekorater; svugdje `strict=False` |
| 8 | isti alat, `__oscal_meta__` u `__dict__` klase vs naslijeđen | nijedna konkretna klasa nema vlastiti, sve su pod vratima |
| 9 | neusklađena konekcija nad stvarnom `OSCALConnection` (`base_e2e.py`) | `OSCALPolicyError` u konstruktoru, trag prazan — `create_connection` se nije izvršio |
| 10 | `grep -rn "oscal_policy="` nad `core`, `workflow`, `processors`, `examples` | **nula pojava** |
| 10 | sintetička dekorirana komponenta bez kwarga, `strict=True` (`gate_probe.py`) | `OSCALPolicyError: … oscal_policy kwarg is required …` — zato je zadano `strict=False` |

**Trojka (D-10):** alat = `fr_evidence.py`; kriterij = §8 t.1–6; platforma = Python 3.11.15,
Linux (WSL2), radno stablo `oscal` 2026-08-21, artefakti ASD ISM E8 ML1 i NIST→ISM crosswalk.

> **Slijepa pjega (D-11), dvije.** (a) Skripta nije u repozitoriju (`CLAUDE.md` §4). (b) Provjera
> je izvedena **izravnim pozivom** `verify`, ne kroz dekorater — put `@oscal_connection` →
> `OSCAL_CONTROLS` → `verify` nije izvršen u ovoj provjeri jer dekorateri žive u distribuciji
> `wattleflow-workflow`.

## 10. Nefunkcionalni zahtjevi

| NFR | posljedica |
|---|---|
| `NFRQ-SEC-05` | gate je disciplina ulaganja: odbija deklaraciju izvan opsega, ne procjenjuje rizik |
| `NFRQ-SEC-04` | poruka imenuje komponentu, baseline i sporne `id`-eve — nalaz je odmah radnja, ne istraga |
| `NFRQ-SEC-01` | jedan policy po baselineu; prekršaj imenuje `uuid`, pa se opseg vidi |
| `NFRQ-SEC-02` | `__slots__`; policy ne izlaže ni baseline za izmjenu (`frozenset`) |

## 11. Otvoreno

1. **Prazna deklaracija prolazi.** `declared ⊆ baseline` je istinito za prazan skup, pa komponenta
   koja ne deklarira ništa nikad ne pada. Gate time ne razlikuje „nema što deklarirati" od
   „zaboravljeno deklarirati". Kandidat: obvezna neprazna deklaracija za komponente pod
   dekoraterom, ili zaseban razred nalaza. Traži odluku.
2. **Ne provjerava se pokrivenost baselinea** — nitko ne pita je li svaka kontrola iz baselinea
   pokrivena barem jednom komponentom. To je druga polovica compliance pitanja i nema je.
3. **Dekorateri žive u drugoj distribuciji** (`wattleflow-workflow`), a
   `wattleflow.decorators.oscal.policy` uvozi `wattleflow.oscal.policy` **eager**. Ovisnost je
   deklarirana 2026-08-21, pa uvoz radi u svakoj urednoj instalaciji; preostaje operativno —
   izdanje workflowa i instalacija u zatečenim okruženjima ([`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) §7 t.1).
   Smjer `oscal → concrete` je pritom **nepovratan**: zbog njega OSCAL ne može ući u `concrete/`
   (isti §7 t.2).
4. **Vrata su prešla s klase na bazu.** Dekorater je od 2026-08-21 na tri komponentne baze
   (`OSCALConnection`, `OSCALDriver`, `OSCALProcessor`), a konkretne klase ih nasljeđuju —
   [`FRQ-OSCAL-14.13`](FRQ-OSCAL-14.13-component-bases.md). Po klasi ostaje dostupan za ulogu izvan
   te tri; unutar njih je zabranjen (t.7).
5. **Vrata su postavljena, ali inertna.** `strict=False` na sve tri baze znači: dok nitko ne
   predaje `oscal_policy=`, provjera se preskače i `BR-OSCAL-03` **ne provodi ništa**. Sa
   `strict=True` komponente bi padale pri konstrukciji. Prelazak na strogi režim je sada izmjena
   na **tri mjesta**, ne na četrnaest. Odluka *tko predaje politiku*:
   [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) §7 t.2.
6. **Vrata nemaju FR u svojoj distribuciji.** Dekorater je izvedba `BR-OSCAL-03` i `BR-OSCAL-11`,
   ali pripada `wattleflow-workflow`, pa ga kategorija `OSCAL` ne pokriva
   ([`DR-WFL-013`](../04-DR/DR-WFL-013-oscal-requirement-category.md) t.2). Njegov ugovor —
   kad se vrata otvaraju, što znači izostanak `_fsm`, kako `merge` razrješava MRO — danas nije
   nigdje zapisan kao zahtjev. Kandidat: `FRQ-WFL-*`.
7. **Dekorirati i pretka i potomka je latentni kvar.** Vanjski omotač popne `oscal_policy` iz
   kwargsa, pa ga unutarnji više ne vidi: pod `strict=True` unutarnji guard odbija **ispravno
   konfiguriranu** komponentu (`OSCALPolicyError: … oscal_policy kwarg is required`), iako je
   pozivatelj politiku predao. Pod `strict=False` kvar je nevidljiv — vidi se samo dvostruki sloj
   `GuardedStateMachine`. Zato nijedna konkretna komponenta ne ponavlja dekorater svoje baze —
   `ConnectionHuggingFace` ga nema ni prema `ProxyConnection` (kriterij 8). Provjereno
   `double_wrap.py`.
8. **`controls_strategy="merge"` onemogućuje sužavanje deklaracije — odgođeno namjerno.**
   Podklasa koja deklarira uži skup i dalje nasljeđuje roditeljske kontrole kroz MRO
   (`('ism-0445',)` → pet kontrola). Ako specijalizacija stvarno provodi manje od pretka,
   `BR-OSCAL-04` traži `replace`. Odluka je odgođena (autor, 2026-08-21) dok se pristup ne
   provjeri empirijski — po komponenti, ne načelno: [`HLRQ-14`](../01-HLRQ/HLRQ-14-oscal.md) §7 t.5.
