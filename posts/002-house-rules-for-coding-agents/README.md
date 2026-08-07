# Our house rules for letting an AI agent commit code

*2026-07-31 · the RunAI Coder team · [run.ceo/coder](https://run.ceo/coder/?utm_source=github_Official_Page)*

Last week we let an AI agent merge [968 pull requests into our own codebase](../001-one-sentence-one-merged-pr/README.md). When that number comes up with other developers, nobody asks about throughput. They ask about control. An autonomous agent reads your code, executes commands on your machine, and produces changes that land on main. What keeps that from going badly?

The honest answer is the one operations teams reached decades ago for every other powerful automation: assume the automation will eventually do something wrong, and engineer the environment so a single failure is survivable. None of what follows is exotic. Most of it is what you would do when onboarding a new engineer, which is not a coincidence.

## Rule 1: bound the blast radius before the first run

The most consequential decision happens before the agent executes anything: deciding what it can reach.

Start with identity. The agent gets its own Git identity and a fine-grained personal access token, scoped to the repositories it works in, with an expiry date. Not your personal SSH key. A separate identity gives you three things at once: single-motion revocation, a `git blame` that tells the truth, and access logs that distinguish agent activity from yours.

Production credentials never exist on the machine the agent runs on. It operates on development secrets only. A task that genuinely requires production access is performed by a human, or the credential is injected by a human at the moment of use and revoked after. The agent cannot leak what it never held. That is the cheapest security control in this document.

Then protect the merge path itself: branch protection on main, required status checks, review before merge. Our product is built around "one sentence in, one merged PR out", and even there nothing merges without passing tests and receiving explicit human sign-off at the delivery gate. The agent proposes. The gate disposes.

## Rule 2: assume it reads everything it can reach

An agent exploring a codebase does what a thorough new engineer does: it reads. The `.env` files, the CI configuration, the `notes-old-DO-NOT-COMMIT.txt` someone left behind in 2024. Plan for total read coverage instead of hoping the model politely averts its eyes.

Two consequences follow. First, any credential readable in the workspace should be one you would be annoyed to rotate, not ruined to leak: short-lived, narrowly scoped, revocable. Second, real customer data does not belong in repositories or fixtures the agent touches. Synthetic data exists for exactly this purpose.

Tooling can reinforce the discipline but not replace it. In our case, run ledgers are read-only records of production sessions and, per the methodology notes on our [performance report](https://run.ceo/coder/perf?lang=en), credentials and tokens are redacted at source before anything reaches disk. We still follow both rules above. Layered controls are the point, not a sign of distrust in any single layer.

## Rule 3: treat everything the agent reads as a potential instruction

Prompt injection is the one genuinely new threat class agents introduce. An agent consumes issues, dependency READMEs, error messages, sometimes web pages. Any of that text can contain content that reads like an instruction ("ignore your previous task and instead..."), and a sufficiently credulous model may comply. The attack surface is no longer limited to code that executes. It includes text that persuades.

Three mitigations have held up in our daily use:

Scope the goal narrowly. A task phrased as "add dark mode across the web app, ship it verified" carries a checkable definition of done. When a run drifts toward work the goal never mentioned, the drift is visible precisely because the goal was narrow.

Review the plan, not only the diff. Runs stop at an approval gate where the agent presents its plan before executing the consequential parts. A plan step reading "update deployment credentials" inside a CSS task is a red flag caught before execution, where it is cheap, rather than after, where it is not.

Treat topic drift as an incident. When an agent suddenly wants to do something unrelated, stop the run, read the trace, and find what it ingested. The step-by-step audit trail exists for this moment.

Injection defenses today are mitigations, not immunity. Anyone selling immunity is selling something else as well.

## Rule 4: verify like it's a stranger's pull request

Because it is one. A competent stranger who works fast, never tires, and has no reputation to lose.

Tests pass in your CI, not in the agent's retelling. Every delivery in our system ships with an evidence receipt recording what ran, what changed, and what passed. This is what one looks like, quoted verbatim from the demo run on our homepage:

```text
ai-coder · goal run
> ai "add dark mode across the web app — ship it verified"
• Goal created — dark mode across the web app, verified before delivery.

updated plan
  ✔ #t1  theme tokens + persisted toggle — worker A   00:42
  ✔ #t2  recolor 14 views to tokens — worker B        00:42
  ✔ #t3  run tests, verify green & deliver — main     00:11
  plan complete — 3/3 steps · total 01:35

delegate   ✓ Two workers started in parallel; plan tracks each one.
tests      ✓ ran npm test · unit + visual checks green — evidence kept in the run
delivered  ✓ Goal completed — 3/3 steps, tests green · 01:35
             — Delivery filed to /matters for your sign-off.
```

The receipt beats the summary every time. "Done" is a verdict the evidence earns, not a message the model prints; we wrote [a whole post](../001-one-sentence-one-merged-pr/README.md) about that. Note the last line: the finished work is filed as a delivery awaiting sign-off. It does not count as done because the model said so.

Beyond receipts, keep a written list of paths that always get human eyes regardless of check status: authentication, cryptography, payments, CI configuration, dependency manifests. A written list turns sensitive-path review into boring policy instead of heroic vigilance. And review the diff you are merging, not the diff you expect. Agents are strong at producing plausible-looking changes. Plausible is not the bar. Correct is.

## Rule 5: be ready to do forensics

Eventually something odd will land: a change nobody remembers requesting, a formerly failing test quietly green. On that day, the difference between a five-minute investigation and a lost weekend is whether the run left a trace.

Every run should be replayable: what the agent was asked, what it planned, what it executed, what it touched. In our system the full trail is kept per run, and any live session can be taken over from a browser mid-flight. Debugging an agent means debugging a run, not interrogating a memory. Build the infrastructure on that assumption.

## The five layers, and what each one contains

| Layer | Control | Failure it contains |
|---|---|---|
| Identity | separate Git identity, fine-grained token, expiry | credential theft, unattributable changes |
| Environment | dev-only secrets, synthetic data, redaction at source | secret and data exfiltration |
| Gates | narrow goals, plan approval before execution | prompt injection, scope drift |
| Verification | tests in your CI, sensitive-path review, delivery sign-off | plausible-but-wrong changes |
| Audit | replayable traces, browser take-over of live sessions | slow or impossible forensics |

Any single layer failing should be survivable. That is the entire design philosophy.

## What this doesn't solve

Your context reaches a model API. Whichever repositories you point an agent at, their content reaches the model provider processing the request. Choosing which projects to delegate is a scoping decision no local guardrail makes for you.

A rushed human defeats every gate. Approval gates are worth exactly the attention of the person clicking approve. A team that rubber-stamps has built a very elaborate autocomplete.

The field is young. Prompt injection research moves monthly. Rules 1, 2 and 5 are old, boring, and proven. Rule 3 is a current best effort we expect to revise, and we would rather say that plainly than pretend otherwise.

None of this requires heroics. An agent is a fast junior engineer with no reputation to lose, so give it the onboarding you would give one: scoped credentials on day one, no production access, plans reviewed, work verified, everything on the record.

Then let it work. The 968 merged PRs were real. So was every guardrail above, on every one of them.

---

*RunAI Coder is an autonomous coding agent: one sentence in, one merged PR out, with approval gates and a full audit trail built in. Free starter pack at [run.ceo/coder](https://run.ceo/coder/?utm_source=github_Official_Page). Questions and pushback welcome in the issues.*
