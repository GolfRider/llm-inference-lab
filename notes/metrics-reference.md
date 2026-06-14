# Verified vLLM metric names
# Source: live /metrics from vLLM 0.6.6.post1, Qwen2.5-0.5B-Instruct, A40.
# These map to the three axes. Verify again if vLLM version changes.

## Latency (request-level — these are the SLOs)
vllm:time_to_first_token_seconds        # TTFT (histogram)
vllm:time_per_output_token_seconds      # TPOT (histogram)
vllm:e2e_request_latency_seconds        # end-to-end (histogram)

## Phase split (the gift — per-phase timing)
vllm:request_prefill_time_seconds       # PREFILL phase  -> compute axis
vllm:request_decode_time_seconds        # DECODE phase   -> bandwidth axis
vllm:request_queue_time_seconds         # WAITING phase  -> queueing
vllm:time_in_queue_requests             # time in queue (histogram)

## Regime / load (server-level — these explain the SLOs)
vllm:num_requests_running               # on GPU now
vllm:num_requests_waiting               # queue depth
vllm:num_preemptions_total              # KV-cache eviction events (cliff signal)

## Capacity (the KV / memory axis)
vllm:gpu_cache_usage_perc               # 0..1, KV-cache utilization
# cache_config_info: num_gpu_blocks=204886, block_size=16
#   -> ~3.28M tokens of KV-cache capacity reserved on this A40
#   -> gpu_memory_utilization=0.9, enable_prefix_caching=False

## Throughput
vllm:avg_prompt_throughput_toks_per_s       # prefill tokens/s
vllm:avg_generation_throughput_toks_per_s   # decode tokens/s
vllm:prompt_tokens_total                    # counter
vllm:generation_tokens_total                # counter