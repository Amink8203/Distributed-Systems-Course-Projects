# Part 1 · Inter-Process Communication with Named Pipes

[← CA1 overview](../README.md)

Two independent Go programs implement an arithmetic request/response service. The interface sends newline-delimited text requests; the worker replies with JSON through a second named pipe.

## Requirements and files

Use Linux, a Linux VM, or WSL. The worker calls `syscall.Mkfifo`, which is unavailable on Windows. Run both processes from **`CA1/part1`** so they share the same relative FIFO paths.

| File or path | Purpose |
| --- | --- |
| [worker.go](worker.go) | Creates the FIFOs, validates requests, and computes results |
| [interface.go](interface.go) | Reads user commands and displays worker responses |
| `./temp/request.fifo` | Interface → worker requests |
| `./temp/response.fifo` | Worker → interface responses |

The worker creates `temp/` and the FIFOs. These runtime files are ignored by Git.

## Build and run

Build each program separately from `CA1/part1`:

```bash
go build -o worker worker.go
go build -o interface interface.go
```

Start the worker in the first terminal:

```bash
./worker
```

Start the interface in a second terminal, in the same directory:

```bash
./interface
```

Opening a FIFO can block while waiting for the other process; start the interface even if the worker appears to be waiting. Enter arithmetic commands at the interface prompt. Stop each process with `Ctrl+C`.

## Request and response protocol

Each request is one line:

```text
OP A B
```

Supported operations are `ADD`, `SUB`, `MUL`, `DIV`, `MOD`, `POW`, `MAX`, and `MIN`. For example:

```text
ADD 5 8
DIV 20 0
MUL 3 6
```

Successful response:

```json
{"status":"OK","result":13}
```

Error response:

```json
{"status":"ERROR","error":"Division by zero"}
```

The `result` and `error` fields use `omitempty`; a zero numeric result may be absent from the JSON and decodes as zero in the interface. The interface displays a simpler representation:

```text
> ADD 5 8
OK 13.000000
> DIV 20 0
ERROR Division by zero
```

The two one-way pipes form a bidirectional channel. Newlines delimit requests and responses so each process can read complete messages.
