# Plan, Build, Review

[English](README.md) | [简体中文](README.zh-CN.md)

一个基于角色分工的 Codex 技能，通过独立审查、明确批准和任务级检查点，
可靠地交付重要里程碑。

## 为什么使用它

- 规划者基于证据制定计划，并且必须通过独立审查。
- 未获得用户明确批准前，不会开始实现。
- 工作按照构建、验证、审查和检查点提交的顺序逐项推进。
- 每轮审查都使用新的审查者，减少构建者与审查者之间的思维偏移。
- 最终集成审查会检查整体行为和跨任务契约。
- 发布必须获得明确批准，并且工作流永远不会自动合并。

```text
预检 → 规划 → 审查 → 批准 → 构建任务 → 审查任务 → 创建检查点
     → 重复 → 集成审查 → 发布批准
```

## 角色

| 角色 | 职责 | 权限 | 模型与推理强度 |
| --- | --- | --- | --- |
| `milestone-explorer` | 范围明确的调查 | 只读 | Terra，medium |
| `milestone-planner` | 制定决策完整的里程碑计划 | 只读 | Sol，xhigh |
| `milestone-builder` | 实现一项已批准的任务 | 工作区写入 | Sol，medium |
| `milestone-reviewer` | 审查计划、任务和最终集成 | 只读 | Sol，high |

角色定义已随仓库提供，可以根据你的 Codex 环境进行调整。

## 安装

使用技能内部名称，将仓库克隆到 Codex 技能目录：

```sh
git clone https://github.com/jameswei/codex-plan-build-review.git \
  ~/.codex/skills/plan-build-review
```

安装四个自定义代理定义：

```sh
mkdir -p ~/.codex/agents
cp ~/.codex/skills/plan-build-review/assets/agents/*.toml ~/.codex/agents/
```

如果已经存在同名代理，请在复制前检查现有文件。安装完成后启动新的
Codex 会话，让 Codex 发现这个技能和它的代理。

## 使用

明确调用这个技能：

```text
使用 $plan-build-review 为此仓库交付 v1.0 里程碑。
```

工作流会在计划通过审查后暂停，等待你的批准，然后才会创建分支或修改实现
文件。推送分支或创建草稿拉取请求前，它还会再次请求批准。

它适合重视范围、正确性和审查可追溯性的里程碑工作。普通编码或文档任务
不会隐式触发这个技能。

## 兼容性

这个软件包专为 Codex 设计。工作流原则可以迁移到其他环境，但技能元数据、
自定义代理、沙箱设置、推理强度配置和子代理上下文控制都依赖 Codex。

## 许可证

[MIT](LICENSE)
