# Part 3 · HTTP Calculator, Docker, and VM Deployment

[← CA1 overview](../README.md)

A Go HTTP service exposes a health check and arithmetic operations on port `8080`. The Dockerfile packages the compiled service for execution inside a Linux container or VM.

## Run locally

From **`CA1/part3`**:

```bash
go run main.go
```

In another terminal:

```bash
curl "http://localhost:8080/health"
curl "http://localhost:8080/compute?operator=add&a=10&b=20"
```

The computation response is:

```json
{"operation":"add","a":10,"b":20,"result":30}
```

Stop the service with `Ctrl+C`.

## HTTP API

| Endpoint | Parameters | Response |
| --- | --- | --- |
| `GET /health` | None | JSON fields `status`, `service`, and current RFC3339 `time` |
| `GET /compute` | `operator`, `a`, `b` | JSON fields `operation`, `a`, `b`, and `result` |

Supported operations: `add`, `sub`, `mul`, `div`, `mod`, and `pow`. Operation names are case-insensitive. The query parameter is **`operator`**.

Missing parameters, invalid numeric operands, unsupported operations, and division/modulo by zero return HTTP `400` with an `error` field. A non-GET request returns HTTP `405`.

```bash
curl "http://localhost:8080/compute?operator=div&a=8&b=0"
```

```json
{"error":"Division By Zero!"}
```

## Docker deployment

Run from `CA1/part3`, where the [Dockerfile](Dockerfile) and [main.go](main.go) live:

```bash
docker build -t ds-ca1-calculator .
docker run --rm --name ds-ca1-calculator -p 8080:8080 ds-ca1-calculator
```

The multi-stage build compiles the standalone Go source and copies the executable into an Alpine runtime image. There are no external Go modules to install.

For a detached container, omit `--rm` and add `-d` to the run command. Inspect and clean up that container with:

```bash
docker logs ds-ca1-calculator
docker stop ds-ca1-calculator
docker rm ds-ca1-calculator
```

## Access from the host through a VM

Build and run the container inside the Linux VM. Port mapping `8080:8080` exposes the container's service on the VM's port `8080`.

Find the VM address inside the VM:

```bash
hostname -I
```

From the host, replace `VM_IP` with the reachable VM address:

```bash
curl "http://VM_IP:8080/health"
curl "http://VM_IP:8080/compute?operator=add&a=10&b=20"
```

The VM network mode and firewall must allow host access to TCP port `8080`. In Windows PowerShell, use `curl.exe` if `curl` resolves to `Invoke-WebRequest`.
