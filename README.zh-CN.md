<p align="center">
  <img src="assets/banner.png" alt="ZedClaw" width="100%">
</p>

# ZedClaw

**本项目基于 [Hermes Agent](https://github.com/NousResearch/hermes-agent) 修改/二次开发而来。**

## Credits

- 原作者：[@NousResearch](https://github.com/NousResearch)
- 原仓库：https://github.com/NousResearch/hermes-agent
- 本项目主要修改内容：
  - 将项目重命名并重新定位为 ZedClaw。
  - 新增长程任务 Runtime，用于自主发现任务、按 token 预算拆解子任务，并跨唤醒周期持续执行。
  - 集成 Codex CLI 作为代码类子任务的执行后端。
  - 新增 GitHub/Gmail 反馈摄入、飞书通知、Runtime 状态指令、语言切换和每日复盘能力。

<p align="center">
  <a href="README.md"><img src="https://img.shields.io/badge/Language-English-lightgrey?style=for-the-badge" alt="English"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge" alt="License: MIT"></a>
</p>

ZedClaw 是一个面向代码与数字工作的自主长程任务 Runtime。它可以自主寻找有价值的任务，把长期目标拆解为基于 token 预算的子任务，按状态和预算决定唤醒时间，调用 Codex CLI 或其他适配器继续执行，并通过消息平台定时汇报进度。

项目保留了交互式 Agent、终端工具、消息网关、斜杠指令、模型切换和 Runtime 调度能力，并在此基础上加入了自主任务发现、预算化规划、执行追踪和周期性状态汇报能力。

ZedClaw 的命名来自游戏《英雄联盟》中的“影流之主 劫”：它希望成为用户的数字分身，替用户消耗 token，推进需要持续投入的长期任务。

欢迎提交 Issue 和 Pull Request！

本项目已在 [LINUX DO 社区](https://linux.do) 发布，感谢社区的支持与反馈。

## 核心能力

| 能力 | 说明 |
| --- | --- |
| 自主任务发现 | 从配置方向、仓库、消息或后续适配器中发现候选任务，并交给 Planner 判断价值。 |
| 预算化任务拆解 | 根据 token 预算、当前工作量和执行风险，把长程目标拆成可执行的子任务。 |
| Runtime 自主调度 | 根据任务状态、外部反馈、预算信号和工作队列决定下一次唤醒时间。 |
| 执行适配器 | 通过 Codex CLI 执行代码类子任务，并为更多长程任务适配器预留扩展空间。 |
| 反馈闭环 | 综合 GitHub 与可选 Gmail 通知，识别评论、review、失败和完成信号。 |
| 飞书通知 | 任务启动、暂停、失败、需要人工介入、每日复盘等变化都会通知。 |
| 每日复盘 | 汇总当天任务进展、失败原因和经验教训，写入日记与 Agent 记忆。 |
| 消息端指令 | 通过斜杠指令直接查看 Runtime 状态，不需要额外调用大模型。 |
| 灵活模型配置 | 支持 OpenAI 兼容接口、OpenRouter、Codex OAuth、自定义端点和多种工具执行环境。 |

## 长程任务 Runtime

ZedClaw 的目标是无人值守地推进长期任务：

1. 从配置方向、仓库、消息或未来任务适配器中发现候选任务。
2. 按价值、风险、当前工作负载、可用预算和已知约束筛选任务。
3. 让规划模型把长期目标拆成明确子任务，并决定下一次唤醒时间。
4. 调用 Codex CLI 或其他适配器，在有边界的 Runtime 循环中执行下一步。
5. 监控外部反馈、任务状态、失败原因和完成信号。
6. 遇到新反馈或未完成工作时，继续排期后续子任务。
7. 通过飞书通知操作者，并通过斜杠指令暴露 Runtime 状态。

默认策略偏务实：预算充足时持续推进，把每次执行控制在清晰边界内，只把 Runtime 无法可靠处理的事项留给人工检查。

## 快速开始

克隆仓库并以 editable 模式安装：

```bash
git clone <your-zedclaw-repo-url>
cd zedclaw
python3 -m venv .venv
source .venv/bin/activate
pip install -e ".[all,dev]"
```

Windows 用户可以使用 PowerShell 安装脚本：`scripts/install.ps1`。

启动 CLI：

```bash
zedclaw
```

运行完整配置：

```bash
zedclaw setup
```

配置长程任务 / OSS 适配器：

```bash
zedclaw setup osspr
```

## 环境要求

- Python 3.11 或更新版本
- Git 和 GitHub CLI (`gh`)
- 如果启用代码类任务自动化，需要 Codex CLI 在 `PATH` 中可用
- 通过 `zedclaw model` 配置模型提供商，或配置 Codex OAuth provider
- 可选：飞书应用凭据，用于通知
- 可选：Gmail IMAP/app-password 配置，用于读取 PR 相关邮件反馈

## 常用命令

```bash
zedclaw                 # 启动交互式 CLI
zedclaw setup           # 运行完整配置向导
zedclaw setup osspr     # 配置 OSS PR Agent
zedclaw model           # 选择模型提供商和模型
zedclaw gateway         # 启动消息网关
zedclaw doctor          # 检查本地配置问题
```

CLI 与消息端常用斜杠指令：

| 指令 | 作用 |
| --- | --- |
| `/osspr` | 查看 OSS 适配器的 Runtime 状态、当前任务、已提交 PR 数、已记录合并 PR 数和下次唤醒时间。 |
| `/humanreview` | 查看真实需要人工处理的待办事项。 |
| `/language` | 在中文和英文之间切换 Runtime 的用户可见输出。 |
| `/method` | 更换任务发现主题，例如 `/method all` 或 `/method eval harness`。 |
| `/status` | 查看消息平台状态，具体取决于平台支持。 |
| `/new` | 开启新会话。 |
| `/model` | 切换当前模型。 |

## 配置

长程任务 / OSS 适配器相关重要配置包括：

| 配置项 | 含义 |
| --- | --- |
| `oss_pr_agent.language` | 输出语言：`en` 或 `zh`。 |
| `oss_pr_agent.focus_terms` | 任务发现方向；也可以设置为 `all` 使用默认的 Agent、LLM、Harness Engineering 范围。 |
| `oss_pr_agent.codex_model` | Codex CLI 执行代码类子任务时使用的模型。 |
| `oss_pr_agent.codex_reasoning_effort` | Codex 推理强度，例如 `medium`。 |
| `oss_pr_agent.max_fix_attempts` | 自动修复失败 PR 的最大轮数。 |
| `oss_pr_agent.notify_target` | 通知目标，常用 `feishu`。 |
| `oss_pr_agent.min_repo_stars` | 候选仓库最低 star 数。 |
| `oss_pr_agent.repo_activity_window_days` | 候选仓库允许的最长未活跃时间。 |
| `oss_pr_agent.budget_url` | 可选的 OpenAI/Anthropic 兼容用量端点；返回的 `usage`、`limits`、`remaining` 字段会作为预算信号。 |

优先使用配置向导：

```bash
zedclaw setup osspr
```

## GitHub、Gmail 与飞书

GitHub CLI 用于仓库检查、GitHub 事件查询，以及需要和代码仓库交互的任务工作流：

```bash
gh auth login
gh auth status
```

Gmail 集成是可选项。启用后，ZedClaw 会读取最近的任务相关邮件，在适用时与 GitHub 事件去重，并用小模型判断邮件意图，再交给 Runtime 排期处理。

飞书集成也是可选项，但推荐在无人值守运行时启用。ZedClaw 会在任务启动、推进、失败、进入人工待办、每日复盘完成时发送通知。

## 开发

安装开发依赖：

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -e ".[all,dev]"
```

运行测试：

```bash
python -m pytest
```

提交改动前建议运行：

```bash
zedclaw doctor
python -m pytest tests/ -q
```

## 项目状态

ZedClaw 仍在持续演进。通用 Agent Runtime 已可使用；
长程任务 Runtime 面向能接受无人值守自动化的用户，需要操作者自行关注任务行为、API 用量和仓库权限。

建议为无人值守任务使用专门账号，或使用权限范围清晰的凭据。

## 未来方向

当前重点是把 ZedClaw 打造成通用长程任务 Runtime：让它可以规划、调度、执行、复盘并汇报更广泛的长期数字任务，OSS 仓库工作只是其中一个适配器，而不是项目的全部定位。

## 许可证

MIT。详见 [LICENSE](LICENSE)。
