# DR-005: Observable politika seli u workflow concrete/observable.py
Status: proposed
Kontekst: IObservableReactive je bio konkretna baza u sloju sučelja: RLock,
  snapshot isporuka, gutanje iznimaka, module-level logger — četiri politike
  fiksirane za sve nasljednike, s prefiksom I na konkretnome (ORG-02d).
Odluka: Ugovor (add/remove/notify) ostaje u core kao ABC; politika postaje
  ThreadSafeObservable u wattleflow/concrete/observable.py.
Ugovor: Lomi se za nasljednike koji su računali na naslijeđenu implementaciju
  — moraju naslijediti ThreadSafeObservable umjesto IObservableReactive.
  Provjeriti potrošače u workflow/processors.
Cijena: Runtime nula. Kognitivna negativna: politika isporuke sada je izbor,
  ne zatečeno stanje; A-dimenzija (nevidljivi kvar promatrača) deklarirana.
Svjedočanstvo: DR-001 kriterij (ugovori bez stanja i politika); zero-trust
  nalaz o sigurnosnoj relevantnosti politike iznimaka.
Registar (workflow): concrete/observable.py u domenu concrete; kandidat za
  bases.observer dopunu (ThreadSafeObservable kao konkretna baza obitelji).