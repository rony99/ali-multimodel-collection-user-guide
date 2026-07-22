# 采集用户说明（v1.1）· 自包含包

本文件夹即可完整使用，**不依赖**仓库内其他手册路径。

> 下文 <span style="color:#d93025">红色</span> 为交卷硬要求 / 易踩坑要点。

## 文档怎么用

<span style="color:#d93025">**以 [用户操作步骤.md](./用户操作步骤.md) 为主**</span>：按它做配连接、出题、调难度、整理交卷。  
[甲方要求说明.md](./甲方要求说明.md)（详细门槛 / 字段 / Checklist）只作**信息补充**，有疑问时查阅，不必当日常操作手册通读。

| 文档 / 目录 | 角色 |
| --- | --- |
| **[用户操作步骤.md](./用户操作步骤.md)** | **主线（优先跟这个做）** |
| **[甲方要求说明.md](./甲方要求说明.md)** | **补充**：合格门槛、meta/rubric 字段、Checklist |
| **[甲方数据包参考样例/](./甲方数据包参考样例/)** | 甲方原格式样例包 + **`multi-sessions/`** 附加示意 + 模板 |
| **[参考/Taxonomy标签.md](./参考/Taxonomy标签.md)** | meta 标签枚举 |
| **[上传前预检/](./上传前预检/)** | 上传前结构预检 Skill（预审 ≠ 终审） |

建议顺序：**用户操作步骤** → 对照样例 → 上传前预检；字段/门槛吃不准时再翻甲方要求说明。

## 最终提交长什么样

一道题交<span style="color:#d93025">**两块，都要提交、平级放置**</span>（目录名仅字母、数字、`._-`）：

| 部分 | 样例路径 | 含义 |
| --- | --- | --- |
| ① 甲方要求的最终数据包 | [甲方数据包参考样例/20260617_gateway-raw-http/](./甲方数据包参考样例/20260617_gateway-raw-http/) | Harbor 原格式：题面 / 环境 / 测试 / GT / rubric / meta / `trajectories/` 等 |
| ② 多次执行 session 校验 | [甲方数据包参考样例/multi-sessions/](./甲方数据包参考样例/multi-sessions/) | 三模型多次独立 session + `manifest.json`（每次是否通过测试） |

<span style="color:#d93025">**缺一不可**：只交①或只交②都不算交齐。</span>

```text
<submit_root>/
  <task_id>/                     # ① 甲方原格式数据包
    instruction.md
    task.toml
    meta.json
    manifest.json                # 包内采集说明（可保留）
    environment/ ...
    ground_truth/ ...
    tests/ ...
    rubrics/ ...
    trajectories/ ...            # 甲方结构中的轨迹目录
    reports/ ...
    scores/ ...

  multi-sessions/                # ② 附加：多模型多次 session（与数据包平级）
    claude-opus-4-8/
      <session_id>.jsonl         # Opus ≥2 次，建议 4 次
    glm-5.2/
    qwen-3.7-max/
    manifest.json                # session_id ↔ 模型 ↔ eval_pass
```

示意见 [甲方数据包参考样例/multi-sessions/](./甲方数据包参考样例/multi-sessions/)。

<span style="color:#d93025">**除甲方数据包外，必须交 `multi-sessions/`。** Opus 至少 **≥2** 次独立完整 session 用于核验通过率；正式合格仍为 **pass@4 ≤ 60%**（建议交齐 4 次）。</span>

<span style="color:#d93025">甲方硬门槛摘要：</span> assistant 平均轮次 **≥ 20**；pass@4 偏序 **Opus > GLM > 千问**；Baseline 测挂 / GT 测过；三模型同一份 `instruction.md`。  
<span style="color:#d93025">平台附加：</span> Opus−千问 ≥ 20%；多题批次 ≥ 50% 题 Opus 至少一次通过。

<span style="color:#d93025">**一次交多道题时**：不得整批都是「三个模型都测不过」；其中 **≥ 50%** 须 Opus 对该题至少有一次通过。</span>详见 [用户操作步骤.md](./用户操作步骤.md) §7.1b。

## 预审声明

预检只做结构与完整性检查，**不代表审核通过**。须自行验证通过率与区分度，<span style="color:#d93025">**以实现网和甲方的后期审核为准**</span>。

## FAQ

### 是否可以使用自己的模型？

<span style="color:#d93025">不可以。</span> 正式对比模型固定为平台指定的三套：

- `claude-opus-4.8`（目录可用 `claude-opus-4-8`）
- `glm-5.2`
- `qwen-3.7-max`

须按平台说明配置连接后采集；不要换成自备 API / 本地模型充当这三套成绩。

### 怎么才算「通过」？

<span style="color:#d93025">只看自动测试，不看人眼。</span>

1. 在 Docker 里对当次做题结果跑 `tests/test.sh`  
2. 退出码 `0` = 该次 **通过**；非 `0` = **未通过**  
3. `multi-sessions/manifest.json` 里用 `eval_pass: true/false` 登记每一次  

题目合格还要看汇总门槛（详见操作步骤第 7 步），例如：Opus 同题独立 4 次的 pass@4 **≤ 60%**、偏序 Opus > GLM > 千问、平均 assistant 执行轮次 **≥ 20** 等。  
结构预检通过 ≠ 题目合格；最终以实现网和甲方后期审核为准。

### 「一条数据」是一个模型跑出来的结果吗？

<span style="color:#d93025">不是。</span> 一条交卷数据 = **一道题**，包含：

| 内容 | 说明 |
| --- | --- |
| 甲方数据包 | 题面、环境、测试、GT、rubric、meta 等（三模型共用同一套） |
| `multi-sessions/` | **三个模型**各自多次独立 session（不是只交某一个模型的一次结果） |

更细地说：

- **一道题** → 一个 `<task_id>/` + 一个 `multi-sessions/`  
- **一次 session** → 某个模型干净重做一整题留下的一份 `.jsonl`（算一次是否测过）  
- **pass@4** → 同一模型、同一题独立做 4 次，看过了几次（不是 4 道不同题）

所以：一条数据 ≠「某一个模型跑一次」；而是「一道完整题 + 三模型（含多次）的执行与是否通过」。

### 只交甲方数据包、不交 multi-sessions 可以吗？

<span style="color:#d93025">不可以。</span> 两块都要交、平级放置；只交一块不算交齐。

### 三模型可以用不同的题面吗？

<span style="color:#d93025">不可以。</span> 必须同一份 `instruction.md`，以及同一 Baseline / 测试 / Docker。
