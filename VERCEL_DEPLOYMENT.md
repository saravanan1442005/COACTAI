# CoAct.AI - Vercel + Render Deployment Guide

This guide shows how to deploy your full-stack CoAct.AI application using **Vercel** (frontend) and **Render** (backend).

---

## 🎯 Deployment Architecture

- **Frontend**: Hosted on **Vercel** (React/Vite) → https://your-app.vercel.app
- **Backend**: Hosted on **Render** (Flask/Python) → https://your-backend.onrender.com
- **Database**: **Supabase** (already cloud-hosted)
- **AI Services**: **Azure OpenAI** (already cloud-hosted)

---

## 📋 Prerequisites

- [ ] GitHub account with your repository pushed
- [ ] Vercel account (free tier is fine)
- [ ] Render account (free tier is fine)
- [ ] Supabase project set up
- [ ] Azure OpenAI API keys

---

## 🔧 Part 1: Backend Deployment (Render)

### Step 1: Prepare Backend for Render

Create a `render.yaml` file in your project root:

```yaml
services:
  - type: web
    name: coactai-backend
    env: python
    region: oregon
    plan: free
    buildCommand: "pip install -r inter-ai-backend/requirements.txt"
    startCommand: "cd inter-ai-backend && python app.py"
    envVars:
      - key: PYTHON_VERSION
        value: 3.11.0
      - key: FLASK_ENV
        value: production
      - key: PORT
        value: 5000
```

### Step 2: Update Backend for Production

Edit `inter-ai-backend/app.py` to read the PORT from environment:

```python
# At the end of app.py, change:
if __name__ == '__main__':
    port = int(os.environ.get('PORT', 5000))
    app.run(host='0.0.0.0', port=port, debug=False)
```

### Step 3: Deploy to Render

1. Go to [Render Dashboard](https://dashboard.render.com/)
2. Click **"New +"** → **"Web Service"**
3. Connect your **GitHub repository**
4. Select your repository: `suyashB45/COACTAI`
5. Configure:
   - **Name**: `coactai-backend`
   - **Region**: Choose closest to you
   - **Branch**: `main`
   - **Root Directory**: Leave blank
   - **Runtime**: `Python 3`
   - **Build Command**: `pip install -r inter-ai-backend/requirements.txt`
   - **Start Command**: `cd inter-ai-backend && python app.py`

6. Add **Environment Variables**:
   ```
   AZURE_OPENAI_API_KEY=your-key
   AZURE_OPENAI_ENDPOINT=your-endpoint
   AZURE_OPENAI_API_VERSION=2024-12-01-preview
   GPT_DEPLOYMENT_NAME=gpt-4o-mini
   TTS_DEPLOYMENT=tts
   STT_DEPLOYMENT=whisper
   EMBEDDINGS_DEPLOYMENT=text-embedding-ada-002
   SUPABASE_URL=https://your-project.supabase.co
   SUPABASE_KEY=your-anon-key
   SUPABASE_SERVICE_KEY=your-service-role-key
   DATABASE_URL=postgresql://postgres:password@host.supabase.co:6543/postgres?sslmode=require
   FLASK_ENV=production
   PORT=5000
   ```

7. Click **"Create Web Service"**

8. **Copy your backend URL**: e.g., `https://coactai-backend.onrender.com`

### Step 4: Test Backend

Once deployed, test:
```bash
curl https://coactai-backend.onrender.com/api/health
```

Should return: `{"status": "healthy"}`

---

## 🎨 Part 2: Frontend Deployment (Vercel)

### Step 1: Update Frontend API URL

Edit `inter-ai-frontend/.env` (or create `.env.production`):

```env
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_KEY=your-anon-key
VITE_API_URL=https://coactai-backend.onrender.com
```

### Step 2: Update CORS in Backend

Add your Vercel URL to CORS allowed origins in `inter-ai-backend/app.py`:

```python
CORS(app, resources={
    r"/*": {
        "origins": [
            "http://localhost:3000",
            "http://localhost:5173",
            "https://your-app.vercel.app",  # Add this
            "https://*.vercel.app"          # Allow all preview deployments
        ],
        "allow_headers": ["Content-Type", "Authorization"],
        "supports_credentials": True
    }
})
```

Commit and push this change - Render will auto-redeploy.

### Step 3: Deploy to Vercel

#### Option A: Using Vercel Dashboard (Recommended)

1. Go to [Vercel Dashboard](https://vercel.com/dashboard)
2. Click **"Add New Project"**
3. Import your GitHub repository: `suyashB45/COACTAI`
4. Configure:
   - **Framework Preset**: Vite
   - **Root Directory**: `inter-ai-frontend`
   - **Build Command**: `npm run build`
   - **Output Directory**: `dist`
   - **Install Command**: `npm install`

5. Add **Environment Variables**:
   ```
   VITE_SUPABASE_URL=https://your-project.supabase.co
   VITE_SUPABASE_KEY=your-anon-key
   VITE_API_URL=https://coactai-backend.onrender.com
   ```

6. Click **"Deploy"**

#### Option B: Using Vercel CLI

```bash
# Install Vercel CLI
npm i -g vercel

# Navigate to frontend
cd inter-ai-frontend

# Login to Vercel
vercel login

# Deploy
vercel --prod
```

### Step 4: Configure Custom Domain (Optional)

1. In Vercel Dashboard → **Settings** → **Domains**
2. Add your custom domain (e.g., `coact-ai.com`)
3. Update DNS records as instructed by Vercel
4. SSL is automatically configured!

---

## ✅ Part 3: Verification

### Test Complete Flow

1. Visit your Vercel URL: `https://your-app.vercel.app`
2. **Sign up** for a new account
3. **Login** 
4. Go to **Practice** page
5. Start a **conversation**
6. Verify the AI responds correctly
7. Check **Reports** are generated

### Check Logs

**Backend (Render)**:
- Go to Render Dashboard → Your Service → **Logs**

**Frontend (Vercel)**:
- Go to Vercel Dashboard → Your Project → **Deployments** → Click deployment → **Function Logs**

---

## 🔄 Part 4: Continuous Deployment

Both Vercel and Render support **automatic deployments**:

1. **Make changes** to your code
2. **Commit and push** to GitHub:
   ```bash
   git add .
   git commit -m "Update feature"
   git push origin main
   ```
3. **Auto-deploy**: Both platforms will automatically detect changes and redeploy!

---

## 💰 Cost Breakdown (Free Tier)

| Service | Free Tier | Notes |
|---------|-----------|-------|
| **Vercel** | 100GB bandwidth/month | More than enough for MVP |
| **Render** | 750 hours/month | Backend spins down after 15min inactivity |
| **Supabase** | 500MB database, 50k users | Great for starting |
| **Azure OpenAI** | Pay-per-use | ~$0.15 per 1K tokens (GPT-4o-mini) |

**Total Cost**: ~$0/month + Azure API usage

---

## 🚨 Important Notes

### Render Free Tier Limitations

- **Cold Starts**: Backend spins down after 15 minutes of inactivity
- **First request**: May take 30-60 seconds to wake up
- **Solution**: Upgrade to $7/month for always-on instance

### Handling Cold Starts

Add a health check in frontend to warm up backend:

```typescript
// In inter-ai-frontend/src/App.tsx or main component
useEffect(() => {
  // Warm up backend on app load
  fetch(`${import.meta.env.VITE_API_URL}/api/health`)
    .catch(err => console.log('Backend warming up...'));
}, []);
```

---

## 🔧 Troubleshooting

### Frontend can't reach Backend

**Issue**: CORS errors in browser console

**Fix**:
1. Add Vercel URL to CORS origins in `app.py`
2. Ensure backend `CORS_ORIGINS` includes your Vercel domain
3. Redeploy backend on Render

### Environment Variables Not Working

**Issue**: App says "API key missing"

**Fix**:
1. Check all env vars are set in Vercel/Render dashboards
2. Ensure `VITE_` prefix for frontend vars
3. Redeploy after adding env vars

### Database Connection Failed

**Issue**: Backend can't connect to Supabase

**Fix**:
1. Verify `DATABASE_URL` in Render env vars
2. Check Supabase project is active (not paused)
3. Whitelist Render IP in Supabase (or allow all IPs for testing)

---

## 🎯 Next Steps

### Production Checklist

- [ ] Add custom domain to Vercel
- [ ] Enable Vercel Analytics
- [ ] Set up Render health checks
- [ ] Configure Supabase Row Level Security (RLS)
- [ ] Add error tracking (Sentry)
- [ ] Set up monitoring (Uptime Robot)
- [ ] Configure backup strategy for database

### Upgrade Path

When you outgrow free tier:

1. **Render**: Upgrade to Starter ($7/month) for always-on backend
2. **Vercel**: Pro plan ($20/month) for team collaboration
3. **Supabase**: Pro plan ($25/month) for 8GB database

---

## 📚 Resources

- [Vercel Documentation](https://vercel.com/docs)
- [Render Documentation](https://render.com/docs)
- [Supabase Documentation](https://supabase.com/docs)
- [Your full deployment guide](./DEPLOYMENT.md) (for VPS deployment)

---

**Deployment Date**: February 2026  
**Version**: 1.0  
**Platform**: Vercel + Render + Supabase
