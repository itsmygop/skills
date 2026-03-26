---
name: constraint-guard-factory
description: 约束守卫工厂 — 从分支级开发拆解产物生成 constraint-guard skill 实例
---

# Constraint Guard Factory

## 概述

这个工厂将固定的审查协议与实例化的约束知识分离，使新的分支级开发能快速生成结构化的 constraint-guard skill。

工厂产出平台无关的纯 Markdown + JSON 文档。不依赖任何运行时代码或 CLI 工具。

## 工厂产出结构

```
<skill-id>/
  SKILL.md                          ← 主 skill 文件
  manifest.json                     ← 实例参数（唯一 source-of-truth）
  references/
    <contract-map>.md               ← 约束图
    <review-protocol>.md            ← bundle 协议
    <request-template>.md           ← 审查请求模板
    <guard-update-hook>.md          ← hook prompt
    <acceptance-change-policy>.md   ← 变更策略
    [optional] <visual-standard>.md
    [optional] <domain-hook-*.md>
  scripts/                          ← 可选
  agents/                           ← 可选
    openai.yaml
```

## 输入入口

### 入口 A：头脑风暴拆解

开发者对整个分支进行头脑风暴，拆解出多个 task。

### 入口 B：OpenSpec 初始 task

开发者用 OpenSpec 制成初始 task1。

两种入口汇聚到同一流程。

## 实例化流程

### Step 1: Intake

读取拆解产出 + 相关代码入口。确定：
- 主体名称（subject_name）
- 范围描述（scope_label）
- 核心代码文件列表

### Step 2: Boundary Extraction

提炼：
- 系统边界：数据从哪来、到哪去
- 数据流：请求→响应→存储→交付
- 状态归属：哪些状态由谁管理
- 收口边界：哪些内容只是验证手段，哪些内容才算任务完成；默认对照 `plans/talk.md` 一类对话沉淀文件回放消极收尾

### Step 3: Axis Synthesis

生成 4~8 条 contract axes。每个轴需要：
- 标题（如 "Request Contract"）
- 引导句（summary）
- ≥1 条固定约束
- ≥1 个代码锚点
- ≥1 条常见漂移

与用户交互确认轴定义。

### Step 4: manifest.json 编写

基于确认的轴定义，填写 `manifest.json`。参考 `schema/manifest.schema.md` 获取完整字段说明。

除约束轴外，还必须显式填写 `planning`，把 leader/talk 阶段的收口协议固定下来：
- 默认回放哪个对话沉淀文件
- 输出 `Proposed Plan` 前必须完成哪些 closure checks
- 哪些“验证不是完成”的硬规则要下沉到 GENERATED 区

### Step 5: Skill Rendering

按以下顺序渲染模板（从 `templates/` 目录）：

1. `SKILL.md.tmpl` → `SKILL.md`
2. `contract-map.md.tmpl` → `references/<contract_map.output>`
3. `review-bundle-protocol.md.tmpl` → `references/<review_protocol.output>`
4. `request-template.md.tmpl` → `references/<request_template.output>`
5. `guard-update-hook-prompt.md.tmpl` → `references/<guard_update_hook.output>`
6. `acceptance-change-policy.md.tmpl` → `references/<acceptance_change_policy.output>`

可选模块（仅当 manifest 启用时渲染）：
- `visual-acceptance-standard.md.tmpl`
- `domain-hook-prompt.md.tmpl`（per hook instance）
- `agents-openai.yaml.tmpl`

### Step 6: 后续迭代

修改 `manifest.json` → 重新渲染。GENERATED 区完全覆盖，CUSTOM 区原样保留。

## 渲染规则

### 占位符语法

- 标量替换: `{{path.to.value}}`
- 列表迭代: `{{#each list}}...{{/each}}`
- 条件包含: `{{#if boolean}}...{{/if}}`

### GENERATED + CUSTOM 区块

每个渲染文件包含两个标记区：

```html
<!-- FACTORY:BEGIN GENERATED file="<path>" template="<id>" manifest_version="<ver>" factory_version="<ver>" -->
... 工厂渲染内容（re-render 时完全覆盖）...
<!-- FACTORY:END GENERATED -->

<!-- FACTORY:BEGIN CUSTOM file="<path>" slot="appendix" -->
... 用户手写追加内容（re-render 时原样保留）...
<!-- FACTORY:END CUSTOM -->
```

规则：
- 用户只在 CUSTOM 区写入自定义内容
- GENERATED 区在 re-render 时完全覆盖
- 已有文件无有效标记 → re-render 报错
- 每文件最多一个 CUSTOM 区

### 锚点校验

| 锚点 kind | 校验行为 |
|-----------|---------|
| `code` | 校验文件+符号存在。失败 → WARNING 标记，不阻断 |
| `plan` | 校验文件存在。失败 → WARNING 标记，不阻断 |
| `artifact_pattern` | 不校验 |

### 轴编码

轴编码从 `contracts.axis_prefix` + 数组顺序自动派生：
- prefix="RR", 6 axes → `RR-01`, `RR-02`, ..., `RR-06`

### 固定协议

以下内容嵌入模板中不可配置：
- 审查结果枚举: `KEEP | DRIFT | BLOCKED | MISSING`
- 仓库规则: "约束集服务于业务实现，不能反过来为当前补丁改写 contract"
- Bundle 布局语义: request + evidence/ + diagnostics/
- 证据与诊断材料分流
- Plan/talk 收口协议: bundle、测试、probe、日志、截图与人工验证都不能被写成默认完成结论；未闭环时只能落到 `MISSING` 或 `BLOCKED`

## 占位符清单自检

渲染前检查 manifest.json 是否覆盖以下所有占位符：

### 必填占位符

- [ ] `{{manifest_version}}`
- [ ] `{{factory_version}}`
- [ ] `{{skill.id}}`
- [ ] `{{skill.display_title}}`
- [ ] `{{skill.description}}`
- [ ] `{{skill.subject_name}}`
- [ ] `{{skill.scope_label}}`
- [ ] `{{planning.conversation_artifact}}`
- [ ] `{{planning.closure_statuses}}`
- [ ] `{{planning.hard_rules}}`
- [ ] `{{planning.required_closure_checks}}`
- [ ] `{{contracts.axis_prefix}}`
- [ ] `{{contracts.axes}}` — 4~8 个轴，每个含 title/summary/constraints/anchors/common_drifts
- [ ] `{{reads.groups}}` — 至少一个分组
- [ ] `{{artifacts.review_bundle.*}}` — root_dir/prefix/request_filename/evidence_dir/diagnostics_dir
- [ ] `{{references.*}}` — 5 个核心 reference 的 output 路径

### 可选占位符

- [ ] `{{source_documents.*}}` — plan/proposal/design/tasks（null → "None"）
- [ ] `{{optional_modules.constraint_overlays}}` — 跨轴覆盖层
- [ ] `{{optional_modules.domain_hooks}}` — 额外 hook prompt
- [ ] `{{optional_modules.visual_acceptance}}` — 视觉验收附录
- [ ] `{{optional_modules.diagnostic_scripts}}` — 诊断脚本声明
- [ ] `{{optional_modules.agent_openai}}` — agent adapter

## 校验规则摘要

1. `planning.closure_statuses` 固定为 `KEEP | DRIFT | BLOCKED | MISSING`
2. `planning.hard_rules` 至少 3 条，且不得放宽固定协议
3. `planning.required_closure_checks` 至少 4 条
4. `contracts.axes` 长度 4~8
5. 轴 `title` 唯一
6. 每轴至少 1 constraint + 1 anchor + 1 drift
7. 所有 reference `output` 唯一
8. 启用的可选模块必须提供完整子字段
9. 审查结果枚举不可在 manifest 中出现
