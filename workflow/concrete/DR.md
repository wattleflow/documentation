# DR-001: Singleton je implementacijska politika i seli u workflow concrete/
Status: proposed
Kontekst: ISingleton je bio puna implementacija (metaprogramiranje kroz
  __init_subclass__, omatanje tuđih __init__ metoda, globalni _instances
  registar, per-class lockovi) deklarirana među sučeljima creational.py.
  Krši ORG-02d (I-prefiks na konkretnome); patern bez ijedne apstraktne
  metode — nema ugovora koji bi sučelje deklariralo.
Odluka: Klasa seli u workflow, src/wattleflow/concrete/singleton.py, kao
  Singleton. ISingleton se ukida. Core ne dobiva impl/ sloj: core nudi
  isključivo ugovore, sve politike žive kod potrošača. Opseg proizvoda:
  singleton politika prestaje biti dio wattleflow-core; budući potrošači
  je implementiraju sami ili preuzimaju workflowovu.
Ugovor: Lomi se import putanja preko granice distribucija (core →
  workflow). Potrošači: provjeriti stvarnu upotrebu ISingleton u workflow
  i processors (poznat trag: docstring IScheduler nudi recept
  `class Scheduler(IScheduler, ISingleton)`). Migracija: shim
  `ISingleton = Singleton` + DeprecationWarning u wattleflow/concrete/,
  NE u core/creational.py — core ne smije uvoziti iz workflowa ni radi
  shima (smjer ovisnosti je nepovrediv).
Cijena: Runtime nula. Kognitivna: negativna — granica ugovor/politika
  poklapa se s granicom distribucija, vidljiva u samoj strukturi.
Svjedočanstvo: Aspiracija (idealan core) + zero-trust nalaz (proces-
  globalno mutabilno stanje deklarirano u docstringu kao ambijentalna
  ovlast).
Registar (core, budući): ORG-02d bez iznimaka; novo pravilo: sloj sučelja
  ne uvozi runtime-mehanizme (threading/functools/inspect) — sada bez
  impl/ izuzetka.
Registar (workflow): bases.singleton ili srodan zapis nakon provjere
  upotrebe; concrete/singleton.py ulazi u domenu concrete.