# Networking

## WebSocket transport

The Go server registers `/ws` using Gorilla WebSocket. The HTTP server is started with TLS on port `8080` (`internal.WSport`).

The Godot client uses `WebSocketPeer` in `Globals/Websocket/websockets.cs`. It calls `ConnectToUrl`, polls the peer every process frame, emits connection lifecycle events, and parses incoming binary payloads as Protocol Buffers `Packet` messages.

Client-side events:

- `connectedToServer`
- `connectionClosed`
- `packetRecived` (spelling as defined in source)

`Send(Packet)` sets `SenderId` to zero, serializes the packet, and sends it. The Go WebSocket read pump treats a zero sender ID as the connected client's own ID.

The server uses a buffered send channel of 256 packets per WebSocket client. If the channel is full, outgoing messages are dropped and logged.

## ENet transport

The server initializes go-enet and creates a host on `127.0.0.1:8081`, with a maximum of 4000 clients and 3 channels (from `internal/config.go`).

The Godot ENet node creates a client host with one peer slot, connects to `127.0.0.1:8081`, and requests 3 channels. Its `Send(Packet, channel)` method serializes a protobuf packet and calls `peer.Send(channel, data, 8)`. The source comment describes client sends as unreliable.

The server's ENet `ReadPump` is intentionally empty: incoming ENet traffic is processed centrally by `TrafficHandler`, which services the host and handles connect, disconnect, and receive events.

## ENet encryption

When the server-side `EnetConnection` state is assigned to a client, it:

1. Generates a 32-byte random key.
2. Constructs a ChaCha20-Poly1305 AEAD cipher.
3. Stores the cipher on the logical client.
4. Sends an `EnetAuthKeyMessage` over WebSocket with the client ID, key, and an expiry timestamp one hour in the future.

For outgoing ENet packets, `EnetClient.WritePump` protobuf-serializes the packet, constructs a 12-byte nonce (4 random bytes plus an 8-byte little-endian packet counter), encrypts with AEAD, and sends nonce + ciphertext/tag over ENet.

For incoming ENet packets, `TrafficHandler` checks for a minimum nonce-plus-tag length, separates the nonce and ciphertext/tag, decrypts with the client's AEAD cipher, and unmarshals the protobuf packet.

The expiry timestamp is transmitted, but the inspected receive path does not enforce it. The server's packet counter is a Hub field and should be reviewed for concurrency safety if multiple ENet writers can access it concurrently.

## Channel mapping in the current code

`TransferType` defines WebSocket, `ENet_Channel0`, and `ENet_Channel1`. The ENet client maps received channel ID `2` to `ENet_Channel1`; other non-nil channel IDs map to `ENet_Channel0`. Outbound `Client.SocketSendAs` routes channel 0 or 1 based on the transfer type.

Do not assume the three configured ENet channels have fully specified semantics; only the mapping above is explicit in the current implementation.

## Security and deployment caveats

- The WebSocket upgrader accepts all origins. Restrict origins for production.
- The client uses `TlsOptions.ClientUnsafe()`. Replace it with normal certificate validation.
- The ENet host currently binds to loopback, not a public interface.
- The server currently uses a development self-signed certificate workflow.
- Review nonce/counter concurrency, key expiry enforcement, peer lifecycle, and authentication before production use.
