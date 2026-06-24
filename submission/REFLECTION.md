# Reflection - Lab 20 (Personal Report)

**Ho Ten:** Nguyen Hoang Lan
**Cohort:** A20
**Ngay submit:** 2026-06-24

## 1. Hardware spec

- **OS:** Windows
- **CPU:** 11th Gen Intel(R) Core(TM) i7-11390H @ 3.40GHz
- **Cores:** 4 physical / 8 logical
- **CPU extensions:** AVX2/AVX-512 reported by llama.cpp runtime
- **RAM:** 15.7 GB
- **Accelerator:** NVIDIA GeForce MX450, 2048 MiB
- **llama.cpp backend da chon:** CPU wheel on Windows for the measured run
- **Recommended model tier:** Qwen2.5-1.5B-Instruct (Q4_K_M)

**Setup story:** May co MX450 2 GB nen minh uu tien Windows CPU wheel de setup on dinh, tai Qwen2.5-1.5B Q4_K_M/Q2_K tu Hugging Face, va chay server local OpenAI-compatible tren `localhost:8080`.

## 2. Track 01 - Quickstart numbers

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|---:|---:|---:|---:|---:|
| qwen2.5-1.5b-instruct-q4_k_m.gguf | 2279 | 200 / 248 | 51.7 / 54.3 | 1819 / 1853 / 1854 | 19.4 |
| qwen2.5-1.5b-instruct-q2_k.gguf | 1334 | 267 / 329 | 46.6 / 49.5 | 1710 / 1820 / 1829 | 21.4 |

**Mot quan sat:** Q2_K decode nhanh hon Q4_K_M khoang 10%, nhung loi ich nho so voi mat mat chat luong. Tren may nay Q4_K_M van la default hop ly neu RAM du.

## 3. Track 02 - llama-server load test

Server run: `python -m llama_cpp.server --model models/qwen2.5-1.5b-instruct-q4_k_m.gguf --host 127.0.0.1 --port 8080 --n_threads 4 --n_gpu_layers 0 --n_ctx 2048`

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|---:|---:|---:|---:|---:|---:|
| 10 | 0.16 | 38000 | 52000 | 52000 | 0 |
| 50 | 0.24 | 27000 | 53000 | 53000 | 0 |

**Batching observation:** Python `llama_cpp.server` tren Windows khong expose native `/metrics`, nen khong co `llamacpp:n_busy_slots_per_decode`/`requests_processing` trong run nay. Ket qua locust cho thay server CPU da sat tran decode: tang tu 10 len 50 users chi tang RPS tu 0.16 len 0.24, trong khi P95 van quanh 52-53 giay.

## 4. Track 03 - Milestone integration

- **N16 (Cloud/IaC):** stub: localhost only, OpenAI-compatible llama server on `127.0.0.1:8080`
- **N17 (Data pipeline):** stub: synchronous Python query flow
- **N18 (Lakehouse):** stub: in-memory `TOY_DOCS`
- **N19 (Vector + Feature Store):** stub: keyword-overlap retriever returning provenance IDs

Pipeline timings from `03-milestone-integration/pipeline.py`:

| Query | Retrieved contexts | retrieve (ms) | llama-server (ms) | total (ms) |
|---|---|---:|---:|---:|
| Why is goodput more useful than throughput? | `n20-paged`, `n20-radix`, `n20-disagg` | 0.0 | 9973.1 | 9973.1 |
| What problem does PagedAttention actually solve? | `n20-paged`, `n20-radix`, `n20-disagg` | 0.0 | 3238.7 | 3238.8 |
| When should I think about disaggregated serving? | `n20-disagg`, `n20-paged`, `n20-radix` | 0.0 | 12304.5 | 12304.6 |

**Reflection:** Bottleneck nam gan nhu hoan toan o llama-server. Retrieval in-memory gan bang 0 ms, con generation tren CPU mat 3.2-12.3 giay tuy query va output length. Dieu nay khop ky vong vi decode CPU bi gioi han boi memory bandwidth.

## 5. Bonus - The single change that mattered most

**Change:** Dung Q2_K thay cho Q4_K_M de do tradeoff quantization tren cung model Qwen2.5-1.5B.

**Before vs after:**

```text
before: Q4_K_M TPOT P50 51.7 ms, decode 19.4 tok/s
after:  Q2_K   TPOT P50 46.6 ms, decode 21.4 tok/s
speedup: ~1.10x
```

**Tai sao no work:** Q2_K nen trong so manh hon nen model nho hon va moi token can it data hon tu RAM. Tren CPU laptop, decode thuong bi gioi han boi memory bandwidth hon la compute thuan, vi vay giam bytes per weight co the tang tok/s.

Nhung speedup chi khoang 10%, khong phai 2x. Ly do la overhead tokenizer, prompt processing, sampling, va memory access khac van con do. Voi laptop nay, Q4_K_M co ve dang tien hon: cham hon nhe nhung giu chat luong tot hon Q2_K.

## 6. Dieu ngac nhien nhat

Locust 50 users khong lam throughput tang dang ke. Khi server CPU da saturate, them concurrency chu yeu lam request xep hang lau hon thay vi tang goodput.

## 7. Self-graded checklist

- [x] `hardware.json` da commit
- [x] `models/active.json` da tao
- [x] `benchmarks/01-quickstart-results.md` da tao
- [x] `benchmarks/02-server-results.md` da tao
- [ ] Native `/metrics` CSV chua co vi chua build native llama.cpp server
- [ ] Bonus sweep rieng chua chay
- [ ] Screenshots can duoc kiem tra truoc khi submit LMS
- [x] `make verify`/`python scripts/verify.py` duoc chay lai trong buoc cuoi
- [ ] Repo tren GitHub can de public khi submit
