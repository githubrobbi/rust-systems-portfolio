# TTAPI — Architecture One-Pager

**High-performance financial data platform built with Rust**

**Author:** Robert Nio  •  **Updated:** 2025-11-14

---

## What It Is

TTAPI is a **production-grade Rust application** for financial data collection, processing, and analysis. It demonstrates world-class systems programming with:

- **23x faster** end-to-end pipeline with intelligent caching
- **438,000 rows/second** data import throughput
- **8x more memory efficient** than Python/Pandas equivalents
- **100 concurrent requests** with semaphore-based backpressure
- **Zero-copy data processing** with Polars' columnar format

---

## Architecture

### Workspace Structure (Polars-Inspired)

**5 domain-separated crates:**

1. **`ttapi-core`** — Core types, errors, configuration, TTL utilities
2. **`ttapi-client`** — HTTP client, OAuth2, circuit breakers, API wrappers
3. **`ttapi-platform`** — Business logic, data processing, caching
4. **`ttapi-polars`** — Polars extensions and LazyFrame utilities
5. **`ttapi-onboarding`** — CLI interface with console grid UI

**Layering:** `onboarding` → `platform` → `client` → `core`

**Benefits:**
- Parallel compilation (faster incremental builds)
- Dependency isolation (CLI changes don't recompile platform)
- Reusable components (platform can be used as a library)

---

## Performance Highlights

### Real-World Benchmarks

| Operation | Cold Run | Cached Run | Speedup |
|-----------|----------|------------|---------|
| **Total Pipeline** | 3m 2s | 7.8s | **23x** |
| **Symbol Data** (507 symbols) | 52s | 88ms | **590x** |
| **EOD Processing** (29M rows) | 2m 26s | 4s | **36x** |
| **Core Data** | 16s | 91ms | **176x** |

### Throughput Analysis

- **Symbol collection**: 8,608 symbols/second
- **MetaStock import**: 438,000 rows/second
- **Parquet load**: 690 MB/second
- **Statistical analysis**: 7.3M rows/second

### Resource Utilization

- **Peak memory**: 2.4 GB (8x more efficient than Python/Pandas)
- **CPU utilization**: 800% (8 cores fully utilized)
- **Concurrent requests**: 100 (vs. 10-20 for Python)

---

## Technical Excellence

### Async/Await Concurrency (Tokio)

- **Work-stealing scheduler**: M:N threading (M tasks on N OS threads)
- **Semaphore-based backpressure**: Max 100 concurrent requests
- **Structured task management**: Graceful shutdown with cancellation tokens
- **WebSocket streaming**: dxFeed quote collection with automatic reconnection

### Zero-Copy Data Processing (Polars)

- **Columnar format**: 10x better memory efficiency than row-based
- **LazyFrame query optimization**: Build query plan, optimize, execute once
- **Memory-mapped Parquet**: Direct disk-to-memory mapping (no intermediate buffers)
- **SIMD operations**: Vectorized statistical analysis

### Intelligent Caching (TTL-Based)

- **Automatic invalidation**: No manual cache management
- **Dependency tracking**: Downstream data auto-refreshes when upstream changes
- **Dual-format persistence**: Parquet (analytics) + CSV (debugging)
- **Background I/O**: Compute continues while writing to disk

### Circuit Breakers & Resilience

- **Per-endpoint tracking**: TastyTrade and dxFeed tracked separately
- **Exponential backoff with jitter**: 100ms → 400ms → 800ms (randomized)
- **Graceful degradation**: 20% missing data? Keep processing!
- **HTTP/2 GOAWAY handling**: Automatic connection pool refresh

---

## Code Quality

### Standards (Rust Excellence Playbook 2025)

✅ **≥90% test coverage** (llvm-cov with HTML reports)
✅ **Zero clippy::pedantic warnings** (strict linting)
✅ **File headers** (copyright, license, description)
✅ **Panic-free public APIs** (all errors are `Result<T, E>`)
✅ **Benchmark suites** (criterion/divan with flamegraphs)
✅ **Structured logging** (tracing with request IDs)

### Error Handling

- **Type-safe errors**: `thiserror` with `TTError` hierarchy
- **Explicit propagation**: `?` operator makes error flow visible
- **No null pointers**: `Option<T>` forces handling of missing values
- **Compile-time safety**: All errors caught before runtime

### Testing & Validation

- **Unit tests**: `cargo nextest` with parallel execution
- **Integration tests**: Real API calls with circuit breaker validation
- **Property tests**: `proptest` for edge case discovery
- **Golden files**: Snapshot testing for data transformations

---

## Data Pipeline

### Sources

1. **TastyTrade REST API** — Market data, symbols, accounts, option chains
2. **dxFeed WebSocket** — Real-time streaming quotes
3. **MetaStock CSV** — Historical EOD data (27.6M rows)

### Processing

1. **Parallel collection**: 100 concurrent API requests
2. **Lazy evaluation**: Polars builds query plan, optimizes, executes once
3. **Deduplication**: Remove 130k duplicate rows across sources
4. **Statistical analysis**: Volatility, returns, confidence intervals

### Persistence

1. **Parquet format**: Columnar storage for analytics (2.69 GB)
2. **CSV format**: Human-readable debugging (dual-format)
3. **Background I/O**: Dedicated thread pool (compute doesn't block)
4. **Atomic writes**: Temp file → rename (crash-safe)

---

## Observability

### Console Grid UI

- **Rigid grid layout**: Core Data, Symbol Data, EOD Data, Math
- **In-place progress updates**: Overwrite same line (no scroll spam)
- **Percentage completion**: Real-time progress (e.g., "2800 of 6890 (40.6%)")
- **Cache indicators**: "SKIPPED (cached)" vs. "START/DONE"

### Structured Logging

- **Tracing framework**: Terminal (error level) + file (trace level)
- **Request IDs**: Track operations across async boundaries
- **Performance metrics**: Timing, throughput, memory usage
- **Error context**: Full stack traces with source locations

---

## Demo Videos

🎥 [**Watch Demo Videos**](demo/ttapi-demo-script.md) — See cold run (3m 2s) vs. cached run (7.8s)

**Video 1: Initial Run** (8.4 MB)
- Full data collection pipeline from scratch
- 29.2M rows imported at 438k rows/second
- 507 S&P 500 symbols with option chains

**Video 2: Cached Run** (3.1 MB)
- Same pipeline with TTL-based cache hits
- 23x faster (7.8s vs. 3m 2s)
- 100% cache hit rate

---

## Additional Documentation

📊 [**Performance Benchmarks**](performance-benchmarks.md) — Detailed metrics and throughput analysis
🏗️ [**Rust Excellence**](rust-excellence.md) — Architecture deep-dive with code examples
🔍 [**Architecture Deep Dive**](architecture-deep-dive.md) — System diagrams and data flow
⚖️ [**Why Rust?**](why-rust.md) — Technology comparison (Rust vs. Python/C++/Go)

---

## Key Differentiators

### vs. Python/Pandas
- **40x faster** end-to-end pipeline
- **8x more memory efficient**
- **Compile-time type safety** (zero runtime errors)
- **Fearless concurrency** (no GIL, no data races)

### vs. C++
- **Memory safety without GC** (ownership system)
- **No data races** (borrow checker guarantees)
- **Modern tooling** (Cargo handles builds, deps, tests, benchmarks)

### vs. Go
- **No GC pauses** (critical for low-latency systems)
- **Zero-copy data processing** (ownership enables memory-mapped files)
- **Stronger type safety** (no nil pointers)

---

## Contact

**Robert Nio**
Rust Systems Engineer
[GitHub](https://github.com/githubrobbi) | [LinkedIn](https://linkedin.com/in/robertnio)

---

**Note:** TTAPI source code is private. This documentation showcases architecture and performance with public-safe information.

**Copyright © 2025 SKY, LLC. All rights reserved.**