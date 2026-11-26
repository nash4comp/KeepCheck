# Stub Transcript Service: Interface Contract

Version 0.1, draft. Status: in progress.

This contract is the earliest milestone in the project schedule. The scorer and
the pipeline are both built against it, so changing it late would force rework at
two integration points. It is therefore frozen before any implementation starts.

## 1. Purpose

The Stub Transcript Service (M3) stands in for internal production systems. It
serves synthetic transcripts from the Fixture Store (M2) over a fixed interface.
Because the interface is fixed and the stub is deterministic, the whole pipeline
can run end to end without access to any company system and without any external
approval becoming a schedule precondition.

## 2. Operations

### `get_transcript(session_id) -> Transcript`

Returns the transcript for one session. Raises `SessionNotFound` if the id is not
present in the fixture.

### `list_sessions(caller_id) -> [session_id]`

Returns the session ids belonging to one caller, in ascending timestamp order.
Order is part of the contract: contradiction resolution depends on knowing which
statement came later.

## 3. Transcript schema

```json
{
  "fixture_id": "string",
  "schema_version": "0.1.0",
  "caller_id": "string",
  "session_id": "string",
  "timestamp": "ISO 8601 UTC",
  "turns": [
    { "speaker": "caller | agent", "text": "string" }
  ]
}
```

Constraints:

- `timestamp` is the session start, and is the only ordering signal the pipeline
  may use. Transcript text must not be mined for dates.
- `turns` preserves spoken order.
- The schema carries no label, class, or stratum field. Those live only in the
  label set, behind boundary B3.

## 4. Determinism

The same `session_id` always yields a byte-identical transcript. Without this,
two runs of the pipeline could not be compared, and the before-and-after
measurement would not be attributable to consolidation.

## 5. Error behavior

| Condition | Result |
| --- | --- |
| Unknown `session_id` | `SessionNotFound` |
| Unknown `caller_id` | Empty list, not an error |
| Malformed fixture file | Fail the run; never return a partial transcript |

Failing the run is deliberate. A silently truncated transcript would change the
measurement without any signal that it had done so.

## 6. What this contract deliberately excludes

- No write path. The stub is read-only, so no component can modify fixtures.
- No label access. Label retrieval belongs to the scorer identity alone (NFR-06).
- No network call. The stub reads local files, so B4 is crossed only by the
  consolidation modules.

## 7. Open points before freezing

1. Whether `turns` needs a per-turn timestamp for long sessions.
2. Whether `caller_id` should be opaque or human-readable in fixtures.
3. Whether the schema version belongs in the file or in a manifest.

These are resolved in Unit 3 to 4, after which this document moves from draft to
frozen and the version becomes 1.0.0.
