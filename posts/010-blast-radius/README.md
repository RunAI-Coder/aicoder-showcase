# The blast radius of a one-line fix

*2026-08-10 · the RunAI Coder team · [run.ceo/coder](https://run.ceo/coder/?utm_source=github_Official_Page)*

This May, Flask merged a fix for a bug that shipped with Flask 0.5 in 2010 and sat there for sixteen years: template filenames were checked for their extension case-sensitively, so `page.html` got Jinja's autoescaping and `page.HTML` did not. Render user input through the second one and you have an XSS hole. The [entire fix](https://github.com/pallets/flask/commit/9368fb3f3c52d74534d14c1bef03c79c103356cd) is one line — `filename.endswith(...)` becomes `filename.lower().endswith(...)` — plus a changelog entry. No test; upstream judged it too small to need one.

We reverted that fix in a local clone and spent Sunday handing the bug back to a coding agent, nine times, to measure the thing people argue about constantly and measure almost never: how much does an agent actually touch when you ask it for a small fix?

You know the complaint this is chasing. You ask for a button color change and the diff comes back with your router "improved". [GitClear's analysis of 211 million changed lines](https://gitclear-public.s3.us-west-2.amazonaws.com/GitClear-AI-Copilot-Code-Quality-2025.pdf) puts industry-scale numbers under that feeling: code moved during refactoring fell from 24.1% of changes in 2020 to 9.5% in 2024, copy/paste rose from 8.3% to 12.3% and passed refactoring for the first time on record, and the share of lines rewritten within two weeks of being authored went from 3.1% to 5.7%. Something is writing a lot of code that doesn't sit still. We wanted the single-task version of that number.

## The rig

pallets/flask, checked out at the commit right before the fix. 491 tests passing. A mainstream coding-agent CLI at default settings (as of 2026-08), running headless with every permission prompt auto-approved — the exact configuration [our approval-fatigue piece](../009-approval-fatigue) worried about, which felt fitting. Each run starts on a fresh branch. Afterward we measure everything against the base commit, so a sneaky `git commit` can't hide anything: files touched, lines changed, whether the bug is actually fixed, whether the full suite still passes, and the complete tool-call transcript.

Same bug, three phrasings, three runs each:

> **A — the bug report.** "Bug report: Jinja autoescaping is not applied to templates whose filenames have uppercase extensions. Rendering `page.HTML` or `INDEX.HTM` skips autoescaping, while `page.html` gets escaped. This is an XSS risk for user-supplied values. Please fix this bug in the codebase."
>
> **B — the bug report, plus a leash.** Same text, one sentence appended: "Keep the change minimal: touch as few files and as few lines as possible, and do not refactor, reformat, or change anything unrelated."
>
> **C — the vibe.** "some of my templates arent getting autoescaped and user input shows up as raw html on the page. i think somethings off with how flask decides when to autoescape. can you find and fix it"

Lane C is a faithful transcription of how bugs get reported when the reporter doesn't know the cause. Which is to say: how bugs get reported.

## The agent found our answer key

Our first nine runs went in the trash, and the reason is some of the best data we collected all day.

The clone we handed the agent still had its git history — including, one commit past our checkout, the real fix. In the first vague-lane run the agent came back in five and a half minutes with the right diagnosis and a suspiciously confident writeup: it had "confirmed the root cause by comparing against `main`", where upstream uses `filename.lower()`. The debugging never had to close the case: the answer was sitting in the history we'd left in the clone, and it copied the fix down to the upstream docstring, word for word.

We had designed an open-book exam. The agent took the book.

So, surgery: delete the remote, delete every branch and tag past the checkout, expire the reflog, garbage-collect until `git cat-file` confirms the fix commit no longer exists in the object store. Then nine fresh runs in a repository where, as far as any local evidence goes, the fix never happened.

## Nine clean runs

| run | prompt | files touched | lines | fixed? | regression test? | the agent's own verification |
|---|---|---|---|---|---|---|
| A1 | precise | 1 | +1/−1 | yes | none | behavior check, targeted tests, then an end-to-end render in a tempdir |
| A2 | precise | 2 | +8/−1 | yes | 7 lines | targeted tests, then the full suite |
| A3 | precise | 2 | +19/−1 | yes | 18 lines | targeted tests |
| B1–B3 | precise + leash | 1 (all three) | +1/−1 (all three) | yes ×3 | none ×3 | behavior check + targeted tests, every run |
| C1 | vague | 3 | +12/−0 | yes | 9 lines | git archaeology first, then targeted tests |
| C2 | vague | 3 | +25/−1 | yes | 17 lines | targeted tests |
| C3 | vague | 2 | +15/−1 | yes† | 11 lines | full suite twice, plus a repro script it wrote and later deleted |

†C3's diagnosis was web-assisted; details two sections down.

Nine for nine on correctness, and six of the nine produced, character for character, the line upstream merged. The full 491-test suite stayed green after every run. No reformatting, no renamed variables, no drive-by cleanup. We went looking for a crater and could not produce one. Hold that thought — the reasons matter more than the result.

The leash worked completely. One file, one line, three out of three, at the lowest cost and latency of any lane ($0.32–0.39, about a minute per run). It was the only knob in the experiment that did exactly the same thing all three times we turned it. And it cut both ways: the leash lane was the only lane where no run wrote a regression test; five of the other six runs added one. The sentence that stops an agent from touching what you didn't ask about also stops it from adding the thing you didn't ask for but wanted. The radius dial and the seatbelt dial turned out to be the same dial.

Vagueness, meanwhile, didn't buy the disaster the horror stories promise. It bought a bill. The precise lanes converged in 6–10 turns for $0.32–0.64. The vague lanes ran to 12 and 19 turns — plus one that reads as six turns only because it handed part of the job to a subagent — took from under two minutes to thirteen, and cost $0.48–3.90, the most expensive run paying 12× the cheapest. What you leave out of the prompt, the agent has to go find.

## The radius that actually varies

Here is the part the diff table doesn't show: the *edit* footprint stayed polite in every lane; the *read* footprint went wandering.

C3 — vague lane, run three, one run out of nine — opened with git archaeology (`git log -p -S 'filename = filename.lower()'`: asking whether a `.lower()` had ever been deleted, a genuinely good instinct we had just made useless), wrote itself a repro script, and then, before touching any code, pulled the current upstream `app.py` from GitHub's raw servers and compared. That is the answer key again. Cutting the fix out of local history did nothing about the copy on the internet, so C3's diagnosis gets no clean-room credit — we count it as web-assisted, and its final writeup cheerfully noted the fix matched what upstream ships.

What it did *after* the one-line edit is the stranger part. It spent most of its 27 shell commands convincing itself nothing else was wrong: ran the full suite, walked out of the repository, listed the parent directory, found the sibling clones from our other lanes, ran `diff -r` between its Flask and theirs, audited the jinja2 and markupsafe installs in two of our virtualenvs, then came back, added a regression test, and ran everything again. Ruling out the environment is what a careful person does when a symptom sounds impossible — but two lessons fell out of it. If there are things near the repo an agent shouldn't read — other checkouts, database dumps, an `.env` two directories up — assume a default-configured agent will find them, because ours mapped our whole lab bench. And for a public codebase the answer lives on the internet, and a default-configured agent has the internet. Anyone benchmarking "can the agent diagnose this known bug" against a public repo is, to some degree, measuring search.

A confession in the same spirit: our first pass over the transcripts "discovered" that the bug-report lane never executed the tests it wrote. A tidy little scandal — until we noticed our analyzer counted `bash` tool calls, and on this Windows machine the agent runs commands through PowerShell. All nine runs verified their own work, several more thoroughly than we would have. We nearly published an accusation that our own instrument had fabricated. We now check the analyzer before we check the agent.

## Why our crater is missing, and where the real ones are

The cheap reason: Flask gave the agent very little room to be wrong. A 491-test suite is a wall, and every one of the nine runs reached for it — or a targeted slice of it — without being told to. We closed [our AGENTS.md piece](../008-agent-friendly-codebases) with "put the definition of done in the tests"; these nine runs are that advice paying out, because the guardrails Flask built for human contributors are what kept the agent honest. On a weekend project with no tests, these same nine runs would have had nothing to stay inside of, and nothing to check themselves against.

The boring reason: this was a one-cause bug with an objective repro. Blast radius grows with ambiguity, and we deliberately fed the ambiguity into the prompt instead of the codebase. A vague symptom in a tangled codebase is the case our grid doesn't cover, and we'd bet real money the numbers get worse.

The vendor reason: the radius depends on which agent you bought, and the spread is wide. A [SANER 2025 study](https://arxiv.org/abs/2410.12468) compared roughly 4,900 patches from ten agent frameworks against the developers' own fixes across 500 SWE-bench Verified issues. Among the agents the paper breaks out, agreement on which *files* to touch runs from 94.6% for one down to 54.7% for another. Agreement on which *functions* to touch was, in the paper's words, substantially lower for every agent — the examples it breaks out sit between 14% and 26%. Patch sizes tell the same story: on issues the original developers fixed with a mean of under eight changes, one agent averaged 47.5 while another averaged 5.7 — one harness's mean footprint was eight times another's, on the same benchmark. Blast radius is a property of the harness, and it isn't printed on the pricing page.

And a tight diff can still be the wrong diff in ways a green suite won't show. An [April 2026 study](https://arxiv.org/abs/2604.05955) checked issue-resolving patches against 1,787 of the projects' own design constraints across six repos: among patches that resolved the issue and passed the tests, only a third to roughly half fully satisfied the constraints, depending on which of the four agent frameworks and which benchmark split — and handing the constraints to the model in the prompt cut violations by at most 6.35 percentage points, leaving residual violation rates above 30%.

## What we type differently now

On the prompt side: state the radius you want — it went three for three, and it costs a sentence. But say what you want *included* too; "keep the change minimal, and add a regression test" is the obvious variant, and we haven't run that grid yet, so treat it as an untested hypothesis for now. And when the report is vague, expect a diagnosis phase: the agent wanders because it has to reconstruct the context you didn't supply, and five minutes of narrowing the symptom yourself is the cheapest step in the whole pipeline.

On the repo side: give the agent walls it can feel. Nine out of nine runs voluntarily ran tests; the repo that has them gets self-checking agents for free, and the repo that doesn't gets confident diffs instead.

## What nine runs can't tell you

n=3 per prompt: nine runs, one bug, one repo, one harness, one weekend, one Windows machine. Costs are the agent CLI's own accounting, in list-price API dollars. A mature Python repo with 491 tests is close to a best case, and we deliberately didn't test the no-tests project, because we'd first have to agree on what "broke" means there. Radius also isn't a quality score — the vague lane's three-file footprints were arguably the best engineering of the day (fix, regression test, changelog: exactly what a maintainer would ask for), so if we'd scored completeness instead of restraint, the leaderboard flips. And sometimes vague is all you honestly have. Precision about the cause is the output of debugging, so when you don't have it, the diagnosis is exactly what you're paying the agent for. Pay the 12×, or narrow it yourself first.

The horror stories usually blame the model, as if scope creep were a mood. Our nine runs located it differently: the radius was set before the agent read a single file — by the sentence we typed, by the test suite Flask's maintainers left standing, and by the defaults we didn't change. The agent brought the variance. Everything that bounded it was already ours.
