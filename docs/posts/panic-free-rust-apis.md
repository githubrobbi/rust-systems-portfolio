# Panic‑Free Rust APIs in Production: A Short Playbook

**TL;DR:** For long‑running services and agents, panics are outages. Treat `unwrap/expect/panic/todo` as banned at the workspace level; model failures explicitly.

1) **Deny panics at the workspace**
- Forbid unwrap/expect/panic/todo in clippy and lint config so failures are visible in code review and CI.

2) **Error boundaries and ergonomics**
- Public APIs use typed errors (`thiserror`); glue layers may use `anyhow`. Map transport errors (I/O, HTTP, TLS) to domain errors at boundaries.

3) **Concurrency and shutdown**
- Structured concurrency (Tokio task handles + cancellation tokens). Deterministic, leak‑free shutdown improves test/bench stability.

4) **Guardrails**
- Property tests (proptest); golden files for I/O paths; coverage gates (llvm‑cov); perf harness (criterion/divan); `cargo-deny/audit/outdated`.

5) **When is panic acceptable?**
- In tests and debug‑only assertions; at process boundary for unrecoverable corruption (documented).