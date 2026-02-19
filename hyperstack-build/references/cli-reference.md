# Hyperstack CLI Reference

## Global Options

- **--config, -c <path>** — Path to hyperstack.toml (default: `hyperstack.toml`)
- **--json** — Output as JSON
- **--verbose** — Enable verbose output
- **--help, -h** — Show help
- **--version, -V** — Show version
- **--completions <SHELL>** — Generate shell completions (bash, zsh, fish, powershell, elvish)

## Project Setup

### `hs init`
Initializes a new Hyperstack project in the current directory.

### `hs config validate`
Validate your hyperstack.toml configuration.

## Authentication

### `hs auth login`
Authenticates the CLI with your Hyperstack account.
- **--key, -k** — API key (prompts if not provided)

### `hs auth logout`
Remove stored credentials.

### `hs auth status`
Check local authentication status.

### `hs auth whoami`
Verify authentication with server.

## Deployment

### `hs up [stack-name]`
Builds and deploys the stack to the Hyperstack cloud.
- **--branch, -b <name>** — Deploy to a specific branch
- **--preview** — Create preview deployment
- **--dry-run** — Show what would be deployed without deploying

### `hs status`
Shows the status of your deployed stacks.
- **--json** — Output as JSON

## Stack Management

### `hs stack list`
Lists all stacks in your account.
- **--json** — Output as JSON

### `hs stack push [stack-name]`
Push local stacks to remote.

### `hs stack show <stack-name>`
Show detailed stack information including deployment status and versions.
- **--version, -v <n>** — Show specific version details

### `hs stack versions <stack-name>`
Show version history.
- **--limit, -l <n>** — Maximum number of versions (default: 20)

### `hs stack delete <stack-name>`
Delete a stack from remote.
- **--force, -f** — Skip confirmation prompt

### `hs stack rollback <stack-name>`
Rollback to a previous deployment.
- **--to <version>** — Target version
- **--build <id>** — Target build ID
- **--branch <name>** — Branch to rollback (default: production)
- **--rebuild** — Force full rebuild
- **--no-wait** — Don't watch progress

### `hs stack stop <stack-name>`
Stops a running stack.
- **--branch <name>** — Branch deployment to stop
- **--force, -f** — Skip confirmation prompt

## SDK Generation

### `hs sdk list`
List stacks available for SDK generation.

### `hs sdk create typescript <stack-name>`
Generates TypeScript SDK types for the given stack.
- **--output, -o <path>** — Output file path (overrides config)
- **--package-name, -p <name>** — Package name for TypeScript
- **--url <url>** — WebSocket URL for the stack

### `hs sdk create rust <stack-name>`
Generates Rust SDK types for the given stack.
- **--output, -o <path>** — Output directory path (overrides config)
- **--crate-name <name>** — Custom crate name for generated Rust crate
- **--module** — Generate as module instead of crate
- **--url <url>** — WebSocket URL for the stack

## Discovery

### `hs explore [stack-name] [entity-name]`
Discovers available stacks, entities, and views.
- **--json** — Output schema info in JSON format

## Telemetry

### `hs telemetry status`
Show current telemetry status.

### `hs telemetry enable`
Enable telemetry collection.

### `hs telemetry disable`
Disable telemetry collection.

## Configuration

### `hyperstack.toml`
The main configuration file for your Hyperstack project.
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
