# Hi, I'm Jay 👋

I build automation for traders and small businesses. On the trading side that means options screeners, rule-based trading bots and a multi-broker portfolio dashboard. On the business side it means AI-powered workflows that connect the tools a company already uses, so routine work happens on its own.

Most of my code lives in private repositories. This page explains what each project does and how the pieces fit together.

---

## 🗺️ The big picture

```mermaid
flowchart LR
    subgraph Trading["📈 Trading"]
        OA["Option Ark<br/>screener & platform"]
        BOTS["Trading bots"]
        DASH["Portfolio dashboard"]
    end

    subgraph Consulting["🤖 AI & workflow consulting"]
        SITE["Consulting site"]
        RAG["RAG chatbot<br/>Claude · Qdrant · Airtable"]
        WF["n8n / Make workflows"]
        CHAT["Text & WhatsApp<br/>ordering bot"]
        PLATE["Vanity plate<br/>finder app"]
    end

    MKT[("Market data")] --> OA
    OA -- "signals & watchlists" --> BOTS
    BOTS -- "orders" --> BRK["Tradier · Schwab"]
    BRK --> DASH
    SNAP["SnapTrade"] --> DASH

    OA -. "proof of what<br/>automation can do" .-> SITE
    SITE -- "embedded" --> RAG
    SITE --> WF
    WF --> CHAT
    WF -. "AI prototypes" .-> PLATE
```

---

## 📈 Trading automation

### Option Ark: options screener and trading platform
A Python platform behind [optionark.com](https://optionark.com). It pulls market and options-chain data, scores contracts against a set of strategy rules, and surfaces the best candidates as watchlists and alerts. It is the research layer that feeds the trading bots.

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

- Consulting: [consulting.optionark.com](https://consulting.optionark.com)
- Trading tools: [optionark.com](https://optionark.com)

Happy to walk through any of these projects on a call.
