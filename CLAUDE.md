# Watch-Oid — CLAUDE.md
*For Trinity. Read before touching anything.*

---

## What Watch-Oid Is

Free AI-powered identification tool for watches — vintage, modern, mechanical,
quartz, the lot. Upload a photo — Watch-Oid identifies make/model/movement and
gives an estimated value. Ask Old Cogs (71-year-old master horologist,
apprenticed in Clerkenwell in 1971, autistic, endlessly patient) anything
about movements, fakes, servicing, or what's in grandad's drawer.

Part of the FeelFamous -Oid Ecosystem.

**Live at:** watch-oid.co.uk

---

## The Characters

**Old Cogs** — the identifier voice and chatbot. 71, Clerkenwell-trained,
runs a repair workshop. Dry wit, gentle honesty about fakes, treats a Casio
F-91W with the same respect as a Patek. `chat-cogs.js`.

---

## Stack

- **Static HTML** — single page (`index.html`), no framework, no build step
- **Netlify** — hosting + serverless `/netlify/functions/`
- **Supabase** — users, kudos, leaderboard, broadcast (`pdnjeynugptnavkdbmxh`)
- **Gemini 2.5 Flash** — `chat-cogs.js` (Ask Old Cogs). Never the Anthropic API.
- **Patreon** — membership OAuth (`patreon-auth.js`, referenced by
  `index.html` but the function file itself is not in this repo — see Known
  Issues)

---

## File Map

```
/
├── CLAUDE.md
├── LICENSE                     ← AGPL v3
├── index.html                  ← entire app: identify, Learn, Q&A, Ask Old Cogs, Gear, village/join
└── netlify/functions/
    └── chat-cogs.js             ← Ask Old Cogs chatbot (Gemini 2.5 Flash, ungated)
```

---

## Known Issues

- **`netlify/functions/analyze-image.js` does not exist in this repo.**
  `index.html` (`handleUpload()`, line ~1944) calls
  `/.netlify/functions/analyze-image` unconditionally for the core
  identify/roast flow, and no such file is present in `netlify/functions/`
  or anywhere in git history for this repo. The core "upload a photo of your
  watch" feature is currently broken on production unless this function
  exists some other way outside this checkout. Flag to Chris — this needs
  building (or restoring, if it was meant to be here and got lost), not a
  de-gate fix. When it's built, apply the two Gemini gotchas below.
- **`netlify/functions/patreon-auth.js` is also referenced but not present**
  in this repo (`index.html` calls `/.netlify/functions/patreon-auth?code=`
  on OAuth callback). Same situation — sign-in will fail until it exists.

---

## Free-to-use philosophy (Chris, 2026-07-13 — read before adding any gate)

The core tool is free for everyone, no sign-in, no lock icon, no "Villager+
only" banner. Don't gate the tool itself behind Patreon.

**What Patreon/paid tiers are for:** genuine extras that cost ongoing hosting/
upkeep and aren't required to use the tool. Frame honestly, never as a
shame-lock ("🔒 ... Unlock →"). No tier-comparison shop windows.

**The ask, when there is one:** one honest, low-key line after the task
completes — free to use, tell a mate if it helped, buy-me-a-coffee if you
want to say thanks (one-off, `buymeacoffee.com/chrispteemagician`), Patreon
if you want to be a regular. Not a gate. Not gamified.

**Repo-specific facts (don't relitigate):**
- Audited `index.html` and `netlify/functions/` for `isPro`, `patron_status`,
  "Villager only", "Founders only", "Elder only", and 🔒. Only two real hits,
  both in `handlePatreonCallback()` (line ~1537-38) just reading `data.isPro`
  off the Patreon OAuth response to store a badge/session — never used to
  block identify, chat, or anything else. **No hard gate existed on the core
  tool.**
- `signInPrompt` on the result card ("Sign in to earn kudos, track your
  identifications, and unlock village features!") is bucket 2 — kudos,
  leaderboard, and hamlet/village pages are genuine ongoing Supabase-hosted
  perks. Already framed honestly, no lock icon — left as-is.
- The "Join the Village" pricing card and "Join The Hamlet" tier grid were
  already correctly priced and honestly framed (no shame-lock copy, no
  tier-comparison shop-window feel) — no changes needed there.
- `chat-cogs.js` (Ask Old Cogs) has never had an `isPro` check — confirmed
  fully free/ungated, and has no `thinkingConfig`/`thinkingBudget` set (so
  the Gemini 2.5 Flash `thinkingBudget: 0` bug doesn't apply here).
- Added the standard honesty box (`#honestyBox`) to the result card in
  `index.html`, between the action buttons and the sign-in prompt. Hidden
  automatically for signed-in Patreon members via
  `showPatreonStatus()` (`document.getElementById('honestyBox')?.classList.toggle('hidden', !!patreonSession?.isPro)`).
- Fixed the one grandiosity/oversized-word hit from the ecosystem-wide voice
  pass: "World Domination Through Kindness. One tick at a time." appeared
  twice (The Code card, footer pull-quote) → "Just trying to be useful. One
  tick at a time." (same pattern as radi-oid/sail-oid, watch pun kept). No
  other combat verbs or cast-villain language found.
- No false-scarcity banner present. The `.founding-banner` class name is
  legacy naming only — its actual copy ("It's free to use, always will be.
  £4.95/month keeps the bots out if you want to chip in.") is already
  honest, no "first N only" pattern.
- Pricing already correct everywhere: Villager £4.95/mo, Elder Earned,
  Founder £14.95/mo (see table below) — no stale £3/£7/£15 found.

Full doctrine + mechanical pattern: DocBrain `tech/free-to-use-degate-skill.md`.

---

## Membership Tiers (Patreon — chrisptee campaign)

| Tier | Price | Perk |
|------|-------|------|
| 🏡 Villager | £4.95/mo | Hut in the village, kudos & leaderboard, recognised across all -Oids |
| ⭐ Elder | Earned | Everything in Villager + mini hamlet page, named in the village roll |
| 👑 Founder | £14.95/mo | Full hamlet suite, direct line to Chris, early access, 300 kudos on joining |

All Patreon links go to `https://www.patreon.com/chrisptee`.

---

## Gemini API Rules (Ecosystem-Wide)

Two known pitfalls — apply both when `analyze-image.js` gets built/restored:

1. **Do NOT set `thinkingBudget: 0`** — Gemini 2.5 Flash rejects it with a
   silent 400. Omit `thinkingConfig` entirely (as `chat-cogs.js` already does).
2. **Do NOT hardcode `mime_type: "image/jpeg"`** — always extract the real
   type from the data URL:
   ```js
   const mimeMatch = image.match(/^data:(image\/[\w+.-]+);base64,/);
   const mimeType = mimeMatch ? mimeMatch[1] : 'image/jpeg';
   const rawImage = image.replace(/^data:image\/[\w+.-]+;base64,/, '');
   ```

---

## Deploy

Push to `main` → Netlify auto-deploys. Never drag-to-Netlify. `git pull`
before every push.

---

*"Every watch has a heartbeat. My job is to listen to it." — Old Cogs*
