# Source

- Author: [@iamMrDuncan](https://x.com/iamMrDuncan)
- Article/Post: https://x.com/i/article/2109718238506696704
- Status: https://x.com/iamMrDuncan/status/2109744566001938923
- Date: 2026-10-12
- Title: How I run my Lab

## Long summary

[@iamMrDuncan](https://x.com/iamMrDuncan) describes how a personal research lab is organized: a mixed cluster from commodity hardware up through Blackwell systems, set up so experiments and research run with secure isolation. The write-up covers five areas: The Lab, The Enclave, The AutoResearch, Jilly, and Frontier Research.

**The Lab** hardware at a high level includes 3× DGX Sparks; GPUs (4× P100, 4× V100, 1× Radeaon VII, 1× BC-160, 1× Intel b70); a GPU server (P100/V100) with 128 GB RAM and 16 TB RAID5 disk; a virtualization server (80 cores, 512 GB RAM, 16 TB RAID5 disk); 4× mini PCs (Enclave, Compute Node, IoT Controller, GPU Swapper Bench); a microcontroller bench PC; an old gaming PC for swapping and testing GPUs; plus 2 routers, 2 10 Gbps switches, and battery backups.

**The Enclave** is the command-and-control “brain” of the cluster. It is hidden from the rest of the lab behind a Slate 7 gl.inet router: the Enclave can reach everything, but nothing else can reach it. Specs cited: 128 GB RAM, an Intel ultra CPU, mirrored 4 TB NVMe, two external stores (5 TB and 2 TB NVMe), and an Intel Arc b70. It runs the agent orchestration factory and knowledge base; the GPU is used for embeddings and Clef. Clef (via [@cloudflare](https://x.com/cloudflare)) runs guardrails, permissions checks, and prompt-injection protections on every Enclave request, command, and KB placement decision. Embeddings use the Gemma Embedding 2 model. Models and Podman images live on the external 2 TB NVMe so the mirrored 4 TB NVMe holds the knowledge base. Everything in the Enclave is real-time synced to Cloudflare R2 as offsite backup. The host runs Proxmox and splits into two isolated VMs: Agent Host and KB Host.

Work starts in the Enclave and spreads out. Hermes is the agent harness there, with multiple profiles fine-tuned for workloads, backed by the KB and shared team memory. The author also uses bot mode in Hermes desktop and the Hermex app for multi-agent collaboration. For under-the-hood access, Herdr is set up across the lab; bots (especially “ledger,” which manages AutoResearch agents) can use Herdr to manage agent execution. For Claude Artifacts / ChatGPT-style publishable apps, Coolify runs on the big virtualization server so the Enclave can build and publish apps/sites/pages at `apps.<tailscale-address>.net/<appid>` for private access (progress dashboards and one-off visualizations).

**The AutoResearch** focuses on older cards for local AI at e-waste cost. Six GPUs run active auto research at a time (4× P100s, 1× Radeon VII 7, 1× BC-160). The custom harness is Thermite (planned open source), refining student + teacher + valley-breaking hillclimbs. Earlier versions used pi-autoresearch; the current system spins up a container on the GPU clusters, attaches a GPU, starts an experiment, and the “ledger” bot runs periodic checks and steering. Results go to Prometheus and are viewed in Grafana or custom Hermes desktop dashboards. An embedded update reports Qwen 3.8 27b at about 33 tok/s with MTP on a P100 (~$100 GPU), then flipping the hillclimb to hold that decode rate while increasing prefill speed.

**Jilly** is a free coding-agent service on the Hackers in the Loop Discord, powered by deepseek flash v4.1, with an agent connect setup on an isolated VLAN and sandboxes. Tag her in a thread; she stays free until tokens run out.

**Frontier Research** uses the DGX Sparks (modern Blackwell architecture and current CUDA) as a test bed for ideas that may scale to larger hardware such as B200/B300. Those systems are the most locked down—reachable only from the Enclave or specific machines (personal Mac and other researchers’ devices). High-level topics: latent space thinking, KV Cache improvements, and agent swarm vs large models.

## Key claims

- The lab is a mixed compute cluster from commodity hardware through state-of-the-art Blackwell systems, organized so experiments run with secure setups once configured.
- The Enclave is the isolated command-and-control center (behind a Slate 7 gl.inet router): it can see the lab, but the lab cannot see it.
- Enclave security and orchestration: Proxmox splits Agent Host and KB Host; Clef (Cloudflare) guards requests/commands/KB decisions; Gemma Embedding 2 for embeddings; Cloudflare R2 for real-time offsite backup.
- Hermes (with desktop bot mode / Hermex) is the agent harness; Herdr provides under-the-hood access across the lab; Coolify on the virtualization server publishes private app UIs via Tailscale.
- AutoResearch runs on six older GPUs (4× P100, 1× Radeon VII 7, 1× BC-160) via the custom Thermite harness, with the “ledger” bot steering experiments and Prometheus/Grafana for campaign status.
- An embedded result: Qwen 3.8 27b reached about 33 tok/s with MTP on a P100 (~$100 GPU); follow-on hillclimb aims to keep that and raise prefill speed.
- Jilly is a free Discord coding agent (Hackers in the Loop) on deepseek flash v4.1 with isolated VLAN/sandboxes, free until tokens run out.
- Frontier work on DGX Sparks (locked down) explores latent space thinking, KV Cache improvements, and agent swarm vs large models, as a path toward B200/B300-scale ideas.

## Actionables

- If building a personal research cluster: separate a command-and-control host (Enclave-style) from compute nodes with one-way reachability and strict VM isolation for agents vs knowledge base.
- Put guardrails (permissions, prompt-injection checks) on every agent/KB action; keep models/images on separate storage from the KB; sync critical state offsite (e.g. object storage).
- For older-GPU local-AI research: run containerized experiments with attached GPUs, automated steering checks, and metrics dashboards (Prometheus/Grafana or similar).
- For private agent-built UIs: host a self-serve app publisher (e.g. Coolify) reachable only over a private network (e.g. Tailscale).
- Watch for Thermite’s planned open-source release if you care about student + teacher + valley-breaking hillclimb harnesses.

## References

### X source

- https://x.com/iamMrDuncan/status/2109744566001938923
- https://x.com/iamMrDuncan/status/2107109051301568582
- https://x.com/iamMrDuncan
- https://x.com/cloudflare
