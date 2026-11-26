# KeepCheck Architecture Specification

Version 0.1, December 2026. This document is the source of truth for module
identifiers, requirement identifiers, and trust boundary names used across the
project. Diagrams are in `../design/`.

## 1. Overview

KeepCheck has ten modules arranged in four layers plus one control module. The
Orchestrator does not sit in the data path; it only enforces run order and budget
ceilings.

| Layer                       | Modules                                                    |
| --------------------------- | ---------------------------------------------------------- |
| Input                       | M2 Fixture Store, M3 Stub Transcript Service               |
| Consolidation               | M4 Extractor, M5 Consolidator, M6 Pruner                   |
| Storage                     | M7 Memory Store Adapter, original store, consolidated copy |
| Evaluation and presentation | M8 Scorer, M9 Cost Meter, M10 Dashboard                    |
| Control                     | M1 Orchestrator                                            |

## 2. Module specification

| ID  | Module                  | Input                              | Output                                     | Method                                                                                                |
| --- | ----------------------- | ---------------------------------- | ------------------------------------------ | ----------------------------------------------------------------------------------------------------- |
| M1  | Orchestrator            | Run configuration, budget ceilings | Run log, exit code                         | Sequential stage execution; enforces token and iteration ceilings and aborts on breach                |
| M2  | Fixture Store           | None (static asset)                | Synthetic transcripts, ground-truth labels | Transcripts and labels held on physically separate paths; labels readable only by the scorer identity |
| M3  | Stub Transcript Service | Session id                         | Transcript JSON                            | Fixed interface contract standing in for internal systems; deterministic responses                    |
| M4  | Extractor               | One transcript                     | Candidate claim list                       | Low-cost model call with a constrained output schema                                                  |
| M5  | Consolidator            | Claims, existing memory            | Merge and update operations                | Strong model call; merges duplicates, resolves contradictions, records the reason                     |
| M6  | Pruner                  | Consolidation result               | Prune marks                                | Reuse-value criteria; physical deletion is forbidden                                                  |
| M7  | Memory Store Adapter    | Write operations                   | Consolidated copy, change history          | Original is append-only; writes go to a copy; deletions are tombstones                                |
| M8  | Scorer                  | Consolidated copy, labels          | Per-class metrics                          | Automatic comparison by class; a lost must-keep item fails the run                                    |
| M9  | Cost Meter              | API usage responses                | Per-stage cost, break-even                 | Aggregates against published rates and compares with per-session savings                              |
| M10 | Dashboard               | Metrics, cost                      | Read-only view                             | Loopback binding with token authentication                                                            |

## 3. Trust boundaries

| ID  | Where                            | Why it carries risk                                                  |
| --- | -------------------------------- | -------------------------------------------------------------------- |
| B1  | M3 to M4                         | Transcript text that no KeepCheck component produced enters a prompt |
| B2  | M6 to M7                         | Model output becomes a durable write                                 |
| B3  | M2 labels to M8                  | Ground-truth labels must reach the scorer and nothing else           |
| B4  | M4, M5, M6 to the model provider | Prompt content and credentials leave the host                        |

Every safeguard is placed at one of these four points rather than at the
application edge, because an edge control cannot protect a boundary that sits
inside the pipeline.

## 4. Functional requirements

| ID    | Requirement                                                                                                             |
| ----- | ----------------------------------------------------------------------------------------------------------------------- |
| FR-01 | The system retrieves synthetic transcripts by session through a fixed interface                                         |
| FR-02 | The extractor produces a structured list of candidate claims from a transcript                                          |
| FR-03 | The consolidator merges claims that carry the same meaning                                                              |
| FR-04 | The consolidator decides which of two contradicting claims is current and records the reason                            |
| FR-05 | The pruner marks items with no reuse value for discard                                                                  |
| FR-06 | The store adapter writes only to a copy and never modifies the original                                                 |
| FR-07 | Every memory item is stored with its provenance and a trust level                                                       |
| FR-08 | The scorer reports contradiction-resolution accuracy, duplicate reduction, critical-information loss, and noise pruning |
| FR-09 | The cost meter reports per-stage cost and a break-even session count                                                    |
| FR-10 | The system answers the same queries from the pre- and post-consolidation stores for comparison                          |
| FR-11 | The dashboard presents metrics and cost on one screen                                                                   |
| FR-12 | The orchestrator enforces per-run token and iteration ceilings and aborts on breach                                     |

## 5. Non-functional requirements

| ID     | Category    | Requirement                                                             | Design consequence                                                                         |
| ------ | ----------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| NFR-01 | Performance | One consolidation pass over 100 transcripts completes within 30 minutes | Parallel extraction stage, per-stage caching                                               |
| NFR-02 | Cost        | A single run stays within its cost ceiling                              | Task-appropriate model routing: low-cost for extraction, strong for contradiction judgment |
| NFR-03 | Reliability | Original-store immutability verified on every run                       | Append-only original, automatic hash comparison                                            |
| NFR-04 | Reliability | Every write can be rolled back                                          | Copy-on-write and tombstone deletion                                                       |
| NFR-05 | Security    | No credential is present in the repository                              | Environment-variable injection, pre-commit secret scan                                     |
| NFR-06 | Security    | Least-privilege separation                                              | Three service identities: pipeline, scorer, dashboard                                      |
| NFR-07 | Usability   | A third party can re-run from the README alone                          | Fixed interface, externalized configuration                                                |
| NFR-08 | Scalability | Growing the fixture from 100 to 1,000 items requires no code change     | Schema-driven loader, batch processing                                                     |
| NFR-09 | Fairness    | Loss rates reported separately by stratum                               | Fixture items carry an orthography stratum                                                 |

## 6. Design decisions

**Internal systems are replaced by a stub.** No external approval becomes a
schedule precondition, and no company data enters the system.

**Ground-truth labels sit on a path the pipeline cannot read.** If the pipeline
could see which items are marked must-keep, it would protect exactly those items
and the loss-rate measurement would be meaningless. Separating the credential
makes that failure impossible rather than merely discouraged.

**Deletion is a tombstone, not a physical removal.** Consolidation errors can be
reverted, and every deletion remains auditable.

All three controls live in the storage layer, so a later change to a prompt or a
pruning heuristic cannot bypass them.

## 7. Fairness as measurement, not protection

Must-keep items are split into two strata, conventional and unconventional
transcription, and loss is reported for each separately. The stratum marker is
never exposed to the pipeline. Marking the sensitive stratum inside the pipeline
would let the system protect it and hide the very disparity the project exists to
detect.

## 8. Fixture composition

| Class              | Count | Correct outcome      | Metric                            |
| ------------------ | ----- | -------------------- | --------------------------------- |
| Contradiction pair | 20    | Retain the valid one | Contradiction-resolution accuracy |
| Duplicate pair     | 20    | Merge into one       | Duplicate reduction               |
| Noise              | 30    | Discard              | Noise pruning                     |
| Must-keep          | 30    | Must survive         | Critical-information loss rate    |
