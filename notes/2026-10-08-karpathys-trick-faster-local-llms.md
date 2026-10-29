# Source

- Author: [@akshay_pachaar](https://x.com/akshay_pachaar)
- Article/Post: https://x.com/i/article/2108246386538094592
- Status: https://x.com/akshay_pachaar/status/2108252095380140203
- Date: 2026-10-08
- Title: Karpathy’s Trick for Faster Local LLMs, finally has a proper solution

## Long summary

In 2023, Karpathy explained that local LLMs are slow because a single generation stream leaves the chip waiting on weights from memory while compute sits idle. He then described the workaround: checking several tokens in one pass costs about the same as checking one, so a small draft model can guess ahead and the main model can verify those guesses together.

The first point still sets the ceiling. A base Apple M5 moves 153 GB of data per second. A 4-bit Qwen3.5 9B model weighs about 5.2 GB. At one token per pass that chip tops out near 29 tokens per second (153 ÷ 5.2 ≈ 29). llama.cpp and MLX sit under that line at 22 and 25 tokens per second on the same coding prompt. Local setups such as llama.cpp’s MTP mode typically draft only three or four tokens at a time.

Uzu, an open-source inference engine from Mirai, pushes speculative decoding further on Apple Silicon. It drafts up to 16 tokens ahead, keeps those guesses coherent, and verifies them in one pass on kernels written for the M5. On the same M5, model, and prompt, Uzu wrote 92 to 117 tokens per second.

LLM inference has two phases: prefill (prompt tokens in parallel) and decode (one token at a time). Decode is the relevant bottleneck. At batch size one, each pass streams every weight from memory and does roughly one multiply-add with each. The GPU often finishes the arithmetic before the next weight block arrives, so speed is limited by memory bandwidth. Real engines land a little below the 29 tok/s ceiling because each pass also reads other data such as the cache of earlier tokens. Quantization reduces bytes read per pass, but it still optimizes the same one-token loop.

Uzu targets two quantities in the decoding loop: fewer bytes transferred per pass, and more accepted tokens per pass. Quantization handles the first; speculative decoding handles the second. Many local engines already support 4-bit weights. Mirai publishes multiple optimized checkpoints for the same base model, labeled by size tier (M means Medium, L means Large). Those labels describe the checkpoint, not a different architecture. The Medium checkpoint uses 4-bit asymmetric integer quantization; the Large checkpoint uses 8-bit symmetric quantization. Mirai’s quantization write-up describes combining post-training quantization with quantization-aware distillation.

The integer format is intentional. Unlike older Apple GPUs, the M5 reaches its highest arithmetic throughput in int8. Uzu also quantizes activations to 8 bits on the fly so speculative verification can run on the M5’s hardware-accelerated int8 matrix path.

Group-wise quantization introduces rounding error when a group contains an outlier, because every value in the group shares one scale. Uzu applies a block-diagonal Random Hadamard Transform before quantization. A Hadamard matrix contains only +1 and -1 entries, plus one overall scale. Multiplying a block of weights by it mixes every value into every position, so a single large outlier becomes many moderate values. “Random” means random sign flips before mixing; “block-diagonal” means each block of 32 values is mixed on its own. The transform is exactly reversible, so apart from rounding it does not change what the model computes. Mirai selected 32 elements to match the Apple GPU kernel’s SIMD width (one instruction across 32 GPU threads). The transform block and the quantization group control different operations and need not match.

Speculative decoding adds a small draft model beside the main model. The draft proposes future tokens cheaply; the main model checks those positions together, accepts the valid prefix, and resumes from the first rejected token. If a drafter proposes five tokens and the main model agrees with the first four, that one verification pass advances the sequence by five tokens (four accepted drafts plus the main model’s own token at the fifth position). Ordinary decoding would need five separate main-model passes. A correct acceptance procedure preserves the target model’s output distribution. Verification costs more arithmetic than one-token decoding, but local batch-one inference often has idle arithmetic while weights move through memory.

The trade-off depends on acceptance. Longer drafts create larger verification batches and more chances for an early mistake to invalidate later tokens. Many model-native systems, including multi-token prediction (MTP), draft three or four tokens. Uzu uses budgets of 16 to 32 tokens on its M5 path. The Qwen3.5 9B checkpoint used later in the article ships with a 16-token tree budget for M5-series chips. Checking 16 candidate tokens at once gives the M5’s Neural Accelerators 16 rows of work instead of 1.

A fast parallel drafter predicts several future positions at once, but each position sees the existing context, not tokens selected at earlier draft positions. Individually plausible tokens can form an invalid sequence (the article’s example: “a few of milk”). The Trees from Marginals paper introduces Weaver to repair this. Weaver is a 56.7M-parameter autoregressive adapter in the paper’s research setup. It operates on candidate shortlists rather than the full vocabulary. A parallel DFlash drafter first predicts likely candidates for each future position; Weaver then selects from those candidates in sequence so later choices can depend on earlier ones (the same example can become “a gallon of milk”). Weaver can also retain several coherent paths; the main model verifies the tree and accepts the longest valid path. The tree shape can adapt to the prompt: predictable code can support a long, narrow branch; open-ended writing may need shorter branches with more alternatives. The paper reports a 24.7 percent improvement over an optimized DFlash baseline and a 4.37-fold speedup over autoregressive decoding. Those experiments used CUDA and SGLang and are research results, not universal Uzu measurements. Uzu applies the same drafting idea to its Apple-focused runtime.

Some layers keep a running internal state. A draft tree creates several possible next sequences; updating live state while checking one branch would require restoring it before checking another. Qwen3.5 and Qwen3.6 contain stateful Gated DeltaNet layers. An attention layer keeps a growing cache of every past token; a Gated DeltaNet layer compresses what it has seen into a fixed-size state that every accepted token changes. In Qwen3.5 9B, 24 of the 32 layers are of this type. Uzu separates checking a branch from committing its state: during verification it computes values for every candidate branch without changing the live state, then applies updates only for accepted tokens. The Uzu implementation note calls this rollback-free tree verification. Its Metal kernels compute the DeltaNet outputs for the whole tree in a single recurrence-free pass. Together, Weaver and rollback-free verification let Uzu propose long sequences and check them without repeatedly restoring model state.

On a MacBook Pro with a base M5 and 16 GB of unified memory, the article ran the same Qwen3.5 9B model three ways, all with 4-bit weights and the prompt “Write a python function to merge two sorted lists.”: llama.cpp with no speculative decoding; llama.cpp with the model’s built-in MTP heads drafting 3 tokens ahead; and Uzu with DFlash and Weaver.

Published speculative results for Qwen3.6 27B Mirai-M (15.6 GB) on an M5 Max with 128 GB of memory put Uzu at 2.1 times MTPLX, 3.8 times llama.cpp, and 4.4 times MLX. Uzu reaches 35.54 tokens per second without speculation and 114 tokens per second with speculation (a 3.2-times gain). Speculative speed is a workload property: code and maths have many predictable continuations; open-ended conversation gives the drafter fewer reliable tokens.

Uzu’s command-line tool installs through Homebrew and downloads the selected model on first use. The article ran Mirai CLI 0.6.1 on a MacBook Pro with a base M5, 16 GB of unified memory, and macOS 26.6.2. After a run, the CLI reported time to first token, prefill speed, generation speed, tokens per forward pass, and memory used. An 8.39 t/f value means each forward pass of the 9B model produced about 8 tokens on average, which is how generation reaches 98 tok/s on a chip whose one-token limit is about 29. Across runs of that command, generation speed ranged from 92 to 117 tok/s, with 7.5 to 9.6 tokens per pass. Speculation is enabled per chip: this checkpoint’s speculator configuration lists the M5, M5 Pro, and M5 Max; on M1 to M4 Macs the same command runs without a drafter and reports 1.00 t/f.

Uzu can expose an OpenAI-compatible HTTP endpoint on 127.0.0.1 so the model still runs on the user’s Mac. By default Uzu keeps a prefix cache of already-processed prompt starts; `--no-prefix-cache` turns this off so repeated test requests do the full work. An existing application can point its OpenAI client at `http://127.0.0.1:8000/v1`.

Uzu, MLX LM, and llama.cpp can all expose OpenAI-style chat endpoints. In the article’s recording, Uzu writes about 3.7 times faster than MLX on the same chip. The comparison uses complete engine paths, not a kernel-only test: Uzu uses Mirai-M, MLX uses its own 4-bit conversion, and llama.cpp uses Q4_K_S. Start servers one at a time so only one copy of Qwen3.5 9B occupies unified memory. For llama.cpp, `-c 8192` keeps it from reserving the model’s full 262K context (which would not fit next to the weights on a 16 GB Mac); `-np 1` runs a single slot; MTP drafting uses `--spec-type draft-mtp --spec-draft-n-max 3`. `uv tool update-shell` adds the directory containing `mlx_lm.server` to the shell path. A standalone `compare_client.py` (Python standard library only) sends the same deterministic prompt to any OpenAI-compatible local endpoint, turns thinking off, and reports time to first token, end-to-end time, and decode speed; for Uzu it also prints tokens per pass. Run each client twice, discard the first (warm-up) result, and keep the second. The client’s decode speed includes HTTP streaming, so compare client numbers with each other, not with CLI numbers. The Mirai CLI has no flag to switch speculation off, so the procedure compares complete engine paths rather than isolating DFlash-Weaver.

The conclusion: loading a quantized checkpoint is only the first step. Uzu addresses batch-one generation with quantization, Metal kernels for Apple hardware, DFlash-Weaver for longer proposals, and rollback-free verification. On a base M5, that combination took a 4-bit 9B model from 22 tokens per second with plain llama.cpp to 92 with Uzu on the same prompt. Acceptance changes with the workload; hardware, quantization quality, context length, and thermal state can also alter the result. The correct test uses the application’s prompts and measures quality alongside latency. The Uzu repository contains the engine, local server, build instructions, and bindings for Rust, Swift, Python, and TypeScript.

## Key claims

- Karpathy’s 2023 account still sets the local batch-one ceiling: a base M5 at 153 GB/s with a 4-bit Qwen3.5 9B checkpoint of about 5.2 GB tops out near 29 tokens per second if every pass produces one token.
- llama.cpp and MLX sit under that line at 22 and 25 tokens per second on the same coding prompt; llama.cpp’s MTP mode drafts only three or four tokens at a time.
- Uzu (Mirai) drafts up to 16 tokens ahead on Apple Silicon and, on the same M5, model, and prompt, wrote 92 to 117 tokens per second.
- Decode at batch size one is memory-bound: each token streams the full weight set; quantization reduces bytes per pass but still leaves a one-token loop.
- Mirai Medium checkpoints use 4-bit asymmetric integer quantization; Large checkpoints use 8-bit symmetric quantization. Labels describe the checkpoint, not a different architecture.
- The M5’s highest arithmetic throughput is in int8; Uzu quantizes activations to 8 bits on the fly so verification can use the hardware-accelerated int8 matrix path.
- A block-diagonal Random Hadamard Transform (blocks of 32, matching Apple GPU SIMD width) mixes outliers before quantization; the transform is exactly reversible apart from rounding.
- Speculative decoding spends idle arithmetic to verify several draft tokens in one main-model pass and, with a correct acceptance procedure, preserves the target model’s output distribution.
- Uzu uses 16-to-32-token budgets on its M5 path; the Qwen3.5 9B checkpoint in the article ships with a 16-token tree budget for M5-series chips.
- Parallel drafting can propose individually plausible tokens that form an invalid sequence; Weaver (a 56.7M-parameter autoregressive adapter in the Trees from Marginals paper) selects from DFlash shortlists so later tokens depend on earlier ones and can keep several coherent paths.
- The paper reports a 24.7 percent improvement over an optimized DFlash baseline and a 4.37-fold speedup over autoregressive decoding on CUDA and SGLang; those are research results, not universal Uzu measurements.
- In Qwen3.5 9B, 24 of 32 layers are Gated DeltaNet (fixed-size recurrent state). Uzu’s rollback-free tree verification computes the whole tree without changing live state, then commits updates only along the accepted path.
- Published Qwen3.6 27B Mirai-M figures (15.6 GB, M5 Max, 128 GB) put Uzu at 2.1× MTPLX, 3.8× llama.cpp, and 4.4× MLX; 35.54 tok/s without speculation and 114 tok/s with it (3.2×).
- A Mirai CLI 0.6.1 run on a base M5 / 16 GB / macOS 26.6.2 reached 98 tok/s with 8.39 tokens per forward pass; across runs, 92–117 tok/s and 7.5–9.6 tokens per pass. Speculation is listed for M5, M5 Pro, and M5 Max; M1–M4 run without a drafter (1.00 t/f).
- The three-engine comparison uses complete paths (Uzu Mirai-M, MLX 4-bit conversion, llama.cpp Q4_K_S), not identical quantization recipes; in the recording Uzu was about 3.7× faster than MLX.
- On a base M5, the combination took a 4-bit 9B model from 22 tok/s with plain llama.cpp to 92 tok/s with Uzu on the same prompt. Speculative speed is a workload property; measure on the application’s prompts.

## Actionables

- Install Uzu via Homebrew, run the Mirai CLI on first use so it downloads the selected model, and read the printed stats (time to first token, prefill speed, generation speed, tokens per forward pass, memory).
- For app integration, start Uzu’s OpenAI-compatible local server and point an existing OpenAI client at `http://127.0.0.1:8000/v1`. Use `--no-prefix-cache` when you need comparable repeated measurements.
- Compare Uzu, MLX LM, and llama.cpp with one client and one server at a time so only one Qwen3.5 9B copy occupies unified memory. Discard the first (warm-up) client run; compare client numbers with each other, not with CLI logs.
- For llama.cpp on a 16 GB Mac, use `-c 8192` and `-np 1`; add `--spec-type draft-mtp --spec-draft-n-max 3` to enable MTP drafting.
- Treat speculative speed as workload-dependent. Replace the sample prompt with requests from the actual product and measure quality alongside latency.
- Start from the Uzu repository for the engine, local server, build instructions, and Rust, Swift, Python, and TypeScript bindings.

## References

### X source

- https://x.com/akshay_pachaar/status/2108252095380140203

### Docs

- https://trymirai.com/blog/quantization
- https://trymirai.com/blog/speculative-decoding-in-uzu
- https://trymirai.com/local-models/alibaba-qwen3-6-27b-mirai-mirai-m-4

### GitHub

- https://github.com/trymirai/uzu

### HF

- https://huggingface.co/trymirai/Qwen3.5-9B-M
- https://huggingface.co/mlx-community/Qwen3.5-9B-4bit
- https://huggingface.co/unsloth/Qwen3.5-9B-MTP-GGUF/blob/main/Qwen3.5-9B-Q4_K_S.gguf

### Other

- https://arxiv.org/abs/2607.06763
