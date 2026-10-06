# 02 - Serve: load test + saturation reading

Host `Linux-x86_64` · llama.cpp `b10488` · `--parallel 4` · `ctx=2048` · `threads=1` · `ngl=0`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 7 | 0.14 | 44000 | 48000 | 48000 | 5.6 | 0.0% |
| 50 | 7 | 0.14 | 20000 | 51000 | 51000 | 4.4 | 0.0% |

## Reading

The service is saturated by the 50-user run and likely at or below that offered load: five times as many users delivered only 0.95× throughput (0.14 versus 0.14 requests/s after rounding), with effective concurrency 4.4 against four slots. The strongest evidence is throughput failing to scale; only seven requests completed in each one-minute run, so percentiles are indicative. I would first cap output tokens or move to a faster runtime/model to reduce CPU work per request; increasing parallel slots on a two-logical-core CPU would add contention instead of capacity.
