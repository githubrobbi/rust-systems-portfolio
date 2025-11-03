# Bench‑First Rust Development: Make Performance a Feature

**Premise:** If you don’t measure it, you don’t own it.

1) **Harness first**
- `criterion`/`divan` with fixed seeds and pinned CPU governor; synthetic corpora in `fixtures/`.

2) **Reproducibility**
- Record environment; version bench results; stabilize runs (warmups/samples).

3) **CI perf gates**
- GitHub Action compares PR vs main; fail on >X% p95 regression; publish artifacts.

4) **Investigate with flamegraphs**
- `cargo flamegraph`/`perf` to find hotspots; fix alloc storms/syscalls first; PRs stay small.

5) **Communicate results**
- Report p50/p95 latency, throughput, memory, syscalls; tie to user value (cold‑start time, CPU/RSS on shared hosts).