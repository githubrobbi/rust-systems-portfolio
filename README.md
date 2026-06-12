# Rust Systems Portfolio — Robert Nio

Public-safe artifacts for backend and systems engineering work in Rust.

This repository collects architecture notes, benchmark summaries, demos, and engineering write-ups from my Rust project work. It is meant to show how I approach performance, API design, testing, and operational reliability without exposing private source code.

---

## Start here

If you only open a few links, these are the best entry points:

- **UFFS (public code)**  
  https://github.com/skyllc-ai/UltraFastFileSearch

- **Portfolio home**  
  https://githubrobbi.github.io/rust-systems-portfolio/

- **TTAPI one-pager**  
  https://githubrobbi.github.io/rust-systems-portfolio/ttapi-one-pager.html

- **TTAPI demo**  
  https://githubrobbi.github.io/rust-systems-portfolio/demo/ttapi-demo-script.html

- **Panic-Free Rust APIs**  
  https://githubrobbi.github.io/rust-systems-portfolio/posts/panic-free-rust-apis.html

- **Bench-First Rust Development**  
  https://githubrobbi.github.io/rust-systems-portfolio/posts/bench-first-rust-development.html

---

## What this repository covers

### Public-safe project artifacts
- **UFFS (public code)** — a Rust NTFS search/indexing project with benchmarked cold/warm behavior, persisted cache, and daemon-backed query flow. Current canonical benchmark (v0.5.120): **30/30 head-to-head cells faster than Everything** at p50 (median ~2.8× faster) and a full 7-drive, 23.3M-record CSV export in 12.0 s (~1.95M records/sec) — every number version-stamped and backed by raw logs in the [benchmark hub](https://github.com/skyllc-ai/UltraFastFileSearch/tree/main/docs/benchmarks).
- **TTAPI (private code, public-safe materials)** — architecture notes, benchmark summaries, and demo pages for a Rust data/analytics workspace.
- **Engineering write-ups** — short articles on panic-free APIs, bench-first development, and practical error handling.

### Engineering themes
- Indexing, data-intensive workflows, and caching
- Async concurrency with Tokio
- Data pipelines and structured logging
- Clear APIs, typed errors, and operational predictability
- Reproducible benchmarks and measurable performance work

### Measured examples included in the docs
- Cached vs. uncached workflow comparisons, including a documented example of roughly **23x** speedup from cache reuse
- Data import throughput examples around **438,000 rows/second** in documented runs
- Bounded concurrency and backpressure patterns for networked workflows
- Test, lint, and CI practices used to keep changes reviewable and repeatable

> Performance figures in this repository come from documented runs on specific workloads and environments. They are intended to show measurement discipline and engineering trade-offs, not universal benchmarks.

---

## Live site

**GitHub Pages:** https://githubrobbi.github.io/rust-systems-portfolio/

The site is built from the `/docs` folder using GitHub Pages with Jekyll.

---

## Documentation structure

```text
docs/
├── index.md
├── performance-benchmarks.md
├── rust-excellence.md
├── architecture-deep-dive.md
├── why-rust.md
├── ttapi-one-pager.md
├── demo/
│   ├── ttapi-demo-script.md
│   └── README.md
└── posts/
    ├── panic-free-rust-apis.md
    └── bench-first-rust-development.md
```

---

## About TTAPI

TTAPI is a private Rust workspace for financial data collection and analysis. The public materials in this repository focus on:

- Workspace structure and crate boundaries
- Tokio-based concurrency and bounded parallelism
- Data processing and persistence workflows
- Retry, backoff, caching, and resilience patterns
- Typed error handling and structured logging
- Benchmarking and performance-oriented development practices

The TTAPI source code is **not** included here. Public pages use synthetic or non-sensitive examples and focus on architecture, benchmarks, and engineering practices.

---

## Why this repo exists

This repository is documentation-first by design. Some of my work is public code, and some is represented through public-safe artifacts only. The goal is to make my approach easy to evaluate without exposing private source or sensitive implementation details.

If you want the strongest public code example first, start with **UFFS**:
https://github.com/skyllc-ai/UltraFastFileSearch

---

## Contact

**Robert Nio**  
Backend / Systems Engineer  
[GitHub](https://github.com/githubrobbi) | [LinkedIn](https://linkedin.com/in/robert-nio-46029a)

---

## License

**Documentation:** Creative Commons Attribution 4.0 International (CC BY 4.0)  
**TTAPI Source Code:** MPL-2.0 OR LicenseRef-TTAPI-Commercial (not included in this repository)

See [LICENSE](LICENSE) for full details.

**Copyright © 2025 SKY, LLC. All rights reserved.**

For licensing inquiries regarding TTAPI source code, contact: skylegal@nios.net
