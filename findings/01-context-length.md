# Context length → TTFT (concurrency 1)

Fixed output (64 tokens), swept prompt length. vLLM 0.6.6, Qwen2.5-7B, A40.

| in_tok | TTFT_ms | prefill_ms | decode_ms | TPOT_ms |
|-------:|--------:|-----------:|----------:|--------:|
| 121    | 37.3    | 36.5       | 1831.1    | 29.07   |
| 511    | 73.6    | 73.1       | 1818.1    | 28.86   |
| 2041   | 261.8   | 260.9      | 1824.4    | 28.96   |
| 8191   | 1736.0  | 1734.0     | 1934.3    | 30.70   |

## What it shows
- TTFT ≈ prefill at concurrency 1 (within ~1ms) — with no queue, TTFT *is* prefill.
- Prefill grows super-linearly: input ×4 from 2k→8k, but prefill ×6.6 — attention's O(n²) cost emerging at long context.
- TPOT stays ~flat (29ms), creeps to 30.7ms at 8k — decode is per-token, with a small tax for attending over a longer KV cache.
- Compute axis isolated: input length drives prefill/TTFT, leaves decode largely untouched.