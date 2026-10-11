# Authentication Service · VM2

[← CA2 setup guide](../README.md)

A JSON-RPC server verifies login credentials against the SHA-256 password hashes in [users.json](users.json). The web service on VM1 calls it over TCP port `8081`.

## Run

From **`CA2/auth-vm`**, so the service can read `users.json`:

```bash
go run main.go
```

VM1 must use VM2's address for `authHost`, and port `8081` must be reachable.

## RPC contract

| Item | Value |
| --- | --- |
| Method | `AuthenticationService.Login` |
| Request fields | `Username`, `Password` |
| Response fields | `Success`, `Message` |
| Transport | Go `net/rpc/jsonrpc` over TCP |

The server hashes the supplied password and compares it with the stored hash. A match returns `Success: true`; otherwise the message is `Invalid username or password`. The Go structs have no JSON tags, so their exported field names are also their JSON field names.

## Demo users

| Username | Password |
| --- | --- |
| `alice` | `alice123` |
| `bob` | `bob123` |
| `admin` | `admin123` |

These credentials are supplied for the course demonstration. The data file is a JSON array of objects with `username` and `password_hash` fields. Passwords are entered normally at the web login page; only their hashes are stored in the file.

## Verify through VM1

Start the authentication and file services, then the web service as described in the [CA2 guide](../README.md#setup-and-launch-order). A correct login should display the welcome page and retrieved content. A wrong password should display an error.
