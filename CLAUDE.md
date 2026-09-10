# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single static page (`index.html`) used as the link-in-bio target from Elena Koiro's Instagram
profile. It routes visitors to Nature's Sunshine (NSP) registration and country shops, carrying her
sponsor ID `ru7163310`. No build step, no dependencies, no backend — open the file or serve the
folder.

Python/`pyproject.toml`/`.venv` exist only as ad-hoc tooling (Pillow via `uv run --with pillow`) for
cutting image assets; `main.py` is leftover PyCharm boilerplate and is not part of the page.

## Layout contract — read before changing CSS

The page must fit **entirely on a phone screen with no scrolling** (target: iPhone 15 inside the
Instagram in-app browser, ~393x630-750 CSS px) while keeping the two-column composition of the
approved design (`alean-koira-final.jpeg`).

That is achieved by a fixed-width "sheet" that is scaled to fit, not by responsive breakpoints:

- `.stage` is `position:fixed; inset:0` and centers the sheet; `body` has `overflow:hidden`.
- `.sheet` is a fixed **430 px** wide column (`BASE_WIDTH` in the inline script). All internal sizes
  are authored against that width — hence the unusually small px values (10-13 px text).
- The script measures `sheet.offsetHeight` and applies
  `scale(min(availW / 430, availH / needH, 1.9))`, re-running on resize, orientation change, font
  load and image load.

Consequences to respect when editing:

- **Adding vertical content lowers the scale factor for everyone** — the sheet is currently ~810 px
  tall, giving ~0.89 scale on an iPhone 15. Growing it makes all text physically smaller. Trade
  content out, don't pile it on.
- Don't add media queries that change the sheet's width or reflow it into one column; the scale
  handles small screens.
- Keep `white-space:nowrap` elements short — inside a 430 px sheet a long nowrap label overflows the
  card and gets clipped.

## Content and links

Source of truth for links, ID and contacts is `alena-koira-data.pdf` (extract with
`uv run --with pypdf python -c "..."`; the PDF's `/Annots` carry the URLs). Currently wired:

| Element | Target |
| --- | --- |
| Europe — register | `https://sk4691134.ru.e-naturessunshine.com/` |
| Europe — shop | `https://ru.e-naturessunshine.com/k/77/sklep` |
| Other countries — register | `https://nsp25.com/signup?sid=7163310&market=25` |
| Ukraine / Belarus-Russia / Moldova / Turkey | `nsp.com.ua/partner/`, `nsp.com.ru/`, `nsp.md/`, `natr.com.tr/ru/` |
| Instagram | `https://instagram.com/alena_koira` |
| Telegram / email | `https://t.me/ekoyro`, `ekoyro@gmail.com` |

The sponsor ID appears twice (the chip and step 2 text) plus once in the copy-button JS via
`#sponsor-id` — change all of them together.

## Assets

- `assets/photo.png` — circular avatar, cut from `alena-portrait.jpg` (the original headshot).
- `assets/logo.png` — NSP logo lifted from the mockup with the beige background keyed out by
  luminance; regenerating it means re-running that Pillow threshold, not a plain crop.
- `assets/europe.jpg`, `assets/globe.jpg`, `assets/stone.png` — crops from the mockup, **placeholders
  pending originals**. Their headline text is rendered in HTML, so the crops deliberately start below
  the baked-in text in the mockup.
- `assets/flags/*.svg` — from the open `flag-icons` set (MIT), not cut from the mockup.

Handwritten lines ("В гармонии с собой", "Здоровье доступно каждому!") are live text in the
Marck Script webfont, not images.

## Checking a change

```bash
uv run python -m http.server 8765     # then open http://127.0.0.1:8765/index.html
```

Verify at 393x752 **and** 393x628 (in-app browser with its chrome) that nothing is clipped and
`document.documentElement.scrollHeight === window.innerHeight`.
