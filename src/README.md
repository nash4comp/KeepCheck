# Source layout

Implementation begins in Unit 4. This file records the planned layout so that the
module identifiers used in `../docs/architecture.md`, in the design diagrams, and
in commit messages all refer to the same thing.

| Planned package  | Module | Responsibility                                 |
| ---------------- | ------ | ---------------------------------------------- |
| `orchestrator/`  | M1     | Run order, token and iteration ceilings        |
| `fixture_store/` | M2     | Loads transcripts; never loads labels          |
| `stub_service/`  | M3     | Serves transcripts over the fixed contract     |
| `extractor/`     | M4     | Transcript to candidate claims                 |
| `consolidator/`  | M5     | Merge duplicates, resolve contradictions       |
| `pruner/`        | M6     | Mark items with no reuse value                 |
| `store_adapter/` | M7     | Every durable write; copy-on-write, tombstones |
| `scorer/`        | M8     | Per-class metrics against the label set        |
| `cost_meter/`    | M9     | Token accounting and break-even                |
| `dashboard/`     | M10    | Read-only view, loopback binding               |

## Two rules that constrain the code before it exists

**The pipeline identity holds no credential for `fixtures/labels/`.** M2 loads
transcripts only. Label loading belongs to M8 under a separate identity. This is
NFR-06, and it is what keeps the loss-rate measurement meaningful.

**Only M7 performs a durable write.** No other module may touch the store
directly. This is what makes copy-on-write and tombstone deletion impossible to
bypass by changing a prompt or a pruning rule.

## Build order

The dependency chain decides the order of work: the stub contract is frozen
first, then the stub service and the pipeline, then the scorer once the fixture
schema is settled, then integration and measurement.
