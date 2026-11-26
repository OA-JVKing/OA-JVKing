# Hi, I'm Jay 👋

I work on two separate initiatives:

- **🤖 AI and workflow automation consulting:** AI chatbots, ordering bots and n8n/Make workflows that take routine work off small businesses.
- **📈 Trading automation:** the Option Ark options screener, rule-based trading bots and a multi-broker portfolio dashboard.

Most of my code lives in private repositories. This page explains what each project does and how it works.

---

## 🤖 AI and workflow automation consulting

Through **Option Ark Consulting** ([consulting.optionark.com](https://consulting.optionark.com)) I help small businesses automate the busywork using n8n, Make and AI agents. The consulting site itself is a lightweight static site deployed on Render with automatic preview builds for every pull request.

### RAG knowledge chatbot (live on consulting.optionark.com)
A retrieval-augmented chatbot that answers visitor questions from a curated knowledge base instead of guessing. Content is managed in **Airtable**, embedded and stored in a **Qdrant** vector database, and an **n8n** workflow ties it together: it retrieves the most relevant passages for each question and has **Claude** write a grounded answer. Updating the bot is as simple as editing a row in Airtable.

```mermaid
flowchart LR
    subgraph Ingest["Knowledge ingestion"]
        AT["Airtable<br/>knowledge base"] --> EMB["n8n: chunk & embed"]
        EMB --> QD[("Qdrant<br/>vector store")]
    end

    subgraph Answer["Answering a question"]
        U["Visitor on<br/>consulting.optionark.com"] --> CW["Chat widget"]
        CW --> N8N["n8n workflow"]
        N8N -- "semantic search" --> QD
        QD -- "top matching passages" --> N8N
        N8N -- "question + context" --> CL["Claude"]
        CL -- "grounded answer" --> N8N
        N8N --> CW
    end
```

### Text and WhatsApp ordering for restaurants
A repeatable package for local restaurants. Customers text or WhatsApp the shop to ask about specials, check what's in stock, or place an order. An AI assistant answers in plain language, builds the order, confirms it with the customer, and sends it straight into the restaurant's point-of-sale system, so staff never have to retype it.

```mermaid
sequenceDiagram
    actor C as Customer
    participant M as SMS / WhatsApp
    participant W as Workflow (n8n / Make)
    participant AI as AI assistant
    participant POS as Restaurant POS API

    C->>M: "Do you have a large veggie tonight?"
    M->>W: Incoming message
    W->>POS: Look up menu, specials, stock
    POS-->>W: Live menu data
    W->>AI: Message + menu context
    AI-->>W: Answer and draft order
    W->>M: Reply with order summary
    M->>C: "Large veggie, $18. Confirm?"
    C->>M: "Yes"
    M->>W: Confirmation
    W->>POS: Create order
    POS-->>W: Order number & pickup time
    W->>M: "Order 142, ready in 25 min"
```

### Small-business workflow automation
Custom n8n and Make workflows that connect a business's existing apps (email, forms, spreadsheets, CRMs, messaging) and add an AI step where judgment is needed, such as sorting requests, drafting replies or pulling data out of documents.

```mermaid
flowchart LR
    IN["Trigger<br/>form · email · message · schedule"] --> FLOW["n8n / Make workflow"]
    FLOW --> AI["AI step<br/>classify · extract · draft"]
    AI --> ACT["Actions<br/>update CRM · send reply · create task"]
    ACT --> NOTIFY["Notify the owner"]
```

### Ohio vanity plate finder (Android proof of concept)
An unpublished Android app that makes picking a vanity plate feel like a dating app. The user enters a few interests (say, "outdoors" or "coffee lover"), and **Claude** generates plate ideas to match. Every idea is filtered for profanity and checked against Ohio BMV rules for length and allowed characters. The user swipes right on the ones they like and left on the rest.

Each right-swipe can then be checked against the Ohio BMV through a custom API, hosted on **Vercel**, that reports **available** or **taken**. If a plate is available, the user can go register it. The planned paid tier, designed but not built, would watch taken plates every day and alert the user when one frees up.

```mermaid
flowchart TD
    U["User enters interests<br/>e.g. outdoors, coffee lover"] --> APP["Android app"]
    APP -- "interests" --> CL["Claude API"]
    CL -- "plate ideas" --> FIL{"Profanity filter &<br/>Ohio BMV rules<br/>length · characters"}
    FIL -- "valid ideas" --> SW["Swipe deck<br/>👈 no · yes 👉"]
    SW -- "right swipes" --> SAVE[("Shortlist")]
    SAVE -- "check one at a time" --> API["Custom plate API<br/>(Vercel)"]
    API --> BMV[("Ohio BMV<br/>plate lookup")]
    BMV --> RES{"Result"}
    RES -- "available" --> REG["User registers the plate"]
    RES -- "taken" --> WATCH["Daily watch & alert<br/>planned paid tier"]

    style WATCH stroke-dasharray: 5 5
```

---

## 📈 Trading automation

### Option Ark: options income screener and education platform
[optionark.com](https://optionark.com) scans S&P 500 and Nasdaq stocks every hour during market hours for high-quality **covered call, cash-secured put, put credit spread and call credit spread** setups. It ranks them and explains them in plain language, so traders find good income setups before they go looking.

**How a setup gets picked**
- **Delta-targeted, short-dated entries** with a liquidity floor so every setup is tradeable
- **Minimum return thresholds** for each strategy, ranked by return on capital rather than raw premium
- **Earnings-aware:** setups too close to a report are held back automatically, and any setup that expires after an earnings date gets a ⚠ flag
- **Risk tiers:** Safe, Balanced and Growth, based on the quality of the underlying company

**Options-specific data on every setup**

| Metric | What it tells you |
| --- | --- |
| **IV Rank** | Where implied volatility sits against its own past year |
| **POP** | Estimated probability of profit |
| **Expected move** | The 1-standard-deviation price range by expiration |
| **Win-rate history** | Historic win/loss per ticker and strategy, plus a track record by strategy and delta |
| **ROC & annualized return** | Premium collected against capital at risk |
| **Delta, DTE, OI, RSI** | Odds, time, liquidity and momentum at a glance |

**Platform features**
- **Strategy Screener:** live filters by strategy, ticker, risk tier, delta, DTE, minimum ROC and RSI range, with sorting, side-by-side compare and CSV export
- **Most Active options** ranked by contract volume
- **Earnings calendar** with report dates, before-open/after-close timing and EPS estimates (data from Finnhub), plus a count of affected setups
- **Watchlists, saved screeners and daily email digests** that match a user's saved filters
- **Free and Plus tiers**, with a Pro tier coming soon: position tracker with profit-target and assignment-risk alerts, trade journal, income dashboard and deep-dive play pages

**Education**
A built-in learning section for newer traders. It starts with options basics (puts, calls, premium, strike, assignment). Then it gives each of the four strategies its own lesson with a worked example, a profit/loss diagram, max profit, max loss, break-even, capital needed, when it works best and what can go wrong. It ends with a side-by-side strategy comparison and a glossary of key terms like delta, IV Rank, ROC and POP.

```mermaid
flowchart LR
    DATA[("Market, options &<br/>earnings data")] --> ENGINE["Option Ark<br/>scan & ranking engine"]
    ENGINE --> TOP["Top Plays"]
    ENGINE --> SCR["Screener Plus"]
    ENGINE --> ALERT["Watchlists &<br/>email alerts"]
    ENGINE --> BOTS["Trading bots"]
    EDU["Education"] -.-> TOP
```

### Trading bots
Python bots that turn screener signals into real orders. Each bot runs a rule-based strategy, sizes positions, places and manages orders (including multi-leg options orders), and logs every decision. They connect directly to brokerage APIs at **Tradier** and **Charles Schwab**.

```mermaid
flowchart LR
    DATA[("Market &<br/>options data")] --> SCR["Option Ark screener"]
    SCR -- "candidates" --> STRAT["Strategy rules<br/>entry · exit · sizing"]
    STRAT --> RISK{"Risk checks"}
    RISK -- "pass" --> EXEC["Order manager"]
    RISK -- "fail" --> SKIP["Skip & log"]
    EXEC --> TRAD["Tradier API"]
    EXEC --> SCHW["Charles Schwab API"]
    TRAD --> LOG[("Trade log")]
    SCHW --> LOG
    LOG -- "fills & P/L" --> STRAT
```

### Multi-broker portfolio dashboard
One view of every account I hold, across brokers. The dashboard uses **SnapTrade** to securely link brokerage accounts and pull balances, positions and activity from **Robinhood, Charles Schwab, Tradier and Webull**, then rolls them up into total value, allocation and performance.

```mermaid
flowchart LR
    RH["Robinhood"] --> ST
    SC["Charles Schwab"] --> ST
    TR["Tradier"] --> ST
    WB["Webull"] --> ST
    ST["SnapTrade<br/>account linking API"] --> SYNC["Sync service"]
    SYNC --> DB[("Positions, balances<br/>& activity")]
    DB --> UI["Portfolio dashboard<br/>value · allocation · P/L"]
```

---

## 🧰 Tools and tech

| Area | What I use |
| --- | --- |
| Languages | Python, HTML/CSS, JavaScript |
| Brokerage APIs | Tradier, Charles Schwab, SnapTrade (Robinhood, Schwab, Tradier, Webull) |
| Automation | n8n, Make, AI agents |
| AI | Claude, RAG pipelines, Qdrant vector database |
| Data | Airtable |
| Messaging | SMS, WhatsApp |
| Hosting & DevOps | Render, Vercel, GitHub |

---

## 📫 Get in touch

- Email: [consulting@optionark.com](mailto:consulting@optionark.com)
- Consulting: [consulting.optionark.com](https://consulting.optionark.com)
- Trading tools: [optionark.com](https://optionark.com)

Happy to walk through any of these projects on a call.
