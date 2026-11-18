# Development

## Repository conventions

- Edit `shared/packets.proto` as the canonical packet schema.
- Generated Go code lives in `server/pkg/packets/`.
- Generated C# code is included in the Godot client (the generated `Packets` namespace).
- Server code is under `server/internal/`; the executable entry point is `server/cmd/main.go`.

## Regenerate Protocol Buffers bindings

The source schema includes these commands:

```bash
protoc -I="shared" --go_out="server" "shared/packets.proto"
protoc -I="shared" --csharp_out="client" "shared/packets.proto"
```

The Go command expects the Go protobuf plugin (`protoc-gen-go`) to be installed and available on `PATH`. Ensure the generated Go package output matches the module's `server/pkg/packets` package layout. The C# command generates the C# protobuf types into the client tree.

After changing the schema:

1. Preserve existing field numbers.
2. Add new fields with unused field numbers.
3. Regenerate both language bindings.
4. Build the Go server.
5. Build/open the Godot .NET client.
6. Test compatibility across both transports.

## Go commands

Run from `server/`:

```bash
go mod download
go test ./...
go build ./...
go run ./cmd
```

No Go test files were present in the uploaded snapshot. `go test ./...` is therefore a useful compile/test check, but it should not be interpreted as exercising application behavior.

## Client

Open `client/project.godot` in Godot 4.7 .NET. Build the C# project with the Godot .NET toolchain. The project file declares .NET 8 for desktop and .NET 9 for Android.

## Before a production release

At minimum, add automated tests for packet dispatch, client registration and teardown, authentication, encryption key lifecycle, broadcast filtering, and concurrent ENet sends. Replace unsafe TLS settings, restrict WebSocket origins, configure a reachable ENet bind address, and externalize host/port/certificate/database settings.
