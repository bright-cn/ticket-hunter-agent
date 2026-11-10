<div align="center">

<img src="public/brightdata-dark.svg" height="40" alt="Bright Data" />
&nbsp;&nbsp;&nbsp;×&nbsp;&nbsp;&nbsp;
<img src="public/yutori.png" height="28" alt="Yutori N1" />

<br /><br />

# 票务猎手智能体

**一个自主 AI 智能体，可实时浏览网页，为你查找当前最优票务信息。**

[![使用 Vercel 部署](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/bright-cn/ticket-hunter-agent)


</div>

---

## 这是什么？

票务猎手是一个**完全自主的 AI 智能体**，它可以接收类似 `"Taylor Swift Eras Tour NYC"` 的自然语言查询，并在无需任何人工干预的情况下搜索 Google，打开 Ticketmaster、StubHub、SeatGeek 及其他票务平台，浏览页面，并实时返回经过排序的结构化票务列表。

该智能体会将工作过程实时传输到浏览器：**你可以实时观看它浏览、点击和提取数据。**

**基于 [Bright Data](https://www.bright.cn/) + [Yutori N1](https://yutori.com) 构建**

---

## 工作原理

该智能体以**包含 5 个阶段的 LangGraph 流水线**运行，并通过服务器发送事件将过程实时传输到你的浏览器：

```
User Query
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│  Stage 1 · SERP Search                                  │
│  Bright Data SERP API searches Google for ticket sites  │
│  Scores and selects the top 3 most relevant URLs        │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Stage 2 · Browser Pipeline                             │
│  Connects to Bright Data Browser API via CDP            │
│  Opens each URL with a real Chromium instance           │
│  Takes initial screenshot → streamed to UI live         │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Stage 3 · Yutori N1 Agentic Loop (up to 15 steps)     │
│  N1 vision model views screenshots and decides actions  │
│  Tools: navigate · click · scroll · type · screenshot   │
│  Iterates until tickets are found or step limit reached │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Stage 4 · Structured Extraction                        │
│  OpenRouter (Gemini Flash) extracts JSON from findings  │
│  Fields: event, date, venue, section, row, price, url   │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Stage 5 · Merge & Rank                                 │
│  Deduplicates results across platforms                  │
│  Sorts by price (lowest first) → returned to UI         │
└─────────────────────────────────────────────────────────┘
```

---

## 技术栈

| 层级 | 技术 |
|---|---|
| **框架** | [Next.js 16](https://nextjs.org) · React 19 · TypeScript 5 |
| **样式** | [Tailwind CSS 4](https://tailwindcss.com) |
| **智能体编排** | [LangGraph](https://langchain-ai.github.io/langgraphjs/) |
| **浏览器自动化** | [Playwright Core](https://playwright.dev) + Chrome DevTools Protocol (CDP) |
| **网页基础设施** | [Bright Data 浏览器 API](https://www.bright.cn/products/scraping-browser) + 搜索引擎 API |
| **视觉 LLM** | [Yutori N1](https://yutori.com)（多模态、支持工具调用） |
| **提取 LLM** | [OpenRouter](https://openrouter.ai) → Google Gemini Flash |
| **速率限制** | [Upstash Redis](https://upstash.com)（按 IP 限制，演示版每 24 小时可搜索 1 次） |
| **部署** | 通过 GitHub Actions 部署至 [Vercel](https://vercel.com) |

---

## 支持的票务平台

该智能体会根据 Google 搜索结果，自主检测并浏览以下任意平台：

- Ticketmaster
- StubHub
- SeatGeek
- Vivid Seats
- AXS
- TickPick
- Gametime
- Eventbrite
- Viagogo
- TicketNetwork
- Tickets.com

---

## 开始使用

### 前提条件

- Node.js 18+
- 已启用浏览器 API 和搜索引擎 API 的 [Bright Data](https://www.bright.cn/) 账户
- [Yutori N1](https://yutori.com) API 密钥
- [OpenRouter](https://openrouter.ai) API 密钥
- [Upstash](https://upstash.com) Redis 数据库（用于速率限制）

### 1. 克隆仓库

```bash
git clone https://github.com/bright-cn/ticket-hunter-agent.git
cd ticket-hunter-agent
```

### 2. 安装依赖项

```bash
npm install
```

### 3. 配置环境变量

在项目根目录中创建 `.env.local` 文件：

```env
# Yutori N1 — vision LLM for autonomous browsing
YUTORI_API_KEY=your_yutori_api_key

# Bright Data — Browser API CDP WebSocket endpoint
# Found in your Bright Data dashboard under Browser API > Access Parameters
BRD_CDP_URL=wss://brd-customer-<id>-zone-<zone>:<password>@brd.superproxy.io:9222

# Bright Data — API key for SERP and Web Unlocker
BRIGHTDATA_API_KEY=your_brightdata_api_key
BRIGHTDATA_SERP_ZONE=serp                   # optional, defaults to "serp"
BRIGHTDATA_WEB_UNLOCKER_ZONE=unblocker      # optional, defaults to "unblocker"

# OpenRouter — structured JSON extraction via Gemini Flash
OPENROUTER_API_KEY=your_openrouter_api_key
OPENROUTER_MODEL=google/gemini-flash-1.5    # optional

# Upstash Redis — rate limiting (remove or leave blank to disable)
KV_REST_API_URL=https://your-instance.upstash.io
KV_REST_API_TOKEN=your_upstash_token
```

### 4. 在本地运行

```bash
npm run dev
```

打开 [http://localhost:3000](http://localhost:3000) 并搜索任意活动。

---

## 项目结构

```
src/
├── app/
│   ├── api/search/route.ts          # POST endpoint — SSE stream orchestrator
│   ├── page.tsx                     # Main UI page
│   └── layout.tsx                   # Root layout + global metadata
├── components/
│   ├── SearchForm.tsx               # Query input with example prompts
│   ├── BrowserView.tsx              # Live browser screenshot panels (1–3 platforms)
│   ├── StatusLog.tsx                # Real-time agent activity log
│   └── ResultCard.tsx               # Ticket card with price, venue, seats
├── hooks/
│   └── useTicketSearch.ts           # SSE consumer + client state management
└── lib/
    ├── agent/
    │   ├── graph.ts                 # LangGraph workflow definition
    │   ├── state.ts                 # Agent state schema
    │   ├── stream-events.ts         # SSE event emitters
    │   ├── runtime-session.ts       # Browser session lifecycle
    │   ├── browser-utils.ts         # Playwright helpers
    │   ├── tools.ts                 # N1 tool schemas (click, scroll, etc.)
    │   └── nodes/
    │       ├── serp-search.ts       # Stage 1: Google SERP via Bright Data
    │       ├── browser-open.ts      # Stage 2: CDP browser connection
    │       ├── browser-pipeline.ts  # Stage 2: per-URL browsing orchestration
    │       ├── n1-browse-core.ts    # Stage 3: N1 agentic loop (15-step max)
    │       ├── n1-browse.ts         # Stage 3: N1 LangGraph node wrapper
    │       ├── extract.ts           # Stage 4: structured JSON extraction
    │       └── merge-and-rank.ts    # Stage 5: dedup + price sort
    ├── bright-data.ts               # Bright Data SDK wrapper
    ├── n1-client.ts                 # Yutori N1 OpenAI-compatible client
    ├── openrouter-client.ts         # OpenRouter extraction client
    ├── rate-limit.ts                # Upstash Redis rate limiter
    └── types.ts                     # Shared TypeScript interfaces
```

---

## 部署

该仓库包含一个用于自动部署至 Vercel 的 GitHub Actions 工作流。

### 部署至 Vercel（一键部署）

[![使用 Vercel 部署](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/bright-cn/ticket-hunter-agent)

然后在你的 Vercel 项目设置中添加所有环境变量。

### 通过 GitHub Actions 部署

1. 复刻该仓库
2. 将其连接到 Vercel 项目
3. 将以下密钥添加到你的 GitHub 仓库：
   - `VERCEL_TOKEN`
   - `VERCEL_ORG_ID`
   - `VERCEL_PROJECT_ID`
4. 将所有 `.env.local` 变量添加到 Vercel 的环境设置中
5. 推送到 `main`：工作流将自动部署

### 通过 CLI 手动部署

```bash
npm run build
npx vercel deploy --prod
```

---

## 票务结果架构

智能体返回的每条结果都遵循以下结构：

```typescript
interface TicketResult {
  eventName: string;    // "Taylor Swift | The Eras Tour"
  eventDate: string;    // "Friday, June 7, 2025"
  venue: string;        // "MetLife Stadium"
  city: string;         // "East Rutherford, NJ"
  ticketType: string;   // "Floor A · General Admission"
  section: string;      // "GA Floor"
  row: string;          // "N/A"
  seats: string;        // "2 available"
  quantity: number;     // 2
  price: string;        // "$380"
  currency: string;     // "USD"
  platform: string;     // "StubHub"
  url: string;          // Deep link to the listing
  notes: string;        // Any additional context
}
```

---

## 实时流式传输

该智能体通过**服务器发送事件 (SSE)** 与前端通信。搜索期间会发出以下事件：

| 事件 | 载荷 | 说明 |
|---|---|---|
| `status` | `{ message, level }` | 智能体日志条目（INFO、ACTION、WARN、DONE 等） |
| `screenshot` | `{ data, source }` | 实时流式传输的 Base64 浏览器截图 |
| `inspect_url` | `{ url, source }` | 浏览器当前访问的 URL |
| `result` | `{ tickets, finalAnswer }` | 最终的结构化票务结果 |
| `error` | `{ message }` | 智能体错误 |
| `done` | — | 流式传输完成 |

---

## 速率限制

在线演示通过 Upstash Redis 强制执行**每个 IP 每 24 小时仅可搜索 1 次**的限制。

在本地使用你自己的密钥运行时，除非你在 `.env.local` 中配置 Upstash，否则不会执行此限制。

---

## 技术支持

<table>
<tr>
<td width="50%" valign="top">

### Bright Data

[Bright Data](https://www.bright.cn/) 提供使该智能体得以运行的网页基础设施：

- **浏览器 API**：通过 CDP 连接的真实 Chromium 实例，支持住宅代理路由、反机器人绕过和 JavaScript 执行
- **搜索引擎 API**：提供可靠的 Google 搜索结果，用于发现正确的票务平台 URL

</td>
<td width="50%" valign="top">

### Yutori N1

[Yutori N1](https://yutori.com) 是智能体循环的核心多模态视觉 LLM：

- 接收浏览器截图作为视觉输入
- 决定要执行的操作（点击、滚动、导航、输入）
- 从看到的内容中提取票务信息
- 每个 URL 最多执行 15 个推理步骤

</td>
</tr>
</table>

---

## 许可证

MIT © [Bright Data](https://www.bright.cn/) & [Yutori](https://yutori.com)
