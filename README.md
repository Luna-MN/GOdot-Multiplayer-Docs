# GOdot

GOdot is an experimental client/server multiplayer framework built with **Godot 4.7 C#** and **Go**. It contains a Godot client, a Go server, shared Protocol Buffers message definitions, WebSocket transport for the initial connection, and ENet transport for UDP communication.

> **Project status:** The repository is a framework prototype/test client, not a finished game-server SDK. Some protocol messages and server-side state handlers are defined but not fully implemented. See [Implementation status and limitations](docs/IMPLEMENTATION_STATUS.md) before building on it.

## Repository layout

```text
GOdot/
├── client/                 # Godot 4.7 .NET/C# test client
├── server/                 # Go server
│   ├── cmd/main.go         # Server entry point
│   ├── internal/           # Configuration and server implementation
│   └── pkg/packets/        # Generated Go protobuf code and helpers
├── shared/
│   └── packets.proto       # Shared protocol definition
└── docs/                   # Project documentation
```

## Requirements

The checked-in project declares or uses:

- Godot **4.7 .NET** (the client project uses `Godot.NET.Sdk/4.7.0`).
- .NET 8 for the desktop client target.
- Go **1.25.6** (from `server/go.mod`).
- OpenSSL to generate a development TLS certificate.
- Protocol Buffers compiler (`protoc`) only if regenerating message bindings.

The client project references `Google.Protobuf` 3.34.0-rc1. The Go server uses `google.golang.org/protobuf` 1.36.11, Gorilla WebSocket, go-enet, ChaCha20-Poly1305 from `golang.org/x/crypto`, and modernc SQLite.

## Quick start

### 1. Generate a development TLS certificate

The server entry point expects certificate paths relative to the server working directory:

```bash
cd server
mkdir -p cert
openssl req -x509 -newkey rsa:4096 \
  -keyout cert/server.key \
  -out cert/server.crt \
  -days 365 \
  -nodes
```

This is a self-signed development certificate. The Godot test client currently connects with `TlsOptions.ClientUnsafe()`, which disables normal certificate verification. Do not use that setting in a production client.

### 2. Start the Go server

From the repository root:

```bash
cd server
go mod download
go run ./cmd
```

The WebSocket listener is configured on TCP port `8080`; the ENet host is configured on UDP port `8081`. The ENet host currently binds to `127.0.0.1`, so it is local-machine-only without code/configuration changes.

The server opens `db.sqlite` relative to its current working directory and creates the `users` table from the embedded schema at startup.

### 3. Run the Godot client

Open `client/project.godot` with Godot 4.7 .NET and run the project. The current `Entered` state attempts to connect to:

```text
wss://localhost:8080/ws
```

The client then receives an ID over WebSocket and creates an ENet client connection to `127.0.0.1:8081`.

## Architecture

- **WebSocket/TLS:** initial connection, server-assigned client ID, and control messages.
- **ENet/UDP:** separate transport attached to the already-registered client. The server sends an ENet authentication key over WebSocket and uses ChaCha20-Poly1305 when sending and receiving ENet packet payloads.
- **Protocol Buffers:** a shared `Packet` envelope and message definitions in `shared/packets.proto`.
- **Hub:** registers/unregisters clients and dispatches broadcasts.
- **Client state handlers:** server-side state interface for handling messages according to a client's current state.
- **Shared game objects:** a thread-safe generic collection currently used for player objects.

See [Architecture](docs/ARCHITECTURE.md), [Networking](docs/NETWORKING.md), and [Protocol](docs/PROTOCOL.md).

## Documentation

- [Getting started](docs/GETTING_STARTED.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Networking](docs/NETWORKING.md)
- [Protocol Buffers messages](docs/PROTOCOL.md)
- [Server internals](docs/SERVER.md)
- [Godot client](docs/CLIENT.md)
- [Database](docs/DATABASE.md)
- [Implementation status and limitations](docs/IMPLEMENTATION_STATUS.md)
- [Development and regeneration](docs/DEVELOPMENT.md)

## Security notes

- Replace the self-signed certificate and unsafe client TLS configuration before deployment.
- The WebSocket upgrader currently accepts every Origin (`CheckOrigin` always returns `true`). Restrict this for production.
- ENet authentication keys are generated per connection and sent over WebSocket. The current implementation sets a one-hour expiry timestamp in the message; the ENet receive path does not visibly enforce that expiry.
- The current login/register message definitions are not evidence of a completed authentication flow. Review server state handling before exposing account functionality.
- Do not commit private keys, real credentials, or production database files.

## Contributing

Issues and pull requests are welcome. When changing packet definitions, update `shared/packets.proto` and regenerate both language bindings using the commands documented in [Development](docs/DEVELOPMENT.md).

## License

No license file was present in the inspected repository snapshot. Check with the repository owner before redistributing or incorporating the code into another project.
