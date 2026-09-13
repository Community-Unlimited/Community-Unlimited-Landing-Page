# Community Unlimited — Landing Page

Static landing page built to the Community Unlimited Brand Guide v0.7,
with a browser-based content editor at `/admin`.

No build step, no framework, no dependencies. Vercel serves the files as-is.

---

## Editing content

Everything editable lives in **`content.json`**. Two ways to change it:

**1. The editor (recommended)** — open `https://<your-site>/admin`

- Edit contact numbers, hero copy, events, pathway steps, audience groups, facts, footer
- Add, delete and reorder events, steps, facts and groups
- Upload new photographs
- **Publish to site** commits `content.json` (and any new images) to this repo, which triggers a Vercel rebuild. Live in about a minute.
- **Download JSON** if you'd rather commit the file yourself

Publishing needs a GitHub token — the editor explains how to make one. It is
stored in that browser tab only and is never sent anywhere except GitHub's API.

**2. By hand** — edit `content.json`, commit, push. Vercel redeploys.

### Lock down /admin before sharing the site

`/admin` is publicly reachable. Nobody can change anything without a token, but
you should still restrict it:

> Vercel → Project → Settings → **Deployment Protection** → Password Protection

`robots.txt` and a `noindex` header already keep it out of search results.

---

## How the page stays fast

`index.html` contains the current content as real HTML, so the page renders
immediately and works with JavaScript disabled. On load it fetches
`content.json` and overrides only what has changed. If the fetch fails, the
built-in markup stands.

This means **`index.html` and `content.json` can drift**. If you change text via
the editor, the HTML fallback still holds the older wording. To resync, copy the
new values into the matching `data-cu` elements in `index.html`. It only matters
for the no-JavaScript case.

Fields map through `data-cu` attributes:

| Attribute | Effect |
|---|---|
| `data-cu="hero.subhead"` | sets text content |
| `data-cu-src="hero.image"` | sets an image `src` |
| `data-cu-alt="hero.imageAlt"` | sets `alt` text |
| `data-cu-wa` | builds the WhatsApp link from `contact.*` |
| `data-cu-tel` | builds the `tel:` link |
| `data-cu-list="whatsOn.events"` | re-renders a list from its `<template>` |

To add a new editable field: add it to `content.json`, tag the element in
`index.html`, and add one line to `SCHEMA` in `admin/index.html`.

---

## `/qa` — the exploration build

`qa.html` is served at **`/qa`** (Vercel's `cleanUrls` strips the extension).
URLs are case-sensitive, so `/QA` is redirected to `/qa` in `vercel.json`.
It is a **standalone** page: it does not fetch `content.json` and nothing on
`/` imports from it. Editing content in `/admin` changes the live page only —
`/qa` is untouched, and vice versa.

It exists to answer the 5 Sep feedback deck and the note that the site is
"clean, but a little plastic-ky". What changed:

**From the feedback deck**

| Slide | Change |
|---|---|
| 1 | Hero runs to a 1520px frame instead of 1180px, so it uses far more of the screen |
| 2 | Hero heading reduced ~10%, two lines, tight leading (and now reads `Community Unlimited`) |
| 3 | Band caption box widened to the title's width; `Get involved` + `See what's on` buttons added |
| 4–5 | Purpose section rebuilt as **Option 2** — photo left, icon accordion right |
| 6 | Each pathway step has an icon; steps fade in one at a time on scroll (150ms apart) |
| 7 | `Who it's for` moved up, now directly after the purpose section |
| 8 | On mobile the pathway rail stands on its end — a vertical line with the icons as nodes |
| 9 | Mobile hero: taller photo, panel overlaps it by 24px, heading −10%, tighter spacing, both CTAs full-width and equal, supporting text clamped to two lines, reduced bottom padding, one radius on all four corners |

**From the brief** — "Community Unlimited", not "Community Without Limits";
speak to people who have just retired or are about to; think older, feel younger.

- Hero headline is now the brand name. The hashtag follows: `#CommunityUnlimited`.
- New **"Which one are you?"** section, second on the page. It names the three
  mindsets from the brief in the words people actually use, and answers each
  one. This is the part doing the work against "doesn't feel special".
- `Age is just a number.` replaced with **`Think older. Feel younger.`** The
  original is the line an AAC poster uses; that was half the plastic feeling.
- Hero subhead: `Retired from work. Nowhere near done.`
- A running noticeboard strip under the hero, so the week reads as live.
- `Who it's for` tags changed from `Primary / Secondary / Tertiary` to
  `If this is you / If this is your parent / If you want to back it`. The
  three groups and their copy are unchanged — those labels are a funder's
  hierarchy, not something to show a 63-year-old.

**Three deliberate deviations from Brand Guide v0.7**, on top of the three the
live page already documents below:

- **Fraunces for display type**, Inter kept for everything you read. Inter
  everywhere is what makes a page read "standard website"; a soft serif at
  large sizes is warm and a bit upmarket without being fussy. Body copy stays
  Inter at 19px for legibility.
- **Warm paper ground** `#FCFAF6` and a sand tone `#F4EEE4` instead of pure
  white and mint everywhere. Flat mint on white is the clinical, institutional
  look the feedback was reacting to.
- **Film grain** over the dark fields and the photography, and a mild
  desaturation on the photos. It unifies eight separately-generated images and
  stops large flat colour reading as plastic.

Every contrast pair still meets AA: orange is never used below 20px/600,
bronze `#964E1E` remains the small-text accent.

**Known compromise:** `assets/hero.webp` is a 1600×1000 portrait-ish
composition. At the new full-width crop there is no horizontal slack, so the
senior — the primary audience — ends up behind the emerald panel. The plate is
mirrored on desktop (`transform:scaleX(-1)`) to put her clear of it. Replace it
with a landscape-composed hero photograph and that one CSS rule can go.

`/qa` is `noindex` in the page head, in `robots.txt` and in `vercel.json`.

---

## Files

```
index.html          the landing page
qa.html             the /qa exploration build (standalone, noindex)
content.json        all editable content (feeds index.html only)
admin/index.html    the content editor
assets/             photography (WebP) + mark.svg
assets/brand/       logo lockups (PNG) for decks and print
vercel.json         caching + security headers
robots.txt          keeps /admin and /qa out of search
```

---

## Deploying

Vercel → **Add New → Project** → import this repo → Deploy. No settings to
change; framework preset is "Other". Every push to `main` redeploys.

Local preview (the editor needs a server — `file://` blocks the JSON read):

```bash
npx serve .
# then open http://localhost:3000 and /admin
```

---

## Design notes

Colours, type and layout follow Brand Guide v0.7. Three decisions deviate, on purpose:

- **Body text is 19px**, not the guide's 16px. The primary audience is 60–70 and
  the guide asks for WCAG-friendly contrast and large type; 16px doesn't deliver
  that on screen. Captions are 13px minimum, not 12px.
- **Orange is never used for small text.** `#EA5C2A` on white is 3.5:1, which
  fails AA for body copy. It's restricted to fills and to button labels at 20px
  semibold, where the large-text threshold (3:1) applies.
- **`#964E1E` is used as the accent text colour.** It's the colour the C and U
  actually produce where they overlap in the logo, sampled from the mark. It
  reaches 6.2:1 on white, so it's safe for small labels — and it means the
  right thing. Worth adding to the palette page as a fourth, text-safe token.

The hero's emerald panel deliberately overlaps the photograph. That overlap is
the logo's idea expressed as layout. `assets/mark.svg` is a vector rebuild of
the mark (C: arc centred 128,127.5, radius 107, stroke 42; U: 66-radius bowl,
stroke 34) with the bronze intersection done via an SVG mask, so it's exact
rather than an approximated blend.

---

## Still placeholder

- WhatsApp number `6580000000` and phone `8000 0000`
- The three event names, times and descriptions
- `communityunlimited.sg` is not yet pointed at this deployment
