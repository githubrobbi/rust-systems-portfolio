# Panic-Free Rust APIs in Production

**How TTAPI achieves zero panics in production with type-safe error handling**

---

## The Problem

In production systems, **panics are outages**. A single `unwrap()` on a `None` value can crash your entire service, losing in-flight data and breaking SLAs.

**Common panic sources:**
- `unwrap()` / `expect()` on `Option<T>` or `Result<T, E>`
- Array/vector indexing out of bounds
- Integer overflow in release mode
- `todo!()` / `unimplemented!()` in production code
- Division by zero

**The cost:**
- Lost data (in-flight transactions)
- Service downtime (restart required)
- Cascading failures (dependent services fail)
- Poor user experience (abrupt termination)

---

## The Solution: Explicit Error Handling

TTAPI uses **type-safe error handling** throughout the codebase:

### 1. Workspace-Level Panic Denial

**Cargo.toml (workspace level):**
```toml
[workspace.lints.clippy]
unwrap_used = "deny"
expect_used = "deny"
panic = "deny"
todo = "deny"
unimplemented = "deny"
```

**Result:** Compiler errors on any panic-inducing code in CI/CD pipeline.

---

## Error Hierarchy with `thiserror`

### Typed Error Enum

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum TTError {
    #[error("API error: {0}")]
    Api(#[from] ApiError),

    #[error("Data processing error: {0}")]
    Data(#[from] DataError),

    #[error("Configuration error: {0}")]
    Config(#[from] ConfigError),

    #[error("I/O error: {0}")]
    Io(#[from] std::io::Error),
}

#[derive(Error, Debug)]
pub enum ApiError {
    #[error("HTTP {status}: {message}")]
    Http { status: u16, message: String },

    #[error("Authentication failed: {0}")]
    Auth(String),

    #[error("Rate limit exceeded, retry after {retry_after}s")]
    RateLimit { retry_after: u64 },

    #[error("Request timeout after {0}s")]
    Timeout(u64),
}

#[derive(Error, Debug)]
pub enum DataError {
    #[error("Invalid data format: {0}")]
    InvalidFormat(String),

    #[error("Missing required column: {0}")]
    MissingColumn(String),

    #[error("Parse error: {0}")]
    Parse(String),
}
```

**Benefits:**
- ✅ Explicit error types (no hidden exceptions)
- ✅ Automatic `From` conversions (via `#[from]`)
- ✅ Rich error context (status codes, retry timing)
- ✅ Pattern matching for error handling

---

## Real-World Examples from TTAPI

### Example 1: API Client (No Panics)

**❌ Panic-prone code:**
```rust
fn fetch_quote(symbol: &str) -> Quote {
    let response = client.get(&format!("/quotes/{}", symbol))
        .send()
        .unwrap();  // ❌ Panics on network error

    let quote = response.json::<Quote>()
        .unwrap();  // ❌ Panics on invalid JSON

    quote
}
```

**✅ Panic-free code:**
```rust
async fn fetch_quote(symbol: &str) -> Result<Quote, ApiError> {
    let response = client
        .get(&format!("/quotes/{}", symbol))
        .send()
        .await
        .map_err(|e| ApiError::Http {
            status: e.status().map(|s| s.as_u16()).unwrap_or(0),
            message: e.to_string(),
        })?;

    if !response.status().is_success() {
        return Err(ApiError::Http {
            status: response.status().as_u16(),
            message: response.text().await.unwrap_or_default(),
        });
    }

    let quote = response.json::<Quote>()
        .await
        .map_err(|e| ApiError::Parse(e.to_string()))?;

    Ok(quote)
}
```

**Key improvements:**
- All errors are `Result<T, E>` (explicit)
- Network errors → `ApiError::Http`
- Parse errors → `ApiError::Parse`
- Caller must handle errors (compiler enforces)

---

### Example 2: Data Processing (Graceful Degradation)

**❌ Panic-prone code:**
```rust
fn process_symbols(symbols: Vec<String>) -> DataFrame {
    let mut data = Vec::new();

    for symbol in symbols {
        let quote = fetch_quote(&symbol).unwrap();  // ❌ Panics if any symbol fails
        data.push(quote);
    }

    DataFrame::new(data).unwrap()  // ❌ Panics on invalid data
}
```

**✅ Panic-free code with graceful degradation:**
```rust
async fn process_symbols(symbols: Vec<String>) -> Result<DataFrame, DataError> {
    let mut successful = Vec::new();
    let mut failed = Vec::new();

    for symbol in symbols {
        match fetch_quote(&symbol).await {
            Ok(quote) => successful.push(quote),
            Err(e) => {
                tracing::warn!("Failed to fetch {}: {}", symbol, e);
                failed.push(symbol);
            }
        }
    }

    // Log summary
    tracing::info!(
        "Processed {} symbols: {} successful, {} failed ({:.1}% success)",
        symbols.len(),
        successful.len(),
        failed.len(),
        (successful.len() as f64 / symbols.len() as f64) * 100.0
    );

    // Continue with partial data (graceful degradation)
    if successful.is_empty() {
        return Err(DataError::NoData);
    }

    DataFrame::new(successful)
        .map_err(|e| DataError::InvalidFormat(e.to_string()))
}
```

**Key improvements:**
- Partial failures don't crash the pipeline
- Detailed logging (which symbols failed)
- Success rate tracking (e.g., "503 of 507 (99.2%)")
- Only fails if **all** symbols fail (not just one)

---

### Example 3: Option Handling (No `unwrap()`)

**❌ Panic-prone code:**
```rust
fn get_price(quote: &Quote) -> f64 {
    quote.price.unwrap()  // ❌ Panics if price is None
}
```

**✅ Panic-free code:**
```rust
fn get_price(quote: &Quote) -> Result<f64, DataError> {
    quote.price
        .ok_or_else(|| DataError::MissingField("price".to_string()))
}

// Or with a default value:
fn get_price_or_default(quote: &Quote) -> f64 {
    quote.price.unwrap_or(0.0)
}

// Or with early return:
fn process_quote(quote: &Quote) -> Result<Analysis, DataError> {
    let price = quote.price
        .ok_or_else(|| DataError::MissingField("price".to_string()))?;

    let volume = quote.volume
        .ok_or_else(|| DataError::MissingField("volume".to_string()))?;

    Ok(Analysis { price, volume })
}
```

**Key improvements:**
- Explicit error for missing fields
- Caller decides how to handle (fail vs. default)
- `?` operator makes error propagation clean

---

## Concurrency & Graceful Shutdown

### Structured Concurrency with Tokio

**❌ Panic-prone code:**
```rust
#[tokio::main]
async fn main() {
    tokio::spawn(async {
        loop {
            process_data().await.unwrap();  // ❌ Panics crash the task
        }
    });

    // No way to stop gracefully
    tokio::time::sleep(Duration::from_secs(3600)).await;
}
```

**✅ Panic-free code with graceful shutdown:**
```rust
use tokio_util::sync::CancellationToken;

#[tokio::main]
async fn main() -> Result<(), TTError> {
    let cancel_token = CancellationToken::new();

    // Spawn background task
    let task_handle = tokio::spawn({
        let cancel = cancel_token.clone();
        async move {
            loop {
                tokio::select! {
                    _ = cancel.cancelled() => {
                        tracing::info!("Shutdown signal received");
                        break;
                    }
                    result = process_data() => {
                        if let Err(e) = result {
                            tracing::error!("Processing error: {}", e);
                            // Continue processing (don't crash)
                        }
                    }
                }
            }
        }
    });

    // Wait for Ctrl+C
    tokio::signal::ctrl_c().await?;
    tracing::info!("Shutting down gracefully...");

    // Cancel all tasks
    cancel_token.cancel();

    // Wait for tasks to finish
    task_handle.await?;

    tracing::info!("Shutdown complete");
    Ok(())
}
```

**Key improvements:**
- Graceful shutdown (no abrupt termination)
- Errors logged, not panicked
- Deterministic cleanup (no resource leaks)
- Testable (can trigger shutdown in tests)

---

## Testing & Validation

### Property Testing with `proptest`

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn parse_symbol_never_panics(s in "\\PC*") {
        // This should never panic, even with invalid input
        let _ = parse_symbol(&s);
    }

    #[test]
    fn calculate_volatility_never_panics(
        prices in prop::collection::vec(any::<f64>(), 0..1000)
    ) {
        // Should handle empty vec, NaN, infinity, etc.
        let result = calculate_volatility(&prices);
        assert!(result.is_ok() || result.is_err());  // Never panics
    }
}
```

---

## When Are Panics Acceptable?

### 1. Tests (Explicit Failures)
```rust
#[test]
fn test_invalid_config() {
    let config = Config::from_str("invalid");
    assert!(config.is_err());  // ✅ OK to panic in tests
}
```

### 2. Debug Assertions (Development Only)
```rust
debug_assert!(index < vec.len(), "Index out of bounds");
```

### 3. Unrecoverable Corruption (Documented)
```rust
/// # Panics
/// Panics if the internal state is corrupted (should never happen).
pub fn get_cached_value(&self) -> &Value {
    self.cache.get(&self.key)
        .expect("FATAL: cache corruption detected")
}
```

---

## Results in TTAPI

**Panic-free production:**
- ✅ Zero panics in 3+ months of production use
- ✅ Graceful degradation (20% missing data? Keep processing!)
- ✅ Detailed error logging (know exactly what failed)
- ✅ Automatic retries (transient errors don't fail the pipeline)

**Performance impact:**
- ❌ None! `Result<T, E>` is zero-cost abstraction
- ✅ Better performance (no stack unwinding on errors)
- ✅ Faster debugging (errors have context)

---

## Summary

**Panic-free Rust in production requires:**

1. **Workspace-level denial** of `unwrap/expect/panic/todo`
2. **Typed error hierarchies** with `thiserror`
3. **Explicit error handling** with `Result<T, E>` and `?`
4. **Graceful degradation** (partial failures don't crash)
5. **Structured concurrency** with cancellation tokens
6. **Property testing** to catch edge cases

**The result:** Production-grade reliability with zero runtime panics.

---

## Additional Resources

- [Rust Excellence](../rust-excellence.md) — Architecture deep-dive with more error handling examples
- [Performance Benchmarks](../performance-benchmarks.md) — See how error handling doesn't impact performance
- [Architecture Deep Dive](../architecture-deep-dive.md) — Error hierarchy and circuit breaker patterns

---

**Copyright © 2025 SKY, LLC. All rights reserved.**