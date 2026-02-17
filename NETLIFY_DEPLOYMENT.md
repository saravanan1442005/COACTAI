# CoAct.AI - Netlify + Render Deployment Guide

This guide shows how to deploy your full-stack CoAct.AI application using **Netlify** (frontend) and **Render** (backend).

---

## 🎯 Deployment Architecture

- **Frontend**: Hosted on **Netlify** (React/Vite) → https://your-app.netlify.app
- **Backend**: Hosted on **Render** (Flask/Python) → https://your-backend.onrender.com
- **Database**: **Supabase** (already cloud-hosted)
- **AI Services**: **Azure OpenAI** (already cloud-hosted)

---

## 📋 Prerequisites

- [ ] GitHub account with your repository pushed
- [ ] Netlify account (free tier is fine)
- [ ] Render account (free tier is fine)
- [ ] Supabase project set up
- [ ] Azure OpenAI API keys

---

## 🔧 Part 1: Backend Deployment (Render)

Follow the **same backend setup** as in `VERCEL_DEPLOYMENT.md` - the backend deployment is identical.

**Quick Steps**:
1. Go to [Render Dashboard](https://dashboard.render.com/)
2. Create new Web Service from GitHub
3. Set build command: `pip install -r inter-ai-backend/requirements.txt`
4. Set start command: `cd inter-ai-backend && python app.py`
5. Add all environment variables
6. Deploy and copy your backend URL

---

## 🎨 Part 2: Frontend Deployment (Netlify)

### Step 1: Update Frontend API URL

Create `inter-ai-frontend/.env.production`:

```env
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_KEY=your-anon-key
VITE_API_URL=https://coactai-backend.onrender.com
```

### Step 2: Create Netlify Configuration

Create `netlify.toml` in your project root:

```toml
[build]
  base = "inter-ai-frontend"
  command = "npm run build"
  publish = "dist"

[build.environment]
  NODE_VERSION = "18"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "no-referrer"
```

### Step 3: Update CORS in Backend

Add your Netlify URL to CORS allowed origins in `inter-ai-backend/app.py`:

```python
CORS(app, resources={
    r"/*": {
        "origins": [
            "http://localhost:3000",
            "http://localhost:5173",
            "https://your-app.netlify.app",  # Add this
            "https://*.netlify.app"          # Allow all preview deployments
        ],
        "allow_headers": ["Content-Type", "Authorization"],
        "supports_credentials": True
    }
})
```

### Step 4: Deploy to Netlify

#### Option A: Using Netlify Dashboard (Recommended)

1. Go to [Netlify Dashboard](https://app.netlify.com/)
2. Click **"Add new site"** → **"Import an existing project"**
3. Connect to **GitHub**
4. Select repository: `suyashB45/COACTAI`
5. Configure build settings:
   - **Base directory**: `inter-ai-frontend`
   - **Build command**: `npm run build`
   - **Publish directory**: `inter-ai-frontend/dist`
   - **Branch to deploy**: `main`

6. Click **"Advanced settings"** → **"Add environment variables"**:
   ```
   VITE_SUPABASE_URL=https://your-project.supabase.co
   VITE_SUPABASE_KEY=your-anon-key
   VITE_API_URL=https://coactai-backend.onrender.com
   ```

7. Click **"Deploy site"**

#### Option B: Using Netlify CLI

```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login to Netlify
netlify login

# Navigate to frontend
cd inter-ai-frontend

# Initialize Netlify
netlify init

# Deploy
netlify deploy --prod
```

### Step 5: Configure Custom Domain (Optional)

1. In Netlify Dashboard → **Domain settings**
2. Click **"Add custom domain"**
3. Enter your domain (e.g., `coact-ai.com`)
4. Update DNS records as instructed
5. SSL certificate is automatically provisioned!

---

## 🌟 Netlify-Specific Features

### 1. Deploy Previews

Netlify automatically creates preview deployments for pull requests:
- Create a new branch
- Push changes
- Netlify creates a unique URL for testing
- Merge when ready

### 2. Environment Variables by Deploy Context

Update `netlify.toml` for different environments:

```toml
[context.production.environment]
  VITE_API_URL = "https://coactai-backend.onrender.com"

[context.deploy-preview.environment]
  VITE_API_URL = "https://coactai-backend-dev.onrender.com"
```

### 3. Form Handling (Bonus Feature)

If you add contact forms, Netlify handles submissions automatically:

```html
<form name="contact" method="POST" data-netlify="true">
  <input type="text" name="name" />
  <input type="email" name="email" />
  <button type="submit">Send</button>
</form>
```

### 4. Analytics (Free Tier)

Enable Netlify Analytics for visitor insights:
- Go to **Analytics** tab in dashboard
- Click **"Enable Analytics"**
- Get real-time traffic data

---

## ✅ Part 3: Verification

### Test Complete Flow

1. Visit your Netlify URL: `https://your-app.netlify.app`
2. Open browser DevTools → Network tab
3. Sign up and login
4. Start a practice session
5. Verify API calls go to Render backend
6. Check reports generate correctly

### Check Logs

**Backend Logs (Render)**:
- Render Dashboard → Your Service → **Logs**

**Frontend Logs (Netlify)**:
- Netlify Dashboard → Your Site → **Deploys** → Click deployment → **Deploy Log**

**Function Logs** (if using Netlify Functions):
- Netlify Dashboard → **Functions** → **Logs**

---

## 🚀 Part 4: Continuous Deployment

### Automatic Deployments

1. Push to `main` branch → Production deployment
2. Create PR → Preview deployment
3. Merge PR → Production update

```bash
# Example workflow
git checkout -b feature/new-ui
# Make changes...
git add .
git commit -m "Add new UI component"
git push origin feature/new-ui
# Create PR on GitHub
# Netlify creates preview URL automatically!
```

### Branch Deploys

Configure specific branches for staging:

In `netlify.toml`:
```toml
[context.staging]
  command = "npm run build:staging"
  base = "inter-ai-frontend"

[context.staging.environment]
  VITE_API_URL = "https://coactai-staging.onrender.com"
```

---

## 💰 Cost Breakdown (Free Tier)

| Service | Free Tier | Notes |
|---------|-----------|-------|
| **Netlify** | 100GB bandwidth/month, 300 build minutes | Generous free tier |
| **Render** | 750 hours/month | Backend may sleep after inactivity |
| **Supabase** | 500MB database | Perfect for MVP |
| **Azure OpenAI** | Pay-per-use | ~$0.15 per 1K tokens |

**Total**: $0/month + Azure usage

---

## 🎯 Netlify vs Vercel Comparison

| Feature | Netlify | Vercel |
|---------|---------|--------|
| **Build Minutes** | 300/month | Unlimited |
| **Bandwidth** | 100GB | 100GB |
| **Deploy Previews** | ✅ Yes | ✅ Yes |
| **Forms** | ✅ Built-in | ❌ No |
| **Analytics** | ✅ Free | 💰 Paid |
| **Functions** | ✅ Yes | ✅ Yes (better) |
| **Custom Headers** | ✅ Via netlify.toml | ✅ Via vercel.json |

**Recommendation**: 
- **Netlify**: Better for content-heavy sites with forms
- **Vercel**: Better for Next.js apps and serverless functions

---

## 🔧 Troubleshooting

### Build Fails: "Module not found"

**Issue**: Missing dependencies

**Fix**:
```bash
# In inter-ai-frontend directory
npm install
# Commit package-lock.json
git add package-lock.json
git commit -m "Update dependencies"
git push
```

### Environment Variables Not Loading

**Issue**: `undefined` API URL

**Fix**:
1. Ensure variables start with `VITE_` prefix
2. Re-deploy after adding env vars (Settings → Deploys → Trigger deploy)
3. Check build log shows env vars loaded

### 404 on Page Refresh

**Issue**: React Router routes return 404

**Fix**: Ensure `netlify.toml` has redirect rule:
```toml
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### CORS Errors

**Issue**: Backend rejecting frontend requests

**Fix**:
1. Add Netlify URL to backend CORS origins
2. Include both `.netlify.app` and custom domain
3. Redeploy backend on Render

---

## 🎯 Production Checklist

- [ ] Environment variables configured
- [ ] Custom domain added
- [ ] SSL certificate active
- [ ] CORS origins updated
- [ ] Deploy previews working
- [ ] Error tracking enabled (Sentry)
- [ ] Analytics configured
- [ ] Backup strategy for database

---

## 📚 Additional Resources

- [Netlify Documentation](https://docs.netlify.com/)
- [Netlify CLI Reference](https://cli.netlify.com/)
- [Render Documentation](https://render.com/docs)
- [Vite Deployment Guide](https://vitejs.dev/guide/static-deploy.html#netlify)

---

**Deployment Date**: February 2026  
**Version**: 1.0  
**Platform**: Netlify + Render + Supabase
