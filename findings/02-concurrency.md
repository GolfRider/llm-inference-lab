# Concurrency → saturation

Fixed prompt + output, swept concurrent requests. Two runs: short (~512-token)
and long (~4000-token) prompts.

## Short prompts (~512 tokens)
| conc | tok/s | TTFT_ms | queue_ms |
|-----:|------:|--------:|---------:|
| 1    | 33.5  | 73.6    | 0.4      |
| 16   | 337.9 | 825.9   | 2.5      |
| 64   | 566.8 | 4130.0  | 9.7      |

Throughput scales to ~567 tok/s; queue stays near zero. **Compute-bound** —
TTFT degrades through batch contention, not queueing.

## Long prompts (~4000 tokens)
| conc | tok/s | TTFT_ms | queue_ms |
|-----:|------:|--------:|---------:|
| 1    | 26.5  | 524.0   | 1.2      |
| 8    | 79.4  | 3367.3  | 4.3      |
| 16   | 84.0  | 6379.0  | 2016.9   |
| 48   | 97.8  | 15661.4 | 11144.2  |

Throughput collapses to ~98 tok/s; queue explodes from ~4ms to 11s between
concurrency 8 and 16. **Capacity-bound** — KV cache fills, requests can't be
admitted and wait.

## The key lesson
Same symptom ("slow"), same GPU pegged in both — but one is compute-bound
(fix: add replicas) and one is capacity-bound (fix: cap context / offload
cache / route differently). Neither is visible in CPU or GPU-utilization
dashboards. You need token-level signals (queue time, KV-cache usage).

## Honest note
Instantaneous gauge reads (running/waiting/kv%) were unreliable sampled once
per level; the cumulative histogram metrics (TTFT, queue, throughput) are
trustworthy. This is why production observability scrapes on intervals —
future work: Prometheus + Grafana for the live capacity cliff.