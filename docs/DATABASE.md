# Database

## Engine and location

The server uses SQLite through `modernc.org/sqlite`. `Hub.NewHub` opens `db.sqlite` relative to the server process working directory. At startup, `Hub.Run` executes the embedded `internal/server/db/config/schema.sql`.

Run the server from `server/` so the database is created at `server/db.sqlite`.

## Schema

The current schema creates one table:

```sql
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL
);
```

The database stores a username and password hash, not a plaintext password.

## Generated queries

The SQL source is `internal/server/db/config/queries.sql`:

- `GetUserByUsername`: returns one user row matching a username.
- `CreateUser`: inserts a username and password hash and returns the created row.

The generated `db.Queries` methods are `GetUserByUsername(ctx, username)` and `CreateUser(ctx, CreateUserParams)`.

## Per-client query context

`Hub.NewDbTx` currently creates a background context and a `db.Queries` instance backed by the shared database pool. Despite the `DbTx` name, this structure does not start a SQL transaction in the inspected code.

## Current scope

The schema and query methods exist, but the included `EnetConnection.HandleMessage` is empty. The repository snapshot therefore does not demonstrate a completed registration/login flow or password hashing/verification implementation.
