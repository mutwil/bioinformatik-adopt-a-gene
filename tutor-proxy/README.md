# Module 4 AI tutor — Cloudflare Worker proxy

A tiny serverless proxy that holds your Anthropic API key and forwards student
questions to Claude. The static lesson page (GitHub Pages) calls this Worker, never
Anthropic directly — so the key is never exposed in the browser.

- **Model:** Claude **Haiku 4.5** for hints (cheap/fast), **Sonnet 4.6** when the student
  ticks "deeper answer". System prompt is Socratic (hints, not finished answers).
- **Cost:** roughly a fraction of a cent per question; a whole class for a module is typically **well under $10**.

## Deploy (one time, ~5 min)

You need a free [Cloudflare](https://dash.cloudflare.com/sign-up) account and the
`wrangler` CLI (`npm i -g wrangler`).

```bash
cd tutor-proxy
wrangler login                              # opens a browser
wrangler secret put ANTHROPIC_API_KEY       # paste your Anthropic key (from console.anthropic.com)
wrangler deploy                             # prints your Worker URL, e.g. https://bioinf-tutor.<you>.workers.dev
```

Then wire the page to the Worker:

1. Open `../tutor.html`, set `TUTOR_API_URL` to the URL `wrangler deploy` printed (keep the trailing `/`).
2. In `worker.js`, check `ALLOWED_ORIGINS` includes your Pages origin (`https://mutwil.github.io`) — it does by default. Re-run `wrangler deploy` if you change it.
3. Commit and push — GitHub Actions redeploys the page, and the 🤖 Tutor button appears.

(Until `TUTOR_API_URL` is set, the widget stays hidden, so students never see a broken button.)

## Recommended guardrails (shared key)

The proxy is public, so add cheap protections:

- **Origin allowlist** (built in) — rejects other websites' browsers (note: spoofable outside a browser).
- **Optional passcode:** `wrangler secret put TUTOR_PASSCODE`, then add the same value in `tutor.html`
  as an `x-tutor-pass` header on the `fetch` (stops casual drive-by use; visible in page source, so it's a speed-bump, not real auth).
- **Soft rate limit:** create a KV namespace and uncomment the block in `wrangler.toml`:
  ```bash
  wrangler kv namespace create RL
  ```
  paste the printed id, `wrangler deploy` → caps each IP to 8 questions/min.
- **Cloudflare dashboard rate-limiting rule** on the Worker route (extra layer, free tier).
- **`max_tokens` is capped** in `worker.js` (700 hint / 1200 deep) so a single call can't run away.
- **Monitor & rotate:** watch usage at console.anthropic.com; set a monthly **spend limit** there. If the key ever leaks/abused, `wrangler secret put ANTHROPIC_API_KEY` with a fresh key.

## Privacy note (GDPR)

Student questions + their on-page code are sent to Anthropic's API to generate the reply.
Tell students not to paste personal data. For a stricter setup, point the Worker at a
UCPH/KU institutional LLM gateway instead of the public API (change the `fetch` URL + auth).

## Files
- `worker.js` — the proxy (Socratic system prompt, model routing, streaming, CORS, caps).
- `wrangler.toml` — Worker config (+ optional KV rate-limiter).
- The widget lives in `../tutor.html`, injected into the page via `include-after-body`.
