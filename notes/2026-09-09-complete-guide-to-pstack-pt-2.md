# Source

- Author: [@poteto](https://x.com/poteto)
- Article/Post: https://x.com/poteto/status/2097732320606507506
- Date: 2026-09-09
- Title: The Complete Guide to pstack Pt. 2

## Long summary

Part 2 of [@poteto](https://x.com/poteto)’s pstack guide follows Part 1’s focus on verification. Once agents can verify their own work, the next problem is figuring out what to build. The post walks through research, planning, prototyping, and architecture with pstack—the workflow the author uses to ship thousands of PRs a month while keeping quality high.

Frontier models still fail in two related ways: under/poorly specified intent, and not enough context to do the work correctly. Using agents well means priming the context window with high-quality context. Models can write code better than the operator, so the balance is stating the goal while leaving room for solutions the operator would not have thought of—“the art of supervising someone smarter than you.”

One technique is the **indirect prompt**: draw the problem out of the agent in its own words instead of stating assumptions. Example: after a Slack report, ask `/poteto-mode` to read the thread and restate the underlying issue in plain English before coding. That compresses noise into a problem statement, surfaces misunderstandings early, and avoids leading the agent with incorrect hypotheses.

`/teach` helps the agent explain work intuitively (backed by `/how` and `/why`). `/how` traces runtime mechanics and may spawn parallel explorer agents (e.g. on Grok) for multi-directory/service subsystems. `/why` investigates motivation via historical evidence in parallel: Git/PR comments, Linear, Notion, Slack, Datadog, Sentry, code lineage, and analytics. The research helps both humans and agents, who often sound confident without reading enough code or data.

Past transcripts are treated as rich context. `/recall` pulls recent chat history so fresh agents can resume (e.g. recall yesterday’s virtualization work, then read a Slack bug report). Together, `/teach`, `/recall`, `/how`, and `/why` keep human and agent mental models current and compressed.

On “planning,” the author rejects harness plan modes that over-specify implementation and under-specify everything else—cheekily “I don’t believe in planning,” while still planning through code. For shared packages, **readme-driven development** comes first: craft the README/APIs for a hypothetical user, then work backwards. Building Dune (in-house desktop client framework) started with a tutorial; weak agent docs led to `/technical-writing`, which uses the Diátaxis framework (Tutorial, How-to guide, Reference, Explanation) plus `/unslop` for readable docs. A concrete doc target also gives agents something to check work against.

Design-phase skills compound: recall past work, use `/how`/`/why` on current systems, then `/poteto-mode` planning and `/technical-writing` to draft a tutorial for a new approach, then `/teach` and prove superiority—relying on verification skills from Part 1.

Common planning mistakes: accepting the first design, and overcooking plans without evidence. pstack **playbooks** (not skills) are reference files inside `/poteto-mode`, conditionally loaded for token efficiency—23 playbooks as of 0.15.0. The prototyping playbook runs many attempts, useful for UI and for feature/bug-fix options; with `/control-app` it can take screenshots/videos and measure timing/layout. Prototyping is planning with code: agents explore, surprise, and answer their own questions empirically.

For larger design, `/architect` runs phased work: ground with `/how`/`/why`; sketch competing designs in parallel across model families (usage sketch, types, signatures, rationale; screen for red flags and weak-model failure modes); cross-judge with a different model; implement against the sketch and surface discrepancies; scrap and restart if the design is wrong (e.g. repeated workarounds, `any`/forced casts). Prefer this over adversarially reviewing abstract plans, which invent theoretical risks.

There is no planning skill, but a multi-phase planning playbook turns an approved design into a tactical execution plan (`/poteto-mode turn this design into a plan`). Every task is structured around proof: tests alone are not enough—run the code and verify. Structure is checked by an automated script; execution yields small, self-contained PRs. Large week-scale plans may be committed temporarily so other agents see WIP, then deleted; permanent plan files are not valued.

Concrete prompt examples: investigate ambiguous production timeouts with evidence and hypotheses; `/architect` rate limiting for webhooks and prototype open questions before review; plan a StyleX UI migration into small verifiable PRs with visual regression and live verification aiming for 100% identical output including bugs; Slack classics like `/poteto-mode do it` or repro with `/control-app`, fix on main, and show a video. Many skills load automatically under `/poteto-mode`.

Closing argument: abstract Plan Mode often gives an illusion of progress; pstack combines investigation, empirical evidence, and verification so agent engineering becomes predictable. Part 3 is teased.

## Key claims

- After verification (Part 1), the hard problem is deciding what to build; pstack’s research/planning/prototyping/architecture workflow supports high-volume, high-quality shipping.
- Two recurring agent failures: poorly specified intent, and insufficient context—both fixed by priming with high-quality context.
- Indirect prompting (agent restates the problem in its own words) catches misunderstandings early without locking in the operator’s assumptions.
- `/teach` (via `/how` and `/why`), plus `/recall`, keep human and agent mental models accurate; `/why` pulls multi-source historical evidence in parallel.
- Prefer planning through code (readme-driven development, Diátaxis via `/technical-writing` + `/unslop`) over abstract plan modes that over-specify implementation.
- Playbooks (23 as of pstack 0.15.0) live under `/poteto-mode` and load conditionally; prototyping explores options empirically, often with `/control-app` screenshots/videos.
- `/architect` grounds, parallel-sketches across models, cross-judges, implements against the sketch, and scraps wrong designs when empirical proof (workarounds, `any`/casts) shows failure.
- Multi-phase planning playbook turns designs into verification-centered execution plans with small PRs; temporary committed plans for large WIP are deleted when done.
- Abstract Plan Mode often creates an illusion of progress; investigation + prototypes + verification make agent work predictable.

## Actionables

- Before coding from a noisy report (e.g. Slack), ask the agent to restate the underlying issue in plain English and correct misunderstandings first.
- Use `/how` / `/why` / `/teach` to build shared mental models; use `/recall` so new chats inherit prior context instead of rebuilding from scratch.
- For shared APIs/packages: write the README/tutorial first (Diátaxis modes), then implement against that target; use `/unslop`-style constraints for readable docs.
- Prototype multiple options in code (with screenshots/video/metrics when UI) before accepting a design; do not stop at the first agent proposal.
- For larger features: run an architect-style loop (ground → parallel sketches → cross-judge → implement → scrap if wrong) instead of adversarially reviewing long abstract plans.
- Structure execution plans around run-the-code verification (not tests alone) and keep PRs small and self-contained; drop temporary plan files when the work is done.

## References

### X source

- https://x.com/poteto/status/2097732320606507506
- https://x.com/poteto/status/2094457600259842065

### Docs

- https://diataxis.fr/
- https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1#writing-density

### GitHub

- https://github.com/cursor/plugins/blob/main/pstack/README.md#why-are-there-no-planning-skills
- https://github.com/cursor/plugins/blob/main/pstack/skills/architect/SKILL.md
- https://github.com/cursor/plugins/blob/main/pstack/skills/how/SKILL.md
- https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/multi-phase-plan.md
- https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/playbooks/prototype.md
- https://github.com/cursor/plugins/blob/main/pstack/skills/recall/SKILL.md
- https://github.com/cursor/plugins/blob/main/pstack/skills/teach/SKILL.md
- https://github.com/cursor/plugins/blob/main/pstack/skills/technical-writing/SKILL.md
- https://github.com/cursor/plugins/blob/main/pstack/skills/unslop/SKILL.md
- https://github.com/cursor/plugins/blob/main/pstack/skills/why/SKILL.md

### Other

- https://x.ai/bot/plugin/9717366
