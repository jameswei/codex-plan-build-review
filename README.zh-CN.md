# Plan, Build, Review

[English](README.md) | [简体中文](README.zh-CN.md)

一个基于角色分工的工作流，通过独立审查、明确批准和任务级检查点，可靠地
交付重要里程碑 —— 同时支持 [Codex](https://developers.openai.com/codex/) 和
[Claude Code](https://code.claude.com/)。

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

| 角色 | 职责 | 权限 | Codex 模型 · 推理强度 | Claude 模型 · 推理强度 |
| --- | --- | --- | --- | --- |
| `milestone-explorer` | 范围明确的调查 | 只读 | Terra · medium | Haiku |
| `milestone-planner` | 制定决策完整的里程碑计划 | 只读 | Sol · xhigh | Opus · xhigh |
| `milestone-builder` | 实现一项已批准的任务 | 工作区写入 | Sol · medium | Sonnet · high |
| `milestone-reviewer` | 审查计划、任务和最终集成 | 只读 | Sol · high | Opus · high |

角色定义已随仓库提供，可以根据任一运行环境进行调整。Haiku 不支持可配置的
推理强度等级，因此 `milestone-explorer` 对应的 Claude 一栏留空。

## 安装

### Codex

使用技能内部名称，将仓库克隆到 Codex 技能目录：

```sh
git clone https://github.com/jameswei/plan-build-review.git \
  ~/.codex/skills/plan-build-review
```

安装四个自定义代理定义：

```sh
mkdir -p ~/.codex/agents
cp ~/.codex/skills/plan-build-review/assets/agents/*.toml ~/.codex/agents/
```

如果已经存在同名代理，请在复制前检查现有文件。安装完成后启动新的
Codex 会话，让 Codex 发现这个技能和它的代理。（Codex 也支持项目级的
`.agents/skills` 目录，作为上面用户级 `~/.codex/skills/` 路径之外的
另一种安装位置。）

### Claude Code

将此仓库添加为插件市场，然后安装插件：

```text
/plugin marketplace add jameswei/plan-build-review
/plugin install plan-build-review@plan-build-review
/reload-plugins
```

这一步会同时安装技能和全部四个子代理定义，无需手动复制任何文件。安装后，
技能和角色会带上插件名作为命名空间前缀，例如 `milestone-planner` 会变成
`plan-build-review:milestone-planner`。

## 使用

在 Codex 中，明确调用这个技能：

```text
使用 $plan-build-review 为此仓库交付 v1.0 里程碑。
```

在 Claude Code 中：

```text
使用 /plan-build-review:plan-build-review 为此仓库交付 v1.0 里程碑。
```

工作流会在计划通过审查后暂停，等待你的批准，然后才会创建分支或修改实现
文件。推送分支或创建草稿拉取请求前，它还会再次请求批准。

它适合重视范围、正确性和审查可追溯性的里程碑工作。普通编码或文档任务
不会隐式触发这个技能。

## 兼容性

这个软件包同时支持 Codex 和 Claude Code，不面向其他运行环境。但两边的
安装体验并不完全对等：Claude Code 的插件市场可以一步安装技能和全部四个
子代理，无需手动复制任何文件；Codex 目前没有将自定义代理与技能捆绑安装
的机制，因此它的自定义代理仍需要上文所述的一次性手动复制。

只读权限的实现方式也不对等。Codex 的 `sandbox_mode: read-only` 会在
操作系统层面阻止写入文件系统，即使是在 shell 命令内部也是如此。Claude
Code 没有对应的执行沙箱，因此 `milestone-explorer`、`milestone-planner`
和 `milestone-reviewer` 是通过在 `tools` 允许列表中完全不包含 Bash 和
MCP 工具来实现只读限制的——它们并非被沙箱隔离，而是根本没有被授予可能
写入的工具。

## 许可证

[MIT](LICENSE)
