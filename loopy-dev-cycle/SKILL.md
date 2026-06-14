---
name: loopy-dev-cycle
description: On-demand, human-gated cycle for this repo. Reads GOAL.md as the drift anchor, triages issues, scans code for quality issues (dead code, swallowed errors, style drift, etc.), plans, asks clarifying questions, then builds one agile bucket with a degunk-and-review gate per PR. Two human gates: approve the plan, approve the merge. Logs key decisions. Never merges to main. Trigger manually.
license: MIT
author: Alberto Jiménez Bákit (albertojb)
---

You run this only when I invoke it. It is interactive. You will stop and wait for me twice: once after the plan, once at the PRs. Never merge to main. This cycle assumes GOAL.md, ROADMAP.md, and /specs already exist; if they do not, run seed-project-charter first.

# What "degunk" means

Degunk is an inline code quality scan — no external skill required. When the instructions say to degunk, you do the following yourself:

**What to flag:**
- Dead code: unreachable branches, unused variables/exports/imports, commented-out blocks that are not explanatory
- Swallowed errors: empty catch blocks, `_ =` discards on errors, silent ignores without a logged reason
- Over-defensive checks: null-guards on values that can never be null at that point, redundant type narrowing, defensive fallbacks that hide bugs rather than surface them
- Type-suppression hacks: `@ts-ignore`, `any` casts, `# type: ignore`, `eslint-disable` without a justification comment
- Hallucinated imports: imports of modules that do not exist in the repo or package.json
- Style drift: naming conventions, file structure, or patterns that differ from the surrounding codebase without a reason

**Severity:**
- Critical: broken or silently wrong behavior (swallowed errors that hide failures, hallucinated imports that will crash)
- Major: significant maintenance debt or type-safety holes (broad `any` casts, large dead code blocks)
- Minor: style drift, small over-defensive checks, cosmetic issues

**Full Review Mode** (used in Phase 2a): Read the modules in scope, apply the checklist above, report findings grouped by severity.

**Branch Review Mode** (used in Phase 5): Run `git diff main...HEAD`, apply the checklist to the changed lines and their immediate context only. Focus on slop the agent just introduced, not pre-existing issues.

# Phase 0: Issue triage (not degunk)

Clean the issue tracker only: close stale duplicates, fix mislabels, flag issues missing a clear definition of done. Degunk is NOT used here; it operates on code, not issues. Report what changed, then move on.

# Phase 1: Gather (read only)

Read all of these before planning. Do not modify anything yet.
- GOAL.md. Read this first. The NORTH STAR line (the core decision the project drives) is the top drift anchor for this whole run, above the roadmap and above the specs.
- CONTEXT.md, ROADMAP.md, STATUS.md
- /specs (acceptance criteria)
- All open issues: `gh issue list --state open`

Sort the open issues into two groups and keep them labeled:
1. Human-opened issues. First-class work. Default them above agent-generated items unless the roadmap says otherwise.
2. Agent-opened issues from prior runs.

# Phase 2: Assess and plan

## 2a: Degunk scan (Full Review Mode)

Run a Full Review Mode degunk scan (as defined above) against the modules tied to this run's roadmap priorities, plus any high-risk area they touch (auth, payments, data mutations). Do not full-scan the whole repo; scale to risk.

Turn findings into issues, with limits:
- File only Critical and Major findings as new issues.
- Dedupe against every open issue first.
- Respect the 3-new-issue cap. If the scan surfaces more, file the top 3 by severity and write the rest into a "degunk backlog" note in the plan.
- Park all Minor smells in that backlog note, not as issues.

## 2b: Build the plan

Plan from the NORTH STAR, roadmap priority, human-opened issues, status, and the degunk findings. For each plan item, name the issue number it maps to and the spec or acceptance criterion it satisfies. Do not plan work that has no issue and no roadmap line; file an issue for it first.

Break the plan into agile buckets: epics at the top, story-sized units underneath, each small enough to be one PR. Order the buckets. Mark which bucket this run will build.

# Phase 3: Drift check and clarify, then STOP

Drift check first:
- Restate the NORTH STAR from GOAL.md, verbatim.
- State in one line how the chosen bucket serves it.
- Flag any plan item that does not clearly serve it. If a flagged item still belongs, say why; otherwise cut it.

Then clarify:
- Ask me only the questions that block good work. Cap at 5. Each states why the answer changes the plan.
- Do not ask for things GOAL.md, the context, or the specs already answer.

Stop here. Wait for my answers. Do not write code.

# Phase 4: Adjust and freeze (after my answers)

- Revise the plan and buckets from my answers.
- If the work needs a GOAL.md, roadmap, or spec change, propose it as a diff and wait for my approval. Never change GOAL.md or specs silently.
- Freeze the acceptance criteria for this run's bucket. They do not move for the rest of the run.
- Note the key decisions this bucket commits to; you will log them in Phase 7.

# Phase 5: Build

For the approved bucket only:
- One branch and one PR per story. Small, reviewable diffs.
- Structured commits that reference the issue number.

Before opening each PR, run two checks in this order:
1. Branch Review Mode degunk (as defined above). Run `git diff main...HEAD` and apply the degunk checklist to the changed lines and their immediate context: strip dead code, swallowed errors, over-defensive checks, type-suppression hacks, hallucinated imports, style drift. Match the existing codebase; do not gold-plate.
2. Tests and linter. Never open a red PR.

Then open the PR. Body lists the issue and the frozen criteria it claims to meet.

Then run an independent reviewer in a separate context. The degunk step above was maker-side and does not count as review.
- The reviewer sees only the PR diff and the frozen criteria, not your build reasoning or the degunk log.
- It runs the tests and linter itself. It does not trust your claim.
- It outputs PASS or FAIL per criterion with the proof.
- On FAIL: push a fix or close the PR with a reason. No failing PR left open and silent.

# Phase 6: STOP for merge

Do not merge. Leave the passing PRs open for my review. Merge is mine, enforced by branch protection on main.

# Phase 7: Document and hand off (after I merge)

Once I have merged:
- Update STATUS.md with what landed.
- Update CONTEXT.md with anything future runs need.
- Update ROADMAP.md to reflect progress, and propose the next bucket.
- Append to DECISIONS.md: one line per key decision from this bucket, in the form "decision - why - which GOAL line it serves." This is how we keep intent auditable.
- Carry the degunk backlog note forward so Minor smells are not lost.
- Write a short "next run starts here" note at the top of STATUS.md.

# Guardrails

- Never merge to main.
- GOAL.md and frozen criteria never change without my approved diff. Specs do not change silently.
- Roadmap, status, context, and DECISIONS.md may be updated, but roadmap changes are proposed in the plan, not made mid-build.
- Degunk in Assess files only Critical and Major issues, capped at 3 per run.
- Build one bucket per run. Cap at 4 PRs per run.
- If 2 PRs in a row fail review, stop and report.
- If a tool call fails 3 times, stop and surface it.

# Trigger

Manual only. I invoke this. It does not run on a schedule and does not self-restart.
