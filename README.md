# Codex Agent Harness Layout

`codex-agent-harness-layout` 是一个可安装的 Codex Plugin，用于在项目中创建或维护一套可复用的多智能体协作 Harness。

它会让 Codex 按以下顺序组织工作：

```text
planner -> explorer -> implementer -> reviewer -> tester -> verifier
```

其中 `reviewer`、`tester` 和 `verifier` 只负责检查并向主 Agent 汇报问题；主 Agent 会把需要修复的问题交回 `implementer`，然后重新执行质量检查。

## 主要功能

- 生成项目级 `AGENTS.md` 和 `Agent.md`。
- 配置六个固定角色：`planner`、`explorer`、`implementer`、`reviewer`、`tester`、`verifier`。
- 默认只允许 `implementer` 修改项目文件。
- 建立 `reviewer -> tester -> verifier` 质量门禁。
- 发现问题后，由主 Agent 统一调度 `implementer` 修复。
- 对机器人、底盘、电机、导航、ROS、TCP 控制和真实硬件项目增加安全约束。
- 保留项目中与当前任务无关的已有内容和本地修改。

## 安装

### 1. 添加 GitHub Marketplace

在 PowerShell 中执行：

```powershell
codex plugin marketplace add yuanimperialhal/Codex-agent-harness-layout --ref main
codex plugin marketplace list
```

列表中应出现：

```text
codex-harness-plugins
```

> `codex plugin marketplace add` 只负责添加 Marketplace，不等于安装其中的 Plugin。

### 2. 安装 Plugin

1. 重启 Codex 或 ChatGPT 桌面端。
2. 打开 Plugin 目录。
3. 选择 Marketplace：`Codex Harness Plugins`。
4. 找到 `Codex Agent Harness Layout`。
5. 点击安装。
6. 新建一个任务，使新安装的 Skill 在新任务中加载。

较新的 Codex CLI 或 IDE 版本也可能在 `/plugins` 或 `Settings -> Plugins` 中提供安装入口。本文以桌面端 Plugin 目录作为主流程。

## 使用方法

### 1. 打开目标项目

在 Codex 中打开需要添加 Harness 的项目根目录。这个 Plugin 会写入项目文件，建议先提交或备份重要的本地修改。

### 2. 新建任务并调用 Plugin

推荐提示词：

```text
Use codex-agent-harness-layout to create a multi-agent harness for this project.
```

也可以使用中文：

```text
请使用 codex-agent-harness-layout，为当前项目创建一套多智能体 Harness。
```

如果项目已经存在部分 Harness 文件，可以这样说：

```text
Use codex-agent-harness-layout to inspect and update the existing multi-agent harness in this project. Preserve unrelated project instructions.
```

### 3. 审查生成结果

默认会创建或维护以下文件：

```text
AGENTS.md
Agent.md
.codex/
  config.toml
  agents/
    planner.toml
    explorer.toml
    implementer.toml
    reviewer.toml
    tester.toml
    verifier.toml
every_tasks_harness/
  planner.md
  explorer.md
  implementer.md
  reviewer.md
  tester.md
  verifier.md
```

## 角色说明

| 角色 | 默认权限 | 主要职责 |
| --- | --- | --- |
| `planner` | 只读 | 明确目标、范围、风险和验收标准 |
| `explorer` | 只读 | 检查真实仓库状态、入口、依赖和测试 |
| `implementer` | 可写 | 执行任务范围内的修改并修复质量门禁发现的问题 |
| `reviewer` | 只读 | 检查缺陷、安全问题、回归和规则违反 |
| `tester` | 只读 | 运行或指定聚焦测试并报告准确结果 |
| `verifier` | 只读 | 确认流程、文件、测试结果和最终报告一致 |

## 修复循环

初次实现完成后，主 Agent 会依次执行：

```text
reviewer -> tester -> verifier
```

如果任一质量门禁发现任务范围内的问题：

1. 检查角色把问题报告给主 Agent，不直接修改文件。
2. 主 Agent 汇总问题并交给 `implementer`。
3. `implementer` 进行最小范围修复。
4. 主 Agent 重新执行 `reviewer -> tester -> verifier`。
5. 三个质量门禁全部通过后，任务才可以结束。

## 验证生成的 Harness

可以在项目根目录运行：

```powershell
Test-Path .\AGENTS.md
Test-Path .\Agent.md
Test-Path .\.codex\config.toml
Get-ChildItem .\.codex\agents\*.toml
Get-ChildItem .\every_tasks_harness\*.md
git diff --check
```

应确认：

- 六个 Agent TOML 文件全部存在。
- 每个 TOML 都指向匹配的角色说明文件。
- `reviewer`、`tester`、`verifier` 没有默认写权限。
- 没有未完成的模板标记。
- Git 差异中没有意外修改。
- 真实硬件项目包含安全检查和停止方法。

## 更新

需要获取 Marketplace 的最新版本时，执行：

```powershell
codex plugin marketplace upgrade codex-harness-plugins
codex plugin marketplace list
```

然后重启 Codex，并在新任务中再次测试 Plugin。

## 常见问题

### Marketplace 没有出现

先运行：

```powershell
codex plugin marketplace list
```

确认存在 `codex-harness-plugins`。如果已经存在但桌面端没有显示，请完全退出并重新启动 Codex 或 ChatGPT 桌面端。

### 出现两个同名 Plugin

运行：

```powershell
codex plugin list
```

保留来自以下 Marketplace 的版本：

```text
codex-agent-harness-layout@codex-harness-plugins
```

旧的个人版本通常显示为：

```text
codex-agent-harness-layout@personal
```

请在 Plugin 管理界面卸载旧的 `@personal` 版本，再重启 Codex。不要删除 GitHub Marketplace 中的版本。

### Plugin 没有被调用

- 确认 Plugin 已安装并启用。
- 安装或更新后新建任务，不要只在旧任务中继续测试。
- 在提示词中明确写出 `codex-agent-harness-layout`。
- 确认当前打开的是希望生成 Harness 的项目根目录。

### 项目已经有 AGENTS.md

Plugin 应合并 Harness 规则并保留无关的项目说明。执行后仍需检查 Git diff，确认现有项目约束没有被覆盖。

## 仓库结构

```text
.agents/
  plugins/
    marketplace.json
plugins/
  codex-agent-harness-layout/
    .codex-plugin/
      plugin.json
    skills/
      codex-agent-harness-layout/
        SKILL.md
        agents/
          openai.yaml
```

关键文件：

- [Marketplace 配置](.agents/plugins/marketplace.json)
- [Plugin Manifest](plugins/codex-agent-harness-layout/.codex-plugin/plugin.json)
- [Skill 说明](plugins/codex-agent-harness-layout/skills/codex-agent-harness-layout/SKILL.md)

## 官方参考

- [Build plugins](https://learn.chatgpt.com/docs/build-plugins)
- [Plugins](https://learn.chatgpt.com/docs/plugins)
- [Submit plugins](https://learn.chatgpt.com/docs/submit-plugins)

当前仓库提供的是可通过 GitHub Marketplace 安装的 Plugin。进入 OpenAI 官方公共 Plugin 目录仍需要单独提交和审核。
