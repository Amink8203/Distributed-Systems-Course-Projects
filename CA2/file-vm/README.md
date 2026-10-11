# File and Image Service · VM3

[← CA2 setup guide](../README.md)

A JSON-RPC server reads files from VM3 and returns their contents as base64. VM1 displays the sample text and image after a successful login.

## Run

From **`CA2/file-vm`**, where the sample folders are present:

```bash
go run main.go
```

The service listens on TCP port `9090`. VM1 must point `fileHost` to VM3's address.

## RPC contract

| Item | Value |
| --- | --- |
| Method | `FileService.GetFile` |
| Request field | `Path` |
| Response fields | `Success`, `Message`, `FileName`, `ContentType`, `Data` |
| Transport | Go `net/rpc/jsonrpc` over TCP |

The Go structs have no JSON tags; these exported field names are also the JSON field names. `Data` contains base64-encoded file bytes. Content type is selected by extension, including PNG, JPEG, GIF, and text; other extensions use `application/octet-stream`.

## Sample content

| Requested path | Local file |
| --- | --- |
| `files/sample.txt` | [Sample text](files/sample.txt) |
| `images/sample.png` | [Sample image](images/sample.png) |

The server uses its current working directory as the base directory. VM1 receives these bytes over RPC and does not need a copy of VM3's files.

## Verify through VM1

Follow the [CA2 launch order](../README.md#setup-and-launch-order), log in through VM1, and check that both text and image appear. VM3 logs the requested path when a file is sent successfully.
