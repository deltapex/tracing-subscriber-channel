# tracing-subscribe-sqlite

A tracing Subscriber to send log to tokio broadcast channel.

## Usage

```toml
[dependencies]
tracing-subscriber-channel = "0.1"
tokio = { version = "1", features = ["full"]}
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
    tokio::spawn(async move {
        tracing::info!("in thread");
    });
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
Not support
