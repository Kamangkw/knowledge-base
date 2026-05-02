---
title: Hermes Agent Multi-Agent Collaboration: delegate_task and Independent Processes
date: 2026-05-02
source: Self-research (hermes-agent SKILL.md, subagent-driven-development SKILL.md)
tags: [Hermes-Agent, Multi-Agent, delegate_task, Parallel-Processing]
description: Deep dive into Hermes Agent multi-agent collaboration - when to use delegate_task vs independent processes, effective subagent workflow design.
---

# Hermes Agent Multi-Agent Collaboration

## Two Core Patterns

| Dimension | delegate_task | Independent Hermes Process |
|-----------|---------------|--------------------------|
| Isolation | Separate conversation, shared process | Fully independent process |
| Duration | Minutes | Hours/days |
| Tool access | Parent process tools subset | Full tool access |
| Interactive | No | Yes (PTY mode) |
| Use case | Quick parallel subtasks | Long autonomous missions |

## delegate_task: Lightweight Parallel

delegate_task spawns a subagent within the current process. Good for parallel subtasks that complete quickly.

goal: Specific task description
context: Complete background including objectives, constraints, expected output, verification
toolsets: web, terminal, file (granted permissions)
role: leaf (default) or orchestrator (can delegate further)

### When to Use

Good for:
- Research multiple directions simultaneously
- Tasks that can be fully decomposed
- Need parent session context but isolated subtasks
- Batch processing similar items

Not good for:
- Tasks requiring user feedback (subagent cannot ask questions)
- Results needing verification (assume completed does not mean correctly completed)

### Key Principle: Complete Context

Subagent has NO parent session context. Must provide everything in context parameter.

Wrong: goal="Write an API", context="Use Flask"
Right: goal="Write REST API for user management", context="Project at ~/projects/user-service, Flask 2.x + SQLAlchemy, existing User model fields..."

## Independent Process (tmux)

Hermes uses prompt_toolkit, requires real terminal. Use tmux for PTY:

tmux new-session -d -s agent1 'hermes'
sleep 8
tmux send-keys -t agent1 'Build REST API' Enter
tmux capture-pane -t agent1 -p
tmux kill-session -t agent1

### Worktree Mode: Parallel Dev on Same Repo

hermes -w enables isolated git worktree mode, prevents git conflicts.

tmux new-session -d -s backend 'hermes -w'
tmux new-session -d -s frontend 'hermes -w'

## Two-Stage Review Pattern

For high-quality requirements:
1. Implementer subagent executes
2. Spec compliance reviewer checks
3. If failed - return to step 1
4. Code quality reviewer checks
5. If failed - fix and re-review
6. Complete

## Common Pitfalls

1. Incomplete context - subagent makes wrong assumptions
2. Assuming completed equals correctly completed - must verify
3. Over-fragmentation - every task uses subagent, overhead too high
4. Sequential dependency but parallelized - dependent tasks should not run in parallel
5. Parent session depends on subagent state - state disappears after subagent ends

---

*Last updated: 2026-05-02*
