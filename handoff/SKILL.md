---
name: handoff
description: Writes a stage handoff to docs/handoff/CURRENT.md so that a fresh agent session can resume the work with the pickup skill. Use only when the user explicitly asks for a handoff at the end of a work stage.
license: MIT
compatibility: Requires a git repository.
---

# Write a stage handoff

The user invokes this at the end of a work stage. Write a handoff that lets a brand-new session continue the work by reading this one file. That session cannot see this conversation, only the repository, so the handoff is its only source of what happened before. Wrong content does more harm than missing content, because the next session will act on it.

## Next stage

Any instructions the user gave when invoking this skill describe the next stage. If there are none, infer the next stage from the conversation and ask the user to confirm it in section 7 of the handoff.

## File location

`docs/handoff/CURRENT.md` at the repository root, one per repository. A new handoff overwrites the previous one; earlier versions live in git history (`git log -p docs/handoff/CURRENT.md`).

This skill modifies no other file.

## Steps

1. **Record live facts.** Run these commands. Their output is the source of truth for the branch, commit hashes, uncommitted files, and time. Never write these from memory.
   - `git status --short --branch`: branch and uncommitted changes.
   - `git log --oneline -8`: recent commits; the first line is HEAD.
   - `date +%Y-%m-%dT%H:%M%z`: current time.

2. **Verify state.** Confirm that this stage's results hold right now.
   - Checks run in this session after the last relevant change can be cited directly, with the command and its result.
   - For anything not yet verified, run only fast, local, free checks, such as the relevant unit tests.
   - Do not run commands that cost money, take long, or have external effects, such as paid APIs, deployments, or sending messages. Mark the corresponding claims [unverified] and write down the command that would verify them, for the next session.

3. **Process the previous handoff.** If `docs/handoff/CURRENT.md` exists, read it first. Every constraint, pitfall, open question, and unfinished step in it must go somewhere:
   - Still valid: carry it into the new handoff. Copy constraints and user requirements verbatim.
   - Now long-term knowledge (project conventions, settled design): still carry it forward, and in your report suggest that the user move it into CLAUDE.md, AGENTS.md, or a design document. Do not edit those files yourself.
   - Obsolete or done: drop it, and list it in your report.

   The next session will not read the old handoff. Anything not carried forward no longer exists for it.

   Skip this step if `CURRENT.md` does not exist.

4. **Overwrite `CURRENT.md`.** Follow the template and writing rules below.

5. **Self-check.**
   - At most 150 lines. If it is longer, some content belongs in a design document; keep only the conclusion and a link here.
   - Every item in section 2 is marked [verified] or [unverified], and every [verified] item has its command and result.
   - The first step in section 6 names the file and the command, so the next session can start without asking questions.
   - `branch`, `base_commit`, and `worktree` in the frontmatter match the live facts from step 1.

6. **Report, then stop.** Tell the user:
   - The path and line count of the new handoff.
   - If the `git status` from step 1 showed `docs/handoff/CURRENT.md` as untracked or modified: the previous version was never committed and has now been overwritten, so its original text is not in git history.
   - Which items from the previous handoff were dropped.
   - Which items you suggest moving into long-term documentation.
   - The questions in section 7 that await the user's answer.
   - A reminder if there are uncommitted code changes.

   Do not commit. The user reviews and edits the handoff, then decides when to commit it.

   If this stage's conversation included background explanations meant for the user personally, mention at the end of the report that they may want to save them in their own notes. That content is for humans and does not go into the handoff.

## Template

```markdown
---
date: 2026-10-13T17:40+0800
status: stage-complete        # stage-complete | in-progress (interrupted before the stage was done)
branch: feat/tts-prosody-control
base_commit: d6923be          # HEAD when the handoff was written
worktree: clean               # clean | dirty (if dirty, list the uncommitted files in section 2)
---

# Handoff: <this stage in one sentence> → <next stage in one sentence>

## 1. Goal
What the whole task is meant to achieve and what counts as done. 1 to 3 lines. If the goal has not changed, copy it from the previous handoff.

## 2. Current state
What is true right now. Mark each item [verified] (with the command and its result) or [unverified] (with the command that would verify it).
For half-finished work, state which file and line it stopped at and what is still missing.

## 3. Done this stage
At most 5 items. Each points to a commit hash or file; do not retell the process.

## 4. Decisions and rationale
Only decisions that constrain later work: what was chosen, what was rejected, and why.
If a design document covers it, keep a one-line conclusion and a link.

## 5. Pitfalls and constraints
Approaches that were tried and failed, and why they failed. Things that must not be touched.
Quote the user's explicit requirements verbatim; do not paraphrase.

## 6. Next stage
Ordered steps. The first step names the file to change, the line to start from, and the command to run.
End with the completion criteria.

## 7. Open questions for the user
Only questions the user must answer. If there are none, write "None".

## 8. Read first
At most 5 files or documents (path:line), plus the commands that confirm the current state.
```

When `status` is `in-progress`, focus on the half-finished part of section 2: what was being changed, how far it got, and what the next action is.

## Writing rules

1. The reader is an agent that has not seen this conversation but can access the repository. It cannot resolve shorthand or references like "the approach from earlier"; spell everything out.
2. Point, don't copy. Do not paste code, diffs, or long output; use `path:line` and commit hashes.
3. Keep exact things exact: commands, paths, error messages, numbers, and the user's constraints.
4. Give every decision its rationale and the rejected alternatives, or the next session will propose the rejected option again.
5. Write only what affects the next stage. No conversation history, no commentary, nothing that can be read directly from the code.
6. Write in the language the user uses in the conversation. Keep the section numbers, because the pickup skill refers to sections by number.
