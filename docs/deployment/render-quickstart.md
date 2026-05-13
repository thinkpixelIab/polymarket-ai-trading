# Render Deployment - Quick Start

## ✅ What I Just Set Up

Created a **production-ready Render deployment** that runs your entire Polymarket trading system 24/7 for just **$7/month**.

### Files Created

1. **`render.yaml`** - Render configuration (auto-detected when you connect GitHub)
2. **`RENDER_DEPLOY.md`** - Complete deployment guide with troubleshooting
3. **`scripts/start_all.sh`** - Startup script that runs all 4 processes in one container

### Architecture

```
Single Container ($7/month)
├── Dashboard API (port 8000, public)
├── Conservative Model (background)
├── Moderate Model (background)
└── Aggressive Model (background)
    
All share: /app/data/*.db (SQLite)
```

---

## 🚀 Deploy It Now (5 Steps)

### 1. Go to Render
Visit: https://dashboard.render.com/

### 2. Create New Blueprint
- Click **"New +"** → **"Blueprint"**
- Connect your GitHub account
- Select: `thinkpixelIab/polymarket-ai-trading`
- Click **"Connect"**

### 3. Add OpenAI API Key
- Service will be created: `polymarket-trading-system`
- Go to service → **"Environment"**
- Add variable:
  - **Key**: `OPENAI_API_KEY`
  - **Value**: `sk-proj-ieWz...` (your key)
- Click **"Save Changes"**

### 4. Deploy
- Click **"Manual Deploy"** → **"Deploy latest commit"**
- Wait 5-10 minutes for build
- Watch logs to see all 4 processes start

### 5. Get Your URL
- Copy the public URL (e.g., `https://polymarket-trading-system.onrender.com`)
- Test: `https://your-url.onrender.com/api/health`
- Should return: `{"status": "ok", ...}`

---

## 🔄 Update Vercel Frontend

Once Render is deployed, update your frontend to use the new backend:

### Option 1: Update HTML directly

Edit `vercel-frontend/public/index.html`:

```javascript
// Find this line (around line 20)
const API_URL = 'https://postposted-spent-knife-given.trycloudflare.com';

// Replace with your Render URL
const API_URL = 'https://polymarket-trading-system.onrender.com';
```

Then commit and push:
```bash
git add vercel-frontend/public/index.html
git commit -m "Update API URL to Render deployment"
git push origin master
```

Vercel will auto-redeploy in ~1 minute.

### Option 2: Use environment variable (better)

1. Go to Vercel dashboard
2. Select your project
3. Settings → Environment Variables
4. Add:
   - **Name**: `VITE_API_URL` or `NEXT_PUBLIC_API_URL`
   - **Value**: `https://polymarket-trading-system.onrender.com`
5. Redeploy

Then update your JS to use the env var.

---

## 💰 Cost

**$7/month** (Starter plan)
- 512MB RAM (might need upgrade to 2GB for $25/month if slow)
- 2GB disk
- No spin-down (24/7)
- Shared database access

**Free tier** (if you're testing):
- $0/month
- Spins down after 15 min inactivity
- Takes ~30s to wake up on first request

---

## 📊 Monitoring

### Check It's Working

```bash
# Health check
curl https://your-url.onrender.com/api/health

# Model stats
curl https://your-url.onrender.com/api/models

# Live signals
curl https://your-url.onrender.com/api/signals/live
```

### View Logs

Go to Render dashboard → `polymarket-trading-system` → **"Logs"**

Look for:
```
Starting Conservative model...
Starting Moderate model...
Starting Aggressive model...
All trading models started!
Starting Dashboard API (foreground)...
```

---

## 🐛 Troubleshooting

### "Service failed to start"
- Check logs for Python errors
- Verify `OPENAI_API_KEY` is set
- Make sure you're on Starter plan (not free with auto-suspend)

### "Backend Not Connected" in dashboard
- Check CORS is enabled in `api/dashboard_api.py` (it is)
- Verify Render URL is correct in frontend
- Hard refresh browser (Cmd+Shift+R)

### Out of Memory
- Upgrade to Standard plan (2GB RAM) for $25/month
- Or comment out some models in `scripts/start_all.sh`

---

## 📝 Next Steps

1. ✅ Deploy to Render
2. ✅ Update Vercel frontend URL
3. ✅ Test dashboard live
4. ✅ Monitor logs for 24 hours
5. 🔲 Add custom domain (optional)
6. 🔲 Set up alerting (optional)

---

**All set!** Your trading system will now run 24/7 without needing your laptop open.

**Questions?** Check `RENDER_DEPLOY.md` for the full guide.
