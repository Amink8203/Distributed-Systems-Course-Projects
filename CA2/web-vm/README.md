# Web Interface and RPC Clients · VM1

[← CA2 setup guide](../README.md)

The web service serves a login page, calls VM2 to verify credentials, and retrieves sample text and image content from VM3 through JSON-RPC.

## Configure and run

Update `authHost` and `fileHost` in [main.go](main.go) to the reachable VM addresses. Use `127.0.0.1` for a demonstration with separate processes on one machine.

Start VM2's [authentication service](../auth-vm/README.md) and VM3's [file service](../file-vm/README.md) first. Then run from **`CA2/web-vm`**, so relative template and static paths resolve:

```bash
go run main.go
```

Open `http://WEB_VM_IP:8080/` in a browser, replacing `WEB_VM_IP` with VM1's address. Use `http://localhost:8080/` for a local demonstration.

## Request flow

1. `GET /` renders [templates/login.html](templates/login.html).
2. `POST /login` calls `AuthenticationService.Login` on VM2 at port `8081`.
3. A failed login renders an error on the login page.
4. A successful login calls `FileService.GetFile` on VM3 at port `9090` for `files/sample.txt` and `images/sample.png`.
5. The text is base64-decoded, and the image becomes a data URI in [templates/welcome.html](templates/welcome.html).

Static files under [static/](static/) are served at `/static/`.

## Demo login

Use `alice` / `alice123`, `bob` / `bob123`, or `admin` / `admin123` from the supplied VM2 database.

If authentication cannot connect, check VM2's address and port. If authentication succeeds but content retrieval fails, check VM3, its sample files, and port `9090`.

## Memory-monitoring extension

The [pub/sub publisher](../pubsub/README.md) provides an alternative web service with `/consume-memory` and memory alerts. Run that publisher in place of this program when demonstrating pub/sub; both bind port `8080`.
