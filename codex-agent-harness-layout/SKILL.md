---
name: codex-agent-harness-layout
description: Create or maintain a Codex-style multi-agent harness layout for a project. Use when the user asks to generate, install, enforce, update, or reuse Agent.md, AGENTS.md, .codex/config.toml, .codex/agents/*.toml, every_tasks_harness/*.md, or a planer/explorer/implementer/reviewer/tester/verifier workflow with a feedback repair loop. Keeps business code unchanged unless explicitly requested, uses reviewer as the review role name, and adds robot or hardware safety rules when a project involves chassis, motors, navigation, ROS, TCP robot control, or real devices.
---

# Codex Agent Harness Layout

## Purpose

Create a reusable project-local harness that makes Codex follow this initial role order:

```text
planer -> explorer -> implementer -> reviewer -> tester -> verifier
```

The harness is a control layer for agent behavior. It should not change business code, robot-control code, application code, tests, or runtime configuration unless the user explicitly asks for those changes.

After the initial `implementer` pass, `reviewer`, `tester`, and `verifier` are quality gates. They report findings to the main agent. They do not fix files themselves. The main agent sends actionable findings back to `implementer`, then reruns `reviewer -> tester -> verifier`.

## Target Layout

Create or maintain these files at the target project root:

```text
AGENTS.md
Agent.md
.codex/config.toml
.codex/agents/planer.toml
.codex/agents/explorer.toml
.codex/agents/implementer.toml
.codex/agents/reviewer.toml
.codex/agents/tester.toml
.codex/agents/verifier.toml
every_tasks_harness/planer.md
every_tasks_harness/explorer.md
every_tasks_harness/implementer.md
every_tasks_harness/reviewer.md
every_tasks_harness/tester.md
every_tasks_harness/verifier.md
```

Keep the role names exactly as listed. Use `reviewer` exactly for the review role.

## Workflow

1. Inspect the target root before writing.
   - If `AGENTS.md`, `Agent.md`, `.codex/config.toml`, or matching harness files already exist, preserve existing unrelated guidance and merge the harness section.
   - If an existing file conflicts with the required role order, report the conflict and make the smallest safe update.
   - Do not overwrite user-specific project instructions.

2. Create the harness directories and files.
   - Keep all per-role markdown instructions Chinese-first.
   - Do not leave empty sections, generic unfinished markers, or unfinished template text.
   - Use executable guidance: state inputs, responsibilities, forbidden actions, output format, and verification rules.

3. Add project-level enforcement.
   - `AGENTS.md` must say Codex must read `Agent.md`, `.codex/config.toml`, `.codex/agents/*.toml`, and `every_tasks_harness/*.md` before project work.
   - `AGENTS.md` must forbid creating a different harness, skipping roles, or claiming completion before `verifier`.
   - `Agent.md` must describe the role order, handoff expectations, and final reporting requirements.

4. Keep implementation conservative.
   - Only `implementer` should default to write-capable behavior.
   - `planer`, `explorer`, `reviewer`, `tester`, and `verifier` default to read-only behavior.
   - If Codex has no actual subagent execution tool in the current environment, instruct the main agent to simulate the same sequence in one thread and record each role's output clearly.

5. Use the repair loop whenever quality gates find issues.
   - `reviewer`, `tester`, and `verifier` must report issues to the main agent only.
   - The main agent must summarize the issue list, decide whether each issue is in scope, and send in-scope fixes to `implementer`.
   - `implementer` must make the smallest task-scoped fix, then return changed files and rationale to the main agent.
   - After each implementer fix pass, rerun `reviewer -> tester -> verifier`.
   - Stop only when all three quality gates pass, the user stops the work, or three repair cycles have failed to clear the same blocking issue.
   - If the same blocking issue remains after three repair cycles, stop and report the repeated issue, evidence, attempted fixes, and recommended next action.

## Required Config

Ensure `.codex/config.toml` contains this table. If the file already has other settings, add or update only this table.

```toml
[agents]
max_threads = 6
max_depth = 1
job_max_runtime_seconds = 1800
```

Create six agent TOML files using this shape:

```toml
name = "role-name"
description = "One sentence describing the role."
model_reasoning_effort = "medium"
sandbox_mode = "read-only"
developer_instructions = "Read Agent.md, AGENTS.md, and every_tasks_harness/role-name.md before doing this role. Follow the required harness order and return concise Chinese findings."
```

Use these role defaults:

| File | name | sandbox_mode | model_reasoning_effort |
| --- | --- | --- | --- |
| `.codex/agents/planer.toml` | `planer` | `read-only` | `high` |
| `.codex/agents/explorer.toml` | `explorer` | `read-only` | `medium` |
| `.codex/agents/implementer.toml` | `implementer` | `workspace-write` | `high` |
| `.codex/agents/reviewer.toml` | `reviewer` | `read-only` | `high` |
| `.codex/agents/tester.toml` | `tester` | `read-only` | `medium` |
| `.codex/agents/verifier.toml` | `verifier` | `read-only` | `high` |

Each `developer_instructions` value must explicitly point to the matching `every_tasks_harness/<role>.md`.

## Required Project Guidance

Add a compact `AGENTS.md` section, merging with existing content if needed. The generated text must be Chinese-first and must include these requirements:

- The project must use the repository-local Codex harness workflow.
- Before code edits, debugging, testing, or documentation tasks, Codex must read `Agent.md`, `.codex/config.toml`, `.codex/agents/*.toml`, and `every_tasks_harness/*.md`.
- The required order is `planer -> explorer -> implementer -> reviewer -> tester -> verifier`.
- Codex must not skip `Agent.md`.
- Codex must not create a different harness, role order, or directory layout.
- Codex must use `reviewer` as the review role name.
- `reviewer`, `tester`, and `verifier` must report issues to the main agent and must not edit files.
- When those quality gates find in-scope issues, the main agent must ask `implementer` to fix them, then rerun `reviewer -> tester -> verifier`.
- Codex must not claim completion before `reviewer`, `tester`, and `verifier` all pass.
- If harness files are missing, conflicting, or unreadable, Codex must stop and report the issue instead of inventing a replacement workflow.

For robot, chassis, motor, navigation, ROS, TCP-control, or real-hardware projects, add a Chinese-first robot safety section to `AGENTS.md`, `Agent.md`, and relevant role docs. It must require:

- Check emergency stop, charging state, navigation state, control mode, and connection state before real movement.
- Real robot validation must use low speed, small range, and an interruptible procedure.
- Any movement test must state the stop method.
- The task must end by sending a stop command or explicitly confirming that the control link is stopped.
- Do not increase speed, distance, angular velocity, or motion range until on-site safety is confirmed.

## Role Documents

Each file under `every_tasks_harness/` must be Chinese-first and must include these concepts:

- `planer.md`: clarify goal, scope, risk, role sequence, file boundaries, and acceptance criteria before exploration or edits.
- `explorer.md`: inspect real repo state, identify entry points and tests, collect evidence, and avoid modifications.
- `implementer.md`: make only task-scoped edits, preserve unrelated user changes, record changed files, fix in-scope issues assigned by the main agent, and keep robot actions simulated unless explicitly authorized.
- `reviewer.md`: review for bugs, safety issues, behavioral regressions, missing tests, and harness-rule violations; report findings to the main agent without editing files.
- `tester.md`: run or specify focused checks, report exact commands and outcomes to the main agent, and avoid real hardware movement unless explicitly authorized.
- `verifier.md`: confirm the harness sequence and repair loop ran, required files exist, no unfinished marker text remains, safety rules are present when needed, and final report is accurate.

Each role document should have Chinese headings equivalent to:

```text
role execution constraints
role responsibilities
must read first
allowed actions
forbidden actions
output format
verification requirements
```

The output format for each role should be short Chinese markdown with evidence, not long raw logs.

## Agent.md Contents

`Agent.md` must be the main orchestration document. It should state:

- The main agent is the coordinator.
- The required order is `planer -> explorer -> implementer -> reviewer -> tester -> verifier`.
- Each role must read its own `every_tasks_harness/<role>.md`.
- Only `implementer` may make task-scoped file edits by default.
- `reviewer`, `tester`, and `verifier` must report risks to the main agent and must not edit files.
- The main agent must send in-scope quality-gate findings back to `implementer`, then rerun `reviewer -> tester -> verifier`.
- The repair loop stops only when all three quality gates pass, the user stops the work, or the same blocking issue remains after three repair cycles.
- The final answer must include changed files, verification commands, results, and remaining risks.

For hardware projects, `Agent.md` must repeat the safety requirements and require a final stop command or explicit confirmation that no motion command was sent.

## Verification

After generating or updating the harness, run the strongest safe checks available:

```powershell
Test-Path .\AGENTS.md
Test-Path .\Agent.md
Test-Path .\.codex\config.toml
Get-ChildItem .\.codex\agents\*.toml
Get-ChildItem .\every_tasks_harness\*.md
$unfinishedPattern = 'TO' + 'DO|TB' + 'D|place' + 'holder'
rg -n $unfinishedPattern AGENTS.md Agent.md .codex every_tasks_harness
git diff --check
```

If the target is not a Git repository, skip `git diff --check` and say why.

Report:

- Files created or updated.
- Whether all six agent TOML files exist.
- Whether each TOML points to the matching role doc.
- Whether the unfinished-marker scan is clean.
- Whether robot safety rules were included or not needed.

## Distribution Notes

If the user asks how to share this skill:

- For local personal use, place it under the user's skill directory.
- For repo-scoped use, place a copy under `.agents/skills/codex-agent-harness-layout/`.
- For GitHub installation on another machine, publish `skills/codex-agent-harness-layout/SKILL.md` and use `$skill-installer` with the GitHub tree URL.
- For broader Codex plugin distribution, wrap the skill in a plugin that contains `skills/codex-agent-harness-layout/SKILL.md` and `.codex-plugin/plugin.json`.
