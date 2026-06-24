# 01 — Quickstart Results

Settings: `n_threads=4`, `n_ctx=2048`, `n_batch=512`, `n_gpu_layers=0`.

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|---:|---:|---:|---:|---:|
| qwen2.5-1.5b-instruct-q4_k_m.gguf | 2279 | 200 / 248 | 51.7 / 54.3 | 1819 / 1853 / 1854 | 19.4 |
| qwen2.5-1.5b-instruct-q2_k.gguf | 1334 | 267 / 329 | 46.6 / 49.5 | 1710 / 1820 / 1829 | 21.4 |

## Observations

- TTFT is the prefill cost. With short prompts this is small; with long prompts it dominates.
- TPOT is per-token decode latency. The decode rate is `1000 / TPOT_p50`.
- The bigger quantization (Q4_K_M) is usually only ~30–60% slower than Q2_K but produces noticeably better text. Q2_K is for *truly* tight RAM.
- `n_threads = physical_cores` is usually best on CPU. Hyperthreading (`logical_cores`) often hurts because the work is bandwidth-bound.

(Edit this file with your own observations before submitting.)
