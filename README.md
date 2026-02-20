# Hyperstack Skills

Agent skills for building with [Hyperstack](https://docs.usehyperstack.com), real-time Solana data streaming.

## Install

```bash
npx skills add usehyperstack/skills
```

## Skills

| Skill | Description |
|-------|-------------|
| `hyperstack` | Router skill, detects your intent and points to the right sub-skill |
| `hyperstack-consume` | SDK patterns for consuming streams (TypeScript, React, Rust) |
| `hyperstack-build` | DSL syntax for building custom stacks from Solana program IDLs |

## How It Works

These skills teach AI coding agents how to use Hyperstack. They reference the CLI for live data discovery, so type information is always accurate for the version you have installed.

### CLI Installation

The CLI binary name depends on how you install it:

| Install Method | Command | Binary Name |
|---------------|---------|-------------|
| Cargo (recommended) | `cargo install hyperstack-cli` | `hs` |
| npm | `npm install -g hyperstack-cli` | `hyperstack-cli` |

The skills prefer `hs` if available. All examples use `hs` — substitute `hyperstack-cli` if you installed via npm.
