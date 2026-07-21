# Deploying the AI Assistant Console

This folder contains everything you need:

```
deploy/
├── index.html          ← the frontend (what users see)
├── api/
│   └── assistant.js     ← serverless function that calls Claude (holds the API key)
├── package.json
└── DEPLOYMENT.md         ← this file
```

## Why you need a backend at all

The original single-file version called `api.anthropic.com` directly from the
browser. That only works inside the Claude.ai artifact preview. Anywhere else,
it breaks for two reasons:

1. **Security** — any API key placed in frontend JavaScript is visible to
   anyone who opens dev tools or views source. Within hours of a public
   deploy, a leaked key gets scraped and used by someone else on your bill.
2. **CORS** — the Anthropic API is not meant to be called directly from
   arbitrary browser pages.

The fix: `index.html` now calls `/api/assistant` (your own server), and
`api/assistant.js` is the only place the real Anthropic API key lives.

---

## Option A — Deploy to Vercel (recommended, free, ~5 minutes)

### 1. Get an Anthropic API key
- Go to [console.anthropic.com](https://console.anthropic.com)
- Sign in → **API Keys** → **Create Key**
- Copy the key (starts with `sk-ant-...`) — you won't see it again

### 2. Create a GitHub repo
```bash
cd deploy
git init
git add .
git commit -m "AI Assistant Console"
```
Create a new empty repo on [github.com/new](https://github.com/new), then:
```bash
git remote add origin https://github.com/YOUR_USERNAME/ai-assistant-console.git
git branch -M main
git push -u origin main
```

### 3. Import into Vercel
- Go to [vercel.com](https://vercel.com) → sign up/log in (GitHub login is easiest)
- **Add New… → Project** → select your repo → **Import**
- Leave all build settings at their defaults (no build command needed — it's static HTML + one function)

### 4. Add your API key as an environment variable
Before clicking Deploy (or after, in project settings):
- **Settings → Environment Variables**
- Name: `ANTHROPIC_API_KEY`
- Value: your `sk-ant-...` key
- Environment: Production (and Preview if you want)
- Save

### 5. Deploy
- Click **Deploy**
- Vercel gives you a live URL like `https://ai-assistant-console.vercel.app`
- Open it — the console should work end to end

### 6. Redeploy after changes
Any future `git push` to `main` auto-redeploys. To change the API key later,
update it in Settings → Environment Variables, then **Redeploy** (env var
changes need a redeploy to take effect).

---

## Option B — Netlify (equally easy, same idea)

1. Push the same `deploy/` folder to a GitHub repo (steps above).
2. [netlify.com](https://netlify.com) → **Add new site → Import an existing project**.
3. Netlify needs the function in a specific folder — rename `api/` to
   `netlify/functions/` and change the frontend fetch URL from
   `/api/assistant` to `/.netlify/functions/assistant`.
4. **Site settings → Environment variables** → add `ANTHROPIC_API_KEY`.
5. Deploy.

Use Vercel unless you have a specific reason to prefer Netlify — the folder
structure in this package is already set up for it.

---

## Option C — Run it locally first (test before deploying)

```bash
npm install -g vercel
cd deploy
export ANTHROPIC_API_KEY=sk-ant-your-key-here   # macOS/Linux
# set ANTHROPIC_API_KEY=sk-ant-your-key-here    # Windows (cmd)
vercel dev
```
This runs the exact same setup (static file + serverless function) on
`http://localhost:3000`, so you can test before pushing anywhere.

---

## Option D — No backend, just showing it off (static-only hosting)

If you only need to *demo* the interface (no live API calls) — e.g. for
GitHub Pages, which can't run serverless functions — you can host
`index.html` alone as a static page. The UI, tabs, variant chips, and prompt
inspector all work with zero backend. Clicking "Run assistant" will simply
show a request-failed message, since there's no `/api/assistant` to answer
it. This is fine for a portfolio link or grading walkthrough where you'll
narrate the prompts rather than run them live.

```bash
# GitHub Pages, static-only version
git checkout -b gh-pages
# keep only index.html, remove /api and the fetch call, or accept the error state
git push origin gh-pages
```
Then enable Pages in the repo's **Settings → Pages** for the `gh-pages` branch.

---

## Security checklist before going live

- [ ] `ANTHROPIC_API_KEY` is set as an environment variable, never hardcoded in `index.html` or `assistant.js`
- [ ] `.env` files (if you use one locally) are in `.gitignore` and never committed
- [ ] Consider adding a rate limit or usage cap in the Anthropic Console so a spike in traffic can't run up an unexpected bill
- [ ] If this is for a class submission, don't commit your real API key to a public repo — private repo, or share the key with instructors separately

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| "Request failed: Server is missing ANTHROPIC_API_KEY" | Env var not set, or you deployed before adding it — redeploy after setting it |
| 404 on `/api/assistant` | Function not detected — confirm the file is at `api/assistant.js` relative to project root |
| Works locally, not on Vercel | Env var only added to one environment (e.g. Preview but not Production) |
| CORS error in browser console | You reverted to calling `api.anthropic.com` directly from `index.html` instead of `/api/assistant` |
