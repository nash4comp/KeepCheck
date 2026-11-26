# Tests

Implementation begins in Unit 4. The order below is deliberate: the first test
guards the design decision that everything else depends on.

## Planned checks

| Test                                      | Guards                                                                    | Requirement   |
| ----------------------------------------- | ------------------------------------------------------------------------- | ------------- |
| `test_original_store_immutable`           | The original store is byte-identical before and after a consolidation run | NFR-03, FR-06 |
| `test_pipeline_cannot_read_labels`        | The pipeline identity fails when it attempts to open `fixtures/labels/`   | NFR-06        |
| `test_deletion_leaves_tombstone`          | No item disappears without a tombstone record                             | FR-06, NFR-04 |
| `test_provenance_present_on_every_item`   | Every stored item carries its source session and trust level              | FR-07         |
| `test_run_aborts_on_budget_breach`        | A run exceeding its token or iteration ceiling stops                      | FR-12         |
| `test_scorer_fails_run_on_must_keep_loss` | Losing a must-keep item fails the run rather than lowering a score        | FR-08         |

## Why immutability is tested first

The whole safety argument rests on consolidation touching only a copy. If that
is ever violated, a bad run becomes unrecoverable and the project loses the one
property that makes the experiment safe to repeat. The check is a hash comparison
of the original store taken before and after each run, and it runs on every
execution rather than only in the test suite.

## Why label isolation is a test, not a convention

A convention can be broken by a later refactor without anyone noticing. A failing
test makes the breach visible at the moment it is introduced. If the pipeline
could read which items are marked must-keep, it would protect exactly those items
and the reported loss rate would approach zero for the wrong reason.
