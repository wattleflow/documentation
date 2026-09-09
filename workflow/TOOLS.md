# WEM-LINT-ANALIZA

**Mjerenje modularnosti i sigurnosnih kompromisa: od mjere stanja do mjere odluke**

---

## 1. Apstrakt
Arhitektonska granica u programskom sustavu ne može se obraniti kao jedinstveni skalarni indeks; umjesto njega ova analiza predlaže vektorski profil s Pareto-analizom [[16]](https://doi.org/10.1126/science.103.2684.677). Razlog nije nedostatna preciznost postojećih pokazatelja, nego njihova nesvodivost na zajedničku ljestvicu [[15]](https://openlibrary.org/isbn/9780124254015). Rad razdvaja dvije vrste mjere: mjera stanja opisuje zatečenu strukturu artefakta i za nju je artefakt dovoljan izvor podataka [11], dok mjera odluke procjenjuje isplativost granice pod neizvjesnošću i traži razdiobu vjerojatnosti nad budućim stanjima svijeta [[6]](https://mitpress.mit.edu/9780262024662/design-rules/). Prva je jednostavnija zbog podatkovnih zahtjeva, ne po naravi. Od razmotrenih veličina svojstva istinskog postotka imaju samo tri: propagation cost [[11]](https://doi.org/10.1016/j.respol.2012.04.011), normalizirana uzajamna informacija između particija [[20]](https://doi.org/10.1088/1742-5468/2005/09/P09008) i Gordon–Loebova granica ulaganja [[26]](https://doi.org/10.1145/581271.581274). Pojam volatilnosti pokriva tri različita fenomena, od kojih strateški protivnik poništava pretpostavke opcijskih modela [[32]](https://doi.org/10.1007/978-3-642-12586-7).. Napetost između sigurnosti i udobnosti pokazuje se kao izbor radne točke klasifikatora, a ne kao pitanje kvalitete mehanizma [[31]](https://doi.org/10.1109/TIT.1954.1057460).


## 2. Uvod i glavna pitanja
Klasični kriteriji modularnosti određuju granicu kvalitativno, ali nijedan ne propisuje skalu na kojoj bi se stupanj modularnosti izrazio. Kvantifikacija je dopuštena tek ako mjera zadovoljava reprezentacijski uvjet, odnosno ako je
homomorfizam iz empirijskog relacijskog sustava u numerički [[15]](https://openlibrary.org/isbn/9780124254015). Bez prethodno definirane relacije oblika „A je modularniji od B" brojevi postoje, ali skala ne postoji [[19]](https://doi.org/10.1145/336512.336588). Aksiomatski okviri za vrednovanje metrika složenosti [[17]](https://doi.org/10.1109/32.4634) i za svojstveno utemeljeno mjerenje sprezanja i kohezije [[18]](https://doi.org/10.1109/32.481535) pokazuju da velik dio
raširenih metrika taj uvjet ne zadovoljava.

Četiri polazna kriterija razlikuju se po tome na što vežu granicu. Parnas je veže uz odluku koja se može promijeniti: modul nije skupina srodnih funkcija, nego granica oko takve odluke [[1]](https://doi.org/10.1145/361598.361623). Simon je veže uz gustoću interakcija, koja je
unutar dijela znatno veća nego između dijelova [[2]](https://www.jstor.org/stable/985254). Strukturirani dizajn veže je uz koheziju i sprezanje kao mjerljive kategorije [[3]](https://doi.org/10.1147/sj.132.0115). Kognitivni pristup veže je
uz ograničenja radne memorije čitatelja [[4]](https://doi.org/10.1037/h0043158)[[5]](https://doi.org/10.1207/s15516709cog1202_4).

Iz razmaka između kvalitativnog kriterija i mjerne skale slijede tri pitanja [S]:
1. Može li se stupanj modularnosti izraziti kao postotak i pod kojim uvjetima taj postotak zadržava značenje [[15]](https://openlibrary.org/isbn/9780124254015)?
2. Može li se isplativost pojedine arhitektonske odluke izraziti mjerom usporedivom s mjerom stanja [[6]](https://mitpress.mit.edu/9780262024662/design-rules/)?
3. Je li sukob sigurnosti i udobnosti mjerljiv, i koju ulogu u njemu ima volatilnost [[26]](https://doi.org/10.1145/581271.581274)[[25]](https://doi.org/10.1145/1595676.1595684)?






## 7. Literatura
1. Parnas, D. L. (1972). *On the Criteria To Be Used in Decomposing Systems into Modules*. Communications of the ACM, 15(12), 1053–1058. https://doi.org/10.1145/361598.361623
2. Simon, H. A. (1962). *The Architecture of Complexity*. Proceedings of the American Philosophical Society, 106(6), 467–482. https://www.jstor.org/stable/985254
3. Stevens, W. P., Myers, G. J., & Constantine, L. L. (1974). *Structured Design*. IBM Systems Journal, 13(2), 115–139. https://doi.org/10.1147/sj.132.0115
4. Miller, G. A. (1956). *The Magical Number Seven, Plus or Minus Two*. Psychological Review, 63(2), 81–97. https://doi.org/10.1037/h0043158
5. Sweller, J. (1988). *Cognitive Load During Problem Solving*. Cognitive Science, 12(2), 257–285. https://doi.org/10.1207/s15516709cog1202_4
6. Baldwin, C. Y., & Clark, K. B. (2000). *Design Rules, Vol. 1: The Power of Modularity*. MIT Press. https://mitpress.mit.edu/9780262024662/design-rules/
7. Newman, M. E. J., & Girvan, M. (2004). *Finding and Evaluating Community Structure in Networks*. Physical Review E, 69, 026113. https://doi.org/10.1103/PhysRevE.69.026113
8. Fortunato, S., & Barthélemy, M. (2007). *Resolution Limit in Community Detection*. PNAS, 104(1), 36–41. https://doi.org/10.1073/pnas.0605965104
9. Gall, H., Hajek, K., & Jazayeri, M. (1998). *Detection of Logical Coupling Based on Product Release History*. ICSM 1998. https://doi.org/10.1109/ICSM.1998.738508
10. Zimmermann, T., Weißgerber, P., Diehl, S., & Zeller, A. (2005). *Mining Version Histories to Guide Software Changes*. IEEE TSE, 31(6), 429–445. https://doi.org/10.1109/TSE.2005.72
11. MacCormack, A., Baldwin, C., & Rusnak, J. (2012). *Exploring the Duality Between Product and Organizational Architectures: A Test of the "Mirroring" Hypothesis*. Research Policy, 41(8), 1309–1324. https://doi.org/10.1016/j.respol.2012.04.011
12. Allen, E. B., & Khoshgoftaar, T. M. (1999). *Measuring Coupling and Cohesion: An Information-Theory Approach*. METRICS 1999. https://doi.org/10.1109/METRIC.1999.809743
13. Rissanen, J. (1978). *Modeling by Shortest Data Description*. Automatica, 14(5), 465–471. https://doi.org/10.1016/0005-1098(78)90005-5
14. Cilibrasi, R., & Vitányi, P. M. B. (2005). *Clustering by Compression*. IEEE Transactions on Information Theory, 51(4), 1523–1545. https://doi.org/10.1109/TIT.2005.844059
15. Krantz, D. H., Luce, R. D., Suppes, P., & Tversky, A. (1971). *Foundations of Measurement, Vol. I*. Academic Press. https://openlibrary.org/isbn/9780124254015
16. Stevens, S. S. (1946). *On the Theory of Scales of Measurement*. Science, 103(2684), 677–680. https://doi.org/10.1126/science.103.2684.677
17. Weyuker, E. J. (1988). *Evaluating Software Complexity Measures*. IEEE TSE, 14(9), 1357–1365. https://doi.org/10.1109/32.4634
18. Briand, L. C., Morasca, S., & Basili, V. R. (1996). *Property-Based Software Engineering Measurement*. IEEE TSE, 22(1), 68–86. https://doi.org/10.1109/32.481535
19. Fenton, N. E., & Neil, M. (2000). *Software Metrics: Roadmap*. ICSE 2000 — Future of Software Engineering. https://doi.org/10.1145/336512.336588
20. Danon, L., Díaz-Guilera, A., Duch, J., & Arenas, A. (2005). *Comparing Community Structure Identification*. Journal of Statistical Mechanics, P09008. https://doi.org/10.1088/1742-5468/2005/09/P09008
21. Hubert, L., & Arabie, P. (1985). *Comparing Partitions*. Journal of Classification, 2, 193–218. https://doi.org/10.1007/BF01908075
22. Campbell, D. T. (1979). *Assessing the Impact of Planned Social Change*. Evaluation and Program Planning, 2(1), 67–90. https://doi.org/10.1016/0149-7189(79)90048-X
23. Saltzer, J. H., & Schroeder, M. D. (1975). *The Protection of Information in Computer Systems*. Proceedings of the IEEE, 63(9), 1278–1308. https://doi.org/10.1109/PROC.1975.9939
24. Adams, A., & Sasse, M. A. (1999). *Users Are Not the Enemy*. Communications of the ACM, 42(12), 40–46. https://doi.org/10.1145/322796.322806
25. Beautement, A., Sasse, M. A., & Wonham, M. (2008). *The Compliance Budget: Managing Security Behaviour in Organisations*. NSPW 2008, 47–58. https://doi.org/10.1145/1595676.1595684
26. Gordon, L. A., & Loeb, M. P. (2002). *The Economics of Information Security Investment*. ACM TISSEC, 5(4), 438–457. https://doi.org/10.1145/581271.581274
27. Manadhata, P. K., & Wing, J. M. (2011). *An Attack Surface Metric*. IEEE TSE, 37(3), 371–386. https://doi.org/10.1109/TSE.2010.60
28. Anderson, R. (2001). *Why Information Security Is Hard — An Economic Perspective*. ACSAC 2001. https://doi.org/10.1109/ACSAC.2001.991552
29. Brooks, F. P. (1987). *No Silver Bullet: Essence and Accidents of Software Engineering*. IEEE Computer, 20(4), 10–19. https://doi.org/10.1109/MC.1987.1663532
30. Ashby, W. R. (1956). *An Introduction to Cybernetics*. Chapman & Hall. http://pespmc1.vub.ac.be/books/IntroCyb.pdf
31. Peterson, W. W., Birdsall, T. G., & Fox, W. C. (1954). *The Theory of Signal Detectability*. IRE Transactions on Information Theory, 4(4), 171–212. https://doi.org/10.1109/TIT.1954.1057460
32. von Stackelberg, H. (2011). *Market Structure and Equilibrium* (prijevod izvornika iz 1934.). Springer. https://doi.org/10.1007/978-3-642-12586-7
33. Siegmund, J., Kästner, C., Apel, S., Parnin, C., Bethmann, A., Leich, T., Saake, G., & Brechmann, A. (2014). *Understanding Understanding Source Code with Functional Magnetic Resonance Imaging*. ICSE 2014, 378–389. https://doi.org/10.1145/2568225.2568252
34. NIST National Vulnerability Database. *CVE-2021-44228 (Log4Shell)*. https://nvd.nist.gov/vuln/detail/CVE-2021-44228
