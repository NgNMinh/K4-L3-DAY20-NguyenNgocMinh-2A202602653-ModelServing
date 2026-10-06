# 03 - RAG pipeline

Model `Qwen3.5 0.8B` on the Colab CPU runtime; serving used the real `llama-server`. The pipeline used keyword-overlap retrieval because no embedding model was configured (`embeddings: none`), so embedding is a stub/fallback and retrieval is a simple real stage.

| Stage | Mean latency (ms) |
|:--|--:|
| Embed | 0.0 |
| Retrieve | 0.1 |
| LLM | 15505.0 |
| Total | 15505.2 |

The LLM accounted for approximately 100% of total latency. To halve latency, I would reduce generation work first (shorter output limits or a faster inference backend/model); optimizing the 0.1 ms retrieval step cannot materially change the total.
