# Hedgehog Fit

A private [TRMNL](https://usetrmnl.com) e-ink plugin. On every refresh it shows an 80s-fitness-video hedgehog — sweatband, leg-warmers — with a cheeky German motivational saying nudging you into a small exercise. A random pose and matching saying are picked on each refresh.

## What it shows

Six poses are defined (Planke, Liegestütze, Bein-Dehnung, Aufwärmen, Sitz-Dehnung, Bizeps). Each pose has four possible sayings. The plugin picks a pose and a saying deterministically from the UTC timestamp, so the display changes with each refresh without requiring state.

## Plugin configuration

The plugin exposes one custom field in the TRMNL dashboard:

| Field | Key | Options |
|-------|-----|---------|
| Anrede (form of address) | `anrede` | Neutral, Weiblich, Männlich |

Setting **Weiblich** or **Männlich** causes the hedgehog to insert a gendered term of address (e.g. "Süße", "Großer") into sayings that have an `{ANR}` placeholder. Selecting **Neutral** (the default) leaves those slots empty.

## Implemented sizes

Only the **full** size (800x480) is implemented in `src/full.liquid`. The other size variants (`half_horizontal`, `half_vertical`, `quadrant`) exist as minimal stubs.

## Requirements

- Ruby **4.0.5** (pinned via `.ruby-version`)
- The [`trmnl_preview`](https://rubygems.org/gems/trmnl_preview) gem

Install the preview gem:

```bash
gem install trmnl_preview
```

## Local preview

```bash
trmnlp serve
```

Then open <http://127.0.0.1:4567/full> to see the rendered full-size layout.

The server watches `src/` and `.trmnlp.yml` and reloads on changes. Preview variables (user name, `anrede`, instance name) are configured in `.trmnlp.yml`.

## Regenerating images

The six hedgehog images (`src/assets/igel-1.png` ... `igel-6.png`) are downscaled grayscale PNGs. They are embedded as base64 data-URIs inside `src/shared.liquid` so they render on-device without any external HTTP requests (device network requests can fail or be slow).

`src/shared.liquid` is **generated** — never hand-edit it. After changing any file in `src/assets/`, regenerate it:

```bash
./bin/encode-images.sh
```

## Content and data

All plugin content lives in `src/settings.yml` under `static_data`. The plugin strategy is `static`, so the JSON is merged as template variables on every render. Poses, sayings, and the form-of-address pools are all defined there. To add a pose, a saying, or a new form-of-address term, edit `src/settings.yml`.

## Development notes

**Template files (`src/*.liquid`) must be ASCII-only.** Liquid reads template sources as US-ASCII and rejects non-ASCII bytes with `Invalid template encoding`. Umlauts, em-dashes, and other non-ASCII characters belong exclusively in `src/settings.yml`, which is read as UTF-8. This is why the generated comment in `src/shared.liquid` uses `--` instead of an em-dash, and why the sayings themselves live in `settings.yml` rather than in the template.
