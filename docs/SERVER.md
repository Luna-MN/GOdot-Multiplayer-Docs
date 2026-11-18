# Go server

## Entry point

`server/cmd/main.go`:

1. Creates the Hub with `server.NewHub()`.
2. Registers an HTTP handler at `/ws` that calls `hub.Serve(clients.NewWebSocketClient, ...)`.
3. Initializes the ENet host with `ENet.InitEnet(hub)`.
4. Starts `hub.Run()` in a goroutine.
5. Starts the TLS HTTP server on `internal.WSport`.

The process expects `cert/server.crt` and `cert/server.key` relative to its working directory.

## Configuration

`server/internal/config.go` currently defines:

| Constant | Value | Meaning |
|---|---:|---|
| `WSport` | 8080 | HTTPS/WebSocket listener port |
| `ENetPort` | 8081 | ENet UDP port |
| `ENetChannels` | 3 | ENet host channel count |
| `EnetMaxClients` | 4000 | ENet host client capacity |
| `TickRate` | 0.01 | Declared tick interval (not wired into a loop in the inspected code) |
| `SendRate` | 0.05 | Declared send interval (not wired into a loop in the inspected code) |

The ENet host binds to `127.0.0.1:8081` in `enetTrafficManager.go`.

## Hub and clients

See [Architecture](ARCHITECTURE.md) for the Hub and state model. The WebSocket and ENet transport implementations satisfy `ClientInterfacer`, while `Client` provides the logical connection shared by both.

`Client.SocketSendAs` routes messages to WebSocket or ENet channel 0/1 according to `TransferType`. `Client.Broadcast` submits a `BroadcastItem` to the Hub. Hub broadcasts exclude the sender and, unless `ToAll` is set, only deliver to clients whose current state name matches the originating state name.

## Adding a state

Implement `ClientStateHandler` with the six methods documented in `hub.go`. Use `SetClient` to receive the logical `*Client`, `OnEnter` for state initialization, `HandleMessage` for packet dispatch, and `OnExit` for cleanup. Then transition using `Client.SetState`.

The only included state is `states.EnetConnection`. Its `HandleMessage` and `OnExit` methods are empty in this snapshot, so use it as a structural example rather than a complete gameplay state.

## Shared objects

`SharedGameObjects.Players` is a `*objects.SharedCollection[*objects.Player]`. The `Player` struct currently has `Name`, `Position`, `Direction`, and `Speed` fields.

`SharedCollection[T]` provides `Add`, `Remove`, `Get`, `Len`, and `ForEach`. Map access is mutex-protected. Note that values stored in the collection are not automatically deep-copied; pointer values still refer to shared mutable objects.
