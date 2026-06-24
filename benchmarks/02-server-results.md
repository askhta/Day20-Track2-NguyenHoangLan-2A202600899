# 02 - llama-server Load Test Results

Server: `python -m llama_cpp.server` on `127.0.0.1:8080`

Model: `qwen2.5-1.5b-instruct-q4_k_m.gguf`

Settings: `n_threads=4`, `n_ctx=2048`, `n_gpu_layers=0`

Note: this run used the Python `llama_cpp.server` path on Windows. It serves the OpenAI-compatible `/v1/chat/completions` endpoint, but does not expose native llama.cpp `/metrics`.

| Concurrency | Requests | Total RPS | Median E2E (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|---:|---:|---:|---:|---:|---:|---:|
| 10 | 9 | 0.16 | 38000 | 52000 | 52000 | 0 |
| 50 | 13 | 0.24 | 27000 | 53000 | 53000 | 0 |

## Observations

- The CPU server completed requests without failures, but latency was high because it ran the 1.5B Q4_K_M model on CPU with a single server process.
- Increasing users from 10 to 50 did not improve useful throughput much (`0.16` to `0.24` req/s), which suggests the decode path was already saturated.
- Native `/metrics` was not available in this Windows Python-server run. To capture `llamacpp:n_busy_slots_per_decode` and `requests_processing`, build and run the native `llama-server` with `--metrics`.
