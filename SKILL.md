---
name: agent-social-reader
version: 1.0.2
description: >
  Empower your AI agent to read content from: TikTok, Instagram, X (Twitter), YouTube, Facebook, Reddit, LinkedIn, Threads, Pinterest, Bluesky, Twitch, Snapchat, Kick, Lemon8, Douyin, Xiaohongshu, Weibo, Bilibili, Kuaishou, Xigua, Zhihu, WeChat Official Accounts, WeChat Channels — plus general web pages, RSS feeds, and web search. Automatically summarize and archive to Notion or Obsidian upon user confirmation.
  Trigger when: (1) user shares any web link or social URL, (2) user asks to read, summarize, or save content from a link, (3) user asks to search the web, (4) user asks to subscribe to or check an RSS feed, or (5) user asks to save to Notion or Obsidian.
  Triggers: "save to notion", "save to obsidian", "read this", "summarize", "what's on this", "search for", "archive this", "add to my reading list", "what does this video say".
metadata:
  openclaw:
    homepage: "https://github.com/inkad/agent-social-reader"
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
      # All envVars above are optional. None are required at install time — each is only
      # needed when the corresponding feature is triggered (e.g., OPENAI_API_KEY is
      # needed only when choosing OpenAI Whisper API for video transcription).
---

# Agent-Social-Reader — Skill Guide

## ⚠️ Workspace Rules

**CRITICAL: Never create or modify files directly inside the agent's default workspace unless the user explicitly asks for a workspace artifact.**
- Use `/tmp/` for all temporary outputs and transient cache.
- Use `~/.agent-social-reader/config.json` for long-term local configuration when the current agent does not provide a native secret store.
- Prefer environment variables for secrets. Never print, log, summarize, or commit full API keys/tokens.
- Only write to the agent's long-term memory if the current runtime supports it and the user explicitly approves the exact memory update.
- Only write to a Notion page/database or Obsidian vault after the user asks for that destination or confirms a save prompt.

### Configuration Lookup Order

When a credential or path is needed, check in this order:

1. Current runtime connectors or MCP tools already authenticated by the user.
2. Environment variables: `AGENT_LENS_API_KEY`, `OPENAI_API_KEY`, `NOTION_TOKEN`, `NOTION_DATABASE_ID`, `OBSIDIAN_VAULT_PATH`.
3. Local config file: `~/.agent-social-reader/config.json`.
4. Ask the user for the missing value, explain what it is used for, and ask whether it should be saved locally.

Use this JSON shape for local config:

```json
{
  "agentLensApiKey": "",
  "openAiApiKey": "",
  "notionToken": "",
  "notionDatabaseId": "",
  "obsidianVaultPath": ""
}
```

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

**Routing principle: Exhaust free tools first. Only escalate to AgentLens when free tools fail or are unavailable for the platform.**

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
      Stop — do NOT prompt about Notion/Obsidian

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
     Present publicly available references.
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
      if non-YouTube → skip to Step 2

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

  → Step 4: Download video
      curl -L --fail --max-time 120 -o /tmp/video.mp4 "{sourceUrl}"
      if curl fails + is YouTube → try yt-dlp
      if curl fails + not YouTube → inform user download failed

  → Step 5: Extract audio
      ffmpeg -y -i /tmp/video.mp4 -vn -acodec pcm_s16le -ar 16000 -ac 1 /tmp/audio.wav
      Note: 16kHz mono WAV is the recommended format for Whisper (both local and API),
      and avoids libmp3lame dependency issues in some environments.

  → Step 6: Transcribe
      Local Whisper: faster-whisper (CPU, free)
      API Whisper:   OpenAI Whisper API ($0.006/min)
      Note: OpenAI Whisper API has a 25MB file size limit. For long videos,
      pre-split the audio into ~10-minute chunks:
    mkdir -p /tmp/audio_chunks
    ffmpeg -y -i /tmp/audio.wav -f segment -segment_time 600 -c copy /tmp/audio_chunks/chunk_%03d.wav
    If multiple chunks were transcribed → concatenate all text parts in order before summarization.

  → Step 7: Summarize transcript → deliver
```

---

## 🛠️ Platform Tool Details

### 1. General Web Pages

**Tool**: r.jina.ai
**API key**: None (free)
**Command**:
```bash
curl -L -s "https://r.jina.ai/{url}"
```
Returns clean Markdown. Works for most public web pages.

**Note**: `{url}` must be the full original URL including `https://`, for example:
`curl -L -s "https://r.jina.ai/https://example.com/article"`

**Fallback**: If this fails, the page likely requires JS rendering — try a runtime browser tool or Camoufox for complex pages.

---

### 2. X / Twitter — Public Tweets

**Tool**: FxTwitter API
**API key**: None (free, no login or cookie required)
**Code**:
```python
import urllib.request, urllib.error, re, json

def fetch_tweet(url):
    pattern = r'(?:x\.com|twitter\.com)/([a-zA-Z0-9_]{1,15})/status/(\d+)'
    match = re.search(pattern, url)
    if not match:
        return {"error": f"Cannot parse tweet URL: {url}"}
    username, tweet_id = match.group(1), match.group(2)
    api_url = f"https://api.fxtwitter.com/{username}/status/{tweet_id}"
    req = urllib.request.Request(api_url, headers={"User-Agent": "Mozilla/5.0"})
    try:
        with urllib.request.urlopen(req, timeout=30) as resp:
            data = json.loads(resp.read().decode())
    except urllib.error.HTTPError as e:
        return {"error": f"FxTwitter HTTP {e.code}: could not fetch tweet (may be deleted or private)"}
    except Exception as e:
        return {"error": f"FxTwitter error: {e}"}
    if data.get("code") != 200:
        return {"error": f"FxTwitter error: {data.get('message')}"}
    tweet = data["tweet"]
    return {
        "text": tweet.get("text", ""),
        "author": tweet.get("author", {}).get("name", ""),
        "screen_name": tweet.get("author", {}).get("screen_name", ""),
        "likes": tweet.get("likes", 0),
        "retweets": tweet.get("retweets", 0),
        "views": tweet.get("views", 0),
        "replies": tweet.get("replies", 0),
        "created_at": tweet.get("created_at", ""),
        "media": tweet.get("media", {}).get("all", []),
        "quote": tweet.get("quote", {})
    }
```
**Fallback**: AgentLens API (use when FxTwitter returns NOT_FOUND or rate-limit error)
**Limitation**: Thread replies, user timelines, X Lists, and private accounts are not supported by any free tool. AgentLens does not support these either.

---

### 3. YouTube — Subtitles

**Tool**: youtube-transcript-api
**API key**: None (free)
**Installation**: `pip install youtube-transcript-api`
**Code**:
```python
import re

def get_youtube_subtitle(video_url, lang="en"):
    from youtube_transcript_api import YouTubeTranscriptApi
    patterns = [
        r'(?:youtube\.com/watch\?.*?v=)([a-zA-Z0-9_-]{11})',
        r'(?:youtu\.be/)([a-zA-Z0-9_-]{11})',
        r'(?:youtube\.com/shorts/)([a-zA-Z0-9_-]{11})',
    ]
    video_id = None
    for pat in patterns:
        m = re.search(pat, video_url)
        if m:
            video_id = m.group(1)
            break
    if not video_id:
        return {"error": "Cannot parse YouTube URL"}
    # Language priority: requested lang first, then zh-Hans/zh/en, then any available
    langs = [lang, "zh-Hans", "zh", "en"]
    try:
        # Try 0.x API: YouTubeTranscriptApi.get_transcript(video_id, languages=langs)
        transcript_list = YouTubeTranscriptApi.get_transcript(video_id, languages=langs)
        entries = [
            {"text": entry["text"], "start": entry["start"], "duration": entry["duration"]}
            for entry in transcript_list
        ]
        text = " ".join(e["text"] for e in entries)
        return {"video_id": video_id, "language": lang, "transcript": text, "entries": entries}
    except Exception:
        pass
    # Fallback to 1.x API: YouTubeTranscriptApi().fetch(video_id, languages=langs)
    try:
        api = YouTubeTranscriptApi()
        fetched = api.fetch(video_id, languages=langs)
        entries = [
            {"text": s.text, "start": s.start, "duration": s.duration}
            for s in fetched
        ]
        text = " ".join(e["text"] for e in entries)
        return {"video_id": video_id, "language": lang, "transcript": text, "entries": entries}
    except Exception as e:
        return {"error": f"YouTube transcript unavailable: {e}. Try video summarization via SOP C."}
```
**Transcript source priority**: Creator-uploaded subtitles → auto-generated subtitles.
**Fallback**: If blocked (403/429 from YouTube), fall back to AgentLens API + Whisper (see SOP C).
**Limitation**: Subtitles must be enabled on the video. If fully disabled, use SOP C.

---

### 4. WeChat Official Account

**Tool**: Camoufox (headless browser for JS-rendered content)
**API key**: None (free)
**Installation**: `pip install camoufox`
**Compatibility note**: Camoufox may download a large browser bundle on first launch and can fail with Playwright protocol errors on some Python/browser-version combinations. Treat Camoufox as a best-effort renderer, not a guaranteed dependency.
**Code**:
```python
import asyncio
from camoufox.async_api import AsyncCamoufox

async def read_wechat(url):
    async with AsyncCamoufox(headless=True) as browser:
        page = await browser.new_page()
        try:
            await page.goto(url, wait_until="networkidle", timeout=30000)
            await asyncio.sleep(3)
            content = await page.locator("body").inner_text()
            return content
        finally:
            await page.close()

print(asyncio.run(read_wechat("https://mp.weixin.qq.com/s/ARTICLE_ID"))[:5000])
```
**Fallback**: If Camoufox fails or is unavailable, try AgentLens API if configured. If all automated paths fail, ask the user to share copied text or screenshots.

---

### 5. Weibo

**Tool**: r.jina.ai
**API key**: None (free)
**Command**:
```bash
curl -L -s "https://r.jina.ai/{weibo_url}"
```
r.jina.ai handles Weibo's mobile pages and bypasses login walls in most cases.
**Fallback**: AgentLens API.

---

### 6. AgentLens API — 20+ Social Platforms

**Purpose**: Mandatory handling channel for all platforms not covered by free tools above.
**Primary endpoint**: `POST https://agentlensapi.io/api/v1/fetch`
**Fallback endpoint**: `POST https://agentlensapi.io/v1/fetch`
**Avoid by default**: `https://api.agentlensapi.io/v1/fetch` because some local proxy/fake-IP environments can fail TLS for the API subdomain while the main host works.
**Auth**: `Authorization: Bearer {AGENT_LENS_API_KEY}`

#### Supported Platforms

| Global Platforms | Chinese Platforms |
|:---|:---|
| TikTok | Douyin |
| Instagram | Xiaohongshu |
| YouTube | Bilibili |
| X (Twitter) | Weibo |
| Facebook | Kuaishou |
| Threads | Xigua |
| Reddit | Zhihu (columns) |
| LinkedIn | WeChat Official Account (articles) |
| Twitch (clips) | WeChat Channels |
| Pinterest | |
| Bluesky | |
| Snapchat | |
| Kick (clips) | |
| Lemon8 | |

**AgentLens reads**: main text content + media files (images/videos via `downloadUrlList`)
**AgentLens does NOT read**: comment sections, X/Reddit thread conversations

#### API Request
```http
POST https://agentlensapi.io/api/v1/fetch
Authorization: Bearer {AGENT_LENS_API_KEY}
Content-Type: application/json

{ "url": "https://..." }
```

#### Response Fields
```yaml
data.status:       HTTP-style status code (200 = success)
data.message:      Success message or descriptive error
data.data.platform: Platform identifier (e.g., "youtube", "tiktok")
data.data.name:     Author / channel / username
data.data.title:    Content title
data.data.description: Full text content
data.data.downloadUrlList[].type:    "video" or "pic"
data.data.downloadUrlList[].sourceUrl: Direct media URL
data.data.subtitle: Transcript if available
```
Some error responses may use a top-level envelope instead:
```yaml
success: false
error.code: AUTH_FAILED
error.message: API Key invalid or disabled
```
Parse both shapes.

#### Core Retrieval Script
```python
import urllib.request, urllib.error, json

AGENTLENS_ENDPOINTS = [
    "https://agentlensapi.io/api/v1/fetch",      # primary: main host (no TLS issues)
    "https://agentlensapi.io/v1/fetch",      # fallback: same host, avoids api. subdomain TLS issues
]

def fetch_social_content(url, api_key):
    last_error = None
    for endpoint in AGENTLENS_ENDPOINTS:
        req = urllib.request.Request(
            endpoint,
            data=json.dumps({"url": url}).encode(),
            headers={
                "Authorization": f"Bearer {api_key}",
                "Content-Type": "application/json"
            },
            method="POST"
        )
        try:
            with urllib.request.urlopen(req, timeout=60) as resp:
                result = json.load(resp)
        except urllib.error.HTTPError as e:
            body = e.read().decode("utf-8", "replace")
            try:
                result = json.loads(body)
            except json.JSONDecodeError:
                return {"error": f"AgentLens HTTP {e.code}: {body[:300]}"}
        except Exception as e:
            last_error = f"{type(e).__name__}: {e}"
            continue

        envelope = result.get("data", result)
        status = envelope.get("status") or result.get("status")
        if str(status) == "200" or result.get("success") is True:
            content = envelope.get("data", envelope)
            return {
                "platform": content.get("platform"),
                "author": content.get("name"),
                "title": content.get("title"),
                "description": content.get("description"),
                "media": content.get("downloadUrlList", []),
                "subtitle": content.get("subtitle")
            }

        error = result.get("error") or {}
        code = error.get("code") or envelope.get("code") or status
        message = error.get("message") or envelope.get("message") or result.get("message") or "Unknown error"
        status_str = str(status)
        code_str = str(code).lower()
        message_str = str(message).lower()

        if status_str == "10032" or "unsupported" in code_str or "unsupported" in message_str or "not supported" in message_str:
            return {"error": message, "code": code, "status": status, "action": "trigger_sop_b"}

        return {"error": message, "code": code, "status": status}

    return {"error": last_error or "AgentLens request failed before receiving a response"}
```

#### Error Handling
- **Status 200**: Success — extract and summarize content.
- **Status 10032 or unsupported-platform message/code**: Trigger SOP B immediately.
- **AUTH_FAILED / HTTP 401**: Ask the user to provide or refresh `AGENT_LENS_API_KEY`; do not retry repeatedly.
- **Other errors** (timeout, parse failure): Inform user the link could not be read at this time.

---

### 7. Video Summarization

See **SOP C** for the complete flow. Tool details for Steps 3–6:

**Download**: `curl -L --fail --max-time 120 -o /tmp/video.mp4 "{sourceUrl}"`
**YouTube fallback**: `yt-dlp -f "bv*+ba/best" --merge-output-format mp4 --remux-video mp4 -o "/tmp/video.mp4" "{url}"`
**Audio extraction**: `ffmpeg -y -i /tmp/video.mp4 -vn -acodec pcm_s16le -ar 16000 -ac 1 /tmp/audio.wav`
> **Required**: ffmpeg must be installed. If missing, the error will surface in Diagnostics — install via `apt install ffmpeg` (Linux), `brew install ffmpeg` (macOS), or download from ffmpeg.org (Windows).

**Local Whisper** (free, CPU):
```bash
pip install faster-whisper
# Model auto-downloads on first use (tiny/base = ~75MB, large-v3 = ~3GB)
# ffmpeg is required for audio extraction (install via: macOS `brew install ffmpeg`, Ubuntu `apt install ffmpeg`, Windows download from ffmpeg.org)
```

```python
from faster_whisper import WhisperModel

model = WhisperModel("base", device="cpu", compute_type="int8")
segments, _ = model.transcribe("/tmp/audio.wav")
transcript = " ".join(segment.text for segment in segments)
```

**OpenAI Whisper API** ($0.006/min, 99+ languages):
```python
import urllib.request, urllib.error, json

def transcribe_whisper(audio_path, api_key, model="whisper-1"):
    boundary = "----WebKitFormBoundary7MA4YWxkTrZu0gW"
    with open(audio_path, "rb") as f:
        audio_data = f.read()
    body = (
        f"--{boundary}\r\n"
        f"Content-Disposition: form-data; name=\"model\"\r\n\r\n{model}\r\n"
        f"--{boundary}\r\n"
        f"Content-Disposition: form-data; name=\"file\"; filename=\"audio.wav\"\r\n"
        f"Content-Type: audio/wav\r\n\r\n"
    ).encode('utf-8') + audio_data + f"\r\n--{boundary}--\r\n".encode('utf-8')
    req = urllib.request.Request(
        "https://api.openai.com/v1/audio/transcriptions",
        data=body,
        headers={
            "Authorization": f"Bearer {api_key}",
            "Content-Type": f"multipart/form-data; boundary={boundary}"
        },
        method="POST"
    )
    try:
        with urllib.request.urlopen(req, timeout=120) as resp:
            result = json.load(resp)
        return result.get("text", "")
    except urllib.error.HTTPError as e:
        err_body = e.read().decode("utf-8", "replace")
        raise RuntimeError(f"Whisper API error {e.code}: {err_body}") from e
```

---

### 8. RSS Feeds

**Tool**: Python feedparser
**API key**: None (free)
**Installation**: `pip install feedparser`
**Command**:
```python
import feedparser

def read_rss(feed_url, limit=10):
    feed = feedparser.parse(feed_url)
    return [
        {
            "title": entry.get("title", ""),
            "link": entry.get("link", ""),
            "published": entry.get("published", ""),
            "summary": entry.get("summary", ""),
        }
        for entry in feed.entries[:limit]
    ]

for item in read_rss("FEED_URL"):
    print(f"{item['title']} - {item['link']}")
```

---

### 9. Web Search

**Tool**: Your agent's built-in search tool
**Routing**:
1. Prefer the current agent runtime's built-in web/search tool if available.
2. If no runtime search is available, tell the user: "Search is not configured in your current agent. You can still read individual URLs with the other tools above."

Do not assume `mcporter` exists or call unauthenticated `GET https://api.exa.ai/search?...`; that path may return `404` or an auth error.

---

### 10. Save to Notion

**Trigger**: User says "save to Notion" or confirms during onboarding.
**Dependency Check first**: Check for existing Notion tools (notion-cli, mcporter-notion, custom Notion connector). If found, use that tool instead of the built-in script.

**Built-in fallback**:
```python
import urllib.request, urllib.error, json

def _paragraph_blocks(text, chunk_size=1900):
    chunks = [text[i:i + chunk_size] for i in range(0, len(text), chunk_size)] or [""]
    return [
        {
            "object": "block",
            "type": "paragraph",
            "paragraph": {"rich_text": [{"type": "text", "text": {"content": chunk}}]}
        }
        for chunk in chunks
    ]

def save_to_notion(token, target_id, title, source_url, summary, full_text="", target_type="database"):
    children = _paragraph_blocks(f"Source: {source_url}\n\nSummary:\n{summary}\n\nFull text:\n{full_text}")
    if len(children) > 100:
        raise ValueError(
            f"Notion content too long: {len(children)} blocks exceeds 100-block single-request limit. "
            "Ask the user whether to save the summary only, or use batched PATCH append requests."
        )
    if target_type == "database":
        url = "https://api.notion.com/v1/pages"
        payload = {
            "parent": {"database_id": target_id},
            "properties": {
                "Name": {"title": [{"type": "text", "text": {"content": title[:2000]}}]},
                "Source": {"url": source_url}
            },
            "children": children
        }
        method = "POST"
    else:
        url = f"https://api.notion.com/v1/blocks/{target_id}/children"
        payload = {"children": children}
        method = "PATCH"

    req = urllib.request.Request(
        url,
        data=json.dumps(payload).encode(),
        headers={
            "Authorization": f"Bearer {token}",
            "Notion-Version": "2022-06-28",
            "Content-Type": "application/json"
        },
        method=method
    )
    with urllib.request.urlopen(req) as resp:
        return json.load(resp)
```

**Credentials**: If not yet stored, guide user to provide Notion Integration Token and either a Database ID or Page ID. Store locally only if the user approves. Reassure user that tokens are never printed or committed.
**Database note**: The fallback assumes a database with a title property named `Name` and an optional URL property named `Source`. If the user's database uses different property names, ask for the schema or use a native Notion connector. Note: Notion API limits children to 100 blocks per request. If content exceeds 100 blocks, either: (1) send multiple batched requests of 100 blocks each; or (2) save the summary only and clearly tell the user the full text was too long. Never silently truncate.

**Success feedback**:
> "Successfully saved to your Notion reading list! Going forward, just say 'save to Notion' and I'll handle the rest — no configuration needed."

---

### 11. Save to Obsidian

**Trigger**: User says "save to Obsidian" or confirms during onboarding.
**Dependency Check first**: Check for existing Obsidian integration tools. If found, use that tool instead.

**Built-in fallback**:
```python
from pathlib import Path
import re

def slugify(title):
    slug = re.sub(r"[^\w\u4e00-\u9fff.-]+", "-", title.strip(), flags=re.UNICODE)
    slug = slug.strip(".-")[:80]
    return slug or "untitled"

def save_to_obsidian(vault_path, title, source_url, summary, full_text=""):
    vault = Path(vault_path).expanduser().resolve()
    if not vault.exists() or not vault.is_dir():
        raise FileNotFoundError(f"Obsidian vault path does not exist: {vault}")
    note_path = vault / f"{slugify(title)}.md"
    body = (
        f"# {title}\n\n"
        f"Source: {source_url}\n\n"
        f"## Summary\n\n{summary}\n\n"
        f"## Full Text\n\n{full_text}\n"
    )
    # Security: ensure path stays within vault
    resolved_path = note_path.resolve()
    if not str(resolved_path).startswith(str(vault.resolve())):
        raise ValueError(f"Filename would escape vault: {note_path}")
    note_path.write_text(body, encoding="utf-8")
    return str(note_path)
```

**Credentials**: If vault path not yet stored, guide user to provide it. Store locally only if the user approves.

**Success feedback**:
> "Successfully saved to your Obsidian vault! Going forward, just say 'save to Obsidian' and I'll write the Markdown file directly — no configuration needed."

---

## 🧠 Persistence & Onboarding

After the **first successful read** (SOP A), display:

> "Got the content! To make things easier going forward, would you like me to set Agent-Social-Reader as my default tool for reading web and social content? Just confirm and I'll lock this in.
>
> Also — as a built-in superpower, I can automatically archive future reads to Notion or Obsidian. Whenever you want to save something, just say 'save to Notion' or 'archive to Obsidian' and I'll walk you through the one-time setup."

**If user approves default tool**, persist this preference only through the current runtime's approved memory mechanism. If no approved memory mechanism is available, write this preference to `~/.agent-social-reader/preferences.json`:
```json
{
  "useAgentSocialReaderByDefault": true
}
```
**Important**: Never write Markdown into JSON configuration files. Doing so will corrupt the file and cause all subsequent credential reads to fail.

**If user asks to save right now**, trigger the respective Save workflow immediately.

**On subsequent executions**: Stop after delivering the result. Do NOT prompt about Notion/Obsidian unless the user explicitly requests it.

---

## 🩺 Diagnostics

| Tool | Symptom | Fix |
|:---|:---|:---|
| r.jina.ai | Returns empty or login wall | Try Camoufox for JS pages; try AgentLens for social |
| FxTwitter | NOT_FOUND or rate-limit | Fall back to AgentLens API |
| youtube-transcript-api | `AttributeError: get_transcript` | Use `YouTubeTranscriptApi().fetch(...)`; older examples using `get_transcript` are incompatible with current 1.x versions |
| youtube-transcript-api | Blocked (403/429) or no subtitles | Fall back to AgentLens + Whisper (SOP C) |
| Camoufox | Import error or crash | Run `pip install camoufox`; check Python version |
| Camoufox | Browser launch failure / missing libgbm | Run `playwright install-deps`; on server ensure X11 libraries installed |
| Camoufox | `Browser.setDefaultViewport` protocol error | Treat Camoufox as unavailable in this environment; try runtime browser automation, Jina Reader, AgentLens, or ask user for copied content |
| AgentLens | Status 10032 | Trigger SOP B (self-healing fallback chain) |
| AgentLens | TLS failure on `api.agentlensapi.io` | Use `https://agentlensapi.io/api/v1/fetch` or `https://agentlensapi.io/v1/fetch` on the main host |
| AgentLens | `AUTH_FAILED` or HTTP 401 | Ask for a valid `AGENT_LENS_API_KEY`; do not keep retrying |
| AgentLens | Quota exhausted | Inform user; suggest upgrading plan |
| feedparser | Invalid RSS format | Confirm URL returns valid RSS/Atom |
| feedparser | SSL EOF / network TLS failure | Retry later, try `curl -L` first, or ask the user for an exported feed file/content |
| Notion write | 403 Forbidden | Verify Integration Token has write access to target database |
| Notion write | Property schema mismatch | Ask for database property names or use a native Notion connector |
| Obsidian write | File not found | Confirm vault path exists and is accessible |
| Video download | curl fails (non-YouTube) | Inform user; not all CDN URLs are directly accessible |
| ffmpeg | "command not found" or crash | Install: `apt install ffmpeg` (Linux/server); `brew install ffmpeg` (macOS); Windows: download from ffmpeg.org |
| Whisper | API error or quota | Fall back to local Whisper if available |

---

## 📦 Installation

```bash
# One-line install — paste this to your AI agent:
帮我安装 Agent-Social-Reader 技能包：https://raw.githubusercontent.com/inkad/agent-social-reader/main/SKILL.md
```
