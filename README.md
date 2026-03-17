[README.md](https://github.com/user-attachments/files/26069702/README.md)
# StoreScore — Shopify App Deployment Guide

## What this is
A real Shopify embedded app with:
- Express.js backend (handles OAuth)
- Shopify App Bridge (embeds inside Shopify Admin)
- All APIs proxied server-side (no CORS issues)

## Files
```
storescore-app/
├── server.js          ← Backend: OAuth + API proxy
├── package.json       ← Dependencies
├── .env.example       ← Environment variables template
└── public/
    └── index.html     ← The app UI (embedded in Shopify)
```

---

## STEP 1 — Deploy backend to Railway (free, 5 minutes)

Railway is the easiest free Node.js host.

1. Go to **railway.app** → Sign up with GitHub
2. Click **New Project → Deploy from GitHub repo**
3. Upload these files to a GitHub repo first:
   - Create github.com/YOURNAME/storescore
   - Upload all files maintaining the folder structure
4. Railway auto-detects Node.js and runs `npm start`
5. Go to **Settings → Domains → Generate Domain**
6. Copy your URL: `https://storescore-production.up.railway.app`

**Set environment variables in Railway:**
- Go to Variables tab
- Add each variable from `.env.example`
- Set `HOST` = your Railway URL

---

## STEP 2 — Update Shopify Partners

Go back to your Shopify Partners dashboard → Your app → App setup:

**App URL:**
```
https://storescore-production.up.railway.app
```

**Allowed redirect URLs:**
```
https://storescore-production.up.railway.app/auth/callback
```

**Required scopes:**
```
read_themes,read_script_tags
```

Save → Create version → Release

---

## STEP 3 — Install the app on your store

The install URL is:
```
https://storescore-production.up.railway.app/auth?shop=YOURSTORE.myshopify.com
```

Replace `YOURSTORE` with your actual store name.

1. Open that URL in your browser
2. Shopify shows the permission screen
3. Merchant clicks "Install"
4. App opens inside Shopify Admin ✅

---

## STEP 4 — Submit to Shopify App Store (optional)

Once working:
1. Go to Shopify Partners → App listing
2. Fill in name, description, screenshots
3. Submit for review (takes ~1-2 weeks)
4. Once approved, merchants can find and install via the App Store

---

## How the OAuth flow works

```
Merchant clicks Install
        ↓
GET /auth?shop=store.myshopify.com
        ↓
Server redirects to Shopify permission screen
        ↓
Merchant approves
        ↓
GET /auth/callback?code=xxx&shop=xxx&hmac=xxx
        ↓
Server validates HMAC (security check)
        ↓
Server exchanges code for permanent access_token
        ↓
Redirect to /app?shop=xxx&token=xxx
        ↓
App loads inside Shopify Admin ✅
```

---

## Alternative free hosts

| Host | Free tier | Notes |
|------|-----------|-------|
| Railway | $5 credit/month | Easiest, recommended |
| Render | 750 hrs/month | Spins down after 15min idle |
| Fly.io | 3 shared VMs | More complex setup |
| Cyclic | Always free | Good for small apps |

---

## Troubleshooting

**"HMAC validation failed"**
→ Your `SHOPIFY_API_SECRET` doesn't match. Double-check in Partners dashboard.

**"Redirect URI not allowed"**
→ The exact callback URL must be in your Partners app settings.

**App loads but shows blank**
→ Check Railway logs. Usually a missing env variable.

**"shop parameter missing"**
→ You're visiting the root URL directly. Always install via the /auth?shop= URL.
