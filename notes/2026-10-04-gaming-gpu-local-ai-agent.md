# Source

- Author: [@sudoingX](https://x.com/sudoingX)
- Article/Post: https://x.com/sudoingX/status/2106704758048485654
- Date: 2026-10-04
- Title: Yes! your gaming gpu runs an autonomous local ai agent, here is what to run on 8, 12, 16 and 24gb

## Long summary

Guide to running an autonomous local AI agent on gaming GPUs with 8, 12, 16, and 24 GB of VRAM. Steam’s September hardware survey puts 67% of Steam GPUs between 8 and 16 GB (16 GB most common at 27.21%, 8 GB at 26.71%, 12 GB at 13.06%, 24 GB and up at 7.41%). The single most owned card is the RTX 5070 12 GB at 5.86%.

For 8–16 GB the author tests one file across tiers: bonsai 2 27b, PrismML’s ternary compression of qwen 3.8 27b into 5.95 GB. PrismML’s evaluation puts it at 98.2% of the full model; it keeps qwen’s 262k window and vision. Stock llama.cpp loads it as gibberish, so it needs PrismML’s fork. Out of the box on a 3060 it decoded about 25–26 tok/s; the author’s kernel fix raised that to 40 tok/s, and grafting qwen 3.8’s MTP head reached 50 tok/s. With batch invariance on, output is byte-identical to running without the head. The kernel is pull request #218 on PrismML’s fork; the merged model is on Hugging Face.

VRAM tiers (each tier can run everything that fits smaller cards):

- **8 GB** (RTX 3050, 3060 Ti, 3070, 4060, 5060, 8 GB 4060 Ti and 5060 Ti, and 4060/5060 laptops): bonsai 2 27b with MTP head off and a 96k window. A contributor’s RTX 3060 Ti held 42.8 tok/s out to 96k (full 262k loads in 7.7 GB; speed falls off past about 112k). Vision fits at a 32k window. Prior 8 GB pick was gemma 4 e4b (42.13 tok/s with a 656k window on a GTX 1080 via hermes agent); bonsai 2 is the pick for a 27b on the same card.
- **12 GB** (RTX 3060, 4070, 5070, 3080 12 GB, 4070 Super, 4070 Ti): bonsai 2 27b with the kernel and MTP head, 131k window, vision on top, 11.5 GB. Fresh chat 50.1 tok/s; photo prompt 50.05 tok/s; 40 tok/s with head off. Octopus invaders via hermes agent: five hours, 328k tokens written, eight JS files, 50 tok/s fresh and 22 tok/s average across a 125k-deep session. A 4070 12 GB hit 103 tok/s with the head at a 262k window on PrismML’s integration branch.
- **16 GB** (RTX 5060 Ti 16 GB, 4060 Ti 16 GB, 5070 Ti, 4070 Ti Super, 4080, 5080): bonsai 2 27b with full 262k window, MTP head, and vision in 15.2 of 15.9 GB. On a 5060 Ti: 67.3 tok/s with head, 53.4 without, vs 42.0 on PrismML stock; at 261,000 tokens of context still 12.07 tok/s with head vs 3.73 stock. Four concurrent requests: 124.7 tok/s with head; four agents with 60k context each: ~42 tok/s total (~10–11 each); ~145 W decode, roughly 0.6 kWh per million tokens. Claude Code runs fully local (first turn 24,753 tokens read at 880 tok/s; clean card to first token in 192 seconds and four commands). Stock llama.cpp alternatives on this tier: unsloth’s ud-iq3_s at 12.0 GB and ud-iq4_xs at 14.3 GB of the same qwen 3.8 27b.
- **24 GB** (RTX 3090, 3090 Ti, 4090): qwen 3.8 27b at q4_k_m with the full 262k window in 22.2 GB—no compression needed. On the author’s 3090: 31.0 tok/s, 41.3 with MTP head; qwen38-mtp owners have pushed past 60 tok/s on newer llama.cpp. A contributor’s 3090 ran 51 tok/s at 250 W and 62 tok/s at 275 W with the head. Used 3090 cited as best 24 GB deal at $900–$1,200 locally. For one person on one card the full model is preferred; bonsai 2 still useful for four agents or a giant window with headroom.

Shared settings: fresh llama.cpp build, `-ngl 99`, flash attention on; bonsai needs PrismML’s fork or the author’s prebuilt Hugging Face bundles. Quantize K/V cache (q8_0 cheap; q4_0 how bonsai fits full 262k on 16 GB). One slot per agent; `-np 4` on 16 GB gives four agents 64k each. MTP head at n-max 1 (n-max 2 loses to head-off once context fills). Set reasoning effort to medium—qwen 3.8 / bonsai 2 templates default to xhigh and can spend the output budget thinking with an empty answer.

Catches: 6 GB breaks (only 56 of 64 layers on-card, 4.8 tok/s). MTP head gains shrink deep in the window (e.g. on 5060 Ti: +11.7% at 18k, nothing by 119k, −2.3% at 261k). Numbers are from one card at a time, May–September, on the llama.cpp build of that day.

## Key claims

- About 67% of Steam GPUs have 8–16 GB VRAM; that range is enough to run a real local 27b agent in 2026 without buying a new card.
- bonsai 2 27b is PrismML’s ternary compression of qwen 3.8 27b into 5.95 GB at 98.2% of the full model, keeping the 262k window and vision; it requires PrismML’s llama.cpp fork (stock prints gibberish).
- Author’s kernel fix (PR #218) plus grafting qwen 3.8’s MTP head moved a 3060 from ~25–26 tok/s to 50 tok/s with batch-invariant, byte-identical output.
- 8 GB: bonsai 2 27b, MTP off, 96k window—contributor RTX 3060 Ti held 42.8 tok/s to 96k; vision at 32k.
- 12 GB: same model with kernel + MTP + vision at 131k in 11.5 GB—~50 tok/s fresh on the author’s card; a 4070 hit 103 tok/s at 262k on PrismML’s integration branch.
- 16 GB: full 262k + MTP + vision in 15.2 GB—67.3 tok/s fresh on a 5060 Ti; multi-request and multi-agent serving work on one card; Claude Code runs fully local.
- 24 GB: prefer full qwen 3.8 27b at q4_k_m (22.2 GB, whole 262k); author’s 3090 at 31.0 / 41.3 tok/s without/with MTP; leave power limit alone (250→275 W gave 51→62 tok/s in one report).
- Cross-cutting traps: use PrismML fork/prebuilts for bonsai; quantize K/V for window; MTP n-max 1; set reasoning effort to medium (default xhigh can empty the answer).
- 6 GB is not an agent card; MTP head can hurt at extreme context; all numbers are single-card runs on dated llama.cpp builds.

## Actionables

- Identify your VRAM tier (8 / 12 / 16 / 24 GB) and run the matching recipe above rather than buying a new GPU first.
- For 8–16 GB: pull bonsai 2 27b from the author’s Hugging Face repo (and PrismML fork or prebuilt llama.cpp for RTX 30/40/50); use scripts/sweeps from bonsai2-small-gpu.
- For 24 GB: run qwen 3.8 27b q4_k_m via qwen38-mtp; rebuild llama.cpp before tuning; avoid lowering the power slider.
- Apply shared flags: `-ngl 99`, flash attention on, K/V cache quant (q8_0 / q4_0 as needed), one parallel slot per agent, MTP n-max 1, reasoning effort medium.
- Open a pull request with your card’s measured row so others with the same GPU know what to expect.
- If upgrading later, prefer climbing VRAM tiers; a used 3090 is cited as the best 24 GB deal when bought locally (~$900–$1,200).

## References

### X source

- https://x.com/i/status/2101359610477367794
- https://x.com/i/status/2102829187581317225
- https://x.com/i/status/2104256345759445062
- https://x.com/i/status/2104293129344213170
- https://x.com/i/status/2104601806106136899
- https://x.com/i/status/2104545355585175590
- https://x.com/i/status/2099649647816167789
- https://x.com/i/status/2099855521587335330
- https://x.com/i/status/2104017679908540472
- https://x.com/i/status/2103165265278337390

### GitHub

- https://github.com/sudoingX/bonsai2-small-gpu
- https://github.com/sudoingX/qwen38-mtp
- https://github.com/sudoingX/octopus-invaders
- https://github.com/PrismML-Eng/llama.cpp/pull/218

### HF

- https://huggingface.co/sudoingx/Ternary-Bonsai-2-27B-PTQ1_0-MTP-GGUF
