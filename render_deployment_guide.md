# 🚀 Render Deployment Guide — MyCoinwise Backend

## Fixes Applied

### 1. JWT Secret Mismatch (CRITICAL)
`server.js` used `'super_secret'` as the JWT fallback, but `middleware/auth.js` used `'super_secret_jwt_key_mycoinwise_12345'`. Tokens signed by the server could **never** be verified by the middleware → all authenticated requests would fail silently.

**Fix:** All 3 `jwt.sign()` calls in `server.js` now use the same fallback as `auth.js`.

### 2. CORS Locked to Localhost (CRITICAL)
CORS was hardcoded to `localhost:5173/5174` only. Your deployed frontend would get blocked.

**Fix:** Added `FRONTEND_URL` env var support. Set it on Render to your frontend's deployed URL.

### 3. dotenv Loaded Too Late
`dotenv.config()` was called **after** `require('./db')`, so `MONGO_URI` might not be available when the DB connects (it worked by luck because `db.js` also calls dotenv internally).

**Fix:** Moved `dotenv.config()` to the very first lines of `server.js`.

### 4. Missing .gitignore
No `.gitignore` existed — `.env` file with DB credentials, JWT secrets, and API keys could get committed.

**Fix:** Created `.gitignore` covering `.env`, `node_modules/`, `dist/`, logs, and OS files.

### 5. Wrong `main` + No Engine in package.json
`main` pointed to `index.js` (doesn't exist). No Node engine specified for Render.

**Fix:** Set `main: "server.js"` and `engines: { node: ">=18.0.0" }`.

---

## Render Deployment Steps

### Step 1: Push to GitHub
```bash
cd /Users/sarthakmathapati/Downloads/sem3/Learnings/9-BudgetTracker
git add -A
git commit -m "fix: prepare backend for Render deployment"
git push
```

### Step 2: Create Render Web Service
1. Go to [render.com](https://render.com) → **New** → **Web Service**
2. Connect your GitHub repo
3. Configure:

| Setting | Value |
|---------|-------|
| **Name** | `mycoinwise-api` |
| **Root Directory** | `backend` |
| **Runtime** | Node |
| **Build Command** | `npm install` |
| **Start Command** | `npm start` |
| **Plan** | Free |

### Step 3: Set Environment Variables on Render

> [!IMPORTANT]
> These must be set in Render Dashboard → Your Service → **Environment**

| Variable | Value |
|----------|-------|
| `MONGO_URI` | `mongodb+srv://skenscode_db_user:password_skens_tool@9-the-teenagers-budgett.843iayv.mongodb.net/MyCoinwise?retryWrites=true&w=majority&appName=9-The-teenagers-BudgetTool` |
| `JWT_SECRET` | `super_secret_jwt_key_mycoinwise_12345` |
| `GEMINI_API_KEY` | `AIzaSyCakwcVyZD4SAnEhGX-f6-_q_Us7YyqCbI` |
| `FRONTEND_URL` | Your frontend URL (e.g. `https://your-app.vercel.app`) |
| `PORT` | `5001` (or let Render auto-assign) |

> [!CAUTION]
> Never commit `.env` to git. The values above should ONLY exist as Render environment variables.

### Step 4: Update Frontend API URL
Once Render gives you a URL like `https://mycoinwise-api.onrender.com`, set the frontend env var:

```bash
# In frontend/.env or Vercel env vars:
VITE_API_URL=https://mycoinwise-api.onrender.com/api
```

### Step 5: MongoDB Atlas — Allow Render IP
1. Go to [MongoDB Atlas](https://cloud.mongodb.com)
2. **Network Access** → **Add IP Address**
3. Add `0.0.0.0/0` (allow from anywhere) — required for Render's dynamic IPs

> [!WARNING]
> If you skip this step, Render will get `ECONNREFUSED` errors trying to reach your database.

---

## Verification Checklist
After deploying, test these endpoints:

```bash
# Health check
curl https://mycoinwise-api.onrender.com/api/health

# Register
curl -X POST https://mycoinwise-api.onrender.com/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"Test","email":"test@test.com","password":"test123456"}'
```

Both should return JSON responses (not errors).
