# 企微 / 公众号 AI 客服（CowAgent）

> 篝火出品 · 收录于篝火 App 应用区 · 基于开源项目 [zhayujie/CowAgent](https://github.com/zhayujie/CowAgent)（MIT）

把大模型接到**微信客服、企业微信自建应用、微信公众号**这几个官方通道，客户或同事发消息，由大模型按你设定的人设回答。只推荐这三类官方接口，不使用个人微信协议。

## 和「Dify 知识库客服接入器」（w05）的区别

| | 本工具（w04） | w05 Dify 知识库客服接入器 |
|---|---|---|
| 回答从哪来 | 大模型直接回答，口径靠 `CHARACTER_DESC` 人设写 | 由你在 Dify 里搭好的应用 / 知识库回答 |
| 接入通道 | 微信客服（对外部微信用户）、企微自建应用、公众号订阅号 / 服务号 | 只推荐企微自建应用 |
| 需要先搭 Dify | 不需要，只要一个大模型接口 | 需要 |
| 适合 | 想快速上线、问答口径简单、需要对外部微信用户和公众号提供客服 | 已有 Dify 知识库、答案必须来自你的资料 |

上游的「知识库」是 Agent 给自己整理的学习笔记，不是让你导入 FAQ 文档的客服知识库；要按资料回答，请用 w05。

## 部署

需要一台装好 Docker、**有公网地址**的服务器（x86 与 ARM 均可）。

1. 到 [Releases](https://github.com/campfire-code/wecom-ai-service/releases/latest) 下载 `docker-compose.yml` 和 `.env.example`，同放一个目录，把 `.env.example` 改名为 `.env`。
2. 在 `.env` 里填：大模型接口（地址、密钥、模型名，兼容 OpenAI 格式即可）、客服人设 `CHARACTER_DESC`、接入通道 `CHANNEL_TYPE` 及对应通道的凭证。
3. 运行 `docker compose up -d`。各通道监听端口：微信客服 9888、企微自建应用 9898、公众号 8080，用到哪个就放行哪个，或用反向代理转发。
4. 到对应后台保存回调地址，地址路径分别为：微信客服 `http://你的地址:9888/wxkf/`，企微自建应用 `http://你的地址:9898/wxcomapp`，公众号 `http://你的地址/wx`（容器内不能监听 80 端口，请让反向代理把 80 转到 8080）。
5. 企业微信类通道保存成功后，还要把服务器出口 IP 加到「企业可信 IP」，否则收发不了消息。
6. 各通道的凭证怎么获取，见上游文档：[微信客服](https://docs.cowagent.ai/zh/channels/wechat-kf)、[企微自建应用](https://docs.cowagent.ai/zh/channels/wecom)、[公众号](https://docs.cowagent.ai/zh/channels/wechatmp)，原文也在 `docs/zh/channels/` 下。

## 前置条件

- 一个兼容 OpenAI 接口格式的大模型服务（地址、密钥、模型名由你自己填，密钥只存在你的服务器上）。
- 微信客服：已认证的企业微信，并开通「微信客服」，建议为它单独创建一个企微自建应用。
- 企微自建应用：企业微信管理员权限；认证企业还需要与主体一致的备案域名。
- 公众号：订阅号（个人可申请，长回答需要用户再发一句才能取到）或认证服务号（需有客服接口权限）。
- 公网可访问的服务器地址。命令行部署，没有图形界面；软件日志以英文为主，上游文档有中文版。

## 安全说明

- **默认关闭 Agent 模式**（`.env` 里 `AGENT=False`）。上游 Agent 模式下模型可以调用工具、执行命令，镜像里又是 full-access，面向外部客户时不应开启；需要请自己评估后再改。
- `.env` 里是你的密钥：只放服务器上，不要提交到公开仓库，也不要把容器日志公开发出。
- 篝火版修复了上游启动日志会把 `*_token`、密码类配置以明文打印的问题（上游只遮盖含 key / secret 的配置名），并把 8 位以下的短密钥整串遮盖。

## 验收用例

| 操作 | 应得到 |
|---|---|
| 填好企微自建应用凭证，`docker compose up -d` 后 `docker logs wecom-ai-service` | 出现 `Listening on http://0.0.0.0:9898/wxcomapp/`；日志里 token、secret、aes_key 只显示前后 3 位 |
| 在企业微信里给自建应用发一句话（已填大模型） | 收到模型按人设写的回复 |
| 用微信打开微信客服接入链接发一句话 | 收到回复 |

## 和上游的区别

- 新增中文说明与篝火发布流程（`tool.json`、`.github/workflows/release.yml`、`campfire/` 部署文件），镜像用 `docker/Dockerfile.campfire` 构建（与上游 `docker/Dockerfile.latest` 相同，仅把 `INSTALL_BROWSER` 默认改为 false，不装浏览器，镜像更小）。
- 改了 `config.py` 的日志遮盖：含 token、password 的配置也会遮盖，短密钥整串遮盖。其余代码与上游一致。
- 删除了上游的 GitHub 工作流（依赖上游的密钥）。
- 上游还支持飞书、钉钉、Telegram、个人微信等通道，本版本的部署文件只覆盖上面三类微信官方通道。

## 归属与许可证

软件全部功能与版权归 [zhayujie/CowAgent](https://github.com/zhayujie/CowAgent) 原作者所有，按 MIT 许可证分发，LICENSE 保留；第三方依赖的许可证见上游 `requirements*.txt` 对应项目。

---

以下为上游原文。

---

<p align="center"><img src="https://github.com/user-attachments/assets/eca9a9ec-8534-4615-9e0f-96c5ac1d10a3" alt="CowAgent" width="420" /></p>

<p align="center">
  <a href="https://github.com/zhayujie/CowAgent/releases/latest"><img src="https://img.shields.io/github/v/release/zhayujie/CowAgent?cacheSeconds=3600" alt="Latest release"></a>
  <a href="https://github.com/zhayujie/CowAgent/blob/master/LICENSE"><img src="https://img.shields.io/badge/license-MIT-green.svg" alt="License: MIT"></a>
  <a href="https://github.com/zhayujie/CowAgent"><img src="https://img.shields.io/github/stars/zhayujie/CowAgent?style=flat-square&cacheSeconds=3600" alt="Stars"></a>
  <a href="https://docs.cowagent.ai/"><img src="https://img.shields.io/badge/Docs-cowagent.ai-blue?style=flat&logo=readthedocs&logoColor=white" alt="Docs"></a>
  <a href="https://cdn.link-ai.tech/code/cow/cowagent-wechat-group.png"><img src="https://img.shields.io/badge/WeChat-Group-07C160?style=flat&logo=wechat&logoColor=white" alt="WeChat Group"></a>
  <a href="https://discord.gg/9U8eA8v9TR"><img src="https://img.shields.io/badge/Discord-Join-5865F2?style=flat&logo=discord&logoColor=white" alt="Discord"></a>
</p>

<p align="center">
  <a href="https://trendshift.io/repositories/25763" target="_blank"><img src="https://trendshift.io/api/badge/repositories/25763" alt="zhayujie%2FCowAgent | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/></a>
</p>

<p align="center">
  [English] | [<a href="docs/zh/README.md">中文</a>] | [<a href="docs/zh/README-Hant.md">繁體中文</a>] | [<a href="docs/ja/README.md">日本語</a>]
</p>

**CowAgent** is an open-source super AI assistant that proactively plans tasks, controls your computer and external services, creates and runs Skills, builds a personal knowledge base and long-term memory, forms multi-agent teams, and grows alongside you through self-evolution — a reference implementation of Agent Harness engineering.

CowAgent is lightweight, easy to deploy, and built to extend. Plug in any major LLM provider and run it 24/7 on a personal computer or server, across the web and all major IM platforms.

<p align="center">
  <a href="https://cowagent.ai/">🌐 Website</a> &nbsp;·&nbsp;
  <a href="https://docs.cowagent.ai/intro/index">📖 Docs</a> &nbsp;·&nbsp;
  <a href="https://docs.cowagent.ai/guide/quick-start">🚀 Quick Start</a> &nbsp;·&nbsp;
  <a href="https://skills.cowagent.ai/">🧩 Skill Hub</a> &nbsp;·&nbsp;
  <a href="https://cowagent.ai/download/">💻 Download</a> &nbsp;·&nbsp;
  <a href="https://link-ai.tech/cowagent/create?utm_source=cowagent&utm_medium=readme&utm_campaign=nav">☁️ Try Online</a>
</p>

<br/>

## 🎬 Demo

<p align="center">
  <video src="https://github.com/user-attachments/assets/8625a19f-615c-4343-8be8-3707ce4d4d4e" controls muted playsinline width="720">
    Your browser can't play this video.
    <a href="https://cowagent.ai/">Watch the demo on our website →</a>
  </video>
</p>

<br/>

## 🌟 Highlights

| Capability | Description |
| :--- | :--- |
| [Planning](https://docs.cowagent.ai/intro/architecture) | Decomposes complex tasks and executes them step by step, looping over tools until the goal is reached |
| [Multi-Agent](https://docs.cowagent.ai/multi-agent/team) | Build a team of Agents, each with its own role, model, skills, and knowledge, collaborating within a shared conversation |
| [Memory](https://docs.cowagent.ai/memory/index) | Three-tier architecture (context → daily → core), automatic Deep Dream distillation, hybrid keyword + vector retrieval |
| [Knowledge](https://docs.cowagent.ai/knowledge/index) | Auto-curates structured knowledge into a Markdown wiki, builds an evolving knowledge graph with visual browsing |
| [Evolution](https://docs.cowagent.ai/memory/self-evolution) | Self-Evolution reviews conversations automatically to improve skills, follow up on unfinished tasks, and consolidate memory and knowledge, growing through everyday use |
| [Skills](https://docs.cowagent.ai/skills/index) | One-click install from [Skill Hub](https://skills.cowagent.ai/), GitHub, ClawHub; or create custom skills via natural-language conversation |
| [Tools](https://docs.cowagent.ai/tools/index) | Built-in file I/O, terminal, browser, scheduler, memory retrieval, web search, and 10+ more tools — with native MCP integration |
| [Channels](https://docs.cowagent.ai/channels/index) | Integrates with Web, WeChat, Feishu, DingTalk, WeCom, QQ, Official Accounts, Telegram, and Slack |
| Multimodal | First-class support for text, images, voice, and files — recognition, generation, and delivery |
| [Models](https://docs.cowagent.ai/models/index) | Claude, GPT, Gemini, DeepSeek, Qwen, GLM, Kimi, MiniMax, Doubao, and more — swap providers from the Web console with one click |
| [Deploy](https://docs.cowagent.ai/guide/quick-start) | One-line installer, unified Web console, multiple deployment modes (local, Docker, server) |

<br/>

## 🏗️ Architecture

<img src="https://cdn.jsdelivr.net/gh/zhayujie/cowagent-assets@main/architecture/en/architecture.jpg" alt="CowAgent Architecture" width="750"/>

CowAgent is a complete **Agent Harness**: messages flow in through **Channels**; the **Agent Core** plans and reasons over memory, knowledge, and the available tools and skills; **Models** generate the response, which is sent back through the originating channel. Every layer is decoupled and independently extensible.

Read more in [Architecture](https://docs.cowagent.ai/intro/architecture).

<br/>

## 🚀 Quick Start

A one-line installer takes care of dependencies, configuration, and startup:

**Linux / macOS:**

```bash
bash <(curl -fsSL https://cdn.link-ai.tech/code/cow/run.sh)
```

**Windows (PowerShell):**

```powershell
irm https://cdn.link-ai.tech/code/cow/run.ps1 | iex
```

**Docker:**

```bash
curl -O https://cdn.link-ai.tech/code/cow/docker-compose.yml
docker compose up -d
```

Once started, open `http://localhost:9899` to access the **Web console** — your one-stop hub to chat with the Agent, configure models, connect channels, and install skills.

> Deploying on a server? Set `web_host` to `0.0.0.0` in `config.json` to make the console reachable from outside, and set `web_password` to protect it. Don't forget to open port `9899` in your firewall or security group.

> 📖 Detailed guides: [Quick Start](https://docs.cowagent.ai/guide/quick-start) · [Install from Source](https://docs.cowagent.ai/guide/manual-install) · [Upgrade](https://docs.cowagent.ai/guide/upgrade)

After installation, manage the service with the [cow CLI](https://docs.cowagent.ai/cli/index):

```bash
cow start | stop | restart        # service control
cow status | logs                  # status and logs
cow update                         # pull latest code and restart
cow skill install <name>           # install a skill
cow install-browser                # install browser automation
```

> 💻 Desktop client: download the **[CowAgent Desktop client](https://cowagent.ai/download/)** (macOS / Windows) — the backend is bundled, ready to use out of the box.

<br/>

## 👥 Multi-Agent Team

Create a team of Agents, each with its own memory, capabilities, knowledge, and workspace. Each can focus on its own domain, or collaborate within a shared conversation to tackle complex tasks together. Multi-Agent teams are available on the Web, the desktop client, and IM channel integrations.

<img src="https://cdn.jsdelivr.net/gh/zhayujie/cowagent-assets@main/screenshots/en/agent-team-overview.png" alt="CowAgent Multi-Agent Team" width="800"/>

> See [Agent Team](https://docs.cowagent.ai/multi-agent/team) for details.

<br/>

## 🤖 Models

CowAgent supports all mainstream LLM providers. **Chat, vision, image generation, ASR/TTS, and embeddings** can each be routed to a different vendor. Providers are configured directly in the Web console — no manual file editing required.

| Provider | Featured Models | Chat | Vision | Image Gen | ASR | TTS | Embedding |
| --- | --- | :-: | :-: | :-: | :-: | :-: | :-: |
| [DeepSeek](https://docs.cowagent.ai/models/deepseek) | deepseek-flash (V4.1) / pro | ✅ | ✅ | | | | |
| [Claude](https://docs.cowagent.ai/models/claude) | claude-opus-5-5 / fable-5.1 | ✅ | ✅ | | | | |
| [OpenAI](https://docs.cowagent.ai/models/openai) | gpt-6.1-sol / gpt-6 series | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| [Gemini](https://docs.cowagent.ai/models/gemini) | gemini-3.8-flash | ✅ | ✅ | ✅ | | | |
| [MiniMax](https://docs.cowagent.ai/models/minimax) | MiniMax-M3 | ✅ | ✅ | ✅ | | ✅ | |
| [GLM](https://docs.cowagent.ai/models/glm) | glm-5.3-flash, glm-5v-turbo | ✅ | ✅ | | ✅ | | ✅ |
| [Qwen](https://docs.cowagent.ai/models/qwen) | qwen3.8-flash | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| [Kimi](https://docs.cowagent.ai/models/kimi) | kimi-k3 | ✅ | ✅ | | | | |
| [Doubao](https://docs.cowagent.ai/models/doubao) | doubao-seed-2.1 series | ✅ | ✅ | ✅ | | | ✅ |
| [ERNIE](https://docs.cowagent.ai/models/qianfan) | ernie-5.1 | ✅ | ✅ | | | | |
| [MiMo](https://docs.cowagent.ai/models/mimo) | mimo-v2.6-pro / flash | ✅ | ✅ | | ✅ | ✅ | |
| [LinkAI](https://docs.cowagent.ai/models/linkai) | One key for 100+ models | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| [Custom](https://docs.cowagent.ai/models/custom) | Local models / third-party proxy | ✅ | | | | | |

> For details on each provider, see the [Models overview](https://docs.cowagent.ai/models/index).

<br/>

## 💬 Channels

A single Agent instance can serve multiple channels in parallel. Most channels can be onboarded right from the Web console.

| Channel | Text | Image | File | Voice | Group |
| --- | :-: | :-: | :-: | :-: | :-: |
| [Web Console](https://docs.cowagent.ai/channels/web) (default) | ✅ | ✅ | ✅ | ✅ | |
| [Telegram](https://docs.cowagent.ai/channels/telegram) | ✅ | ✅ | ✅ | ✅ | ✅ |
| [Slack](https://docs.cowagent.ai/channels/slack) | ✅ | ✅ | ✅ | | ✅ |
| [Discord](https://docs.cowagent.ai/channels/discord) | ✅ | ✅ | ✅ | | ✅ |
| [WeChat](https://docs.cowagent.ai/channels/weixin) | ✅ | ✅ | ✅ | ✅ | |
| [Feishu / Lark](https://docs.cowagent.ai/channels/feishu) | ✅ | ✅ | ✅ | ✅ | ✅ |
| [DingTalk](https://docs.cowagent.ai/channels/dingtalk) | ✅ | ✅ | ✅ | ✅ | ✅ |
| [WeCom Bot](https://docs.cowagent.ai/channels/wecom-bot) | ✅ | ✅ | ✅ | ✅ | ✅ |
| [QQ](https://docs.cowagent.ai/channels/qq) | ✅ | ✅ | ✅ | | ✅ |
| [WeCom App](https://docs.cowagent.ai/channels/wecom) | ✅ | ✅ | ✅ | ✅ | |
| [WeChat Customer Service](https://docs.cowagent.ai/channels/wechat-kf) | ✅ | ✅ | ✅ | ✅ | |
| [WeChat Official Account](https://docs.cowagent.ai/channels/wechatmp) | ✅ | ✅ | | ✅ | |

> See the [Channels overview](https://docs.cowagent.ai/channels/index) for setup details.

<img src="https://cdn.jsdelivr.net/gh/zhayujie/cowagent-assets@main/screenshots/en/web-console-chat.png" alt="CowAgent Web Console" width="800"/>

*The Web console is the default channel and the unified entry point to configure models, channels, skills, memory, and more.*

<br/>

## 🧠 Memory & Knowledge Base

**Long-term memory** uses a three-tier architecture: conversation context (short-term) → daily memory (mid-term) → MEMORY.md (long-term). A nightly **Deep Dream** pass distills scattered memories into refined long-term entries and a narrative journal. See [Long-term Memory](https://docs.cowagent.ai/memory/index) · [Deep Dream](https://docs.cowagent.ai/memory/deep-dream).

**Personal knowledge base** complements the time-ordered memory by organizing structured knowledge **by topic**. The Agent automatically curates valuable information from conversations, maintains cross-references and indexes, and the Web console offers an interactive knowledge-graph view. See [Personal Knowledge Base](https://docs.cowagent.ai/knowledge/index).

<table>
  <tr>
    <td width="50%">
      <img src="https://cdn.jsdelivr.net/gh/zhayujie/cowagent-assets@main/screenshots/en/web-console-memory.png" alt="Long-term Memory" />
      <p align="center"><em>Long-term Memory · Three-tier architecture + Deep Dream</em></p>
    </td>
    <td width="50%">
      <img src="https://cdn.jsdelivr.net/gh/zhayujie/cowagent-assets@main/screenshots/en/web-console-knowledge.png" alt="Personal Knowledge Base" />
      <p align="center"><em>Knowledge Base · Auto-curated Markdown wiki</em></p>
    </td>
  </tr>
</table>

<br/>

## 🔧 Tools & Skills

**Tools** are atomic capabilities the Agent uses to interact with system resources. **Skills** are higher-level workflows defined by a manifest file that compose multiple tools to accomplish complex tasks.

### Tool System

**Built-in tools** cover file I/O (`read` / `write` / `edit` / `ls`), terminal (`bash`), file sending (`send`), memory retrieval (`memory`), environment variables (`env_config`), web fetching (`web_fetch`), scheduling (`scheduler`), web search (`web_search`), vision (`vision`), and browser automation (`browser`).

**MCP protocol** integrates the open ecosystem of [Model Context Protocol](https://modelcontextprotocol.io) servers. A single `mcp.json` is enough — supports stdio / SSE transports, hot reload, and zero-code integration.

Learn more: [Tools overview](https://docs.cowagent.ai/tools/index) · [MCP integration](https://docs.cowagent.ai/tools/mcp).

### Skills System

- **[Skill Hub](https://skills.cowagent.ai/)** — open skill marketplace: browse, search, install in one click
- **GitHub / ClawHub / URL and more** — install skills from any source
- **Conversational authoring** — generate custom skills through dialogue with `skill-creator`; turn any workflow or third-party API into a reusable skill

```bash
/skill list                   # list installed skills
/skill search <keyword>        # search the marketplace
/skill install <name>          # one-click install
```

Learn more: [Skills overview](https://docs.cowagent.ai/skills/index) · [Creating Skills](https://docs.cowagent.ai/skills/create).

<br/>

## 🏷 Changelog

> **2026.10.14:** [v2.2.1](https://github.com/zhayujie/CowAgent/releases/tag/2.2.1) — Artifacts center and custom menu, open Agent Chat API, prompt caching and context cost savings, tool, channel and security improvements.

> **2026.09.30:** [v2.2.0](https://github.com/zhayujie/CowAgent/releases/tag/2.2.0) — Capabilities center, multi-Agent collaboration improvements, Web console rebuild, higher model cache hit rates, broad stability and security hardening.

> **2026.09.14:** [v2.1.9](https://github.com/zhayujie/CowAgent/releases/tag/2.1.9) — Multi-Agent collaboration refinements, configurable model lists and multiple fallback models, channel integration fixes, new image models (gpt-image-2.5), plus voice fixes.

> **2026.09.10:** [v2.1.8](https://github.com/zhayujie/CowAgent/releases/tag/2.1.8) — Multi-Agent teams, manual scheduled task creation, context-usage visualization, new models and search providers, workspace file editing.

> **2026.08.20:** [v2.1.7](https://github.com/zhayujie/CowAgent/releases/tag/2.1.7) — Multiple workspaces, session-level permission modes, task notifications, desktop improvements, plus new model support.

> **2026.08.12:** [v2.1.6](https://github.com/zhayujie/CowAgent/releases/tag/2.1.6) — Sub agents for parallel task delegation, reasoning-effort settings, a pluggable memory vector backend, plus experience and security improvements.

> **2026.07.29:** [v2.1.5](https://github.com/zhayujie/CowAgent/releases/tag/2.1.5) — Workspace with file preview, core tool improvements (file search, write-time validation, background commands), context compaction (`/compact`), one-click prompt optimization, security hardening.

> **2026.07.20:** [v2.1.4](https://github.com/zhayujie/CowAgent/releases/tag/2.1.4) — Desktop experience improvements, MCP OAuth authorization, Lark channel enhancements, scheduler improvements and data backup, new models.

> **2026.07.08:** [v2.1.3](https://github.com/zhayujie/CowAgent/releases/tag/2.1.3) — [Desktop client](https://cowagent.ai/download/) for macOS / Windows, knowledge base document management, on-demand MCP tool retrieval, Traditional Chinese support, new models.

> **2026.06.18:** [v2.1.2](https://github.com/zhayujie/CowAgent/releases/tag/2.1.2) — Web console upgrades (scheduled task management, knowledge base categories, multiple custom model providers), Self-Evolution improvements, new models (kimi-k2.7-code, glm-5.2), security hardening and refinements.

> **2026.06.09:** [v2.1.1](https://github.com/zhayujie/CowAgent/releases/tag/2.1.1) — Self-Evolution, Web console upgrades (message management, parallel sessions), cross-platform MCP enhancements with concurrent calls, new models (MiniMax-M3, qwen3.7-plus), Python 3.13 support.

> **2026.06.01:** [v2.1.0](https://github.com/zhayujie/CowAgent/releases/tag/2.1.0) — Internationalization, new channels (Telegram, Discord, Slack, WeChat Customer Service), CLI interaction upgrades, streamlined one-line install, MCP Streamable HTTP support, new models (claude-opus-4-8, MiMo).

Full history: [Release Notes](https://docs.cowagent.ai/releases/overview)

<br/>

## 🤝 Community & Support

Join our [**Discord server**](https://discord.gg/9U8eA8v9TR) to ask questions, share skills, and follow development:

<a href="https://discord.gg/9U8eA8v9TR"><img src="https://img.shields.io/badge/Discord-Join%20the%20community-5865F2?style=for-the-badge&logo=discord&logoColor=white" alt="Join our Discord"></a>

You can also [file an issue](https://github.com/zhayujie/CowAgent/issues) on GitHub, or scan the QR code below to join our WeChat group:

<img width="130" src="https://cdn.jsdelivr.net/gh/zhayujie/cowagent-assets@main/community/open-community.png">

<br/>

## 🔗 Related Projects

- **[Cow Skill Hub](https://github.com/zhayujie/cow-skill-hub)** — open skill marketplace for AI Agents; works with CowAgent, OpenClaw, Claude Code, and more
- **[bot-on-anything](https://github.com/zhayujie/bot-on-anything)** — lightweight LLM application framework with integrations for Slack, Telegram, Discord, Gmail, and more
- **[AgentMesh](https://github.com/MinimalFuture/AgentMesh)** — open-source multi-agent framework for solving complex problems through team collaboration

<br/>

## 🏢 Enterprise Services

[**LinkAI**](https://link-ai.tech/?utm_source=cowagent&utm_medium=readme&utm_campaign=enterprise) is an all-in-one AI Agent platform for enterprises and developers, offering managed hosting and enterprise-grade support for CowAgent:

- **🚀 Zero-deployment hosted runtime** — spin up a [CowAgent online assistant](https://link-ai.tech/cowagent/create?utm_source=cowagent&utm_medium=readme&utm_campaign=enterprise) in under a minute, no server required
- **🧠 Agent infrastructure** — unified access to LLMs, knowledge bases, databases, skills, and workflows; plug-and-play building blocks that extend what CowAgent can do
- **🏢 Team & enterprise features** — workspaces, role-based access, audit logs, and private deployment for production use cases

For enterprise inquiries: sales@simple-future.tech or [scan the QR code](https://cdn.link-ai.tech/contact/customer-support.png) to reach our team on WeChat.

<br/>

## 🛠️ Development & Contributing

All kinds of contributions are welcome — new features, bug fixes, performance improvements, docs, or sharing your own skills on the [Skill Hub](https://skills.cowagent.ai/submit). See [CONTRIBUTING.md](CONTRIBUTING.md) to get started, then open an Issue to discuss or send a PR directly.

⭐ Star the project to show your support, and Watch → Custom → Releases to get notified of new versions. PRs and Issues are always welcome.

## 🌟 Contributors

![cow contributors](https://contrib.rocks/image?repo=zhayujie/CowAgent&max=1000)

<br/>

## ⚠️ Disclaimer

1. This project is licensed under the [MIT License](LICENSE) and is intended for technical research and learning. You are responsible for complying with applicable laws and regulations in your jurisdiction; the maintainers assume no liability for any consequences arising from use of this project.
2. **Cost & safety:** Agent mode consumes substantially more tokens than regular chat — pick models that balance quality and cost. The Agent has access to your local operating system, so only deploy it in trusted environments.
3. CowAgent is a pure open-source project and does not participate in, authorize, or issue any cryptocurrency.

<br/>

## 📌 Project Renaming Notice

This project was previously named `chatgpt-on-wechat` and is now officially **CowAgent**. The old GitHub URL redirects automatically; existing users may optionally run `git remote set-url origin https://github.com/zhayujie/CowAgent.git` to update the local remote.
