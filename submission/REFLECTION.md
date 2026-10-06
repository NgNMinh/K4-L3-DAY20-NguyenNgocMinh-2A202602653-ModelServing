# Reflection — Day 20 Lab

- **Họ Tên:** Nguyễn Ngọc Minh
- **MSSV:** 2A202602653
- **Cohort:** K4-L3 / Track 02
- **Ngày submit:** 2026-10-06

## 1. Hardware & runtime

- **OS:** Colab Linux 6.6.122+ (x86_64)
- **CPU:** Intel Xeon @ 2.20 GHz
- **Cores:** 1 physical / 2 logical
- **CPU extensions:** AVX2
- **RAM:** 12.7 GB
- **Accelerator:** CPU only; no GPU detected in this Colab runtime
- **llama.cpp asset:** `llama-b10488-bin-ubuntu-x64.tar.gz` (build b10488)
- **Model:** Qwen3.5 0.8B (`LAB_MODEL=qwen35-0.8b`)
- **Quantization:** Q4_K_M + UD-Q2_K_XL
- **Chạy ở đâu:** Google Colab CPU fallback. I chose the cloud route to keep model/runtime downloads and their disk use off the laptop; these measurements describe the Colab VM. The local hardware probe also detected an RTX 3050, so cloud was a convenience choice rather than evidence that the laptop lacks compute.

**Setup story:** I ran the cloud notebook on CPU, used port 8090 because Colab reserves 8080, and kept the smaller Qwen model. The 0.9 GB of model weights and prebuilt llama.cpp runtime stayed in the temporary VM; no CUDA build was needed.

## 2. Measurement

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|---:|---:|---:|---:|
| Q4_K_M | 0.50 | 3330 | 969 / 1817 | 102.8 / 153.4 | 7460 / 10046 / 10046 | 9.7 |
| UD-Q2_K_XL | 0.39 | 4505 | 1660 / 2451 | 122.7 / 133.4 | 9812 / 10292 / 10292 | 8.2 |

Q2 reduced the model file by 0.11 GB (22%), but ran 1.18× slower on this one-physical-core CPU. That points to dequantization cost outweighing saved memory traffic here. I did not run a controlled same-prompt answer-quality comparison, so I cannot make a quality judgment from this run.

## 3. Serving under load

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Effective concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 0.14 | 44000 | 48000 | 48000 | 5.6 | 0.0% |
| 50 | 0.14 | 20000 | 51000 | 51000 | 4.4 | 0.0% |

- Offered load increased 5×; measured throughput changed 0.95× before rounding (both runs display 0.14 requests/s).
- P95 increased 1.06×.
- At 50 users, effective concurrency was 4.4 with `--parallel=4`.
- **Peak `llamacpp:n_busy_slots_per_decode`:** 4 / 4 slots; the metrics run also recorded 46 deferred requests.

The server was saturated at or below the 50-user load: five times the offered users did not increase throughput, the 4 decode slots were occupied, and 46 requests were deferred. Only 7 requests completed in each one-minute run, so the percentile estimates are thin. To improve goodput at a P95 SLO, I would first reduce output-token work or use a faster runtime/model; adding slots to a 2-logical-core CPU risks more contention. At a 45 s P95 target both observed runs miss the target, but the short summary does not let me calculate the exact fraction of requests meeting it.

## 4. Integration

| Day | Piece | Real or stub? |
|---|---|---|
| N16 Cloud/IaC | Not integrated in this pipeline | Stub / not run |
| N17 Data pipeline | Not integrated in this pipeline | Stub / not run |
| N18 Lakehouse | Not integrated in this pipeline | Stub / not run |
| N19 Vector + features | Keyword-overlap retrieval fallback; no embedding model | Embedding stub, simple retrieval fallback |
| N20 Serving | `llama-server` | Real |

Mean latency over the 3 pipeline queries: embed **0.0 ms**, retrieve **0.1 ms**, LLM **15505.0 ms**, total **15505.2 ms**. The LLM accounted for about **100%** of the measured total. The result matches the expectation that generation on a CPU dominates; for a 2× reduction I would reduce generated tokens or move inference to a faster backend before optimizing retrieval.

## 5. The single change that mattered most

**Change:** Increase llama.cpp thread count from 1 to 2 in the Colab sweep.

```text
before:  9.8 tok/s (tg128, -t 1)
after:   9.9 tok/s (tg128, -t 2)
speedup: 1.01×
```

The second logical thread made almost no difference: the measured gain was about 1%, so this sweep has no meaningful knee. A second logical CPU is an SMT thread sharing the VM's physical core resources, not another full core. Scheduling noise can be as large as this small change. I would keep the configuration simple rather than claim a useful optimization from a 0.1 tok/s difference. This result applies to the Colab VM and should not be read as a tuning result for my Ryzen laptop.

## 6. Bonus

Not completed.

## 7. What surprised me

The 2-bit model used less disk but decoded more slowly than Q4 on the Colab CPU. Lower precision did not automatically make inference faster because the VM had very few CPU resources and no GPU offload.

## 8. Self-check

- [x] Cloud hardware/runtime recorded in `hardware.json`
- [x] Model manifest recorded in `models/active.json`
- [x] Benchmark and load-test reports written
- [ ] Five notebook screenshots saved under `submission/screenshots/`
- [ ] `make verify` exits 0 and all files are tracked
- [ ] Public repository pushed and URL submitted to LMS

## 9. AI use disclosure

I used OpenAI Codex to inspect the repository instructions.