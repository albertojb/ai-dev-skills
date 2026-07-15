---
name: seed-project-charter
description: One-time kickoff. Interview me to define the actual goal and the core decision this project drives, write GOAL.md and seed ROADMAP.md and /specs, and set up the git branch model loopy relies on (a dev integration branch as the repo default, with master kept as release only). Run once at project start, before loopy-dev-cycle. Do not write application code.
license: MIT
author: Alberto Jiménez Bákit (albertojb)
---

Run this once, at the start of a project. You do not write code here. Output is GOAL.md plus a draft ROADMAP.md and the first /specs.

# Step 0: Ingest (existing repos only)

Skip this step entirely if the repo has no code and no commits.

If the repo already has code or commits, run this before anything else:

- Read: README, docs/, any existing GOAL.md / ROADMAP.md / STATUS.md / CONTEXT.md, package manifests (package.json, pyproject.toml, Cargo.toml, go.mod, etc.), and the top-level file tree. Do not read every file — sample structure and key docs, scale to the repo size.
- Skim the last ~30 commits and open issues if reachable via git log or gh.
- Output a short current-state summary covering:
  - What the project appears to do and who it seems for.
  - What was worked on recently.
  - What is missing or ambiguous.
  - The goal and core decision you infer from what you read.

Stop after the summary. Wait for me to respond before continuing.

# Step 1: Interview, one cluster at a time

**Existing repo:** Confirm or correct the goal and core decision inferred in Step 0. Present them as your best guess, ask me to confirm or correct, then probe only the gaps ingest could not resolve. Do not re-ask anything the ingest already answered clearly.

**Greenfield repo:** Ask from a blank slate, one cluster at a time, waiting for my answer before the next.

Clusters to cover (either confirming from ingest or asking fresh):

- The actual goal. Not the feature list, the outcome. What is true when this works that is not true today.
- The core decision or action this project is meant to drive. Who acts on it, and what do they do differently once it exists. This is the most important answer in the interview.
- Who it is for.
- Definition of done. A finish line we can test against, not "it feels good."
- What is explicitly out of scope.
- Fixed constraints: stack, deadline, budget, compliance, anything non-negotiable.

Cap your follow-ups. Stop interviewing when you can state the goal and the core decision in two sentences I would sign.

# Step 2: Reflect back, then STOP

Write back, in my words:
- The actual goal, one sentence.
- The core decision the project drives, one sentence. This becomes the drift anchor for every later bucket.
- Definition of done.
- Out of scope.

Stop. Wait for me to confirm or correct. Do not proceed on assumption.

# Step 3: Write artifacts (after I confirm)

**Greenfield:** Create GOAL.md, ROADMAP.md, and /specs from scratch.

**Existing repo:** If GOAL.md, ROADMAP.md, or /specs already exist, propose a diff for each file and wait for my approval before writing anything. Do not overwrite. Derive the roadmap from the actual current state plus remaining work, not from scratch — what is already done should be marked done, not re-listed as future work.

In all cases, artifacts contain:

- GOAL.md: goal, core decision, definition of done, out of scope, fixed constraints. The top line is the core decision, verbatim, labeled NORTH STAR.
- ROADMAP.md: an ordered set of epics that each serve the goal. Mark it draft. On existing repos, reflect what is already complete.
- /specs: acceptance criteria for the first unfinished epic only. Leave later epics unspecced until their turn.

# Step 4: Set up the branch model (after artifacts, if a git remote exists)

Loopy depends on a specific two-branch layout. Set it up once here so every later run inherits it. Skip only if there is no git repo or no remote yet; if so, tell me to create the GitHub repo, then re-run this step.

The model, stated plainly so it lands in the repo's own docs:
- **`dev` is the default branch and the integration branch.** All loopy work branches off `dev` and merges back into `dev`. New clones and PRs default to `dev`.
- **`master` (or `main`) is production, and release only.** Loopy never touches it. It advances only through an explicit, human-requested, twice-confirmed release from `dev`.

Do this:
1. Detect the production branch name: `master` or `main`, whichever the repo already uses. Do not rename it.
2. Create `dev` from it if it does not exist, and push it: `git checkout <prod> && git pull && git checkout -b dev && git push -u origin dev`. If `dev` already exists, leave it.
3. Make `dev` the repo default branch: `gh repo edit --default-branch dev`. This is the directive: after this, everything points at `dev` and `master` is release-only.
4. Best effort, protect the production branch so nothing merges into it by accident: `gh api -X PUT repos/{owner}/{repo}/branches/<prod>/protection ...` requiring a PR before merge. If this fails (needs admin rights or a paid plan), do not block. Print one line telling me to enable branch protection on `<prod>` manually in GitHub settings and continue.
5. Record the branch model in a short section of README.md (or CONTEXT.md if README is not the right home): dev is default and where work happens, master is release-only, and loopy opens one PR per run into dev.

Report what you set: the production branch name, that `dev` exists and is now the default, and whether branch protection was applied or needs a manual step from me.

# Handoff

GOAL.md is read-only to the build cycle. It changes only through a diff I approve. The branch model is now: `dev` is the default and integration branch, `master` is release-only. From here, run loopy-dev-cycle to execute one run at a time; each run cuts one branch off `dev` and opens one PR into `dev`. Every drift check in that cycle measures against the NORTH STAR line in GOAL.md, not against the specs.
