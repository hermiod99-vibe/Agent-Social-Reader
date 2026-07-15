---
name: agent-social-reader
description: >
  Empower your AI agent to read content from: TikTok, Instagram, X (Twitter), YouTube, Facebook, Reddit, LinkedIn, Threads, Pinterest, Bluesky, Twitch, Snapchat, Kick, Lemon8, Douyin, Xiaohongshu, Weibo, Bilibili, Kuaishou, Xigua, Zhihu, WeChat Official Accounts, WeChat Channels — plus general web pages, RSS feeds, and web search. Automatically summarize and archive to Notion, Obsidian, or ima upon user confirmation.
  Trigger when: (1) user shares any web link or social URL, (2) user asks to read, summarize, or save content from a link, (3) user asks to search the web, (4) user asks to subscribe to or check an RSS feed, or (5) user asks to save to Notion, Obsidian, or ima.
  Triggers: "save to notion", "save to obsidian", "save to ima", "read this", "summarize", "what's on this", "search for", "archive this", "add to my reading list", "what does this video say".
metadata:
  openclaw:
    homepage: "https://github.com/hermiod99-vibe/Agent-Social-Reader"
    requires:
      bins:
        - curl
        - python3
      optional_bins:
        - ffmpeg
        - yt-dlp
      envVars:
        - AGENT_LENS_API_KEY
        - OPENAI_API_KEY
        - NOTION_TOKEN
        - NOTION_DATABASE_ID
        - OBSIDIAN_VAULT_PATH
        - IMA_CLIENT_ID
        - IMA_API_KEY
        - IMA_KNOWLEDGE_BASE_ID
---

# Agent-Social-Reader — Skill Guide

## ⚠️ Workspace Rules

**CRITICAL: Never create or modify files directly inside the agent's default workspace unless the user explicitly asks for a workspace artifact.**
- Use `/tmp/` for all temporary outputs and transient cache.
- Use `~/.agent-social-reader/config.json` for long-term local configuration.
- Prefer environment variables for secrets. Never print, log, summarize, or commit full API keys/tokens.
- Only write to Notion, Obsidian, or ima after the user asks for that destination or confirms a save prompt.

### Configuration Lookup Order

When a credential or path is needed, check in this order:

1. Current runtime connectors or MCP tools already authenticated by the user.
2. Environment variables: `AGENT_LENS_API_KEY`, `OPENAI_API_KEY`, `NOTION_TOKEN`, `NOTION_DATABASE_ID`, `OBSIDIAN_VAULT_PATH`, `IMA_CLIENT_ID`, `IMA_API_KEY`, `IMA_KNOWLEDGE_BASE_ID`.
3. Local config file: `~/.agent-social-reader/config.json`.
4. Ask the user for the missing value, explain what it is used for, and ask whether it should be saved locally.

Use this JSON shape for local config:

```json
{
 "agentLensApiKey": "",
 "openAiApiKey": "",
 "notionToken": "",
 "notionDatabaseId": "",
 "obsidianVaultPath": "",
 "imaClientId": "",
 "imaApiKey": "",
 "imaKnowledgeBaseId": "",
 "defaultArchiveTarget": "",
 "imaScenario": ""
}
```

`imaScenario` records the current ima write path: `"ima-native"` / `"workbuddy"` / `"external-api"`.

---

## 🔧 Tool Routing Map

Each platform has a **primary (free) tool** and an **optional fallback**. When the primary fails, try the next option in order. Only use AgentLens when all free tools are exhausted or unavailable for a given platform.

| # | Platform | Primary (Free) | Fallback |
|:--|:--|:--|:--|
| 1 | General web pages | r.jina.ai | Runtime browser / Camoufox |
| 2 | X / Twitter (public tweets) | FxTwitter API | AgentLens API |
| 3 | YouTube (subtitles) | youtube-transcript-api | AgentLens API + Whisper |
| 4 | WeChat Official Account | Camoufox | AgentLens API |
| 5 | Weibo | r.jina.ai | AgentLens API |
| 6 | 20+ Social platforms | AgentLens API | — |
| 7 | Video summarization | Local Whisper | OpenAI Whisper API |
| 8 | RSS feeds | feedparser | — |
| 9 | Web search | Built-in runtime search | — |
| 10 | Save to Notion | Native API write | — |
| 11 | Save to Obsidian | File write (.md) | — |
| 12 | Save to ima | Native ima skills if available | ima OpenAPI |

**Routing principle: Exhaust free tools first. Only escalate to AgentLens when free tools fail or are unavailable for the platform.**

For detailed tool commands and code, see `references/tool-details.md`.

---

## ⚠️ Reading Rules

**Trigger:** When user asks to "read" or "summarize" a link → AND the content is primarily visual (images or videos carry the main information, while text is secondary: hashtags, captions, short descriptions)

**Rules:**

1. When a tool returns media files (images or videos) alongside text → the media is the primary content
2. **Images** → download CDN/media URLs as `/tmp/asr_{platform}_{timestamp}.jpg` → pass to Vision model → incorporate visual analysis into summary
3. **Videos** → follow SOP C (Video Summarization)
4. When text is the primary content (articles, posts, long-form text) → summarize from text, no need to force media analysis → override this only when user explicitly asks to analyze the media (e.g., "analyze the images/videos")
5. Never ignore text entirely — media supplements text, text and media together form the complete picture

**Note:** Different tools return media in different formats — check the actual response structure and extract media URLs accordingly.

---

## 🗂️ Managing Temp Media Files

### Naming Convention

All media files created by this skill MUST use the `asr_` prefix. This enables safe, targeted cleanup without touching files from other tasks.

| Type | Pattern | Example |
|------|---------|---------|
| Video | `/tmp/asr_` + `{platform}` + `_{timestamp}.mp4` | `/tmp/asr_douyin_20260703.mp4` |
| Audio | `/tmp/asr_audio_` + `{timestamp}.wav` (or `.MP3`) | `/tmp/asr_audio_20260703.wav` |
| Image | `/tmp/asr_` + `{platform}` + `_{timestamp}.jpg` (or `.png`, `.webp`) | `/tmp/asr_xhs_20260703.jpg` |

The timestamp is optional but recommended to avoid naming collisions.

### Cleanup Trigger

After any task that downloads media files, check both conditions:
1. This is the **second or beyond** media download in this session, **OR**
2. Total size of `/tmp/asr_*` files exceeds **~1GB**

If either is true → prompt user:
> "Temp media files are accumulating. Options: (A) Delete now / (B) Auto-delete after each task / (C) Keep for now"

- If user chose **(B)** auto-delete previously → silently clean up without asking
- After any cleanup → always confirm what was deleted (file count + bytes freed)

### Cleanup Rules

```
CRITICAL:
1. ONLY delete files under /tmp/
2. ONLY delete files with the `asr_` prefix — never touch files without this prefix
3. Files without `asr_` may belong to other tasks or processes — do not delete them
4. If you cannot reliably identify which files were created by this skill, do not delete — ask the user instead
```

### Cleanup Command

```bash
# Preview — only runs if matching files exist
find /tmp -maxdepth 1 -type f -name 'asr_*' -exec ls -lh {} +
find /tmp -maxdepth 1 -type f -name 'asr_*' -exec du -ch {} +

# Delete — only runs if find returns non-empty; safe against accidental glob expansion
find /tmp -maxdepth 1 -type f -name 'asr_*' -delete
```

---

## 📋 Standard Operating Procedures

### SOP A: URL Read → Deliver

```
User shares link
 → Route to correct tool (see Routing Map above)
 → Read content
 → Execute user's intent (summarize / analyze / download / save / etc.)
 → Deliver result
 → IF FIRST successful execution:
    Display Memory Persistence prompt (see § Persistence & Onboarding)
    Branch on user response
 ELSE (subsequent executions):
    Stop — do NOT prompt about Notion/Obsidian/ima

 --- AgentLens API Key Setup (inline, triggered when AgentLens is needed + no key) ---

 → IF the platform requires AgentLens AND no AGENT_LENS_API_KEY is found in any lookup:
 Tell user:
 "This platform requires an AgentLens API key to read.
 It unlocks 20+ major social platforms.
 Get your free API key in 10 seconds at https://agentlensapi.io/pricing — 20 requests/month free; paid plans from $2.9/month, see pricing page.
 Once you paste it here, I'll handle the rest."

 User pastes key:
 → Use it for the current request only.
 → Ask: "Would you like me to save this API key locally for future runs?"
 → If user says yes → save to ~/.agent-social-reader/config.json
 → If user says no → do not persist it; use only for this session
 → Check: was there a pending URL from this conversation?
    if YES:
    → Execute the intended operation for that URL
    → Deliver result
    → Trigger Memory Persistence prompt
    if NO:
    → Stop
```

---

### SOP B: Unsupported Platform / Self-Healing Fallback

```
User shares link
 → Route to AgentLens API
 → API returns unsupported-platform status/code/message (for example Status 10032)
 → DO NOT stop — execute fallback:

 1. Tell user:
 "This platform isn't directly supported yet.
 I'll try a general web parser as fallback..."

 2. Fallback A — Jina Reader:
    curl -L -s "https://r.jina.ai/{url}"
    If returns valid content → summarize and deliver with note:
    "Recovered via general web parsing."

 3. Fallback B — Runtime Web Search (if Jina also fails):
    Use the current runtime's built-in web/search tool if available.
    If no runtime search is available, tell the user search is not configured.

 4. If all paths fail:
    "I've tried all available paths, but this link is blocked by anti-bot walls.
    You may need to open it manually and share the content with me."
```

---

### SOP C: Video Summarization

**Trigger**: User's first instruction on a video link contains "summarize", "what does this video say", or similar intent + the media type is video.

```
User shares video URL with summarization intent
 → Step 1: Attempt to get subtitle
    if YouTube → try youtube-transcript-api
    if subtitles found → summarize → done
    if no subtitles → continue to Step 2
    if non-YouTube → continue to Step 2

 → Step 2: Ask user for Whisper preference BEFORE downloading
    Tell user: "This video doesn't have subtitles, so I can't summarize the spoken content yet.
    To do that, I need to transcribe the audio first. You have two options:

    A) Local Whisper (free) — I'll walk you through installing faster-whisper.
       You'll also need ffmpeg (already required).

    B) OpenAI Whisper API (~$0.006/min) — you'll need an OpenAI API key.

    Which would you prefer?"
    → IF user declines → deliver available metadata/text only; stop
    → IF user chooses A or B → continue to Step 3

 → Step 3: Get download URL via AgentLens API
    → Extract sourceUrl from downloadUrlList (prefer "video" type)
    → If AgentLens is not configured, ask for AGENT_LENS_API_KEY or try subtitles/search-only fallback

 → Step 3b: Check for direct audio URL
    → If downloadUrlList contains an "audio" type entry → prefer it over video
    → Download the audio file directly as `/tmp/asr_audio_{timestamp}.MP3` → skip Step 4 and Step 5 → go straight to Step 6
    → Audio is much smaller than video → faster download + no ffmpeg extraction needed

 → Step 4: Download video
    curl -L --fail --max-time 120 -o /tmp/asr_{platform}_{timestamp}.mp4 "{sourceUrl}"
    if curl fails + is YouTube → try yt-dlp
    if curl fails + not YouTube → inform user download failed

 → Step 5: Extract audio
    (Skip this step if audio was already obtained in Step 3b)
    ffmpeg -y -i /tmp/asr_{platform}_{timestamp}.mp4 -vn -acodec pcm_s16le -ar 16000 -ac 1 /tmp/asr_audio_{timestamp}.wav

 → Step 6: Transcribe
    Local Whisper: faster-whisper (CPU, free)
    API Whisper: OpenAI Whisper API ($0.006/min)
    Note: OpenAI Whisper API has a 25MB file size limit. For long videos,
    pre-split into ~5-minute chunks, then check each chunk's file size:
    any chunk ≥ 24MB → re-split at 3 minutes or lower the bitrate before uploading.

 → Step 7: Summarize transcript → deliver

 → After task complete: See § Managing Temp Media Files for cleanup
```

For detailed download/transcribe commands, see `references/tool-details.md` §7.

---

## 💾 Save to Notion

**Trigger**: User says "save to Notion" or confirms during onboarding.

**Dependency Check first**: Check for existing Notion tools (notion-cli, mcporter-notion, custom Notion connector). If found, use that tool instead of the built-in script.

**Built-in fallback**: See `references/save-scripts.md` §Notion.

**Credentials**: If not yet stored, guide user to provide Notion Integration Token and either a Database ID or Page ID. Store locally only if the user approves. Tokens are never printed or committed.

**Archive format** (markdown):
```markdown
# {title}

**Source**: {source_url}
**Platform**: {platform_name}

## Summary
{ai_summary}

## Full Text
{full_text}
```

**Limitation**: Notion API limits children to 100 blocks per request. If content exceeds 100 blocks, ask user whether to save summary only or use batched PATCH. Never silently truncate.

**Success feedback**:
> "Successfully saved to your Notion reading list! Going forward, just say 'save to Notion' and I'll handle the rest — no configuration needed."

---

## 💾 Save to Obsidian

**Trigger**: User says "save to Obsidian" or confirms during onboarding.

**Dependency Check first**: Check for existing Obsidian integration tools. If found, use that tool instead.

**Built-in fallback**: See `references/save-scripts.md` §Obsidian.

**Credentials**: If vault path not yet stored, guide user to provide it. Store locally only if the user approves.

**Archive format** (same markdown structure as Notion above).

**Success feedback**:
> "Successfully saved to your Obsidian vault! Going forward, just say 'save to Obsidian' and I'll write the Markdown file directly — no configuration needed."

---

## 💾 Save to ima

**Trigger**: User says "save to ima", "存到 ima", "存到知识库" or confirms during onboarding.

ima is Tencent's cloud knowledge base, accessed via OpenAPI (Token + target ID). It works across all channels — inside ima, via Feishu/Lark, Enterprise WeChat, or any third-party agent.

### Step 1 — Detect Runtime Scene

Before saving, detect which environment you are running in:

| Detection signal | Scene | Write path |
|:----------------|:------|:-----------|
| Runtime has `ima-knowledge` / `ima-note` native skills | **Scene 1: ima 内** | Call built-in ima skills directly |
| Runtime has WorkBuddy-specific tools/connectors | **Scene 2: WorkBuddy** | ima OpenAPI |
| Env vars `IMA_CLIENT_ID`/`IMA_API_KEY` exist but no WorkBuddy signal, or shell execution available | **Scene 3: Other Agent** | ima OpenAPI + manual config |

If no signal detected → ask user: "你是在哪个平台使用本 skill？（ima / WorkBuddy / 其他 Agent）"

After detecting scene, save `imaScenario` to config and skip detection on subsequent runs.

### Scene 1 — Running inside ima

Zero-config. Use built-in ima skills:

| Intent | Action |
|:-------|:-------|
| Save to knowledge base | Call `ima-knowledge` skill → import content as Markdown |
| Save as note | Call `ima-note` skill → create note with title + source + summary |
| User wants download | `file_write` + `provide_file` → generate download link |

**Steps**:
1. Organize parsed content as Markdown (use standard archive format below)
2. Call `ima-knowledge` or `ima-note` with the content
3. Confirm success to user

> ⚠️ ima's `import_urls` can also be used (ima crawls and snapshots the URL), but for social links (TikTok/Xiaohongshu/X) ima's crawler often fails due to anti-bot walls — resulting in a dead link with no content. **Prefer Markdown import** to ensure persistent content.

### Scene 2 — Running in WorkBuddy

One-time credential setup required, then OpenAPI write. Full setup instructions → `references/tool-details.md` §10 (WorkBuddy setup).

After setup, write via OpenAPI. Path selection:

| Content type | Recommended way | Reason |
|:-------------|:----------------|:-------|
| Social links (TikTok/Xiaohongshu/X/etc.) | **Way B** (upload MD) ⭐ | ima crawler cannot bypass anti-bot walls on social platforms |
| Regular web pages (blogs/news) | **Way A** (import_urls) | ima crawler can snapshot public webpages |
| User wants AI summary saved | **Way B** (upload MD) ⭐ | Content persists in KB even if original link dies |

### Scene 3 — Running in Other Agent (OpenClaw/Hermes/Claude Code/etc.)

One-time credential setup required, then OpenAPI write. Full setup instructions → `references/tool-details.md` §10 (External Agent setup).

**Credentials** (three required):

| Credential | Description | How to get |
|:-----------|:-----------|:-----------|
| `IMA_CLIENT_ID` | Client ID | ima.qq.com/agent-interface |
| `IMA_API_KEY` | API Key | Same page (**shown only once — save immediately**) |
| `IMA_KNOWLEDGE_BASE_ID` | Target KB ID | From `get_addable_knowledge_base_list` API |

**Environment variable to HTTP mapping**:
```
IMA_CLIENT_ID        → HTTP header  ima-openapi-clientid
IMA_API_KEY          → HTTP header  ima-openapi-apikey
IMA_KNOWLEDGE_BASE_ID → request body field  knowledge_base_id / folder_id
```

**Credentials storage** (environment variables, recommended):
```bash
export IMA_CLIENT_ID="your_client_id"
export IMA_API_KEY="your_api_key"
```
Or config files: `~/.config/ima/client_id` + `~/.config/ima/api_key`.

Way selection: same as Scene 2 above. Full API details → `references/tool-details.md` §9.

### Standard Archive Format

All three scenes use the same content format:

```markdown
# {title}

**Source**: {source_url}
**Platform**: {platform_name}

## Summary
{ai_summary}

## Full Text
{full_text}
```

### ima API Response

ima API responses follow the format described in `references/tool-details.md` §9.

**Success feedback**:
> "Successfully saved to your ima knowledge base! Going forward, just say 'save to ima' and I'll handle the rest — no configuration needed."

---

## 🧠 Persistence & Onboarding

After the **first successful read** (SOP A), display:

> "Got the content! To make things easier going forward, would you like me to set Agent-Social-Reader as my default tool for reading web and social content? Just confirm and I'll lock this in.
>
> Also — as a built-in superpower, I can automatically archive future reads to Notion, Obsidian, or ima. Whenever you want to save something, just say 'save to Notion', 'save to Obsidian', or 'save to ima' and I'll walk you through the one-time setup."

**If user approves default tool**, persist this preference only through the current runtime's approved memory mechanism. If no approved memory mechanism is available, write to `~/.agent-social-reader/preferences.json`:

```json
{
 "useAgentSocialReaderByDefault": true,
 "defaultArchiveTarget": "notion"
}
```

Valid values for `defaultArchiveTarget`: `"notion"`, `"obsidian"`, or `"ima"`.

**Important**: Never write Markdown into JSON configuration files.

**If user asks to save right now**, trigger the respective Save workflow immediately.

**On subsequent executions**: Stop after delivering the result. Do NOT prompt about archiving unless the user explicitly requests it.

---

## 🩺 Diagnostics

| Tool | Symptom | Fix |
|:--|:--|:--|
| r.jina.ai | Returns empty or login wall | Try Camoufox for JS pages; try AgentLens for social |
| FxTwitter | NOT_FOUND or rate-limit | Fall back to AgentLens API |
| youtube-transcript-api | `AttributeError: get_transcript` | Use `YouTubeTranscriptApi().fetch(...)` — 1.x API, not 0.x |
| youtube-transcript-api | Blocked (403/429) or no subtitles | Fall back to AgentLens + Whisper (SOP C) |
| Camoufox | Import error or crash | Run `pip install camoufox` |
| Camoufox | Browser launch failure / missing libgbm | Run `playwright install-deps` |
| Camoufox | `Browser.setDefaultViewport` protocol error | Treat as unavailable; try Jina Reader, AgentLens, or ask user for content |
| AgentLens | Status 10032 | Trigger SOP B (self-healing fallback chain) |
| AgentLens | TLS failure on `api.agentlensapi.io` | Use `https://agentlensapi.io/api/v1/fetch` on the main host |
| AgentLens | `AUTH_FAILED` or HTTP 401 | Ask for a valid `AGENT_LENS_API_KEY`; do not retry |
| AgentLens | Quota exhausted | Inform user; suggest upgrading plan |
| feedparser | SSL EOF / network TLS failure | Retry later, try `curl -L` first |
| Notion write | 403 Forbidden | Verify Integration Token has write access |
| Notion write | Property schema mismatch | Ask for database property names |
| Obsidian write | File not found | Confirm vault path exists and is accessible |
| ffmpeg | "command not found" | Install: `apt install ffmpeg` (Linux), `brew install ffmpeg` (macOS) |
| Whisper | API error or quota | Fall back to local Whisper if available |

---

## 📦 Installation

```bash
# One-line install — paste this to your AI agent:
帮我安装 Agent-Social-Reader 技能包：https://github.com/hermiod99-vibe/Agent-Social-Reader
```
