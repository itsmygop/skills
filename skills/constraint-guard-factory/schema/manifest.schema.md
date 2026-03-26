# manifest.json Schema

manifest.json 是每个 constraint-guard skill 实例的唯一 source-of-truth。所有渲染产物（SKILL.md、references/*、agents/*）均从 manifest 派生。

## 顶层字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `manifest_version` | string | 是 | manifest 自身版本号 |
| `factory_version` | string | 是 | 使用的工厂版本号，用于兼容性追踪 |
| `skill` | object | 是 | skill 元数据 |
| `planning` | object | 是 | plan/talk 收口协议配置 |
| `contracts` | object | 是 | 约束轴定义 |
| `reads` | object | 是 | 文件读取分组 |
| `artifacts` | object | 是 | 产物路径配置 |
| `references` | object | 是 | reference 文件输出路径 |
| `source_documents` | object | 否 | 相关计划/设计文档引用（nullable） |
| `optional_modules` | object | 否 | 可选模块声明 |

## skill

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | 是 | slug-safe 唯一标识，匹配 `[a-z0-9-]+` |
| `display_title` | string | 是 | 标题，用于 heading 和 agent adapter |
| `description` | string | 是 | 一行描述 |
| `subject_name` | string | 是 | 主体标签（如 "Foo"） |
| `scope_label` | string | 是 | 范围短语（如 "本仓库 Foo 业务开发"） |

## planning

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `conversation_artifact` | string | 是 | 默认用于回放 leader/talk 收口问题的对话沉淀文件（如 `plans/talk.md`） |
| `closure_statuses` | string[] | 是 | 固定收口状态枚举，必须按顺序写为 `KEEP`、`DRIFT`、`BLOCKED`、`MISSING` |
| `hard_rules` | string[] | 是 | plan/talk 阶段的硬规则列表，至少 3 条 |
| `required_closure_checks` | string[] | 是 | 输出 `Proposed Plan` 或同等收口结论前必须确认的检查项，至少 4 条 |

## contracts

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `axis_prefix` | string | 是 | 大写前缀，如 "RR"。轴编码自动派生为 `<prefix>-01`, `<prefix>-02`, ... |
| `axes` | array | 是 | 4~8 个轴对象，数组顺序即渲染顺序 |

### axes[] 每项

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `title` | string | 是 | 轴标题（同一 manifest 内唯一） |
| `summary` | string | 是 | 引导句，用于 SKILL.md 和 contract-map |
| `constraints` | string[] | 是 | ≥1 条固定约束 |
| `anchors` | anchor[] | 是 | ≥1 个锚点 |
| `common_drifts` | string[] | 是 | ≥1 条常见漂移描述 |

### anchor 对象

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `kind` | string | 是 | `"code"` / `"plan"` / `"artifact_pattern"` |
| `path` | string | 是 | 文件路径 |
| `symbol` | string | 否 | 代码符号（仅 kind=code 时有意义） |
| `label` | string | 否 | 显示标签 |

锚点校验规则：
- `code`: 校验文件存在；有 symbol 时校验符号存在。失败 → WARNING 标记，不阻断
- `plan`: 校验文件存在。失败 → WARNING 标记，不阻断
- `artifact_pattern`: 不校验

## reads

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `groups` | group[] | 是 | 可为空数组 |

### group 对象

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | 是 | 分组标识 |
| `trigger` | string | 是 | 触发条件描述（渲染到 SKILL.md "先读哪些文件"段） |
| `entries` | entry[] | 是 | ≥1 条文件条目 |
| `optional_module` | string | 否 | 关联的可选模块 ID |

### entry 对象

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `path` | string | 是 | 文件路径 |
| `mode` | string | 否 | `"read"` (default) / `"run"` |

## artifacts

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `review_bundle` | object | 是 | 审查 bundle 配置 |
| `visual_results_dir` | string | 否 | 视觉验收结果目录（默认 `test/results`） |

### review_bundle

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `root_dir` | string | 是 | bundle 根目录（如 `test/new`） |
| `prefix` | string | 是 | bundle 命名前缀（如 `foo_rr_review`） |
| `request_filename` | string | 是 | 请求文件名（如 `request.md`） |
| `evidence_dir` | string | 是 | 证据目录名（如 `evidence`） |
| `diagnostics_dir` | string | 是 | 诊断目录名（如 `diagnostics`） |

## references

每个 reference 必须有 `output` 字段指定实际输出文件名。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `contract_map` | `{ output }` | 是 | 约束图文件 |
| `review_protocol` | `{ output }` | 是 | bundle 协议文件 |
| `request_template` | `{ output }` | 是 | 审查请求模板文件 |
| `guard_update_hook` | `{ output }` | 是 | hook prompt 文件 |
| `acceptance_change_policy` | `{ output }` | 是 | 变更策略文件 |

## source_documents（可选）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `plan` | string / null | 否 | 计划文档路径 |
| `proposal` | string / null | 否 | 提案文档路径 |
| `design` | string / null | 否 | 设计文档路径 |
| `tasks` | string / null | 否 | 任务文档路径 |

null 值在模板中渲染为 `None`。

## optional_modules（可选）

### constraint_overlays[]

跨轴约束覆盖层（如主题/日夜模式约束）。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | 是 | 覆盖层标识 |
| `title` | string | 是 | 标题 |
| `applies_to_axes` | string[] | 是 | 生效的轴编码列表 |
| `constraints` | string[] | 是 | 约束列表 |
| `anchors` | anchor[] | 是 | 锚点列表 |
| `common_drifts` | string[] | 是 | 漂移列表 |

### domain_hooks[]

额外 hook prompt（如 viewer 开发 hook）。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | 是 | hook 标识 |
| `title` | string | 是 | 标题 |
| `output` | string | 是 | 输出文件名 |
| `trigger` | string | 是 | 使用时机描述 |
| `body_sections` | string[] | 是 | prompt 内容段落 |

### visual_acceptance

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `enabled` | boolean | 是 | 是否启用 |
| `output` | string | 是 | 输出文件名 |
| `statuses` | string[] | 是 | 视觉验收状态枚举 |
| `checks` | string[] | 是 | 检查项列表 |

### diagnostic_scripts[]

声明式引用，不模板化脚本内容。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | 是 | 脚本标识 |
| `path` | string | 是 | 脚本路径 |
| `purpose` | string | 是 | 用途描述 |

### agent_openai

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `enabled` | boolean | 是 | 是否启用 |
| `output` | string | 是 | 输出路径（如 `agents/openai.yaml`） |
| `interface_display_name` | string | 是 | UI 显示名 |
| `short_description` | string | 是 | 简短描述 |

## 校验规则

1. `contracts.axes` 长度必须在 4~8（含）
2. `planning.closure_statuses` 必须且只能是 `KEEP`、`DRIFT`、`BLOCKED`、`MISSING`
3. `planning.hard_rules` 至少 3 条，且不得放宽“验证不是完成”“bundle 不是完成结论”“人工验证不能作为默认收尾”的固定协议
4. `planning.required_closure_checks` 至少 4 条
5. `contracts.axis_prefix` 非空，建议大写
6. 轴 `title` 在同一 manifest 内唯一
7. 每个轴至少 1 条 constraint、1 个 anchor、1 条 common_drift
8. 所有 reference `output` 路径唯一
9. `agent_openai.enabled = true` 时必须提供 `output`、`interface_display_name`、`short_description`
10. `visual_acceptance.enabled = true` 时必须提供 `output`、`statuses`、`checks`
11. 审查结果枚举 `KEEP | DRIFT | BLOCKED | MISSING` 为固定协议，不可在 manifest 中配置

## 完整示例

```json
{
  "manifest_version": "1.0",
  "factory_version": "1.0",
  "skill": {
    "id": "foo-ui-acceptance-guard",
    "display_title": "Foo 约束集与请求响应审查",
    "description": "约束 Foo 请求响应 contract、状态归属与 test/new 审查产物",
    "subject_name": "Foo",
    "scope_label": "本仓库 Foo 业务开发"
  },
  "planning": {
    "conversation_artifact": "plans/talk.md",
    "closure_statuses": ["KEEP", "DRIFT", "BLOCKED", "MISSING"],
    "hard_rules": [
      "bundle、测试、probe、日志与截图只属于验证手段，不是完成结论",
      "若主结论仍依赖用户手动验证，不得用“已实现”或“已修复”口吻收尾",
      "自动验证仍有关键缺口时，默认落到 MISSING；受外部环境阻塞时才是 BLOCKED"
    ],
    "required_closure_checks": [
      "是否指出真实业务代码改动",
      "是否指出行为变化",
      "是否指出最小自动验证",
      "是否指出剩余未闭环部分及当前状态"
    ]
  },
  "contracts": {
    "axis_prefix": "RR",
    "axes": [
      {
        "title": "Request Contract",
        "summary": "检查请求拼装是否保持语义",
        "constraints": [
          "排序通过远端请求参数实现，不允许本地排序",
          "空搜索词进入首页",
          "翻页数量固定 30",
          "请求竞争必须保留代次保护",
          "搜索请求必须写日志，能回溯 params"
        ],
        "anchors": [
          { "kind": "code", "path": "utils/script/image/foo.py", "symbol": "build_foo_search_tags" },
          { "kind": "code", "path": "GUI/script/foo/core.py", "symbol": "FooSearchController.start_search" },
        ],
        "common_drifts": [
          "在前端直接对已有列表排序",
          "空词时拒绝请求",
          "把 order:* 塞进目录名",
          "去掉 request_token",
          "不记录 params"
        ]
      }
    ]
  },
  "reads": {
    "groups": [
      {
        "id": "default",
        "trigger": "默认先读",
        "entries": [
          { "path": "references/request-response-constraint-map.md" }
        ]
      },
      {
        "id": "request-response",
        "trigger": "涉及请求参数、响应结构、下载链路、状态归属时",
        "entries": [
          { "path": "plans/ui.md" },
          { "path": "utils/script/image/foo.py" },
          { "path": "utils/config/qc.py" }
        ]
      }
    ]
  },
  "artifacts": {
    "review_bundle": {
      "root_dir": "test/new",
      "prefix": "foo_rr_review",
      "request_filename": "request.md",
      "evidence_dir": "evidence",
      "diagnostics_dir": "diagnostics"
    },
    "visual_results_dir": "test/results"
  },
  "references": {
    "contract_map": { "output": "request-response-constraint-map.md" },
    "review_protocol": { "output": "tests-new-report-protocol.md" },
    "request_template": { "output": "request-template.md" },
    "guard_update_hook": { "output": "guard-update-hook-prompt.md" },
    "acceptance_change_policy": { "output": "acceptance-change-policy.md" }
  },
  "source_documents": {
    "plan": "plans/tasks.md",
    "proposal": null,
    "design": null,
    "tasks": null
  }
}
```

注意：示例仅展示 1 个轴，完整 Foo manifest 有 6 个轴。
