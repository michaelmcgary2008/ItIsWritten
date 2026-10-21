# It Is Written

A scripture search web app. Search a topic and get five relevant Bible verses with
context and insight, a model prayer for each verse, and a four-round memorization tool
with optional voice input.

## Files

- `it-is-written.html` — the complete app as handed off: one self-contained file with
  inline CSS and JavaScript, no build step.
- `developer-brief.md` — the handoff notes: how the app works, what must change before
  deployment, hosting options, and the PWA / native roadmap.

## Status

The app as committed calls the Claude API through the Anthropic artifact runtime
(`claude.use('sample')`), which only works inside that environment. Before hosting it,
the two API calls need to go through a small backend proxy that holds the API key, as
described in `developer-brief.md`.
