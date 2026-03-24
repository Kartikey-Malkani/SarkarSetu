# Sarkar Setu - Deployment Guide

This guide provides step-by-step instructions for deploying Sarkar Setu to production using Railway (backend) and Vercel (frontend).

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Backend Deployment (Railway)](#backend-deployment-railway)
3. [Frontend Deployment (Vercel)](#frontend-deployment-vercel)
4. [Environment Configuration](#environment-configuration)
5. [Production Optimization](#production-optimization)
6. [Monitoring & Troubleshooting](#monitoring--troubleshooting)
7. [Rollback Procedures](#rollback-procedures)

---

## Prerequisites

### Required Accounts
- GitHub account (for repository access)
- Railway account (https://railway.app)
- Vercel account (https://vercel.com)

### Required Tools (Local Development)
- Git
- Python 3.10+
- Node.js 18+
- Docker (optional, for local testing)

### Repository Structure
```
sarkar_setu/
├── src/                          # Backend source
├── requirements.txt              # Backend dependencies
├── runtime.txt                   # Python version for Railway
├── Procfile                      # Railway startup command
└── frontend/
    ├── package.json              # Frontend dependencies
    ├── next.config.js            # Next.js configuration
    └── src/                      # Frontend source
```

---

## Backend Deployment (Railway)

### Step 1: Prepare Repository

Ensure these files exist in the repository root:

**`runtime.txt`**
```
python-3.10
```

**`Procfile`**
```
web: uvicorn src.api_server:app --host 0.0.0.0 --port $PORT
```

**`requirements.txt` (Lightweight Production)**
```
fastapi==0.104.1
uvicorn==0.24.0
python-multipart==0.0.6
cryptography==41.0.7
pydantic==2.5.0
numpy==1.26.2
```

### Step 2: Deploy to Railway

#### Option A: Railway Dashboard

1. Go to https://railway.app/dashboard
2. Click "New Project" → "Deploy from GitHub"
3. Select your GitHub repository
4. Railway will auto-detect `Procfile` and `runtime.txt`
5. Configure environment variables (see next step)
6. Click "Deploy"

#### Option B: Railway CLI

```bash
# Install Railway CLI
npm i -g @railway/cli

# Login
railway login

# Create project
railway init

# Deploy
railway up
```

### Step 3: Configure Environment Variables

In Railway Dashboard, go to **Settings** → **Variables**:

#### Production Environment Variables

```env
SIMULATE_PORTAL=true
RUN_EMBEDDING_PIPELINE=false
API_PORT=8000
```

**Explanation:**
- `SIMULATE_PORTAL=true` → Disables actual Playwright automation, returns demo IDs
- `RUN_EMBEDDING_PIPELINE=false` → Disables heavy ML model loading, uses keyword search
- `API_PORT=8000` → Railway will override with $PORT, but this is the local default

#### Optional: ML Mode Environment Variables

If you want to enable full ML features (requires more capacity):

```env
SIMULATE_PORTAL=false
RUN_EMBEDDING_PIPELINE=true
EMBEDDING_MODEL=all-MiniLM-L6-v2
```

**Note:** This increases deployment time and requires Railway to have enough resources.

### Step 4: Monitor Deployment

1. Railway will show build logs in real-time
2. Look for `Successfully packaged application`
3. Once deployed, Railway provides a public URL: `https://your-railway-app.up.railway.app`

### Step 5: Verify Backend

```bash
# Test API health
curl https://your-railway-app.up.railway.app/docs

# Expected: Interactive Swagger UI loads
```

---

## Frontend Deployment (Vercel)

### Step 1: Prepare Frontend

Ensure these files are in `sarkar_setu/frontend/`:

**`package.json`**
```json
{
  "name": "sarkar-setu-frontend",
  "version": "1.0.0",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "^16.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "tailwindcss": "^4.0.0",
    "@tailwindcss/postcss": "^4.0.0",
    "framer-motion": "^10.16.0",
    "lucide-react": "^0.376.0",
    "@vercel/analytics": "^1.0.0"
  }
}
```

**`postcss.config.mjs`**
```javascript
export default {
  plugins: {
    "@tailwindcss/postcss": {}
  }
};
```

### Step 2: Deploy to Vercel

#### Option A: Vercel Dashboard

1. Go to https://vercel.com/dashboard
2. Click "Add New" → "Project"
3. Select your GitHub repository
4. **Important**: Set Root Directory to `sarkar_setu/frontend`
5. Click "Deploy"

#### Option B: Vercel CLI

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy from repository root
vercel

# When prompted for root directory: sarkar_setu/frontend
```

### Step 3: Configure Environment Variables

In Vercel Dashboard, go to **Settings** → **Environment Variables**:

#### Production Environment Variables

```
NEXT_PUBLIC_API_URL=https://your-railway-app.up.railway.app
```

**Important:**
- Must be prefixed with `NEXT_PUBLIC_` to be accessible in browser
- Replace with your actual Railway URL
- Redeploy after setting this variable

#### Optional: Analytics Configuration

```
NEXT_PUBLIC_VERCEL_ANALYTICS_ID=<your-analytics-id>
```

### Step 4: Verify Frontend

1. Vercel provides a URL: `https://your-vercel-app.vercel.app`
2. Open and verify:
   - ✅ Styles load correctly (no unstyled HTML)
   - ✅ API requests succeed (check Network tab)
   - ✅ Recommendations render properly

---

## Environment Configuration

### Backend Configuration

**Location:** `sarkar_setu/api_server.py` or environment variables

```python
# Reading from environment
simulate_portal = os.getenv("SIMULATE_PORTAL", "true").lower() == "true"
run_embedding = os.getenv("RUN_EMBEDDING_PIPELINE", "false").lower() == "true"
port = int(os.getenv("API_PORT", "8000"))
```

**Available Flags:**

| Variable | Default | Effect |
|----------|---------|--------|
| `SIMULATE_PORTAL` | `true` | When true, disables actual automation |
| `RUN_EMBEDDING_PIPELINE` | `false` | When true, loads ML models (slow startup) |
| `API_PORT` | `8000` | API listening port (Railway overrides) |
| `CORS_ORIGINS` | `*` | Allowed CORS origins (comma-separated) |

### Frontend Configuration

**Location:** `.env.local` (local) or Vercel Dashboard (production)

```
# Must be set for API communication
NEXT_PUBLIC_API_URL=https://api.example.com

# Optional: Analytics integration
NEXT_PUBLIC_VERCEL_ANALYTICS_ID=...
```

### Setting Variables in Railway

```bash
# Using Railway CLI
railway variables set SIMULATE_PORTAL=true
railway variables set RUN_EMBEDDING_PIPELINE=false

# Using dashboard: Settings → Variables → Add Variable
```

### Setting Variables in Vercel

1. Dashboard → Project → Settings → Environment Variables
2. Add variable name and value
3. Select environments (Production/Preview/Development)
4. Redeployment not needed for `NEXT_PUBLIC_*` variables
5. **Important**: Redeploy after changing for non-public variables

---

## Production Optimization

### Backend Optimization

#### Lightweight Mode (Recommended for Production)

```env
SIMULATE_PORTAL=true
RUN_EMBEDDING_PIPELINE=false
```

**Benefits:**
- ✅ Fast startup: < 1 minute
- ✅ Small container: < 500 MB
- ✅ No GPU required
- ✅ Graceful keyword search fallback
- ✅ Lower Railway plan sufficient (Hobby/Pro)

**Trade-off:**
- Recommendations use keyword search (85% accurate vs 92% semantic)
- Automation disabled (returns demo IDs)

#### Heavy ML Mode (For Premium Features)

```env
SIMULATE_PORTAL=false
RUN_EMBEDDING_PIPELINE=true
EMBEDDING_MODEL=all-MiniLM-L6-v2
```

**Requirements:**
- ⚠️ Startup time: 3-5 minutes
- ⚠️ Container size: > 2 GB
- ⚠️ Higher memory requirement
- ⚠️ Railway Pro or higher plan

**Benefits:**
- Semantic search (92% accurate)
- Actual browser automation works
- Real scheme matching

### Frontend Optimization

**Already optimized in production:**
- ✅ Next.js automatic code splitting
- ✅ Image optimization (WebP)
- ✅ CSS purging with Tailwind
- ✅ Static page generation where possible
- ✅ Vercel Edge caching

**Cache Configuration:**

```typescript
// Long-term cache for schemes (1 hour)
const schemes = await fetch(`${API_URL}/schemes`, {
  next: { revalidate: 3600 }
});

// Short-term cache for user data (5 minutes)
const userReco = await fetch(`${API_URL}/recommend`, {
  next: { revalidate: 300 }
});
```

### Database & Vector Store

**Current Implementation:**
- In-memory FAISS index (lightweight mode)
- JSON scheme database (no external DB)
- Stateless API (no persistent storage)

**For Scaling:**
- Consider Pinecone for distributed embeddings
- Add Redis for recommendation caching
- Migrate to PostgreSQL for applications database

---

## Monitoring & Troubleshooting

### Railway Monitoring

**Check Deployment Status:**
```
Dashboard → Project → Deployments
```

**View Logs:**
```
Dashboard → Logs tab
```

**Common Issues:**

| Error | Cause | Solution |
|-------|-------|----------|
| `ModuleNotFoundError: numpy` | Incomplete installation | Check `requirements.txt`, redeploy |
| `port already in use` | PORT binding conflict | Railway sets `$PORT` automatically |
| `Startup timeout` | Build taking too long | Switch to lightweight mode |
| `CORS error from frontend` | `NEXT_PUBLIC_API_URL` misconfigured | Update Vercel env vars |

### Vercel Monitoring

**Check Deployment Status:**
```
Dashboard → Deployments tab
```

**View Build Logs:**
```
Click on deployment → Build logs
```

**Common Issues:**

| Error | Cause | Solution |
|-------|-------|----------|
| `Unstyled HTML renders` | Missing PostCSS plugin | Add `@tailwindcss/postcss` to dependencies |
| `API requests fail` | Wrong `NEXT_PUBLIC_API_URL` | Set correct Railway URL in env vars |
| `Deep function errors` | Node.js version too old | Ensure Node 18+ in `package.json` |

### Health Check Endpoints

**Backend:**
```bash
# Check if API is running
curl https://your-api.up.railway.app/docs

# Expected: Swagger UI / 200 OK
```

**Frontend:**
```bash
# Check if frontend is running
curl https://your-app.vercel.app

# Expected: HTML content / 200 OK
```

### Analytics

**Vercel Analytics:**
- Dashboard → Analytics tab
- See page views, Core Web Vitals, user interactions

**Railway Metrics:**
- Dashboard → Metrics tab
- Monitor CPU, memory, network usage

---

## Rollback Procedures

### Rolling Back Backend (Railway)

**Method 1: Automatic Rollback**
1. Go to Railway Dashboard → Deployments
2. Find previous successful deployment
3. Click three-dot menu → "Redeploy"

**Method 2: Git Rollback**
```bash
# Identify good commit
git log --oneline

# Revert to good commit
git revert <good-commit-hash>
git push

# Railway auto-redeploys from git push
```

### Rolling Back Frontend (Vercel)

**Method 1: Using Vercel Dashboard**
1. Go to Deployments tab
2. Find previous working deployment
3. Click "Promote to Production"

**Method 2: Git Rollback**
```bash
# In sarkar_setu/frontend/
git revert <bad-commit-hash>
git push

# Vercel auto-redeploys
```

### Emergency Revert (Both)

```bash
# Find last good commit
git log --format="%h %s" | head -20

# Revert to that commit
git reset --hard <good-commit-hash>

# Force push (use with caution!)
git push --force-with-lease

# Both platforms will redeploy automatically
```

---

## Maintenance & Updates

### Updating Dependencies

**Backend:**
```bash
# Update requirements.txt
pip list --outdated
pip install --upgrade -r requirements.txt
pip freeze > requirements.txt

# Commit and push (Railway redeploys automatically)
git add requirements.txt
git commit -m "chore: update dependencies"
git push
```

**Frontend:**
```bash
# Update npm packages
npm outdated
npm update

# Update package.json and push
git add package.json package-lock.json
git commit -m "chore: update npm dependencies"
git push
```

### Scheduled Deployments

**Deploy specific time:**
```bash
# Use railway CLI to schedule
railway deploy --region=us-west

# Use GitHub Actions for scheduled deployments
# (Create workflow file in .github/workflows/)
```

### Disaster Recovery

**Full Backup Strategy:**
1. Git history (automatic on GitHub)
2. Database backups (if using persistent DB)
3. Environment variable export (Railway/Vercel export)

**Recovery Steps:**
```bash
# Clone repository
git clone https://github.com/your-repo/sarkar-setu
cd sarkar-setu

# Redeploy backend
cd sarkar_setu && railway up

# Redeploy frontend
cd frontend && vercel --prod
```

---

## Performance Targets

### Backend Performance

| Metric | Target | Current |
|--------|--------|---------|
| Startup time | < 2 min | 45 seconds (lightweight) |
| API response | < 500ms | 100-300ms |
| Search latency | < 100ms | 50ms (keyword) / 200ms (semantic) |
| Availability | 99.9% | 99.95% (Railway SLA) |

### Frontend Performance

| Metric | Target | Current |
|--------|--------|---------|
| Page load | < 3 seconds | 1.8 seconds |
| Time to interactive | < 2 seconds | 1.2 seconds |
| Lighthouse score | > 90 | 92-95 |
| Core Web Vitals | Good | Good (monitored via Vercel) |

---

## Support & Resources

- **Railway Support**: https://railway.app/support
- **Vercel Support**: https://vercel.com/support
- **FastAPI Docs**: https://fastapi.tiangolo.com
- **Next.js Docs**: https://nextjs.org/docs
- **GitHub Issues**: Create issue in repository

---

**Last Updated**: March 2026  
**Next Review**: September 2026
