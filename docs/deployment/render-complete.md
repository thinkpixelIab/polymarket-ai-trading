# Render Deployment Guide

Deploy your Polymarket AI Trading System to Render for 24/7 operation.

## 💰 Cost Breakdown

**Total: $7/month** (Starter plan - single service)

| Service | Plan | Cost/month | Purpose |
|---------|------|------------|---------|
| Trading System (all-in-one) | Starter | $7 | Dashboard API + 3 trading models |
| **Total** | | **$7** | |

**What you get:**
- ✅ No spin-down (24/7 uptime)
- ✅ 512MB RAM (shared across all 4 processes)
- ✅ 2GB persistent disk storage
- ✅ Auto-deploy from GitHub
- ✅ All services share same SQLite databases

**Why one service instead of four?**
- ✅ Lower cost ($7 vs $28/month)
- ✅ Shared database access (SQLite works properly)
- ✅ Simpler deployment
- ✅ Less resource overhead

**Free tier alternative**: Use free plan (~$0/month) but service spins down after 15 min inactivity

---

## 🚀 Deployment Steps

### 1. Prerequisites

- [ ] GitHub account
- [ ] Render account ([sign up here](https://render.com))
- [ ] OpenAI API key
- [ ] Code pushed to GitHub

### 2. Connect GitHub to Render

1. Go to [dashboard.render.com](https://dashboard.render.com)
2. Click **"New +"** → **"Blueprint"**
3. Connect your GitHub account
4. Select repository: `thinkpixelIab/polymarket-ai-trading`
5. Click **"Connect"**

### 3. Configure Environment Variables

Render will detect the `render.yaml` file. Before deploying, you need to set environment variables:

1. In Render dashboard, go to `polymarket-trading-system` service
2. Click **"Environment"**
3. Add environment variable:
   - **Key**: `OPENAI_API_KEY`
   - **Value**: `sk-...` (your OpenAI API key)
4. Click **"Save Changes"**

That's it! Only one service to configure.

### 4. Deploy

1. Click **"Apply"** to create the service
2. Render will:
   - Build the Docker image
   - Deploy all 4 processes (dashboard + 3 models) in one container
   - Provision a 2GB persistent disk
   - Generate a public URL

**First deployment takes ~5-10 minutes**

Watch the build logs to see progress.

### 5. Get Your Dashboard URL

1. Go to the `polymarket-trading-system` service
2. Copy the public URL (e.g., `https://polymarket-trading-system.onrender.com`)
3. Test it: `https://your-url.onrender.com/api/health`

Should return:
```json
{
  "status": "ok",
  "models_running": true,
  "timestamp": "2026-01-28T..."
}
```

**Check logs to verify all models started:**
```
Starting Conservative model...
Starting Moderate model...
Starting Aggressive model...
All trading models started!
Starting Dashboard API (foreground)...
```

---

## 🔄 Update Vercel Frontend

Now update your Vercel frontend to use the Render backend URL:

1. Open `vercel-frontend/public/index.html`
2. Find the line:
   ```javascript
   const API_URL = 'https://postposted-spent-knife-given.trycloudflare.com';
   ```
3. Replace with your Render URL:
   ```javascript
   const API_URL = 'https://polymarket-dashboard.onrender.com';
   ```
4. Commit and push:
   ```bash
   git add vercel-frontend/public/index.html
   git commit -m "Update API URL to Render deployment"
   git push origin master
   ```
5. Vercel will auto-redeploy

---

## 📊 Monitoring

### Check Service Status

**Dashboard (Render):**
- Go to [dashboard.render.com](https://dashboard.render.com)
- Click on `polymarket-trading-system` service
- View **"Logs"** to see all 4 processes
- Check CPU/RAM usage
- Monitor disk space (should be <50% of 2GB)

**View individual model logs:**
```bash
# Conservative model
tail -f /app/logs/conservative.log

# Moderate model
tail -f /app/logs/moderate.log

# Aggressive model
tail -f /app/logs/aggressive.log
```

(Access these via Render's shell or download logs)

**Health Endpoints:**
```bash
# Overall health
curl https://your-url.onrender.com/api/health

# Model stats
curl https://your-url.onrender.com/api/models

# Live signals
curl https://your-url.onrender.com/api/signals/live
```

### View Logs

**Via Render Dashboard:**
1. Go to your service
2. Click **"Logs"** tab
3. See combined output from all 4 processes

**Using Render CLI** (optional):
```bash
# Install CLI
npm install -g render

# View live logs
render logs --service polymarket-trading-system --tail

# Search logs
render logs --service polymarket-trading-system | grep "Conservative"
```

**Log files on disk:**
- `/app/logs/conservative.log`
- `/app/logs/moderate.log`
- `/app/logs/aggressive.log`
- Dashboard logs go to stdout (visible in Render dashboard)

---

## 🛠️ Architecture: All-in-One Container

**How it works:**

```
┌─────────────────────────────────────────────┐
│  Render Container (512MB RAM, 2GB Disk)    │
│                                             │
│  ┌────────────────────────────────────┐   │
│  │  Process 1: Conservative Model     │   │
│  │  (background, logs to file)        │   │
│  └────────────────────────────────────┘   │
│                                             │
│  ┌────────────────────────────────────┐   │
│  │  Process 2: Moderate Model         │   │
│  │  (background, logs to file)        │   │
│  └────────────────────────────────────┘   │
│                                             │
│  ┌────────────────────────────────────┐   │
│  │  Process 3: Aggressive Model       │   │
│  │  (background, logs to file)        │   │
│  └────────────────────────────────────┘   │
│                                             │
│  ┌────────────────────────────────────┐   │
│  │  Process 4: Dashboard API          │   │
│  │  (foreground, keeps container alive)│   │
│  │  Port 8000 → Public URL            │   │
│  └────────────────────────────────────┘   │
│                                             │
│  Shared: /app/data/*.db (SQLite)          │
│  Shared: /app/logs/*.log                  │
└─────────────────────────────────────────────┘
```

**Benefits:**
- All processes share the same SQLite databases
- Lower cost (one service instead of four)
- Simpler deployment
- Logs accessible via Render dashboard

---

## 🔐 Security

### Environment Variables

Never commit these to Git:
- `OPENAI_API_KEY`
- `POLYGON_PRIVATE_KEY` (if you add real trading)

Use Render's environment variable management instead.

### API Access

The dashboard API is **public by default**. To secure it:

**Option 1: Add authentication**
```python
# In api/dashboard_api.py
from fastapi import Header, HTTPException

async def verify_token(x_api_key: str = Header(None)):
    if x_api_key != os.getenv('API_SECRET_KEY'):
        raise HTTPException(status_code=401, detail="Invalid API key")

@app.get("/api/models", dependencies=[Depends(verify_token)])
async def get_all_models():
    # ...
```

**Option 2: Use Render's IP allowlist** (Pro plan only)

**For now**: The dashboard is public but read-only (no sensitive data exposed)

---

## 🐛 Troubleshooting

### Services Not Starting

**Check logs:**
1. Go to Render dashboard
2. Click `polymarket-trading-system` service
3. View **"Logs"** tab
4. Look for errors in startup sequence

**Common issues:**
- Missing dependencies (add to `Dockerfile`)
- Environment variable `OPENAI_API_KEY` not set
- Out of memory (512MB might be tight for 4 processes)
- Port conflict (should be 8000)

**If out of memory:**
Upgrade to Standard plan (2GB RAM) for $25/month

### Dashboard Shows "Backend Not Connected"

**Debug steps:**
1. Check dashboard service is running: `https://your-url.onrender.com/api/health`
2. Check CORS settings in `api/dashboard_api.py`:
   ```python
   app.add_middleware(
       CORSMiddleware,
       allow_origins=["*"],  # Should allow Vercel domain
       ...
   )
   ```
3. Check browser console for errors

### Models Not Trading

**Check individual model logs:**
1. Go to service → **"Shell"** (if available)
2. Run: `tail -f /app/logs/conservative.log`
3. Or download logs from Render dashboard

**Common issues:**
- No markets found (normal if low volume)
- API rate limits (Polymarket throttling)
- Configuration errors (check `config/*.yaml` files)
- Models writing to DB but not finding signals (expected behavior)

### Out of Disk Space

**Check disk usage:**
1. Go to service → **"Disk"** tab
2. If >80% full, increase size:
   - Edit `render.yaml`
   - Change `sizeGB: 1` to `sizeGB: 2`
   - Push to GitHub (auto-redeploys)

---

## 🔄 Auto-Deploy from GitHub

**Render automatically deploys when you push to `master`:**

```bash
# Make changes
git add .
git commit -m "Update trading logic"
git push origin master

# Render detects push and redeploys automatically
```

**To disable auto-deploy:**
1. Go to service → **"Settings"**
2. Toggle **"Auto-Deploy"** off

---

## 📈 Scaling

### If You Need More Power

**Upgrade to Standard plan** ($25/month):
- ✅ 2GB RAM (vs 512MB) - **Recommended if models are slow**
- ✅ Faster CPU
- ✅ More disk I/O

**Or use Pro plan** ($85/month):
- 4GB RAM
- Dedicated CPU
- IP allowlist

### Cost Optimization

**If $7/month is too much:**
- Use **free tier** (with 15-min spin-down)
- Models restart on each request (slow but functional)

**To reduce resource usage:**
- Run only Conservative model (comment out others in `start_all.sh`)
- Increase sleep intervals in `systematic_trader.py` (check less frequently)
- Use PostgreSQL instead of SQLite (future enhancement)

---

## 🔗 Links

- **Render Dashboard**: https://dashboard.render.com
- **Render Docs**: https://render.com/docs
- **Your Services**: https://dashboard.render.com/services
- **Support**: https://render.com/support

---

## 📝 Next Steps After Deployment

1. ✅ Verify service is running and healthy
2. ✅ Check `/api/health` endpoint returns 200
3. ✅ View logs to confirm all 4 processes started
4. ✅ Update Vercel frontend with Render URL
5. ✅ Test dashboard live at your Vercel URL
6. ✅ Monitor logs for first 24 hours
7. ✅ Check disk usage (should be <500MB)
8. ✅ Monitor RAM usage (if >400MB, consider upgrading)
9. 🔲 Set up alerting (Discord webhook, email, etc.)
10. 🔲 Add PostgreSQL for better performance (optional)
11. 🔲 Configure custom domain (optional)

---

**Questions?** Check the logs first, then file an issue on GitHub.

**Cost too high?** Use free tier (with spin-down) or comment out models you don't need.

**Performance issues?** Upgrade to Standard plan ($25/month) for 2GB RAM.
