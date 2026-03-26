---
name: next
description: Maintain and resume a compact session handoff anchored only on `plans/next.md`. Use only when the user explicitly invokes `/next`, explicitly asks to save/update the next handoff, or explicitly asks to resume from the saved handoff.
---

# /next

Use [`plans/next.md`](./plans/next.md) as the only handoff file.

## Trigger contract

- Treat `/next + 描述` as the primary invocation.
- Only enter this workflow when the user explicitly triggers `/next`, explicitly asks to write/update `plans/next.md`, or explicitly asks to resume from it.
- Do not proactively read `plans/next.md` during ordinary task execution.
- Do not proactively write or refresh `plans/next.md` mid-task, at session end, or just because the work state changed.
- If the user starts a new session and explicitly asks to continue from `/next`, read `plans/next.md` first, align with the new description, then proceed.
- If the user only discusses future work or continuation in general without explicitly invoking this workflow, do not read or write `plans/next.md`.

## Hard rules

1. Only use `plans/next.md` for this workflow.
2. Do not create extra handoff, summary, review, todo, or memory files.
3. Keep the note compact and execution-oriented.
4. Do not dump full conversation history.
5. Do not write guesses as conclusions.
6. Explicit user intent is required for every read or write of `plans/next.md`.

## Record workflow

When the user wants to save the current state:

1. Read `plans/next.md` if it exists.
2. Rewrite it so it reflects only the latest valid continuation state.
3. Keep these sections and nothing more unless the task truly needs extra structure:
   - `任务锚点`
   - `当前结论`
   - `已完成`
   - `未完成`
   - `关键文件`
   - `直接命令`
   - `下一步第一动作`
   - `状态口径`
4. If the user gives a narrower description, prune unrelated old content.
5. After writing, return to the normal task flow. Do not keep syncing `plans/next.md` unless the user explicitly triggers `/next` again.

## Resume workflow

When the user wants to continue from `/next`:

1. Read `plans/next.md` completely before acting.
2. Treat the new user description as the scope override if it conflicts with the saved note.
3. Do only the minimum repository verification needed to avoid acting on stale assumptions.
4. Start from `下一步第一动作` unless it is clearly stale.
5. Do not rewrite `plans/next.md` automatically after resuming. Rewrite it only if the user explicitly asks to save/update `/next` again.

## File format

Write `plans/next.md` in plain Markdown with this top-level shape:

```md
# /next

## 当前记录

### 任务锚点
### 当前结论
### 已完成
### 未完成
### 关键文件
### 直接命令
### 下一步第一动作
### 状态口径
```

Keep each section concise. Prefer bullet lists. Preserve directly runnable commands when they are valuable for fast continuation.
