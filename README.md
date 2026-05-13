# Polymarket AI Trading System

[![GitHub Org](https://img.shields.io/badge/org-thinkpixelIab-181717?logo=github)](https://github.com/thinkpixelIab)
[![Node](https://img.shields.io/badge/node-%3E%3D20-brightgreen)](https://nodejs.org)

**Paper-only trading tools** for [Polymarket](https://polymarket.com): mean-reversion ideas from research, optional OpenAI scoring, Kelly-style sizing, and a dashboard to watch everything — **no real money moves through this repo by default.**

| | |
|---|---|
| **Repository** | **[github.com/thinkpixelIab/polymarket-ai-trading](https://github.com/thinkpixelIab/polymarket-ai-trading)** |
| **Try the dashboard** | [polymarket-trading-dashboard.vercel.app](https://polymarket-trading-dashboard.vercel.app) |
| **Stack** | Node.js (Express) · OpenAI · SQLite · static dashboard (Vercel) |

Credit: originated by [b1rdmania](https://github.com/b1rdmania); upstream [HKUDS-AI](https://github.com/HKUDS-AI) · [@jeanbro7](https://github.com/jeanbro7). **This fork:** [thinkpixelIab](https://github.com/thinkpixelIab).

---

## Start here (pick your path)

| If you want to… | Do this |
|-----------------|--------|
| **See it running** | Open the **[live dashboard](https://polymarket-trading-dashboard.vercel.app)** — green connection dot means the backend is reachable. |
| **Run it on your machine** | Follow **[Quick start (local)](#quick-start-local)** — Docker + `.env` + OpenAI key. |
| **Host it 24/7** | Use **[Deploy on Render](#deploy-on-render)** (~\$7/mo for always-on; free tier may sleep). |
| **Read step-by-step docs** | Open **[Getting started](docs/guides/getting-started.md)** or the **[docs index](docs/)**. |

---

## Table of contents

- [Who this is for](#who-this-is-for)
- [What you get](#what-you-get)
- [Current status](#current-status)
- [How the pieces fit together](#how-the-pieces-fit-together)
- [Trading models (simple view)](#trading-models-simple-view)
- [Quick start (local)](#quick-start-local)
- [Deploy on Render](#deploy-on-render)
- [Documentation map](#documentation-map)
- [Research background](#research-background)
- [Toolkit modules](#toolkit-modules)
- [Dashboard](#dashboard)
- [Docker details](#docker-details)
- [Safety & secrets](#safety--secrets)
- [Monitoring](#monitoring)
- [Contributing](#contributing)
- [Links](#links)
- [Disclaimer](#disclaimer)

---

## Who this is for

- You’re curious about **prediction markets** and want a **research-style, paper** setup.
- You’re OK with **Node.js**, **Docker** (for the full stack), and **environment variables**.
- You understand this is **not** a “plug in a wallet and print money” project — it’s for learning, experimentation, and **simulated** P&L.

---

## What you get

In plain terms:

- **Live Polymarket data** (CLOB API) wired into a small **Node/Express** service.
- **Three parallel “personalities”** — Conservative, Moderate, Aggressive — same mean-reversion idea, different risk knobs (still **paper**).
- **Optional AI layer**: OpenAI can help with analysis and **market quality** scoring when you set `OPENAI_API_KEY`.
- **SQLite** stores history so you can compare models and resolutions over time.
- **A web dashboard** to watch signals, model stats, and quality scores.

---

## Current status

| | |
|------------------|-----|
| **Stage** | Paper trading & research |
| **Public dashboard** | [polymarket-trading-dashboard.vercel.app](https://polymarket-trading-dashboard.vercel.app) |
| **Example backend (hosted)** | [polymarket-trading-system.onrender.com](https://polymarket-trading-system.onrender.com) |

**Working today**

- Streaming / market data from Polymarket, multi-model paper loop, signals, resolution tracking, embeddings search (with API key), Dockerized layout, dashboard talking to the API.

**Still evolving**

- Deeper backtests, richer history, execution refinements — still **paper-first**.

**Not the focus of this README**

- Live wallet trading, guaranteed profitability, or production-grade custody — treat those as **separate, high-risk** projects.

---

## How the pieces fit together

```
┌─────────────────────────────────────────┐
│     Dashboard (static host, e.g. Vercel) │
│  Tickers · models · signals · AI panels   │
└─────────────────┬───────────────────────┘
                  │ HTTP (REST)
                  ▼
┌─────────────────────────────────────────┐
│     Node.js API (Express, e.g. :8000)    │
│  /api/models · /api/signals/live · /api/ai/*  │
└─────────────────┬───────────────────────┘
                  │ SQLite files
                  ▼
┌─────────────────────────────────────────┐
│  Docker: 3 traders + API “dashboard” svc │
│  (Conservative / Moderate / Aggressive)  │
└─────────────────┬───────────────────────┘
                  │ HTTP / WebSocket (Polymarket)
                  ▼
┌─────────────────────────────────────────┐
│           Polymarket CLOB API            │
└─────────────────────────────────────────┘
```

---

## Trading models (simple view)

All three models share the same **mean reversion** idea; they differ mainly in **how bold** they are about entries and sizing (Kelly-style multipliers — still simulated).

| Model | Risk vibe | Rough idea |
|-------|-----------|------------|
| **Conservative** | Cautious | Smaller size, higher bar for “this counts as a signal.” |
| **Moderate** | Balanced | Middle ground on size and confidence. |
| **Aggressive** | Bold | Larger size, takes weaker-looking setups (monitor closely). |

Everything stays in **paper mode** unless you deliberately build something else.

---

## Quick start (local)

### Prerequisites

- **Docker Desktop** (recommended for the full 4-service setup).
- **Node 20+** if you run pieces manually ([nodejs.org](https://nodejs.org)).
- An **OpenAI API key** if you want AI scoring / GPT-assisted bits ([platform.openai.com](https://platform.openai.com/api-keys)).

**Windows note:** `scripts/docker.sh` is a **bash** script. Use **Git Bash**, **WSL**, or another bash environment so the commands below work as written.

### Steps

1. **Clone and enter the repo**

```bash
git clone https://github.com/thinkpixelIab/polymarket-ai-trading.git
cd polymarket-ai-trading
```

2. **Configure environment**

```bash
cp .env.example .env
# Edit .env — at minimum set:
# OPENAI_API_KEY=sk-...
```

Never commit `.env`. It should stay gitignored.

3. **Start with Docker** (full stack)

```bash
bash scripts/docker.sh start
bash scripts/docker.sh status    # optional: see services
bash scripts/docker.sh logs      # optional: tail logs
bash scripts/docker.sh smoke     # optional: sanity checks
```

4. **Open the app**

- API + bundled UI: **http://localhost:8000**

**npm (optional, lighter exploration)**

From the repo root, `package.json` also exposes helpers such as `npm run api` (API only) and `npm run dev:frontend` (static frontend on port 3000). Use these when you already know which piece you need; Docker is still the smoothest path for “everything at once.”

---

## Deploy on Render

Good when you want the backend **always online** without your laptop open.

- **Typical cost:** about **\$7/month** for always-on (free tiers may spin down after idle).
- **Outline:** Render Dashboard → **New** → **Blueprint** → connect **`thinkpixelIab/polymarket-ai-trading`** → add **`OPENAI_API_KEY`** → apply.

**More detail:** [RENDER_QUICKSTART.md](RENDER_QUICKSTART.md) · [RENDER_DEPLOY.md](RENDER_DEPLOY.md) · [docs/deployment/render-quickstart.md](docs/deployment/render-quickstart.md)

---

## Documentation map

**[Full docs folder →](docs/)**

| I want to… | Open |
|------------|------|
| Set up from zero | [Getting started](docs/guides/getting-started.md) |
| Understand paper mode | [Paper trading](docs/guides/paper-trading.md) |
| Run backtests | [Backtesting](docs/guides/backtesting.md) |
| Docker locally | [Docker](docs/deployment/docker.md) |
| Production on Render | [Render quickstart](docs/deployment/render-quickstart.md) |
| Frontend hosting | [Vercel](docs/deployment/vercel.md) |

---

## Research background

This project borrows ideas from **prediction-market** and **behavioral** literature — e.g. favorite/longshot dynamics (**Berg & Rietz**), cognitive bias framing (**Munger**), and practical Polymarket context from traders such as **@the_smart_ape**.  

Primary references and notes live under [`research/`](research/).

---

## Toolkit modules

| Module | Role |
|--------|------|
| **polymarket-data** | Fetch and normalize market data |
| **mean-reversion** | Signal logic around mispriced-ish probabilities |
| **execution-engine** | Trade lifecycle (paper today) |
| **volatility-alerts** | Notable move detection |
| **whale-tracker** | Large-position tracking (partial) |

---

## Dashboard

You’ll find:

- **Model comparison** (three profiles side by side)
- **Live signals** with strength hints
- **Market quality** scores (liquidity, spread, activity, clarity)
- **AI insights** when OpenAI is configured
- **Resolution tracking** and **semantic search** over markets

Implementation sketch: vanilla JS in `vercel-frontend/public`, API in `src/server.mjs`, trader loop in `src/trader.mjs`, YAML under `config/`, SQLite under `data/`.

---

## Docker details

Four services typical layout:

- **conservative** / **moderate** / **aggressive** — trader processes  
- **dashboard** — Express API + web UI (port **8000**)

Shared folders:

- `./data` — SQLite  
- `./logs` — logs  
- `./config` — model YAML  

Health checks hit trader processes and `GET /api/health` on the API container. Containers are set to restart on failure.

---

## Safety & secrets

**Paper mode (default path)**

- No on-chain sends from this repo’s happy path  
- No wallet keys required to experiment  
- Keep keys only in `.env`

**If you ever pursue live trading**

Treat that as a **new** security project: custody, kill switches, limits, monitoring, and legal/compliance review — not something this README endorses out of the box.

**Never commit**

- `OPENAI_API_KEY`  
- `POLYGON_PRIVATE_KEY` or any live wallet secret  

---

## Monitoring

Worth watching on the dashboard:

- **Trades / win rate / paper P&L** per model  
- **Market quality** subscores  
- **Backend** connectivity (green = talking to API)  
- **Host health** if self-hosting (Docker/Render uptime, disk for SQLite)

**URLs**

- Local: `http://localhost:8000`  
- Public UI: [polymarket-trading-dashboard.vercel.app](https://polymarket-trading-dashboard.vercel.app)

---

## Contributing

Welcome:

- Bug reports and fixes  
- Doc improvements  
- Research notes and careful strategy discussion  

Please:

- Avoid PRs that **enable live trading** without strong safety review  
- Keep **secrets** out of git  

Forks and experiments encouraged — stay in **paper** until you genuinely trust your setup.

---

## Links

- **Repo:** [github.com/thinkpixelIab/polymarket-ai-trading](https://github.com/thinkpixelIab/polymarket-ai-trading)  
- **Org:** [github.com/thinkpixelIab](https://github.com/thinkpixelIab)  
- **Dashboard:** [polymarket-trading-dashboard.vercel.app](https://polymarket-trading-dashboard.vercel.app)  
- **Polymarket:** [polymarket.com](https://polymarket.com)  
- **Original author:** [@b1rdmania](https://github.com/b1rdmania)

### Related projects

- [Canton Prediction Markets](https://github.com/b1rdmania/canton-prediction-markets)  
- [Aztec Auction Analysis](https://github.com/b1rdmania/aztec-auction-analysis)

---

## Disclaimer

**Educational and research use.** Not financial advice. Paper results ≠ live results. Prediction markets can lose money. Use only what you can afford to lose; no warranty.
