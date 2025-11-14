# Rust Systems Portfolio — Robert Nio

**World-class Rust systems programming: safety, performance, and production-grade architecture.**

This repository showcases my Rust systems work through **public-safe documentation** and **real-world performance data** from TTAPI, a high-performance financial data platform.

---

## What's Inside

### 📊 Performance Benchmarks
- **23x faster** end-to-end pipeline with intelligent caching
- **438,000 rows/second** data import throughput
- **8x more memory efficient** than Python/Pandas equivalents
- Real-world performance data from production runs

### 🏗️ Architecture Deep Dives
- Workspace structure (Polars-inspired domain separation)
- Async/await concurrency with Tokio
- Zero-copy data processing with Polars
- Circuit breakers & resilience patterns
- TTL-based intelligent caching

### ⚖️ Technology Comparisons
- Rust vs. Python/Pandas (40x faster)
- Rust vs. C++ (memory safety without GC)
- Rust vs. Go (no GC pauses)
- Data-driven performance analysis

### 📝 Engineering Write-Ups
- Panic-free Rust APIs (420 lines with real TTAPI examples)
- Bench-first development (445 lines with optimization process)
- Production-grade error handling with `thiserror`

### 🎥 Demo Videos
- Initial run (cold cache): 3m 2s full pipeline
- Cached run: 7.8s with TTL-based cache hits (23x speedup)
- Real production performance with actual data

---

## Live Site

**GitHub Pages:** https://githubrobbi.github.io/rust-systems-portfolio/

The site is built from the `/docs` folder using GitHub Pages with Jekyll.

---

## Key Highlights

🚀 **23x faster** end-to-end pipeline with intelligent caching
⚡ **438,000 rows/second** data import throughput
💾 **8x more memory efficient** than Python/Pandas equivalents
🔄 **100 concurrent requests** with semaphore-based backpressure
🎯 **≥90% test coverage** with zero clippy::pedantic warnings
🛡️ **Circuit breakers & resilience** for fault-tolerant operation

---

## Documentation Structure

```
docs/
├── index.md                          # Main landing page (162 lines)
├── performance-benchmarks.md         # Detailed performance metrics (233 lines)
├── rust-excellence.md                # Architecture & code examples (233 lines)
├── architecture-deep-dive.md         # System diagrams & data flow (233 lines)
├── why-rust.md                       # Technology comparison (233 lines)
├── ttapi-one-pager.md               # Quick technical overview (224 lines)
├── demo/
│   ├── ttapi-demo-script.md         # Demo videos page (103 lines)
│   ├── TTAPI 1st run 2025-11-13.mov # Initial run video (8.4 MB)
│   ├── TTAPI 2nd run 2025-11-13.mov # Cached run video (3.1 MB)
│   └── README.md                     # Video documentation
└── posts/
    ├── panic-free-rust-apis.md      # Error handling patterns (420 lines)
    └── bench-first-rust-development.md  # Performance practices (445 lines)
```

**Total:** 2,286 lines of comprehensive technical documentation + 2 demo videos

---

## About TTAPI

TTAPI is a **production-grade Rust application** for financial data collection and analysis, demonstrating:
- Async/await concurrency with Tokio's work-stealing scheduler
- Zero-copy data processing with Polars' columnar format
- Type-safe error handling with `thiserror` and `Result<T, E>`
- Workspace architecture inspired by Polars (domain-separated crates)
- Circuit breakers & resilience patterns for fault-tolerant systems
- TTL-based intelligent caching with automatic invalidation
- Dual-format persistence (Parquet + CSV) with background I/O

**Note:** The TTAPI source code is private. This repository contains only public-safe documentation and performance data.

---

## Contact

**Robert Nio**
Rust Systems Engineer
[GitHub](https://github.com/githubrobbi) | [LinkedIn](https://linkedin.com/in/robertnio)

---

## License

**Documentation:** Creative Commons Attribution 4.0 International (CC BY 4.0)
**TTAPI Source Code:** MPL-2.0 OR LicenseRef-TTAPI-Commercial (NOT included in this repository)

See [LICENSE](LICENSE) for full details.

**Copyright © 2025 SKY, LLC. All rights reserved.**

For licensing inquiries regarding TTAPI source code, contact: skylegal@nios.net
