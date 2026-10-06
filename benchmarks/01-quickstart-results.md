# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` · Colab CPU `Linux-x86_64` · llama.cpp `b10488`
Settings: `threads=1` `ngl=0` `ctx=2048` · `max_tokens=64` · warm-up discarded
Completed requests: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 3330 | 969 / 1817 | 102.8 / 153.4 | 7460 / 10046 / 10046 | 9.7 |
| UD-Q2_K_XL | 0.39 | 4505 | 1660 / 2451 | 122.7 / 133.4 | 9812 / 10292 / 10292 | 8.2 |

## Observation

Q2 saves 0.11 GB (about 22% of the Q4 file size), but decodes 1.18× slower and takes longer to load on this 1-physical-core CPU. The extra dequantization work appears to outweigh the memory savings here. I did not run a controlled side-by-side answer-quality comparison, so I make no quality claim from these timing measurements.
