# SME Growth Diagnostic — Claude Code Deployment Instructions

Paste the contents of this file directly into Claude Code as your first message.

---

## Your task

I have a single-file HTML webapp called `sme_growth_diagnostic.html` that I need you to deploy as a live public website. It is an AI-powered business diagnostic tool that calls the Anthropic API to generate personalised reports.

Please complete all of the following steps in order. Work through each one autonomously and let me know when you need any input from me (e.g. to confirm a login or paste an API key).

---

## Step 1 — Read the app file

Open and read `sme_growth_diagnostic.html` so you understand its structure before making any changes.

---

## Step 2 — Create a Netlify serverless proxy for the API key

The HTML file currently calls `https://api.anthropic.com/v1/messages` directly from the browser. This exposes the API key. Replace this with a secure serverless function.

### Create this folder structure:

```
/netlify/functions/claude-proxy.js
```

### Contents of `claude-proxy.js`:

```javascript
exports.handler = async (event) => {
  if (event.httpMethod !== 'POST') {
    return { statusCode: 405, body: 'Method not allowed' };
  }

  try {
    const body = JSON.parse(event.body);

    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': process.env.ANTHROPIC_API_KEY,
        'anthropic-version': '2023-06-01'
      },
      body: JSON.stringify(body)
    });

    const data = await response.json();

    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
      },
      body: JSON.stringify(data)
    };
  } catch (err) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: err.message })
    };
  }
};
```

### Create `netlify.toml` in the project root:

```toml
[build]
  publish = "."
  functions = "netlify/functions"

[[redirects]]
  from = "/api/claude"
  to = "/.netlify/functions/claude-proxy"
  status = 200
```

---

## Step 3 — Update the fetch call in the HTML file

In `sme_growth_diagnostic.html`, find the `fetch` call that points to `https://api.anthropic.com/v1/messages`.

Replace the URL with `/api/claude` and remove the `x-api-key` and `anthropic-version` headers from the client-side request (they are now handled server-side in the proxy).

The updated fetch should look like this:

```javascript
const response = await fetch('/api/claude', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1000,
    messages: [{ role: 'user', content: prompt }]
  })
});
```

---

## Step 4 — Create a `package.json`

Create a minimal `package.json` in the project root so Netlify recognises this as a Node project:

```json
{
  "name": "sme-growth-diagnostic",
  "version": "1.0.0",
  "description": "SME Growth Diagnostic by Traction Studio",
  "engines": {
    "node": "18"
  }
}
```

---

## Step 5 — Initialise a Git repository and push to GitHub

1. Initialise a git repo in this project folder: `git init`
2. Create a `.gitignore` with the following contents:
```
node_modules/
.env
.env.local
```
3. Stage all files: `git add .`
4. Commit: `git commit -m "Initial deploy: SME Growth Diagnostic"`
5. Create a new **private** repository on GitHub called `sme-growth-diagnostic`
6. Push to GitHub: `git remote add origin <repo-url>` then `git push -u origin main`

If you cannot create the GitHub repo automatically, pause here and tell me — I will create it manually and give you the URL to push to.

---

## Step 6 — Deploy to Netlify

1. Go to [netlify.com](https://netlify.com) and log in (or create a free account)
2. Click "Add new site" → "Import an existing project" → connect to GitHub
3. Select the `sme-growth-diagnostic` repository
4. Build settings should auto-detect from `netlify.toml` — confirm:
   - Build command: *(leave blank)*
   - Publish directory: `.`
   - Functions directory: `netlify/functions`
5. Click "Deploy site"

If you can automate this via the Netlify CLI instead, please do so:

```bash
npm install -g netlify-cli
netlify login
netlify init
netlify deploy --prod
```

---

## Step 7 — Add the Anthropic API key as an environment variable

Once the site is deployed:

1. In the Netlify dashboard, go to **Site settings → Environment variables**
2. Add a new variable:
   - Key: `ANTHROPIC_API_KEY`
   - Value: *(I will paste this — please prompt me for it)*
3. Trigger a redeploy after adding the variable

Do not proceed with this step until you have prompted me for the API key value.

---

## Step 8 — Set a custom subdomain (optional but recommended)

In the Netlify dashboard:
1. Go to **Site settings → Domain management**
2. Click "Edit site name"
3. Set it to: `traction-diagnostic` (this gives the URL `traction-diagnostic.netlify.app`)

If I have a custom domain I want to use instead, I will tell you at this point.

---

## Step 9 — Smoke test the live site

Once deployed, please:

1. Open the live URL in the browser
2. Confirm the intro screen loads correctly
3. Walk through 3–4 questions to confirm the quiz flow works
4. Submit a test email and confirm the loading screen appears
5. Confirm the results page renders with scores and report content
6. Check the browser console for any errors

Report back with the live URL and whether all tests passed.

---

## Step 10 — Summary report

When complete, give me:

- The live URL
- A confirmation that the API proxy is working and the key is not exposed client-side
- Any issues encountered and how they were resolved
- Suggested next steps (e.g. connecting a custom domain, setting up email capture via Netlify Forms, or adding analytics)

---

## Files in this project

```
sme_growth_diagnostic.html     ← main app (already built)
netlify/functions/
  claude-proxy.js               ← you will create this
netlify.toml                    ← you will create this
package.json                    ← you will create this
.gitignore                      ← you will create this
DEPLOY.md                       ← this file
```

---

## Notes for Claude Code

- This is a static single-page HTML app — no build step, no bundler, no framework
- The only server-side component is the Netlify function acting as an API proxy
- Node 18+ is required for native `fetch` support in the serverless function
- If Netlify CLI is available, prefer it over manual dashboard steps
- If any step fails, diagnose the error, fix it, and continue — do not stop and wait unless you genuinely need my input
- The Anthropic API key will be provided by me when prompted — never hardcode it or commit it to git
