# Replica Server

[← CA3 overview](../README.md)

Each replica is an independent Go HTTP server with its own map, JSON persistence file, and peer list. The same [main.go](main.go) runs all three instances using different configuration files.

## Start a replica

From **`CA3`**:

```bash
go run replica/main.go -config configs/replica1.json
```

In separate terminals, use `configs/replica2.json` and `configs/replica3.json` for the other instances. See the [quick start](../README.md#quick-start) for the complete launch sequence.

## Configuration

The checked-in replica 1 configuration is:

```json
{
  "id": "replica1",
  "port": 8001,
  "peers": ["http://localhost:8002", "http://localhost:8003"],
  "replication_delay_ms": 2000,
  "data_file": "results/replica1_data.json"
}
```

| Field | Purpose |
| --- | --- |
| `id` | Replica identity and equal-version conflict priority |
| `port` | Local HTTP listening port |
| `peers` | HTTP base URLs of the other replicas |
| `replication_delay_ms` | Sleep before each outgoing replication attempt |
| `data_file` | Persistence path relative to the process's working directory |

Ports are `8001`, `8002`, and `8003`. Keep the data-file parent directory present. Existing data is loaded on startup; updates are written as JSON. Edit configurations before starting or restart the servers after changing them.

## HTTP API

| Endpoint | Purpose |
| --- | --- |
| `POST /put` | Write a key/value with a selected mode |
| `GET /get?key=x&mode=eventual` | Read a key with a selected mode |
| `GET /dump` | Return the selected replica's local map |
| `POST /internal/replicate` | Apply a peer's versioned record |
| `GET /internal/get?key=x` | Return a local record to a coordinating peer |

The root path `/` has no handler and returns `404`; use `/dump` to inspect a running replica.

A write body uses string values:

```json
{"key":"x","value":"10","mode":"eventual"}
```

Omitting `mode` selects `eventual`. Valid modes are `eventual` and `strong`; an invalid mode or an empty key returns HTTP `400`. A missing key on a read returns `404`. A `strong` write without a majority returns `503`.

## Replication and record selection

Eventual writes save locally and start asynchronous replication. Strong writes contact both peers concurrently, wait for their attempts, and require at least one acknowledgement before saving locally. Each peer HTTP request has a two-second timeout, separate from the configured artificial delay.

Coordinated `strong` reads query peers sequentially, select the best available record, and repair the receiving replica. They do not require a read quorum. `checked_replicas` starts at one for the local server and increases only when a peer returns a record successfully.

Records are ordered by version, replica ID, and then timestamp. Replication treats matching key, value, version, and author fields as duplicates. See [versioning and conflict resolution](../README.md#versioning-and-conflict-resolution) and [implementation scope](../README.md#consistency-modes) for the full behavior.

## Persistence and fresh experiments

Generated data files under `results/` are ignored by Git. To reset an experiment, stop all replicas, remove the three specific data files, and restart the servers. The [CA3 persistence guide](../README.md#configuration-and-persistence) gives commands for Bash and PowerShell that preserve the recorded scenario observations.
