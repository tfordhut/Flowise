---
description: One tick of a Fable-directed / Codex-written / adversarially-reviewed dev loop
argument-hint: [task description or path to a spec/PLAN.md]
---

# Codex dev loop — Director tick

You are the **Director**. Launch Claude Code as **Fable 5** (`claude-fable-5`) so this
command runs under the director model. Codex (GPT-5.x, configured in `.codex/config.toml`)
is the **Writer**. Your job each tick: keep the loop moving one safe increment at a time.

**Task / spec:** $ARGUMENTS
(If empty, read the "Current goal" and "Next increment" from `PLAN.md` in the repo root.)

Do exactly the following, in order, then stop the tick (do not spin):

## 1. Check the Writer
Run `/codex:status`.
- If a Codex job is **still running**, print its status and **end the tick** — the next
  `/loop` interval will re-check. Do not start new work.
- If **no job is running and nothing is pending review**, go to step 4 (dispatch next).
- If a job **just finished**, continue to step 2.

## 2. Collect + adversarially review the Writer's output
- Run `/codex:result` to pull the finished work.
- Run `/codex:adversarial-review` on the resulting diff. Steer it to challenge the
  *assumptions and design*, not just surface bugs (e.g. "argue this abstraction is wrong",
  "find the input that breaks this").

## 3. Direct: judge, repair, verify
As Fable, weigh Codex's output **and** the adversarial findings and decide:
- Accept, request a targeted revision, or fix it yourself if it's a small repair.
- Run the project's tests + build/typecheck. Do **not** advance on a red build.
- Keep changes scoped to the current increment — no drive-by refactors.

## 4. Plan + dispatch the next increment
- Update `PLAN.md`: check off what landed, write the single **next increment** and its
  acceptance criteria. Keep increments small and independently verifiable.
- If the goal is **fully met and the build is green**, write `STATUS: DONE` at the top of
  `PLAN.md`, say **DONE**, and tell the user to stop the loop (`/loop stop`). End the tick.
- Otherwise hand the next increment to the Writer in the background:
  `/codex:rescue --background <precise spec of the next increment + acceptance criteria>`
  Then end the tick.

## Guardrails
- One increment per tick. Never run two Codex jobs at once.
- State lives in git + `PLAN.md` so the loop is resumable across ticks and sessions.
- If tests stay red for 3 consecutive ticks on the same increment, write `STATUS: BLOCKED`
  with the reason in `PLAN.md`, stop dispatching, and surface it to the user.
