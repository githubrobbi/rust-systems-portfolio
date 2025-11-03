# TTAPI — 2‑Minute Demo Script (public-safe)

**Goal:** illustrate layering, observability, and benchmark harness with synthetic data.

## 1) Ingest → transform → persist
```bash
tt ingest --source fixtures/synth.csv --out .tt/cache --log-level info
tt transform --in .tt/cache --op normalize --out .tt/out
tt persist --in .tt/out --format parquet --ttl 7d
```

### Callouts
- Structured logs (`tracing`) with request IDs and progress.
- Panic‑free public APIs; graceful shutdown via cancellation tokens.
- (Optional) Simulated HTTP with rate limiting and jittered backoff.

## 2) Coverage and benchmarks
```bash
just coverage   # llvm-cov; HTML report
just bench      # criterion/divan; artifacts under target/criterion
```

## 3) Close with architecture
- Crates: core, client, platform, polars, cli, devenv.
- Resilience: circuit breaker, retry/backoff, GOAWAY handling, adaptive rate limiting.
- Release: cargo-dist installers; release-plz workflow; SPDX/REUSE.