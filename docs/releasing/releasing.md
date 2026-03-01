---
summary: Release process for clihub, including version and annotated-tag requirements.
read_when:
  - You are preparing a release or pushing to main.
  - A push to main is blocked by VERSION or tag checks.
  - You need to recover from missing or misaligned release tags.
---

# Releasing

## Release invariants

Pushes to `main` are gated by the pre-push hook and must satisfy both:

1. `VERSION` is updated in the pushed commit(s).
2. Annotated tag `vX.Y.Z` exists, where `X.Y.Z` matches `VERSION` in the pushed `main` commit, and the tag points to that exact commit.

These checks are enforced by:
- `/.githooks/pre-push`
- `/scripts/pre-push-checks.sh`

## Prerequisites

1. Repo hooks installed: `bash scripts/install-hooks.sh`
2. Clean local state for release commit actions.
3. Passing checks:
- `go test ./...`
- `go vet ./...`
- formatting (`gofmt`)
- optional `golangci-lint run` if installed

`install-hooks.sh` also sets `push.followTags=true` in local git config.

## Standard release flow

For a normal patch release from `main`:

```bash
bash scripts/bump-version.sh --prepare-push patch
git push --follow-tags origin main
```

What `--prepare-push` does:
1. Bumps `VERSION`.
2. Stages `VERSION`.
3. Amends `HEAD` commit (`git commit --amend --no-edit`).
4. Creates or replaces annotated tag `vX.Y.Z` on amended `HEAD`.

Use `minor` or `major` instead of `patch` when needed.

## Tag sync / repair flow

If `VERSION` is already correct in `HEAD` but tag is missing or points elsewhere:

```bash
bash scripts/bump-version.sh --sync-tag
git push --follow-tags origin main
```

This ensures an annotated tag matching current `VERSION` points at current `HEAD`.

## Validation commands

Validate `VERSION` format:

```bash
bash scripts/bump-version.sh --verify
```

Inspect current version and tags:

```bash
cat VERSION
git tag --list 'v*' --sort=-version:refname | head
```

## Common push failures and fixes

1. Error: `VERSION not updated` when pushing to `main`.
- Fix: run `bash scripts/bump-version.sh --prepare-push patch`.

2. Error: missing release tag `vX.Y.Z`.
- Fix: run `bash scripts/bump-version.sh --sync-tag`.

3. Error: tag is lightweight instead of annotated.
- Fix: run `bash scripts/bump-version.sh --sync-tag` (recreates annotated tag).

4. Error: tag does not point at pushed commit.
- Fix: run `bash scripts/bump-version.sh --sync-tag` after confirming correct `HEAD`.

## Notes for contributors

1. The release script amends the latest commit when `--prepare-push` is used.
2. Do not run release bump/tag flows on unrelated feature branches unless you intend release semantics there.
3. If you bypass hooks (`--no-verify`), you are responsible for meeting release invariants before merge/push.

## CI and release hygiene

Before tagging/pushing to `main`:
1. Ensure test and vet pass locally.
2. Confirm docs are updated for any behavior/API changes.
3. Confirm release commit message and content are intentional after amend.
