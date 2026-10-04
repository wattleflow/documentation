<link rel="stylesheet" href="requirements/styles/wattleflow.css">

# Registar teorijskih postulata

| | |
|---|---|
| **Verzija** | Draft v0.1 |
| **Nadređeni dokumenti** | [`PHILOSOPHY.md`](PHILOSOPHY.md), [`METHODOLOGY.md`](METHODOLOGY.md) |
| **Srodni registri** | [`LITERATURE.md`](LITERATURE.md) (gdje piše), [`dictionary.yaml`](dictionary.yaml) (što pojam znači) — ovaj registar bilježi **tvrdnje same** |
| **Upravljanje** | §Održavanje |

## Svrha

Reference kažu *gdje* nešto piše; rječnik *što pojam znači*; ovaj registar
čuva **teorijske postulate** — nosive tvrdnje na kojima doktrina stoji — s
podrijetlom (tko, kada, u kojoj tradiciji), preciznim iskazom i mjestom
primjene u Wattleflowu. Time se sprječava da „gem" ostane raspršen po
raspravama i izgubi rodoslovlje, i da se tuđa misao neopaženo pretvori u
„opće mjesto" bez atribucije.

Što je `postulat` i čime se razlikuje od `hipoteze`
definira rječnik. Ukratko: postulat se **koristi** (kaže *zašto*), hipoteza se **provjerava**
(kaže *radi li*).

Format natuknice: **P-NN — Iskaz** / *Podrijetlo* / *Primjena* / `[ključ]`.

`Analiza.md` u primjenama označava istraživački rad `workflow/Analiza.md`,
koji od 2026-09-19 postoji (dotad `TOOLS.md`), ali nosi samo **§1, §2 i §7**. Pozivi na §3–§6
i dalje su pozivi na tekst koji čitatelj ne može otvoriti — deklarirana rupa (`DOCUMENTATION.md` §3.1).

---

## I. Sustavsko mišljenje i operacijska istraživanja

**P-01 — Povratna petlja je znanstveni instrument: vrti se nad teorijom
(hipoteza → mjerenje → zaključak), ne nad popisom posla.**
*Podrijetlo:* Shewhartov ciklus specifikacija–proizvodnja–inspekcija
(1930-e, statistička kontrola procesa) [47], koji Deming 1950. donosi u
Japan i razvija u PDSA (Plan–Do–Study–Act; *Study*, ne *Check* — proučavanje
nasuprot pukoj provjeri) [48]. Temelj Demingova učenja i pristupa analizi,
a preko OR tradicije i cijele discipline operacijskih istraživanja.
*Primjena:* Motivacijska petlja filozofije („empirijski dokaz → vraća se u
načela"); razlika prema agilnoj iteraciji (rječnik: „iteracija").
`[47][48]`

**P-02 — Iskustvo bez teorije ne uči ništa.**
*Podrijetlo:* Deming, teorija znanja unutar Sustava dubokog znanja (System
of Profound Knowledge): informacija nije znanje; znanje traži teoriju koja
predviđa, i reviziju teorije dokazima [48][55].
*Primjena:* obrazloženje zašto mjerenje bez hipoteze nije metodologija
(METHODOLOGY §1 t.1); zašto backlog-petlja nije empirizam.
`[48][55]`

**P-03 — Svojstva sustava žive u interakcijama dijelova, ne u dijelovima;
analiza rastavlja, sinteza objašnjava.**
*Podrijetlo:* Ackoff, sustavska škola; formalizacija sustavskih pojmova u
sustav pojmova [49]; pionirska OR tradicija (prvi udžbenik operacijskih
istraživanja: Churchman, Ackoff & Arnoff, 1957) [54].
*Primjena:* argument publike dokumentacije (iz hrpe pojedinačnih zapisa ne
može se izvesti integracijsko svojstvo); `Analiza.md` §5 — particije i njihova
neslaganja kao interakcijska svojstva.
`[49][54]`

**P-04 — Svaki je sustav, u odnosu na ponašanje svojih dijelova, ili
varijetet-povećavajući ili varijetet-smanjujući.**
*Podrijetlo:* Ackoff 1971, propozicija u sklopu sustava sustavskih pojmova
[49]; komplementarno Ashbyjevu zakonu.
*Primjena:* dokumentacijski sustav i registri kao varijetet-smanjujući
mehanizmi (kontrolirani vokabular smanjuje semantički varijetet);
arhitektonske granice kao regulatori varijeteta.
`[49]`

**P-05 — Zakon nužne raznolikosti: samo raznolikost može apsorbirati
raznolikost — regulator mora imati barem onoliko stanja koliko i ono što
regulira.**
*Podrijetlo:* Ashby, kibernetika [30]; regulacija i povratna veza kao opći
mehanizam: Wiener [37].
*Primjena:* jedan dokumentacijski format za sedam publika krši zakon po
definiciji (`PHILOSOPHY.md`, „Arhitektura kao obrazovni artefakt"); dimenzije
vektorskog nalaza kao raznolikost instrumenta.
`[30][37]`

## II. Informacijske znanosti

**P-06 — Informacija je mjerljiva nesigurnost: entropija ograničava što se
kanalom može prenijeti, neovisno o značenju poruke.**
*Podrijetlo:* Shannon 1948 [36].
*Primjena:* granica formalizma — Shannon mjeri prijenos, ne značenje; zato
semantika treba vlastite instrumente (P-07, P-08) i zato `Analiza.md` §3.3
(nesumjerljivost) stoji.
`[36]`

**P-07 — Informacija nije podatak: informacija je funkcija podatka,
primateljeva predznanja i vremena interpretacije (infološka jednadžba).**
*Podrijetlo:* Langefors, skandinavska škola informacijskih sustava [39].
*Primjena:* dokumentacija po publikama (isti podatak, različita
informacija); obrazovna vrijednost arhitekture kao prijenos predznanja.
`[39]`

**P-08 — Znak i značenje stabiliziraju se konvencijom; organizacija je
semiotički sustav u kojem se ta konvencija mora održavati.**
*Podrijetlo:* semiotika (Peirce tradicija; Eco [51]); organizacijska
semiotika i semiotičke ljestve: Stamper [38].
*Primjena:* registri kontroliranog vokabulara kao semiotički stabilizatori
(kod, diskurs, reference); hipoteza H2.
`[38][51]`

## III. Mjerenje

**P-09 — Mjerenje je homomorfizam: brojevi smiju tvrditi samo one odnose
koji postoje u empirijskom sustavu; tip skale određuje dopuštene
operacije.**
*Podrijetlo:* reprezentacijska teorija mjerenja (Krantz, Luce, Suppes,
Tversky) [15]; tipologija skala: Stevens [16].
*Primjena:* protokol V1–V6; zabrana skalara nad nominalnim/ordinalnim
(vektorski nalaz); rječnik „mjera vs metrika vs indeks".
`[15][16]`

**P-10 — Pokazatelj korišten za upravljanje prestaje mjeriti ono što je
mjerio: mjera pod pritiskom korumpira i sebe i proces koji nadzire.**
*Podrijetlo:* Campbell 1979 [22] (srodno Goodhartu).
*Primjena:* dijagnostička naspram upravljačke uporabe (METHODOLOGY §1
t.3); gate samo na binarnim pravilima; hipoteza H3; ABS-03 incident kao
empirijska nulta točka tihe korozije instrumenta.
`[22]`

## IV. Softversko i sustavsko inženjerstvo

**P-11 — Modul je granica oko odluke koja se može promijeniti; skrivanje
informacija štiti ostatak sustava od te promjene.**
*Podrijetlo:* Parnas 1972 [1].
*Primjena:* registar kao Parnasov potez (pravila u podacima, ne u alatu);
kriterij dekompozicije jezgre; rječnik „modul".
`[1]`

**P-12 — Složeni sustavi koji rade nastaju iz jednostavnih sustava koji
rade; hijerarhija i gotovo-rastavljivost su uvjet evolucije složenosti.**
*Podrijetlo:* Simon 1962 [2] (near-decomposability); popularno srodno:
Gallov zakon.
*Primjena:* slojevi Core → Workflow → alati; serija malih zelenih koraka
umjesto big-banga.
`[2]`

**P-13 — Esencijalna složenost se ne uklanja, nego premješta; alat koji
tvrdi da ju je uklonio mjeri njezino premještanje.**
*Podrijetlo:* Brooks 1987 [29]; kibernetički oblik istog uvida: Ashby [30].
*Primjena:* granice onoga što lint i indeksi smiju tvrditi (`Analiza.md` §3.2);
stupac „što alat ne može mjeriti".
`[29][30]`

**P-14 — Idealni racionalni proces nije dostižan, ali dokumentacija mora
prikazati racionaliziranu strukturu: zapis je specifikacija znanja, ne
dnevnik rada.**
*Podrijetlo:* Parnas & Clements 1986 [43].
*Primjena:* sinteze v0.x dokumenata (stihijski nacrt → rigorozna
revizija); Bilješke o sintezi kao pošteni trag racionalizacije.
`[43]`

**P-15 — Jednosmjerni sekvencijalni prolaz „poziva na neuspjeh": izvorni
„vodopadni" model bio je iterativan i propisivao je gradnju pilota („do it
twice").**
*Podrijetlo:* Royce 1970, tekst izvornika (str. 2) — ne sekundarne
karikature [40]; povijesni pregled iterativnosti prije manifesta: Larman &
Basili [41].
*Primjena:* korekcija procesnog narativa u Motivaciji filozofije; obrana
teze da empirizam nije agilni izum.
`[40][41]`

**P-16 — Program je teorija koju tim drži o sustavu i svijetu; artefakti
su nužni, ali nedovoljni nositelji te teorije — smrt teorije je smrt
programa.**
*Podrijetlo:* Naur 1985 [53].
*Primjena:* epistemološki temelj vrijednosti arhitekture: arhitektura je
eksternalizirani dio teorije, odluke njezin dokazni zapis, prikazi njezine
projekcije; „arhitektura kao obrazovni artefakt" = prijenos teorije.
*Dva čitanja:* agilna tradicija iz Naura izvodi da teorija živi u ljudima, ne u
dokumentima; ovdje se isti postulat čita kao nalog za discipliniraniji zapis, jer su
artefakti nužni uvjet obnove teorije. Pri citiranju izreći oba.
`[53]`

**P-22 — Separation of Concerns: jedan aspekt problema razmatra se odvojeno od ostalih, uz svijest
o njihovoj vezi; to je jedina poznata tehnika uređivanja mišljenja o složenom sustavu.**
*Podrijetlo:* Dijkstra 1974, „separation of concerns” [65]; katalog design patterna kojim se razdvojene
brige imenuju: Gamma et al. 1994 [66].
*Primjena:* svaka briga ima jednu komponentu i jedan ugovor — format je formater, perzistencija
driver, izbor strategija, vrijednosti konfiguracija; slojevi zahtjeva (zašto, što, kako). Granica
prema P-11: Separation of Concerns određuje *koje su* brige, information hiding *gdje* je granica modula.
`[65][66]`

---

## V. Vlastiti postulati [S] — tvrdnje koje se brane, ne citiraju

**P-17 [S] — Modularnost uvodi rizik istim potezom kojim daje korist:
granica koja skriva odluku skriva i podrijetlo.**
*Podrijetlo:* vlastita inverzija Parnasa (središnja teza, `Analiza.md` §2).
*Primjena:* supply-chain analiza; particija povjerenja naspram
arhitektonske particije.
*Status:* konceptualna — nema Hn; put obrane: mjerenja
`Analiza.md` §4–5, odjeljaka kojih u toj datoteci nema (rupa, `DOCUMENTATION.md` §3.1).

**P-18 [S] — Povratna petlja testira samo ono što promatra unutar svoje
latencije: signali s latencijom duljom od ciklusa strukturno su nevidljivi
procesu, i to je specifikacija slijepe pjege, ne vrijednosni sud.**
*Podrijetlo:* vlastita formulacija, izgrađena na P-01 i P-10.
*Primjena:* argument o sigurnosnim posljedicama procesa; opravdanje alata
koji mjere ono što petlja ne vidi.
*Status:* konceptualna; kandidat za operacionalizaciju (latencija klasa
signala po procesu).

**P-19 [S] — Konformnost nije kvaliteta: zeleni nalaz dokazuje usklađenost
s deklariranim kriterijem, a vrijednost nalaza ovisi o valjanosti
kriterija.**
*Podrijetlo:* vlastita destilacija P-09 + P-10 kroz praksu C-snimki.
*Primjena:* rječnik („konformnost vs kvaliteta"); iskaz svakog vektora.
*Status:* usvojeno pravilo diskursa.

**P-20 [S] — Prikaz nikad nije izvor istine: pogled se generira iz stanja
ili se prema njemu verificira; drift prikaza ista je klasa kvara kao drift
dokumentacije.**
*Podrijetlo:* vlastita primjena semiotike (P-08) i ISO 42010 distinkcija.
*Primjena:* pet kontroliranih vokabulara (D-12); pravilo za dijagrame;
`dictionary.yaml` → `DICTIONARY.md` smjer.
*Status:* usvojeno pravilo doktrine.

**P-21 [S] — Konkatenacija nije kompozicija: spajanje bez ugovora
proizvodi cjelinu čija svojstva nitko nije dizajnirao.**
*Podrijetlo:* vlastita formulacija: kompozicija ima algebru (ugovor na granici jamči
svojstvo cjeline; konformno ∘ konformno = konformno), konkatenacija je jukstapozicija bez
jamstva. Oslonac: P-03 (nedizajnirane interakcije = nedizajnirana svojstva), P-12 (zašto
kratkoročno radi), P-18 (zašto se cijena vidi kasno).
*Primjena:* supply-chain analiza (P-17 je konkatenacija ovisnosti);
registar + lint kao mehanizam pretvaranja konkatenacije u kompoziciju
(granica dobiva ugovor i provjeru); semantička entropija kao semiotički
oblik iste pojave (konkatenirani vokabulari).
*Status:* konceptualna; put obrane: mjerljiva razlika sustava s
provedenim ugovorima granica naspram bez njih (H1, H2 signali).

**P-23 [S] — Tajna nije konfiguracija: vjerodajnica (korisničko ime, lozinka,
ključ, token) ulazi u sustav na granici pokretanja, iz okoline koju repozitorij
ne prati, i u čitljivom obliku nikad ne postaje dio koda, predloška ni povijesti.**
*Podrijetlo:* vlastita primjena Saltzer–Schroeder [23]: *open design* (zaštita ne smije
ovisiti o tajnosti mehanizma, nego samo o tajnosti ključa — ključ u repozitoriju nije ključ)
i *least privilege*; oslonac P-17 (granica koja skriva odluku skriva podrijetlo: tajna u
povijesti ima podrijetlo koje se ne može povući) i P-21 (predložak s upisanom tajnom je
konkatenacija koda i okoline). Konfiguracija koja *ne* ovlašćuje (putanje, portovi, pragovi)
ostaje u repozitoriju — postulat razdvaja dvije stvari koje isti format spaja.
*Primjena:* `NFRQ-SEC-09`…`SEC-14` (CIA instance); `HLRQ-20`; obrazac `.env` +
`.env.example` + `${VAR:?}` u `blackwattle/dockers/`; skener stabla i povijesti
(lokalni pregled, ne objavljuje se).
*Status:* usvojeno pravilo doktrine; put obrane: skener nad praćenim
stablom i cijelom poviješću svakog repozitorija — nalaz s vrijednošću vjerodajnice obara
tvrdnju da je pravilo provedeno; slučaj u kojem sustav *ne može* početi bez praćene tajne
oborio bi postulat sam (nije zabilježen).

---

## Održavanje

1. Postulat ulazi s dokumentiranom odlukom; iskaz mora biti jedna rečenica koja se može
   citirati samostalno, podrijetlo s ključem, primjena s mjestom u
   doktrini.
2. [S] postulati nose status (hipoteza / konceptualna / usvojeno pravilo)
   i put obrane; [S] bez puta obrane ne ulazi. Status *hipoteza* traži
   odgovarajući Hn u `PHILOSOPHY.md`; bez njega status je *konceptualna*.
3. Sukob postulata razrješava se eksplicitno (dokumentiranom odlukom), nikad prešutnim
   biranjem u tekstu.
4. Registar hrani `PHILOSOPHY.md` (Znanstveni temelji), `METHODOLOGY.md` §5
   (Znanstveni temelji) i `Analiza.md` §5; ti dokumenti citiraju P-oznake gdje
   tvrdnju koriste.
