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

## Dependency Setup

Run this block before any CLI commands. It detects missing tools and installs them automatically.

```bash
# ── 1. Detect OS ────────────────────────────────────────────────────────────
OS="$(uname -s 2>/dev/null || echo Windows)"

# ── 2. Ensure cargo is available (needed if installing via cargo) ────────────
if ! command -v cargo &>/dev/null; then
  echo "cargo not found — installing Rust toolchain via rustup..."
  if [ "$OS" = "Darwin" ] || [ "$OS" = "Linux" ]; then
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --no-modify-path
    # shellcheck source=/dev/null
    source "$HOME/.cargo/env"
  else
    # Windows — download rustup-init.exe
    curl -sSLo /tmp/rustup-init.exe https://win.rustup.rs/x86_64
    /tmp/rustup-init.exe -y
    export PATH="$USERPROFILE/.cargo/bin:$PATH"
  fi
fi

# ── 3. Ensure npm is available (needed if installing via npm) ────────────────
if ! command -v npm &>/dev/null; then
  echo "npm not found — installing Node.js..."
  if [ "$OS" = "Darwin" ]; then
    if command -v brew &>/dev/null; then
      brew install node
    else
      # Install nvm, then node LTS
      curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
      export NVM_DIR="$HOME/.nvm" && [ -s "$NVM_DIR/nvm.sh" ] && source "$NVM_DIR/nvm.sh"
      nvm install --lts && nvm use --lts
    fi
  elif [ "$OS" = "Linux" ]; then
    if command -v apt-get &>/dev/null; then
      sudo apt-get update -qq && sudo apt-get install -y nodejs npm
    elif command -v dnf &>/dev/null; then
      sudo dnf install -y nodejs npm
    else
      curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
      export NVM_DIR="$HOME/.nvm" && [ -s "$NVM_DIR/nvm.sh" ] && source "$NVM_DIR/nvm.sh"
      nvm install --lts && nvm use --lts
    fi
  else
    # Windows — use winget if available, else instruct manually
    if command -v winget &>/dev/null; then
      winget install OpenJS.NodeJS.LTS -e --silent
      export PATH="/c/Program Files/nodejs:$PATH"
    else
      echo "Please install Node.js from https://nodejs.org and re-run." && exit 1
    fi
  fi
fi

# ── 4. Detect / install Hyperstack CLI ──────────────────────────────────────
if command -v hs &>/dev/null; then
  HS_CLI="hs"
elif command -v hyperstack-cli &>/dev/null; then
  HS_CLI="hyperstack-cli"
else
  echo "Hyperstack CLI not found — installing via cargo (recommended)..."
  cargo install hyperstack-cli
  HS_CLI="hs"
fi

echo "Using Hyperstack CLI: $HS_CLI"
```

> **Prefer `hs`** (cargo install) when possible — it's the primary binary name used in all examples.
> If you installed via npm (`npm install -g hyperstack-cli`), the binary is `hyperstack-cli`.
> The block above sets `$HS_CLI` — use it for all subsequent commands.

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
