# 5S Audit — standalone PWA

Why this exists: inside Claude's chat preview, the app runs in a sandboxed
iframe that blocks microphone access and caps storage per key — that's why
voice fill failed silently and archived audits sometimes wouldn't reopen.
Hosting it as a real PWA (same pattern as your Avon Traders / Azad Guest
House apps) removes both limits.

## 1. Deploy the Cloudflare Worker (holds your Anthropic API key)

```
cd 5s-audit-pwa
npx wrangler login
npx wrangler secret put ANTHROPIC_API_KEY
npx wrangler deploy
```

Wrangler will print your worker URL, e.g.
`https://5s-audit-ai.<your-subdomain>.workers.dev`

## 2. Point the app at your worker

In `index.html`, update:

```js
const AI_WORKER_URL = 'https://5s-audit-ai.YOUR-SUBDOMAIN.workers.dev/extract';
```

to your actual worker URL (append `/extract` — or drop it, the worker
doesn't care about the path, only the method).

## 3. Deploy the static app to GitHub Pages

```
git init
git add .
git commit -m "5S audit PWA"
git remote add origin <your-repo-url>
git push -u origin main
```

Then in the repo: Settings → Pages → Deploy from branch → `main` / root.

Your app will be live at `https://<you>.github.io/<repo>/` — camera capture,
voice fill, and full-size localStorage all work normally there since it's a
real origin, not a sandboxed preview.

## Notes

- Data (current audit + past audits) is stored in the browser's
  `localStorage` on the device, same as your other PWAs. Nothing leaves the
  phone except the short voice transcript sent to your Worker for AI
  extraction.
- Voice fill needs Chrome/Edge on Android; iOS Safari doesn't support the
  Web Speech API, so on iPhone you'd type the observation instead.
- Icons (`icon-192.png`, `icon-512.png`) are placeholders — swap in your
  own branding if you want a custom home-screen icon.
