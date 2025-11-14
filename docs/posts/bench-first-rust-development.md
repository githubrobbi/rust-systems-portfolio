# Bench-First Rust Development: Make Performance a Feature

**How TTAPI achieves 23x speedup through systematic benchmarking and optimization**

---

## The Philosophy

**"If you don’t measure it, you don’t own it."**

Performance isn't an afterthought—it's a **feature** that requires:
- Continuous measurement (benchmarks in CI/CD)
- Regression prevention (performance gates)
- Data-driven optimization (flamegraphs, profiling)
- Clear communication (throughput, latency, memory)

---

## The TTAPI Approach

### 1. Benchmark Harness First

**Before writing production code, set up benchmarking infrastructure:**

```rust
// benches/symbol_collection.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion};
use ttapi_platform::collect_symbols;

fn bench_symbol_collection(c: &mut Criterion) {
    let rt = tokio::runtime::Runtime::new().unwrap();

    c.bench_function("collect_symbols_sp500", |b| {
        b.to_async(&rt).iter(|| async {
            let symbols = collect_symbols(black_box("sp500")).await.unwrap();
            black_box(symbols)
        });
    });
}

criterion_group!(benches, bench_symbol_collection);
criterion_main!(benches);
```

**Run benchmarks:**
```bash
cargo bench --bench symbol_collection
```

**Output:**
```
collect_symbols_sp500  time:   [52.234 s 52.456 s 52.678 s]
                       thrpt:  [9.63 symbols/s 9.67 symbols/s 9.71 symbols/s]
```

---

### 2. Real-World Performance Data

**TTAPI benchmarks from production runs:**

| Operation | Cold Run | Cached Run | Speedup |
|-----------|----------|------------|---------|
| **Total Pipeline** | 3m 2s | 7.8s | **23x** |
| **Symbol Collection** | 52s | 88ms | **590x** |
| **EOD Processing** | 2m 26s | 4s | **36x** |
| **Core Data** | 16s | 91ms | **176x** |

**Throughput benchmarks:**
- **Symbol collection**: 8,608 symbols/second
- **MetaStock import**: 438,000 rows/second
- **Parquet load**: 690 MB/second
- **Statistical analysis**: 7.3M rows/second

---

### 3. Optimization Process

#### Step 1: Measure Baseline

**Initial implementation (naive):**
```rust
async fn collect_symbols_sequential(symbols: Vec<String>) -> Result<Vec<Quote>> {
    let mut quotes = Vec::new();

    for symbol in symbols {
        let quote = fetch_quote(&symbol).await?;
        quotes.push(quote);
    }

    Ok(quotes)
}
```

**Benchmark:**
```
collect_symbols_sequential  time:   [2535 s 2540 s 2545 s]
                            thrpt:  [0.20 symbols/s]
```

**Analysis:** 507 symbols × 5s each = 2,535s (42 minutes) — **too slow!**

---

#### Step 2: Parallelize with Tokio

**Optimized implementation:**
```rust
async fn collect_symbols_parallel(symbols: Vec<String>) -> Result<Vec<Quote>> {
    let semaphore = Arc::new(Semaphore::new(100));  // Max 100 concurrent

    let tasks: Vec<_> = symbols.iter().map(|symbol| {
        let sem = semaphore.clone();
        let symbol = symbol.clone();

        tokio::spawn(async move {
            let _permit = sem.acquire().await.unwrap();
            fetch_quote(&symbol).await
        })
    }).collect();

    let results = futures::future::join_all(tasks).await;

    // Collect successful results
    results.into_iter()
        .filter_map(|r| r.ok().and_then(|q| q.ok()))
        .collect()
}
```

**Benchmark:**
```
collect_symbols_parallel  time:   [52.234 s 52.456 s 52.678 s]
                          thrpt:  [9.67 symbols/s]
```

**Improvement:** 2,535s → 52s = **48x faster!**

---

#### Step 3: Add Caching

**With TTL-based caching:**
```rust
async fn collect_symbols_cached(symbols: Vec<String>) -> Result<Vec<Quote>> {
    // Check cache first
    if let Some(cached) = CacheManager::load("symbols").await? {
        if !cached.is_expired() {
            return Ok(cached.data);
        }
    }

    // Collect fresh data
    let quotes = collect_symbols_parallel(symbols).await?;

    // Persist in background
    CacheManager::persist_in_background("symbols", &quotes, Duration::from_secs(86400));

    Ok(quotes)
}
```

**Benchmark:**
```
collect_symbols_cached  time:   [88.234 ms 88.456 ms 88.678 ms]
                        thrpt:  [5,720 symbols/s]
```

**Improvement:** 52s → 88ms = **590x faster!**

---

### 4. Flamegraph Analysis

**Generate flamegraph:**
```bash
cargo flamegraph --bench symbol_collection
```

**Findings from flamegraph:**
1. **30% time in JSON parsing** → Switch to `simd-json` (10% faster)
2. **20% time in HTTP connection setup** → Reuse connection pool (15% faster)
3. **15% time in allocations** → Use `SmallVec` for small collections (5% faster)

**Total improvement:** 30% faster after flamegraph-driven optimizations

---

### 5. Memory Profiling

**Before optimization:**
```
Peak memory: 20.4 GB (Python/Pandas equivalent)
Allocations: 2.4M allocations/second
```

**After optimization (Polars columnar format):**
```
Peak memory: 2.4 GB (8x more efficient)
Allocations: 180k allocations/second (13x fewer)
```

**Key changes:**
- Row-based → Columnar format (Polars)
- Eager evaluation → Lazy evaluation (LazyFrame)
- Intermediate buffers → Zero-copy operations

---

## Reproducibility

### Environment Recording

**Benchmark metadata:**
```toml
[benchmark.environment]
os = "macOS 14.5"
cpu = "Apple M2 Pro (8 cores)"
ram = "16 GB"
rust_version = "1.75.0"
cargo_version = "1.75.0"

[benchmark.config]
warmup_time = "3s"
measurement_time = "10s"
sample_size = 100
```

**Record in benchmark output:**
```rust
fn bench_with_metadata(c: &mut Criterion) {
    println!("Environment:");
    println!("  OS: {}", std::env::consts::OS);
    println!("  Arch: {}", std::env::consts::ARCH);
    println!("  Cores: {}", num_cpus::get());

    c.bench_function("my_benchmark", |b| {
        // benchmark code
    });
}
```

---

### Versioned Results

**Store benchmark results in repo:**
```
benchmarks/
├── 2025-11-01_baseline.json
├── 2025-11-05_parallel.json
├── 2025-11-10_cached.json
└── 2025-11-14_optimized.json
```

**Compare over time:**
```bash
cargo bench --save-baseline main
git checkout feature-branch
cargo bench --baseline main
```

---

## CI Performance Gates

### GitHub Actions Workflow

```yaml
name: Performance Regression Check

on: [pull_request]

jobs:
  benchmark:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run benchmarks (main)
        run: |
          git checkout main
          cargo bench --bench symbol_collection -- --save-baseline main

      - name: Run benchmarks (PR)
        run: |
          git checkout ${{ github.head_ref }}
          cargo bench --bench symbol_collection -- --baseline main

      - name: Check for regressions
        run: |
          # Fail if p95 latency increased by >10%
          python scripts/check_regression.py --threshold 0.10
```

**Regression detection:**
```python
# scripts/check_regression.py
import json
import sys

def check_regression(baseline, current, threshold=0.10):
    baseline_p95 = baseline['p95']
    current_p95 = current['p95']

    regression = (current_p95 - baseline_p95) / baseline_p95

    if regression > threshold:
        print(f"❌ Performance regression: {regression*100:.1f}%")
        print(f"   Baseline p95: {baseline_p95:.2f}ms")
        print(f"   Current p95:  {current_p95:.2f}ms")
        sys.exit(1)
    else:
        print(f"✅ No regression: {regression*100:.1f}%")
        sys.exit(0)
```

---

## Communication: Benchmark Reports

### Internal Report (Engineering Team)

```markdown
## Performance Benchmark Report — Symbol Collection Optimization

**Date:** 2025-11-14
**Branch:** feature/parallel-collection
**Baseline:** main (commit abc123)

### Summary
- ✅ 48x faster than baseline (2,535s → 52s)
- ✅ 8x more memory efficient (20.4 GB → 2.4 GB)
- ✅ No regressions in other benchmarks

### Detailed Results

| Metric | Baseline | Optimized | Change |
|--------|----------|-----------|--------|
| **Latency (p50)** | 2,535s | 52s | **-98%** |
| **Latency (p95)** | 2,580s | 54s | **-98%** |
| **Throughput** | 0.20 symbols/s | 9.67 symbols/s | **+4,735%** |
| **Memory (peak)** | 20.4 GB | 2.4 GB | **-88%** |
| **Allocations** | 2.4M/s | 180k/s | **-93%** |

### Flamegraph Analysis
- 30% time in JSON parsing → Optimized with simd-json
- 20% time in HTTP setup → Connection pool reuse
- 15% time in allocations → SmallVec for small collections

### Next Steps
- [ ] Add caching layer (estimated 590x speedup)
- [ ] Optimize Parquet I/O (estimated 2x speedup)
- [ ] Profile dxFeed WebSocket (unknown potential)
```

---

### External Report (Portfolio/Resume)

```markdown
## TTAPI Performance Highlights

**23x faster** end-to-end pipeline with intelligent caching
- Cold run: 3m 2s
- Cached run: 7.8s

**438,000 rows/second** data import throughput
- 27.6M rows imported in 63 seconds
- Parallel CSV parsing with rayon

**8x more memory efficient** than Python/Pandas
- 2.4 GB vs. 20+ GB for equivalent workload
- Polars columnar format with zero-copy operations
```

---

## Tools & Techniques

### Benchmarking Tools

1. **Criterion** — Statistical benchmarking with regression detection
2. **Divan** — Fast benchmarking with minimal overhead
3. **cargo-flamegraph** — CPU profiling with flamegraphs
4. **perf** — Linux performance analysis
5. **Instruments** — macOS profiling (Time Profiler, Allocations)

### Profiling Commands

```bash
# CPU profiling (flamegraph)
cargo flamegraph --bench symbol_collection

# Memory profiling (valgrind)
valgrind --tool=massif target/release/ttapi

# System calls (strace)
strace -c target/release/ttapi

# Tokio console (async profiling)
RUSTFLAGS="--cfg tokio_unstable" cargo run --features tokio-console
```

---

## Results in TTAPI

**Performance as a feature:**
- ✅ 23x faster end-to-end pipeline
- ✅ 438,000 rows/second throughput
- ✅ 8x more memory efficient
- ✅ Zero performance regressions in 3+ months

**Development velocity:**
- ✅ Benchmarks catch regressions in CI
- ✅ Flamegraphs guide optimization efforts
- ✅ Clear metrics for stakeholder communication
- ✅ Confidence in production deployments

---

## Summary

**Bench-first development requires:**

1. **Harness first** — Set up benchmarking before optimization
2. **Measure baseline** — Know where you're starting from
3. **Optimize systematically** — Parallelize, cache, profile
4. **Prevent regressions** — CI gates with threshold checks
5. **Communicate clearly** — Throughput, latency, memory

**The result:** Performance is a measurable, improvable feature—not a hope.

---

## Additional Resources

- [Performance Benchmarks](../performance-benchmarks.md) — Detailed TTAPI performance data
- [Architecture Deep Dive](../architecture-deep-dive.md) — How architecture enables performance
- [Why Rust?](../why-rust.md) — Performance comparison with Python/C++/Go

---

**Copyright © 2025 SKY, LLC. All rights reserved.**