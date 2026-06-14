# Environment & setup notes

## Working stack (reproducible baseline)
- Provider: RunPod, region EU-SE
- GPU: NVIDIA A40, 48GB (46068 MiB usable)
- Driver: 565.57.01, CUDA driver version 12.7
- vLLM: 0.6.6.post1
- torch: 2.5.1 (cu124 build)
- transformers: 4.46.*
- Model (mechanics): Qwen/Qwen2.5-0.5B-Instruct
- HF_HOME=/workspace/hf  (model cache on the persistent network volume)

## Launch command that worked
    export HF_HOME=/workspace/hf
    vllm serve Qwen/Qwen2.5-0.5B-Instruct
    # server: http://0.0.0.0:8000  (OpenAI-compatible + /metrics)

## Dependency gotchas (day-2 platform pain, real talk material)
1. CUDA driver too old: latest vLLM is built for a newer CUDA than the
   pod's 12.7 driver. Error: "NVIDIA driver too old (found version 12070)".
   Fix: pin to a vLLM built for CUDA <=12.7  ->  pip install "vllm==0.6.*"
   (installed 0.6.6.post1 + torch 2.5.1/cu124).
2. transformers mismatch: pinned vLLM 0.6.6 expected an older transformers;
   newer Qwen2Tokenizer lacked `all_special_tokens_extended`.
   Fix: pip install "transformers==4.46.*"
   Root cause: generic PyTorch template ships mismatched library versions.
   Cleaner path next time: use a RunPod vLLM-specific template.

   Lesson: there are TWO CUDA versions — the host *driver* CUDA (fixed, 12.7)
   and the *runtime* CUDA that torch/vLLM were built against. Runtime must be
   <= driver. You can't raise the driver on a rented box, so lower the framework.

## Cold-start baseline (why you can't autoscale LLMs in milliseconds)
- CUDA graph capture: 35 shapes, ~17s, 0.15 GiB
- Engine init (profile + create KV cache + warmup): 20.93s
- This ~21s is most of the cold start a new replica must pay before serving.

## KV-cache pool (capacity axis, concrete)
- num_gpu_blocks=204886, block_size=16  ->  ~3.28M tokens capacity
- gpu_memory_utilization=0.9, enable_prefix_caching=False
- Note: 7-8B model will reserve far fewer blocks -> easier to hit the wall.

## Cost discipline
- Stop the pod when not actively working, to keep the costs under control.
