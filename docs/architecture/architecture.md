---
summary: Architecture of clihub generation, codegen, and compile pipeline.
read_when:
  - You are changing command flow, generation behavior, or package boundaries.
  - You need to trace how MCP tools become generated CLI commands.
  - You are debugging failures across connect, discover, generate, or compile steps.
---

# Architecture

## System overview

`clihub` is a build-time transformer:
1. Input: MCP server endpoint and generation flags.
2. Process: discover MCP tools, generate Go source, compile binaries.
3. Output: standalone CLI executable(s) with embedded server config.

Core entrypoint: `clihub generate` in `/cmd/generate.go`.

## End-to-end flow

1. Validate flags and Go toolchain.
2. Resolve auth and build MCP client (HTTP or stdio).
3. Start transport, run MCP initialize handshake.
4. Call `tools/list` and collect tool schemas.
5. Filter included/excluded tools.
6. Convert tool schemas to option definitions.
7. Build codegen context.
8. Generate temporary Go project (`main.go`, `go.mod`, `go.sum`).
9. Compile for target platform(s).
10. Run smoke test for host-platform binary.
11. Print output summary and binary paths.

## Module map

## CLI layer

- `/cmd/root.go`: root command, version wiring.
- `/cmd/generate.go`: main orchestration path.

Responsibilities:
1. Parse/validate flags.
2. Drive MCP discovery and filtering.
3. Assemble codegen context.
4. Trigger compilation and smoke test.

## Auth layer

- `/internal/auth/*`

Responsibilities:
1. Provider implementations (bearer, API key, basic, OAuth2, S2S OAuth2, Google SA).
2. OAuth browser flow and metadata discovery.
3. Credential file load/save and v1->v2 migration.
4. `WWW-Authenticate` parsing for auth auto-detection.

## Schema layer

- `/internal/schema/*`

Responsibilities:
1. Parse MCP tool input JSON Schema.
2. Map schema fields to Go/Cobra flag options.
3. Normalize names for command/flag compatibility.

## Codegen layer

- `/internal/codegen/*`

Responsibilities:
1. Build generation context and tool metadata.
2. Render generated `main.go` and `go.mod` from templates.
3. Run `go mod tidy` in generated project.

Notes:
- Main template is intentionally large: `/internal/codegen/main_tmpl.go`.
- Generated CLIs include runtime MCP client, auth handling, output formatting, and tool commands.

## Compile layer

- `/internal/compile/*`

Responsibilities:
1. Parse and validate target platforms.
2. Invoke `go build` with target GOOS/GOARCH and `CGO_ENABLED=0`.
3. Name output binaries (single- and multi-platform modes).
4. Run smoke test (`--help`) on host-platform output.

## Supporting utilities

- `/internal/nameutil/*`: infer binary names from URL/stdio commands.
- `/internal/toolfilter/*`: include/exclude matching with fuzzy help.
- `/internal/gocheck/check.go`: minimum Go version enforcement.

## Data flow and key structures

Generation context:
- `GenerateContext` in `/internal/codegen/context.go` carries CLI name, transport mode, server config, env key names, and tool definitions.

Tool representation:
- MCP `Tool` -> internal `ToolDef` -> generated Cobra command.
- Schema properties -> `ToolOption` -> typed flags + optional enum/default behavior.

Build artifact flow:
1. Temporary project directory is created.
2. Generated source is compiled into `--output` directory.
3. Temp project is cleaned on success.
4. On compile/smoke failure, temp source path is preserved and printed.

## Design constraints

1. Current command set focuses on `tools/list` + `tools/call`; resources/prompts generation is not yet implemented.
2. Schema handling is intentionally pragmatic; complex composition support is partial.
3. Generated runtime behavior and clihub runtime behavior must stay aligned, especially for auth and security-sensitive paths.
4. Single command currently owns most orchestration (`cmd/generate.go`), so behavior changes can span several concerns.

## Extension points

1. New auth providers: add under `/internal/auth`, then wire into provider resolution paths.
2. New schema keywords/types: extend `/internal/schema` mapping and add tests.
3. Generated CLI capabilities: modify template + context model + codegen tests.
4. New top-level commands: add under `/cmd` and register in root command.
