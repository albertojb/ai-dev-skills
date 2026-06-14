# AI Dev Skills

Two composable agent skills for running a disciplined, human-gated build loop on a repo. Claude-first, and the same files load in GitHub Copilot CLI and Zo Computer.

## What they do

- **seed-project-charter** (run once): interviews you to lock the actual goal and the core decision the project drives, then writes `GOAL.md` and seeds `ROADMAP.md` and `/specs`.
- **loopy-dev-cycle** (run per bucket): reads `GOAL.md` as the drift anchor, triages issues, scans code with `code-degunker`, plans, checkpoints with you, builds one agile bucket per run, logs key decisions, and never merges to main without your approval.

Run `seed-project-charter` once at the start of a project. Then run `loopy-dev-cycle` once per bucket, on your trigger.

## Why one format works everywhere

Both files are Agent Skills: a `SKILL.md` with YAML frontmatter (`name`, `description`). Claude, Copilot CLI, and Zo all load this same format and inject it into the agent's context when the description matches the task.

## Install

**Claude Code**
Copy each folder into `.claude/skills/` (per project) or `~/.claude/skills/` (global). Invoke with `/seed-project-charter` and `/loopy-dev-cycle`.

**GitHub Copilot CLI**
Copy each folder into your Copilot skills directory. Run `/skills list` to confirm they loaded, then invoke with `/loopy-dev-cycle`. Reference: GitHub Docs, "Adding agent skills for GitHub Copilot CLI."

**Zo Computer**
Copy each folder into `Skills/`. Zo reads the frontmatter and loads the `SKILL.md` when a task matches.

## License

The two skills here (`seed-project-charter`, `loopy-dev-cycle`) are released under the MIT License, attribution preserved. See `LICENSE`. The copyright line is set to Alberto Jiménez Bákit; edit it to just `albertojb` if you prefer. If you would rather use a content-style attribution license, swap MIT for CC BY 4.0; the structure is the same.

## Credits

`loopy-dev-cycle` calls `code-degunker`, a third-party code-review skill from Zo Computer Skills (https://www.zo.computer/skills/code-degunker). It is referenced, not bundled, and keeps its own license. Install it separately on the same surface, or remove the degunker steps in Phase 2a and Phase 5 if you do not have it.

## Notes

- `GOAL.md` is read-only to the cycle. It changes only through a diff you approve.
- The cycle enforces a human merge gate. Pair it with branch protection on `main` that requires your review, so the gate is real and not advisory.
