---
name: pickup
description: Resumes work in a fresh session from the stage handoff in docs/handoff/CURRENT.md written by the handoff skill. Reconciles it against the repository, recaps the goal, state, and first step, then waits for confirmation. Use only when the user explicitly asks to pick up or resume from a handoff.
license: MIT
compatibility: Requires a git repository.
---

# Pick up the previous stage

The user invokes this at the start of a new session. At the end of the previous stage, an earlier session wrote a handoff with the handoff skill; use it to recover what happened before.

The handoff is a statement by the previous session, and the repository may have changed since it was written. Treat it as a lead, not as fact: when it contradicts the code, git, or test results, the latter win.

Any instructions the user gave when invoking this skill are additional context for this session.

## Steps

1. **Record live facts.** Run these commands and use their output as the current state:
   - `git status --short --branch`: branch and uncommitted changes.
   - `git log --oneline -8`: recent commits; the first line is HEAD.

2. **Read the handoff.** Read `docs/handoff/CURRENT.md` at the repository root.
   - Earlier handoffs live in git history; look at them only if the user asks for a retrospective.
   - If the file does not exist, tell the user and stop.

3. **Reconcile.** Compare the handoff's frontmatter with the live facts:
   - Does `branch` match the current branch?
   - Has anything changed since `base_commit`? The handoff itself is usually one commit after `base_commit` that touches only `docs/handoff/`; that is expected. Inspect with `git log --oneline <base_commit>..HEAD` and `git diff --stat <base_commit>..HEAD`.
   - Does the `worktree` field match the current uncommitted changes?

   Where they differ, find out what changed, go by the current state of the repository, and tell the user in your recap.

4. **Read what is needed.** Read the files listed in section 8 ("Read first") and run the commands it lists for confirming the current state.
   - Read only these for now; read anything else on demand once you start working.
   - Do not run commands that cost money, take long, or have external effects yet; ask the user about them in your recap.

5. **Check the claims you depend on.** Among the claims that the first step of the next stage depends on, verify those marked [unverified] first. Re-verify those marked [verified] if reconciliation shows that the relevant code changed after the handoff.

6. **Recap, then wait for confirmation.** In three lines:
   - Goal: what the whole task is meant to achieve.
   - State: where things stand now, including any differences found during reconciliation.
   - First step: what you plan to do first.

   Then list the questions from section 7 ("Open questions for the user"). Start making changes only after the user confirms.

## Rules

- Do not modify `CURRENT.md`. It is the record of the previous stage; at the end of this stage, the handoff skill overwrites it with a new one.
- Sections 4 and 5 record rejected approaches and known pitfalls. Do not propose a rejected approach again unless there is new information the handoff did not consider; in that case, explain it to the user first.
- Constraints and preferences the user stated in section 5 remain in force. However, permissions the user granted for specific actions in the previous session do not carry over, such as committing, pushing, or paid API calls. Ask again when needed.
