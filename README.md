# handoff-pickup

Three [Agent Skills](https://agentskills.io) for carrying work across agent sessions and workers.

- **handoff**: at the end of a work stage, writes `docs/handoff/CURRENT.md`, a short, verified briefing for the next session.
- **pickup**: at the start of a new session, reads that file, reconciles it against the repository, recaps, and waits for your go-ahead.
- **dispatch**: hands one subtask to a worker (subagent, CLI, or IDE) through `docs/dispatch/task-NNN.md`, and later checks the worker's report in `task-NNN.result.md` against the repository.

Plain Markdown, no scripts. They work with any agent that supports the `SKILL.md` format and can run `git`.

## Why

A new session has no memory of the last one. Summaries written from memory drift, and stale summaries are worse than none. These skills treat the handoff as a disciplined document:

- **Verified, not remembered.** Every state claim is marked `[verified]` (with command and result) or `[unverified]` (with the command that would verify it). Branch, commit, and time come from live `git` and `date` output.
- **Nothing silently lost.** Each new handoff must account for every constraint, pitfall, and open question in the previous one: carry it forward, drop it (and say so), or propose promoting it to `CLAUDE.md` / `AGENTS.md` / design docs.
- **Your words, verbatim.** User constraints are quoted, not paraphrased. Rejected approaches are recorded with reasons so they are not proposed again.
- **A lead, not a fact.** pickup checks the handoff against `git log <base_commit>..HEAD`, the current branch, and the worktree, re-verifies what the first step depends on, and goes with the repository when they disagree.
- **No inherited permissions.** Approval to commit, push, or make paid calls in an earlier session does not carry over.
- **Small.** At most 150 lines; it points to `path:line` and commit hashes instead of copying code.

## Install

Clone the repository, then either copy or symlink the three directories into your agent's skills directory. The examples below include Claude Code (`~/.claude/skills`) and Codex (`~/.agents/skills`); use the commands labeled for your agent. Run all commands from the parent directory of the clone; if you already cloned the repository, skip `git clone`.

### macOS / Linux (Bash)

```bash
git clone https://github.com/blizzardwj/handoff-pickup.git

# Claude Code
mkdir -p ~/.claude/skills

# Codex
mkdir -p ~/.agents/skills
```

Then either copy them:

```bash
# Claude Code
cp -r handoff-pickup/handoff handoff-pickup/pickup handoff-pickup/dispatch ~/.claude/skills/

# Codex
cp -r handoff-pickup/handoff handoff-pickup/pickup handoff-pickup/dispatch ~/.agents/skills/
```

or symlink them, so that `git pull` in the clone updates the installed skills:

```bash
# Claude Code
ln -s "$PWD/handoff-pickup/handoff" ~/.claude/skills/handoff
ln -s "$PWD/handoff-pickup/pickup" ~/.claude/skills/pickup
ln -s "$PWD/handoff-pickup/dispatch" ~/.claude/skills/dispatch

# Codex
ln -s "$PWD/handoff-pickup/handoff" ~/.agents/skills/handoff
ln -s "$PWD/handoff-pickup/pickup" ~/.agents/skills/pickup
ln -s "$PWD/handoff-pickup/dispatch" ~/.agents/skills/dispatch
```

### Windows (PowerShell)

Clone the repository and create the skills directory:

```powershell
git clone https://github.com/blizzardwj/handoff-pickup.git

# Claude Code
New-Item -ItemType Directory -Path "$HOME\.claude\skills" -Force | Out-Null

# Codex
New-Item -ItemType Directory -Path "$HOME\.agents\skills" -Force | Out-Null
```

Then either copy the three directories (separate multiple source paths with a comma in PowerShell):

```powershell
# Claude Code
Copy-Item -Path .\handoff-pickup\handoff, .\handoff-pickup\pickup, .\handoff-pickup\dispatch -Destination "$HOME\.claude\skills" -Recurse

# Codex
Copy-Item -Path .\handoff-pickup\handoff, .\handoff-pickup\pickup, .\handoff-pickup\dispatch -Destination "$HOME\.agents\skills" -Recurse
```

or create symbolic links, so that `git pull` in the clone updates the installed skills. Use this option when the destination `handoff`, `pickup`, and `dispatch` paths do not already exist:

```powershell
$repoPath = (Resolve-Path .\handoff-pickup).Path

# Claude Code
New-Item -ItemType SymbolicLink -Path "$HOME\.claude\skills\handoff" -Target "$repoPath\handoff"
New-Item -ItemType SymbolicLink -Path "$HOME\.claude\skills\pickup" -Target "$repoPath\pickup"
New-Item -ItemType SymbolicLink -Path "$HOME\.claude\skills\dispatch" -Target "$repoPath\dispatch"

# Codex
New-Item -ItemType SymbolicLink -Path "$HOME\.agents\skills\handoff" -Target "$repoPath\handoff"
New-Item -ItemType SymbolicLink -Path "$HOME\.agents\skills\pickup" -Target "$repoPath\pickup"
New-Item -ItemType SymbolicLink -Path "$HOME\.agents\skills\dispatch" -Target "$repoPath\dispatch"
```

Creating symbolic links on Windows may require running PowerShell as Administrator. PowerShell 7 with Windows Developer Mode enabled can create them without elevation. Copying does not require this.

If your PowerShell session is already in the cloned repository, use `.\handoff, .\pickup, .\dispatch` as the copy source paths, or `(Resolve-Path .).Path` for `$repoPath` when creating links.

For other agents, use the skills directory described in their documentation.

**Claude Code tip:** all three skills are meant to run only when you ask for them. To enforce that, add `disable-model-invocation: true` to the frontmatter of each `SKILL.md`.

## Usage

At the end of a stage, invoke the handoff skill, optionally with what the next stage should be (in Claude Code: `/handoff next: add retry logic to the uploader`). Review the file, edit it if needed, and commit it yourself; the skill never commits.

In a new session, invoke the pickup skill (`/pickup`). It reports the goal, the current state (including anything that changed since the handoff), and its planned first step, then waits for you.

To hand one subtask to a worker while keeping the current session in charge, invoke the dispatch skill (`/dispatch`). It writes `docs/dispatch/task-NNN.md` and gives you the instruction for the worker: `Read docs/dispatch/task-NNN.md and do what it says.` When the worker finishes and writes `docs/dispatch/task-NNN.result.md`, invoke dispatch again to check the changes and acceptance criteria.

## The handoff file

One file per repository, `docs/handoff/CURRENT.md`, overwritten each stage. History lives in git: `git log -p docs/handoff/CURRENT.md`.

Frontmatter: `date`, `status` (`stage-complete` | `in-progress`), `branch`, `base_commit`, `worktree` (`clean` | `dirty`).

Sections: 1 Goal · 2 Current state · 3 Done this stage · 4 Decisions and rationale · 5 Pitfalls and constraints · 6 Next stage · 7 Open questions for the user · 8 Read first.

## 中文说明

三个配套的 Agent Skills：阶段结束时用 handoff 写一份经过核实的交接记录（`docs/handoff/CURRENT.md`），新会话开始时用 pickup 读取、与仓库对账、复述后等你确认再动手；当前会话保持主导时用 dispatch 把单个子任务写进 `docs/dispatch/task-NNN.md` 交给其他 worker（IDE、CLI 或 subagent），完成后读取 `task-NNN.result.md` 核对改动与验收结果。文档按用户在对话中使用的语言书写，中文对话会得到中文交接与任务单。安装方法见上文 Install。

## License

[MIT](LICENSE)
