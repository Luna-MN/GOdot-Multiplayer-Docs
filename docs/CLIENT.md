# Godot C# client

## Project

The client is a Godot .NET project in `client/`. `TestClient.csproj` targets .NET 8 for the normal target and .NET 9 for Android. It references `Google.Protobuf` 3.34.0-rc1.

`project.godot` autoloads:

- `WS`: `Globals/Websocket/websockets.cs`
- `GameManager`: `Globals/GameManager/GameManager.cs`

`main.cs` resolves those autoloads, assigns the static references in `globals`, and sets the initial game state to `Entered`.

## Global references

`Globals/globals.cs` stores static references to the WebSocket node, game manager, and ENet node:

- `globals.WS`
- `globals.GM`
- `globals.ENet` (declared; assignment is not shown in the inspected `Entered` flow)

## GameManager

`GameManager` defines `Entered`, `Connected`, and `InGame` states and maps them to scene paths. `SetState` frees the previous scene root, loads the scene, instantiates it, and adds it as a child.

The repository snapshot references `res://states/connected/connected.tscn` and `res://states/ingame/inGame.tscn`; those paths were not present in the uploaded file listing. The included state scene is `states/entered/entered.tscn`.

## WebSocket node

`websockets` wraps Godot's `WebSocketPeer`:

- `connectToUrl(url, tlsOptions)` starts a connection.
- `Send(Packet)` serializes a protobuf packet and sends it.
- `poll()` advances the peer, emits connection events, and dispatches received packets.
- `Close()` closes the connection.
- `clear()` replaces the peer with a new instance.
- `GetSocket()` exposes the underlying peer.

The `packetRecived` event is the spelling used by the source code.

## Entered state

The `Entered` scene subscribes to WebSocket events, connects to `wss://localhost:8080/ws`, and handles incoming ID packets. When it receives an ID, it stores the ID in `GameManager`, creates an ENet node, sends an ID packet over WebSocket, and transitions to `Connected`.

## ENet node

`ENet.cs` creates an `ENetConnection`, a one-peer client host, and connects to `127.0.0.1:8081` with three channels and the client ID as user data. `Send(Packet, channel)` serializes a packet and sends it through the selected channel.

The client ENet implementation shown in this snapshot does not include receive-event processing or handling of the server-provided key packet. Do not infer that client-side encrypted ENet receive is complete.
