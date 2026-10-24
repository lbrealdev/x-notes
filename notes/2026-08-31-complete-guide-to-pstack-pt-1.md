# Source

- Author: [@poteto](https://x.com/poteto)
- Article/Post: https://x.com/poteto/status/2094457600259842065
- Date: 2026-08-31
- Title: The Complete Guide to pstack Pt. 1

## Long summary

Part 1 of [@poteto](https://x.com/poteto)’s pstack guide introduces pstack—the author’s personal set of skills for rigorous engineering work—and centers on verification as the foundation. The author reports shipping 2,000 PRs a month to production with high confidence using pstack. Before agents, PR/LOC volume felt like a vanity metric; with agents that hold or raise quality, volume matters. On Grok @Bot (started ~2 months earlier, while the codebase was still young), pstack let the author keep quality high amid hundreds of PRs a day—monitoring code, refactoring, adding lints/checks, and shipping features—acting as gardener/maintainer and refactoring foundations mid-growth with no downtime so the codebase scales for engineers and non-engineers.

**Part 1 – Verification is all you need.** Verification (an agent verifying its own work and closing the loop without a human bottleneck) is treated as critical infrastructure, not “just” a skill; done well it can 100–1000x team output, including non-engineers. Background on the author’s first Cursor verification skill is in a prior post, *Loops You Can Trust*.

To build one: install pstack, run `/create-verification-skill`, and add Dr Eggbot (ships with pstack) to help create high-quality bots—including engineer bots that run `/create-verification-skill` and a daily `/maintain-verification-skill` routine. The create skill is a meta-skill distilled from verification skills used on Grok @Bot and Cursor. Tech stack choice matters: rich debug surfaces (e.g. Chrome DevTools Protocol for Electron/web, iOS simulator) let agents interact, debug, and take perf traces; otherwise agents may need custom tools (e.g. lldb, sidecars). The author argues agentic verification is important enough to justify building rich debug tools or even changing stacks—hard-to-debug stacks limit productive agent use.

**Make it Reproducible.** pstack’s “Build the Lever” principle prefers giving agents tools over markdown alone. For verification, that means a small agent-friendly CLI that scripts app interaction/debugging: fewer tokens than throwaway scripts, and more reproducible/testable skills. Also plan seeding a dev database, auth/test users, staging APIs, and consistent env bring-up. Agent-friendly CLI properties cited: composable APIs (John Ousterhout’s deep modules), `--dry-run` for destructive commands, subcommands for gradual disclosure, descriptive errors that say what to do instead, rich `--help`, and machine-readable output (e.g. JSON). Invest in a solid CLI before advanced work.

**Go faster with parallelism with Cloud Agents instead of worktrees.** After verification lands a few PRs, parallelization tempts people toward git worktrees; the author recommends against them (storage/resource cost; maybe ~10 agents on a strong machine). Prefer Cursor cloud agents on real cloud machines that can install deps, run the app, capture video/screenshots, and interact like a user. A good local dev experience transfers; first cloud setup is agent-assisted, then a snapshot makes later runs fast. Cloud parallelism unlocks large scale (hundreds of subagents teased for a later post).

**Keep agents smart with Feature Maps.** A Feature Map is a searchable catalog of app features, what they do, and how users reach them—mentioned from the verification `SKILL.md`, often under `references/features` with a README map plus detail pages (example: fictional Atlas app). `/create-verification-skill` auto-catalogs the app; with the CLI, this is a main reason pstack verification works well—compact “materialized memory” that saves tokens and is shared via the codebase. Run `/maintain-verification-skill` at least daily; agents may also update maps while working.

**How to use your verification skill.** Example skill: https://github.com/poteto/verification-skill-example (basic CLI + Feature Map from `/create-verification-skill`). Typical flow: start with `/poteto-mode` (in Cursor, Opt + Enter on autocomplete pins it as a Custom Mode so each turn reminds the agent; in Grok @Bot, install the plugin then `/poteto-mode`). New features: `/poteto-mode` plus `/control-app` (the created verification skill) for video/screenshot proof; in Grok @Bot, prefer spawning a cloud agent so the bot stays a coordinator with a clean context and free local machine. Perf: baseline with `/control-app`, fix, then confirm with `/control-app` + `/swarm` (fans out cloud agents for sample size / fuzz / regression). Mature skills can sit in Grok @Bot routines or Cursor Automations—e.g. Slack feedback that auto-reproduces (and optionally auto-fixes) via cloud agents.

**Invest in your verification skill.** Keep it sharp with `/maintain-verification-skill`; treat CLI and skill like critical infra (even oncall). It underpins later pstack skills. Links given: pstack `https://x.ai/bot/plugin/9717366` and Dr Eggbot `https://x.ai/bot/93gOz3op1UQdBdbekQFLK`. Part 2 is teased.

## Key claims

- pstack enabled ~2,000 production PRs/month with high confidence; with agents that preserve quality, volume matters (vs pre-agent vanity metrics).
- On Grok @Bot, pstack supported high-quality gardening/refactoring under hundreds of daily PRs and growing (engineer and non-engineer) contributors.
- High-quality verification is critical infrastructure: agents close their own loop; done well it can 100–1000x team output.
- Build via `/create-verification-skill` (and Dr Eggbot / daily `/maintain-verification-skill`); the skill is a meta-skill from Grok @Bot and Cursor practice.
- Prefer stacks and tooling agents can drive (CDP, simulators, custom CLIs); “Build the Lever” favors agent-friendly CLIs over markdown-only skills.
- Prefer Cursor cloud agents for parallelism over local git worktrees; snapshots make subsequent runs fast.
- Feature Maps plus the CLI give agents compact shared memory of features and navigation; maintain at least daily.
- Compose with `/poteto-mode`, `/control-app`, `/swarm`, and routines/automations (e.g. auto-repro of user reports); keep investing in the skill like critical infra.

## Actionables

- Install pstack, run `/create-verification-skill`, and add Dr Eggbot; set a daily `/maintain-verification-skill` routine.
- Prefer giving agents a small, tested CLI for app control/debug (dry-run, subcommands, rich help, JSON) over one-off scripts; invest in reproducible local/cloud dev setup (seed DB, test auth, staging APIs).
- Prefer Cursor cloud agents for parallel work instead of scaling local worktrees; set up the cloud environment and snapshot early.
- Let `/create-verification-skill` generate a Feature Map; keep it current with `/maintain-verification-skill` at least once a day.
- Use `/poteto-mode` with `/control-app` for feature and perf work (video/screenshot proof); use `/swarm` to confirm wins or fuzz; wire mature verification into routines/automations for auto-repro of feedback.

## References

### X source

- https://x.com/poteto/status/2094457600259842065
- https://x.com/poteto/status/2069824386283319343
- https://x.com/poteto/status/2097732320606507506

### Docs

- https://chromedevtools.github.io/devtools-protocol/
- https://cursor.com/changelog/08-19-26#custom-modes
- https://cursor.com/docs/cloud-agent
- https://cursor.com/docs/cloud-agent/automations
- https://cursor.com/docs/cloud-agent/builds

### GitHub

- https://github.com/cursor/plugins/tree/main/pstack
- https://github.com/cursor/plugins/blob/main/pstack/skills/create-verification-skill/SKILL.md
- https://github.com/cursor/plugins/blob/main/pstack/skills/principle-build-the-lever/SKILL.md
- https://github.com/cursor/plugins/blob/main/pstack/skills/swarm/SKILL.md
- https://github.com/poteto/verification-skill-example
- https://github.com/poteto/verification-skill-example/blob/main/.cursor/skills/verify-atlas/references/features/README.md

### Other

- https://x.ai/bot/plugin/9717366
- https://x.ai/bot/93gOz3op1UQdBdbekQFLK
