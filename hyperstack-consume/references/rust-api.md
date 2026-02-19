# Rust SDK API Reference

## Connection

### `HyperStack::<T>::connect()`
Asynchronously connects to a Hyperstack WebSocket.
- **T**, The generated Stack type.
- **Note**: The WebSocket URL is embedded in the stack definition (e.g., `ORE_STREAM_STACK`). No URL argument is needed.

## View Methods

### `.listen()`
Returns a stream that yields updates. Use with `.next().await` to iterate over updates.

## Integration

### Tokio
The Rust SDK uses `tokio` for async execution and communication. Ensure you have a `#[tokio::main]` attribute or a running `tokio` runtime.

Add to `Cargo.toml`:
```toml
[dependencies]
hyperstack-sdk = "0.5"
tokio = { version = "1", features = ["full"] }
```

## Example

```rust
use hyperstack_sdk::prelude::*;
use hyperstack_stacks::ore::{OreStack, OreRound};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let hs = HyperStack::<OreStack>::connect().await?;
    
    let mut stream = hs.views.ore_round.latest().listen().take(1);
    while let Some(round) = stream.next().await {
        println!("Round # {:?}", round.id.round_id);
        println!("Motherlode: {:?}", round.state.motherlode);
    }
    
    Ok(())
}
```

This example is from the [official Hyperstack documentation](https://docs.usehyperstack.com).
