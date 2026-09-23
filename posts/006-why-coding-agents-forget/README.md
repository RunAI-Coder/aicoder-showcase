# Why your coding agent forgets: a field guide to context rot

*2026-08-04 · the RunAI Coder team · [run.ceo/coder](https://run.ceo/coder/?utm_source=github_Official_Page)*

Two hours into a refactor, the agent writes a helper function. A nice one. The problem is that it wrote the same helper ninety minutes ago, in a file it created, which it has since re-read twice. It apologizes, of course. It always apologizes.

If you have run long agent sessions, you have met some version of this. The constraint you stated early ("keep the public API frozen") quietly stops applying. The agent re-reads the same file for the ninth time. It asks a question you answered an hour ago. The tempting diagnosis is that the model ran out of context window. The uncomfortable one: the session was at 120K tokens of a 200K window, and the model had already been degrading for a while. Nothing overflowed.

The context didn't run out. It rotted.

## One window, two budgets

The mental model that fixed this for me: a context window is not one budget but two. The hard budget is the advertised token limit, and when you cross it, something explicit happens: an error, a truncation, a forced summarization. You notice. The soft budget is attention quality, and it depletes silently, well before the hard one.

The soft budget is measurable. In 2025, Chroma ran the systematic version of the experiment across 18 frontier models, including the GPT-4.1, Claude 4, Gemini 2.5 and Qwen3 families: performance declines as input grows, on every model tested, even when the task itself stays fixed, and the decline often starts far below the advertised limit. Harder tasks degrade faster. This is on top of the older finding from Liu et al. that accuracy depends on where in the context a fact sits: high near the start and the end, sagging in the middle, the "lost in the middle" curve.

Neither result says long windows are useless. They say the marginal token is not free even when it is paid for. Every token you add competes with every token already there; the measured effect is as if the model had a roughly fixed budget of attention to spread.

Which is why a 1M-token window doesn't dissolve the problem. It moves the cliff and charges you for the extra runway.

## Why coding agents hit this hardest

An agent session has terrible signal decay. Turn by turn it accumulates full file dumps, test output, stack traces, directory listings, diffs. Almost all of it was load-bearing for exactly one turn. Then it becomes sediment: the 4,000-line file the agent read to change six lines, still sitting in context an hour later, still competing for attention with the thing you actually care about.

Coding tasks also concentrate the failure. They are long-horizon (a real bug fix commonly spans dozens or hundreds of turns), they are stateful (that constraint from turn 3 still binds at turn 140), and Chroma's results show harder tasks degrading fastest, which bodes poorly for work as demanding as coding. A chat that rots gets rambly. A coding session that rots writes plausible code against a spec it no longer remembers.

## The toolbox, from gentle to drastic

The engineering answer has a name that went mainstream in mid-2025: context engineering, the discipline of deciding what deserves to be in the window at each moment, rather than what to say in the prompt. As of August 2026 the working toolbox looks like this, roughly in escalating order.

The gentlest lever is not ingesting junk in the first place. Read the 40 relevant lines instead of the whole file; trim tool output to what the next decision needs; return conclusions instead of transcripts. Unglamorous, and it buys more than anything else on this list because it delays every downstream problem.

Next is eviction: deleting stale material from the live context, typically old tool results that already served their purpose. This is now a platform feature rather than a hack. Anthropic's context editing reports a 29% improvement on an internal agentic-search benchmark from automatic tool-result clearing alone, and an 84% reduction in tokens consumed on a 100-turn task that otherwise fails by exhaustion. Internal benchmark, vendor numbers, usual caveats; the direction matches what practitioners see.

Then comes compaction: summarize the session so far, restart from the summary. This is the workhorse and the most dangerous tool on the shelf, for the same reason: a summary is a model's opinion about what mattered. A good compaction keeps open goals, decisions with their reasons, invariants and constraints, file paths touched, and what failed and why. It drops raw logs, dead-end details, and file contents that can be re-read on demand. Timing matters more than people expect: compact at a chosen phase boundary, where you can verify the handoff, rather than at the cliff edge where the compactor inherits a context that is already rotten.

Beyond the window sits external memory: notes the agent writes to files and re-reads later, task lists, scratchpads. In the same Anthropic evaluation, memory plus context editing reached 39% over baseline. The principle is older than any of this tooling: writing things down beats remembering them, for machines as much as for people.

And structurally, you can stop sharing one window at all: fan scoped subtasks out to sub-agents with fresh contexts that return short conclusions. The orchestrator's context stays small; each worker rots alone, briefly, and is discarded before it matters.

## What compaction quietly costs

Every summary is lossy, and the loss is adversarial in a specific way: the detail you lose is, by definition, the one you didn't know you would need.

The sharpest measurement of this I have seen is a June 2026 paper on what it calls governance decay. Agents given safety policies violated them 0% of the time while the policy sat in full context. After compaction, violation rates averaged 30% across 1,323 test episodes, reaching 59% in some models. The split is the telling part: when the constraint survived the summary, violations stayed at 0%; when it got dropped, they ran 38%. The failure isn't disobedience. It's amnesia. The paper's fix, pinning constraints so the compactor cannot drop them, restored 0%.

That result generalizes past safety policies. Anything that must remain true for the whole session, an API you promised not to break, a directory you must not touch, a definition of done, is one unlucky summarization away from silently vanishing. If a constraint has to survive, it cannot live only in turn 3 of the conversation.

The frontier is moving here too. As of mid-2026, compaction is becoming something models are trained to do rather than a heuristic bolted on afterward: a Tsinghua paper called CompactionRL trains summarization jointly with task execution under a shared reward and reports gains of 5.5 to 7 points on SWE-bench Verified for the models it fine-tunes. Learned compaction is likely to make the average summary better. It will not repeal the lossiness.

## What to actually do, if you just use these tools

You don't control the harness, but you control more of the soft budget than you might think.

Put durable constraints in files, not in chat. Most coding agents reload a project instruction file (`AGENTS.md`, `CLAUDE.md`, and equivalents) every session; that is constraint pinning you get for free. If it must survive four hours, it belongs there, not in something you said once at turn 3.

Prefer several scoped sessions over one epic. A fresh session with a written handoff beats a long session with a silent one, because you get to review a handoff. Restate what matters at phase boundaries; it feels redundant precisely when it is most useful, since the middle of a long context is where recall sags.

And learn the smell of rot. When the agent re-reads files it has already read, re-asks answered questions, or re-implements its own helpers, it is not being thorough. That is your cue to compact deliberately, or hand off to a fresh session, on your terms instead of the model's.

Three honest boundaries on all of the above. There is no consensus on when to compact; the phase-boundary advice is practice, not theorem. The two 2026 papers cited here are preprints, not yet peer-reviewed. And degradation curves differ widely between models and tasks, so your mileage will genuinely vary; the only universal advice is to measure your own sessions.

The windows will keep growing; the papers above suggest the attention economics degrade before the limits, in every window size tested so far. Treat the context as a workspace you tidy, not a tape you append to, and long sessions stop being a gamble.
