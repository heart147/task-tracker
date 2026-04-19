---
name: task-tracker
description: Use when the user wants to start a new project, continue a previous task, ask where a task currently stands, resume multi-session work tracked in a project-root instruction.md file, or needs a strict cross-session execution checklist instead of loose planning.
---

# 任务清单

## Overview
Use this skill to manage long-running, cross-session project work through a single `instruction.md` file stored at the project root.

`instruction.md` is not a loose note. It is the authoritative execution ledger for the project: goals, scope boundaries, current phase, current step, blockers, validations, changed files, line ranges, and the one next step to execute.

This skill is for projects that span multiple conversations and must remain understandable even to an AI with no prior memory.

## When to Use
Use this skill when the user:
- says `继续任务 X`
- asks `X 任务进行到哪里了`
- asks to start a new project that should be tracked over multiple sessions
- wants work to continue from a previously maintained `instruction.md`
- needs a strict task ledger instead of a loose plan or casual todo list

## Core Rules
1. `instruction.md` in the project root is the single source of truth.
2. Always read `instruction.md` before continuing existing work.
3. Never infer project state from memory when `instruction.md` exists.
4. Every phase must end with a `[校验点]` step.
5. Every completed, warning, or blocked step must record changed files and line ranges.
6. Every step must have a clear goal, completion standard, and validation method.
7. Do not mark a step complete if validation is missing.
8. Do not continue past a blocked step unless the task list explicitly records the reason a later step can proceed safely.
9. Status markers must use these exact forms:
   - `🟢 [✔]` 完成
   - `🟡 [!]` 遇到问题/需注意
   - `🔴 [❌]` 错误/阻塞
   - `⚪ [ ]` 未开始

## State Transition Rules
Use statuses as a state machine, not decoration.

- `⚪ [ ] -> 🟢 [✔]`
  - only if execution finished, validation finished, and changed files/line ranges were recorded
- `⚪ [ ] -> 🟡 [!]`
  - if the main work is usable but has unresolved caveats, temporary workarounds, or follow-up risks
- `⚪ [ ] -> 🔴 [❌]`
  - if the step failed and blocks intended progress
- `🟡 [!] -> 🟢 [✔]`
  - only after the warning condition is resolved and revalidated
- `🔴 [❌] -> ⚪ [ ]`
  - only when the user explicitly wants a retry, rollback, or redesign

Never use `🟢 [✔]` to mean “attempted”, “partially done”, or “probably OK”.

## Task Splitting Rules
Break work until each step is operationally clear.

A good step:
- has one direct objective
- can be completed in one focused work unit
- has a clear completion standard
- has a clear validation method
- maps to concrete file changes or a concrete verification action

Split a step further if:
- it changes multiple unrelated modules
- it contains both implementation and debugging
- it contains both implementation and validation
- it cannot be judged complete from the written record
- it would require multiple independent decisions during execution

Bad step examples:
- `完成用户系统开发`
- `完善前端功能`
- `处理后端逻辑`

Good step examples:
- `新增登录请求类型定义`
- `实现登录 API 路由处理逻辑`
- `[校验点] 验证登录成功与失败分支`

## Hierarchy and Execution Order Rules
Treat nested numbering as a strict dependency hierarchy, not visual formatting.

Hierarchy meaning:
- `1` = phase or top-level task group
- `1.1` = child phase, feature block, or sub-task group under `1`
- `1.1.1` = executable leaf step under `1.1`
- `1.x.y [校验点]` = validation node for that level

Execution rules:
1. Only leaf nodes may be executed directly.
2. Parent nodes are summary nodes unless explicitly written as a checkpoint.
3. A parent node cannot be marked `🟢 [✔]` before all of its direct child nodes are `🟢 [✔]`.
4. If any direct child node is `🔴 [❌]`, the parent node must not be marked complete.
5. If child nodes are still `⚪ [ ]`, the parent node must remain incomplete.
6. If child nodes contain `🟡 [!]` but no unresolved blockers, the parent node may be `🟡 [!]` or remain incomplete, but not `🟢 [✔]` until the warning condition is intentionally accepted or resolved.
7. A level should normally end with a checkpoint node that validates the whole sibling group before the parent is considered complete.

Completion order example:
- complete `1.1.1`
- complete `1.1.2`
- complete `1.1.3 [校验点]`
- then mark `1.1` complete
- repeat for `1.2.*`
- complete `1.3 [校验点]`
- then mark `1` complete

Never skip upward in the hierarchy by marking `1.1` or `1` complete just because some lower-level work exists.

## Parent Status Derivation Rules
When a node has children, derive the parent status from child status.

Recommended derivation:
- all children `🟢 [✔]` -> parent may become `🟢 [✔]`
- any child `🔴 [❌]` -> parent should be `🔴 [❌]` or remain not complete, depending on whether the parent is blocked as a whole
- no `🔴 [❌]`, but one or more `🟡 [!]` -> parent should be `🟡 [!]` or remain incomplete
- any child `⚪ [ ]` -> parent must not be `🟢 [✔]`

If the task list uses both summary nodes and executable nodes at the same level, explicitly say so in the step text. Otherwise assume only leaf nodes are executable.

## Next Step Selection Rules
Apply these rules when deciding what to do next. Place this logic after hierarchy rules and before workflow because it controls resume behavior and execution order.

1. The default next step must be the earliest executable unfinished leaf node in document order.
2. Do not skip an earlier sibling leaf node just because a later node looks more interesting or easier.
3. Do not execute a parent summary node as the next step unless it is explicitly a checkpoint node.
4. Do not cross into the next sibling group if the current sibling group has unfinished executable children.
5. Do not cross a `[校验点]` node. If a checkpoint is the next pending executable node, it must be handled before later siblings or later phases.
6. If a node is blocked with `🔴 [❌]`, treat it as the current stopping point unless `instruction.md` explicitly states that later work may continue safely.
7. If a warning node `🟡 [!]` leaves follow-up work, prefer the documented follow-up before unrelated later work, unless the task list explicitly says the warning is intentionally deferred.
8. `下一步唯一入口` must match the actual earliest valid next executable node. If they disagree, reconcile the task list before proceeding.

Selection algorithm:
- start from the top of the progress list
- descend into the first unfinished branch
- continue descending until reaching the earliest unfinished executable leaf node or checkpoint node
- if that node is blocked, stop and report the blocker
- if that node is executable, propose it as the next step

Example ordering:
- if `1.1.1` is complete and `1.1.2` is pending, the next step is `1.1.2`
- not `1.1.3`, not `1.2.1`, not `1.3`
- if `1.1.2` is complete and `1.1.3 [校验点]` is pending, the next step is `1.1.3`
- only after `1.1.3` passes may `1.2.1` become the next step

When resuming, never choose the next step by intuition. Choose it by hierarchy order plus checkpoint order.

## Parallel Execution Exception Rules
Default behavior is sequential execution in hierarchy order. Parallel work is an exception, not the default.

Only allow parallel execution if all of the following are true:
1. the candidate steps are both leaf nodes
2. neither step depends on the output, validation, or state transition of the other
3. neither step crosses an unfinished checkpoint that should gate later work
4. the shared files, modules, or interfaces are unlikely to conflict
5. `instruction.md` explicitly records that the steps are parallel-safe

If any of the above is uncertain, do not parallelize.

Never parallelize:
- parent summary nodes
- checkpoint nodes with the work they validate
- two steps that modify the same file or tightly coupled interface unless the task list explicitly allows it
- later siblings when an earlier sibling is the logical prerequisite
- steps separated by an unresolved `🔴 [❌]` blocker

When parallel work is allowed:
- mark it explicitly in `instruction.md`
- keep each parallel step as its own leaf node
- add a later synchronization or checkpoint node before declaring the parent complete

Recommended notation:

```markdown
- ⚪ [ ] 步骤 2.1.1：[并行] 实现前端表单静态布局
- ⚪ [ ] 步骤 2.1.2：[并行] 定义后端请求参数类型
- ⚪ [ ] 步骤 2.1.3：[汇合校验点] 验证前后端接口字段一致
```

Parallel selection rule:
- if multiple pending leaf nodes are explicitly marked parallel-safe, they may be worked on in parallel
- otherwise choose the earliest valid next executable node in normal order

## Workflow

### Starting a New Project
When starting a new tracked project:
1. Clarify the project goal and requirements if needed.
2. Create `instruction.md` in the project root.
3. Write a detailed overall goal section including:
   - project background
   - project objective
   - core requirements
   - acceptance criteria
   - non-goals when useful
4. Break work into phases and numbered steps.
5. Add exactly one `[校验点]` step at the end of each phase.
6. Fill in the `当前阶段`, `当前步骤`, `当前阻塞`, and `下一步唯一入口` sections.
7. Ask the user to confirm before starting execution.

### Continuing Existing Work
When resuming:
1. Read `instruction.md` immediately.
2. Identify:
   - current overall goal
   - current phase
   - current step
   - last completed step
   - active warnings or blockers
   - next executable step
3. Reply in Chinese using this structure:

```markdown
## 当前任务状态
- **任务：**
- **总目标：**
- **当前阶段：**
- **当前步骤：**
- **最近完成：**
- **当前问题：**
- **下一步建议：**

是否按照任务清单开始执行 `步骤 X.X`？
```

### Updating Progress
After finishing or attempting a step:
1. Update the step status immediately.
2. Record changed file paths and line ranges.
3. Record change type when useful: 新增 / 修改 / 删除.
4. Record notes, warnings, or blocker details.
5. Update `当前步骤`, `当前阻塞`, `最近一次修改摘要`, and `下一步唯一入口` if they changed.
6. If a step is marked `🟢 [✔]`, create a git commit for that completed step before moving to the next step.
7. The commit should include only the files relevant to that step or its immediately required task-record update.
8. The task record update in `instruction.md` should be included in the same commit as the completed step whenever practical.
9. If the step is `🟡 [!]` or `🔴 [❌]`, do not auto-commit unless the user explicitly wants checkpoint commits for incomplete or blocked work.
10. Do not leave finished work undocumented.

### Commit Boundary Rules
Use git commits as step boundaries.

- One completed step should normally correspond to one commit.
- Do not bundle multiple unrelated completed steps into one commit unless `instruction.md` explicitly marks them as one atomic delivery unit.
- Do not mark a step `🟢 [✔]` and then continue implementation for later steps before committing the finished step.
- If validation is required for a step, commit only after validation passes.
- If a checkpoint step completes, that checkpoint may have its own commit if it represents a meaningful boundary.
- Commit messages should describe the completed step outcome, not a vague phase summary.
- The commit message should normally include the step number, the step name, and a short outcome summary.

Recommended commit message pattern:
- `step 1.1.1: define login request types`
- `step 1.1.2: implement login route handler`
- `step 1.1.3: validate login success and failure flow`

If a step name is already precise, keep the message concise rather than rewriting it into a broader summary.

Example:
- complete `1.1.1`
- update `instruction.md`
- commit files for `1.1.1`
- then continue to `1.1.2`

### Validation Checkpoints
A checkpoint may use one or more of these validation types:
- 结构校验
- 静态校验
- 运行校验
- 功能校验
- 回归校验
- 人工校验

Do not assume tests already exist.
Choose the most appropriate validation method for the current phase.

A checkpoint is only complete when its pass condition is explicitly written and actually checked.

### Blocker Handling
If a step fails:
1. Do not silently continue.
2. Mark the step `🔴 [❌]` if it blocks intended progress.
3. Record:
   - the failure symptom
   - trigger conditions
   - likely cause if known
   - affected scope
   - whether later work can safely continue
4. If progress can continue safely, create or update a later step explaining the dependency break clearly.
5. If progress cannot continue, make the blocker explicit in `当前阻塞` and `下一步唯一入口`.

## Required instruction.md Format

```markdown
# 任务：[项目/任务名称]

## 项目元信息
- **任务名：** [项目/任务名称]
- **当前状态：** [进行中 / 阻塞 / 已完成]
- **当前阶段：** [例如：阶段 2 - 用户认证]
- **当前步骤：** [例如：步骤 2.2 / 步骤 2.1.3]
- **优先级：** [高 / 中 / 低]
- **创建时间：** [YYYY-MM-DD]
- **最后更新时间：** [YYYY-MM-DD HH:mm]

## 🎯 总目标
- **项目背景：** [为什么要做这个项目]
- **项目目标：** [最终要实现什么]
- **核心需求：**
  1. [需求 1]
  2. [需求 2]
- **非目标：**
  1. [本轮不做什么，可选]
- **验收标准：** [如何判断项目完成]

## ⚠ 当前阻塞
- [无 / 具体阻塞说明]

## 📝 最近一次修改摘要
- [最近一次完成或尝试的步骤]
- [关键改动]
- [是否已验证]

## ⏭ 下一步唯一入口
- **推荐执行：** [步骤 X.X / X.X.X]
- **执行动作：** [一句话说明下一步要做什么]
- **执行前检查：** [执行前需要确认什么]

## 📋 进度清单
*状态图例：🟢 [✔] 完成 | 🟡 [!] 遇到问题/需注意 | 🔴 [❌] 错误/阻塞 | ⚪ [ ] 未开始*

### 阶段 1：[阶段名称]
- ⚪ [ ] 步骤 1.1：[子阶段/功能块汇总节点，不直接执行]
  > **目标：** [这一组子步骤整体要达成什么]
  > **子步骤完成条件：** [例如：1.1.1~1.1.3 全部完成]
  > **注释：** 待定

  - ⚪ [ ] 步骤 1.1.1：[可执行原子步骤]
    > **目标：** [这一小步要达成什么]
    > **前置条件：** [执行前必须满足什么]
    > **执行动作：** [本步要做什么]
    > **变动文件：** 待定
    > **验证方式：** [如何验证]
    > **完成标准：** [满足什么才算完成]
    > **注释：** 待定

  - ⚪ [ ] 步骤 1.1.2：[可执行原子步骤]
    > **目标：** [这一小步要达成什么]
    > **前置条件：** [执行前必须满足什么]
    > **执行动作：** [本步要做什么]
    > **变动文件：** 待定
    > **验证方式：** [如何验证]
    > **完成标准：** [满足什么才算完成]
    > **注释：** 待定

  - ⚪ [ ] 步骤 1.1.3：[校验点] [验证 1.1 整体成果]
    > **校验类型：** [结构校验 / 静态校验 / 运行校验 / 功能校验 / 回归校验 / 人工校验]
    > **变动文件：** 待定
    > **验证方式：** [具体如何验证]
    > **通过标准：** [什么结果算通过]
    > **注释：** 待定

- ⚪ [ ] 步骤 1.2：[子阶段/功能块汇总节点，不直接执行]
  > **目标：** [这一组子步骤整体要达成什么]
  > **子步骤完成条件：** [例如：1.2.1~1.2.3 全部完成]
  > **注释：** 待定

- ⚪ [ ] 步骤 1.3：[校验点] [验证阶段 1 整体成果]
  > **校验类型：** [结构校验 / 静态校验 / 运行校验 / 功能校验 / 回归校验 / 人工校验]
  > **变动文件：** 待定
  > **验证方式：** [具体如何验证]
  > **通过标准：** [什么结果算通过]
  > **注释：** 待定
```

## Change Recording Rules
When recording changed files, prefer this structure:

```markdown
> **变动文件：**
> - `src/auth/login.ts` (行 12-48) - 修改 - 新增登录逻辑
> - `src/auth/types.ts` (行 1-10) - 新增 - 定义登录请求类型
```

If no file changed because the step was purely verification, explicitly say so.

## Resume Behavior
Never guess progress from memory alone.
Always use the current `instruction.md` contents as the authoritative project state.
If the file and current code reality conflict, trust current observed reality, then update `instruction.md`.

## Common Mistakes
- Continuing work without reading `instruction.md`
- Updating code without updating the task record
- Recording vague file changes without line ranges
- Marking a step complete without validation
- Skipping a phase checkpoint
- Assuming only automated tests count as validation
- Writing steps so large that completion cannot be judged objectively
- Proceeding past a blocker without explicitly recording why it is safe
