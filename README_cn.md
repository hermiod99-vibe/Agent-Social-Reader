# 👁️ Agent-Social-Reader

<p align="center">
  <strong>丢个社媒链接  →  Agent自动总结  →  一键存进知识库<br/>免登录 · 免 Cookie · 账号更安心<br/><br/>
  支持读取 X、Instagram、YouTube、抖音、小红书、微信公众号等社交内容，并一键归档到 Notion / Obsidian</strong>
</p>
<p align="center">
  <a href="#-快速理解">快速理解</a> · 
  <a href="#-功能列表">功能列表</a> · 
  <a href="#-支持平台">支持平台</a> · 
  <a href="#-快速上手">快速开始</a> · 
  <a href="README.md">English</a>
</p>




---

## 🎯 快速理解

作为知识工作者/产品经理/重度 AI Agent 使用者，我们每天丢给 AI 各种 X (Twitter)、Reddit、微信公众号或小红书链接时，最常遇到的就是 **AI 访问不了这些社交媒体平台**。即使用了各种硬核开源工具，也会遇到各种**平台限制和数据格式碎片化**问题，导致 AI 吐回来一堆报错和 HTML 乱码。

**这个 Skill 解决的问题**：

❌ 之前： 

```
你：丢给 Agent 一个 Twitter/Reddit/小红书链接 
Agent："我访问不了" / 返回乱码 
你：只能手动复制粘贴，或者截图上传，或者放弃
```

✅ 现在： 

```
你：丢给 Agent 任何社交媒体链接 
Agent：自动读取 → 自动总结 
你：把它存到 Notion
→ 内容整整齐齐存进知识库
```

**核心价值**：
- ✅ **完整闭环**：阅读 → 总结 → 存档，一条龙搞定
- ✅ **广泛覆盖**：20+ 全球社交平台（TikTok、Instagram、X、YouTube、Reddit、抖音、小红书、B站等）
- ✅ **更低风险**：不需要登录个人社交账号，不需要 Cookie，降低因 Cookie 导致的封号风险
- ✅ **灵活成本**：免费工具优先，可选付费增强（最低 $2.90/月）

> 💡 **设计理念**：能免费的绝不花钱，免费搞不定的再用性价比最高的付费方案。用户自己决定用哪个。

> ❤️ **特别致谢：**[Agent-Reach](https://github.com/Panniantong/Agent-Reach) by [@Panniantong](https://github.com/Panniantong)。Agent-Reach 证明了让 Agent 读取网络内容这件事是可以做到的 - 本项目是同一思路的个人迭代，更聚焦社交内容读取与知识库归档的完整闭环。

---

## ✨ 功能列表

安装这个 Skill 之后，你的 Agent 会获得以下能力（**无需你写任何代码**，告诉 Agent，Agent 自动执行）：

| 内容来源                             | 使用工具               | 成本说明                                            |
| ------------------------------------ | ---------------------- | --------------------------------------------------- |
| 普通网页                             | Jina Reader (r.jina.ai) | ✅ 免费，无需 Key                                    |
| X/Twitter 公开推文                   | FxTwitter API          | ✅ 免费，无需 Key 或 cookie                          |
| YouTube（有字幕）                    | youtube-transcript-api | ✅ 免费，无需 Key                                    |
| 微博                                 | Jina Reader (r.jina.ai) | ✅ 免费，绕过登录墙                                |
| 微信公众号                           | Camoufox               | ✅ 免费，无头浏览器处理 JS 渲染<br/>（失败时回退到 AgentLens 或提示你粘贴正文）                 |
| **20+ 全球社交平台**                 | AgentLens API          | 🎁 每月20 次免费使用额度<br>💵 之后 $2.90/月（200 次） |
| 视频总结（TikTok、YouTube 等无字幕） | Whisper 转录 + LLM     | ✅ 本地免费（需安装）<br>💵 OpenAI API $0.006/分钟    |
| RSS 订阅                             | Python feedparser      | ✅ 免费                                              |
| 全网语义搜索                         | Agent runtime 内置搜索   | ✅ 免费（前提是当前运行时支持）                         |
| 一句话存 Notion                 | Agent 确认后写入         | ✅ 免费（你的 Notion Token）                         |
| 一句话存 Obsidian                   | Agent 确认后写入         | ✅ 免费（本地路径）                                  |

---

## 🌐 支持平台

经测试，通过 **AgentLens API** 可访问以下社交媒体平台（如发现有误或还支持更多平台，欢迎告知）：


| 全球主流平台   | 中国主流平台       |
| :------------- | :----------------- |
| TikTok         | 抖音               |
| Instagram      | 小红书             |
| YouTube        | 哔哩哔哩           |
| X (Twitter)    | 微博               |
| Facebook       | 快手               |
| Threads        | 西瓜视频           |
| Reddit         | 知乎（专栏）       |
| LinkedIn       | 微信公众号（文章） |
| Twitch (clips) | 微信视频号         |
| Pinterest      |                    |
| Bluesky        |                    |
| Snapchat       |                    |
| Kick (clips)   |                    |
| Lemon8         |                    |

**AgentLens 能读取**：主文本内容 + 配图/视频文件（获取原始媒体文件链接）  
**AgentLens 不读取**：评论区、时间线、X Lists、私密账号内容，及 X 和 Reddit 等平台的 Thread/对话流

---

## 🚀 快速上手

### 第一步：安装 Skill（10 秒）

复制下面这句话，直接发给你正在使用的 AI Agent（Claude Code、Cursor、Codex、OpenClaw、Windsurf 等均可）：

```
帮我安装 Agent-Social-Reader 技能包：https://github.com/hermiod99-vibe/Agent-Social-Reader
```

就这一步。Agent 会自己完成剩下的所有事情。

---

### 第二步：开始使用（立即）

装完就能立即使用以下**完全免费**的功能：

- "帮我看看这个微博"
- "总结一下这篇公众号文章"
- "这个网页讲了什么"
- "订阅这个 RSS 源"

---

### 第三步：可选增强（按需配置）

#### 🔓 解锁 20+ 社交平台读取（TikTok、Reddit、Instagram、抖音、小红书、B站等；X 单条公开推文、微博、微信公众号文章优先走免费工具，失败时使用 AgentLens）

当你第一次让 Agent 读取这些平台的内容时，Agent 会提示：

> "读取这个平台的内容，我需要AgentLens的API key。AgentLens能读取20+主流社交媒体的内容，并提供每月 20 次免费使用额度，之后是 $2.90/月（200 次）。如果需要，点击 https://agentlensapi.io/pricing 注册（10 秒），把 API key 复制给我，就搞定了。"

完整定价：[agentlensapi.io/pricing](https://agentlensapi.io/pricing)

---

#### 🎥 解锁视频内容总结（TikTok、YouTube等无字幕视频）

如果你需要总结没有字幕的视频内容，有两种方案：

**方案 A：完全免费（需要配置）**

本地安装 Whisper：

```bash
pip install faster-whisper
# macOS: brew install ffmpeg | Ubuntu: sudo apt install ffmpeg
```

安装好后告诉 Agent：

> "我已安装本地 Whisper，使用 CPU 模式运行"

如果当前 Agent 支持记忆功能且你同意，Agent 可记住你的偏好；之后遇到无字幕视频时会自动走"下载 → 提取音频 → 本地转写 → 总结"流程。

> ⚠️ 硬件要求：小模型（tiny/base）在普通电脑可跑；大模型（large-v3）推荐有 GPU。首次使用会自动下载模型（75MB~3GB）。

**方案 B：付费便捷（推荐普通用户）**

在 [OpenAI 平台](https://platform.openai.com/api-keys) 获取 Whisper API Key，告诉 Agent：

> "这是我的 OpenAI API Key: [YOUR_KEY]"

如果当前 Agent 支持记忆功能且你同意，Agent 可记住你的偏好；之后遇到无字幕视频时自动调用。

> 💡 费用低：$0.006/分钟。一个 1 分钟 TikTok 约 $0.006，一个 10 分钟 YouTube 约 $0.06。

------

#### 📁 连接知识库（Notion / Obsidian）

当你第一次说"存到 Notion"或"存到 Obsidian"时，Agent 会引导你完成一次性配置：

- **Notion**：需要 Integration Token 和 Database ID 或 Page ID（凭据存本地，端到端直连）
  💡 默认写入使用 Name（标题）和 Source（URL）属性；如数据库属性名不同或想写入单个页面而非数据库，首次保存时请告知 Agent
- Obsidian：需要本地知识库路径

在同一运行环境中配置一次，之后通常无需重复配置。

------

### 第四步：一次配置，长久自动

当你配置好并第一次成功让 Agent 读取、总结、或保存完一条内容后，Agent 会主动问你：

> "已经帮您整理好了！为了让您以后更省心，建议把 Agent-Social-Reader 设为我读取网络和社交媒体内容的默认工具。如果同意，我会写入长期记忆或本地偏好配置。以后您直接丢链接，我会自动调用这个工具。"

一旦确认，以后你只需要说：

- "帮我看看这篇推特写了啥"
- "把这个存到 Notion"

Agent 会自动调用相应工具，不需要每次都提 Skill 名字。如需保存到 Notion，直接说「存到 Notion」即可。

## 👍🏻 我的日常使用流程

```
我分享链接 → Agent 读取 → Agent 总结 → 我说"把它存到Notion里" → Notion 里就有了
```

存进 Notion/Obsidian 的内容包括：

- 原始链接
- 完整正文
- AI 总结

方便你后续检索和回顾。

------

## 💰 费用说明（完全透明）

根据 Skill 市场规定，使用付费服务需明示价格。以下是完整费用说明：

### 完全免费的功能

- ✅ 普通网页（r.jina.ai）
- ✅ 单条 X/Twitter 公开推文（FxTwitter API）
- ✅ YouTube 有字幕视频 （youtube-transcript-api）
- ✅ 微博（r.jina.ai）
- ✅ 微信公众号（Camoufox）
- ✅ RSS 订阅（feedparser）
- ✅ 全网搜索（Agent runtime 内置搜索）
- ✅ Notion/Obsidian 存档
- ✅ 本地 Whisper 视频转录（需自己安装）

### 可选付费功能

#### 📱 AgentLens API（20+ 社交平台读取）

| 套餐   | 每月次数 | 价格     |
| :----- | :------- | :------- |
| 免费版 | 20 次    | $0       |
| Basic  | 200 次   | $2.90/月 |
| Ultra  | 500 次   | $5.90/月 |
| Mega   | 1,000 次 | $9.90/月 |

完整定价：[agentlensapi.io/pricing](https://agentlensapi.io/pricing)

为什么选择 AgentLens（我的使用体验）：

- 一个 KEY 搞定 20+ 平台（vs 每个平台配一个工具）
- 无需 Cookie（vs 担心封号）
- 无需代理（vs 自己维护 IP 池）
- 稳定维护（vs 平台规则变化要自己追踪）

对我来说，$2.90/月（200 次，够用一个月）换来省心，性价比很高。

------

#### 🎥 OpenAI Whisper API（视频转录）

- 价格：约 $0.006/分钟
- 示例：
  - 1 分钟 TikTok：$0.006
  - 10 分钟 YouTube：$0.06
  - 30 分钟播客：$0.18

完整定价：[OpenAI Pricing - Whisper](https://platform.openai.com/docs/pricing)

> 💡 如果你设备配置足够且愿意自己配置，可以本地安装 Whisper（`pip install faster-whisper`），完全免费。

------

## 💡 设计理念

聚焦完成"阅读 → 总结 → 存档"的场景闭环。

在这个闭环中：

### 阅读层（免费优先）

| 内容来源           | 工具方案               | 成本   |
| :----------------- | :--------------------- | :----- |
| 普通网页           | Jina Reader            | ✅ 免费 |
| X/Twitter 公开推文 | FxTwitter API          | ✅ 免费 |
| YouTube 字幕       | youtube-transcript-api | ✅ 免费 |
| 微博               | r.jina.ai              | ✅ 免费 |
| 微信公众号         | Camoufox               | ✅ 免费 |
| RSS 订阅           | feedparser             | ✅ 免费 |
| 20+ 社交平台       | AgentLens API          | 💵 可选 |

🔐 **核心优势：**免社媒账号登录 / 不使用 Cookie
底层完全不需要你登录任何个人社交账号、输入密码或提供 Cookie，因此平台无法追踪到你的个人账号，可降低因使用个人账号 Cookie 带来的封号风险。

------

### 总结层（调用 LLM）

| 内容类型       | 总结方式                                         |
| :------------- | :----------------------------------------------- |
| 文本、图片     | 直接调 LLM 总结                                  |
| YouTube 有字幕 | 字幕文本 → LLM 总结                              |
| 视频无字幕     | 音频转写 （✅本地免费，或💵可选付费API）→ LLM 总结 |

------

### 存档层（完全免费）

Notion 和 Obsidian 均免费。需要时只需对 Agent 说"存 Notion"或"存 Obsidian"，Agent 会引导你完成一次性配置，之后当你再次要求保存时，可直接自动写入，无需重复配置。

存档内容包括：

- 原始链接
- 完整正文
- AI 总结

------

## 🔒 安全与隐私

- 账号更安全（无需登录）：市面上不少方案需要你安装插件或导出 Cookie 才能实现内容获取。本 Skill 完全不需要你登录任何个人社交账号或提供 Cookie，从而降低因使用个人账号 Cookie 带来的封号风险。
- 端到端直连：无论是你的 Notion Token、Obsidian 本地路径，还是 AgentLens 的 API KEY，全部由你的 Agent 严格存储在你自己电脑的本地文件（~/.agent-social-reader/config.json 或本地环境变量）中。在发起请求时，Agent 直接从当前运行环境调用各服务方 API，凭据仅发送给对应服务方；第三方服务的数据处理以其官方隐私政策为准。

------

## 🛠️ 技术栈

<details>
<summary>👀 点击展开技术栈列表</summary>

本 Skill 集成了以下工具和服务：

**免费开源工具及解析库：**
- [Jina Reader](https://jina.ai/reader) - 普通网页内容提取
- [FxTwitter API](https://github.com/FixTweet/FxTwitter) - X/Twitter 公开推文解析
- [youtube-transcript-api](https://github.com/jdepoix/youtube-transcript-api) - YouTube 既有字幕提取
- [yt-dlp](https://github.com/yt-dlp/yt-dlp) - 视频下载 fallback（curl 失败时用于提取视频以便转写）
- [Jina Reader](https://r.jina.ai) - 微博内容提取
- [Camoufox](https://github.com/daijro/camoufox) - 微信公众号动态渲染与读取
- Agent runtime 内置搜索 - 全网语义搜索（无额外依赖）
- Python `feedparser` - RSS 订阅源解析

**第三方可选增强 API（用户按需选择）：**
- [AgentLens API](https://agentlensapi.io) - 20+ 社交媒体平台公开内容标准化统一入口
- [OpenAI Whisper API](https://platform.openai.com/docs/guides/speech-to-text) - 远程音视频高精度转录

**本地处理方案（完全免费）：**
- [faster-whisper](https://github.com/SYSTRAN/faster-whisper) - 本地运行的 Whisper 语音转文字引擎

------

</details>

## 🙏 写在最后

这是我第一次将自己的 Skill 发布，欢迎大家多多提意见。

如果遇到问题尽管提 issue，收到后我会第一时间修复。

如果我发现有新的工具或解决方案，会第一时间加上。

大家有新需求或者想要的渠道，欢迎提 PR。

如果这个 Skill 帮你的 Agent 打通了从「社媒阅读 → 知识归档」的一条龙流程，顺手点个 ⭐ Star 支持一下吧！

**MIT License**
