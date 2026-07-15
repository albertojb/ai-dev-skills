---
name: loopy-dev-cycle
description: On-demand, human-gated dev cycle for this repo. Reads GOAL.md as the drift anchor, scopes the run to all open issues plus anything new you name at invocation, groups the work into coherent chunks, warns you if the total is too much, then after one plan approval builds the whole run on a single branch cut from dev in an uninterrupted, crash-resumable pass (git commits are the checkpoints; a run-manifest issue is the live state). One run produces exactly one branch and one pull request into dev; you test that branch and you merge it. Loopy never merges to master (master is release only) and never merges into dev on its own. Trigger manually; re-invoking resumes an interrupted run.
license: MIT
author: Alberto Jiménez Bákit (albertojb)
---

You run this only when I invoke it.

# Branch model (read first, non-negotiable)

This repo uses two long-lived branches:
- **`master`** is production, and release only. Loopy NEVER commits to it, NEVER opens a PR into it, and NEVER merges into it during a normal run. The only path to master is an explicit release I ask for (see "Releasing to master").
- **`dev`** is the integration branch and the repo default. It holds work that has been tested and accepted. Loopy branches off `dev` and targets `dev`, never `master`.

**One run makes exactly one branch and one pull request.** For the whole run you cut a single branch off `dev` named `loopy/run-<manifest-issue#>`. Every chunk lands as commits on that one branch. At the end you open exactly one PR from that branch into `dev`, then STOP. You never merge it yourself; I test the branch and I merge it. Never open a branch or PR per chunk. A single run producing six branches is a bug, not the design.

There are exactly two human gates: I approve the plan, and I merge the run's PR. Between them you never stop, never ask "shall I continue?", and never invent your own batches, segments, or extra branches beyond the one run branch and the chunks in the approved plan. This cycle assumes GOAL.md, ROADMAP.md, and /specs already exist; if they do not, run seed-project-charter first (it also creates `dev` and makes it the default branch).

# Resume check (before anything else)

If an open run-manifest issue with unticked items exists, this invocation is a resume, not a new run:
- Re-read the manifest and the frozen criteria in it verbatim. They still govern. Do not reinterpret or renegotiate them.
- Check out the existing run branch `loopy/run-<manifest-issue#>`. `git log` it to confirm what actually landed, then continue from the first unticked item on that same branch. Do not cut a new branch.
- Do not redo completed items. Do not re-open the plan gate.

# Phase 1: Gather (read only)

Read all of these before planning. Do not modify code yet.
- GOAL.md first. The NORTH STAR line (the core decision the project drives) is the top drift anchor for the whole run, above the roadmap and above the specs.
- CONTEXT.md, ROADMAP.md, STATUS.md, /specs (acceptance criteria).
- All open issues: `gh issue list --state open`. While reading, do light hygiene as a side effect: close obvious stale duplicates on sight; note anything mislabeled or missing a definition of done as one line in the plan. No separate triage phase, no triage report.
- Confirm the branch setup: `dev` exists and is the default branch. If it does not exist, stop and tell me to run seed-project-charter first; do not improvise a branch layout.

# Phase 2: Plan

**Scope the run.** The run covers all open issues plus anything new I named when invoking. Exclude an issue only if it does not serve the NORTH STAR or has no workable definition of done, and flag it in the plan rather than silently dropping it. Human-opened issues outrank agent-opened ones unless the roadmap says otherwise.

**Be honest about size.** State the real total up front, however long. If the scope is too much for one run, say so plainly and propose what to defer; I decide at the plan gate. Do not silently shrink the scope to look fast, and do not pad it to look thorough.

**Divide into chunks.** Group the work into coherent chunks (same module, same theme, or dependency-ordered). Chunks organize the work and the per-chunk quality review; they do NOT each get a branch or a PR. Order chunks by dependency first, then impact. Never order or group by ease; difficulty is not a sorting key.

**Write the full work manifest:** per chunk, every file to change, every test to write, every step, as a checklist, plus the acceptance criteria (from the issues and /specs) that chunk must prove.

**Drift check:**
- Restate the NORTH STAR from GOAL.md, verbatim.
- State in one line how each chunk serves it.
- Flag any manifest item that does not clearly serve it. If a flagged item still belongs, say why; otherwise cut it.

**Clarify:**
- Ask me only the questions that block good work. Cap at 5. Each states why the answer changes the plan.
- Do not ask for things GOAL.md, the context, or the specs already answer.

STOP. Wait for my approval. Do not write code.

# Phase 3: Freeze (after my answers)

- Revise the scope, chunks, and manifest from my answers.
- If the work needs a GOAL.md, roadmap, or spec change, propose it as a diff and wait for my approval. Never change GOAL.md or specs silently.
- Open the run-manifest issue: the frozen acceptance criteria plus the full chunk/item checklist. This issue is the live run state. The criteria do not move for the rest of the run.
- Cut the run branch: `git checkout dev && git pull && git checkout -b loopy/run-<this-issue#>`. This is the only branch the run creates.
- Note the key decisions this run commits to; you will log them in Phase 7.

# Phase 4: Build (uninterrupted, one branch)

Work the chunks in the approved order, all on the single run branch. Do not create additional branches.
- After each completed item: tick it in the run-manifest issue, commit, push the run branch. The commit is the checkpoint. A crash or limit costs at most one item, never the run.
- Structured commits that reference the issue numbers.
- After each chunk clears its quality gate (Phase 5), post a one-line note ("chunk 3/7 done, on run branch") without stopping or waiting for a reply, and start the next chunk immediately.
- Stopping mid-run is a rule violation unless you are genuinely blocked: broken environment, contradictory frozen criteria, something only I can resolve. If blocked, say exactly what is blocking, not "shall I continue?".

# Phase 5: Per-chunk quality gate (no PR yet)

As each chunk finishes on the run branch, before moving to the next chunk, run two checks in order:
1. **Degunk the chunk's diff.** Diff the chunk's commits and clean the changed lines and their immediate context only: dead code, swallowed errors, over-defensive checks, type-suppression hacks (`@ts-ignore`, `any` casts, `# type: ignore`, `eslint-disable` without a justification comment), hallucinated imports, style drift. Match the existing codebase; do not gold-plate. Focus on slop this run introduced, not pre-existing issues.
2. **Full test suite and linter**, run by you. The run branch must stay green.

Then run an independent reviewer in a separate context. Your degunk above was maker-side and does not count as review.
- The reviewer sees only the chunk's diff and the frozen criteria, not your build reasoning, not the degunk log.
- It verifies each criterion against the diff and runs only the tests that prove that criterion. It does not re-run the full suite; that was your gate above. It does not trust your claims. PASS or FAIL per criterion, with the proof.
- On FAIL: push a fix and re-review once. If the chunk fails review a second time, **park it**: revert that chunk's commits off the run branch so the branch stays green and mergeable, note the parked chunk and exactly what failed in the run-manifest issue, park any chunks that depend on it, and continue with the remaining independent chunks. A bad chunk never sinks the run and never leaves the run branch red.
- If every remaining chunk is parked or blocked, stop and report.

# Phase 6: Open ONE PR into dev, then STOP

When the last chunk is done (or parked), open exactly one pull request from the run branch into `dev`. Not into master. The body lists: the issues covered, the frozen criteria with PASS proof, the completed chunk manifest, and any parked chunks with the reason they were parked.

Then STOP. Do not merge. Merging the run's PR into `dev` is mine. Post the run report: what is in the PR, what is parked, anything deferred at the plan gate. The skill never merges the PR itself.

# Phase 7: Document and hand off (after I merge into dev)

- STATUS.md: a short entry, what landed, which issues, with a "next run starts here" line at the top. Always.
- DECISIONS.md: one line per key decision from this run, in the form "decision, why, which GOAL line it serves." Always. This is how we keep intent auditable.
- ROADMAP.md: only if a roadmap line completed or changed. Propose the next run's target.
- CONTEXT.md: only if the run discovered a durable constraint future runs need. This is the exception, not a routine update.
- Close the run-manifest issue, noting any parked chunks still awaiting a decision.

# Releasing to master (rare, explicit, double-confirmed)

Loopy does NOT release as part of a normal run. A release happens only when I explicitly ask for it in a separate instruction ("release dev to master", "cut a production release", or similar).

When and only when I ask:
1. Confirm with me a FIRST time, in its own message: state plainly that this promotes `dev` to `master` (production), summarize what is in `dev` since the last release, and ask me to confirm.
2. After I confirm, confirm a SECOND time, in a separate message: restate that this is the production release and ask me to confirm once more.
3. Only after BOTH confirmations, open a PR from `dev` into `master` and hand it to me to merge. Prefer letting me click merge. Merge to master yourself only if I explicitly tell you to in that same release instruction.

If I have not explicitly asked for a release, you never touch master, never open a dev-to-master PR, and never bring master up in a normal run. Two agent confirmations do not substitute for my explicit request; they come after it.

# Guardrails

- One run = one branch (`loopy/run-<manifest-issue#>`) off `dev` = one PR into `dev`. Never a branch or PR per chunk. Never more than one run branch.
- Loopy never commits to, opens a PR into, or merges into `master` during a run. Master is reached only through an explicit, twice-confirmed release.
- Loopy never merges the run's PR into `dev` itself. I merge it.
- GOAL.md, specs, and frozen criteria change only through a diff I approve.
- Chunks are grouped by coherence and ordered by dependency and impact, never by ease.
- No self-imposed pauses between the two gates (plan approval, and my merge).
- A chunk that fails review twice is parked (its commits reverted off the run branch), not retried forever; dependent chunks park with it.
- If every remaining chunk is parked or blocked, stop and report.
- If a tool call fails 3 times, stop and surface it.

# Trigger

Manual only. I invoke this. It does not run on a schedule and does not self-restart. Re-invoking after an interruption resumes the run on the existing run branch (see Resume check).
