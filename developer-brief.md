# It Is Written — Developer Handoff

## Overview
A scripture search web app. Users search a topic, receive 5 relevant Bible verses with contextual insight, a model prayer, and an interactive memorization tool. The goal is to eventually ship this as a mobile app (iOS + Android), but the first milestone is a hosted web app.

---

## What's Included
- `it-is-written.html` — the complete, self-contained app. Single file, no dependencies to install, no build step required.

---

## How It Works
The app is a single HTML file with inline CSS and JavaScript. It calls the **Anthropic Claude API** (`/v1/messages`) to:
1. Return 5 Bible verses for a given topic (with context and application notes)
2. Generate a model prayer for any verse
3. The memorization tool is fully client-side — no API call needed

### API Details
- Endpoint: `https://api.anthropic.com/v1/messages`
- Model: `claude-sonnet-4-6`
- The current file is wired to run inside Anthropic's artifact environment, which handles auth automatically. **Before deploying, the API calls need to be updated to pass a real API key.**

### What the developer needs to change
In `it-is-written.html`, find the two `claude.use('sample')` calls (around lines 650 and 750) and replace the artifact-style API pattern with direct fetch calls. Example replacement:

**Current (artifact environment only):**
```javascript
const sample = await claude.use('sample');
const result = await sample(prompt, { modelTier: 'default' });
const text = result.text;
```

**Replace with:**
```javascript
const response = await fetch('https://api.anthropic.com/v1/messages', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-api-key': 'YOUR_API_KEY_HERE',
    'anthropic-version': '2023-06-01'
  },
  body: JSON.stringify({
    model: 'claude-sonnet-4-6',
    max_tokens: 1000,
    messages: [{ role: 'user', content: prompt }]
  })
});
const data = await response.json();
const text = data.content[0].text;
```

> ⚠️ **Important:** Do not expose the API key in client-side code in production. The API key should live on a small backend (a simple Vercel Edge Function, Cloudflare Worker, or Express endpoint works fine) that proxies requests. The frontend calls your backend; your backend calls Anthropic.

---

## Features
| Feature | How it works |
|---|---|
| Topic search | User types a topic or taps a chip → Claude returns 5 verses as JSON |
| Scripture cards | Verse, translation badge, one-line relevance note |
| Context & Insight | Toggle panel: original audience, common misreading (if any), faithful application |
| Pray This | Generates a personalized prayer woven from the verse text |
| Memorize | 4-round progressive fill-in-the-blank. 25% → 50% → 75% → 100% blanked. Real-time per-keystroke validation with ding sound and auto-advance |
| Voice input | Web Speech API (`continuous: true`). One tap starts a session that listens through the whole round. Works in standalone browser — blocked inside iframes |
| Light / Dark mode | Manual toggle, defaults to dark |

---

## Recommended Hosting
Any static host works for the HTML file itself. For the API proxy:
- **Vercel** — easiest. Deploy the HTML as a static site, add one serverless function as the proxy endpoint.
- **Netlify** — same pattern, Netlify Functions.
- **Cloudflare Pages + Workers** — fastest globally, free tier is generous.

---

## Domain & PWA
Once hosted, the app can be made installable on phones as a **Progressive Web App (PWA)** with about 30 lines of additional code (a `manifest.json` and a minimal service worker). This gives users an icon on their home screen and offline caching — without going through the App Store. Recommended as a first step before native iOS/Android.

---

## Future: Native App
When ready to go native:
- The UI logic can be ported to **React Native** (recommended — large community, ships to both iOS and Android)
- Or wrapped with **Capacitor** (drop the existing HTML/JS directly into a native shell — lower effort, slightly less native feel)
- Voice input in a native app uses the OS speech APIs directly, no iframe restrictions

---

## Questions
The person who commissioned this is available to clarify anything. The design, feature set, and copy are all final — this is a handoff for hosting and productionization only.
