# 02 - Continuous batching evidence (50 users)

During the one-minute 50-user Locust run, the server used `--parallel 4`. The metrics recording overlapped the load test and observed up to **4 busy decode slots** with **46 deferred requests**. This shows the queue was waiting behind occupied slots; raising the slot count alone is unlikely to help on a VM with one physical CPU core. The run completed 7 requests with no failures, so this is a short, indicative sample.
