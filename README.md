# Michael — Portfolio Site

Personal portfolio built with **Jekyll** and hosted on **GitHub Pages**.

Live: [clrzr.github.io](https://clrzr.github.io)

---

## Editing Content

All content lives in `_data/`. You never need to touch HTML to update copy.

| File | What it controls |
|---|---|
| `_data/signals.yml` | Homepage — Selected Signals (the 4 proof points) |
| `_data/what_i_do.yml` | Homepage — What I Actually Do (title, body, story) |
| `_data/mental_models.yml` | Homepage — How I Think (3 named models) |
| `_data/carousel.yml` | Homepage — Product Thinking carousel slides |
| `_data/products.yml` | Products page — all 6 product cards + modal detail |
| `_data/engineering.yml` | Homepage — Engineering section cards |
| `_data/timeline.yml` | About page — full career timeline |
| `_data/nav.yml` | Navigation links (all pages) |

**To add a new timeline entry**, open `_data/timeline.yml` and append:

```yaml
- id: new-entry
  role: "Your Role"
  org: "Organisation"
  date: "2025 — Present"
  current: false
  skill_thread: false
  summary: "One-line summary shown collapsed."
  bullets:
    - "Proof point one"
    - "Proof point two"
  images:
    - src: "your-image[800x600].webp"
      alt: "Description"
```

Push. GitHub builds it. Done.

---

## Images

All images go in `/images/`. File names include their dimensions so the HTML always knows what size to expect:

```
images/headshot[480x600].webp
images/about-kwab-1[800x600].webp
images/product-edms[1280x720].webp
```

Replace placeholder images by dropping the real file with the same name.

---

## Running Locally

Requires Ruby 3.2+. Install via [RubyInstaller](https://rubyinstaller.org/).

```bash
# First time only
gem install bundler
bundle install

# Start local preview server
bundle exec jekyll serve
```

Open `http://localhost:4000`. The server auto-reloads on file changes.

---

## Project Structure

```
_data/          ← Edit content here
_layouts/       ← Shared HTML shell (base.html)
_includes/      ← Reusable components (nav, timeline entry)
images/         ← All images (placeholders + real)
index.html      ← Homepage template
about/          ← About page
products/       ← Products page
_config.yml     ← Jekyll settings (title, URL, excludes)
Gemfile         ← Ruby dependencies
```

---

## Deployment

Push to `main`. GitHub Pages detects `_config.yml`, runs Jekyll, and deploys automatically. No build step needed.
