# A Better Place Consulting Dashboard

The live business dashboard for A Better Place Consulting. One page, eight tabs, real API data, and a launch calendar the whole team can bookmark.

## What this is

A single-file dashboard (`index.html`) that pulls live numbers from YouTube, Instagram, HubSpot, Trello, and Simplero, tracks every program and event, holds the content calendar and team checklists, and stores everything you type in the browser. It deploys free on GitHub Pages.

## File list

- `index.html` — the whole dashboard, self-contained
- `config.js` — your API keys. Local only. Listed in `.gitignore`, never pushed to GitHub
- `.gitignore` — keeps `config.js` and system files out of Git
- `README.md` — this file

## First-time setup

1. Open `index.html` in Chrome (double-click it, or drag it into a Chrome tab).
2. Go to the **Settings** tab.
3. Add your API keys in each section and click **Save**. Keys save to the browser and stay in `config.js` locally.
4. Set your goals in the **Goals** section.
5. Click **Sync All** in the top nav. Every connected source pulls at once and stamps a last-synced time.

## Setup guide for each integration

### YouTube (works directly in the browser)
1. Go to console.cloud.google.com and create or pick a project.
2. Enable **YouTube Data API v3**.
3. Go to Credentials and create an **API Key**.
4. Find your **Channel ID** in YouTube Studio under Settings, then Channel, then Advanced settings.
5. Paste both into Settings, then YouTube. Click Test Connection.

### Instagram (Meta Graph API)
1. Open **Meta Business Suite**.
2. Go to Settings, then System Users.
3. Generate a token with the **instagram_basic** scope.
4. Get your Instagram **Business Account User ID** from the same area.
5. Paste both into Settings, then Instagram.
6. The token expires every 60 days. Set a calendar reminder to refresh it. If Instagram shows an error, the token has likely expired.

### HubSpot (Private App)
1. In HubSpot go to Settings, then Integrations, then Private Apps.
2. Create an app with the **crm.objects.contacts.read** and **crm.objects.deals.read** scopes.
3. Copy the token. It starts with `pat-na1-`.
4. Paste it into Settings, then HubSpot. Click Test Connection.

### Trello (supports browser calls)
1. Go to trello.com/app-key.
2. Copy the **API Key**.
3. Use the link on that page to generate a **Token**.
4. Get the **Board ID** from the board URL, or add `.json` to the board URL to read it.
5. Paste all three into Settings, then Trello. Priority color coding reads card labels: red or orange is high, yellow is medium, everything else is low.

### Simplero (needs a CORS proxy)
1. In the Simplero admin go to Account Settings, then the API tab.
2. Copy your **API Key** and the **List IDs** for Alchemy and DEFY.
3. Paste them into Settings, then Simplero.

Simplero blocks direct browser calls with CORS. See the next section for the one-time proxy Gino sets up.

### Q4 Pipeline (Google Sheets, private)
The Q4 Pipeline tab shows the Q4 Revenue Reactivation Tracker: the ranked leads, what to say to each one, who owns it, and a Status you can change from the tab. Changes save straight back to the Sheet.

This repo and the live page are public, so **no lead data ever goes into this repo**. The tab signs in with Google and reads the private Sheet only while the page is open. Nothing about the leads is saved in the browser. Anyone who can't open the Sheet sees a sign-in screen and nothing else.

One-time setup, about 10 minutes:
1. Go to console.cloud.google.com and open the project you use for YouTube, or create one.
2. APIs & Services, then Library. Search for **Google Sheets API** and click Enable.
3. APIs & Services, then **OAuth consent screen**, then Get started. App name: ABP Dashboard. Audience: **Internal** if everyone signs in with a bunnyyoung.com Google account. Otherwise **External**, then add Bunny's and the EA's Google emails as test users on the Audience tab.
4. Clients tab, then Create client. Type: **Web application**. Under Authorized JavaScript origins add `https://thebunnyyoung.github.io`. Create it and copy the **Client ID**. No secret is needed.
5. In the dashboard, Settings, then Q4 Pipeline: paste the Client ID and the tracker's link, click Save, then Test Connection and sign in. If Google says the app isn't verified, click Continue. That's normal for a private app.
6. Share the tracker Sheet with the EA as an Editor. Then click **Copy setup link for your EA** and send it. Opening that link fills in the EA's settings in one step.

What the tab writes: only the Status, Date sent, Reply / outcome and Notes columns of the Priority Leads tab, one cell at a time. Before every save it checks that the row still holds the same person, so sorting the Sheet can't put an update on the wrong row.

### Podcast
- Apple Podcasts does not expose subscriber counts.
- Use the **Buzzsprout** or **Libsyn** API for reach, or enter a **Manual Count** in Settings.
- Pick the platform, add the API key and podcast ID, and set a subscriber goal. If the API cannot return a subscriber number, the manual count is used.

## CORS proxy for Simplero (Netlify Functions)

Simplero has no CORS headers, so a browser cannot call it directly. Stand up a tiny free proxy once.

1. Create a folder with this file at `netlify/functions/simplero.js`:

```js
exports.handler = async (event) => {
  const { listId, status } = event.queryStringParameters;
  const key = process.env.SIMPLERO_API_KEY;
  const url = `https://simplero.com/api/v1/lists/${listId}/subscriptions?api_key=${key}&status=${status || 'active'}`;
  const res = await fetch(url);
  const body = await res.text();
  return {
    statusCode: 200,
    headers: { 'Access-Control-Allow-Origin': '*', 'Content-Type': 'application/json' },
    body,
  };
};
```

2. Add a `netlify.toml`:

```toml
[build]
  functions = "netlify/functions"
```

3. Push to Netlify, set the `SIMPLERO_API_KEY` environment variable in the Netlify dashboard, and deploy.
4. In `index.html`, point the Simplero fetch at your function URL, for example `https://your-site.netlify.app/.netlify/functions/simplero?listId=LIST_ID`. The API key then lives in Netlify, not in the browser.

## GitHub deployment

### First push
From inside the `abp-dashboard` folder:

```bash
git init
git add index.html README.md .gitignore
git commit -m "Initial ABP dashboard"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/abp-dashboard.git
git push -u origin main
```

Then turn on Pages: your repo on GitHub, then Settings, then Pages. Source: Deploy from a branch. Branch: main. Folder: / (root). Save. Two minutes later the dashboard is live at:

`https://YOUR_USERNAME.github.io/abp-dashboard`

### Weekly update

```bash
git add index.html README.md
git commit -m "Weekly update"
git push
```

Pages redeploys on its own within a couple of minutes.

## Never push config.js

`config.js` holds your API keys and is listed in `.gitignore`. Never add it to Git. Only ever run `git add index.html README.md .gitignore`. If you ever see `config.js` in `git status` under staged changes, remove it with `git reset config.js` before you commit.

## Gino's weekly maintenance routine

1. Open the live dashboard.
2. Click **Sync All**.
3. Update the manual numbers: DEFY members, FIT calls this week, program enrollments, amplifier activity.
4. Screenshot the **Overview** tab and send it to Bunny in Slack.

That is the whole loop. Live numbers where the API allows, a fast manual pass everywhere else, one screenshot to Bunny.

## Never put client data in this repo

The repo and the GitHub Pages site are public. Client names, emails, notes and numbers stay in the Google Sheet. The Q4 Pipeline tab reads them live after a Google sign-in. Do not paste lead lists into `index.html`, the README, or any file you add here.
