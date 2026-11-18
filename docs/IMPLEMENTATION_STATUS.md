# Implementation status and limitations

This page separates code that exists in the repository from behavior that should not be assumed to be complete.

## Present in the snapshot

- Godot 4.7 .NET/C# client project.
- Go server module and server entry point.
- TLS WebSocket endpoint at `/ws`.
- ENet host and central ENet event loop.
- Logical client abstraction shared by WebSocket and ENet transports.
- Hub registration, unregistration, and broadcast dispatch.
- Protocol Buffers schema and generated Go/C# bindings.
- ChaCha20-Poly1305 encryption/decryption path for ENet packet payloads on the server.
- SQLite users table and generated select/insert queries.
- Generic mutex-protected `SharedCollection`.
- Basic `Player` and vector structs.
- Client-side initial connection scene and GameManager state switching.

## Defined but not demonstrated as complete

- Login and registration packet types.
- Chat message types and chat broadcast policy.
- Player and player-input messages.
- Character-list message.
- Command message.
- Server-side client state interface and state transitions.
- Shared player collection and player model.

## Incomplete or notable gaps observed

1. **Server state handling:** `EnetConnection.HandleMessage` is empty, as is `OnExit`. There is no complete packet dispatch for login, registration, chat, player input, or commands in this state.
2. **Client ENet receive path:** the Godot ENet node services the connection but does not show event/packet receive handling. The client does not show consuming the server's key packet for encrypted ENet traffic.
3. **Client scene references:** `GameManager` refers to Connected and InGame scenes that were not included in the uploaded archive listing.
4. **Client global ENet reference:** `globals.ENet` exists, but the shown `Entered` code creates and adds an ENet node without assigning it to that static field.
5. **Configuration constants:** `TickRate` and `SendRate` are declared but are not connected to an update/broadcast loop in the inspected files.
6. **Authentication security:** login/register messages are schema definitions only in this snapshot. The database has password-hash storage, but no hashing/verification implementation was found in the state handler.
7. **TLS client validation:** the test client uses `TlsOptions.ClientUnsafe()`.
8. **WebSocket origin policy:** the server's upgrader accepts all origins.
9. **ENet binding:** the server binds to `127.0.0.1`, limiting connections to the local host.
10. **Key expiry:** the server sends an expiry timestamp with the ENet key, but the inspected receive path does not check it.
11. **Packet counter concurrency:** ENet writers update a Hub-level packet counter. Review synchronization if multiple goroutines can write concurrently.
12. **Channel semantics:** three ENet channels are configured, but the current transfer-type mapping explicitly distinguishes only channel 0 and channel 1; received channel ID 2 is mapped to `ENet_Channel1`.

These are observations about the uploaded repository snapshot, not claims about uncommitted branches or later versions.
