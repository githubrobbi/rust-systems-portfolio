# Architecture Deep Dive — TTAPI

**A comprehensive look at the design decisions, patterns, and trade-offs in a world-class Rust system.**

---

## System Overview

TTAPI is a **high-performance financial data platform** built with Rust, designed to:
- Collect data from multiple sources (TastyTrade API, dxFeed WebSocket, MetaStock CSV)
- Process millions of rows with zero-copy operations (Polars LazyFrames)
- Persist data in dual formats (Parquet for analytics, CSV for debugging)
- Provide real-time streaming quotes via WebSocket
- Cache intelligently with TTL-based invalidation

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         TTAPI Application                            │
│                                                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   CLI/UI     │  │   Streaming  │  │   Background │              │
│  │  (Console    │  │   Service    │  │   Tasks      │              │
│  │   Grid)      │  │  (WebSocket) │  │ (Persistence)│              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │
│         │                  │                  │                       │
│         └──────────────────┼──────────────────┘                       │
│                            │                                          │
│  ┌─────────────────────────▼──────────────────────────────────────┐ │
│  │              ttapi-platform (Business Logic)                    │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │ │
│  │  │ Data         │  │ Processing   │  │ Cache        │         │ │
│  │  │ Collection   │  │ Pipeline     │  │ Manager      │         │ │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │ │
│  └─────────┼──────────────────┼──────────────────┼─────────────────┘ │
│            │                  │                  │                   │
│  ┌─────────▼──────────────────▼──────────────────▼─────────────────┐ │
│  │              ttapi-polars (Data Processing)                      │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │ │
│  │  │ LazyFrame    │  │ DataFrame    │  │ Parquet I/O  │         │ │
│  │  │ Utils        │  │ Utils        │  │              │         │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘         │ │
│  └──────────────────────────────────────────────────────────────────┘ │
│                                                                       │
│  ┌──────────────────────────────────────────────────────────────────┐ │
│  │              ttapi-client (HTTP & WebSocket)                     │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │ │
│  │  │ Dynamic      │  │ OAuth2       │  │ Circuit      │         │ │
│  │  │ Client       │  │ Manager      │  │ Breaker      │         │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘         │ │
│  └──────────────────────────────────────────────────────────────────┘ │
│                                                                       │
│  ┌──────────────────────────────────────────────────────────────────┐ │
│  │              ttapi-core (Types & Utilities)                      │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │ │
│  │  │ Error Types  │  │ Config       │  │ TTL Utils    │         │ │
│  │  │ (TTError)    │  │ (RMS)        │  │              │         │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘         │ │
│  └──────────────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────────────┘

External APIs:
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ TastyTrade   │  │ dxFeed       │  │ MetaStock    │
│ REST API     │  │ WebSocket    │  │ CSV Files    │
└──────────────┘  └──────────────┘  └──────────────┘
```

---

## Crate Responsibilities

### ttapi-core
**Purpose:** Core types, utilities, and configuration  
**Dependencies:** Minimal (std, serde, chrono)  
**Exports:**
- `TTError` — Unified error type hierarchy
- `Config` — Application configuration (RMS-based)
- `TTL` — Time-to-live utilities for cache invalidation
- `ResourceStore` — Trait for resource management

**Why separate?**
- Reusable in other projects without pulling in HTTP client
- Fast compilation (minimal dependencies)
- Clear ownership of core types

---

### ttapi-client
**Purpose:** HTTP client, authentication, API wrappers  
**Dependencies:** reqwest, tokio, oauth2  
**Exports:**
- `DynamicClient` — HTTP client with circuit breakers
- `OAuth2Manager` — Token management with automatic refresh
- `ApiError` — HTTP-specific error types
- `CircuitBreaker` — Per-endpoint fault tolerance

**Why separate?**
- Isolates network I/O from business logic
- Easy to mock for testing (trait-based design)
- Circuit breaker state is per-client, not global

---

### ttapi-platform
**Purpose:** Business logic, data processing, caching  
**Dependencies:** ttapi-core, ttapi-client, ttapi-polars  
**Exports:**
- `collect_symbols()` — Parallel symbol collection
- `import_eod_data()` — Hybrid MetaStock + dxFeed import
- `CacheManager` — TTL-based cache with background persistence
- `StreamingService` — WebSocket quote accumulation

**Why separate?**
- Business logic is independent of CLI/UI
- Can be used as a library (e.g., in a web server)
- Clear separation from data processing (ttapi-polars)

---

### ttapi-polars
**Purpose:** Polars extensions, LazyFrame utilities  
**Dependencies:** polars, ttapi-core  
**Exports:**
- `collect_blocking()` — Async wrapper for Polars collect
- `save_parquet()` — Async Parquet persistence
- `load_parquet()` — Async Parquet loading
- `dataframe_utils` — Common DataFrame operations

**Why separate?**
- Polars is a heavy dependency (long compile times)
- Changes to Polars code don't recompile HTTP client
- Reusable Polars utilities for other projects

---

### ttapi-onboarding (CLI)
**Purpose:** User-facing interface, console grid  
**Dependencies:** All other crates  
**Exports:**
- `ConsoleGrid` — Rigid grid layout with in-place updates
- `ProgressManager` — Progress tracking for long operations
- CLI argument parsing

**Why separate?**
- UI changes don't recompile business logic
- Easy to swap CLI for web UI or API server
- Clear entry point for the application

---

## Data Flow

### 1. Symbol Data Collection

```
User runs `tt`
    │
    ▼
CLI parses args (TT_LIST=2 → S&P 500)
    │
    ▼
Platform spawns parallel tasks (507 symbols)
    │
    ├─► Task 1: fetch_market_metrics(symbol_1)
    ├─► Task 2: fetch_market_metrics(symbol_2)
    ├─► ...
    └─► Task 507: fetch_market_metrics(symbol_507)
    │
    ▼
Client makes HTTP requests (max 100 concurrent)
    │
    ├─► Circuit breaker checks state (Closed/Open/Half-Open)
    ├─► Retry with exponential backoff (3 attempts)
    └─► Return Result<Data, ApiError>
    │
    ▼
Platform collects results (graceful degradation)
    │
    ├─► 503 successful (0.7% missing)
    └─► 4 failed (logged, not fatal)
    │
    ▼
Polars builds DataFrame (columnar format)
    │
    ▼
CacheManager persists in background (Parquet + CSV)
    │
    ▼
Console grid updates (in-place progress)
```

---

### 2. EOD Data Processing

```
User runs `tt` (EOD grid)
    │
    ▼
Platform checks TTL (is cache fresh?)
    │
    ├─► YES: Load from Parquet (3.978s)
    │        └─► Return DataFrame
    │
    └─► NO: Run hybrid import
         │
         ├─► Scan MetaStock CSV files (15,798 found)
         │
         ├─► Import MetaStock data (27.6M rows, 63s)
         │   └─► Parallel CSV parsing (rayon)
         │
         ├─► Complement with dxFeed (1.7M rows, 63s)
         │   └─► Parallel API calls (tokio)
         │
         ├─► Cleanup & deduplication (130k rows removed)
         │   └─► Polars LazyFrame operations
         │
         ├─► Persist snapshot (Parquet, 16.7s)
         │   └─► Background thread pool
         │
         └─► Return DataFrame
```

---

## Concurrency Model

### Tokio Work-Stealing Scheduler

```
OS Threads (N=8):
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│Thread 1│ │Thread 2│ │Thread 3│ │Thread 4│ ...
└───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘
    │          │          │          │
    ▼          ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│ Queue  │ │ Queue  │ │ Queue  │ │ Queue  │
│ [T1]   │ │ [T2]   │ │ [T3]   │ │ [T4]   │
│ [T5]   │ │ [T6]   │ │ [T7]   │ │ [T8]   │
│ [T9]   │ │ [T10]  │ │ [T11]  │ │ [T12]  │
└────────┘ └────────┘ └────────┘ └────────┘

Tasks (M=507):
- Each symbol = 1 task
- Tasks distributed across queues
- Idle threads steal from busy threads
- Result: 800% CPU utilization (8 cores)
```

### Semaphore-Based Backpressure

```
Semaphore (max 100 permits):
┌────────────────────────────────────────┐
│ Available permits: 100                 │
└────────────────────────────────────────┘
         │
         ▼
Task 1 acquires permit (99 available)
Task 2 acquires permit (98 available)
...
Task 100 acquires permit (0 available)
Task 101 WAITS (no permits available)
         │
         ▼
Task 1 completes, releases permit (1 available)
         │
         ▼
Task 101 acquires permit (0 available)
```

**Why 100?**
- API rate limits: ~100 requests/second
- Network bandwidth: ~100 concurrent connections
- Memory: ~100 MB per task (10 GB total)

---

## Caching Strategy

### TTL-Based Invalidation

```
Cache Entry:
┌────────────────────────────────────────┐
│ Data Type: symbols                     │
│ Path: /path/to/symbols.parquet         │
│ TTL: 24 hours                          │
│ Modified: 2025-11-13 21:41:18          │
│ Size: 2.3 MB                           │
└────────────────────────────────────────┘

Cache Check (2025-11-14 10:00:00):
    │
    ▼
Age = now - modified = 12h 18m 42s
    │
    ▼
Is age < TTL? (12h < 24h)
    │
    ├─► YES: Load from cache (12.675ms)
    │
    └─► NO: Fetch fresh data (2.596s)
         └─► Persist to cache (background)
```

### Dependency Tracking

```
Unified Calendar (TTL: 7 days)
    │
    ├─► Modified: 2025-11-13 21:41:22
    │
    ▼
Expiration Series (depends on Unified Calendar)
    │
    ├─► Check: Has Unified Calendar changed?
    │   └─► YES: Rebuild Expiration Series
    │   └─► NO: Use cached Expiration Series
    │
    ▼
Returns StdDev (depends on Expiration Series)
    │
    ├─► Check: Has Expiration Series changed?
    │   └─► YES: Recompute Returns StdDev
    │   └─► NO: Use cached Returns StdDev
```

---

## Error Handling

### Error Hierarchy

```
TTError (top-level)
├─► ApiError (HTTP/network)
│   ├─► Http { status, message }
│   ├─► Auth(String)
│   ├─► RateLimit { retry_after }
│   └─► Timeout
│
├─► DataError (processing)
│   ├─► InvalidFormat(String)
│   ├─► MissingColumn(String)
│   └─► ParseError(String)
│
├─► ConfigError (configuration)
│   ├─► MissingEnvVar(String)
│   └─► InvalidValue(String)
│
└─► IoError (file I/O)
    └─► std::io::Error
```

### Circuit Breaker State Machine

```
┌─────────┐
│ Closed  │ ◄─────────────────────────┐
│ (Normal)│                            │
└────┬────┘                            │
     │                                 │
     │ N failures                      │ M successes
     ▼                                 │
┌─────────┐                       ┌────┴────┐
│  Open   │ ──────────────────►   │ Half-   │
│ (Fail   │   Timeout expired     │ Open    │
│  Fast)  │                       │ (Test)  │
└─────────┘                       └─────────┘
```

---

## Performance Optimizations

### 1. Lazy Evaluation (Polars)
- Build query plan (no execution)
- Optimize query plan (eliminate redundant operations)
- Execute once (multi-threaded)

### 2. Memory-Mapped Parquet
- Direct disk-to-memory mapping
- No intermediate buffers
- Zero-copy reads

### 3. Background Persistence
- Dedicated thread pool for I/O
- Compute continues while writing
- Atomic writes (temp file → rename)

### 4. Columnar Format
- Better cache locality (CPU cache hits)
- Efficient compression (similar values)
- SIMD operations (vectorized)

---

## Conclusion

TTAPI's architecture demonstrates:
- **Workspace structure** for compile-time optimization
- **Async/await concurrency** for maximum throughput
- **Zero-copy data processing** for memory efficiency
- **TTL-based caching** for automatic freshness
- **Circuit breakers** for fault tolerance
- **Type-safe error handling** for production reliability

Built with **Rust's safety guarantees** and **world-class performance**.

