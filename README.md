# Daily Tech Brief

**English** · **[简体中文](README.zh-CN.md)**

Let your local [Claude Code](https://claude.com/claude-code) push a daily digest straight to your Telegram every morning at 8am:

- **Top 3** AI projects (GitHub trending)
- **Top 3** AI papers (HuggingFace daily)
- **Top 3** tech headlines (Hacker News)
- **Top 3** funding / market news (US tech VC + crypto)
- **Top 10** hottest crypto/blockchain posts on X — English original + Chinese translation (sent as a second message)

No backend. No server. Everything runs inside your local Claude Code.

## How it works

```
Claude Code (cron trigger)
  └─ runs the /daily-brief slash command
       ├─ WebFetch  github.com/trending
       ├─ WebFetch  huggingface.co/papers
       ├─ WebFetch  news.ycombinator.com
       ├─ WebSearch tech funding / crypto
       ├─ WebFetch  lunarcrush.com + cryptopanic.com (X-on-crypto)
       ├─ Claude picks Top items + writes Chinese commentary / translation
       └─ curl → Telegram Bot API (2 messages: main digest + crypto X feed)
```

The whole thing is one prompt — [`.claude/commands/daily-brief.md`](.claude/commands/daily-brief.md). Claude reads it, fetches sources, ranks items, summarizes, and pushes. No code to maintain.

## Quick start

### 1. Create a Telegram bot

1. In Telegram, open [@BotFather](https://t.me/BotFather), send `/newbot`, follow the prompts.
2. Save the HTTP API token it returns (looks like `123456789:AAExxxx...`).
3. Search for your new bot in Telegram and send it any message (e.g. `hi`). **This step is required** — bots can only message users who have contacted them first.
4. Open `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates` in your browser. The number under `"chat":{"id":XXX}` is your `chat_id`.

### 2. Clone & configure

```bash
git clone https://github.com/jayxu7777/daily-tech-brief.git
cd daily-tech-brief
cp .env.example .env
# Edit .env and fill in TELEGRAM_BOT_TOKEN and TELEGRAM_CHAT_ID
```

### 3. Run it once

Open Claude Code in the project directory and run:

```
/daily-brief
```

Within a minute or so your Telegram should receive today's digest.

### 4. Schedule it daily

Inside Claude Code:

```
/schedule
```

Create a cron trigger:

- Cron expression: `0 8 * * *` (every day at 08:00 local time)
- Prompt: `/daily-brief`

Done. You'll get a digest every morning.

## Customize

Edit [`.claude/commands/daily-brief.md`](.claude/commands/daily-brief.md):

- **Add or change sources?** Edit the fetch list in Step 2.
- **Change the Top-3 selection rubric?** Edit Step 3.
- **Change the output format / language?** Edit the template in Step 4.
- **Push to Slack / Lark / email instead?** Replace the `curl` call in Step 5.

It's a prompt, not code — you don't need to know any programming language to modify it.

## Sample output

A real push from 2026-04-26 (excerpt):

```
📰 每日简报 · 2026-04-26（周日）

━━━━━━━━━━━━━━━━━━━
🤖 AI 项目 Top 3（GitHub 今日热门）
━━━━━━━━━━━━━━━━━━━

1️⃣ mattpocock/skills  ⭐ +2,507 / 23.3k
Agent Skills for real engineers
作者把自己 .claude 目录里的 skills 集合直接开源,一夜爆火。
说明社区已经从"玩 prompt"进化到"沉淀可复用 agent 能力"。
https://github.com/mattpocock/skills

2️⃣ abhigyanpatwari/GitNexus  ⭐ +667 / 30.1k
Client-side Graph RAG Agent for code exploration
纯前端跑的代码知识图谱 + Graph RAG。

3️⃣ trycua/cua  ⭐ +200 / 14.3k
Infrastructure for Computer-Use Agents

━━━━━━━━━━━━━━━━━━━
📄 AI 论文 Top 3（HuggingFace Daily）
━━━━━━━━━━━━━━━━━━━

1️⃣ Vista4D: Video Reshooting with 4D Point Clouds  👍 118
Eyeline Labs

2️⃣ LLaTiSA: Difficulty-Stratified Time Series Reasoning  👍 81

3️⃣ WorldMark: Benchmark for Interactive Video World Models  👍 34

━━━━━━━━━━━━━━━━━━━
🔥 科技热点 Top 3（Hacker News）
━━━━━━━━━━━━━━━━━━━

1️⃣ OpenAI: Why we no longer evaluate SWE-bench Verified  🔥 221
2️⃣ AI should elevate your thinking, not replace it  🔥 180
3️⃣ Fast16: 高精度软件破坏,比 Stuxnet 早 5 年  🔥 87

━━━━━━━━━━━━━━━━━━━
💰 投资新闻 Top 3（美股科技 + Crypto）
━━━━━━━━━━━━━━━━━━━

1️⃣ Bezos 的 Project Prometheus 接近完成 $10B 融资 @ $38B 估值
2️⃣ Q1 2026 全球 VC 投资创纪录:$300B / 6,000 家公司
3️⃣ Bitcoin 站稳 $77k,4 月有望成近一年最强月

━━━━━━━━━━━━━━━━━━━
✨ 一句话总结
━━━━━━━━━━━━━━━━━━━
今天主线:Agent 基础设施在 GitHub 集中爆发;
资本继续重仓 embodied AI;Crypto 流动性回归。
```

The default output is in Chinese (with English titles preserved). Want all-English output? Edit Step 4 of the slash command.

## Security notes

- `.env` is in `.gitignore` — your token will not be committed.
- If you ever leak a token (paste in chat, push by accident), go to [@BotFather](https://t.me/BotFather), send `/revoke`, and rotate it.
- Telegram bots can only message users who have contacted them first, so even a leaked token can't be used to spam strangers.

## License

[MIT](LICENSE)
