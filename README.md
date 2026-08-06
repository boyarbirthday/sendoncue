# sendoncue.com

The marketing site and legal pages for **Cue**, the iPhone app that remembers the
day, drafts the note in your voice, and hands it to you ready to send.

Live at [sendoncue.com](https://sendoncue.com). The app itself lives in a separate
private repository.

## What is here

| File | What it is |
| --- | --- |
| `index.html` | The landing page |
| `privacy.html` | Privacy policy. **Required by App Store Connect.** |
| `terms.html` | Terms of Use / EULA, including the auto renewing subscription terms and Apple's required minimum terms. **Linked from the paywall inside the app.** |
| `support.html` | Support page. **Required by App Store Connect.** |
| `404.html` | Not found page |
| `assets/` | Stylesheet plus the app's own brand art: the handwritten wordmark, the mail pigeon, the paper grain tile, the icon |

## Design

There is no build step, no framework and no dependency. Four static pages and one
stylesheet.

The color tokens in `assets/styles.css` are copied from the app's `Theme.swift`, so
the site and the app are literally the same palette in light and in dark. Type uses
the system stack, which means an iPhone visitor is served New York and SF Pro: the
exact fonts the app draws with, with no webfont request and no layout shift.

House rules for any copy added here: no emojis, and no em dashes.

## Editing

Open a file, change it, push to `main`. The Pages workflow redeploys in about a
minute. To preview locally:

```sh
python3 -m http.server 8000
# then open http://localhost:8000
```

## Deployment

GitHub Actions builds and deploys on every push to `main`
(`.github/workflows/pages.yml`). The repo needs **Settings > Pages > Source: GitHub
Actions** set once.

The custom domain is pinned by the `CNAME` file, so it survives redeploys. DNS at
Namecheap points the apex at GitHub's four Pages addresses and `www` at
`boyarbirthday.github.io`.

## If a legal page changes

The privacy policy and the terms are both referenced from inside the app and from
the App Store listing. When either one changes materially, bump the effective date
at the top of the page and make sure the app's Settings screen still links to the
right place.
