# Task Tracker

A Claude Code Skill for managing long-running, cross-session project tasks.

## 📖 Overview

This Skill manages project work across multiple conversation sessions through a single `instruction.md` file at the project root.

`instruction.md` is not a loose note. It is the **authoritative execution ledger** for the project, containing:
- Project goals and scope boundaries
- Current phase and step
- Blockers
- Validations
- Changed files with line ranges
- The one next step to execute

## 🚀 When to Use

Use this Skill when you:
- Want to continue a previous task (`继续任务 X`)
- Need to check task progress (`X 任务进行到哪里了`)
- Start a new project that spans multiple sessions
- Want to resume work from an existing `instruction.md`
- Need a strict task ledger instead of a loose plan

## 📋 Core Features

### Status Marker System
- 🟢 `[✔]` Complete
- 🟡 `[!]` Warning/Issue
- 🔴 `[❌]` Error/Blocked
- ⚪ `[ ]` Not Started

### Strict Hierarchy Execution Order
- Only leaf nodes are directly executable
- Parent node status derived from children
- Checkpoint `[校验点]` must pass before moving to next phase

### Change Tracking

Every completed step must record:
- Changed file paths
- Line ranges
- Change type (added/modified/deleted)

### Git Commit Boundaries
- One completed step = one commit
- Commits should include only step-related files
- Task record updates should be in the same commit

## 📄 instruction.md Template

For the complete template format, see [SKILL.md](SKILL.md) under "Required instruction.md Format".

## 📚 Documentation

- **Detailed Spec**: [SKILL.md](SKILL.md)
- **Chinese README**: [README.md](README.md)

## 🛠 Installation

This Skill is published on GitHub:
```
https://github.com/heart147/task-tracker
```

## 📝 Example

```markdown
## 📋 进度清单
*Status Legend: 🟢 [✔] Complete | 🟡 [!] Warning | 🔴 [❌] Error/Blocked | ⚪ [ ] Not Started*

### Phase 1: Project Initialization
- ⚪ [ ] Step 1.1: Requirements Analysis
  > **Goal:** Clarify project requirements and boundaries
  > **Action:** Analyze user requirement documents
  > **Validation:** Output requirements list
  > **Completion Criteria:** Requirements list confirmed
```

## 📄 License

MIT
