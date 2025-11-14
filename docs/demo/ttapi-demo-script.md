# TTAPI — Demo Videos

**Real-world performance demonstrations showing cold run vs. cached run.**

---

## 🎥 Demo Video 1: Initial Run (Cold Cache)

**Watch the full data collection pipeline from scratch:**

<video width="100%" controls>
  <source src="TTAPI 1st run 2025-11-13.mov" type="video/quicktime">
  <source src="TTAPI 1st run 2025-11-13.mov" type="video/mp4">
  Your browser does not support the video tag. <a href="TTAPI 1st run 2025-11-13.mov">Download the video</a>.
</video>

[**⬇️ Download Video**](TTAPI 1st run 2025-11-13.mov) (8.4 MB)

**What you'll see:**
- **Core Data Collection**: 22,348 symbols, 76,109 trading days, 110,080 calendar dates
- **Symbol Data Collection**: 507 S&P 500 symbols with market metrics, dividends, earnings, option chains
- **EOD Data Processing**: 29.2M rows imported from MetaStock CSV + dxFeed complement
- **Statistical Analysis**: Volatility calculations across all symbols

**Performance highlights:**
- Total pipeline time: **3m 2s**
- MetaStock import: **438,000 rows/second**
- Option chains: **251,377 data points** collected
- Memory usage: **2.4 GB peak**

---

## 🎥 Demo Video 2: Cached Run (TTL-Based Cache Hits)

**Watch the same pipeline with intelligent caching:**

<video width="100%" controls>
  <source src="TTAPI 2nd run 2025-11-13.mov" type="video/quicktime">
  <source src="TTAPI 2nd run 2025-11-13.mov" type="video/mp4">
  Your browser does not support the video tag. <a href="TTAPI 2nd run 2025-11-13.mov">Download the video</a>.
</video>

[**⬇️ Download Video**](TTAPI 2nd run 2025-11-13.mov) (3.1 MB)

**What you'll see:**
- **Instant cache hits**: All data loaded from Parquet files
- **Sub-millisecond operations**: Most operations complete in 16ms
- **Automatic TTL validation**: No manual cache management
- **Same results, 23x faster**: Complete pipeline in 7.8 seconds

**Performance highlights:**
- Total pipeline time: **7.8s** (23x faster than cold run)
- Symbol data: **88ms** (590x faster)
- EOD data load: **3.978s** (36x faster)
- Cache hit rate: **100%** (all data fresh within TTL)

---

## 📊 Side-by-Side Comparison

| Operation | Cold Run | Cached Run | Speedup |
|-----------|----------|------------|---------|
| **Total Pipeline** | 3m 2s | 7.8s | **23x** |
| **Core Data** | 16s | 91ms | **176x** |
| **Symbol Data** | 52s | 88ms | **590x** |
| **EOD Processing** | 2m 26s | 4s | **36x** |

---

## 🎯 Key Takeaways

### Architecture Highlights
- **Workspace structure**: 5 domain-separated crates (Polars-inspired)
- **Async/await concurrency**: Tokio work-stealing scheduler with 100 concurrent requests
- **Zero-copy data processing**: Polars columnar format with LazyFrame optimization
- **TTL-based caching**: Automatic invalidation with dependency tracking
- **Circuit breakers**: Per-endpoint fault tolerance with exponential backoff

### Performance Highlights
- **23x faster** with intelligent caching
- **438,000 rows/second** import throughput
- **8x more memory efficient** than Python/Pandas
- **100 concurrent requests** (vs. 10-20 for Python)
- **85% cache hit rate** on subsequent runs

### Code Quality
- **≥90% test coverage** (llvm-cov)
- **Zero clippy::pedantic warnings**
- **Panic-free public APIs** (all errors are `Result<T, E>`)
- **Benchmark suites** (criterion/divan)

---

## 📝 Additional Resources

- [Performance Benchmarks](../performance-benchmarks.md) — Detailed metrics and throughput analysis
- [Rust Excellence](../rust-excellence.md) — Architecture deep-dive with code examples
- [Architecture Deep Dive](../architecture-deep-dive.md) — System diagrams and data flow
- [Why Rust?](../why-rust.md) — Technology comparison (Rust vs. Python/C++/Go)

---

**Note:** TTAPI source code is private. These videos demonstrate real production performance with actual data.