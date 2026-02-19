# Hyperstack Skills

Agent skills for building with [Hyperstack](https://usehyperstack.com), real-time Solana data streaming.

## Install

```bash
npx skills add usehyperstack/skills
```

## Skills

| Skill | Description |
|-------|-------------|
| `hyperstack` | Router skill, detects your intent and points to the right sub-skill |
| `hyperstack-consume` | SDK patterns for consuming streams (TypeScript, React, Rust) |
| `hyperstack-build` | DSL syntax for building custom stacks from Anchor IDLs |

## How It Works

These skills teach AI coding agents how to use Hyperstack. They reference the `hs` CLI for live data discovery, so type information is always accurate for the version you have installed.

Install the CLI: `npm install -g hyperstack-cli` or `cargo install hyperstack-cli`
