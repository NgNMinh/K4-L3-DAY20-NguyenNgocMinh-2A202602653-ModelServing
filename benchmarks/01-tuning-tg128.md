# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` · Colab CPU · llama.cpp `b10488`
CPU: **1 physical · 2 logical** cores · `ngl=0` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 9.8 | 99% |
| 2 | 9.9 | 100% |

**Best**: `-t 2` at 9.9 tok/s · **Slowest**: `-t 1` at 9.8 tok/s · spread `1.01x`.

## Explanation

There is no clear knee in this short sweep: the second logical thread improves throughput by only 0.1 tok/s (about 1%). The curve is effectively flat, so SMT offers little extra compute for this model on the allocated VM; measurement noise and shared-host scheduling could explain a change this small. The result describes the Colab VM, not my laptop.
