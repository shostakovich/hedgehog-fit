# Hedgehog Fit

A private [TRMNL](https://usetrmnl.com) e-ink plugin. On every refresh it shows an 80s-fitness-video hedgehog — sweatband, leg-warmers — with a cheeky German motivational saying nudging you into a small exercise. An exercise and a matching saying are picked on each refresh.

## What it shows

Twenty office-friendly bodyweight exercises are defined, evenly spread across **mobilisation**, **strength** and **stretching** (each takes under a minute). Every exercise has a name, a short description, a matching hedgehog image and three possible sayings. The plugin picks an exercise and a saying deterministically from the UTC timestamp, so the display changes with each refresh without requiring state.

The exercise/image reference lives in [`docs/uebungen-referenz.md`](docs/uebungen-referenz.md) (rendered overview: [`docs/uebungen-referenz.html`](docs/uebungen-referenz.html)); the sayings in [`docs/uebungen-texte.md`](docs/uebungen-texte.md).

## Plugin configuration

The plugin exposes these custom fields in the TRMNL dashboard:

| Field | Key | Type | Options / notes |
|-------|-----|------|------|
| Bild-Basis-URL | `bild_basis_url` | url | Optional. Base URL hosting the 20 hedgehog images (trailing slash). Leave empty to use the default GitHub Pages-hosted images. |
| Anrede (form of address) | `anrede` | select | Neutral, Weiblich, Männlich |

**Anrede** — setting **Weiblich** or **Männlich** causes the hedgehog to insert a gendered term of address (e.g. "Stachelfee", "Igelheld") into sayings that have an `{ANR}` placeholder. Selecting **Neutral** (the default) leaves those slots empty.

## Images

The 20 hedgehog images are not embedded; they are loaded by URL. The image files live in [`docs/assets/uebungen/`](docs/assets/uebungen/) and are published via **GitHub Pages** (served from the `main` branch's `/docs` folder) at:

```
https://shostakovich.github.io/hedgehog-fit/assets/uebungen/
```

That URL is the built-in default, so the plugin works out of the box with no configuration. To host the images elsewhere, set **Bild-Basis-URL** to the directory that contains the files; the plugin appends each exercise's filename (`01-schulterkreisen.png` … `20-ganzkoerper-streckung.png`).

## Implemented sizes

Only the **full** size (800x480) is implemented in `src/full.liquid`. The other size variants (`half_horizontal`, `half_vertical`, `quadrant`) exist as minimal stubs.

## Requirements

- Ruby **4.0.5** (pinned via `.ruby-version`)
- The [`trmnl_preview`](https://rubygems.org/gems/trmnl_preview) gem (`gem install trmnl_preview`)

## Local preview

```bash
trmnlp serve     # dev server at http://127.0.0.1:4567/full
trmnlp build     # write static HTML to _build/
trmnlp lint      # check against TRMNL best practices
```

For local preview the variables (user name, `anrede`, instance name) are configured in `.trmnlp.yml`. To see real images locally, set `bild_basis_url` there to a reachable URL.

## Content and data

All plugin content lives in `src/settings.yml` under `static_data`. The plugin strategy is `static`, so the JSON is merged as template variables on every render. Exercises (name, image filename, description, sayings) and the form-of-address pools are all defined there. To add/edit an exercise or saying, edit `src/settings.yml`.

## Development notes

**Template files (`src/*.liquid`) must be ASCII-only.** Liquid reads template sources as US-ASCII and rejects non-ASCII bytes with `Invalid template encoding`. Umlauts, em-dashes, and other non-ASCII characters belong exclusively in `src/settings.yml`, which is read as UTF-8. Keep comments and markup in the `.liquid` files plain ASCII; all German text with umlauts lives in `settings.yml`.
