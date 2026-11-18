# Recipe app agent instructions

## Commit completed work

- When a requested change is complete, validate it, review the diff, and commit
  the task's changes before reporting completion. This includes documentation
  and configuration. Do not wait for a separate request to commit.
- Make focused, descriptive commits for independently meaningful changes. Include
  the documentation for a change in the same commit.
- Inspect the working tree and branch workflow before editing. Stage only this
  task's files or hunks; preserve unrelated work and existing history.
- Commit locally. Pushing, publishing, and merging require the user's instruction.
- If committing is blocked, explain the blocker and what remains uncommitted.
  Read-only tasks do not require an empty commit.

## Keep documentation current

- Assess documentation impact on every change. Update affected documentation and
  add missing guidance for new behavior, UI, configuration, and workflows.
- Write usage guidance for someone who uses the app but does not write its code.
  Explain where features are, their inputs, steps, and expected results.
- Keep the README overview and setup instructions accurate and link detailed
  guides. Keep status notes current and distinguish plans from implemented work.
- If no documentation change is needed, state why in the completion report.

## Completion report

Explain what changed and how to use it, documentation updated (or why none was
needed), validation and outcomes, commit hashes and branch, and limitations or
blockers. Do not describe incomplete work as finished.

## Project conventions

- Read `.docs/DESIGN.md` before implementing features. It is the agreed product
  baseline; clearly identify proposed changes to that baseline.
- Use `codex/` branches for task work unless the user specifies otherwise.
- Keep personal cookbook data, images, generated PDFs, credentials, and local
  settings out of source control. Use synthetic recipes for automated checks.
- Keep the implementation status in README.md accurate. Do not describe planned
  features as implemented or claim a runnable build before one exists.
