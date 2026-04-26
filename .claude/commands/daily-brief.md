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

## Step 5 — Push the main digest to Telegram

Write the digest to `/tmp/daily-brief-$(date +%Y-%m-%d).txt`, then send via `curl`:

```bash
curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
  --data-urlencode "chat_id=${TELEGRAM_CHAT_ID}" \
  --data-urlencode "disable_web_page_preview=true" \
  --data-urlencode "text@/tmp/daily-brief-$(date +%Y-%m-%d).txt"
```

Source the `.env` file first so the variables are available: `set -a; source .env; set +a`.

If the API returns `"ok":true`, continue to Step 6. Otherwise show the error and stop.

## Step 6 — Crypto/X Top 10 (separate message)

After the main digest is delivered, fetch the **Top 10 hottest crypto / blockchain posts circulating on X today** and push them as a second Telegram message.

> **Why a second message**: Telegram's 4096-char limit per message — combined with 10 items each having English original + Chinese translation + commentary, the crypto block needs its own message.

### Sources (no X API needed)

X.com blocks unauthenticated WebFetch and the API is paid. Most aggregators (LunarCrush / CryptoPanic / CoinDesk / The Block) also return 403 to unauthenticated requests as of 2026. The reliable approach is:

**Primary (must work)**:
1. **WebFetch** https://crypto.news/ — front page surfaces KOL takes (Saylor, Novogratz, Garlinghouse, etc.), institutional moves, regulatory news, on-chain events. Stable, no auth required.
2. **WebSearch** query: `crypto twitter trending today {{today_date}}` — pulls X-referenced commentary indexed by Google.

**Optional (try, skip silently on 403/error — do NOT retry)**:
3. WebFetch https://lunarcrush.com/categories/cryptocurrencies/social
4. WebFetch https://cryptopanic.com/?filter=hot
5. WebFetch https://www.theblock.co/latest

If primary sources succeed, you have enough material. Don't block the whole pipeline waiting on optional sources.

Pick the **10 highest-signal items**. Selection rubric:
- Prefer named KOLs / projects / institutions (Saylor, Vitalik, Novogratz, BlackRock, JPMorgan, Aave, Solana Foundation, etc.) over anonymous chatter.
- Drop pure price-shilling / "to the moon" content.
- Diversify topics: BTC macro, ETH/L2, Solana/alt-L1, DeFi, RWA/tokenization, regulation, security/exploits, on-chain analytics.
- If the day's news is light, 8–10 items is fine — don't pad to 10 with weak content.

### Format (second message)

```
🪙 Crypto on X · Top 10 · YYYY-MM-DD
━━━━━━━━━━━━━━━━━━━

1️⃣ <Author handle / source>
EN: <original English headline or tweet quote>
中: <Chinese translation, faithful, no editorial tone>
💡 <1 short Chinese sentence — why it matters>
🔗 <link>

2️⃣ ...
...
🔟 ...

━━━━━━━━━━━━━━━━━━━
📊 今日加密一句话
━━━━━━━━━━━━━━━━━━━
<one-line Chinese summary of the dominant crypto narrative today>
```

Style rules for this message:
- Original headline must be in **English**. If the source is in another language, translate to English first, then to Chinese.
- Chinese translation should be literal, not summarized. The 💡 line is where commentary goes.
- Keep this message under 4000 chars total. If items run long, trim the 💡 commentary first.
- If LunarCrush / CryptoPanic / X were inaccessible (likely), append a short footer noting that today's content was sourced from crypto.news + WebSearch — transparency over fake completeness.

### Push it

```bash
curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
  --data-urlencode "chat_id=${TELEGRAM_CHAT_ID}" \
  --data-urlencode "disable_web_page_preview=true" \
  --data-urlencode "text@/tmp/crypto-x-$(date +%Y-%m-%d).txt"
```

If the API returns `"ok":true` for both messages, report success with date + counts (e.g. "✓ main digest 12 items + crypto X 10 items pushed to Telegram"). Otherwise show whichever step failed.
