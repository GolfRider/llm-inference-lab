# llm-inference-lab

A hands-on lab measuring how LLM inference behaves on a single GPU, from a
Kubernetes platform engineer's perspective. Backs the talk
"From HTTP Services to Token Services: Diagnosing LLM Inference on Kubernetes."

## The problem

An inference pod can be Ready, return 200s, and show normal CPU/GPU metrics
while users wait seconds for the first token. Inference doesn't behave like a
stateless HTTP service. It has three resource axes — compute (prefill),
memory bandwidth (decode), memory capacity (KV cache) — and three latency
signals — TTFT, TPOT, queue time — that move independently. "Slow inference"
is not one problem; the same symptom has different causes and different fixes.

## Setup

- Model: Qwen2.5-7B-Instruct on vLLM 0.6.6.post1
- GPU: NVIDIA A40 (48GB), single node
- Metrics read from vLLM's /metrics (Prometheus format)

## Experiments

- [Context length → TTFT](findings/01-context-length.md): prompt length drives
  prefill/TTFT super-linearly; decode/TPOT stays roughly flat.
- [Concurrency → saturation](findings/02-concurrency.md): short prompts
  saturate compute; long prompts saturate KV-cache capacity — same symptom,
  different cause.

## Glossary

Key inference terms — TTFT, TPOT, prefill, decode, KV cache — are defined
[here](notes/inference-glossary.md).

## Status

Single-node diagnostics. Next: Prometheus/Grafana dashboards, and inference-
aware routing (Gateway API Inference Extension) for the multi-node case.