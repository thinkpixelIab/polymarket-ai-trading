# Documentation

Complete documentation for the Polymarket AI Trading System.

## 🚀 Quick Links

- **Getting Started**: [docs/guides/getting-started.md](guides/getting-started.md)
- **Deploy to Render**: [docs/deployment/render-quickstart.md](deployment/render-quickstart.md)
- **Paper Trading**: [docs/guides/paper-trading.md](guides/paper-trading.md)

---

## 📁 Documentation Structure

### Deployment Guides

Set up the system locally or in production.

| Guide | Description |
|-------|-------------|
| [render-quickstart.md](deployment/render-quickstart.md) | **Deploy to Render in 5 steps** ($7/month) |
| [render-complete.md](deployment/render-complete.md) | Complete Render guide with troubleshooting |
| [docker.md](deployment/docker.md) | Local Docker setup |
| [vercel.md](deployment/vercel.md) | Deploy frontend to Vercel |
| [general.md](deployment/general.md) | General deployment concepts |

### User Guides

Learn how to use the system.

| Guide | Description |
|-------|-------------|
| [getting-started.md](guides/getting-started.md) | Complete setup walkthrough |
| [paper-trading.md](guides/paper-trading.md) | How paper trading works |
| [backtesting.md](guides/backtesting.md) | Run historical backtests |
| [raspberry-pi.md](guides/raspberry-pi.md) | Deploy on Raspberry Pi |
| [wallet-setup.md](guides/wallet-setup.md) | Wallet configuration (for live trading) |
| [go-live.md](guides/go-live.md) | ⚠️ Moving to real capital (not recommended) |

### Archive

Historical documentation and design notes.

| Document | Description |
|----------|-------------|
| [SCIEMO_DESIGN_ANALYSIS.md](archive/SCIEMO_DESIGN_ANALYSIS.md) | Dashboard UX design decisions |
| [LESSONS_FROM_SMART_APE.md](archive/LESSONS_FROM_SMART_APE.md) | Trading insights and strategy notes |
| [IMPLEMENTATION_SUMMARY.md](archive/IMPLEMENTATION_SUMMARY.md) | Development history |
| [INTEGRATION_SUMMARY_V2.md](archive/INTEGRATION_SUMMARY_V2.md) | Integration notes |
| [MULTI_MODEL_SUMMARY.md](archive/MULTI_MODEL_SUMMARY.md) | Multi-model architecture notes |
| [REPO_UPDATE_SUMMARY.md](archive/REPO_UPDATE_SUMMARY.md) | Repository update history |
| [BACKTEST_*.md](archive/) | Backtesting implementation notes |

---

## 🆘 Getting Help

1. **Check the guides above** - most questions are answered there
2. **View logs** - `docker compose logs -f` (local) or Render dashboard (production)
3. **Health check** - `curl http://localhost:8000/api/health`
4. **File an issue** - [GitHub Issues](https://github.com/thinkpixelIab/polymarket-ai-trading/issues)

---

## 📝 Contributing to Docs

Found an error or want to improve documentation?

1. Edit the relevant `.md` file
2. Commit: `git commit -m "docs: improve X guide"`
3. Push and create a PR

Keep docs:
- Clear and concise
- Up-to-date with code
- Example-driven
- Beginner-friendly
