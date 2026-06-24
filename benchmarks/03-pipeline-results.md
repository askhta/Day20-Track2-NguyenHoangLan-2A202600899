# 03 - Milestone Integration Results

Server: `python -m llama_cpp.server` on `127.0.0.1:8080`

Pipeline: `03-milestone-integration/pipeline.py`

| Query | Retrieved contexts | Retrieve (ms) | llama-server (ms) | Total (ms) |
|---|---|---:|---:|---:|
| Why is goodput more useful than throughput? | `n20-paged`, `n20-radix`, `n20-disagg` | 0.0 | 9973.1 | 9973.1 |
| What problem does PagedAttention actually solve? | `n20-paged`, `n20-radix`, `n20-disagg` | 0.0 | 3238.7 | 3238.8 |
| When should I think about disaggregated serving? | `n20-disagg`, `n20-paged`, `n20-radix` | 0.0 | 12304.5 | 12304.6 |

## Notes

- N16 was stubbed as localhost serving.
- N17 was stubbed as an in-process batch/query flow.
- N18 was stubbed as in-memory document records.
- N19 was stubbed as keyword-overlap retrieval over `TOY_DOCS`.
- The pipeline prints retrieved context IDs for provenance before the generated answer.
