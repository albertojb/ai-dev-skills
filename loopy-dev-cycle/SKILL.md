---
name: loopy-dev-cycle
description: On-demand, human-gated dev cycle for this repo. Reads GOAL.md as the drift anchor, picks the single most impactful open issue, plans a full work manifest, then builds it as one PR in an uninterrupted, crash-resumable run — git commits are the checkpoints. Two human gates only: approve the plan, approve the merge. Never merges to main. Trigger manually; re-invoking resumes an interrupted run.
license: MIT
author: Alberto Jiménez Bákit (albertojb)
---

You run this only when I invoke it. There are exactly two human gates: approve the plan, approve the merge. Between them you never stop, never ask "shall I continue?", and never invent your own batches, segments, or checkpoints. Never merge to main. This cycle assumes GOAL.md, ROADMAP.md, and /specs already exist; if they do not, run seed-project-charter first.

# Resume check (before anything else)

If a run branch with a MANIFEST.md containing unticked items exists, this invocation is a resume, not a new run:
- Re-read the manifest and the frozen criteria in it verbatim. They still govern. Do not reinterpret or renegotiate them.
- `git log` the branch to confirm what actually landed, then continue from the first unticked item.
- Do not redo completed items. Do not re-open the plan gate.

# Phase 1: Gather (read only)

Read all of these before planning. Do not modify code yet.
- GOAL.md first. The NORTH STAR line (the core decision the project drives) is the top drift anchor for the whole run, above the roadmap and above the specs.
- CONTEXT.md, ROADMAP.md, STATUS.md, /specs (acceptance criteria).
- All open issues: `gh issue list --state open`. While reading, do light hygiene as a side effect: close obvious stale duplicates on sight; note anything mislabeled or missing a definition of done as one line in the plan. No separate triage phase, no triage report.

# Phase 2: Plan

**Pick ONE issue** — the one that most advances the NORTH STAR and the current roadmap priority. Never pick by ease. If the most important issue is hard, that is the issue. Human-opened issues outrank agent-opened ones unless the roadmap says otherwise. If the top issue cannot fit one reviewable PR, take its largest coherent slice; do not drop down to an easier issue.

**Write the full, honest work manifest** for that issue:
- Every file to change, every test to write, every step — as a checklist. State the real total up front, however long it is. Do not hide scope to look fast, and do not pad it to look thorough.
- The acceptance criteria (from the issue and /specs) the PR must prove.

**Drift check:**
- Restate the NORTH STAR from GOAL.md, verbatim.
- State in one line how the chosen issue serves it.
- Flag any manifest item that does not clearly serve it. If a flagged item still belongs, say why; otherwise cut it.

**Clarify:**
- Ask me only the questions that block good work. Cap at 5. Each states why the answer changes the plan.
- Do not ask for things GOAL.md, the context, or the specs already answer.

STOP. Wait for my approval. Do not write code.

# Phase 3: Freeze (after my answers)

- Revise the manifest from my answers.
- If the work needs a GOAL.md, roadmap, or spec change, propose it as a diff and wait for my approval. Never change GOAL.md or specs silently.
- Create the branch. Commit MANIFEST.md to it: the checklist plus the frozen acceptance criteria. The criteria do not move for the rest of the run.
- Note the key decisions this run commits to; you will log them in Phase 7.

# Phase 4: Build (uninterrupted)

- Work the manifest top to bottom. After each completed item: tick it in MANIFEST.md, commit, push. The commit is the checkpoint — a crash or limit costs at most one item, never the run.
- Structured commits that reference the issue number.
- At roughly the halfway point, post a one-line progress note ("9 of 18 done") without stopping or waiting for a reply.
- Stopping mid-build is a rule violation unless you are genuinely blocked: broken environment, contradictory frozen criteria, something only I can resolve. If blocked, say exactly what is blocking — not "shall I continue?".

# Phase 5: Review and PR

Before opening the PR, two checks in order:
1. **Degunk the diff.** Run `git diff main...HEAD` and clean the changed lines and their immediate context only: dead code, swallowed errors, over-defensive checks, type-suppression hacks (`@ts-ignore`, `any` casts, `# type: ignore`, `eslint-disable` without a justification comment), hallucinated imports, style drift. Match the existing codebase; do not gold-plate. Focus on slop this run introduced, not pre-existing issues.
2. **Full test suite and linter**, run by you. Never open a red PR.

Then the last manifest item: remove MANIFEST.md, commit, and open the PR. The body lists the issue, the frozen criteria, and the completed manifest.

Then run an independent reviewer in a separate context. Your degunk above was maker-side and does not count as review.
- The reviewer sees only the PR diff and the frozen criteria — not your build reasoning, not the degunk log.
- It verifies each criterion against the diff and runs only the tests that prove that criterion. It does not re-run the full suite; that was your pre-PR gate. It does not trust your claims — PASS or FAIL per criterion, with the proof.
- On FAIL: push a fix or close the PR with a reason. If the reviewer fails the PR twice, stop and report. No failing PR left open and silent.

# Phase 6: STOP for merge

Do not merge. Leave the passing PR open for my review. Merge is mine, enforced by branch protection on main.

# Phase 7: Document and hand off (after I merge)

- STATUS.md: a short entry — what landed, the issue number — with a "next run starts here" line at the top. Always.
- DECISIONS.md: one line per key decision from this run, in the form "decision — why — which GOAL line it serves." Always. This is how we keep intent auditable.
- ROADMAP.md: only if a roadmap line completed or changed. Propose the next run's target.
- CONTEXT.md: only if the run discovered a durable constraint future runs need. This is the exception, not a routine update.

# Guardrails

- Never merge to main.
- GOAL.md, specs, and frozen criteria change only through a diff I approve.
- One issue, one PR per run. Split into more PRs only when a single diff would be genuinely unreviewable — and say so in the plan, not mid-build.
- No self-imposed pauses between the two gates.
- If the independent reviewer fails the PR twice, stop and report.
- If a tool call fails 3 times, stop and surface it.

# Trigger

Manual only. I invoke this. It does not run on a schedule and does not self-restart. Re-invoking after an interruption resumes the run (see Resume check).
