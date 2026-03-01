---
summary: Authentication architecture, provider resolution, and credential storage behavior.
read_when:
  - You are modifying auth flags, provider logic, or credential persistence.
  - You are debugging 401/403 errors, OAuth flows, or token refresh behavior.
  - You are updating generated CLI auth behavior and need parity with clihub runtime.
---

# Auth

## Auth surfaces

There are two auth surfaces in this repository:

1. `clihub generate` authentication.
- Used so clihub can connect to the target MCP server and discover tools.

2. Generated CLI runtime authentication.
- Embedded in generated binaries so tool calls can authenticate at runtime.

Both surfaces support similar auth types, but implementation lives in different places:
- clihub runtime: `/cmd/generate.go` + `/internal/auth/*`
- generated runtime: `/internal/codegen/main_tmpl.go`

## Supported auth types

Supported providers in current code paths:
1. `bearer` / `bearer_token`
2. `api_key`
3. `basic` / `basic_auth`
4. `oauth2`
5. `dcr_oauth`
6. `s2s_oauth2`
7. `google_sa`
8. `none` / `no_auth`

The `--oauth` flag is an alias for `--auth-type oauth2` in `clihub generate`.

## Resolution order for `clihub generate`

For HTTP endpoints, auth provider resolution order is:

1. Explicit `--auth-type` (+ related flags).
2. `--auth-token` (infers bearer token provider).
3. `CLIHUB_AUTH_TOKEN` environment variable.
4. `~/.clihub/credentials.json` entry matching server URL.
5. No-auth provider.

After no-auth selection, clihub can probe the server and auto-detect auth requirements from `401` + `WWW-Authenticate` and trigger OAuth discovery/flow when possible.

## HTTP auth auto-detection behavior

When no explicit auth is set for an HTTP server:

1. Probe via HTTP POST to MCP URL.
2. If response is `401`, parse `WWW-Authenticate` challenge.
3. If bearer challenge indicates OAuth metadata, run OAuth flow.
4. If discovery fails, return a guidance error (for token or `--oauth`).

Important:
- Current auth-error detection is primarily `401`-oriented in generate path.

## Credential storage model

Default path:
- `~/.clihub/credentials.json`

Current schema (version 2) stores server-keyed credentials, including provider type and provider-specific fields.

Current behavior:
1. Missing credentials file returns empty in-memory config.
2. Version 1 records are auto-migrated to version 2 on load.
3. Save writes with `0600` file mode and creates parent directory with `0700`.

Known limitations (tracked in roadmap):
1. Writes are not yet atomic.
2. File locking is not yet implemented.
3. Secret material is still file-based, not keychain-backed.

## Generate-time flags and hidden auth options

`clihub generate` exposes auth options via `--help-auth` and hides most auth flags from default help output.

Representative auth flags:
1. `--oauth`
2. `--auth-token`
3. `--auth-type`
4. `--auth-header-name`
5. `--auth-key-file`
6. `--client-id`
7. `--client-secret`
8. `--save-credentials`

## Generated CLI auth behavior

Generated CLIs include:
1. persistent auth flags (`--auth-token`, `--auth-type`, etc.)
2. hidden-by-default auth flags + `--help-auth`
3. HTTP mode `auth` command for token workflows

Generated runtime and clihub runtime should remain behaviorally aligned for:
1. provider selection
2. credential loading/saving
3. token refresh logic
4. error redaction/safety expectations

## Safe-change checklist

When editing auth behavior:
1. Update both runtime and generated-template paths if applicable.
2. Verify provider precedence and backwards compatibility.
3. Check credential schema migration impact.
4. Add/update tests under `/internal/auth` and `/internal/codegen`.
5. Update docs in this file and `README.md` when user-facing behavior changes.
