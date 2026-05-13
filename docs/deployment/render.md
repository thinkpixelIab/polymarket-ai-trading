# Render Deployment Guide

Complete guide for deploying the Polymarket AI Trading System to Render.

**Cost**: $7/month for 24/7 uptime

## Quick Deploy (5 Steps)

### 1. Sign up for Render
Visit: https://dashboard.render.com/

### 2. Connect GitHub
- Click **"New +"** → **"Blueprint"**
- Connect your GitHub account
- Select: `thinkpixelIab/polymarket-ai-trading`

### 3. Add Environment Variable
- Service created: `polymarket-trading-system`
- Go to **Environment** tab
- Add: `OPENAI_API_KEY` = `sk-...`

### 4. Deploy
- Click **"Manual Deploy"**
- Wait 5-10 minutes for build

### 5. Get Your URL
- Copy public URL (e.g., `https://polymarket-trading-system.onrender.com`)
- Test: `curl https://your-url.onrender.com/api/health`

---

For full documentation, see the original guides:
- [RENDER_QUICKSTART.md](../../RENDER_QUICKSTART.md)
- [RENDER_DEPLOY.md](../../RENDER_DEPLOY.md)
