# Getting sendoncue.com live

**Already done.** The repo exists, Pages is on, the site is built and serving, and
the custom domain is registered on GitHub's side:

- Live now at <https://boyarbirthday.github.io/sendoncue/>
- Every push to `main` redeploys in about a minute

**Left to do**, all of it at Namecheap and all of it Raquel only: sections 0, 3 and
4 below. Sections 1 and 2 are kept as a record of how it was set up.

---

## 0. First: verify the domain contact. This one has a clock on it.

The Namecheap domain list shows **ALERT** and a **VERIFY CONTACTS** button next to
sendoncue.com. That is the ICANN registrant email verification, not a Namecheap
upsell. **If it is not confirmed within 15 days of registration, the domain is
suspended** and the site stops resolving, whatever else is set up.

Namecheap sent a verification email to the registrant address when the domain was
registered on August 6, 2026. Find it and click the link. If it cannot be found,
press **VERIFY CONTACTS** in the domain list to have it resent.

---

## 1. Create the repository and push (done)

The site lives in its own repository rather than inside the app repo, for one
reason: **GitHub Pages on a free organization only serves public repositories**,
and the Cue app source should stay private. A separate public repo for the website
keeps the app private and costs nothing.

```sh
cd ~/sendoncue
gh repo create boyarbirthday/sendoncue --public --source=. --remote=origin --push
```

Nothing secret is in here. It is four HTML pages, one stylesheet, and brand art
that ships in the app anyway.

---

## 2. Turn Pages on (done)

Source is set to **GitHub Actions**, and `.github/workflows/pages.yml` deploys on
every push to `main`. The first run is green and the site is serving.

One thing worth knowing: GitHub will not start a workflow run for a push made with
a token it treats as automation, which includes the `gh` CLI's own OAuth token. So
the pushes made while setting this up did not auto-deploy and were dispatched by
hand. **An ordinary `git push` from your machine, or an edit saved in the GitHub
web editor, does trigger it.** If a deploy ever fails to appear, run
`gh workflow run pages.yml`, or press **Run workflow** on the Actions tab.

The custom domain `sendoncue.com` is also already registered on GitHub's side, both
by the `CNAME` file in this repo and through the Pages settings. It will start
working the moment DNS points at it, which is the next section.

---

## 3. Point the domain at it

### At Namecheap

Domain List, sendoncue.com, **Manage**.

First, under **Nameservers**, make sure it is set to **Namecheap BasicDNS**. The
Advanced DNS records below are ignored if it is set to anything else.

Then open the **Advanced DNS** tab.

**Delete the two records Namecheap adds by default.** They will fight with these
and cause a redirect loop:

- the `CNAME Record` for `www` pointing at `parkingpage.cash.namecheap.com`
- any `URL Redirect Record` on `@`

**Add these seven records:**

| Type | Host | Value | TTL |
| --- | --- | --- | --- |
| A Record | `@` | `185.199.108.153` | Automatic |
| A Record | `@` | `185.199.109.153` | Automatic |
| A Record | `@` | `185.199.110.153` | Automatic |
| A Record | `@` | `185.199.111.153` | Automatic |
| CNAME Record | `www` | `boyarbirthday.github.io.` | Automatic |

The four A records are GitHub's Pages addresses. All four, not one: they are how
GitHub load balances and fails over. The CNAME value keeps its trailing dot.

Optionally add IPv6 as well, which makes the site a little faster for some people
and costs nothing:

| Type | Host | Value |
| --- | --- | --- |
| AAAA Record | `@` | `2606:50c0:8000::153` |
| AAAA Record | `@` | `2606:50c0:8001::153` |
| AAAA Record | `@` | `2606:50c0:8002::153` |
| AAAA Record | `@` | `2606:50c0:8003::153` |

### Back at GitHub

Settings, Pages, **Custom domain**: enter `sendoncue.com` and save. It should
already be filled in from the `CNAME` file in the repo; if it is, just confirm the
check passes.

Wait for the DNS check to go green, then tick **Enforce HTTPS**. The certificate is
issued by GitHub automatically and is free. The tickbox stays greyed out until the
certificate exists, which usually takes a few minutes but can take up to an hour.

DNS propagation is normally 5 to 30 minutes on Namecheap. Check it with:

```sh
dig +short sendoncue.com
dig +short www.sendoncue.com
```

The first should return the four `185.199.x.153` addresses. The second should
return `boyarbirthday.github.io`.

---

## 4. Set up the support email

The site and the app both point at **hello@sendoncue.com**, and the App Store
listing gives it as the support contact. **It has to actually receive mail.**

Namecheap includes free email forwarding on domains registered with them:

Domain List, sendoncue.com, Manage, **Redirect Email** tab, add an alias:

| Alias | Forwards to |
| --- | --- |
| `hello` | `boyarbirthday@gmail.com` |

Test it by emailing hello@sendoncue.com from somewhere else and confirming it lands.

If you would rather not set this up, change the address in three places instead:
`index.html`, `privacy.html`, `terms.html`, `support.html` on this side, and
`CueLinks.email` in `Cue/Subscriptions.swift` on the app side. But a real address
on the domain reads better on an App Store listing than a Gmail address.

---

## 5. Afterwards

- Open every page on a phone and check the light and the dark rendering
- Submit `https://sendoncue.com/sitemap.xml` in Google Search Console and Bing
  Webmaster Tools, so the site is findable by name
- When the app is approved, add the App Store link to the hero button in
  `index.html`, replacing the "Tell me when it lands" mailto

---

## Checklist

- [x] `boyarbirthday/sendoncue` created as public and pushed
- [x] Settings, Pages, Source set to GitHub Actions
- [x] First workflow run finished green, site serving at boyarbirthday.github.io/sendoncue
- [x] Custom domain `sendoncue.com` registered on GitHub
- [ ] **Domain contact verified at Namecheap, so the domain is not suspended**
- [ ] Namecheap on BasicDNS, parking records deleted, four A records plus the www CNAME added
- [ ] GitHub DNS check passed, then Enforce HTTPS ticked
- [ ] `hello@sendoncue.com` forwarding set up and tested
- [ ] All four pages load over https on a phone

As of the last check, `sendoncue.com` still resolves to `192.64.119.215`, which is
Namecheap's parking page. That is the one thing standing between the site and the
domain.
