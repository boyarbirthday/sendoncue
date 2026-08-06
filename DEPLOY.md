# Getting sendoncue.com live

**Already done.** The repo exists, Pages is on, and the whole site is built and
serving right now:

- **<https://boyarbirthday.github.io/sendoncue/>** — open it, this is the real site
- Every push to `main` redeploys in about a minute

**Left to do**, all Raquel only: sections 0, 3 and 4 below. Sections 1 and 2 are
kept as a record of how it was set up.

The custom domain is deliberately **not** set on GitHub yet, and there is
deliberately no `CNAME` file in this repo. GitHub freezes a Pages deployment in
"waiting" the moment a custom domain is configured whose DNS still points
somewhere else, which is the case until the Namecheap records change. Setting it
early wedges the deploy queue and makes the github.io preview redirect to a parked
page. So: DNS first, then the domain. Section 3 does them in that order.

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
every push to `main`. The site is serving.

One thing worth knowing: GitHub will not start a workflow run for a push made with
a token it treats as automation, which includes the `gh` CLI's own OAuth token. So
the pushes made while setting this up did not auto-deploy and were dispatched by
hand. **An ordinary `git push` from your machine, or an edit saved in the GitHub
web editor, does trigger it.** If a deploy ever fails to appear, run
`gh workflow run pages.yml`, or press **Run workflow** on the Actions tab.

There are two old runs sitting in "waiting" and "pending" on the Actions tab, left
over from setting the custom domain too early. They are harmless and block nothing,
because the concurrency group is now keyed by ref. GitHub's API refuses to cancel
them; cancelling from the Actions tab in a browser usually works if the clutter
bothers you.

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

### Wait for DNS, and only then set the domain on GitHub

Propagation is normally 5 to 30 minutes on Namecheap. Watch it with:

```sh
dig +short sendoncue.com
dig +short www.sendoncue.com
```

The first should return the four `185.199.x.153` addresses. The second should
return `boyarbirthday.github.io`. **Do not move on until it does**, because setting
the domain on GitHub while DNS still points at the parking page is what wedges the
deploy queue.

Once it resolves: Settings, Pages, **Custom domain**, enter `sendoncue.com`, save.
The DNS check should go green within a few seconds.

Then tick **Enforce HTTPS**. The certificate is issued by GitHub automatically and
is free. That tickbox stays greyed out until the certificate exists, usually a few
minutes, occasionally up to an hour.

With the Actions deploy flow, the domain set in Settings is authoritative and
survives every redeploy on its own. There is no `CNAME` file in the repo and none
is needed.

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
- When the app is approved, swap in the App Store link in `index.html` at the two
  commented spots (the hero CTA and the closing "get" section), and change both
  "Soon on the App Store" pills to "On the App Store"

---

## Checklist

- [x] `boyarbirthday/sendoncue` created as public and pushed
- [x] Settings, Pages, Source set to GitHub Actions
- [x] Deploy green, every page serving 200 at boyarbirthday.github.io/sendoncue
- [ ] **Domain contact verified at Namecheap, so the domain is not suspended**
- [ ] Namecheap on BasicDNS, parking records deleted, four A records plus the www CNAME added
- [ ] `dig +short sendoncue.com` returns the four GitHub addresses
- [ ] Custom domain then set in Settings, Pages, and the DNS check passed
- [ ] Enforce HTTPS ticked
- [ ] `hello@sendoncue.com` forwarding set up and tested
- [ ] All four pages load over https on a phone

As of the last check, `sendoncue.com` still resolves to `192.64.119.215`, which is
Namecheap's parking page. That is the one thing standing between the finished site
and the domain.
