---
name: style-refactor
description: Use this skill when the user wants a second-pass structural cleanup after functional code changes, or asks to unify code according to their architecture/style preferences. Trigger on requests about abstracting responsibilities, reducing private-method sprawl, removing defensive redundancy, clarifying ownership boundaries, or refactoring managers/controllers/views into cleaner layers.
---

# Style Refactor Pass

Read [`references/style-rules.md`](references/style-rules.md) before editing.

## When To Use

Use this skill when the user asks for any of these after or during implementation:

- 按这套风格统一处理
- 再精炼一轮
- 做结构清洗 / 抽象化整理
- 去掉防御性冗余
- 这个类私有方法太多了
- 归属关系不对，重新分层

## Workflow

1. 先画归属关系。
   明确 state、view、controller、manager 分别属于谁，谁跟随谁。
   同时画对外边界：哪些符号需要跨文件、跨包稳定暴露，哪些只是实现细节。

2. 识别结构坏味道。
   重点看：
   - 一个类里堆了一串只服务彼此的私有方法
   - 单个私有方法依赖多个其他私有方法
   - manager 同时承担条目、动画、下载、badge、状态同步等多种职责
   - 已被启用状态或调用链保证的重复防御判断
   - 拆文件后 `__init__.py` 变成旧大文件的符号搬运站，导出远多于真实外部调用
   - 已有稳定 owner（类、store、inspector），又额外铺一层 `get_xxx` / `set_xxx` / `clear_xxx` / `build_xxx` 单行转发

3. 选择处理方式。
   - 单次使用且很短：直接内联
   - 单次使用但形成稳定职责：抽成独立控制类
   - 单次使用且只是转发：删掉包装，让调用方直接依赖真正 owner
   - 语义是整体替换：直接整体替换，不手搓字段同步

4. 做整片重构，不要一脚一步。
   不要只删除用户点名的单个私有方法。要顺着同一职责链一次性整理干净。

5. 收尾时验证两件事。
   - 类边界是否一眼可见
   - 当前类是否还残留成片的中转私有方法

## Editing Rules

- 优先改结构，不先堆局部函数或包装层。
- 不要用“换位置不换复杂度”的办法冒充抽象。
- `__init__.py` 只暴露真实稳定边界，不为“拆分后看起来还像旧单文件”而批量 re-export。
- 单行函数若不形成独立领域语义，不要因为“拆了私有方法”就升级成模块级公共函数。
- 不要把面板级状态挂到条目级视图，也不要把条目级逻辑堆回总 manager。
- 槽函数不重复做已由 UI 状态保证的前置判断。
- 如果用户在当前对话中确认了新的稳定风格偏好，顺手更新 `references/style-rules.md`，让 skill 继续演进。
