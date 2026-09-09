# Konformnost — C-snimke i finding-vektori

**Ovo je stablo prazno, i to je nalaz, ne stanje mirovanja.**

Ovdje pripadaju rezultati mjerenja konformnosti (`CLAUDE.md` §9):

- **C-snimka** — arhivira se samo **zeleni** run; nosi `vector`, `reproducibility_triple`
  (`tool`, `criterion`, `platform`), `source`, `rules_selected` i blok `blind_spots`.
- **finding-vektor** — run s greškama; nikad se ne arhivira kao C-snimka (D-10).

## Zašto je prazno

Nijedna snimka još nije uzeta. Alat se pokreće **na zahtjev**, jer CI nije odlučen
([`DR-WFL-023`](../../04-DR/DR-WFL-023-test-framework-is-stdlib-unittest.md) §Otvoreno):

```
python tools/wem_lint.py --src src/wattleflow --snapshot \
    ../documentation/workflow/conformance/<distribucija>-<datum>.json
```

`--snapshot` prima **putanju**; bez argumenta alat prekida s `error: argument --snapshot:
expected one argument` (provjereno 2026-09-09, `wem_lint` 1.17.0). Kanonsko odredište je ovo
stablo — tako kaže i `--help`.

Dok je stablo prazno vrijedi §9: **nemjereno se ne smije čitati kao čisto.** Prazan
direktorij ne znači zelen sustav — znači da mjerenja nema.

## Deklarirane rupe (D-11)

1. **Nijedna distribucija nema snimku.** Alat je nad `blackwattle` stablom izvršen 2026-08-22
   (0 error, warningi zatečeni), ali vektor nije arhiviran — vodi se u
   [`TODO.md`](../TODO.md).
2. **Za `NFRQ-OBS-01/02/03` snimka ne može biti valjana** dok kriterij lintera mjeri po
   `DR-WFL-018`, a nije usklađen s `DR-WFL-021` ni `DR-WFL-028`. Snimka uzeta prije
   usklađenja nosila bi vektor oblika punog runa nad zastarjelim kriterijem.

Ovaj `README` postoji da poveznice iz `CLAUDE.md` §3.1/§9/§10, `DOCTRINE.md` D-10,
[`NFRQ-000-EN.md`](../../03-NFRQ/NFRQ-000-EN.md) i `README.md` ne pokazuju u prazno —
ne kao tvrdnja da mjerenja ima.
