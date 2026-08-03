# GIT-ASSISTANCE

Zbirka git komandi korištenih i objašnjenih tijekom rada na `wattleflow-processors`.
Grupirano po zadaći. Sve komande su lokalne osim gdje je izrijekom navedeno da diraju remote.

> Kontekst grane: trenutna grana je `default`, upstream `origin/default`.

---

## 1. Symlink direktoriji (dev stablo)

Repozitorij u dev stablu ima symlinkane foreign pakete (`core/`, `workflow/`, `oscal/` …).
Za njihov pregled preko `find`:

```bash
# Symlinkovi koji pokazuju na direktorij (bez rekurzije)
find . -maxdepth 1 -type l -xtype d

# Rekurzivno kroz cijelo stablo
find . -type l -xtype d

# Uredan ispis: ime -> cilj
find . -maxdepth 1 -type l -xtype d -printf '%f -> %l\n' | sort
```

| Dio | Značenje |
|---|---|
| `-type l` | unos je symlink (ne provjerava cilj) |
| `-xtype d` | slijedi symlink i traži da **cilj** bude direktorij |
| `-printf '%f -> %l\n'` | `%f` = ime, `%l` = cilj symlinka |

---

## 2. `.gitignore` i praćene (tracked) datoteke

**Ključno pravilo:** `.gitignore` djeluje **samo na nepraćene (untracked) datoteke.**
Kad je datoteka jednom u indexu/historiji, git je nastavlja pratiti i ignore-pravilo se
za nju u potpunosti preskače. Zato se već commitani `examples/` i dalje pojavljuje u
*Staged Changes* unatoč `examples/` unosu u `.gitignore`.

### Dijagnostika ignore-pravila

```bash
# Prijavljuje ignore SAMO za nepraćene putanje (tracked se ne prijavljuje -> exit 1)
git check-ignore -v <putanja>

# Gleda samo pravila, ignorira index -> pokazuje da pravilo POSTOJI i za tracked fajl
git check-ignore -v --no-index <putanja>
```

| Ishod | Značenje |
|---|---|
| exit 1 / prazno (bez `--no-index`) | datoteka je **praćena** — ignore je ne dira |
| `.gitignore:NNN:pravilo  putanja` (exit 0) | pravilo matcha; datoteka je untracked i ignorirana |

### Provjera je li datoteka praćena

```bash
# Vraća putanju ako je praćena, inače error (exit 1)
git ls-files --error-unmatch <putanja>

# Popis svih praćenih datoteka pod direktorijem
git ls-files <dir>/
```

### Prekid praćenja bez brisanja s diska

Datoteka ostaje fizički na disku (`--cached` dira samo index); nakon commita više nije
u repozitoriju ni u sdist/wheel buildu.

```bash
# Cijeli direktorij
git rm -r --cached <dir>/

# Jedna datoteka
git rm --cached <putanja>
```

Provjera nakon toga:

```bash
git ls-files <dir>/                    # prazan ispis = git više ne prati
git check-ignore -v <putanja>          # sad exit 0 + matcha pravilo
ls <dir>/                              # datoteke i dalje na disku
```

---

## 3. Commit — poništavanje i izmjena

### Izmjena poruke zadnjeg commita

```bash
git commit --amend -m "nova poruka"
```

> Ako je commit **već pushan**, `--amend` mijenja hash → sljedeći push traži
> `git push --force-with-lease` (sigurnija varijanta od `--force`).

### Poništavanje zadnjeg commita

| Cilj | Komanda | Učinak |
|---|---|---|
| Skini commit, **zadrži** promjene stageane | `git reset --soft HEAD~1` | index netaknut, spreman za novi commit |
| Skini commit, **odstageaj** promjene | `git reset --mixed HEAD~1` *(ili `git reset HEAD~1`)* | promjene ostaju u working tree, ne u indexu |
| Potpuni povrat (vrati i praćenje) | vidi niže | poništava i `git rm --cached` |

```bash
# Potpuni povrat: skini commit i vrati datoteke kao praćene
git reset --soft HEAD~1
git restore --staged <dir>/
git checkout HEAD -- <dir>/
```

Prije bilo čega provjeri stanje:

```bash
git log --oneline -3
git status
```

---

## 4. Pregled izmjena prije sinkronizacije s GitHubom

Sve što je lokalno ispred remotea (`@{u}` = upstream trenutne grane).

### Nepushani commitovi

```bash
git log @{u}..HEAD --oneline
# eksplicitno ako nema upstreama:
git log origin/default..HEAD --oneline
```

### Popis izmijenjenih datoteka (za commit/tag poruku)

```bash
# Samo imena
git diff @{u}..HEAD --name-only

# S vrstom izmjene (M/A/D)
git diff @{u}..HEAD --name-status

# Osnovna imena, spojena zarezom — za format "vX.Y.Z changes: (...)"
git diff @{u}..HEAD --name-only | xargs -n1 basename | sort -u | paste -sd, -
```

### Puni sadržaj izmjena

```bash
git diff @{u}..HEAD                 # cijeli diff
git log @{u}..HEAD -p --stat        # commit-po-commit, s diffom i statistikom
```

### Još neukomitane promjene (working tree + staged)

```bash
git status
git diff HEAD --stat
```

> Ako `@{u}` javi `no upstream configured`, postavi vezu:
> `git branch --set-upstream-to=origin/default`

---

## 5. Konvencija poruka i tagova

Format tagova u projektu: `v0.0.8 changes: (imena izmijenjenih datoteka)`.
Popis datoteka za poruku dobiva se komandom iz §4.
