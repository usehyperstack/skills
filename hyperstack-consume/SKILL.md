---
name: hyperstack-consume
description: Consume Hyperstack streams using TypeScript, React, or Rust SDKs. Covers connecting to stacks, subscribing to views, handling real-time updates, and using typed entity data. Use when the user wants to read or stream Solana data from deployed Hyperstack stacks.
allowed-tools: Bash(hs:*) Bash(npx:hyperstack-cli*)
metadata:
  version: "0.5"
---

# Consuming Hyperstack Streams

## Before You Start

**Always discover the stack schema first:**
```bash
# Use 'hs' (cargo install) or 'hyperstack-cli' (npm install) depending on your setup
hs explore <stack-name> --json
```
This gives you the exact entity names, field paths, types, and available views. Never guess.

> **CLI binary name:** `hs` if installed via cargo, `hyperstack-cli` if installed via npm. Prefer `hs` if available.

## Planning Data Requirements

Before writing code, discover what the stack offers and check if it satisfies the user's needs.

### Step 1: Fetch the Stack Schema FIRST

**Always run `hs explore` before analyzing requirements.** You cannot know what fields exist until you check.

```bash
# Get full schema for the stack
hs explore <stack-name> --json

# Get detailed fields for a specific entity
hs explore <stack-name> <EntityName> --json
```

Review the output to understand:
- Which entities exist (e.g., `OreRound`, `OreMiner`, `OreTreasury`)
- What sections each entity has (e.g., `id`, `state`, `metrics`)
- What fields are in each section (e.g., `state.motherlode`, `id.round_id`)
- What views are available (e.g., `latest`, `list`, custom views)

### Step 2: Compare User Requirements Against Available Fields

Now that you know what's available, map the user's request to actual fields:

| User wants | Required data | Available in stack? |
|------------|---------------|---------------------|
| "Show mining rewards" | reward amount, miner address | ✅ `OreMiner.state.reward`, `OreMiner.id.miner_pubkey` |
| "Current round info" | round ID, total miners | ✅ `OreRound.id.round_id`, `OreRound.state.total_miners` |
| "Transaction history" | past transactions | ❌ Not in stack |

### Step 3: Warn User About Missing Data

**If required data doesn't exist in the stack, tell the user immediately:**

```
❌ "I checked the ore stack schema and it doesn't expose transaction history. 
   The stack tracks live state, not historical transactions. You'd need to 
   query Solana directly via RPC for that data."

⚠️ "The ore stack has current round data but doesn't store historical rounds. 
   You'll get live updates but can't query past rounds."

✅ "The ore stack has everything you need — miner rewards are in OreMiner.state.reward 
   and round info is in OreRound."
```

Don't proceed with implementation if critical data is missing. Clarify with the user first.

### Step 4: Plan Multi-Entity Streaming

If the user's requirements span multiple entities, plan to stream them together:

**Example:** User wants "each miner's rewards for the current round"

From `hs explore ore --json`:
- `OreMiner` has `state.reward` and `id.miner_pubkey`
- `OreRound` has `id.round_id` and `state.total_miners`
- Both have `round_id` as a linkable field

```typescript
// Stream both entities in parallel, correlate by round_id
const [rounds, miners] = await Promise.all([
  hs.views.OreRound.latest.get(),
  hs.views.OreMiner.list.get(),
]);

const currentRoundId = rounds.values().next().value?.id?.round_id;
const minersInRound = [...miners.values()].filter(
  m => m.state?.current_round_id === currentRoundId
);
```

## Common Mistakes

**Never guess entity names or field paths.** Always run `hs explore <stack> --json` first. The LLM's training data may be outdated.

**Don't confuse `.use()` with `.watch()`.** Use `.use()` for simple iteration (emits `T`). Use `.watch()` only when you need the operation type (`upsert`/`patch`/`delete`).

**React hooks return `{ data, isLoading, error }`, not raw arrays.** Always destructure.

**`WatchOptions.skip` is a number (pagination), not field names.** Use `{ skip: 20, take: 10 }` for pagination.

## Installation

### TypeScript (framework-agnostic)
```bash
npm install hyperstack-typescript
# For prepackaged stacks:
npm install hyperstack-stacks
```

### React
```bash
npm install hyperstack-react hyperstack-stacks
```

### Rust
```toml
[dependencies]
hyperstack-sdk = "0.5"
```

## TypeScript SDK

### Connecting

```typescript
import { HyperStack } from 'hyperstack-typescript';
import { ORE_STREAM_STACK } from 'hyperstack-stacks/ore';

// Stack definition includes the WebSocket URL and type info
const hs = await HyperStack.connect(ORE_STREAM_STACK);
```

For custom stacks (after running `hs sdk create typescript`):
```typescript
import { HyperStack } from 'hyperstack-typescript';
import MY_STACK from './generated/my-stack';

const hs = await HyperStack.connect(MY_STACK);
```

### Streaming Data

#### `.use()` — Stream Merged Entities (Simplest)

```typescript
// Stream entities with full type safety
for await (const round of hs.views.OreRound.latest.use()) {
  console.log("Round:", round.id.round_id);
  console.log("Motherlode:", round.state.motherlode);
}

// With pagination options
for await (const round of hs.views.OreRound.latest.use({
  take: 10,    // Limit to 10 items
  skip: 20,    // Skip first 20 (pagination)
})) {
  // ...
}
```

#### `.watch()` — Stream Raw Updates

Use when you need to know the operation type:

```typescript
for await (const update of hs.views.OreRound.latest.watch()) {
  switch (update.type) {
    case 'upsert':
      console.log('Created or replaced:', update.data.id?.round_id);
      break;
    case 'patch':
      console.log('Partial update:', update.data);
      break;
    case 'delete':
      console.log('Deleted:', update.key);
      break;
  }
}

// With options
for await (const update of hs.views.OreRound.latest.watch({
  take: 5,
  skip: 20,
})) {
  // ...
}
```

#### `.watchRich()` — Stream with Before/After Diffs

```typescript
for await (const update of hs.views.OreRound.latest.watchRich()) {
  switch (update.type) {
    case 'created':
      console.log('New entity:', update.data);
      break;
    case 'updated':
      console.log(`Changed: ${update.before.state.motherlode} → ${update.after.state.motherlode}`);
      break;
    case 'deleted':
      console.log('Removed:', update.lastKnown);
      break;
  }
}
```

### One-Shot Reads

```typescript
// List view — returns T[]
const rounds = await hs.views.OreRound.list.get();

// State view — returns T | null (null means not found)
const round = await hs.views.OreRound.state.get(roundAddress);

// Synchronous — returns T[] | undefined (undefined means not yet loaded, distinct from empty)
const cached = hs.views.OreRound.list.getSync();

// State getSync — three distinct values:
//   T          → found
//   null        → not found
//   undefined   → not yet loaded (subscription hasn't received data yet)
const cachedRound = hs.views.OreRound.state.getSync(roundAddress);
```



### Schema Validation

Pass a `schema` to `.use()`, `.watch()`, or `.watchRich()` to filter out entities that don't pass validation. Entities that fail are silently skipped — they never reach your loop:

```typescript
import { OreRoundCompletedSchema } from 'hyperstack-stacks/ore';

// Only emit rounds where every field is present
for await (const round of hs.views.OreRound.latest.use({
  schema: OreRoundCompletedSchema,
})) {
  // round.id and round.state are guaranteed non-null here
  console.log(round.id.round_id, round.state.motherlode);
}
```

Every stack ships two schema variants per entity:
- `OreRoundSchema` — all fields optional (matches partial streaming frames)
- `OreRoundCompletedSchema` — all fields required (use to assert fully-hydrated entities)

You can also define a custom Zod schema to validate only the fields your code needs:

```typescript
import { z } from 'zod';

const TradableTokenSchema = z.object({
  id: z.object({ mint: z.string() }),
  reserves: z.object({ current_price_sol: z.number() }),
});

for await (const token of hs.views.PumpfunToken.list.use({
  schema: TradableTokenSchema,
})) {
  console.log(token.id.mint, token.reserves.current_price_sol);
}
```

Enable `validateFrames` on connect to validate every incoming WebSocket frame and drop malformed data before it enters the local store:

```typescript
const hs = await HyperStack.connect(ORE_STREAM_STACK, {
  validateFrames: true,
});
```

### Stream Control

#### Stop on Condition

Break out of the loop to stop streaming:

```typescript
for await (const round of hs.views.OreRound.latest.use()) {
  if ((round.state.motherlode ?? 0) > 1_000_000_000) {
    console.log("Found high-value round:", round.id.round_id);
    break; // stops the stream
  }
}
```

#### Cancellable Streams (AbortController)

Cancel from outside the loop — useful for timeouts or user-initiated stops:

```typescript
const controller = new AbortController();
setTimeout(() => controller.abort(), 30_000); // cancel after 30s

try {
  for await (const round of hs.views.OreRound.latest.use()) {
    if (controller.signal.aborted) break;
    console.log("Round:", round.id.round_id);
  }
} catch (e) {
  if (!controller.signal.aborted) throw e;
}
```

## React SDK

### Setup Provider

```tsx
import { HyperstackProvider } from 'hyperstack-react';

function App() {
  return (
    <HyperstackProvider>
      <MiningDashboard />
    </HyperstackProvider>
  );
}
```

### Using Hooks

```tsx
import { useHyperstack } from 'hyperstack-react';
import { ORE_STREAM_STACK } from 'hyperstack-stacks/ore';

function MiningDashboard() {
  const { views, isConnected } = useHyperstack(ORE_STREAM_STACK);

  // .use() returns { data, isLoading, error }
  const { data: rounds, isLoading } = views.OreRound.latest.use();

  if (isLoading) return <p>Connecting...</p>;

  return (
    <ul>
      {rounds?.map((round) => (
        <li key={round.id?.round_id}>
          Round #{round.id?.round_id}
          Motherlode: {round.state?.motherlode}
          Miners: {round.state?.total_miners}
        </li>
      ))}
    </ul>
  );
}
```

### Schema Filtering

Pass a `schema` to `.use()` or `.useOne()` to silently exclude entities that fail validation. Use the generated "Completed" variant to ensure all fields are present before rendering:

```tsx
import { OreRoundCompletedSchema } from 'hyperstack-stacks/ore';

function FullRounds() {
  const { views } = useHyperstack(ORE_STREAM_STACK);

  // Entities missing required fields are filtered out automatically
  const { data: rounds } = views.OreRound.latest.use({
    schema: OreRoundCompletedSchema,
  });

  return (
    <ul>
      {rounds?.map((round) => (
        // No optional chaining needed — schema guarantees all fields present
        <li key={round.id.round_id}>Round #{round.id.round_id}</li>
      ))}
    </ul>
  );
}
```

Custom Zod schemas work too — validate only the fields your component needs:

```tsx
import { z } from 'zod';

const TradableTokenSchema = z.object({
  id: z.object({ mint: z.string() }),
  reserves: z.object({ current_price_sol: z.number() }),
});

function TokenList() {
  const { views } = useHyperstack(PUMPFUN_STREAM_STACK);
  const { data: tokens } = views.PumpfunToken.list.use({
    schema: TradableTokenSchema,
  });
  // ...
}
```

### Single Item Query

```tsx
function LatestRound() {
  const { views } = useHyperstack(ORE_STREAM_STACK);

  // useOne() returns single entity or undefined
  const { data: round, isLoading } = views.OreRound.latest.useOne();

  if (isLoading) return <p>Loading...</p>;
  if (!round) return <p>No rounds</p>;

  return <div>Round #{round.id?.round_id}</div>;
}
```

### Conditional Subscription

Use `enabled` to prevent subscribing until a required value is available:

```tsx
function RoundDetail({ roundAddress }: { roundAddress: string | null }) {
  const { views } = useHyperstack(ORE_STREAM_STACK);

  const { data: round } = views.OreRound.state.use(
    { key: roundAddress ?? "" },
    { enabled: !!roundAddress }, // won't subscribe until address is set
  );
}
```

### Initial Data

Use `initialData` to avoid a loading flash — show placeholder data immediately while the real data loads:

```tsx
const { data: rounds } = views.OreRound.list.use({}, { initialData: [] });
// rounds is [] immediately, replaced with real data on first update
```

### Connection State

```tsx
function StatusBar() {
  const { connectionState, isConnected } = useHyperstack(ORE_STREAM_STACK);
  // connectionState: 'connecting' | 'connected' | 'disconnected' | 'reconnecting' | 'error'

  return <div>Status: {isConnected ? "🟢 Live" : connectionState}</div>;
}
```

### Accessing Core SDK

`hyperstack-react` re-exports everything from `hyperstack-typescript`. You don't need to install both packages — import low-level APIs directly from the React package:

```typescript
import { HyperStack, ConnectionManager } from "hyperstack-react";
```

## Rust SDK

```rust
use hyperstack_sdk::prelude::*;
use hyperstack_stacks::ore::{OreStack, OreRound};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let hs = HyperStack::<OreStack>::connect().await?;

    // Stream entities with .listen()
    let mut stream = hs.views.ore_round.latest().listen();
    while let Some(round) = stream.next().await {
        println!("Round: {:?}", round.id.round_id);
        println!("Motherlode: {:?}", round.state.motherlode);
    }

    Ok(())
}
```

## Working with Entity Data

All entity fields are organized in sections. For example, `OreRound` has:
- `id` section: `round_id`, `round_address`
- `state` section: `motherlode`, `total_miners`, `total_deployed`, `expires_at`, `estimated_expires_at_unix`
- `metrics` section: `deploy_count`, `checkpoint_count`
- `results` section: `top_miner`, `top_miner_reward`, `did_hit_motherlode`, `winning_square`
- `entropy` section: `entropy_value`, `entropy_seed`
- `ore_metadata` section: `mint`, `name`, `symbol`, `decimals`

Access fields via dot notation: `round.state?.motherlode`, `round.id?.round_id`

**Always check `hs explore <stack> <entity> --json` for the exact field paths and types for any stack.**

## Common Patterns

### Filtering Updates

```typescript
for await (const update of hs.views.OreRound.list.watch()) {
  if (update.type !== 'upsert') continue;
  const round = update.data;

  // Filter for rounds with large motherlode
  if ((round.state?.motherlode ?? 0) > 1000) {
    console.log('Big motherlode:', round.id?.round_id, round.state?.motherlode);
  }
}
```

### Multiple Views

```typescript
const hs = await HyperStack.connect(ORE_STREAM_STACK);

// Stream rounds and treasury in parallel
const streamRounds = async () => {
  for await (const update of hs.views.OreRound.latest.watch({ take: 1 })) {
    console.log('Round:', update.data.id?.round_id);
  }
};

const streamTreasury = async () => {
  for await (const update of hs.views.OreTreasury.list.watch({ take: 1 })) {
    console.log('Treasury balance:', update.data.state?.balance);
  }
};

await Promise.all([streamRounds(), streamTreasury()]);
```

## Generating SDK Types for Custom Stacks

If someone has deployed a custom stack and shared its WebSocket URL:
```bash
hs sdk create typescript <stack-name> --url wss://their-stack.stack.usehyperstack.com
```

For reference details, see the `references/` directory.
