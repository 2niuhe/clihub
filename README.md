# clihub (Contributor Guide)

This README is for people contributing to `clihub`.
If you want product-style usage examples first, start from `docs/project/project.md` and the main command help.

## What this project does

`clihub` turns an MCP server into a compiled CLI binary.

Contributor-level flow:
1. Connect to MCP server (HTTP or stdio).
2. Discover server tools.
3. Convert tool schemas to CLI flag definitions.
4. Generate a Go CLI project from templates.
5. Compile to one or more target binaries.

## Quick start for contributors

1. Clone and enter repo.
```bash
git clone https://github.com/thellimist/clihub.git
cd clihub
```

2. Install hooks once per clone.
```bash
bash scripts/install-hooks.sh
```

3. Verify environment.
- Go 1.24+ required (repo currently targets Go 1.25.x in `go.mod`).

4. Run tests.
```bash
go test ./...
```

5. Build local binary.
```bash
go build -o clihub .
```

## Core commands

Generate from HTTP MCP server:
```bash
./clihub generate --url https://mcp.linear.app/mcp
```

Generate from stdio MCP server:
```bash
./clihub generate --stdio "npx @modelcontextprotocol/server-github" --env GITHUB_TOKEN=$GITHUB_TOKEN
```

List all generation flags:
```bash
./clihub generate --help
./clihub generate --help-auth
```

## Code map

- `cmd/`: command entrypoints (`root`, `generate`)
- `internal/auth/`: auth providers, OAuth flow, credentials
- `internal/schema/`: JSON Schema -> CLI options
- `internal/codegen/`: template rendering for generated CLIs
- `internal/compile/`: platform parsing, build, smoke test
- `internal/nameutil/`: output name inference
- `internal/toolfilter/`: include/exclude matching
- `scripts/`: hooks + version/release scripts

## Docs for contributors

- Project goals and scope: `docs/project/project.md`
- Architecture: `docs/architecture/architecture.md`
- Auth model: `docs/auth/auth.md`
- Release process: `docs/releasing/releasing.md`

Run this to catalog docs and read hints:
```bash
docs-catalog
```

## Contribution workflow

1. Create a branch.
```bash
git checkout -b <type>/<short-name>
```

2. Implement with tests where appropriate.
3. Run local checks (same checks as pre-push hook):
```bash
go vet ./...
go test ./...
```
4. Ensure formatting:
```bash
gofmt -w <changed-go-files>
```
5. Commit using conventional commit style.

## Release notes

Pushes to `main` are gated:
1. `VERSION` must be updated.
2. Annotated tag `vX.Y.Z` must match `VERSION` and point to pushed `main` commit.

Use release scripts:
```bash
bash scripts/bump-version.sh --prepare-push patch
# or
bash scripts/bump-version.sh --sync-tag
```

Detailed release runbook: `docs/releasing/releasing.md`

## License

MIT (`LICENSE`)
