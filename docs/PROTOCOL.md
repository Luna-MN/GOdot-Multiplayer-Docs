# Protocol Buffers protocol

## Source of truth

`shared/packets.proto` is the canonical protocol definition. The repository includes generated Go code in `server/pkg/packets/packets.pb.go` and generated C# code in the client.

The transport payload is a serialized `Packet` message. The envelope contains `sender_id` and a `oneof msg` body.

## Packet envelope

| Field | Number | Type |
|---|---:|---|
| `sender_id` | 1 | `uint64` |
| `chat` | 2 | `ChatMessage` |
| `id` | 3 | `IdMessage` |
| `loginRequest` | 4 | `LoginRequestMessage` |
| `registerRequest` | 5 | `RegisterRequestMessage` |
| `OK` | 6 | `OkResponseMessage` |
| `Deny` | 7 | `DenyResponseMessage` |
| `Player` | 8 | `PlayerMessage` |
| `player_input` | 9 | `PlayerInputMessage` |
| `key` | 10 | `enetAuthKeyMessage` |
| `characters` | 11 | `UsersCharacterMessage` |
| `command` | 12 | `CommandMessage` |

Only one `oneof` body is set per packet.

## Message definitions

### `Vector3Msg`

| Field | Number | Type |
|---|---:|---|
| `x` | 1 | `float` |
| `y` | 2 | `float` |
| `z` | 3 | `float` |

### `ChatType`

| Value | Number |
|---|---:|
| `Nothing` | 0 |
| `Global` | 1 |
| `Local` | 2 |
| `Party` | 3 |
| `Max` | 4 |

### `ChatMessage`

`type` (1, `ChatType`), `msg` (2, `string`), `Username` (3, `string`).

### `IdMessage`

`id` (1, `uint64`).

### `enetAuthKeyMessage`

`id` (1, `uint64`), `key` (2, `bytes`), `expiresAt` (3, `int64` Unix timestamp).

### Authentication response/request messages

- `LoginRequestMessage`: `username` (1), `password` (2).
- `RegisterRequestMessage`: `username` (1), `password` (2).
- `OkResponseMessage`: empty message.
- `DenyResponseMessage`: `reason` (1).

The schema defines these messages; the current server state handler does not implement a complete login/register exchange.

### Player messages

`PlayerMessage` contains `clientId` (1, `uint64`), `characterId` (2, `uint64`), `name` (3, `string`), `level` (4, `uint32`), `position` (5, `Vector3Msg`), `direction` (6, `float`), `speed` (7, `float`), and `rotation` (8, `Vector3Msg`).

`PlayerInputMessage` contains `position` (1, `Vector3Msg`) and `rotation` (2, `Vector3Msg`).

`UsersCharacterMessage` contains repeated `players` (1, `PlayerMessage`).

### `CommandMessage`

`command` (1, `string`) and repeated `args` (2, `string`).

## Go helpers

`server/pkg/packets/util.go` provides:

- `NewChat(msg, msgType)`
- `NewId(id)`
- `NewOK()`
- `NewDeny(reason)`
- `NewKey(id, key, timestamp)`
- `NewVector3(x, y, z)`
- `UnpackVector3(v)`

`Msg` aliases the generated `isPacket_Msg` oneof interface.

## Compatibility

When changing the schema, preserve field numbers for existing fields. Do not reuse removed field numbers. Regenerate both Go and C# bindings from the shared `.proto` file. See [Development](DEVELOPMENT.md).
