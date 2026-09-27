GlassMetric / The Glass Metric / Glass Metric OS

**Unified analytics dashboard for financial trading, sports betting, digital marketing, and bio-performance.**

[![standard-readme compliant](https://img.shields.io/badge/standard--readme-OK-green.svg?style=flat-square)](https://github.com/RichardLitt/standard-readme)

This project is a personal "System Monitor" for the high-performance internet entrepreneur. It aggregates data from the four pillars of the digital economy—**Trading, Betting, Creator Growth, and Bio-Data**—to map how outcomes in one discipline cascade into others.

* **Core Question:** "How do results in one domain map to outcomes in others? How do these feedback loops compound over time?"
* **Goal:** "Vibe code" a functional prototype fast using standard web technologies + a hybrid mobile wrapper.

## 📋 Table of Contents

- [Architecture: The Hybrid Model](#architecture-the-hybrid-model)
- [Tech Stack & Rationale](#tech-stack--rationale)
- [Core Data Specs (The "Systemic" Dataset)](#core-data-specs-the-systemic-dataset)
- [Data Connectors & Setup Guide](#data-connectors--setup-guide)
  - [1. Financial (Prop Firms, Crypto, TradingView)](#1-financial-trading)
  - [2. Sports Betting (Markets & History)](#2-sports-betting)
  - [3. Marketing & Ecommerce (Output & Intent)](#3-marketing--ecommerce)
  - [4. Health & Bio-Data (The Workaround)](#4-health--bio-data-the-workaround)
- [Development Roadmap](#development-roadmap)
- [Getting Started](#getting-started)

## Architecture: The Hybrid Model

We use a **Unified UI / Native Bridge** architecture. You write the dashboard interface once, and it serves both desktop and mobile experiences.

1.  **The "HQ" (Desktop Web):** The primary command center. Optimized for deep work alongside trading terminals.
2.  **The "Mule" (Mobile App):** A lightweight React Native shell. It displays the web dashboard for checking stats on the go, but its primary function is to run a **background service** that syncs offline Apple Health data to the cloud.

---

## Tech Stack & Rationale

We selected this stack to maximize development speed ("vibe coding") while solving the hard problem of accessing local device sensors.

### 🖥️ Frontend (Web Dashboard)
* **Next.js 14 (App Router):** Chosen for its robust server-side rendering and ease of deployment on Vercel. It allows us to build a fast, SEO-friendly dashboard.
* **Tailwind CSS + shadcn/ui:** Provides a high-quality "Glossy Dark Mode" aesthetic out of the box with copy-paste components. No time wasted writing custom CSS.
* **Tremor:** React chart library specifically built for financial dashboards. Looks professional immediately.

### 🗄️ Backend (Infrastructure)
* **Supabase:** The "Firebase for SQL." It gives us a PostgreSQL database, Authentication, and Realtime subscriptions instantly.
* **Edge Functions (Deno):** Used to run the Cron jobs that poll external APIs (TradeLocker, TikTok) and process Webhooks.
* **pgvector:** Built-in vector search support in Postgres, enabling the future "AI Assistant" to query our data using RAG (Retrieval-Augmented Generation).

### 📱 Mobile (The Bridge)
* **React Native (Expo):** Allows us to build the iOS app using JavaScript/React. We avoid writing complex Swift code.
* **react-native-health:** A pre-built bridge to Apple's HealthKit framework.
* **expo-background-fetch:** Enables the app to wake up periodically (even when closed) to sync data.

---

## Core Data Specs (The "Systemic" Dataset)

The dashboard tracks **Inputs (State)** vs. **Outputs (Results)**. We distinguish between "Busyness" (Steps) and "Training" (Distance), and "Vanity" (Followers) vs. "Output" (Velocity).

### 1. 🏥 Health & Bio-Data (The "State" Layer)
* **Heart Rate Variability (HRV):** The primary "Stress/Readiness" detector. Low HRV = High likelihood of impulsive decisions.
* **Resting Heart Rate (RHR):** Burnout indicator. Rising RHR signals need for recovery.
* **Sleep Efficiency:** Time Asleep vs. Time in Bed (Quality > Quantity).
* **Respiratory Rate:** Leading indicator for illness or hangovers.
* **Mindful Minutes:** Meditation/Breathwork sessions.
* **Activity Split:**
    * **Steps:** General activity (often correlates with busy/anxious days).
    * **Distance (Run/Walk/Cycle):** Intentional exercise (Flow state trigger).
* **Time in Daylight:** Dopamine regulation metric (seasonality checks).

### 2. 📈 Financial Trading (The "Discipline" Layer)
* **P&L & Drawdown:** Daily net result and max negative excursion.
* **Max Adverse Excursion (MAE):** How much pain did you endure? High Profit + High MAE = Luck/Recklessness.
* **Number of Trades:** High frequency often signals "Tilt" or chasing losses.
* **Avg. Holding Time:** Measures patience. (e.g., "Do I cut winners early when tired?").
* **Win/Loss Ratio:** Accuracy vs. Profitability check.

### 3. 🎲 Sports Betting (The "Edge" Layer)
* **Closing Line Value (CLV):** Did you beat the market? (True Skill indicator).
* **Turnover (Volume):** Total wagered. Spikes correlate with dopamine chasing.
* **ROI %:** Pure efficiency metric.
* **Average Odds Wagered:** Are you betting safe (1.20) or chasing longshots (5.00+)?
* **Sport Breakdown:** Performance by category (e.g., "Stop betting on Table Tennis").

### 4. 📱 Marketing & Ecommerce (The "Attention" Layer)
* **Daily Content Volume:** The count of "units" shipped today across Threads/TikTok. Tracks high-frequency output vs. paralysis.
* **First-Hour Velocity:** The engagement rate (likes/comments) in the *first 60 minutes* of a post. (Flow state signal).
* **Social Energy (Replies):** Count of DMs/Comments you responded to. (Burnout signal).
* **Cart Addition Rate:** The % of visitors who added to cart (Intent) vs just viewing.
* **Post Consistency:** Daily streak (Discipline metric).

### 5. 🏷️ Context (The "Ghost" Metric)
* **Daily Tag:** A simple manual input field (1-10 Rating or Tag: *Sick, Hungover, Focused, Traveling*) to explain data anomalies.

---

## Data Connectors & Setup Guide

### 1. Financial Trading
*Connects prop firms, crypto exchanges, and chart alerts.*

#### **A. Prop Firms (TradeLocker)**
* **Type:** REST API (Polled).
* **Base URL:** `https://live.tradelocker.com/backend-api`
1.  **Auth:** POST `/auth/jwt/token` with email/password/server to get Bearer Token.
2.  **Context:** GET `/auth/jwt/all-accounts` to retrieve `accountId`.
3.  **Sync:** Create a Supabase Edge Function (Cron: 1h) to poll `/trade/accounts/{id}/state` for Equity/Balance.

#### **B. Crypto (Binance & Coinbase)**
* **Binance:** Use `GET /api/v3/myTrades` (Spot) signed with HMAC SHA256.
* **Coinbase:** Use `GET /v2/accounts/{account_id}/transactions`.
* **Metric:** Aggregate `realized_pnl` daily.

#### **C. TradingView (Webhooks)**
* **Use Case:** Log strategy execution (e.g., "Entered Long on BTC").
1.  Create Supabase Function: `https://[project].supabase.co/functions/v1/tv-webhook`.
2.  **Payload:** `{"ticker": "BTCUSD", "action": "buy", "price": {{close}}, "strategy": "VWAP_Cross"}`.
3.  Add this URL to your TradingView Alert settings.

### 2. Sports Betting
*Tracks prediction markets and value betting.*

#### **A. Prediction Markets (Polymarket)**
* **Type:** REST API (Public).
* **Endpoint:** `https://data-api.polymarket.com/positions?user={wallet_address}`
* **Logic:** Sum `currentValue` of positions vs `costBasis` to determine open P&L.

#### **B. Sportsbooks (The Odds API & Betfair)**
* **Market Data:** Use **[The Odds API](https://the-odds-api.com/)** (Free Tier) to fetch closing lines (CLV).
* **History:** Use **Betfair API** (`listClearedOrders`) for exchange history. For standard books (DraftKings/FanDuel), use a **CSV Import** tool in the dashboard for weekly updates.

### 3. Marketing & Ecommerce
*Tracks the funnel from Attention -> Conversion, focusing on high-frequency output.*

#### **A. High-Volume Social (Threads & TikTok)**
* **Threads (Meta):**
    * **API:** Threads Graph API (`graph.threads.net`).
    * **Metric:** `media_product_type`, `likes`, `replies`.
    * **Rate Limit:** 250 posts / 24h (Great for high-volume logging).
    * **Setup:** Poll `GET /me/threads` hourly to calculate **First-Hour Velocity**.
* **TikTok:**
    * **API:** TikTok Display API (`open.tiktokapis.com`).
    * **Metric:** `follower_count`, `video_count`, `likes_count`.
    * **Setup:** Fetch `GET /v2/video/list` daily.

#### **B. Professional Social (LinkedIn)**
* **Tool:** **Apify** (Official API is restricted for personal analytics).
* **Actor:** `linkedin-profile-scraper` or `linkedin-post-scraper`.
* **Metric:** Engagement Rate (Likes + Comments / Impressions).
* **Why Apify?** LinkedIn's official API requires "Marketing Partner" status for deeper analytics. Apify is the "Vibe Code" workaround.

#### **C. Ecommerce (Shopify)**
* **Type:** GraphQL Admin API.
* **Query:** Fetch `orders` and `checkouts` (Add to Cart events) to measure Intent.
* **Webhook:** Enable `orders/create` to push sales instantly.

### 4. Health & Bio-Data (The Workaround)
*Apple Health data lives strictly on the device (iPhone). It has no cloud API. To access it, we build a custom "Data Mule" app using React Native.*

#### **Why this workaround?**
Web browsers (PWAs) are sandboxed and cannot read `HealthKit`. We need a native app to "tunnel" this data to our Supabase database.

#### **Implementation Steps**
1.  **Repo:** Clone the `/mobile-wrapper-app` folder.
2.  **Dependencies:** We use `react-native-health` to read data and `expo-background-fetch` to run tasks when the app is closed.
3.  **The Logic:**
    * **Task:** Register a background task named `BACKGROUND_SYNC_TASK`.
    * **Frequency:** Set `minimumInterval` to 3600 seconds (1 hour). *Note: iOS determines the exact timing based on battery life.*
    * **Action:**
        ```javascript
        // Pseudo-code for background task
        const healthData = await AppleHealthKit.getSamples({
            startDate: new Date(Date.now() - 60 * 60 * 1000).toISOString(), // Last 1 hr
        });
        await fetch('https://[your-supabase-url]/functions/v1/ingest-health', {
            method: 'POST',
            body: JSON.stringify(healthData)
        });
        ```
4.  **User Flow:** You install the app via TestFlight or direct build. You sign in once. You grant "Read All" permissions. You effectively never open the app again; it just silently feeds your desktop dashboard.

---

## Development Roadmap

### 🏁 Phase 1: The "Unified View" (Weeks 1-2)
*Goal: Get raw data flowing into Supabase and visible on a screen.*
- [ ] **Database:** Schema design (`financial_logs`, `betting_logs`, `marketing_logs`, `health_logs`, `context_logs`).
- [ ] **Web:** Build "Glossy Black" dashboard (Next.js + Tremor).
- [ ] **Mobile Wrapper:** Build basic Expo app with `react-native-webview` pointing to localhost/production URL.
- [ ] **Connectors:** Write Node.js scripts to poll TradeLocker, Binance, and Social APIs.

### 🧠 Phase 2: The "Systemic Engine" (Weeks 3-4)
*Goal: Identify compounding patterns and feedback loops.*
- [ ] **Context Engine:** Build the simple "Pop-up Survey" UI in the Web Dashboard.
- [ ] **Correlation Logic:** Write SQL functions to compare daily rows across domains.
- [ ] **Example Insights to Explore:**
    - "⚠️ **Tilt Index:** High Betting Volume + Low Confidence (Context) + Low HRV."
    - "📉 **The Hangover:** 'Substances' tag (Context) predicts -20% Trading P&L next day."
    - "🚀 **Flow State:** High Sleep + 'High Clarity' tag -> 2x Content Velocity."
- [ ] **Visuals:** Add multi-axis Scatter Plots (e.g., "Financial Result" vs. "Bio-State").

### 🤖 Phase 3: The Language Assistant (Week 5+)
*Goal: Conversational querying.*
- [ ] **Vector Store:** Embed daily summaries into Supabase.
- [ ] **Chat:** Add floating AI bubble to query data: "Show me my betting ROI on days I rated my confidence > 8/10."

---

## Getting Started

### 1. Web Dashboard (Core UI)
```bash
git clone [https://github.com/nicholasmacaskill/the-compounding-creator.git](https://github.com/nicholasmacaskill/the-compounding-creator.git)
cd web-dashboard
npm install
npm run dev
# Dashboard available at http://localhost:3000
2. Mobile Wrapper (Sync + View)
Bash

cd mobile-wrapper-app
npx expo install
# Edit App.js to point WebView to your local or prod URL
npx expo run:ios
