# `wattleflow.core` — dijagrami sučelja

Jedan PlantUML class dijagram po sučelju (86), plus rasterizirana slika svakog od njih.
Ugrađeni su u [`../INTERFACES.md`](../INTERFACES.md), po jedan uz zapis sučelja.

**Pogled, ne izvor istine (D-13).** Izvor je kod u `src/wattleflow/core/`; `.puml` je generiran
iz njega, a `.jpg` je rasterizirani `.puml`. Ništa se ovdje ne uređuje ručno — mijenja se kod
pa se regenerira. Ime datoteke je ime sučelja (`IRepository.puml`, `IRepository.jpg`).

## Gledište

Namjerno usko: **jedno sučelje kao predmet**, nasljedni put koji do njega vodi, i core sučelja
koja iz njega izravno proizlaze. Dijagram cijelog paketa bio bi 86 kutija i ne bi rekao ništa
što indeks već ne kaže.

- Predmet je istaknut; preci nose vlastite apstraktne članove, pa je stog kutija cijeli ugovor.
- Potomci su samo imena — svaki ima vlastiti dijagram.
- Lepeza šira od 10 potomaka **se ne crta nego se broji** u bilješci (`IWattleflow` ih ima 79).
  Izostavljanje je deklarirano, ne prešućeno (D-11).
- Tipski parametri se čitaju ondje gdje su deklarirani: `Generic[...]` ide u kutiju, a
  potparametrizirana baza (`IComponent[Element]`) na brid.
- `ABC` se crta ondje gdje prvi put ulazi u obitelj; nasljeđivanje je tranzitivno, pa
  ponavljanje deklaracije niže govori o zapisu, ne o arhitekturi.
- Odstupanje od konvencije docstringa vidljivo je kao bilješka `Finding:` — deklarirana rupa
  (D-11), ne greška renderiranja.

## Regeneracija — dva koraka

Alat piše `.puml` i imenuje sliku koju `INTERFACES.md` očekuje; **ne rasterizira**, jer su
alati core repozitorija stdlib-only (`.gitignore`), a PlantUML traži Javu.

```bash
# 1. .puml + INTERFACES.md (iz core repozitorija)
cd ~/projects/wattleflow/core
python tools/core_index.py --uml documentation/uml --markdown documentation/INTERFACES.md

# 2. .jpg — PlantUML piše PNG, JPEG nije njegov izlazni format
cd ~/projects/wattleflow/documentation/core/uml
java -jar plantuml.jar -tpng -nbthread auto -o /tmp/uml-png "*.puml"
python -c "
from pathlib import Path
from PIL import Image
for png in sorted(Path('/tmp/uml-png').glob('*.png')):
    image = Image.open(png)
    flat = Image.new('RGB', image.size, (255, 255, 255))
    flat.paste(image, mask=image.split()[-1] if image.mode == 'RGBA' else None)
    flat.save(Path('.') / f'{png.stem}.jpg', 'JPEG', quality=92, optimize=True, subsampling=0)
"
```

JPEG nema alfa kanal, pa se PNG **spaja na bijelo** — bez toga prozirna pozadina padne u crno.
Pillow nije ovisnost core repozitorija; korak 2 se izvodi u okruženju koje ga ima.

Alat briše `.puml` koji ne pripada nijednom sučelju, ali **ne dira rastere** — datoteke koje ne
stvara ne uklanja. Nakon uklanjanja sučelja obriši i pripadni `.jpg` ručno.

## Trojka reproducibilnosti (D-10)

| član | vrijednost |
|---|---|
| alat | `tools/core_index.py 1.1.0` (core repozitorij) |
| kriterij | docstringovi i AST `src/wattleflow/core/` — 86 sučelja, 80/86 usklađeno s konvencijom |
| platforma | PlantUML 1.2025.4, OpenJDK 17.0.18, Pillow 12.2.0, Linux (WSL2) |

Zabilježeno 2026-09-08. Šest sučelja bez docstringa (`IComponent`, `IComposite`, `IDecorator`,
`IFlyweight`, `IDocument`, `ISignal`) nose `(role not declared)` u naslovu i bilješku `Finding:`
u dijagramu; popravak ide kroz `DR-COR`, ne uz regeneraciju.
