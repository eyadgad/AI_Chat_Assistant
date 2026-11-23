# Backend service

Run the HTTP or gRPC service from the repository root with Bun:

```sh
bun install --frozen-lockfile
bun run backend
```

Configure the variables in `backend/.env.example` in the process environment or
the root `.env` first. The service expects the existing assistant conversation,
message, state and audit tables, together with the authorized research gateway
in `backend/database/gateway.sql`. It does not create or migrate those tables at
startup. The runtime account needs the conversation-table permissions and the
gateway function grants; provision these before starting the service.

`NIA_TRANSPORT=grpc` serves `backend/proto/nia.proto`. `NIA_TRANSPORT=http` serves
the browser API. HTTP production mode also requires `STANDALONE_DEMO_PASSWORD`,
`JWT_SECRET` (at least 32 characters), and an explicit `FRONTEND_ORIGIN` or
`CORS_ORIGINS`. `PUBLIC_TEST_PASSWORD` optionally enables the second HTTP account.
The default bind address is `127.0.0.1:8000`; use `APP_HOST` and `APP_PORT` to
override it, or `PORT` on a hosting platform. Model, reasoning effort and speech
options are configured in `backend/reasoning.json`.

Each gRPC conversation and community selection reuses a research session. When
that session must be created or rebuilt, the current question includes at most
the last **two question-and-answer exchanges** from the same selection. Each
question is clipped to 600 characters and each answer to 1,500 characters.
Existing sessions retain their conversation context; this is a restoration
limit, not a limit on the number of follow-up questions. Saved conversation
history remains available through the History RPC.

Run the backend checks with:

```sh
bun run test:backend
```

The service tests use synthetic records and an in-process PostgreSQL fixture.
These checks do not require a live model or speech provider. To check a running
gRPC deployment:

```sh
bun run backend/src/probe.ts --target 127.0.0.1:8000 --plaintext
```
