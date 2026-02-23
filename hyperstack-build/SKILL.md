---
name: hyperstack-build
description: Build custom Hyperstack stacks from Solana program IDLs using the Rust DSL. Covers entity definitions, field mappings, views, computed fields, PDA resolution, and deployment with the hs CLI. Use when the user wants to create their own real-time data streaming stack.
allowed-tools: Bash(hs:*) Bash(cargo:*)
---

# Building Hyperstack Stacks

## Prerequisites

Required tools: Rust toolchain (`rustup` + `cargo`), Hyperstack CLI (`hs`), and an IDL JSON file for the Solana program(s) you want to stream.

Run this block once before doing anything else. It detects missing tools and installs them.

```bash
# ── 1. Detect OS ────────────────────────────────────────────────────────────
OS="$(uname -s 2>/dev/null || echo Windows)"

# ── 2. Ensure Rust / cargo ──────────────────────────────────────────────────
if ! command -v cargo &>/dev/null; then
  echo "cargo not found — installing Rust toolchain via rustup..."
  if [ "$OS" = "Darwin" ] || [ "$OS" = "Linux" ]; then
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --no-modify-path
    # shellcheck source=/dev/null
    source "$HOME/.cargo/env"
  else
    # Windows
    curl -sSLo /tmp/rustup-init.exe https://win.rustup.rs/x86_64
    /tmp/rustup-init.exe -y
    export PATH="$USERPROFILE/.cargo/bin:$PATH"
  fi
else
  echo "cargo $(cargo --version) — OK"
fi

# ── 3. Ensure Hyperstack CLI (hs) ────────────────────────────────────────────
if command -v hs &>/dev/null; then
  HS_CLI="hs"
  echo "hs $(hs --version 2>/dev/null) — OK"
elif command -v hyperstack-cli &>/dev/null; then
  HS_CLI="hyperstack-cli"
  echo "hyperstack-cli found (npm install) — OK"
else
  echo "Hyperstack CLI not found — installing via cargo..."
  cargo install hyperstack-cli
  HS_CLI="hs"
fi
```

> **CLI binary name:** `cargo install` creates the `hs` binary. `npm install -g hyperstack-cli` creates the `hyperstack-cli` binary. All examples below use `hs` — substitute `$HS_CLI` if you ran the block above.

## Finding the IDL

You need the IDL JSON for the program you want to stream. Hyperstack supports Anchor and other framework IDL formats. Try these in order:

**1. Program GitHub repo** — Most Solana programs are open source. Common locations:
- `target/idl/program_name.json`
- `idl/program_name.json`
- The Releases page as an attached asset

**2. Anchor CLI fetch** — If the developer uploaded the IDL on-chain:
```bash
anchor idl fetch <PROGRAM_ID> --provider.cluster mainnet -o idl/program.json
```
Returns an error if no IDL is on-chain.

**3. NPM / Rust packages** — Many protocols publish SDKs that bundle the IDL:
- NPM: check `node_modules/@protocol/sdk/dist/idl.json` or similar
- Crates.io: some Rust crates include the IDL as a resource

**4. Block explorers** — Solscan and Solana.fm sometimes host IDLs for verified programs. Look for a "Contract" or "IDL" tab on the program address page.

**5. Manual creation** — If an IDL isn't available, tools like Kinobi or Codama can generate one by parsing the Rust source.

### Where to put it

Store IDL files in an `idl/` directory at the root of your stack:

```
my-stack/
├── idl/
│   └── program_name.json
├── src/
│   └── stack.rs
├── hyperstack.toml
└── Cargo.toml
```

Then reference it with the `#[hyperstack]` macro (see Stack Definition below).

## Project Setup

```bash
cargo new --lib my-stack
cd my-stack
```

Add to `Cargo.toml`:
```toml
[dependencies]
hyperstack = "0.1"

[build-dependencies]
hyperstack-build = "0.1"

# Required for the macro to find your IDL
[package.metadata.hyperstack]
idls = ["idl/my_program.json"]
```

## Stack Definition

A stack watches Solana programs and maps on-chain account state to structured entities.

```rust
use hyperstack::prelude::*;

#[hyperstack(idl = ["idl/my_program.json"])]
mod my_stack {
    /// Token with live pricing data
    #[entity]
    #[primary_key(id.mint)]
    pub struct MyToken {
        pub id: MyTokenId,
        pub state: MyTokenState,
    }

    #[derive(Stream)]
    pub struct MyTokenId {
        /// The token's mint address
        #[map(my_program::accounts::TokenState, key = mint)]
        pub mint: String,
    }

    #[derive(Stream)]  
    pub struct MyTokenState {
        /// Current balance in lamports
        #[map(my_program::accounts::TokenState::balance, strategy = LastWrite)]
        pub balance: u64,
        
        /// Total number of transfers
        #[map(my_program::accounts::TokenState::transfer_count, strategy = LastWrite)]
        pub transfer_count: u64,
    }
}
```

## Key Concepts

### Entities
An entity is a domain object tracked by the stack. Each entity has a primary key and sections.

### Sections  
Sections group related fields. They map to the top-level sections in the TypeScript/Rust SDK types. Any field whose type derives `Stream` is automatically a section — no `#[section]` annotation needed.

### Field Mappings
`#[map(...)]` attributes define how on-chain account data maps to entity fields.

Mapping strategies:
- `LastWrite` — Use the most recent value (default)
- `Append` — Append to a `Vec`
- `Merge` — Deep-merge objects (for nested structs)
- `Sum` — Accumulate numeric values
- `Count` — Increment by 1 for each occurrence
- `Min` / `Max` — Track extremes
- `UniqueCount` — Track unique values and store the count

### Advanced Field Macros

**`#[from_instruction]`** — Maps a field from an instruction's arguments or accounts:
```rust
#[from_instruction(PlaceTrade::amount, strategy = Append)]
pub trade_amounts: Vec<u64>,
```

**`#[snapshot]`** — Captures the entire state of a source account:
```rust
#[snapshot(from = BondingCurve, strategy = LastWrite)]
pub latest_state: BondingCurve,
```

**`#[aggregate]`** — Declarative aggregation from instructions:
```rust
#[aggregate(from = [Buy, Sell], field = amount, strategy = Sum)]
pub total_volume: u64,
```

**`#[derive_from]`** — Derives from instruction metadata (`__timestamp`, `__slot`, `__signature`):
```rust
#[derive_from(from = [Buy, Sell], field = __timestamp)]
pub last_updated: i64,
```

**`TokenMetadata`** — Automatically enriches entities with off-chain token metadata. Add a field typed as `TokenMetadata` and Hyperstack resolves it server-side from the entity's mint address.

### Views
Views are lenses on entities. Every entity automatically gets `state` (keyed lookup) and `list` (all items) views.

Custom views:
```rust
#[entity]
#[view(latest, sort_by = "id.round_id", order = "desc")]
pub struct OreRound { ... }
```

### Computed Fields
Fields derived from other fields:
```rust
#[computed(expr = "reserves.virtual_sol_reserves / reserves.virtual_token_reserves")]
pub current_price: f64,
```

### PDA Resolution
Resolve related accounts via PDA derivation:
```rust
#[resolve(
    program = "my_program",
    seeds = [b"user-account", user_pubkey],
    extracts = [balance, last_active]
)]
pub user_data: UserData,
```

## Configuration

`hyperstack.toml`:
```toml
[project]
name = "my-project"

[sdk]
typescript_output_dir = "./frontend/src/generated"
rust_output_dir = "./crates/generated"
```

## Build & Deploy

```bash
# Build the stack (generates .hyperstack/*.stack.json)
cargo build

# Authenticate
hs auth login

# Deploy (push, build, and deploy)
hs up my-stack

# Generate SDK types
hs sdk create typescript my-stack
hs sdk create rust my-stack

# Check status
hs status
```

## Branch Deploys

```bash
# Deploy to a branch URL
hs up my-stack --branch staging
# URL: wss://my-stack-staging.stack.usehyperstack.com

# Clean up
hs stack stop my-stack --branch staging
```

## Verifying Your Deployment

```bash
# See what's deployed
hs explore my-stack --json

# Check entity schemas
hs explore my-stack MyToken --json
```

For DSL reference, see the `references/` directory.
