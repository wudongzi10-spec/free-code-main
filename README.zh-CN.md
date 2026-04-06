<p align="center">
  <img src="assets/screenshot.png" alt="free-code" width="720" />
</p>

<h1 align="center">free-code</h1>

<p align="center">
  <strong>Claude Code 的自由构建版。</strong><br>
  去除全部遥测。移除全部护栏。解锁全部实验功能。<br>
  一个二进制，无任何回传。
</p>

<p align="center">
  <a href="#快速安装"><img src="https://img.shields.io/badge/install-one--liner-blue?style=flat-square" alt="Install" /></a>
  <a href="https://github.com/paoloanzn/free-code/stargazers"><img src="https://img.shields.io/github/stars/paoloanzn/free-code?style=flat-square" alt="Stars" /></a>
  <a href="https://github.com/paoloanzn/free-code/issues"><img src="https://img.shields.io/github/issues/paoloanzn/free-code?style=flat-square" alt="Issues" /></a>
  <a href="https://github.com/paoloanzn/free-code/blob/main/FEATURES.md"><img src="https://img.shields.io/badge/features-88%20flags-orange?style=flat-square" alt="Feature Flags" /></a>
  <a href="#ipfs-镜像"><img src="https://img.shields.io/badge/IPFS-mirrored-teal?style=flat-square" alt="IPFS" /></a>
</p>

---

## 快速安装

```bash
curl -fsSL https://raw.githubusercontent.com/paoloanzn/free-code/main/install.sh | bash
```

检查系统，必要时安装 Bun，克隆仓库，以全部实验功能启用的方式构建，并在 PATH 上创建 `free-code` 的软链接。

然后运行 `free-code`，使用 `/login` 命令完成你选择的模型提供商的认证。

---

## 目录

- [这是什么](#这是什么)
- [模型提供商](#模型提供商)
- [快速安装](#快速安装)
- [系统要求](#系统要求)
- [构建](#构建)
- [使用](#使用)
- [实验功能](#实验功能)
- [项目结构](#项目结构)
- [技术栈](#技术栈)
- [IPFS 镜像](#ipfs-镜像)
- [贡献](#贡献)
- [许可证](#许可证)

---

## 这是什么

这是 Anthropic 的 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI（终端原生 AI 编码代理）的一个干净、可构建的分支。上游源码在 2026-03-31 通过 npm 分发包中的 source map 暴露而公开。

该分支在该快照之上做了三类改动：

### 移除遥测

上游二进制通过 OpenTelemetry/gRPC、GrowthBook 分析、Sentry 错误上报和自定义事件日志进行回传。本构建中：

- 所有外部遥测端点被死代码消除或 stub 掉
- GrowthBook 特性开关仍可本地评估（运行时特性门控需要），但不会回传
- 不再有崩溃报告、使用分析或会话指纹采集

### 移除安全提示护栏

Anthropic 会向每次对话注入系统级指令，以进一步限制 Claude 行为。包括硬编码拒绝模式、注入的“网络风险”指令块，以及从 Anthropic 服务器推送的安全叠加设置。

本构建去除了这些注入。模型自身的安全训练仍然有效——这里只是移除 CLI 对其额外包裹的提示级限制。

### 解锁实验功能

Claude Code 通过 `bun:bundle` 编译期开关提供 88 个特性标志。公版 npm 发行版多数关闭。本构建解锁了能正常编译的 54 个标志。详见下文 [实验功能](#实验功能) 或 [FEATURES.md](FEATURES.md) 的完整审计。

---

## 模型提供商

free-code 默认支持 **五种 API 提供商**。通过设置环境变量切换提供商，无需修改代码。

### Anthropic（直连 API）— 默认

直接使用 Anthropic 的第一方 API。

| 模型 | ID |
|---|---|
| Claude Opus 4.6 | `claude-opus-4-6` |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` |
| Claude Haiku 4.5 | `claude-haiku-4-5` |

### OpenAI Codex

使用 OpenAI 的 Codex 模型生成代码。需要 Codex 订阅。

| 模型 | ID |
|---|---|
| GPT-5.3 Codex（推荐） | `gpt-5.3-codex` |
| GPT-5.4 | `gpt-5.4` |
| GPT-5.4 Mini | `gpt-5.4-mini` |

```bash
export CLAUDE_CODE_USE_OPENAI=1
free-code
```

### AWS Bedrock

通过你的 AWS 账户使用 Amazon Bedrock。

```bash
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION="us-east-1"   # 或 AWS_DEFAULT_REGION
free-code
```

使用标准 AWS 凭据（环境变量、`~/.aws/config` 或 IAM role）。模型会自动映射为 Bedrock ARN 格式（如 `us.anthropic.claude-opus-4-6-v1`）。

| 变量 | 作用 |
|---|---|
| `CLAUDE_CODE_USE_BEDROCK` | 启用 Bedrock 提供商 |
| `AWS_REGION` / `AWS_DEFAULT_REGION` | AWS 区域（默认：`us-east-1`） |
| `ANTHROPIC_BEDROCK_BASE_URL` | 自定义 Bedrock 端点 |
| `AWS_BEARER_TOKEN_BEDROCK` | Bearer token 认证 |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | 跳过认证（测试） |

### Google Cloud Vertex AI

通过 GCP 项目使用 Vertex AI。

```bash
export CLAUDE_CODE_USE_VERTEX=1
free-code
```

使用 Google Cloud 应用默认凭据（`gcloud auth application-default login`）。模型会自动映射为 Vertex 格式（如 `claude-opus-4-6@latest`）。

### Anthropic Foundry

使用 Anthropic Foundry 进行专用部署。

```bash
export CLAUDE_CODE_USE_FOUNDRY=1
export ANTHROPIC_FOUNDRY_API_KEY="..."
free-code
```

支持将自定义部署 ID 作为模型名使用。

### 提供商选择一览

| 提供商 | 环境变量 | 认证方式 |
|---|---|---|
| Anthropic（默认） | -- | `ANTHROPIC_API_KEY` 或 OAuth |
| OpenAI Codex | `CLAUDE_CODE_USE_OPENAI=1` | OpenAI OAuth |
| AWS Bedrock | `CLAUDE_CODE_USE_BEDROCK=1` | AWS 凭据 |
| Google Vertex AI | `CLAUDE_CODE_USE_VERTEX=1` | `gcloud` ADC |
| Anthropic Foundry | `CLAUDE_CODE_USE_FOUNDRY=1` | `ANTHROPIC_FOUNDRY_API_KEY` |

---

## 系统要求

- **运行时**：[Bun](https://bun.sh) >= 1.3.11
- **OS**：macOS 或 Linux（Windows 通过 WSL）
- **认证**：所选提供商的 API key 或 OAuth 登录

```bash
# 如果尚未安装 Bun
curl -fsSL https://bun.sh/install | bash
```

---

## 构建

```bash
git clone https://github.com/paoloanzn/free-code.git
cd free-code
bun build
./cli
```

### 构建变体

| 命令 | 输出 | 特性 | 描述 |
|---|---|---|---|
| `bun run build` | `./cli` | 仅 `VOICE_MODE` | 接近生产的二进制 |
| `bun run build:dev` | `./cli-dev` | 仅 `VOICE_MODE` | 带 dev 版本戳 |
| `bun run build:dev:full` | `./cli-dev` | 全部 54 个实验标志 | 完整解锁构建 |
| `bun run compile` | `./dist/cli` | 仅 `VOICE_MODE` | 另一种输出路径 |

### 自定义特性标志

不使用完整解锁也可启用指定标志：

```bash
# 只启用 ultraplan 和 ultrathink
bun run ./scripts/build.ts --feature=ULTRAPLAN --feature=ULTRATHINK

# 在 dev 构建上额外加一个标志
bun run ./scripts/build.ts --dev --feature=BRIDGE_MODE
```

---

## 使用

```bash
# 交互式 REPL（默认）
./cli

# 一次性模式
./cli -p "what files are in this directory?"

# 指定模型
./cli --model claude-opus-4-6

# 从源码运行（启动更慢）
bun run dev

# OAuth 登录
./cli /login
```

### 环境变量参考

| 变量 | 作用 |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `ANTHROPIC_AUTH_TOKEN` | 认证 token（替代） |
| `ANTHROPIC_MODEL` | 覆盖默认模型 |
| `ANTHROPIC_BASE_URL` | 自定义 API 端点 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | 自定义 Opus 模型 ID |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | 自定义 Sonnet 模型 ID |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | 自定义 Haiku 模型 ID |
| `CLAUDE_CODE_OAUTH_TOKEN` | 通过环境变量提供 OAuth token |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | API key helper 缓存 TTL |

---

## 实验功能

`bun run build:dev:full` 会启用全部 54 个可用实验标志。重点包括：

### 交互与 UI

| 标志 | 描述 |
|---|---|
| `ULTRAPLAN` | Claude Code Web（Opus 级）远程多代理规划 |
| `ULTRATHINK` | 深度思考模式——输入 “ultrathink” 以提升推理强度 |
| `VOICE_MODE` | 按住说话语音输入与听写 |
| `TOKEN_BUDGET` | Token 预算跟踪与告警 |
| `HISTORY_PICKER` | 交互式提示历史选择器 |
| `MESSAGE_ACTIONS` | UI 中的消息操作入口 |
| `QUICK_SEARCH` | 提示快速搜索 |
| `SHOT_STATS` | Shot 分布统计 |

### 代理、记忆与规划

| 标志 | 描述 |
|---|---|
| `BUILTIN_EXPLORE_PLAN_AGENTS` | 内置 explore/plan 代理预设 |
| `VERIFICATION_AGENT` | 任务验证代理 |
| `AGENT_TRIGGERS` | 本地 cron/触发器工具（后台自动化） |
| `AGENT_TRIGGERS_REMOTE` | 远程触发器工具路径 |
| `EXTRACT_MEMORIES` | 查询后自动提取记忆 |
| `COMPACTION_REMINDERS` | 智能提醒上下文压缩 |
| `CACHED_MICROCOMPACT` | 查询流中的微压缩缓存状态 |
| `TEAMMEM` | 团队记忆文件与 watcher hooks |

### 工具与基础设施

| 标志 | 描述 |
|---|---|
| `BRIDGE_MODE` | IDE 远程控制桥（VS Code、JetBrains） |
| `BASH_CLASSIFIER` | Bash 权限决策分类器辅助 |
| `PROMPT_CACHE_BREAK_DETECTION` | 压缩/查询流的缓存破坏检测 |

完整审计请参阅 [FEATURES.md](FEATURES.md)（含 34 个故障标志的修复注记）。

---

## 项目结构

```
scripts/
  build.ts                # 带特性标志系统的构建脚本

src/
  entrypoints/cli.tsx     # CLI 入口
  commands.ts             # 命令注册（斜杠命令）
  tools.ts                # 工具注册（代理工具）
  QueryEngine.ts          # LLM 查询引擎
  screens/REPL.tsx        # 主要交互 UI（Ink/React）

  commands/               # /slash 命令实现
  tools/                  # 代理工具实现（Bash、Read、Edit 等）
  components/             # Ink/React 终端 UI 组件
  hooks/                  # React hooks
  services/               # API 客户端、MCP、OAuth、分析
    api/                  # API 客户端 + Codex fetch 适配器
    oauth/                # OAuth 流（Anthropic + OpenAI）
  state/                  # 应用状态存储
  utils/                  # 工具函数
    model/                # 模型配置、提供商、校验
  skills/                 # Skill 系统
  plugins/                # 插件系统
  bridge/                 # IDE bridge
  voice/                  # 语音输入
  tasks/                  # 后台任务管理
```

---

## 技术栈

| | |
|---|---|
| **运行时** | [Bun](https://bun.sh) |
| **语言** | TypeScript |
| **终端 UI** | React + [Ink](https://github.com/vadimdemedes/ink) |
| **CLI 解析** | [Commander.js](https://github.com/tj/commander.js) |
| **Schema 校验** | Zod v4 |
| **代码搜索** | ripgrep（内置） |
| **协议** | MCP, LSP |
| **API** | Anthropic Messages, OpenAI Codex, AWS Bedrock, Google Vertex AI |

---

## IPFS 镜像

该仓库的完整副本已通过 Filecoin 永久固定在 IPFS 上：

| | |
|---|---|
| **CID** | `bafybeiegvef3dt24n2znnnmzcud2vxat7y7rl5ikz7y7yoglxappim54bm` |
| **Gateway** | https://w3s.link/ipfs/bafybeiegvef3dt24n2znnnmzcud2vxat7y7rl5ikz7y7yoglxappim54bm |

如果仓库被下线，代码仍然可用。

---

## 贡献

欢迎贡献。如果你打算修复 34 个故障特性标志中的某一个，请先查看 [FEATURES.md](FEATURES.md) 的重建说明——许多只缺一个小封装或资源即可编译。

1. Fork 本仓库
2. 创建功能分支（`git checkout -b feat/my-feature`）
3. 提交更改（`git commit -m 'feat: add something'`）
4. 推送分支（`git push origin feat/my-feature`）
5. 提交 Pull Request

---

## 许可证

Claude Code 的原始源码归 Anthropic 所有。本分支由于其源码曾通过 npm 分发包公开而存在。请自行斟酌使用。
