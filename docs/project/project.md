---
summary: Project overview, goals, scope, and roadmap anchors for clihub.
read_when:
  - You are new to this repository and need the high-level context first.
  - You need to confirm whether a change aligns with clihub's core goals.
  - You are deciding if a request belongs in scope or should be deferred.
---

# Project

## What clihub is

`clihub` turns an MCP server into a compiled CLI binary.

Given an MCP endpoint (HTTP or stdio), clihub:
1. Connects and initializes MCP.
2. Discovers tools.
3. Generates Go code with one subcommand per tool.
4. Compiles to a standalone binary.

The generated binary can run without clihub at runtime.

## Problem it solves

MCP servers are powerful, but many workflows still need plain CLI binaries:
- automation in scripts/CI
- shell-first and SSH workflows
- environments where agents are not always running
- distribution of a single executable with no runtime dependency on clihub

clihub bridges MCP servers to standard CLI workflows.

## Goals

### Primary goals

1. One-command generation from MCP server to working CLI binary.
2. Broad auth support (OAuth, bearer, API key, basic, S2S OAuth2, Google SA).
3. Predictable generated interfaces from JSON Schema-derived flags.
4. Cross-platform output through Go cross-compilation.
5. Agent-friendly operation with clear errors and low setup friction.

### Quality goals

1. Safe credential handling and migration compatibility.
2. Good failure diagnostics for handshake, auth, schema, and compile paths.
3. Deterministic generation behavior for repeatable automation.
