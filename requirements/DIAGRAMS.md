# Registar dijagrama — sloj zahtjeva

<!-- Generated from the register tree by tools/scripts/diagrams.py; do not edit. -->

Što je nacrtano i što je **rezervirano**: stupac *Nedostaje* nosi ime datoteke koje će
poveznica dobiti kad dijagram bude generiran. Dijagram je **pogled** s deklariranim
gledištem i publikom, nikad izvor istine (D-13, ISO/IEC/IEEE 42010).

> **Očekivani skup pogleda je prijedlog, ne prihvaćena norma** (D-03). Danas glasi:
> `StRS` — kontekst (context), korisnički slučajevi (usecase) i tok (activity); `SyRS` —
> struktura (class) i tok (sequence ili activity); `NFRQ` — nijedan obavezno, jer su njegovi kriteriji mjerni, a ne strukturni.
> Kriterij živi u `tools/scripts/diagrams.py` (`EXPECTED`); proširenje ide uz dokumentiranu izmjenu.
> Dijagram je ili `.puml` datoteka u `uml/`, ili blok `plantuml` unutar samog zapisa (*inline*);
> registar čita oba, a zapis smije imati oba.

Provjera: `python3 tools/scripts/diagrams.py check` — javlja razilaženje ovog prikaza sa stablom
i svaki `.puml` koji nijedan zapis ne prisvaja.

## `01-StRS` — StRS

16 zapisa, 16 s barem jednim dijagramom.

| Zapis | Naslov | Imamo | Nedostaje |
|---|---|---|---|
| [StRS-BLACKBOARD](01-StRS/StRS-BLACKBOARD.md) | Blackboard: privremena pohrana dokumenata tijekom obrade | [context (inline)](01-StRS/StRS-BLACKBOARD.md) · [usecase (inline)](01-StRS/StRS-BLACKBOARD.md) · [state (inline)](01-StRS/StRS-BLACKBOARD.md) · [activity (inline)](01-StRS/StRS-BLACKBOARD.md) | — |
| [StRS-CONNECTION](01-StRS/StRS-CONNECTION.md) | Generička konekcija: jedina granica prema vanjskom sustavu | [context (inline)](01-StRS/StRS-CONNECTION.md) · [usecase (inline)](01-StRS/StRS-CONNECTION.md) · [state (inline)](01-StRS/StRS-CONNECTION.md) · [activity (inline)](01-StRS/StRS-CONNECTION.md) | — |
| [StRS-DOCUMENT](01-StRS/StRS-DOCUMENT.md) | Dokument: jedinica podataka koja putuje tokom | [context (inline)](01-StRS/StRS-DOCUMENT.md) · [usecase (inline)](01-StRS/StRS-DOCUMENT.md) · [activity (inline)](01-StRS/StRS-DOCUMENT.md) | — |
| [StRS-DRIVER](01-StRS/StRS-DRIVER.md) | Driver: jedinstven pristup vanjskim resursima | [context (inline)](01-StRS/StRS-DRIVER.md) · [usecase (inline)](01-StRS/StRS-DRIVER.md) · [state (inline)](01-StRS/StRS-DRIVER.md) · [activity (inline)](01-StRS/StRS-DRIVER.md) | — |
| [StRS-FACTORY](01-StRS/StRS-FACTORY.md) | Workflow Factory: sastavljanje toka iz konfiguracije | [context (inline)](01-StRS/StRS-FACTORY.md) · [usecase (inline)](01-StRS/StRS-FACTORY.md) · [activity (inline)](01-StRS/StRS-FACTORY.md) | — |
| [StRS-FAULT-TOLERANCE](01-StRS/StRS-FAULT-TOLERANCE.md) | Otpornost na greške: prijava, odluka i trajan zapis svake greške obrade | [context (inline)](01-StRS/StRS-FAULT-TOLERANCE.md) · [usecase (inline)](01-StRS/StRS-FAULT-TOLERANCE.md) · [state (inline)](01-StRS/StRS-FAULT-TOLERANCE.md) · [activity (inline)](01-StRS/StRS-FAULT-TOLERANCE.md) | — |
| [StRS-MANAGER](01-StRS/StRS-MANAGER.md) | Upravitelji: evidencija konekcija, drivera i procesora po imenu | [context (inline)](01-StRS/StRS-MANAGER.md) · [usecase (inline)](01-StRS/StRS-MANAGER.md) · [activity (inline)](01-StRS/StRS-MANAGER.md) | — |
| [StRS-MEMENTO-STORE](01-StRS/StRS-MEMENTO-STORE.md) | Pohrana snimki: čuvanje stanja između pokretanja | [context (inline)](01-StRS/StRS-MEMENTO-STORE.md) · [usecase (inline)](01-StRS/StRS-MEMENTO-STORE.md) · [activity (inline)](01-StRS/StRS-MEMENTO-STORE.md) | — |
| [StRS-MEMENTO](01-StRS/StRS-MEMENTO.md) | Snimka stanja: nepromjenjiv zapis stanja komponente | [context (inline)](01-StRS/StRS-MEMENTO.md) · [usecase (inline)](01-StRS/StRS-MEMENTO.md) · [activity (inline)](01-StRS/StRS-MEMENTO.md) | — |
| [StRS-PIPELINE](01-StRS/StRS-PIPELINE.md) | Generički pipeline: jedna transformacija nad jednim dokumentom | [context (inline)](01-StRS/StRS-PIPELINE.md) · [usecase (inline)](01-StRS/StRS-PIPELINE.md) · [activity (inline)](01-StRS/StRS-PIPELINE.md) | — |
| [StRS-PROCESSOR](01-StRS/StRS-PROCESSOR.md) | Generički procesor: obrada dokumenta kroz pipelineove | [context (inline)](01-StRS/StRS-PROCESSOR.md) · [usecase (inline)](01-StRS/StRS-PROCESSOR.md) · [state (inline)](01-StRS/StRS-PROCESSOR.md) · [activity (inline)](01-StRS/StRS-PROCESSOR.md) | — |
| [StRS-REPOSITORY](01-StRS/StRS-REPOSITORY.md) | Repozitorij: trajna pohrana dokumenata kroz strategije | [context (inline)](01-StRS/StRS-REPOSITORY.md) · [usecase (inline)](01-StRS/StRS-REPOSITORY.md) · [activity (inline)](01-StRS/StRS-REPOSITORY.md) | — |
| [StRS-SERIALISATION](01-StRS/StRS-SERIALISATION.md) | Granica formata: parser, formater i konverter (generički sloj) | [context (inline)](01-StRS/StRS-SERIALISATION.md) · [usecase (inline)](01-StRS/StRS-SERIALISATION.md) · [activity (inline)](01-StRS/StRS-SERIALISATION.md) | — |
| [StRS-STATE-MACHINE](01-StRS/StRS-STATE-MACHINE.md) | Stroj stanja: dopušteni prijelazi između stanja | [context (inline)](01-StRS/StRS-STATE-MACHINE.md) · [usecase (inline)](01-StRS/StRS-STATE-MACHINE.md) · [activity (inline)](01-StRS/StRS-STATE-MACHINE.md) | — |
| [StRS-STRATEGY](01-StRS/StRS-STRATEGY.md) | Strategija: zamjenjiv način da se obavi jedan posao | [context (inline)](01-StRS/StRS-STRATEGY.md) · [usecase (inline)](01-StRS/StRS-STRATEGY.md) · [activity (inline)](01-StRS/StRS-STRATEGY.md) | — |
| [StRS-WORKFLOW](01-StRS/StRS-WORKFLOW.md) | Workflow: vlasnik glavnog procesa koji pokreće procesore i evidentira greške | [context (inline)](01-StRS/StRS-WORKFLOW.md) · [usecase (inline)](01-StRS/StRS-WORKFLOW.md) · [activity (inline)](01-StRS/StRS-WORKFLOW.md) | — |

## `02-SyRS` — SyRS

26 zapisa, 26 s barem jednim dijagramom.

| Zapis | Naslov | Imamo | Nedostaje |
|---|---|---|---|
| [SyRS-BBD](02-SyRS/SyRS-BBD-blackboard.md) | Blackboard: što sustav mora raditi da dokumenti privremeno žive na zajedničkom platnu | [class (inline)](02-SyRS/SyRS-BBD-blackboard.md) · [state (inline)](02-SyRS/SyRS-BBD-blackboard.md) · [sequence (inline)](02-SyRS/SyRS-BBD-blackboard.md) | — |
| [SyRS-CNC](02-SyRS/SyRS-CNC-connection.md) | Konekcija: što sustav mora raditi na jedinoj granici prema vanjskom sustavu | [class (inline)](02-SyRS/SyRS-CNC-connection.md) · [state (inline)](02-SyRS/SyRS-CNC-connection.md) · [sequence (inline)](02-SyRS/SyRS-CNC-connection.md) | — |
| [SyRS-DCM](02-SyRS/SyRS-DCM-document.md) | Dokument: što sustav mora raditi s jedinicom podataka koja putuje tokom | [class (inline)](02-SyRS/SyRS-DCM-document.md) · [sequence (inline)](02-SyRS/SyRS-DCM-document.md) | — |
| [SyRS-DRV](02-SyRS/SyRS-DRV-driver.md) | Driver: što sustav mora raditi za jedinstven pristup vanjskim resursima | [class (inline)](02-SyRS/SyRS-DRV-driver.md) · [state (inline)](02-SyRS/SyRS-DRV-driver.md) · [sequence (inline)](02-SyRS/SyRS-DRV-driver.md) | — |
| [SyRS-EXC](02-SyRS/SyRS-EXC-exception.md) | Iznimke frameworka: što sustav mora jamčiti o greškama koje komponente dižu | [class (inline)](02-SyRS/SyRS-EXC-exception.md) | `SyRS-EXC-sequence.puml` |
| [SyRS-FCT](02-SyRS/SyRS-FCT-factory.md) | Workflow Factory: što sustav mora raditi da iz konfiguracije sastavi tok | [class (inline)](02-SyRS/SyRS-FCT-factory.md) · [sequence (inline)](02-SyRS/SyRS-FCT-factory.md) | — |
| [SyRS-FLT](02-SyRS/SyRS-FLT-fault-tolerance.md) | Otpornost na greške: što sustav mora raditi da nijedno izvođenje ni neuspjeli dokument ne nestanu bez zapisa | [class (inline)](02-SyRS/SyRS-FLT-fault-tolerance.md) · [state (inline)](02-SyRS/SyRS-FLT-fault-tolerance.md) · [sequence (inline)](02-SyRS/SyRS-FLT-fault-tolerance.md) | — |
| [SyRS-HLP](02-SyRS/SyRS-HLP-helpers.md) | Pomoćnici Attribute i NameHelper: što sustav mora raditi pri provjeri postavki i imenovanju | [class (inline)](02-SyRS/SyRS-HLP-helpers.md) · [sequence (inline)](02-SyRS/SyRS-HLP-helpers.md) | — |
| [SyRS-ITR](02-SyRS/SyRS-ITR-lazy-iterators.md) | Lijeni iteratori: što sustav mora raditi da izvor izgradi tek pri prvom dohvatu | [class (inline)](02-SyRS/SyRS-ITR-lazy-iterators.md) · [sequence (inline)](02-SyRS/SyRS-ITR-lazy-iterators.md) | — |
| [SyRS-MGR](02-SyRS/SyRS-MGR-managers.md) | Upravitelji: što sustav mora raditi da konekcije, driveri i procesori budu dostupni po imenu | [class (inline)](02-SyRS/SyRS-MGR-managers.md) · [sequence (inline)](02-SyRS/SyRS-MGR-managers.md) | — |
| [SyRS-MMN](02-SyRS/SyRS-MMN-memento.md) | Snimka stanja: što sustav mora raditi s nepromjenjivim zapisom stanja | [class (inline)](02-SyRS/SyRS-MMN-memento.md) | `SyRS-MMN-sequence.puml` |
| [SyRS-MMS](02-SyRS/SyRS-MMS-memento-store.md) | Pohrana snimki: što sustav mora raditi da snimka preživi između pokretanja | [class (inline)](02-SyRS/SyRS-MMS-memento-store.md) · [sequence (inline)](02-SyRS/SyRS-MMS-memento-store.md) | — |
| [SyRS-OBS](02-SyRS/SyRS-OBS-observable.md) | Promatrani objekt: što sustav mora raditi da promatrače obavijesti sigurno među nitima | [class (inline)](02-SyRS/SyRS-OBS-observable.md) · [sequence (inline)](02-SyRS/SyRS-OBS-observable.md) | — |
| [SyRS-ORC](02-SyRS/SyRS-ORC-orchestrator.md) | Orkestrator: što sustav mora raditi da pokrene procesore redom ili paralelno i javi događaje | [class (inline)](02-SyRS/SyRS-ORC-orchestrator.md) · [sequence (inline)](02-SyRS/SyRS-ORC-orchestrator.md) | — |
| [SyRS-PPL](02-SyRS/SyRS-PPL-pipeline.md) | Pipeline: što sustav mora raditi da jedan dokument prođe jednu transformaciju | [class (inline)](02-SyRS/SyRS-PPL-pipeline.md) · [sequence (inline)](02-SyRS/SyRS-PPL-pipeline.md) | — |
| [SyRS-PRC](02-SyRS/SyRS-PRC-processor.md) | Procesor: što sustav mora raditi da dokumente obradi jedan po jedan kroz pipelineove | [class (inline)](02-SyRS/SyRS-PRC-processor.md) · [state (inline)](02-SyRS/SyRS-PRC-processor.md) · [sequence (inline)](02-SyRS/SyRS-PRC-processor.md) | — |
| [SyRS-PTN](02-SyRS/SyRS-PTN-root-base.md) | Korijenska baza Wattleflow: što sustav mora jamčiti svakom objektu frameworka | [class (inline)](02-SyRS/SyRS-PTN-root-base.md) | `SyRS-PTN-sequence.puml` |
| [SyRS-RPS](02-SyRS/SyRS-RPS-repository.md) | Repozitorij: što sustav mora raditi da dokument zapiše i pročita kroz strategije | [class (inline)](02-SyRS/SyRS-RPS-repository.md) · [sequence (inline)](02-SyRS/SyRS-RPS-repository.md) | — |
| [SyRS-SCH](02-SyRS/SyRS-SCH-scheduler.md) | Raspoređivač: što sustav mora raditi da pokrene orkestraciju i javi događaje | [class (inline)](02-SyRS/SyRS-SCH-scheduler.md) · [sequence (inline)](02-SyRS/SyRS-SCH-scheduler.md) | — |
| [SyRS-SGT](02-SyRS/SyRS-SGT-singleton.md) | Singleton: što sustav mora raditi da podklasa ima jednu instancu | [class (inline)](02-SyRS/SyRS-SGT-singleton.md) · [sequence (inline)](02-SyRS/SyRS-SGT-singleton.md) | — |
| [SyRS-SMC](02-SyRS/SyRS-SMC-state-machine.md) | Stroj stanja: što sustav mora raditi da izvodi samo dopuštene prijelaze | [class (inline)](02-SyRS/SyRS-SMC-state-machine.md) · [sequence (inline)](02-SyRS/SyRS-SMC-state-machine.md) | — |
| [SyRS-SRL-CNV](02-SyRS/SyRS-SRL-CNV-converter.md) | Konverter: što sustav mora raditi da pretvorbu izvede jednom strategijom | [class (inline)](02-SyRS/SyRS-SRL-CNV-converter.md) · [sequence (inline)](02-SyRS/SyRS-SRL-CNV-converter.md) | — |
| [SyRS-SRL-FMT](02-SyRS/SyRS-SRL-FMT-formatter.md) | Formater: što sustav mora raditi da sadržaj prikaže u izlazu određenog formata | [class (inline)](02-SyRS/SyRS-SRL-FMT-formatter.md) · [sequence (inline)](02-SyRS/SyRS-SRL-FMT-formatter.md) | — |
| [SyRS-SRL-PAR](02-SyRS/SyRS-SRL-PAR-parser.md) | Parser: što sustav mora raditi da čita izvor podataka u sadržaj | [class (inline)](02-SyRS/SyRS-SRL-PAR-parser.md) · [sequence (inline)](02-SyRS/SyRS-SRL-PAR-parser.md) | — |
| [SyRS-STR](02-SyRS/SyRS-STR-strategy.md) | Strategija: što sustav mora raditi da posao izvede kroz jedan način pozivanja | [class (inline)](02-SyRS/SyRS-STR-strategy.md) · [sequence (inline)](02-SyRS/SyRS-STR-strategy.md) | — |
| [SyRS-WRK](02-SyRS/SyRS-WRK-workflow.md) | Workflow: što sustav mora raditi kao vlasnik glavnog procesa | [class (inline)](02-SyRS/SyRS-WRK-workflow.md) · [sequence (inline)](02-SyRS/SyRS-WRK-workflow.md) | — |

## `03-NFRQ` — NFRQ

48 zapisa, 0 s barem jednim dijagramom.

| Zapis | Naslov | Imamo | Nedostaje |
|---|---|---|---|
| [NFRQ-000](03-NFRQ/NFRQ-000-PREGLED.md) | NFRQ pregled — radni popis nalaza | — | — |
| [NFRQ-APX-01](03-NFRQ/NFRQ-APX-01-facet-order.md) | Appendix A: facet order in class nomenclature | — | — |
| [NFRQ-DEF-01](03-NFRQ/NFRQ-DEF-01-common-definitions.md) | Common definitions | — | — |
| [NFRQ-DEF-02](03-NFRQ/NFRQ-DEF-02-measurement-charter.md) | Charter for measurable [M] criteria | — | — |
| [NFRQ-DEF-03](03-NFRQ/NFRQ-DEF-03-comparison-and-boundary-values.md) | Comparison and boundary-value definitions | — | — |
| [NFRQ-DEF-04](03-NFRQ/NFRQ-DEF-04-moment-aware-helper.md) | MomentAwareHelper | — | — |
| [NFRQ-DEF-05](03-NFRQ/NFRQ-DEF-05-moment-naive-helper.md) | MomentNaiveHelper | — | — |
| [NFRQ-DFN-06](03-NFRQ/NFRQ-DFN-06-error-handling-policy.md) | Error-handling policy: the facets every component declares | — | — |
| [NFRQ-FUN-01](03-NFRQ/NFRQ-FUN-01-conversion-fidelity.md) | Conversion fidelity: nothing lost silently, the round trip is a fixed point | — | — |
| [NFRQ-FUN-02](03-NFRQ/NFRQ-FUN-02-recognition-accuracy.md) | Recognition accuracy is measured per engine on a declared reference set | — | — |
| [NFRQ-MEM-01](03-NFRQ/NFRQ-MEM-01-memory.md) | Instance memory: classes that inherit Wattleflow declare __slots__ | — | — |
| [NFRQ-OBS-01](03-NFRQ/NFRQ-OBS-01-audit-levels.md) | Audit levels: meaning, audience and placement | — | — |
| [NFRQ-OBS-02](03-NFRQ/NFRQ-OBS-02-audit-fields.md) | Audit-record fields | — | — |
| [NFRQ-OBS-03](03-NFRQ/NFRQ-OBS-03-audit-ownership-volume.md) | Ownership, order and volume of audit [M] | — | — |
| [NFRQ-OBS-04](03-NFRQ/NFRQ-OBS-04-metric-admissibility.md) | What a metric must carry to be admissible | — | — |
| [NFRQ-ORG-01](03-NFRQ/NFRQ-ORG-01-helper-locality.md) | Dependency locality of helper classes | — | — |
| [NFRQ-ORG-02](03-NFRQ/NFRQ-ORG-02-class-nomenclature.md) | Class nomenclature | — | — |
| [NFRQ-ORG-03](03-NFRQ/NFRQ-ORG-03-typevar-nomenclature.md) | Type-variable nomenclature | — | — |
| [NFRQ-ORG-04](03-NFRQ/NFRQ-ORG-04-crosscutting-capability.md) | Cross-cutting capability as a helper | — | — |
| [NFRQ-ORG-05](03-NFRQ/NFRQ-ORG-05-self-referencing-helpers.md) | Encapsulating self-referencing helper methods | — | — |
| [NFRQ-ORG-07](03-NFRQ/NFRQ-ORG-07-preset-allowed-declaration.md) | Component input surface (ALLOWED) | — | — |
| [NFRQ-ORG-08](03-NFRQ/NFRQ-ORG-08-deduplication.md) | Deduplication: one place per rule, not per shape | — | — |
| [NFRQ-ORG-09](03-NFRQ/NFRQ-ORG-09-external-standard.md) | A specialisation over an external standard exposes its model | — | — |
| [NFRQ-ORG-10](03-NFRQ/NFRQ-ORG-10-learned-artefact.md) | A learned artefact as a decision criterion | — | — |
| [NFRQ-ORG-11](03-NFRQ/NFRQ-ORG-11-classes-over-module-functions.md) | The class is the unit of code; a module function is a declared exception | — | — |
| [NFRQ-ORG-12](03-NFRQ/NFRQ-ORG-12-constants-and-enumerations.md) | Constants and enumerations live in their designated modules | — | — |
| [NFRQ-ORG-13](03-NFRQ/NFRQ-ORG-13-public-surface-is-the-interface.md) | The public surface of a contract class is its interface | — | — |
| [NFRQ-ORG-14](03-NFRQ/NFRQ-ORG-14-consumer-reaches-the-interface.md) | A consumer reaches a converter only through its interface | — | — |
| [NFRQ-ORG-15](03-NFRQ/NFRQ-ORG-15-separation-of-concerns.md) | Separation of concerns: each component owns its rules | — | — |
| [NFRQ-PRF-01](03-NFRQ/NFRQ-PRF-01-work-proportional-to-input.md) | Work proportional to input | — | — |
| [NFRQ-PRF-02](03-NFRQ/NFRQ-PRF-02-measurement-speed.md) | Measurement speed: the cost of measuring is bounded by the work measured | — | — |
| [NFRQ-SEC-01](03-NFRQ/NFRQ-SEC-01-blast-radius.md) | Blast-radius containment | — | — |
| [NFRQ-SEC-02](03-NFRQ/NFRQ-SEC-02-attack-surface.md) | Attack-surface minimality | — | — |
| [NFRQ-SEC-03](03-NFRQ/NFRQ-SEC-03-supply-chain-locality.md) | Supply-chain trust and distribution locality | — | — |
| [NFRQ-SEC-04](03-NFRQ/NFRQ-SEC-04-detection-operating-point.md) | Detection operating point and psychological acceptability | — | — |
| [NFRQ-SEC-05](03-NFRQ/NFRQ-SEC-05-adversary-model.md) | Adversary model and investment discipline | — | — |
| [NFRQ-SEC-06](03-NFRQ/NFRQ-SEC-06-audit-confidentiality.md) | Audit-record confidentiality | — | — |
| [NFRQ-SEC-07](03-NFRQ/NFRQ-SEC-07-emission-authorisation.md) | Emission is an authorised act, not a configuration key | — | — |
| [NFRQ-SEC-08](03-NFRQ/NFRQ-SEC-08-emitted-document-confidentiality.md) | Emitted-document confidentiality: a declared marking, no residue | — | — |
| [NFRQ-SEC-09](03-NFRQ/NFRQ-SEC-09-no-secret-in-version-control.md) | No secret in version control | — | — |
| [NFRQ-SEC-10](03-NFRQ/NFRQ-SEC-10-secrets-enter-at-the-process-boundary.md) | A secret enters at the process boundary and never leaves through a record | — | — |
| [NFRQ-SEC-11](03-NFRQ/NFRQ-SEC-11-instance-integrity.md) | Integrity of a deployed instance | — | — |
| [NFRQ-SEC-12](03-NFRQ/NFRQ-SEC-12-instance-availability.md) | Availability of a supporting instance | — | — |
| [NFRQ-SEC-13](03-NFRQ/NFRQ-SEC-13-network-exposure-and-least-privilege.md) | Network exposure and least privilege of an instance | — | — |
| [NFRQ-SEC-14](03-NFRQ/NFRQ-SEC-14-telemetry-confidentiality.md) | Telemetry confidentiality at the export boundary | — | — |
| [NFRQ-SEC-15](03-NFRQ/NFRQ-SEC-15-operation-authorisation.md) | Data-plane operation authorisation is declared configuration, not a call-time flag | — | — |
| [NFRQ-SEC-16](03-NFRQ/NFRQ-SEC-16-no-runtime-installation.md) | Nothing is installed or acquired from code at run time | — | — |
| [NFRQ-SEC-17](03-NFRQ/NFRQ-SEC-17-deferred-third-party-loading.md) | Third-party code is loaded at use, never at import | — | — |
