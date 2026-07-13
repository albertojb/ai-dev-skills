---
name: loopy-dev-cycle
description: On-demand, human-gated dev cycle for this repo. Reads GOAL.md as the drift anchor, scopes the run to all open issues plus anything new you name at invocation, groups the work into coherent chunks (one PR each), warns you if the total is too much, then after one plan approval builds every chunk in an uninterrupted, crash-resumable run — git commits are the checkpoints, a run-manifest issue is the live state. Two human gates only: approve the plan, approve the merges. Never merges to main. Trigger manually; re-invoking resumes an interrupted run.
license: MIT
author: Alberto Jiménez Bákit (albertojb)
---

You run this only when I invoke it. There are exactly two human gates: approve the plan, approve the merges. Between them you never stop, never ask "shall I continue?", and never invent your own batches, segments, or checkpoints beyond the chunks in the approved plan. Never merge to main. This cycle assumes GOAL.md, ROADMAP.md, and /specs already exist; if they do not, run seed-project-charter first.

# Resume check (before anything else)

If an open run-manifest issue with unticked items exists, this invocation is a resume, not a new run:
- Re-read the manifest and the frozen criteria in it verbatim. They still govern. Do not reinterpret or renegotiate them.
- `git log` the chunk branches and check open PRs to confirm what actually landed, then continue from the first unticked item.
- Do not redo completed items. Do not re-open the plan gate.

# Phase 1: Gather (read only)

Read all of these before planning. Do not modify code yet.
- GOAL.md first. The NORTH STAR line (the core decision the project drives) is the top drift anchor for the whole run, above the roadmap and above the specs.
- CONTEXT.md, ROADMAP.md, STATUS.md, /specs (acceptance criteria).
- All open issues: `gh issue list --state open`. While reading, do light hygiene as a side effect: close obvious stale duplicates on sight; note anything mislabeled or missing a definition of done as one line in the plan. No separate triage phase, no triage report.

# Phase 2: Plan

**Scope the run.** The run covers all open issues plus anything new I named when invoking. Exclude an issue only if it does not serve the NORTH STAR or has no workable definition of done — and flag it in the plan rather than silently dropping it. Human-opened issues outrank agent-opened ones unless the roadmap says otherwise.

**Be honest about size.** State the real total up front, however long. If the scope is too much for one run, say so plainly and propose what to defer; I decide at the plan gate. Do not silently shrink the scope to look fast, and do not pad it to look thorough.

**Divide into chunks.** Group the work into coherent chunks — same module, same theme, or dependency-ordered — each small enough to be one reviewable PR. Order chunks by dependency first, then impact. Never order or group by ease; difficulty is not a sorting key.

**Write the full work manifest:** per chunk, every file to change, every test to write, every step — as a checklist — plus the acceptance criteria (from the issues and /specs) that chunk's PR must prove.

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
- Note the key decisions this run commits to; you will log them in Phase 7.

# Phase 4: Build (uninterrupted)

Work the chunks in the approved order. For each chunk: branch off main, then work its manifest items top to bottom.
- After each completed item: tick it in the run-manifest issue, commit, push. The commit is the checkpoint — a crash or limit costs at most one item, never the run.
- Structured commits that reference the issue numbers.
- When a chunk's PR opens (Phase 5), post a one-line note ("chunk 3/7: PR #42 open") without stopping or waiting for a reply, and start the next chunk immediately.
- Stopping mid-run is a rule violation unless you are genuinely blocked: broken environment, contradictory frozen criteria, something only I can resolve. If blocked, say exactly what is blocking — not "shall I continue?".

# Phase 5: Review and PR (per chunk)

Before opening each chunk's PR, two checks in order:
1. **Degunk the diff.** Run `git diff main...HEAD` and clean the changed lines and their immediate context only: dead code, swallowed errors, over-defensive checks, type-suppression hacks (`@ts-ignore`, `any` casts, `# type: ignore`, `eslint-disable` without a justification comment), hallucinated imports, style drift. Match the existing codebase; do not gold-plate. Focus on slop this run introduced, not pre-existing issues.
2. **Full test suite and linter**, run by you. Never open a red PR.

Then open the PR. The body lists the issues covered, the frozen criteria, and the chunk's completed manifest.

Then run an independent reviewer in a separate context. Your degunk above was maker-side and does not count as review.
- The reviewer sees only the PR diff and the frozen criteria — not your build reasoning, not the degunk log.
- It verifies each criterion against the diff and runs only the tests that prove that criterion. It does not re-run the full suite; that was your pre-PR gate. It does not trust your claims — PASS or FAIL per criterion, with the proof.
- On FAIL: push a fix and re-review once. If the PR fails review a second time, **park it**: convert it to draft with a note explaining exactly what failed, park any chunks that depend on it (noted in the run-manifest issue), and continue with the remaining independent chunks. A bad chunk never sinks the run. No failing PR left open and silent.
- If every remaining chunk is parked or blocked, stop and report.

# Phase 6: STOP for merge

Do not merge. When the last chunk is done, post the run report: shipped PRs, parked PRs with reasons, anything deferred at the plan gate. Merge is mine, enforced by branch protection on main.

# Phase 7: Document and hand off (after I merge)

- STATUS.md: a short entry — what landed, which issues — with a "next run starts here" line at the top. Always.
- DECISIONS.md: one line per key decision from this run, in the form "decision — why — which GOAL line it serves." Always. This is how we keep intent auditable.
- ROADMAP.md: only if a roadmap line completed or changed. Propose the next run's target.
- CONTEXT.md: only if the run discovered a durable constraint future runs need. This is the exception, not a routine update.
- Close the run-manifest issue, noting any parked PRs still awaiting a decision.

# Guardrails

- Never merge to main.
- GOAL.md, specs, and frozen criteria change only through a diff I approve.
- Chunks are grouped by coherence and ordered by dependency and impact — never by ease.
- No self-imposed pauses between the two gates.
- A PR that fails review twice is parked, not retried forever; dependent chunks park with it.
- If every remaining chunk is parked or blocked, stop and report.
- If a tool call fails 3 times, stop and surface it.

# Trigger

Manual only. I invoke this. It does not run on a schedule and does not self-restart. Re-invoking after an interruption resumes the run (see Resume check).
