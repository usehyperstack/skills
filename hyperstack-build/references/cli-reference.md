# Hyperstack CLI Reference

All commands accept `--json` for machine-readable output and `--verbose` for debug info. Use `--json` by default in agent workflows.

## IDL Explorer (`hs idl`)

Every subcommand takes `<path>` (the IDL JSON file) as its first argument. Lookups are case-insensitive with fuzzy matching on typos.

### Data inspection

| Command | Description |
|---------|-------------|
| `hs idl summary <path>` | Program overview: name, format, address, section counts |
| `hs idl instructions <path>` | List all instructions (name, account count, arg count) |
| `hs idl instruction <path> <name>` | One instruction's accounts (writable/signer/PDA flags) and args |
| `hs idl accounts <path>` | List all account types |
| `hs idl account <path> <name>` | One account's field layout |
| `hs idl types <path>` | List all custom types and enums |
| `hs idl type <path> <name>` | One type's field layout |
| `hs idl events <path>` | List all events |
| `hs idl errors <path>` | List all error codes and messages |
| `hs idl constants <path>` | List all constants |
| `hs idl search <path> <query>` | Fuzzy-search across all sections |
| `hs idl discriminator <path> <name>` | Compute Anchor discriminator for an instruction or account |

### Relationship analysis

| Command | Description |
|---------|-------------|
| `hs idl relations <path>` | Classify accounts as Entity / Infrastructure / Role / Other |
| `hs idl account-usage <path> <account>` | Which instructions use this account (grouped by writable/signer/readonly) |
| `hs idl links <path> <a> <b>` | Instructions that involve both accounts together |
| `hs idl pda-graph <path>` | PDA seed derivation graph (which accounts seed which PDAs) |
| `hs idl type-graph <path>` | Pubkey field references across types (e.g. `TradeEvent.mint` → ?) |

### Stack integration

| Command | Description |
|---------|-------------|
| `hs idl connect <path> <new-account>` | How a new account connects to existing ones |

Options for `connect`:
- **--existing <a,b,c>** — Comma-separated existing account names
- **--suggest-hs** — Suggest HyperStack integration points (`register_from`, `aggregate`)
- **--json** — Machine-readable output

`connect --suggest-hs` output maps directly to `lookup_index(register_from = ...)` and `#[aggregate]` decisions.

## Project & Config

| Command | Description |
|---------|-------------|
| `hs init` | Create `hyperstack.toml` in current directory |
| `hs config validate` | Validate config file |

### `hyperstack.toml`

```toml
[project]
name = "my-project"

[sdk]
typescript_output_dir = "./frontend/src/generated"
rust_output_dir = "./crates/generated"
typescript_package = "@myorg/my-sdk"
rust_module_mode = false

[[stacks]]
name = "my-game"
stack = "SettlementGame"
typescript_output_file = "./src/generated/game.ts"
rust_output_crate = "./crates/game-stack"
rust_module = true
```

## Auth

| Command | Description |
|---------|-------------|
| `hs auth login` | Authenticate (accepts `--key, -k` for API key) |
| `hs auth logout` | Remove credentials |
| `hs auth status` | Check local auth state |
| `hs auth whoami` | Verify with server |

## Build & Deploy

| Command | Description |
|---------|-------------|
| `hs up [stack-name]` | Push + build + deploy. Accepts `--branch`, `--preview`, `--dry-run` |
| `hs status` | Show deployment status |
| `hs stack list` | List all stacks |
| `hs stack show <name>` | Detailed stack info. Accepts `--version <n>` |
| `hs stack versions <name>` | Version history. Accepts `--limit <n>` |
| `hs stack push [name]` | Push local stacks to remote |
| `hs stack stop <name>` | Stop a running stack. Accepts `--branch`, `--force` |
| `hs stack delete <name>` | Delete a stack. Accepts `--force` |
| `hs stack rollback <name>` | Rollback deployment. Accepts `--to`, `--build`, `--branch`, `--rebuild` |

## SDK Generation

| Command | Description |
|---------|-------------|
| `hs sdk list` | List stacks available for SDK generation |
| `hs sdk create typescript <stack>` | Generate TypeScript SDK. Accepts `--output`, `--package-name`, `--url` |
| `hs sdk create rust <stack>` | Generate Rust SDK. Accepts `--output`, `--crate-name`, `--module`, `--url` |

## Discovery

| Command | Description |
|---------|-------------|
| `hs explore [stack] [entity]` | Browse deployed stacks, entities, and views |
