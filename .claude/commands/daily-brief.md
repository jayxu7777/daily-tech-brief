---
description: Fetch top tech news + AI projects + papers + funding news and push the digest to Telegram
---

You are a daily tech-news curator. Your job: fetch today's top items across four categories, summarize them in Chinese, and push the result to Telegram.

## Step 1 — Load credentials

Read Telegram credentials from `.env` in the project root. Required variables:

- `TELEGRAM_BOT_TOKEN`
- `TELEGRAM_CHAT_ID`

If `.env` is missing or either variable is empty, stop and tell the user to copy `.env.example` to `.env` and fill in the values.

## Step 2 — Fetch sources in parallel

Use a single message with parallel tool calls to fetch all of these:

1. **AI projects** — `WebFetch` https://github.com/trending?since=daily — list top 10 trending repos with description, language, stars today, total stars. Look especially for AI / agent / LLM-related projects.
2. **AI papers** — `WebFetch` https://huggingface.co/papers — list today's top 10 with title, authors/org, upvote count, link.
3. **Tech headlines** — `WebFetch` https://news.ycombinator.com/ — list top 10 stories with points and URLs.
4. **Tech funding (US/global)** — `WebSearch` query: `tech startup funding round announcement {{current_month_year}}` — extract the most notable rounds.
5. **Crypto / Web3** — `WebSearch` query: `crypto Bitcoin Ethereum news today {{today_date}}` — extract today's market-moving items.

Replace `{{current_month_year}}` and `{{today_date}}` with the actual current values.

## Step 3 — Pick Top 3 per category

For each of the four categories, pick the **3 most signal-rich** items. Selection rubric:

- **AI projects**: prefer agent/LLM/AI-infra repos with high "stars today". Skip generic infra (curl, build-your-own-x), skip security tools, skip purely educational compilations unless directly AI-related.
- **AI papers**: prefer papers with strong upvotes from reputable orgs (DeepMind, Meta FAIR, OpenAI, Anthropic, top universities, well-known labs). Diversify topics if possible.
- **Tech headlines**: prefer items that signal a real industry shift (model releases, eval changes, big-co decisions) over personal blog posts unless very high-scoring.
- **Funding/Crypto**: prefer named-entity rounds with $ amounts, or market-structural news (ETF flows, regulatory changes).

## Step 4 — Format the digest

Build a plain-text Telegram message in this exact structure (no Markdown parse mode — Telegram will auto-link URLs):

```
📰 每日简报 · YYYY-MM-DD（周X）

━━━━━━━━━━━━━━━━━━━
🤖 AI 项目 Top 3（GitHub 今日热门）
━━━━━━━━━━━━━━━━━━━

1️⃣ owner/repo  ⭐ +N today / Mk total
<English description from repo>
<1-2 sentence Chinese take: what it is + why it matters>
<URL>

2️⃣ ...
3️⃣ ...

━━━━━━━━━━━━━━━━━━━
📄 AI 论文 Top 3（HuggingFace Daily）
━━━━━━━━━━━━━━━━━━━

1️⃣ <English paper title>  👍 N
<Org / authors>
<1-2 sentence Chinese summary: contribution + why it matters>
<URL>

...

━━━━━━━━━━━━━━━━━━━
🔥 科技热点 Top 3（Hacker News）
━━━━━━━━━━━━━━━━━━━

1️⃣ <English title>  🔥 N points
<1-2 sentence Chinese take>
<URL>

...

━━━━━━━━━━━━━━━━━━━
💰 投资新闻 Top 3（美股科技 + Crypto）
━━━━━━━━━━━━━━━━━━━

1️⃣ <Headline in Chinese with key entities/$ in it>
<1-2 sentence Chinese context>
<URL>

...

━━━━━━━━━━━━━━━━━━━
✨ 一句话总结
━━━━━━━━━━━━━━━━━━━
<one-line summary tying together today's signal>
```

Style rules:
- Keep total length under 4000 chars (Telegram limit is 4096).
- English titles/repo names verbatim. All commentary in Chinese.
- No filler ("今天我们来看看…"). Get to the point.
- Don't fabricate. If a category has fewer than 3 strong items, say so.

## Step 5 — Push to Telegram

Write the digest to `/tmp/daily-brief-$(date +%Y-%m-%d).txt`, then send via `curl`:

```bash
curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
  --data-urlencode "chat_id=${TELEGRAM_CHAT_ID}" \
  --data-urlencode "disable_web_page_preview=true" \
  --data-urlencode "text@/tmp/daily-brief-$(date +%Y-%m-%d).txt"
```

Source the `.env` file first so the variables are available: `set -a; source .env; set +a`.

If the API returns `"ok":true`, report success to the user with the date and item counts. If it returns an error, show the error message.
