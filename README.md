<p align="center">
  <img src="assets/banner.png" alt="ZedClaw" width="100%">
</p>

# ZedClaw

**This project is based on [Hermes Agent](https://github.com/NousResearch/hermes-agent) and has been modified / further developed from it.**

## Credits

- Original author: [@NousResearch](https://github.com/NousResearch)
- Original repository: https://github.com/NousResearch/hermes-agent
- Main changes in this project:
  - Renamed and reoriented the project as ZedClaw.
  - Added a long-task runtime that can autonomously discover work, split it into budget-aware subtasks, and resume execution across wake cycles.
  - Integrated Codex CLI as an execution backend for coding-oriented subtasks.
  - Added GitHub/Gmail feedback intake, Feishu notifications, runtime status commands, language switching, and daily review behavior.

<p align="center">
  <a href="README.zh-CN.md"><img src="https://img.shields.io/badge/Language-中文-red?style=for-the-badge" alt="中文"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge" alt="License: MIT"></a>
</p>

ZedClaw is an autonomous long-task runtime for coding and digital work. It can discover useful tasks, break long-running goals into token-budget-aware subtasks, schedule wake-ups, resume execution through Codex CLI or other adapters, and report status through messaging channels.

The project keeps the interactive agent experience, terminal tools, messaging gateway, slash commands, model switching, and scheduled runtime capabilities, then adds a runtime layer for autonomous task discovery, budget-aware planning, execution tracking, and periodic progress reports.

The name comes from Zed, the Master of Shadows in *League of Legends*: ZedClaw is meant to act like a digital shadow clone for the user, spending tokens to move long-running work forward while the user stays focused on higher-level decisions.

Issues and pull requests are welcome!

This project has been published in the [LINUX DO community](https://linux.do). Thanks to the community for its support and feedback.

## Highlights

| Capability | What it does |
| --- | --- |
| Autonomous task discovery | Finds candidate work from configured sources and lets the planner decide what is worth doing next. |
| Budget-aware decomposition | Breaks long-running goals into smaller subtasks based on token budget, active workload, and execution risk. |
| Runtime scheduling | Lets the agent decide when to wake based on task state, pending feedback, active work, and budget signals. |
| Execution adapters | Runs coding-oriented subtasks through Codex CLI and leaves room for additional long-task adapters. |
| Feedback loop | Watches GitHub and optional Gmail notifications, then schedules follow-up work when comments, reviews, or failures appear. |
| Feishu notifications | Sends updates when tasks start, pause, fail, need human review, or complete meaningful progress. |
| Daily review | Summarizes daily outcomes and lessons into markdown notes and persistent agent memory. |
| Messaging commands | Provides low-cost runtime status through slash commands without asking the LLM. |
| Model flexibility | Supports OpenAI-compatible providers, OpenRouter, Codex OAuth, custom endpoints, and local/runtime tool execution. |

## Long-Task Runtime

ZedClaw is designed for unattended long-running work:

1. Discover candidate tasks from configured directions, repositories, messages, or future task adapters.
2. Filter and rank work by value, risk, current workload, available budget, and known constraints.
3. Ask the planning model to split long goals into concrete subtasks and choose the next wake time.
4. Invoke Codex CLI or another adapter to execute the next subtask inside a bounded runtime loop.
5. Monitor external feedback, task state, failures, and completion signals.
6. Schedule follow-up subtasks when new feedback or unfinished work requires another pass.
7. Notify the operator through Feishu and expose runtime state through slash commands.

The default philosophy is pragmatic: keep moving while budget is available, keep each execution step bounded, and reserve human review for cases the runtime cannot safely resolve.

## Quick Start

Clone the repository and install it in editable mode:

```bash
git clone <your-zedclaw-repo-url>
cd zedclaw
python3 -m venv .venv
source .venv/bin/activate
pip install -e ".[all,dev]"
```

On Windows, you can use the PowerShell installer at `scripts/install.ps1`.

Start the CLI:

```bash
zedclaw
```

Run setup:

```bash
zedclaw setup
```

Configure the long-task / OSS task runtime:

```bash
zedclaw setup osspr
```

## Requirements

- Python 3.11 or newer
- Git and GitHub CLI (`gh`)
- Codex CLI available on `PATH` if coding-task automation is enabled
- A model provider or OAuth-backed Codex provider configured through `zedclaw model`
- Optional: Feishu app credentials for notifications
- Optional: Gmail IMAP/app-password configuration for email-based PR feedback intake

## Common Commands

```bash
zedclaw                 # Start the interactive CLI
zedclaw setup           # Run the full setup wizard
zedclaw setup osspr     # Configure the OSS PR Agent
zedclaw model           # Choose model provider and model
zedclaw gateway         # Start the messaging gateway
zedclaw doctor          # Diagnose local configuration
```

Messaging and CLI slash commands:

| Command | Purpose |
| --- | --- |
| `/osspr` | Show runtime status, active task, submitted PR count, merged PR count, and next wake time for the OSS adapter. |
| `/humanreview` | Show real human-review items that require operator action. |
| `/language` | Switch runtime user-facing output between English and Chinese. |
| `/method` | Change the task discovery theme, for example `/method all` or `/method eval harness`. |
| `/status` | Show messaging platform status where supported. |
| `/new` | Start a new conversation. |
| `/model` | Change the active model. |

## Configuration

Important long-task / OSS adapter settings include:

| Setting | Meaning |
| --- | --- |
| `oss_pr_agent.language` | Output language: `en` or `zh`. |
| `oss_pr_agent.focus_terms` | Task discovery directions, or `all` for the default agent, LLM, and harness engineering scope. |
| `oss_pr_agent.codex_model` | Model used by Codex CLI for coding-task execution. |
| `oss_pr_agent.codex_reasoning_effort` | Codex reasoning effort, for example `medium`. |
| `oss_pr_agent.max_fix_attempts` | Maximum automatic fix attempts before human review. |
| `oss_pr_agent.notify_target` | Notification target, commonly `feishu`. |
| `oss_pr_agent.min_repo_stars` | Minimum repository stars for candidate repositories. |
| `oss_pr_agent.repo_activity_window_days` | Maximum allowed inactivity window for candidate repositories. |
| `oss_pr_agent.budget_url` | Optional OpenAI/Anthropic-compatible usage endpoint; `usage`, `limits`, and `remaining` fields are treated as budget signals. |

Use the setup wizard when possible:

```bash
zedclaw setup osspr
```

## GitHub, Gmail, and Feishu

GitHub CLI is used for repository inspection, GitHub event polling, and coding-task workflows that interact with repositories:

```bash
gh auth login
gh auth status
```

Gmail integration is optional. When enabled, ZedClaw reads recent task-related messages, deduplicates them against GitHub events where applicable, and classifies intent with a small model before scheduling work.

Feishu integration is optional but recommended for unattended operation. It is used to notify you when tasks start, progress, fail, complete daily reviews, or require human review.

## Development

Install development dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -e ".[all,dev]"
```

Run tests:

```bash
python -m pytest
```

Run targeted checks before submitting changes:

```bash
zedclaw doctor
python -m pytest tests/ -q
```

## Project Status

ZedClaw is actively evolving. The general agent runtime is usable, while the long-task runtime is intended for operators who are comfortable with unattended automation and can review task behavior, API usage, and repository permissions.

Use dedicated accounts or carefully scoped credentials for unattended work.

## Roadmap

The current focus is turning ZedClaw into a general long-task runtime: it should plan, schedule, execute, review, and report on many kinds of long-running digital work, with OSS repository work as one adapter rather than the core identity.

## License

MIT. See [LICENSE](LICENSE).
