# Merchant Menu Optimizer

An AI-powered tool that scores and rewrites restaurant menu items using the Claude API.

## What it does

- Paste menu items as CSV (name, description, category, price)
- Claude analyzes each item and returns:
  - A health score (1–10)
  - A rewritten, appetizing description
  - Photo flags, category suggestions, allergen notes
- Export results as CSV or JSON

---

## Setup & hosting

### 1. Upload to GitHub

1. Create a new GitHub repository (public or private)
2. Upload `index.html` to the root of the repo
3. Upload this `README.md`

### 2. Enable GitHub Pages

1. Go to your repo → **Settings** → **Pages**
2. Under "Source", select **Deploy from a branch**
3. Branch: `main`, folder: `/ (root)`
4. Click **Save**

Your app will be live at:
```
https://YOUR_USERNAME.github.io/YOUR_REPO_NAME
```

---

## API key options

### Option A — Direct key (quick demo only)

Enter your Anthropic API key directly in the app's input field.

⚠️ **Only use this for local testing or private demos.** Never commit your API key to a public repo.

Get a key at: https://console.anthropic.com

### Option B — Cloudflare Worker proxy (recommended for production)

This hides your API key server-side so it's never exposed in the browser.

#### Step 1: Create the Worker

1. Go to https://workers.cloudflare.com and sign up (free)
2. Click **Create a Worker**
3. Replace the default code with:

```javascript
export default {
  async fetch(request, env) {
    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': '*',
          'Access-Control-Allow-Methods': 'POST, OPTIONS',
          'Access-Control-Allow-Headers': 'Content-Type',
        }
      });
    }

    const body = await request.json();

    const resp = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': env.ANTHROPIC_API_KEY,
        'anthropic-version': '2023-06-01',
      },
      body: JSON.stringify(body),
    });

    const data = await resp.json();

    return new Response(JSON.stringify(data), {
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
      }
    });
  }
};
```

4. Click **Save and Deploy**

#### Step 2: Add your API key as a secret

1. In the Worker dashboard → **Settings** → **Variables**
2. Under "Secret Variables", click **Add variable**
3. Name: `ANTHROPIC_API_KEY`, Value: your key from console.anthropic.com
4. Click **Save**

#### Step 3: Use your Worker URL in the app

Your Worker URL looks like:
```
https://menu-optimizer.YOUR_SUBDOMAIN.workers.dev
```

Paste this URL into the API configuration field in the app instead of your raw API key.

---

## CSV format

```
name,description,category,price
Spicy Chicken Sandwich,Chicken with sriracha mayo,Sandwiches,12.99
Veggie Bowl,,Bowls,10.50
```

- `name` — required
- `description` — optional (AI will note it's missing)
- `category` — optional
- `price` — optional

---

## Tech stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML/CSS/JS (no framework) |
| AI model | Claude claude-sonnet-4-6 via Anthropic API |
| Hosting | GitHub Pages (free) |
| Proxy | Cloudflare Workers (free tier: 100k req/day) |

---

## Local development

No build step needed. Just open `index.html` in your browser:

```bash
open index.html
# or
python3 -m http.server 8080
```

---

## Customizing the AI prompt

The system prompt is in `index.html` as the `SYSTEM_PROMPT` constant. Edit it to:
- Change the scoring rubric
- Add cuisine-specific guidance
- Enforce description length limits
- Add brand voice instructions

No backend changes needed — just edit the string and redeploy.
