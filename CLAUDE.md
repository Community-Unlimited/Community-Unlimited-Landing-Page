# Community Unlimited — working notes

## Share production links, not previews

The team reviews on **https://community-unlimited-landing-page.vercel.app**,
which is served from `main`. Vercel branch previews
(`…-git-<hash>-….vercel.app`) are not where they look.

So when handing over a finished page, give the production path —
`https://community-unlimited-landing-page.vercel.app/<route>` — and say plainly
whether it is live there yet or still sitting on a branch. Work that only exists
on a preview reads to this team as not done.

The site owner has asked more than once for exploration pages to go to
production rather than wait in an open PR. Confirm before merging rather than
assuming it carries over from a previous session.

**Never merge without asking** when a change touches `index.html`,
`content.json`, `admin/`, or `assets/`. That is the live public page and its
editor.

## Route map

| Route | File | What it is |
|---|---|---|
| `/` | `index.html` | The live page. Hydrates from `content.json`; `/admin` publishes to it. |
| `/qa` | `qa.html` | Exploration build — warm, editorial, Fraunces display type. |
| `/new` | `new.html` | Exploration build — dark, loud, heavy motion. |
| `/admin` | `admin/index.html` | Content editor. Writes `content.json` only. |

Exploration pages are **standalone**: no `content.json` fetch, and nothing on
`/` imports from them. That is deliberate — `/admin` must not be able to break
an exploration page, and an exploration page must not be able to change `/`.
Keep it that way when adding more.

Paths are **case-sensitive** on Vercel, and `cleanUrls` is on, so `foo.html`
serves at `/foo`. Use lowercase filenames — `/QA` had to be renamed to `/qa`
for exactly this reason.

Each exploration route carries `noindex` in three places: the page `<head>`,
`robots.txt`, and a header rule in `vercel.json`. Scope the `vercel.json` rule
to the exact path so it cannot leak onto `/`.

## Non-negotiables

The audience is 60–70 year olds. These do not bend for visual ambition:

- Body text stays **19px**; captions 13px minimum.
- Tap targets **48px+**.
- **Every** text/background pair meets WCAG AA — measured, not estimated.
  - Orange `#EA5C2A` is 3.5:1 on white and 3.46:1 under white. It is only legal
    as *large* text (20px/600+) or as a fill. Never small text.
  - On orange fills use **near-black text** (5.58:1), not white.
  - `#964E1E` is the small-text accent on light grounds (6.2:1).
  - `#FF7A47` where brand orange is too dark against a dark card.
- `prefers-reduced-motion: reduce` must switch animation **off**, not down,
  and what replaces it must be **static** — never a scroll container.
  Swapping a marquee's animation for `overflow-x: auto` leaves a dead strip
  with a native scrollbar under it, which reads as a broken widget. Wrap the
  content and hide any duplicate copy that existed only to make the loop
  seamless. Headless Chromium uses invisible overlay scrollbars, so this class
  of bug does **not** show up in scripted screenshots — check the computed
  `overflow-x` and `scrollWidth > clientWidth`, not just the picture.
  Motion sensitivity rises with age. Verify it rather than assuming the media
  query covered everything.

## Gotchas already hit here

- `max-width: NNch` resolves against **that element's** font-size. On a wrapper
  inheriting 19px body text it yields a ~200px column, not a heading-width one.
- `position: sticky` on a **grid item** is constrained to its own grid area, so
  sticky cards can never overlap. Use block flow for stacking effects.
- Padding on a sticky element's parent sits **outside** the content box that
  constrains it. To extend a sticky range, add a real in-flow spacer element.
- `overflow-x: hidden` on `body` breaks `position: sticky`. Use `clip`.
- `scroll-behavior: smooth` makes scripted `scrollTo` measurements read stale.
  Set `scrollBehavior = 'auto'` before measuring in a browser test.
- Content that is `white-space: nowrap` inside a flex row next to a fixed-width
  sibling can overflow its container and be **silently clipped** by an ancestor's
  `overflow: hidden` — no scrollbar, no error, just missing text. The `/qa`
  ticker lost an item this way at phone width.
- `assets/hero.webp` is 1600×1000, composed as a portrait. At full-bleed crops
  the senior — the primary audience — falls out of frame, and both exploration
  pages carry a workaround. A landscape-composed hero photograph retires both.

## Testing

No build step, no framework, no test suite. Verify by rendering:

```bash
python3 -m http.server 8899
# drive Chromium at /opt/pw-browsers/chromium-1194/chrome-linux/chrome
```

Check at 390px and 1512px: no console errors, no horizontal overflow, all
images load, contrast computed numerically, and reduced-motion genuinely quiet.
