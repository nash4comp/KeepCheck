# KeepCheck

Measuring the utility, safety, and economics of memory consolidation in LLM agents.

MSIT Capstone Project (MSIT 5910), University of the People.
Author: Inkyung Baek.

## What this project is

An LLM agent with long-term memory accumulates duplicates, contradictions, and
remarks with no reuse value as sessions pile up. Consolidation features that
reorganize such a store already ship commercially, yet no public, reproducible
evidence shows how much must-keep information they destroy, or how many sessions
are needed to repay their cost.

KeepCheck runs a three-stage consolidation pipeline over a synthetic fixture in
which the correct outcome of every planted item is designed in advance, then
measures three things in one controlled experiment:

| Question                 | Metric                                                 |
| ------------------------ | ------------------------------------------------------ |
| Does consolidation work? | Contradiction-resolution accuracy, duplicate reduction |
| Is it safe?              | Critical-information loss rate, reported independently |
| Does it pay for itself?  | Break-even session count from token accounting         |

The project succeeds either way. If consolidation performs well, the break-even
figure becomes an adoption criterion. If it destroys must-keep information, that
loss rate is a warning any organization considering the feature can act on.

## Repository layout

| Path                    | Contents                                                           |
| ----------------------- | ------------------------------------------------------------------ |
| `src/`                  | Module source code. Planned layout is described in `src/README.md` |
| `docs/`                 | Architecture specification and the stub interface contract         |
| `design/`               | Draw.io sources and exported diagrams                              |
| `fixtures/transcripts/` | Synthetic call transcripts                                         |
| `fixtures/labels/`      | Ground-truth labels, deliberately kept on a separate path          |
| `tests/`                | Automated checks, starting with original-store immutability        |
| `results/`              | Run outputs (excluded from version control)                        |

Labels live apart from transcripts because the pipeline identity must not be able
to read which items are marked must-keep. A pipeline that could see those marks
would protect exactly those items and make the measurement worthless. Access
separation therefore begins in the directory structure.

## Status

Design complete; implementation begins in Unit 4.

| Milestone                                   | State       |
| ------------------------------------------- | ----------- |
| Literature survey and research gap          | Done        |
| Proposal, scope, and SMART goals            | Done        |
| Architecture and requirements specification | Done        |
| Git repository and branch setup             | Done        |
| Stub interface contract frozen              | In progress |
| Fixture authoring, 3 then 100 items         | Not started |
| Pipeline and scorer implementation          | Not started |
| Integration, first measurement, break-even  | Not started |

## Reproducing a run

Implementation is not yet available. Once the pipeline lands, a run will require
only a model API key in the environment and a single entry point, so that a third
party can reproduce the measurement from this README alone. That constraint is
recorded as NFR-07 in `docs/architecture.md`.

## Documents

- `docs/architecture.md` — modules, trust boundaries, functional and non-functional requirements
- `docs/interface-contract.md` — stub transcript service contract
- `design/` — system architecture and data flow diagrams

## License

Fixtures and scorer are authored by the project owner and are intended for open
release. A license file will be added when the first implementation lands.
