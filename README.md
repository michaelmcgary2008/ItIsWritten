# It Is Written

A scripture search web app. Search a topic and get five relevant Bible verses with
context and insight, a model prayer for each verse, and a four-round memorization tool
with optional voice input.

Not a developer? To edit the site and publish a change, see [PUBLISHING.md](PUBLISHING.md).

## Files

- `index.html` — the complete front end: one self-contained file with inline CSS and
  JavaScript, no build step. It calls `/api/claude` for the model requests (verse
  search, context, and prayer).
- `api/claude.js` — the serverless proxy (Vercel Node.js function). It holds the
  Anthropic API key and builds the prompts server-side. It only accepts
  `{"action":"search","topic"}` (topic → the five cards; it also accepts an optional
  `exclude` list of up to 30 references already shown, which the "More verses" button
  sends to get five different verses, so one search shows at most 25 verses),
  `{"action":"context","topic","verses"}` (topic + the five verses → audience, caution
  and application, requested by the page in the background after the cards render) or
  `{"action":"prayer","ref","verse","topic"}`, with length caps (topic 80, ref 40,
  verse 800 characters) and per-action output caps.
  The model is `claude-sonnet-5-5`, set at the top of this file.
- `dev.mjs` — a zero-dependency local server (Node 20+) that serves `index.html` and
  mounts `api/claude.js`, so you do not need the Vercel CLI for local development.
- `developer-brief.md` — the handoff notes: how the app works, hosting options, and the
  PWA / native roadmap.
- `.env.example` — the environment variable the app needs (`ANTHROPIC_API_KEY`).

## Behaviour notes

- The theme follows the device setting on first visit and remembers a manual choice
  (`localStorage` key `iiw-theme`). The `theme-color` meta tag follows the theme.
- A search is reflected in the URL as `?q=topic`, so a results page can be shared,
  refreshed, and navigated with Back and Forward.
- Screen-reader support: a live status region announces searches, results, prayers,
  memorize rounds and errors; cards are articles labelled by their reference; the card
  buttons expose expanded state; a skip link appears on keyboard focus.
- Voice input in Memorize is disabled (the Speak button is gated off in `index.html`).

## Run locally

```sh
cp .env.example .env.local   # then put your key in .env.local
npm run dev
```

Open http://localhost:3000. `PORT=3001 npm run dev` picks another port.

## Deploy to Vercel

Production now deploys automatically. Vercel's Git integration does not support the
self-hosted Gitea repo directly, so Gitea push-mirrors every commit on `main` to a
GitHub relay repo, `github.com/michaelmcgary2008/ItIsWritten`, and Vercel's GitHub
integration deploys production from it. The relay is public on purpose: Vercel blocks
deployments on private repos when the commit author is not a Vercel team member, and
commits made on Gitea never are. The source is public on Gitea anyway, and the repo
holds no secrets. Nobody should push to the GitHub repo directly, because Gitea
overwrites it.

The app runs as the Vercel project `it-is-written` in the Waters Church team, at
https://it-is-written-waters-church.vercel.app.

**Manual fallback.** If the automatic deploy is unavailable, deploy from a local
checkout with the Vercel CLI (`npx vercel`, signed in to the team):

```sh
npx vercel link --yes --project it-is-written --scope waters-church   # once per checkout
npx vercel deploy --scope waters-church          # preview deployment (login-protected)
npx vercel deploy --prod --scope waters-church   # production
```

`ANTHROPIC_API_KEY` is already set in the project as a sensitive environment variable
for Production and Preview. To rotate it:

```sh
npx vercel env rm ANTHROPIC_API_KEY production --scope waters-church
npx vercel env add ANTHROPIC_API_KEY production --sensitive --scope waters-church < key.txt
```

(same for `preview`), then redeploy.

Deployment Protection (Vercel Authentication) is set to "Only Preview Deployments" in
the project settings, so the production address is public and preview deployments
require a Vercel login (or a share link from the dashboard).

## Secrets

The API key must never be committed or put in client code. It is read only from the
server environment (`.env.local` locally, Vercel environment variables when deployed).
`.env.local` (and all other `.env*` files except `.env.example`) are git-ignored.

## Protecting spend

Anyone who can load the site can call `/api/claude`, so the spend is protected in three
layers:

1. **Anthropic Console.** Create a dedicated workspace for this app, set a monthly spend
   limit on it, and create the API key inside that workspace. Set a usage alert below
   the limit.
2. **A single-purpose proxy.** The proxy only builds its three fixed prompts, caps input
   and output size, and its system prompt tells the model to return an empty result for
   abusive or off-topic input (the UI shows "No verses found" in that case).
3. **Rate limit per IP.** A Vercel Firewall rule named "Rate limit claude proxy" is
   published on the project: `/api/claude` is limited to 60 requests per minute per IP,
   then denied for the rest of that minute. One person cannot reach that, but a crowd
   on one shared Wi-Fi network (one public IP) could, so raise it if that happens. It
   was created with:

   ```sh
   npx vercel firewall rules add "Rate limit claude proxy" \
     --condition '{"type":"path","op":"eq","value":"/api/claude"}' \
     --action rate_limit --rate-limit-window 60 --rate-limit-requests 60 \
     --rate-limit-keys ip --rate-limit-action deny --scope waters-church --yes
   npx vercel firewall publish --scope waters-church --yes
   ```
