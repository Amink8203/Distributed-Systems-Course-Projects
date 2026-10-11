# Command-Line Client

[← CA3 quick start](../README.md#quick-start)

The client sends HTTP requests to a selected replica and prints the HTTP status, client-measured latency, and formatted JSON response.

**Run these commands from `CA3`**, with the replicas already running.

## Flags

| Flag | Default | Meaning |
| --- | --- | --- |
| `-op` | `get` | `put`, `get`, or `dump` |
| `-replica` | `http://localhost:8001` | Target replica URL |
| `-key` | Empty | Required for `put` and `get` |
| `-value` | Empty | String value to write |
| `-mode` | `eventual` | `eventual` or `strong`; ignored by `dump` |

## Examples

Write asynchronously:

```bash
go run client/main.go -op put -replica http://localhost:8001 -key x -value 10 -mode eventual
```

Read only replica 2's local state:

```bash
go run client/main.go -op get -replica http://localhost:8002 -key x -mode eventual
```

Request a majority-acknowledged write and a coordinated read:

```bash
go run client/main.go -op put -replica http://localhost:8001 -key y -value 20 -mode strong
go run client/main.go -op get -replica http://localhost:8002 -key y -mode strong
```

Inspect all local records on replica 3:

```bash
go run client/main.go -op dump -replica http://localhost:8003
```

See the [consistency-mode comparison](../README.md#consistency-modes) for the exact behavior of `strong` mode.

## Build once for timing experiments

Repeated `go run` commands add startup time between requests. Build the client before testing the short interval before replication:

```bash
go build -o ds-client client/main.go
./ds-client -op put -replica http://localhost:8001 -key fresh-key -value 10 -mode eventual
./ds-client -op get -replica http://localhost:8002 -key fresh-key -mode eventual
```

On Windows PowerShell:

```powershell
go build -o ds-client.exe client/main.go
.\ds-client.exe -op put -replica http://localhost:8001 -key fresh-key -value 10 -mode eventual
.\ds-client.exe -op get -replica http://localhost:8002 -key fresh-key -mode eventual
```

Use a fresh key, then repeat the read after the configured replication delay. The reported client latency covers the HTTP operation, not compilation or the wait between commands.

## Interpret the output

| Field | Meaning |
| --- | --- |
| `ok`, `message` | Server outcome and explanation |
| `record` | Key, value, version, author, and timestamp |
| `updated_replicas` | Local replica plus acknowledged peers for a write; `1` for an immediate eventual write response |
| `checked_replicas` | Local replica plus peers returning a record for a coordinated read |
| `latency_ms` | Time measured inside the server handler |

The HTTP status is printed even when the server returns an error such as `404` or `503`. A received HTTP error does not itself cause a nonzero client process exit; inspect the printed status and response body.
