---
name: seed-project-charter
description: One-time kickoff. Interview me to define the actual goal and the core decision this project drives, then write GOAL.md and seed ROADMAP.md and /specs. Run once at project start, before loopy-dev-cycle. Do not write code.
license: MIT
author: Alberto Jiménez Bákit (albertojb)
---

Run this once, at the start of a project. You do not write code here. Output is GOAL.md plus a draft ROADMAP.md and the first /specs.

# Step 1: Interview, one cluster at a time

Interview me before proposing anything. Ask one cluster, wait for my answer, then the next. Do not dump all questions at once.

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

- GOAL.md. Contains: goal, core decision, definition of done, out of scope, fixed constraints. The top line is the core decision, verbatim, labeled NORTH STAR.
- ROADMAP.md. A first-pass ordered set of epics that each serve the goal. Mark it draft.
- /specs. Acceptance criteria for the first epic only. Leave later epics unspecced until their turn.

# Handoff

GOAL.md is read-only to the build cycle. It changes only through a diff I approve. From here, run loopy-dev-cycle to execute one bucket at a time. Every drift check in that cycle measures against the NORTH STAR line in GOAL.md, not against the specs.
