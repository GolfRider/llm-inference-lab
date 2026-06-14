## Glossary

# TTFT (time to first token)
Time from request arrival to the first generated token reaching the user.
Roughly = queue wait + prefill. Important nuance: under load, high TTFT is
usually queueing, not slow compute — the request is sitting in line.

# TPOT (time per output token)
Time to generate each subsequent token after the first, i.e. how fast the
answer streams. Set by the decode phase; memory-bandwidth-bound.

# Tokens/sec
Throughput — total tokens generated per second across all in-flight requests.
A server-level health/throughput signal, distinct from per-request latency.

# Prefill
First phase of inference: process all prompt tokens in a single parallel
forward pass, populating the KV cache for the prompt. Compute-bound; cost
scales with prompt length; dominates TTFT.

# Decode
Second phase: generate output tokens one at a time, sequentially, each step
depending on the last. Memory-bandwidth-bound (re-streams model weights each
step); sets TPOT. Every generated token's key/value vectors are appended to
the KV cache.

# KV cache
Per-request GPU memory holding the key/value vectors of all tokens processed
so far, so each new token can attend to the past without recomputing it.
This is why decode is fast (reuses past work) and why memory fills up (every
token's K/V stays resident for the request's life). Size grows with
context length × concurrent requests, and it's bounded by GPU memory —
the capacity axis.

# Context length
Total tokens in a request's context (prompt + tokens generated so far).
Longer context = more prefill compute (higher TTFT) and more KV-cache memory
consumed. The lever behind both the compute and capacity axes.

# Queue depth
Number of requests waiting to be admitted to the running batch. Grows when
arrivals outpace batch capacity — from compute saturation OR KV-cache being
full (no memory to admit a new request). Under healthy load it's a transient
buffer; sustained growth means saturation.

# Batching
Processing many requests together to keep the massively parallel GPU busy —
raises total throughput. But larger batches and higher concurrency raise
per-request latency and KV-cache pressure. Batching is the throughput-vs-
latency knob, not a free win.

# Continuous batching
vLLM's scheduling approach: instead of fixed batches, it merges and releases
requests into a rolling batch token-by-token, so finished requests free
capacity immediately. This is why adding concurrency raises throughput up to
a knee, then raises latency past it.

# Throughput vs latency operating point
The core reframe: there is no fixed "fast" or "slow." Batching/concurrency
settings choose a point on a curve — more batching = more throughput, worse
tail latency. You pick the point that meets your SLO.

## The three axes (the unifying model)
Every "why is it slow?" reduces to which axis is the bottleneck right now:
- Compute      → prefill        → sets TTFT
- Bandwidth    → decode         → sets TPOT
- Capacity     → KV cache       → causes the saturation cliff
  Three resource axes, three latency numbers, moving independently.