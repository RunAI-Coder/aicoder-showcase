# Nobody reads your architecture overview. Including the agent.

*2026-08-06 · the RunAI Coder team · [run.ceo/coder](https://run.ceo/coder/?utm_source=github_Official_Page)*

In January, Vercel published an eval where adding an AGENTS.md file took a coding agent from 53% to 100% on tasks using new framework APIs. In February, a preprint (arXiv 2602.11988) measured context files across agents and models and found they generally don't improve task success at all, while inflating inference cost by over 20%. In April, Augment ran the same experiment on their own monorepo and split the difference: a good file was worth 10 to 15 percent under specific conditions, and a bad one was worse than no file.

So: does the instruction file work, or not?

All three results are right. They just weren't measuring the same thing, and the resolution fits in one sentence. Agents follow instructions and ignore essays.

## What the three studies actually measured

Vercel's eval targeted APIs that shipped after the model's training data ended. The agent could not know `cacheLife()` or `'use cache'` from pretraining; the file was the only source. Result: 53% to 100%. When the file is the only place a fact lives, the file wins.

The February preprint tested closer to the opposite case: SWE-bench tasks drawn from popular repositories, with context files generated along official recommendations, plus a second collection of repos with developer-committed AGENTS.md files. No general improvement, 20%-plus cost increase, and two internal splits that got less attention than the headline. Instructions in the files were followed well, while repository overviews didn't help, despite being the thing most templates tell you to write first. And the authors' own carve-out points the same direction: context files earn their keep for specifying non-standard practices, the things nothing in pretraining or the code itself would tell the model.

Augment's numbers put a price on each content type. Procedural workflows, the checklist kind, dropped PRs-with-missing-wiring from 40% to 10%. Decision tables lifted convention adherence by a quarter. Three-to-ten-line examples lifted code reuse by a fifth. Their headline 10-to-15-percent gain came from a specific shape: a 100-to-150-line main file pointing at focused reference docs, on mid-sized modules, and the gain declined once the main file grew past that length. A big architecture overview did the opposite of helping: the agent dragged roughly 80K tokens of context it didn't need into the session, and completeness dropped 25%. Note what that failure mode is made of. The agent did read the overview. It just couldn't bill the reading to anything.

Same shape everywhere. What the model cannot know, write down. What the model can infer from the code, or already absorbed in pretraining, becomes a tax.

## The file is rent, not documentation

The mental model that makes all of this predictable: an instruction file is not documentation. Documentation waits until someone needs it. An instruction file is loaded into the context window of every session, which means every line is billed on every request the session makes, in the two currencies coding agents pay with. Dollars, because the file rides along in input tokens for each of the session's dozens of turns, softened but not repealed by prompt caching. And attention, where no cache discount exists: we went through the evidence in an earlier piece on context rot, and the short version is that degradation from irrelevant context starts well before any hard limit.

So each line has to pass a rent test. Would the agent otherwise have to excavate this fact, at higher cost, or get it wrong? Commands with non-obvious flags pass. The one directory you must never touch passes. A tour of the module graph fails, because the agent can read the module graph, and mostly doesn't need to.

We ran a small experiment to see the economics up close.

## A twenty-minute field test

We took a fresh clone of Flask, the Python web framework, and asked a coding agent one question a real session asks constantly: what exact command does CI use to run the tests, and what runs the type checks. The true answer is `uv run --locked --no-default-groups --group dev tox run` behind a `TOX_ENV` variable, plus the same invocation with `-e typing`. Three non-default flags and an environment variable: not guessable. The agent has to open the CI workflow file to find it, or be told.

Five configurations, two runs each, same question, same harness. No instruction file. A lean file of about 950 bytes stating the commands and a few project facts. A bloated file of 86 KB, which is the lean content buried at the bottom of a long architecture overview built from Flask's real design docs. Then the lean and bloated files again under a second filename, for reasons that will be obvious in a moment.

| Configuration | File in context? | Turns | Total input tokens | Correct |
|---|---|---|---|---|
| No file | — | 3–4 | ~98,500 | yes |
| Lean, standard name | no | 3–4 | ~98,900 | yes |
| Bloated, standard name | no | 4 | ~98,500 | yes |
| Lean, harness's own name | yes | 2 | ~66,100 | yes |
| Bloated, harness's own name | yes | 3 | ~180,200 | yes |

*(Total input = uncached input + cache writes + cache reads, per request, summed for the run; token totals shown from the warm-cache run of each pair, turn counts are the range across both. Caching discounts what a cache-read token costs in dollars, so the dollar gaps are narrower than the token gaps; the attention ledger takes no such discount. Single repo, single question, n = 2 per lane: a portrait, not a benchmark.)*

Three findings, in increasing order of embarrassment.

First, the lean file that actually got read saved a third of the tokens. Not because 950 bytes is cheap, but because it saved a turn. The agent answered from the file instead of excavating the CI config, and an avoided turn is the whole conversation-so-far not being re-sent one more time. That's the real unit of savings: turns, not bytes.

Second, the bloated file was worse than no file at all. It burned 83% more input tokens than the bare run, because 86 KB of overview rode along on every turn. And it didn't even save the excavation: in both runs the agent spent an extra tool turn rather than answering straight from the file, and in one of them it explicitly reported double-checking the CI workflow, apparently unwilling to trust two useful lines drowned at the bottom of an essay. Rent paid, no service rendered. That points the same direction as the February preprint's cost finding, deliberately exaggerated: their ordinary-sized files averaged plus 20 percent, our pathological one hit 83.

Third, and the reason the table has five rows: the harness we ran silently ignored the standard-named file. The lanes with AGENTS.md, lean or bloated, are indistinguishable from the lane with nothing; the differences are smaller than the gap between two runs of the same lane. The format's own site lists this harness among its thirty-plus supporting tools. With default settings, in the current version, on our machine: not in context. We double-checked with a no-tools probe, asking what the instruction file says while forbidding file reads. Under the standard name the agent answered UNKNOWN; under the harness's own preferred filename it quoted the file verbatim. We did not expect this to be finding number one, but it is, because every downstream question, lean versus bloated, overview versus commands, is moot if the file never enters the window.

All ten runs answered correctly, which is its own small lesson: on a lookup task, quality was never in danger. Only the bill was.

## What earns its rent

Pooling the three studies, the evidence sorts content into payers and freeloaders with unusual agreement for a field this young.

What pays: exact commands with their flags, especially ones that can't be guessed. Procedural checklists for multi-step chores, the "when adding an endpoint, also touch these three files" kind. Decision tables that disambiguate close calls. Short real code snippets over long prose descriptions of style; an analysis of instruction files across 2,500 repositories was blunt enough to state it as a rule: one real snippet beats three paragraphs describing it. Prohibitions paired with what to do instead. Boundaries in three tiers: always, ask first, never.

What freeloads: the architecture overview. Restating what the code says. Style guidance a linter already enforces, which belongs in the linter, where it's machine-checked instead of politely requested. Long lists of warnings; past thirty or so, Augment measured agents getting distracted into code they had no reason to touch.

Two structural notes that fall out of the discovery data. Augment measured how reliably agents find things: the root instruction file was found in every session, files it explicitly references in over 90%, directory READMEs in 80%, nested READMEs in 40%, and orphaned docs folders in fewer than one session in ten. So a lean root file that points to focused reference docs gets you progressive disclosure that mostly works, keeping the always-loaded rent small while the details stay one hop away. And in monorepos the convention is nested instruction files where the closest one to the edited file wins, which is the same rent logic applied spatially: pay for the context of the code you're actually touching.

## The file is the small lever anyway

An honest ordering of what made repositories legible to agents, in our experience, puts the instruction file second or third, not first. The biggest lever is a machine-checkable definition of done: a test suite the agent can run, types, lint, whatever turns "I think I'm finished" into a verdict the agent can check before it reports back. Instruction files are read at best; test failures are obeyed. The second lever is task scope, handing over work whose completion is verifiable in the first place. The file mostly encodes how to run those checks and the handful of facts the checks can't express.

## Boundaries, honestly

Our probe measures excavation economics on one lookup question, one repo, one harness, two runs per lane. It says nothing about whether instruction files improve code quality on real multi-hour tasks; the preprint's author, asked about the developer-committed files, put their average effect at about plus 4 points, and negative on at least one model. Vercel's 53-to-100 came from an eval suite small enough that commenters asked for hundreds of runs before believing the gap, and the same thread carried a sharper warning about fragility: one practitioner flipped an agent's file-adherence from zero-out-of-three to three-out-of-three by rewording the instructions from imperative to first person. Nobody has fully mapped that.

And the field is moving under the convention faster than the convention settles. As of August 2026 the AGENTS.md format is stewarded by a Linux Foundation body, lists thirty-plus supporting tools, and is used by over 60,000 open-source projects; whether your particular harness reads it, at what precedence, with what size limits, is a per-tool, per-version fact. Ours didn't. The twenty-minute check above is cheap enough to just run.

So: write the twenty lines you can defend. Skip the essay. Put the style rules in the linter and the definition of done in the tests. And before polishing a single word of the file, check that your agent reads it at all, because the best-written rent in the world buys nothing from a tenant who never moved in.
