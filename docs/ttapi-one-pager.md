# TTAPI — Public‑Safe One‑Pager (Rust)

**Owner:** Robert Nio  •  **Date:** 2025-11-03

## What it is
A multi‑crate **Rust** workspace for options data ingest/analysis with production‑grade engineering rigor. CLI‑first design with strict layering and a thin app façade.

## Architecture (sanitized overview)
- **Crates**: `ttapi-core` (types, errors, config, TTL, concurrency, macros), `ttapi-client` (auth, crypto, circuit breaker, HTTP/2 tuning, retry/backoff, rate limiting + chunking, CSV/Parquet I/O), `ttapi-platform` (domain orchestration via traits), `ttapi-polars` (Polars utilities), `ttapi-cli` (UX), `ttapi-devenv` (dev/release automation).
- **Layering**: app → platform → client → core. Business logic and I/O are independent of the CLI surface.

## Concurrency, resilience, and performance
- Tokio (rt‑multi‑thread), structured task management and cancellation; WebSocket streaming; HTTP/2 GOAWAY aware.
- Circuit breaker, jittered retry/backoff, adaptive rate limiting, chunked I/O; memory discipline (SmallVec, explicit precision).
- Bench suites (criterion/divan); flamegraphs and perf scripts.

## Data & persistence
- Polars (LazyFrame‑first), CSV/Parquet I/O; TTL‑aware caches with on‑disk fallback; content‑addressed patterns where useful.

## Quality, release, and supply chain
- Workspace lints: clippy pedantic/nursery/cargo **deny**; forbid `unwrap/expect/panic/todo`; documented `unsafe`.
- Tests: `nextest`, property tests (proptest), golden files; coverage gates **≥90%** via `cargo-llvm-cov`.
- Release: `cargo-dist` installers, `release-plz` automation, SPDX/REUSE; `cargo-deny/audit/outdated` for dependencies.

## Observability
- `tracing` to terminal and file, progress surfaces for long‑running tasks; tokio‑console guidance in docs.

## Demo (no proprietary data)
1) Run CLI against synthetic fixtures to show ingest → transform → persist.
2) Show coverage/bench artifacts.
3) Review the architecture diagram and resilience patterns.

> Code is private; live demo available under NDA. This one‑pager is public‑safe.