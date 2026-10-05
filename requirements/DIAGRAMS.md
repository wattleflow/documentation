# Registar dijagrama — sloj zahtjeva

<!-- Generated from the register tree by tools/diagrams.py; do not edit. -->

Što je nacrtano i što je **rezervirano**: stupac *Nedostaje* nosi ime datoteke koje će
poveznica dobiti kad dijagram bude generiran. Dijagram je **pogled** s deklariranim
gledištem i publikom, nikad izvor istine (D-13, ISO/IEC/IEEE 42010).

> **Očekivani skup pogleda je prijedlog, ne prihvaćena norma** (D-03). Danas glasi:
> `HLRQ` — dekompozicija na svoju djecu; `FRQ` — struktura (class) i tok (sequence ili
> activity); `NFRQ` — nijedan obavezno, jer su njegovi kriteriji mjerni, a ne strukturni.
> Kriterij živi u `tools/diagrams.py` (`EXPECTED`); proširenje ide uz dokumentiranu izmjenu.
> Dijagram je ili `.puml` datoteka u `uml/`, ili blok `plantuml` unutar samog zapisa (*inline*);
> registar čita oba, a zapis smije imati oba.

Provjera: `python3 tools/diagrams.py check` — javlja razilaženje ovog prikaza sa stablom
i svaki `.puml` koji nijedan zapis ne prisvaja.

## `01-HLRQ` — HLRQ

2 zapisa, 1 s barem jednim dijagramom.

| Zapis | Naslov | Imamo | Nedostaje |
|---|---|---|---|
| [HLRQ-01](01-HLRQ/HLRQ-01-GENERIC-LAYER.md) | Cjevovodna obrada kroz zamjenjive primitive (generički sloj) | — | `HLRQ-01-decomposition.puml` |
| [HLRQ-10](01-HLRQ/HLRQ-10-SERIALISATION.md) | Granica formata: parser, formater i konverter (generički sloj) | [class (inline)](01-HLRQ/HLRQ-10-SERIALISATION.md) | `HLRQ-10-decomposition.puml` |

## `02-FRQ` — FRQ

24 zapisa, 24 s barem jednim dijagramom.

| Zapis | Naslov | Imamo | Nedostaje |
|---|---|---|---|
| [FRQ-AUD-01](02-FRQ/FRQ-AUD-01-audit-record-path.md) | Put audit zapisa od komponente do spremišta | [class (inline)](02-FRQ/FRQ-AUD-01-audit-record-path.md) · [context (inline)](02-FRQ/FRQ-AUD-01-audit-record-path.md) · [usecase (inline)](02-FRQ/FRQ-AUD-01-audit-record-path.md) · [sequence (inline)](02-FRQ/FRQ-AUD-01-audit-record-path.md) · [activity (inline)](02-FRQ/FRQ-AUD-01-audit-record-path.md) | — |
| [FRQ-BBD](02-FRQ/FRQ-BBD-blackboard.md) | GenericBlackboard | [class (inline)](02-FRQ/FRQ-BBD-blackboard.md) · [context (inline)](02-FRQ/FRQ-BBD-blackboard.md) · [usecase (inline)](02-FRQ/FRQ-BBD-blackboard.md) · [sequence (inline)](02-FRQ/FRQ-BBD-blackboard.md) · [activity (inline)](02-FRQ/FRQ-BBD-blackboard.md) · [activity (inline)](02-FRQ/FRQ-BBD-blackboard.md) · [state (inline)](02-FRQ/FRQ-BBD-blackboard.md) | — |
| [FRQ-CON](02-FRQ/FRQ-CON-connection.md) | GenericConnection | [class (inline)](02-FRQ/FRQ-CON-connection.md) · [context (inline)](02-FRQ/FRQ-CON-connection.md) · [usecase (inline)](02-FRQ/FRQ-CON-connection.md) · [sequence (inline)](02-FRQ/FRQ-CON-connection.md) · [activity (inline)](02-FRQ/FRQ-CON-connection.md) · [state (inline)](02-FRQ/FRQ-CON-connection.md) | — |
| [FRQ-DOC](02-FRQ/FRQ-DOC-document.md) | Dokument, adapter i fasada | [class (inline)](02-FRQ/FRQ-DOC-document.md) · [context (inline)](02-FRQ/FRQ-DOC-document.md) · [usecase (inline)](02-FRQ/FRQ-DOC-document.md) · [sequence (inline)](02-FRQ/FRQ-DOC-document.md) · [activity (inline)](02-FRQ/FRQ-DOC-document.md) | — |
| [FRQ-DRV](02-FRQ/FRQ-DRV-driver.md) | Generički driver i odgođeni proxy | [class (inline)](02-FRQ/FRQ-DRV-driver.md) · [context (inline)](02-FRQ/FRQ-DRV-driver.md) · [usecase (inline)](02-FRQ/FRQ-DRV-driver.md) · [sequence (inline)](02-FRQ/FRQ-DRV-driver.md) · [activity (inline)](02-FRQ/FRQ-DRV-driver.md) · [state (inline)](02-FRQ/FRQ-DRV-driver.md) | — |
| [FRQ-HLP](02-FRQ/FRQ-HLP-helpers.md) | Pomoćnici za atribute i imena | [class (inline)](02-FRQ/FRQ-HLP-helpers.md) · [context (inline)](02-FRQ/FRQ-HLP-helpers.md) · [usecase (inline)](02-FRQ/FRQ-HLP-helpers.md) · [sequence (inline)](02-FRQ/FRQ-HLP-helpers.md) · [activity (inline)](02-FRQ/FRQ-HLP-helpers.md) | — |
| [FRQ-ITR](02-FRQ/FRQ-ITR-lazy-iterators.md) | Lijeni iteratori | [class (inline)](02-FRQ/FRQ-ITR-lazy-iterators.md) · [context (inline)](02-FRQ/FRQ-ITR-lazy-iterators.md) · [usecase (inline)](02-FRQ/FRQ-ITR-lazy-iterators.md) · [sequence (inline)](02-FRQ/FRQ-ITR-lazy-iterators.md) · [activity (inline)](02-FRQ/FRQ-ITR-lazy-iterators.md) | — |
| [FRQ-MEM](02-FRQ/FRQ-MEM-memento.md) | Snimka stanja (memento) | [class (inline)](02-FRQ/FRQ-MEM-memento.md) · [context (inline)](02-FRQ/FRQ-MEM-memento.md) · [usecase (inline)](02-FRQ/FRQ-MEM-memento.md) · [sequence (inline)](02-FRQ/FRQ-MEM-memento.md) · [sequence (inline)](02-FRQ/FRQ-MEM-memento.md) · [activity (inline)](02-FRQ/FRQ-MEM-memento.md) | — |
| [FRQ-MGR](02-FRQ/FRQ-MGR-managers.md) | Upravitelji konekcija, drivera i procesora | [class (inline)](02-FRQ/FRQ-MGR-managers.md) · [context (inline)](02-FRQ/FRQ-MGR-managers.md) · [usecase (inline)](02-FRQ/FRQ-MGR-managers.md) · [sequence (inline)](02-FRQ/FRQ-MGR-managers.md) · [activity (inline)](02-FRQ/FRQ-MGR-managers.md) | — |
| [FRQ-OBS](02-FRQ/FRQ-OBS-observable.md) | Promatrani objekt (nit-siguran) | [class (inline)](02-FRQ/FRQ-OBS-observable.md) · [context (inline)](02-FRQ/FRQ-OBS-observable.md) · [usecase (inline)](02-FRQ/FRQ-OBS-observable.md) · [sequence (inline)](02-FRQ/FRQ-OBS-observable.md) · [activity (inline)](02-FRQ/FRQ-OBS-observable.md) | — |
| [FRQ-ORC](02-FRQ/FRQ-ORC-orchestrator.md) | Orkestrator | [class (inline)](02-FRQ/FRQ-ORC-orchestrator.md) · [context (inline)](02-FRQ/FRQ-ORC-orchestrator.md) · [usecase (inline)](02-FRQ/FRQ-ORC-orchestrator.md) · [sequence (inline)](02-FRQ/FRQ-ORC-orchestrator.md) · [activity (inline)](02-FRQ/FRQ-ORC-orchestrator.md) | — |
| [FRQ-PIP](02-FRQ/FRQ-PIP-pipeline.md) | Generički pipeline | [class (inline)](02-FRQ/FRQ-PIP-pipeline.md) · [context (inline)](02-FRQ/FRQ-PIP-pipeline.md) · [usecase (inline)](02-FRQ/FRQ-PIP-pipeline.md) · [sequence (inline)](02-FRQ/FRQ-PIP-pipeline.md) · [activity (inline)](02-FRQ/FRQ-PIP-pipeline.md) | — |
| [FRQ-PRC-01.22](02-FRQ/FRQ-PRC-01.22-document-flow.md) | Tok dokumenta: nastanak i pohrana | [class (inline)](02-FRQ/FRQ-PRC-01.22-document-flow.md) · [context (inline)](02-FRQ/FRQ-PRC-01.22-document-flow.md) · [usecase (inline)](02-FRQ/FRQ-PRC-01.22-document-flow.md) · [sequence (inline)](02-FRQ/FRQ-PRC-01.22-document-flow.md) · [activity (inline)](02-FRQ/FRQ-PRC-01.22-document-flow.md) | — |
| [FRQ-PRC](02-FRQ/FRQ-PRC-processor.md) | Generički procesor | [class (inline)](02-FRQ/FRQ-PRC-processor.md) · [context (inline)](02-FRQ/FRQ-PRC-processor.md) · [usecase (inline)](02-FRQ/FRQ-PRC-processor.md) · [sequence (inline)](02-FRQ/FRQ-PRC-processor.md) · [activity (inline)](02-FRQ/FRQ-PRC-processor.md) · [state (inline)](02-FRQ/FRQ-PRC-processor.md) | — |
| [FRQ-PTN](02-FRQ/FRQ-PTN-root-base.md) | Korijenska baza Wattleflow | [class (inline)](02-FRQ/FRQ-PTN-root-base.md) · [context (inline)](02-FRQ/FRQ-PTN-root-base.md) · [usecase (inline)](02-FRQ/FRQ-PTN-root-base.md) · [sequence (inline)](02-FRQ/FRQ-PTN-root-base.md) · [activity (inline)](02-FRQ/FRQ-PTN-root-base.md) | — |
| [FRQ-REP](02-FRQ/FRQ-REP-repository.md) | Generičko spremište i varijanta s driverom | [class (inline)](02-FRQ/FRQ-REP-repository.md) · [context (inline)](02-FRQ/FRQ-REP-repository.md) · [usecase (inline)](02-FRQ/FRQ-REP-repository.md) · [sequence (inline)](02-FRQ/FRQ-REP-repository.md) · [activity (inline)](02-FRQ/FRQ-REP-repository.md) | — |
| [FRQ-SCH](02-FRQ/FRQ-SCH-scheduler.md) | Raspoređivač | [class (inline)](02-FRQ/FRQ-SCH-scheduler.md) · [context (inline)](02-FRQ/FRQ-SCH-scheduler.md) · [usecase (inline)](02-FRQ/FRQ-SCH-scheduler.md) · [sequence (inline)](02-FRQ/FRQ-SCH-scheduler.md) · [activity (inline)](02-FRQ/FRQ-SCH-scheduler.md) | — |
| [FRQ-SER-CNV](02-FRQ/FRQ-SER-CNV-converter.md) | Generički konverter | [class (inline)](02-FRQ/FRQ-SER-CNV-converter.md) · [context (inline)](02-FRQ/FRQ-SER-CNV-converter.md) · [usecase (inline)](02-FRQ/FRQ-SER-CNV-converter.md) · [sequence (inline)](02-FRQ/FRQ-SER-CNV-converter.md) · [activity (inline)](02-FRQ/FRQ-SER-CNV-converter.md) | — |
| [FRQ-SER-FMT](02-FRQ/FRQ-SER-FMT-formatter.md) | Generički formater | [class (inline)](02-FRQ/FRQ-SER-FMT-formatter.md) · [context (inline)](02-FRQ/FRQ-SER-FMT-formatter.md) · [usecase (inline)](02-FRQ/FRQ-SER-FMT-formatter.md) · [sequence (inline)](02-FRQ/FRQ-SER-FMT-formatter.md) · [activity (inline)](02-FRQ/FRQ-SER-FMT-formatter.md) | — |
| [FRQ-SER-PAR](02-FRQ/FRQ-SER-PAR-parser.md) | Generički parser | [class (inline)](02-FRQ/FRQ-SER-PAR-parser.md) · [context (inline)](02-FRQ/FRQ-SER-PAR-parser.md) · [usecase (inline)](02-FRQ/FRQ-SER-PAR-parser.md) · [sequence (inline)](02-FRQ/FRQ-SER-PAR-parser.md) · [activity (inline)](02-FRQ/FRQ-SER-PAR-parser.md) | — |
| [FRQ-SGT](02-FRQ/FRQ-SGT-singleton.md) | Singleton | [class (inline)](02-FRQ/FRQ-SGT-singleton.md) · [context (inline)](02-FRQ/FRQ-SGT-singleton.md) · [usecase (inline)](02-FRQ/FRQ-SGT-singleton.md) · [sequence (inline)](02-FRQ/FRQ-SGT-singleton.md) · [activity (inline)](02-FRQ/FRQ-SGT-singleton.md) | — |
| [FRQ-SMC](02-FRQ/FRQ-SMC-state-machine.md) | Automat stanja | [class (inline)](02-FRQ/FRQ-SMC-state-machine.md) · [context (inline)](02-FRQ/FRQ-SMC-state-machine.md) · [usecase (inline)](02-FRQ/FRQ-SMC-state-machine.md) · [sequence (inline)](02-FRQ/FRQ-SMC-state-machine.md) · [activity (inline)](02-FRQ/FRQ-SMC-state-machine.md) | — |
| [FRQ-STR](02-FRQ/FRQ-STR-strategy.md) | Obitelj strategija | [class (inline)](02-FRQ/FRQ-STR-strategy.md) · [context (inline)](02-FRQ/FRQ-STR-strategy.md) · [usecase (inline)](02-FRQ/FRQ-STR-strategy.md) · [sequence (inline)](02-FRQ/FRQ-STR-strategy.md) · [activity (inline)](02-FRQ/FRQ-STR-strategy.md) | — |
| [FRQ-WFL](02-FRQ/FRQ-WFL-workflow.md) | Generički workflow i tvornica | [class (inline)](02-FRQ/FRQ-WFL-workflow.md) · [context (inline)](02-FRQ/FRQ-WFL-workflow.md) · [usecase (inline)](02-FRQ/FRQ-WFL-workflow.md) · [sequence (inline)](02-FRQ/FRQ-WFL-workflow.md) · [activity (inline)](02-FRQ/FRQ-WFL-workflow.md) | — |

## `03-NFRQ` — NFRQ

43 zapisa, 0 s barem jednim dijagramom.

| Zapis | Naslov | Imamo | Nedostaje |
|---|---|---|---|
| [NFRQ-APX-01](03-NFRQ/NFRQ-APX-01-facet-order.md) | Appendix A: facet order in class nomenclature | — | — |
| [NFRQ-DEF-01](03-NFRQ/NFRQ-DEF-01-common-definitions.md) | Common definitions | — | — |
| [NFRQ-DEF-02](03-NFRQ/NFRQ-DEF-02-measurement-charter.md) | Charter for measurable [M] criteria | — | — |
| [NFRQ-DEF-03](03-NFRQ/NFRQ-DEF-03-comparison-and-boundary-values.md) | Comparison and boundary-value definitions | — | — |
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
