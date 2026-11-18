# Architecture

## High-level components

```text
Godot C# client
  ├── WebSocketPeer ─────────────── TLS WebSocket :8080/ws
  ├── ENetConnection ───────────── UDP ENet :8081
  ├── GameManager / scene states
  └── Generated Protocol Buffers types
                    │
                    ▼
Go server
  ├── HTTP/WebSocket handler
  ├── ENet traffic handler
  ├── Hub
  │    ├── client registry
  │    ├── registration/unregistration channels
  │    ├── broadcast channel
  │    └── shared game objects
  ├── Client + ClientStateHandler
  ├── SQLite database layer
  └── Generated Protocol Buffers types

shared/packets.proto
  └── canonical message definitions
```

## Connection model

The repository uses a two-transport model. WebSocket creates and registers the logical client. The server assigns its ID through the Hub. The client then opens an ENet connection and passes that ID as ENet connect user data. `TrafficHandler` uses the ENet peer's connect ID to find the corresponding logical client.

The ENet transport is therefore attached to an existing WebSocket client; it is not independently registered as a new logical player.

## Hub

`server/internal/server/hub.go` defines `Hub`. It owns:

- `Clients`: a `SharedCollection[*Client]`.
- `RegisterChan`: incoming transport clients awaiting registration.
- `UnregisterChan`: clients leaving the Hub.
- `BroadcastChan`: messages to distribute to other clients.
- `SharedGameObjects`: currently includes a shared player collection.
- `PeerToClient`: maps ENet peer connect IDs to logical clients.
- SQLite connection pool and a packet counter.

`Hub.Run` is the central select loop. Registration creates a `Client`, adds it to the collection, and initializes the transport/client IDs. Broadcast handling skips the sender and can filter recipients by state name unless `ToAll` is true.

## Client and states

`Client` (`client.go`) represents the logical connection across both transports. It stores WebSocket and ENet transport interfaces, current state, database query context, optional player data, and an AEAD cipher.

`ClientStateHandler` defines the server-side state contract:

- `Name`
- `SetClient`
- `CanReceiveGlobalChat`
- `OnEnter`
- `HandleMessage`
- `OnExit`

`Client.SetState` calls `OnExit` on the old state, assigns the new state, injects the client, and calls `OnEnter`.

The only server state in the snapshot is `states.EnetConnection`. Its message handler and exit hook are currently empty.

## Shared game objects

`SharedGameObjects` currently exposes `Players`, a `SharedCollection[*objects.Player]`. The collection protects map operations with a mutex. Its `ForEach` method copies the map while locked and invokes callbacks after releasing the lock.

## Important distinction

The server contains abstractions for broadcasts, player data, state handling, and account records. Their presence does not imply that complete gameplay, chat, authentication, or replication flows are implemented. See [Implementation status](IMPLEMENTATION_STATUS.md).
