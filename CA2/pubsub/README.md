# Publish/Subscribe · Memory Alerts

[← CA2 setup guide](../README.md)

The publisher extends VM1's web application with Go heap monitoring. When heap allocation exceeds a threshold, it sends a newline-delimited JSON event directly to a TCP subscriber.

| Program | Responsibility | Listening port |
| --- | --- | --- |
| [publisher.go](publisher.go) | Web interface, RPC clients, memory allocation, monitoring, and publication | HTTP `8080` |
| [subscriber.go](subscriber.go) | Receive events and print alerts | TCP `7000` |

## Configuration

Edit these constants in `publisher.go`:

| Constant | Meaning | Default |
| --- | --- | --- |
| `authHost`, `authPort` | Authentication service | `192.168.137.131:8081` |
| `fileHost`, `filePort` | File service | `192.168.137.132:9090` |
| `subHost`, `subPort` | Subscriber | `192.168.137.130:7000` |
| `thresholdMB` | Heap allocation threshold in MiB | `300` |
| `monitorIntervalSecond` | Monitor interval | `3` seconds |

Use reachable VM addresses to reproduce the course deployment. For separate local processes, set all three host constants to `127.0.0.1`.

## Run

Start the [authentication](../auth-vm/README.md) and [file](../file-vm/README.md) services first.

From **`CA2/pubsub`** on the subscriber host:

```bash
go run subscriber.go
```

From **`CA2/pubsub`** on VM1, in another terminal:

```bash
go run publisher.go
```

The publisher needs its local `templates/` directory. Build or run the two files separately because both define a `main` function. The publisher uses port `8080`, so stop the basic `web-vm` service before starting it.

## Demonstrate an alert

The login and file-retrieval flow remains available at `http://WEB_VM_IP:8080/`. Replace `WEB_VM_IP` with the publisher's address; use `localhost` for a local demo.

Increase heap allocation:

```bash
curl "http://WEB_VM_IP:8080/consume-memory?mb=100"
```

Repeat until allocation exceeds 300 MiB. The endpoint touches memory pages and retains the blocks, so allocated memory stays referenced until the publisher stops. Restart the publisher to release it, or lower `thresholdMB` before starting a smaller demonstration.

The monitor uses `runtime.ReadMemStats().HeapAlloc`, rather than total machine memory. It sends one event after exceeding the threshold, suppresses further alerts while usage remains high, and rearms if usage drops below or equals the threshold. A failed publication is logged and retried on the next monitoring iteration.

## Event format

Example event:

```json
{
  "event_type": "HIGH_MEMORY_USAGE",
  "service": "web-server",
  "memory_mb": 345,
  "threshold_mb": 300,
  "timestamp": "2026-05-31T17:50:00Z"
}
```

The publisher opens a TCP connection, writes a JSON object followed by a newline, and closes the connection. The subscriber reads that line and displays `ALERT: HIGH MEMORY USAGE RECEIVED` with the event details. This implementation connects directly to one configured subscriber; it has no external message broker.
