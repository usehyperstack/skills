---
name: hyperstack
description: Build with Hyperstack, real-time Solana data streaming. Covers consuming existing streams and building custom stacks. Use when the user mentions Hyperstack, real-time Solana data, on-chain streaming, ORE mining, or wants to consume/build data stacks.
allowed-tools: Bash(hs:*) Bash(npx:hyperstack-cli*)
metadata:
  version: "0.5"
---

# Hyperstack

Hyperstack provides real-time streaming data from Solana programs via WebSocket. Think of it as a typed, structured pub/sub system for on-chain state.

**Key concepts:**
- **Stack**: A deployed streaming server that watches Solana programs and produces structured entity updates
- **Entity**: A domain object (e.g. `OreRound`, `OreMiner`) with typed fields organized in sections
- **View**: A lens on an entity, either `state` (keyed lookup) or `list` (collection). Custom views can add sorting and filtering.
- **SDK**: TypeScript, React, or Rust client libraries that connect to a stack's WebSocket and provide typed access to views

## CLI Setup

The Hyperstack CLI has different binary names depending on how it was installed:
- **cargo install**: `hs`
- **npm install -g**: `hyperstack-cli`

**Before running any CLI commands, detect which binary is available:**

```bash
# Check which CLI is available (prefer hs)
if command -v hs &> /dev/null; then
  HS_CLI="hs"
elif command -v hyperstack-cli &> /dev/null; then
  HS_CLI="hyperstack-cli"
else
  echo "Hyperstack CLI not found. Install with: cargo install hyperstack-cli (recommended) or npm install -g hyperstack-cli"
  exit 1
fi
```

Then use `$HS_CLI` for all commands, or just use `hs` directly if you confirmed it's available.

## Quick Start

```bash
# Scaffold a new app from a template
npx hyperstack-cli create my-app

# Or install the CLI globally (cargo is preferred)
cargo install hyperstack-cli  # Installs as 'hs'
# OR
npm install -g hyperstack-cli  # Installs as 'hyperstack-cli'
```

## Discovering Available Stacks and Schemas

**CRITICAL: Always run the explore command before writing Hyperstack code.** Never guess entity names, field paths, or types.

```bash
# List all available stacks (use 'hs' or 'hyperstack-cli' depending on your install)
hs explore --json

# Get entities and views for a specific stack
hs explore <stack-name> --json

# Get detailed field info for a specific entity
hs explore <stack-name> <EntityName> --json
```

> **Note:** All CLI examples use `hs`. If you installed via npm, substitute `hyperstack-cli` for `hs`.

The `--json` flag gives machine-readable output with exact field paths, types, and view definitions.

## Which Skill Do You Need?

**Consuming an existing stack** (connecting to streams, subscribing to data, building UIs):
-> See the `hyperstack-consume` skill

**Building your own stack** (writing Rust DSL, defining entities from program IDLs, deploying):
-> See the `hyperstack-build` skill

## Prepackaged Stacks

Run `hs explore` to see what's available. Current public stacks include:
- **ore**, ORE mining rounds, miners, treasury state

Install the prepackaged SDK types: `npm install hyperstack-stacks`
