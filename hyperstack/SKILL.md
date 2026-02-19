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

## Quick Start

```bash
# Scaffold a new app from a template
npx hyperstack-cli create my-app

# Or install the CLI globally
npm install -g hyperstack-cli
# Then: hs explore to discover available stacks
```

## Discovering Available Stacks and Schemas

**CRITICAL: Always run `hs explore` before writing Hyperstack code.** Never guess entity names, field paths, or types.

```bash
# List all available stacks
hs explore --json

# Get entities and views for a specific stack
hs explore <stack-name> --json

# Get detailed field info for a specific entity
hs explore <stack-name> <EntityName> --json
```

The `--json` flag gives machine-readable output with exact field paths, types, and view definitions.

## Which Skill Do You Need?

**Consuming an existing stack** (connecting to streams, subscribing to data, building UIs):
-> See the `hyperstack-consume` skill

**Building your own stack** (writing Rust DSL, defining entities from Anchor IDLs, deploying):
-> See the `hyperstack-build` skill

## Prepackaged Stacks

Run `hs explore` to see what's available. Current public stacks include:
- **ore**, ORE mining rounds, miners, treasury state

Install the prepackaged SDK types: `npm install hyperstack-stacks`
