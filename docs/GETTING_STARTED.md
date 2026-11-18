# Getting started

This guide describes the checked-in development setup. It does not assume a production deployment.

## Prerequisites

- Godot 4.7 .NET.
- .NET 8 SDK.
- Go 1.25.6.
- OpenSSL.
- Git.

The versions above are based on `client/TestClient.csproj`, `client/project.godot`, and `server/go.mod`.

## Start the server

The server expects to be run with `server/` as its working directory. This matters because the TLS files and SQLite database use relative paths.

```bash
cd server
mkdir -p cert
openssl req -x509 -newkey rsa:4096 \
  -keyout cert/server.key \
  -out cert/server.crt \
  -days 365 \
  -nodes

go mod download
go run ./cmd
```

The WebSocket endpoint is `wss://localhost:8080/ws`. ENet binds to `127.0.0.1:8081`.

## Run the client

Open `client/project.godot` in Godot 4.7 .NET and run the project. The initial `Entered` scene connects to the WebSocket endpoint with `TlsOptions.ClientUnsafe()`. After receiving an ID message, it creates an ENet node and moves the game manager to the `Connected` state.

## Local connection flow

1. The Godot client opens a WebSocket connection to `/ws`.
2. The Hub registers the WebSocket client and assigns a numeric client ID.
3. The server's initial state (`EnetConnection`) generates a 32-byte key and sends a key packet over WebSocket.
4. The client receives the ID packet, stores the ID, adds an ENet node, and sends an ID packet over WebSocket.
5. The ENet node connects to `127.0.0.1:8081`, passing the client ID as ENet user data.
6. The ENet traffic handler uses that user data to associate the ENet peer with the previously registered Hub client.

The current `EnetConnection.HandleMessage` method is empty, and the checked-in client does not show a complete gameplay loop.

## Working directory

Run the server from `server/`, not the repository root. `main.go` checks `cert/server.crt` and `cert/server.key`, and `NewHub` opens `db.sqlite`; both paths are relative to the process working directory.
