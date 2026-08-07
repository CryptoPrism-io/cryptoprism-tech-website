---
name: plausible-analytics
description: Develop and manage Plausible analytics for cryptoprism.tech — install/verify the cookieless tracker, add custom events, register Goals. Use when adding or debugging tracking, events, or goals in this repo.
---

# Plausible Analytics — cryptoprism.tech

## Fixed facts
- Analytics host: `https://plausible.yogeshsahu.xyz` (self-hosted Plausible CE)
- Tracker: `/js/script.outbound-links.js` (auto: pageviews, outbound clicks, file downloads, scroll, engagement)
- `data-domain` is a label, not a URL — one host serves every product
- This site: `cryptoprism.tech` (already registered — do not re-register)
- Unregistered domain (new/friend product): owner registers FIRST (see yogeshsahu-website `.claude/skills/plausible-analytics/SKILL.md`); events are dropped otherwise.

## 1. Install/verify tracker
Entry point: `public/index.html` (Firebase static hosting; check `firebase.json` headers). Tag:

```html
<script defer data-domain="cryptoprism.tech" data-outbound-links data-file-downloads
  src="https://plausible.yogeshsahu.xyz/js/script.outbound-links.js"></script>
```

CSP: if a `Content-Security-Policy` header exists in `firebase.json`, allow the host in
`script-src` AND `connect-src` (event POST is cross-origin) — otherwise events get blocked.

## 2. trackEvent helper (if none)
```ts
declare global { interface Window { plausible?: (event: string, opts?: { props?: Record<string, string | number> }) => void } }
export function trackEvent(name: string, props: Record<string, string | number> = {}): void {
  if (typeof window === "undefined" || typeof window.plausible !== "function") return;
  window.plausible(name, { props });
}
```

## 3. Instrument key interactions
snake_case names + `source` prop when multiple triggers. Conversion: CTA clicks, signup/login, form submit. Engagement: feature open, search, nav. Content: read, open, click. Comment each: `// event: <name>`.

## 4. Goals (conversion events only)
```sql
INSERT INTO goals (event_name, inserted_at, updated_at, site_id, display_name, scroll_threshold)
SELECT '<event_name>', now(), now(), id, '<Display Name>', -1 FROM sites WHERE domain = 'cryptoprism.tech'
ON CONFLICT DO NOTHING;
```
Run on the Plausible VM + restart the plausible container (owner-side).

## 5. Verify
1. `curl -sI https://plausible.yogeshsahu.xyz/js/script.outbound-links.js` → 200
2. POST `https://plausible.yogeshsahu.xyz/api/event` (JSON body to a file; `--data-binary @file` on Windows) with `"domain":"cryptoprism.tech"` → **202**
3. Redeploy Firebase hosting; hard-refresh the live site; confirm the event in Plausible Realtime.

## Hard rules
- No dummy/synthetic events — real interactions only; ask first if unsure.
- Minimal diff, follow repo conventions.
- Never commit secrets (tracker host is public; API keys are not).
- Register BEFORE tracker for unregistered domains.
