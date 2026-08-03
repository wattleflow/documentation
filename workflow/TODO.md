TODO LISTA
- prebaciti CLAUDE.md u documentation/CLAUDE.md i napraviti link na njega. 
- Tri nalaza za raspravu
    1. Instrument skriva stvarni prekršaj (H3 signal). helpers/config.py isključen je jer uvozi yaml — a upravo on nosi ORG-01 ciklus helpers → concrete.exception. Clean-core filter i ORG-01 pravilo se sudaraju: filter briše modul prije nego ga pravilo vidi. Sada je barem deklarirano, ali prekršaj i dalje nije izmjeren.
    2. Dva ERROR-a su greška registra, ne koda. constants je u domains:, a ne uvozi ništa iz wattleflowa — čisti list. Poruka to i kaže („update the registry"). Registar mijenjamo tek u koraku D uz DR, pa ostaje.
    3. --quiet je brisao slijepe pjege iz vektora. Popravio sam: --quiet sada krati samo ispis detalja; vektor i snimka ostaju cjeloviti. Prekidač ne smije moći pretvoriti „nije mjereno" u „mjereno čisto".
    Ostaje neriješeno iz Faze 0: MANIFEST.in i dalje proizvodi sdist bez ijedne .py datoteke — release blocker, nezavisan od linta.
    Idemo na D (migracija registra u dictionary.yaml + DR zapis), ili prvo raspraviti ova tri nalaza?
- dodatni zadatak

DONE
+ analzirati .gitignore datoteku i uskladiti je sa projektom
+ analzirati MANIFEST datoteku i uskladiti je sa projektom i .gitignore
- analzirati pyproject.toml i uskladiti je sa projektom


- dodatni zadatak
Uvod u Apache NiFi
Uvod
Što je Apache NiFi?
Definicija
Povijest i razvoj
Primjene u industriji
2. Osnovne Značajke
Arhitektura
Komponente NiFi sustava
Podatkovni tok (Data Flow)
User Interface (Korisničko sučelje)
Pregled glavnih funkcionalnosti
Demonstracija korisničkog sučelja
3. Korištenje Apache NiFi
Instalacija i konfiguracija
Osnovni koraci za instalaciju
Ključne konfiguracijske datoteke
Kreiranje i upravljanje podatkovnim tokovima
Procesori, konektori i kontrolne ploče
Praktični primjer: Jednostavan podatkovni tok
4. Sigurnost i Skalabilnost
Sigurnosne značajke
Autentifikacija i autorizacija
SSL/TLS enkripcija
Skalabilnost
Horizontalno i vertikalno skaliranje
Cluster konfiguracija
5. Integracija i Ekstenzibilnost
Integracija s drugim alatima
Apache Kafka, Hadoop, Spark
Cloud usluge (AWS, Azure, GCP)
Dodavanje prilagođenih procesora
Pregled NiFi SDK-a
Razvoj i implementacija prilagođenih procesora
6. Studije Slučaja
Primjeri iz prakse
Korporativne implementacije
Prednosti i izazovi
7. Zaključak
Sumiranje ključnih točaka
Prednosti korištenja Apache NiFi
Budući razvoj i trendovi
Q&A
Otvorena rasprava i odgovori na pitanja
Dodatni Materijali
Preporučena literatura
Službena dokumentacija
Online tečajevi i tutorijali
Demo i resursi
GitHub repozitoriji
Primjeri konfiguracija i skripti
Ovaj prijedlog strukturiran je tako da pruža temeljit pregled Apache NiFi-a, prilagođen kako početnicima tako i iskusnijim korisnicima. Sadrži praktične primjere i studije slučaja za bolje razumijevanje stvarnih primjena.


Koje probleme NiFi pokusava rjesiti?

Apache NiFi pokušava riješiti nekoliko ključnih problema vezanih uz upravljanje podatkovnim tokovima i integraciju podataka. Evo nekoliko najvažnijih problema koje NiFi rješava:
Automatizacija i Orkestracija Podatkovnih Tokova:
NiFi omogućava automatizirano upravljanje i orkestraciju podatkovnih tokova između različitih sustava i aplikacija. To uključuje prikupljanje, obradu, transformaciju i distribuciju podataka.
Vizualizacija i Jednostavno Upravljanje Podacima:
Nudi intuitivno grafičko korisničko sučelje koje omogućuje korisnicima da vizualno dizajniraju, upravljaju i nadziru podatkovne tokove bez potrebe za pisanjem koda.
Sigurnost Podataka:
Pruža napredne sigurnosne značajke, uključujući enkripciju podataka u prijenosu (SSL/TLS), autentifikaciju i autorizaciju, čime osigurava da su podaci sigurni tijekom cijelog toka.
Skalabilnost i Performanse:
NiFi podržava horizontalno i vertikalno skaliranje, omogućujući prilagodbu performansi prema potrebama. Može raditi u klasteru kako bi se osigurala visoka dostupnost i balansiranje opterećenja.
Praćenje i Nadgledanje:
Omogućava detaljno praćenje i nadgledanje podatkovnih tokova u stvarnom vremenu. Korisnici mogu vidjeti točno gdje se podaci nalaze, koliko ih je obrađeno i gdje se javljaju eventualni problemi.
Fleksibilnost i Ekstenzibilnost:
NiFi podržava širok raspon izvora i odredišta podataka, uključujući relacijske baze podataka, NoSQL baze, datoteke, API-je, cloud servise itd. Također omogućuje dodavanje prilagođenih procesora za specifične potrebe.
Pouzdanost i Robusnost:
Dizajniran je da bude pouzdan i otporan na greške. NiFi osigurava točnost i integritet podataka kroz mehanizme za ponovni pokušaj, provjeru uspješnosti i transakcijsku obradu.
Real-Time i Batch Obrada:
Podržava obradu podataka u stvarnom vremenu (streaming) kao i batch obradu, omogućujući korisnicima da odaberu najprikladniji način obrade prema svojim potrebama.
Integracija i Interoperabilnost:
NiFi omogućava lako povezivanje različitih sustava i aplikacija, olakšavajući integraciju podataka iz heterogenih izvora i njihovu konsolidaciju u centralizirane sustave.
Rješavajući ove probleme, Apache NiFi postaje moćan alat za upravljanje složenim podatkovnim tokovima i integraciju podataka u različitim okruženjima i industrijama.

Usporedba Apache NiFi i Talend Data Integration
Apache NiFi i Talend Data Integration su popularni alati za upravljanje podatkovnim tokovima i integraciju podataka. Evo usporedbe njihovih ključnih značajki i mogućnosti:
1. Namjena i Primjena
Apache NiFi:
Primarno fokusiran na automatizaciju, orkestraciju i vizualno upravljanje podatkovnim tokovima.
Pogodan za real-time obradu podataka (streaming) i batch obradu.
Koristi se za prikupljanje, obradu i distribuciju podataka u heterogenim okruženjima.
Talend Data Integration:
Fokusiran na ETL (Extract, Transform, Load) procese i širu integraciju podataka.
Pogodan za složene transformacije podataka i integraciju sa širokim spektrom sustava.
Koristi se u poslovnim okruženjima za konsolidaciju podataka, migraciju i sinkronizaciju.
2. Korisničko Sučelje
Apache NiFi:
Intuitivno grafičko korisničko sučelje (GUI) za dizajn podatkovnih tokova.
Vizualno upravljanje omogućava jednostavno praćenje i nadzor nad podacima.
Talend Data Integration:
Grafički dizajner za ETL procese.
Pruža bogatije alate za transformaciju podataka i složenije mape procesa.
3. Skalabilnost i Performanse
Apache NiFi:
Podržava horizontalno skaliranje kroz klastere.
Dizajniran za visoku propusnost i nisku latenciju.
Talend Data Integration:
Također podržava skaliranje, ali obično se koristi u vertikalno skaliranim sustavima.
Dobro optimiziran za složene ETL zadatke, ali može zahtijevati više resursa za velike podatkovne skupove.
4. Sigurnost i Upravljanje Pristupom
Apache NiFi:
Pruža napredne sigurnosne opcije uključujući SSL/TLS enkripciju, autentifikaciju i autorizaciju.
Omogućuje detaljno praćenje pristupa i aktivnosti.
Talend Data Integration:
Ima ugrađene sigurnosne funkcije, ali obično zahtijeva dodatne konfiguracije za napredne sigurnosne potrebe.
Integrira se s postojećim sigurnosnim sustavima unutar organizacije.
5. Ekosustav i Integracija
Apache NiFi:
Fleksibilan alat s mogućnošću integracije s mnogim sustavima kao što su Kafka, Hadoop, Spark, različiti DBMS-ovi, cloud servisi.
Pruža mogućnost razvoja prilagođenih procesora.
Talend Data Integration:
Podržava širok spektar konektora za različite baze podataka, aplikacije, cloud servise.
Dolazi s bogatim ekosustavom dodatnih alata za upravljanje podacima, kvalitetu podataka i MDM (Master Data Management).
6. Cijena i Licenciranje
Apache NiFi:
Open-source alat dostupan besplatno pod Apache 2.0 licencom.
Talend Data Integration:
Dostupan kao open-source verzija (Talend Open Studio), ali napredne funkcije zahtijevaju komercijalnu licencu.
Zaključak
Apache NiFi je idealan za organizacije koje trebaju fleksibilno i skalabilno rješenje za real-time obradu podataka i upravljanje podatkovnim tokovima.
Talend Data Integration je pogodan za složene ETL zadatke i integraciju podataka u poslovnim okruženjima, posebno kada je potrebna bogata transformacija podataka i integracija s različitim sustavima.
Izbor između NiFi i Talend ovisi o specifičnim potrebama vaše organizacije, uključujući zahtjeve za real-time obradom, složenošću transformacija podataka i budžetom za softverske alate.


Da, Apache Foundation ima nekoliko projekata koji se mogu smatrati sličnim Apache NiFi-u, iako svaki od njih ima specifične karakteristike i namjene. Evo nekoliko primjera:
1. Apache Flume
Namjena: Flume je alat za prikupljanje, agregaciju i pomicanje velikih količina podataka, posebno dizajniran za prijenos podataka u Hadoop.
Primjena: Često se koristi za prikupljanje logova iz različitih izvora i prijenos tih logova u Hadoop HDFS za daljnju analizu.
Ključne Značajke:
Pouzdana isporuka podataka.
Skalabilna arhitektura.
Podržava prilagodljive podatkovne tokove.
2. Apache Kafka
Namjena: Kafka je distribuirana streaming platforma koja se koristi za izgradnju real-time podatkovnih cjevovoda i aplikacija koje reagiraju na tok podataka.
Primjena: Pogodan za izgradnju podatkovnih streaming aplikacija, real-time analitiku, i prikupljanje podataka u velikom mjerilu.
Ključne Značajke:
Visoka propusnost i niska latencija.
Tranzakcijska podrška.
Skalabilnost kroz distribuciju.
3. Apache Storm
Namjena: Storm je distribuirani real-time računalni sustav koji omogućava obradu podataka dok stižu (streaming).
Primjena: Koristi se za real-time analitiku, online strojno učenje, kontinuirani proračun i obradu događaja.
Ključne Značajke:
Brza obrada podataka.
Skalabilnost i otpornost na greške.
Integracija s različitim sustavima kao što su Hadoop i Kafka.
4. Apache Airflow
Namjena: Airflow je platforma za programiranje, raspoređivanje i nadzor tijekova rada (workflows).
Primjena: Pogodan za upravljanje i automatizaciju složenih podatkovnih tokova, ETL zadataka, i tijekova rada u strojnom učenju.
Ključne Značajke:
Prilagodljiv tijek rada definiran kodom (Python).
Snažan nadzor i praćenje.
Skalabilnost kroz izvršne radnike (workers).
5. Apache Camel
Namjena: Camel je alat za integraciju koji omogućava definiranje pravila za usmjeravanje i transformaciju podataka putem Enterprise Integration Patterns (EIP).
Primjena: Koristi se za integraciju aplikacija, prebacivanje podataka između sustava, i orkestraciju servisa.
Ključne Značajke:
Podrška za velik broj protokola i API-ja.
Jednostavno definiranje pravila za usmjeravanje.
Integracija s različitim sustavima i alatima.
Zaključak
Svaki od ovih alata ima svoje prednosti i primjene, te se mogu koristiti u različitim scenarijima ovisno o specifičnim potrebama. Apache NiFi se ističe po svojoj sposobnosti da vizualno upravlja podatkovnim tokovima i podržava širok raspon izvora i odredišta podataka. Ostali alati, kao što su Apache Flume, Kafka, Storm, Airflow i Camel, nude različite pristupe integraciji i obradi podataka, omogućujući korisnicima da odaberu najbolji alat za svoje specifične potrebe.
