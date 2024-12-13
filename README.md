# tracing-subscribe-sqlite

A tracing Subscriber to send log to tokio broadcast channel (WIP).

## Usage

```toml
[dependencies]
tracing-subscriber-channel = "0.1"
```

```rust
use tokio::sync::broadcast::channel;
use tracing_subscriber_channel::SubscriberBuilder;

#[tokio::main]
async fn main() {
    let (tx, mut rx) = channel(10);
    tracing::subscriber::set_global_default(
        SubscriberBuilder::new()
            .with_black_list(["h2::client", "h2::codec"])
            .build(tx),
    )
        .unwrap();
    tracing::info!(x = 100, "test info");
    tracing::debug!(x = 100, "test trace");
    while let Ok(entry) = rx.recv().await {
        println!(
            "{} [{}] [{:?}/{:?}:{:?}] {}, data: {}",
            entry.time,
            entry.level.as_str(),
            entry.module.unwrap(),
            entry.file.unwrap(),
            entry.line.unwrap(),
            entry.message,
            serde_json::to_string(&entry.structured).unwrap()
        );
    }
}
```

### `log` Compatibility

Use `tracing-log` to send `log`'s records to `tracing` ecosystem.

```toml
[dependencies]
tracing-subscriber-channel = { version = "0.1", features = ["tracing-log"]}
```

```rust
use std::sync::mpsc::channel;
use tokio::sync::broadcast::channel;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let (tx, mut rx) = channel(1);
    tracing_log::LogTracer::init().unwrap(); // handle `log`'s records
    tracing::subscriber::set_global_default(tracing_subscriber_channel::Subscriber::new(tx))?;
    log::warn!("log warning");
    if let Ok(entry) = rx.recv().await {
        println!(
            "{} [{}] [{:?}/{:?}:{:?}] {}, data: {}",
            entry.time,
            entry.level.as_str(),
            entry.module.unwrap(),
            entry.file.unwrap(),
            entry.line.unwrap(),
            entry.message,
            serde_json::to_string(&entry.structured).unwrap()
        );
    }
    Ok(())
}
```
