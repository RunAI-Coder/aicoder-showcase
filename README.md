<p align="center">
  <a href="https://run.ceo/coder/?lang=en"><img src="assets/logo-mark.svg" width="72" alt="RunAI Coder"></a>
</p>

<h1 align="center">RunAI Coder · Showcase</h1>

<p align="center">
  <em>One sentence in, one merged PR out.</em><br>
  Write-ups, with receipts, from the team building <a href="https://run.ceo/coder/?lang=en">RunAI Coder</a> — an autonomous coding agent.
</p>

<p align="center">
  <a href="https://run.ceo/coder/?lang=en"><img src="assets/og-image.png" width="720" alt="RunAI Coder · Your goal. Delivered."></a>
</p>

---

RunAI Coder is an autonomous coding agent: you hand it a goal in natural language, it plans, fans the work out to parallel workers, runs the tests, and files the result as a delivery for your sign-off — with the diff, the test run and the full step-by-step trace attached. The unit of work isn't a suggestion; it's a **merged PR**.

Everything in this repo follows the same rule as the product: **claims ship with receipts.** Posts link to measured data, real runs and reproducible methodology, and we state limitations ourselves rather than waiting for someone else to.

## A real run, unedited

A public SWE task on [pallets/flask](https://github.com/pallets/flask) (CLI feature + tests, all green), recorded from a production session — playback sped up 8×, content untouched. The money-bag indicator at the bottom left is the live compression savings multiple climbing as cached and compressed context accumulates:

<p align="center">
  <img src="assets/demo-run.webp" width="720" alt="Screen recording of a real RunAI Coder run: savings indicator climbs from 2.0x to 5.1x, peaking at 6.1x">
</p>

## Measured, not projected

| Metric | Value | Scope |
|---|---|---|
| Merged squash PRs in 7 days | **968** | 2026-07-14 → 07-21 PDT, one developer, one machine |
| First-pass completion | **98.8%** (1,786 / 1,807 deliveries) | same window |
| Best single day | **98 merged PRs** | 2026-07-21 |
| Context compression | **2.83×** median (p10–p90 2.26–3.08×) | n = 7,222 production requests |
| Effective input price | **≈ $0.26 / M tokens** (≈ 2.6% of list) | measured account, caveats published |

Every number traces back to a machine-recorded ledger or a reproducible script — formulas, sample windows and known limitations are in the [performance report](https://run.ceo/coder/perf?lang=en).

## Posts

| # | Title | Date |
|---|---|---|
| 001 | [One sentence in, one merged PR out](posts/001-one-sentence-one-merged-pr/README.md) — what RunAI Coder is, a week of dogfooding numbers with the caveats attached, and why a merged PR costs $10 | 2026-07-30 |
| 002 | [Our house rules for letting an AI agent commit code](posts/002-house-rules-for-coding-agents/README.md) — five boring-on-purpose guardrails for giving an agent write access, and what they don't solve | 2026-07-31 |
| 005 | [The bill is 99% input tokens: cost engineering for a coding agent](posts/005-cost-engineering-for-coding-agents/README.md) — one ledger day, three levers from $10/M to an effective $0.26/M, and the SWE-bench A/B that keeps it honest | 2026-08-03 |
| 006 | [Why your coding agent forgets: a field guide to context rot](posts/006-why-coding-agents-forget/README.md) — the two-budget model of a context window, what compaction quietly costs, and what helps if you just use these tools | 2026-08-04 |
| 007 | [33k tokens before hello: the anatomy of your coding agent's preamble](posts/007-agent-preamble-anatomy/README.md) — what the viral token-overhead study really found: configuration swamps defaults, cache churn beats size, and a two-minute way to measure your own | 2026-08-05 |
| 008 | [Nobody reads your architecture overview. Including the agent.](posts/008-agent-friendly-codebases/README.md) — why an instruction file is rent, not documentation: three conflicting evals reconciled, a five-lane probe on Flask, and the harness that never read our AGENTS.md at all | 2026-08-06 |
| 009 | [The human in the loop approves one in three attacks](posts/009-approval-fatigue/README.md) — 409,000 approve/deny decisions say the approval prompt asks the wrong question: where the misses concentrate, why vigilance doesn't train, and where human attention actually buys safety | 2026-08-07 |
| 010 | [The blast radius of a one-line fix](posts/010-blast-radius/README.md) — a 16-year-old Flask bug, three prompt styles, nine clean-room runs: what actually sets how much a coding agent touches, and the vague run that audited our whole lab | 2026-08-10 |
| 011 | [The tests your agent writes defend the code it saw. Bugs included.](posts/011-agent-written-tests/README.md) — eleven agent-written test suites versus seven planted faults: fix-time tests catch everything, cold tests mirror whatever they met, and two of three suites written against buggy code would block the real fix in CI | 2026-08-11 |
| 012 | [Why your coding agent spawns clones of itself](posts/012-why-agents-spawn-clones/README.md) — subagents as disposable context windows: why read-heavy work fans out and parallel writes collide, the 4×/15× token bill, and where the 2025 multi-agent argument landed by mid-2026 | 2026-08-11 |
| 013 | [A million tokens of context, and the agent still greps](posts/013-million-tokens-still-greps/README.md) — the long-context paradox priced out: per-turn re-billing arithmetic, the advertised-vs-effective gap in the degradation literature, why agentic retrieval survived the 1M era, and when whole-corpus pasting is actually right | 2026-08-12 |
| 014 | [Agents code best where the internet coded most](posts/014-agents-home-field/README.md) — the competence cliff on niche stacks: training-data frequency as the hidden skill map, the version-boundary problem, what actually pulls the floor up, and whether agent-friendliness should pick your stack | 2026-08-13 |
| 015 | [The harness is half your agent](posts/015-harness-half-your-agent/README.md) — anatomy of the loop between you and the model: what the harness assembles, executes, retries, and stops; why identical weights land different scores under different scaffolds; and the two version numbers worth logging | 2026-08-14 |
| 016 | [Every turn, the harness decides what the model sees](posts/016-what-the-model-sees/README.md) — the per-turn compile of your agent's context: a stateless API underneath, what clearing and compaction quietly remove, why the stack order follows the caching hierarchy, and the reflexes that keep constraints alive | 2026-08-17 |
| 017 | [Your coding agent's safety net was written in 2005](posts/017-git-safety-net/README.md) — the quiet risk layer under every agent session: branch-and-diff economics, the reflog's 90-day memory, worktrees as the parallel-agent structure, the five-step surgery it takes to truly lose a commit, and where the net has holes | 2026-08-18 |
| 018 | [Agents debug as well as your error messages let them](posts/018-errors-are-prompts/README.md) — error messages as the agent's primary sensor: the four properties of a machine-legible failure, why a vague raise multiplies your bill, what compiler teams figured out early, and the training-data caveat on custom formats | 2026-08-19 |
| 019 | [AI one-shots tiny games because you are the test suite](posts/019-you-are-the-test-suite/README.md) — why the one-sentence game demo lands: the corpus rehearsed it, the task has edges, and verification collapses into playing; the rule worth stealing (put the acceptance where your senses are) and where the trick stops working | 2026-08-20 |
| 020 | [Don't take your subagent's word for it](posts/020-subagent-reports/README.md) — a completion report is a summary written by the thing being evaluated: the mechanics of default-only-summary delegation, three structural distortions, and the dispatch habits (verbatim quotes, artifacts over adjectives, blast-radius spot-checks) that make reports checkable | 2026-08-23 |
| 021 | [The tools your agent never called are still on the bill](posts/021-tool-catalog-tax/README.md) — the tool catalog re-enters context on every request: the hidden per-model tool-use system prompt, why touching one tool definition invalidates the whole cache, the 30-50 tool accuracy cliff, and how deferred loading (tool search) changes the economics | 2026-08-25 |
| 022 | [A second model pays off in two shapes](posts/022-second-model-org-charts/README.md) — Anthropic's cost guide measured exactly two org charts where model mixing earns its keep: the advisor (low-cost executor escalates hard decisions) and the orchestrator (frontier chair delegates bulk work); the consult-rate failure mode, the noise at the top of the menu, and why caching is still the bigger lever | 2026-08-27 |
| 023 | [The productivity is real. It's parked in front of the merge button.](posts/023-review-bottleneck/README.md) — review time up 5x while no-review merges rose 31%: the arithmetic of near-zero-cost generation meeting human-speed acceptance, the METR perception gap, and why every fix is either compressing acceptance or leveraging it | 2026-08-29 |
| 024 | [Your one-line prompt ships with a spec you didn't write](posts/024-spec-you-didnt-write/README.md) — every dispatch has a spec; a one-liner just delegates it to training-corpus defaults: the two dials that predict when that's safe (how long a wrong guess stays invisible, distance from the average project), a five-stop thickness menu, and why our dispatches settled in the middle | 2026-08-28 |
| 025 | [Why your agent apologizes and then does it again](posts/025-doom-loop/README.md) — the doom loop is structural: a stateless model re-reads a transcript where the failed attempt is the dominant pattern, so history paves the bad path instead of fencing it off; the token-level vs plan-level repetition split, trajectory evidence (right file 72-81% of the time, wrong hypothesis held anyway), and three circuit breakers | 2026-08-31 |
| 026 | [What survives the summary](posts/026-what-survives-the-summary/README.md) — mid-session, compaction has a model rewrite your conversation history: what the losses consistently are (conclusions outlive reasoning, decisions outlive their conditions, firm statements outlive hedged ones), why the post-compaction agent feels different, and a three-layer placement rule for information that must not degrade | 2026-09-01 |
| 027 | [Rerun it before you believe it](posts/027-rerun-before-you-believe/README.md) — a failing test has two possible senders, the code or everything else in the room, and the quickest way to tell them apart is a rerun the agent rarely makes on its own: what flakiness is made of (45% async waits in a 2014 Apache study; 24% of the fixes changed the code under test and 94% of those fixed a real bug), the three exits a stuck agent takes and why the skip is the quiet one, and the harness-side rule that hands it the instrument | 2026-09-02 |
| 028 | [When the skill outranks the task](posts/028-when-the-skill-outranks-the-task/README.md) — an agent skill is a procedure gated by one paragraph and then resident in context with the standing of an instruction: why the gate fails closed (a 56% trigger rate in one eval), why the harm comes from on-topic skills displacing task requirements rather than wrong skills firing (86 of 125 functional failures in an August study), why procedure costs more than length (114 of 182 efficiency regressions), and how to read a skill before it reads your task | 2026-09-04 |
| 029 | [To the model, a tool is its return value](posts/029-a-tool-is-its-return-value/README.md) — the model never sees a tool's backend, only its description going in and its result coming back, so a precise tool that returns an address loses to grep returning the line: a pilot where attaching ±2 lines of source to the same LSP references lifted pass@1 from 0.67 to 0.83 and cut follow-up reads from 15.2 to 3.2, where the language server still earns its place (noisy repos, find-every-caller tasks), and why a routing rule in the rules file stayed advisory until it became a PreToolUse hook | 2026-09-07 |
| 030 | [The sandbox never reads the command](posts/030-the-sandbox-never-reads-the-command/README.md) — a coding agent's "no" can live in three places: the text the model reads, the harness matching the tool call as text before it runs, and the OS sandbox watching the process; why a Read(./.env) deny rule never meets a Python script that opens .env, how the documented "escape hatch" out of the sandbox is a permission prompt, what the o1 system card's Docker story shows when read as mechanism, and which rule belongs at which gate | 2026-09-08 |
| 031 | [The fast turns are the guessable ones](posts/031-the-fast-turns-are-the-guessable-ones/README.md) — where the seconds go in one coding-agent turn: the pause before the first character (a long prefix read from the top unless the cache holds), one forward pass per output token and the paid fast lanes that speed up only that part, why speculative decoding makes the same model type faster on guessable text (first-draft-token acceptance 94% on math vs 89% on Python code, 66% vs 49% by the fifth, from a vLLM-blog study), and the waits with no model in them | 2026-09-08 |
| 032 | [Answer-first needs somewhere else to put the thinking](posts/032-answer-first-needs-somewhere-to-think/README.md) — why a coding agent buries the answer and when a "lead with the answer" rule costs the answer: reasoning written in text has to precede the conclusion (in the 2022 chain-of-thought ablation, reasoning after the answer scored like no reasoning), padding is RLHF length residue and free to cut, hidden thinking moves the cost off the page, and where a format rule lives (skill body, system prompt, schema field order) decides whether it survives turn forty | 2026-09-09 |
| 033 | [Naming a test technique buys you its motions](posts/033-naming-a-test-technique-buys-its-motions/README.md) — what a coding agent does with a named testing technique: in Dan Luu's 160-runs-per-condition experiment, naming QuickCheck, fuzzing, TLA+, differential testing or TDD changed the shape of the tests and left the check where it was, because every technique's value is an oracle that did not come from the code under test, and a name does not supply one | 2026-09-10 |
| 034 | [The reasoning you pay for and the reasoning you can read](posts/034-the-reasoning-you-pay-for/README.md) — three places a model's reasoning can live and what each does to cost, steering and readability: visible text you can read and steer; hidden tokens you are charged for and shown only as a count and a summary, which on newer models stay in context as input; and recurrent depth, extra passes through the same layers that produce no tokens and no field at all, read against the Astra reports, the system card and both vendors' docs | 2026-09-11 |

## Get it

Terminal apps for **Windows** (10/11 x64), **macOS** (Apple Silicon) and **Linux** (x64), CLI included — an **Android** remote-control app is rolling out, iOS next. Free starter pack: US$25–30 of value, one-time, 14 days, no card required.

→ [run.ceo/coder](https://run.ceo/coder/?lang=en#download)

## License

Text and images in this repository: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) unless noted otherwise. Product screenshots, recordings and brand assets belong to RunAI Coder.
