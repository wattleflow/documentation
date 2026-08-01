
# DR-001: Korijen frameworka je konkretan i ne nosi I-prefiks
*Status*: proposed
Kontekst: IWattleflow je konkretna klasa (stanje, implementacija) s prefiksom
  koji konvencija jezgre rezervira za apstraktna sučelja. Korijen nomenklature
  bio je njezina najveća iznimka; lint pravilo ORG-02d (I ⇒ inspect.isabstract)
  nije provedivo dok iznimka postoji.

Odluka: Korijen se preimenuje IWattleflow → Wattleflow. I-prefiks od sada
  označava isključivo apstraktne klase; ORG-02d ulazi u registar bez iznimaka.

Ugovor: Mijenja se javni API core distribucije. Potrošači: svih 5 core modula
  (zvijezda uvoza) + workflow paket + processors. Migracija: mehanička zamjena
  uvoza; prijelazni alias `IWattleflow = Wattleflow` s DeprecationWarning
  kroz jedno minor izdanje, zatim uklanjanje.

Cijena: Jednokratna (rename preko svih distribucija). Runtime: nula.
  Kognitivna: negativna cijena — ime prestaje lagati o prirodi klase.

Svjedočanstvo: ORG-02 (ime nosi informaciju); nalaz da 6 klasa krši implicitnu
  konvenciju, s korijenom kao uzorkom ostalima.

Registar: bases.root: [Wattleflow]; novo pravilo ORG-02d (severity: error,
  waiver: dr_required); alias razdoblje zabilježeno u ključu `tolerated`.


# DR-002: name je read-only property izveden iz tipa
Status: proposed
Kontekst: name je bio mutabilan instance atribut postavljen u __init__.
  Bilo koji kod mogao je preimenovati bilo koji objekt nakon konstrukcije;
  __str__ (vjerojatni ulaz u audit zapise) ispisivao bi krivotvoreni
  identitet. Za framework s audit domenom to je integritetska (I u CIA)
  odluka fiksirana u korijenu i naslijeđena od svih.
Odluka: name postaje property nad type(self).__name__. Ukida se __init__
  (korijen bez stanja); __slots__ postaje prazan.
Ugovor: Poziv obj.name ostaje istog oblika i tipa. Lomi se: (a) kod koji
  name PIŠE — sada AttributeError, što je namjera odluke; (b) kod koji
  očekuje konstruktorski lanac korijena. Potrošači se utvrđuju grep-om
  po workflow/processors prije prihvaćanja (poznat rizik: dinamička
  imena u driver/manager sloju, ako postoje, trebaju vlastiti atribut
  pod drugim imenom, npr. label).
Cijena: Pristup name-u: atribut (~20 ns) → property (~80 ns) po čitanju.
  Konstrukcija: jeftinija (nema __init__ poziva, nema slot zapisa, nema
  po-instanci stringa). Neto pozitivno za mnogo kratkoživućih objekata
  (pipeline dokumenti); negativno za vruću petlju koja čita name —
  takva petlja nije poznata, ali odluka se revidira ako se pojavi.
Svjedočanstvo: Aspiracija utemeljena na NFR smjeru (integritet audita);
  empirijska potvrda čeka analizu audit/ paketa.
Registar: Nema izmjene vokabulara. Kandidat za novo dijagnostičko pravilo:
  zabrana pisanja u name (statički detektabilno kao attribute assignment).