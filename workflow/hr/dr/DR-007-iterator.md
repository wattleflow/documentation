# DR-007: Lijena iterator-mašinerija seli u concrete/iterator.py
Status: proposed
Kontekst: IIterator/IAsyncIterator držali su stanje (_iterator), __init__ i
  politiku lijene konstrukcije s keširanjem u sloju sučelja (DR-001 test:
  stanje + politika, nije definirajući algoritam paterna).
Odluka: Core ugovori se svode na create_iterator() + naslijeđeni protokol;
  LazyIterator/LazyAsyncIterator u wattleflow/concrete/iterator.py.
Ugovor: Nasljednici koji su računali na naslijeđeni __next__ prelaze na
  Lazy* baze. Provjeriti potrošače (generator-sloj GenericProcessor?).
Cijena: Runtime nula; kognitivna negativna (lijenost postaje izbor).
Svjedočanstvo: DR-001 kriterij.
Registar (workflow): concrete/iterator.py u domenu concrete.

# DR-008: Higijena behavioural — trag umjesto mrtvog koda
Status: proposed
Odluka: (a) BUG #1/#2 (IStrategy caller potpis, LSP popravak) prelazi iz
  region FIX komentara u ovaj zapis; potpis execute(caller, **kwargs) je
  važeći ugovor. (b) ILogger.log(level, ...) alternativa odbijena i
  brisana; API po razinama je namjeran. (c) WattleType duplikat ukinut;
  uloga Element. (d) Norma imena: behavioural (britanska, usklađeno s
  initialise/finalise); lint provjerava podudarnost zaglavlja i datoteke.