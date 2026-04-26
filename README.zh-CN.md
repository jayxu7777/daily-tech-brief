# Daily Tech Brief

**[English](README.md)** · **简体中文**

每天早上 8 点，让 [Claude Code](https://claude.com/claude-code) 自动把 **AI 项目 / AI 论文 / 科技热点 / 投资新闻** 各 Top 3 推送到你的 Telegram。

无需后端，无需服务器。一切跑在你本地的 Claude Code 里。

## 它是怎么工作的

```
Claude Code (cron trigger)
  └─ 执行 /daily-brief 斜杠命令
       ├─ WebFetch  github.com/trending
       ├─ WebFetch  huggingface.co/papers
       ├─ WebFetch  news.ycombinator.com
       ├─ WebSearch 科技投融资 / 加密货币
       ├─ Claude 自己挑 Top 3 + 写中文点评
       └─ curl → Telegram Bot API
```

核心是一个写好的 prompt（[`.claude/commands/daily-brief.md`](.claude/commands/daily-brief.md)），Claude 看完会自己抓取、筛选、总结、推送。

## 快速开始

### 1. 准备 Telegram bot

1. Telegram 里找 [@BotFather](https://t.me/BotFather)，发 `/newbot`，按提示起名字
2. BotFather 会回一段 token，形如 `123456789:AAExxxx...` —— **保存好**
3. 在 Telegram 搜你刚创建的 bot，给它发一条 `hi`（必须先主动联系，bot 才能回信）
4. 浏览器访问 `https://api.telegram.org/bot<你的TOKEN>/getUpdates`，里面 `"chat":{"id":XXX}` 的数字就是 chat_id

### 2. Clone & 配置

```bash
git clone https://github.com/jayxu7777/daily-tech-brief.git
cd daily-tech-brief
cp .env.example .env
# 编辑 .env，填入 TELEGRAM_BOT_TOKEN 和 TELEGRAM_CHAT_ID
```

### 3. 测试一次

在项目目录下打开 Claude Code，输入：

```
/daily-brief
```

几十秒后你的 Telegram 应该会收到一份当天简报。

### 4. 设置每天自动跑

在 Claude Code 里：

```
/schedule
```

按提示创建一个 cron trigger：

- 表达式：`0 8 * * *`（每天本地时间 8:00）
- Prompt：`/daily-brief`

完事。

## 自定义

直接编辑 [`.claude/commands/daily-brief.md`](.claude/commands/daily-brief.md)：

- 想加 / 改信息源？修改 Step 2 的 fetch 列表
- 想换 Top 3 的筛选逻辑？修改 Step 3 的 rubric
- 想换输出格式 / 语言？修改 Step 4 的模板
- 想推到 Slack / 飞书 / 邮件？把 Step 5 的 curl 换掉

因为是 prompt 而不是代码，所以改起来不用懂任何编程。

## 示例输出

下面是 2026-04-26 的一份真实推送（节选）：

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
纯前端跑的代码知识图谱 + Graph RAG,让 agent 能"看懂"整个 repo 结构。
https://github.com/abhigyanpatwari/GitNexus

3️⃣ trycua/cua  ⭐ +200 / 14.3k
Infrastructure for Computer-Use Agents
专为 Computer-Use agent 做的 sandbox + 评测基准。

━━━━━━━━━━━━━━━━━━━
📄 AI 论文 Top 3（HuggingFace Daily）
━━━━━━━━━━━━━━━━━━━

1️⃣ Vista4D: Video Reshooting with 4D Point Clouds  👍 118
Eyeline Labs
用 4D 点云做"重拍视频"——给一段视频,可以换机位、改运镜重新生成。
视频生成从"造内容"走向"可控编辑"。

2️⃣ LLaTiSA: Difficulty-Stratified Time Series Reasoning  👍 81
让 LLM 真正"看懂"时间序列。金融/运维/IoT 场景的 LLM 落地基础工作。

3️⃣ WorldMark: Benchmark for Interactive Video World Models  👍 34
首个针对"交互式视频世界模型"的统一评测套件。

━━━━━━━━━━━━━━━━━━━
🔥 科技热点 Top 3（Hacker News）
━━━━━━━━━━━━━━━━━━━

1️⃣ OpenAI: Why we no longer evaluate SWE-bench Verified  🔥 221
OpenAI 官宣不再用 SWE-bench Verified 评测前沿编码能力。
AI coding 评测体系要换代了。

2️⃣ AI should elevate your thinking, not replace it  🔥 180
关于 AI 协作哲学的反思:把 AI 当外脑而非替身。

3️⃣ Fast16: 比 Stuxnet 早 5 年的国家级网络武器  🔥 87

━━━━━━━━━━━━━━━━━━━
💰 投资新闻 Top 3（美股科技 + Crypto）
━━━━━━━━━━━━━━━━━━━

1️⃣ Bezos 的 Project Prometheus 接近完成 $10B 融资 @ $38B 估值
贝索斯亲自下场的 AI 实验室,做"理解物理世界"的模型。
资本继续向 embodied AI 集中。

2️⃣ Q1 2026 全球 VC 投资创纪录:$300B / 6,000 家公司
AI 是绝对主力。

3️⃣ Bitcoin 站稳 $77k,4 月有望成近一年最强月
USDT 供应量逼近 $1500 亿;IBIT 期权超过 Deribit。

━━━━━━━━━━━━━━━━━━━
✨ 一句话总结
━━━━━━━━━━━━━━━━━━━
今天主线:Agent 基础设施在 GitHub 集中爆发,AI coding 评测进入下一代;
资本继续重仓 embodied AI;Crypto 流动性回归。
```

## 安全提示

- `.env` 在 `.gitignore` 里，不会被提交
- 如果你把 token 不小心暴露过（粘到聊天记录、推 commit），去 [@BotFather](https://t.me/BotFather) 发 `/revoke` 重新生成
- bot 只能给「主动联系过它的用户」发消息，所以即使 token 泄露，攻击者也没法用它给陌生人发垃圾

## License

[MIT](LICENSE)
