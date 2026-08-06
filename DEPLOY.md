# Getting sendoncue.com live

Three things, in this order. Only the first one is urgent.

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

## 1. Create the repository and push

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

## 2. Turn Pages on

In the new repo: **Settings, Pages, Build and deployment, Source: GitHub Actions**.

That is the only click needed. `.github/workflows/pages.yml` already handles the
rest, and it runs on every push to `main`. Watch the first run under the Actions
tab; it takes about a minute.

At this point the site is live at `https://boyarbirthday.github.io/sendoncue/`.
The custom domain comes next.

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

- [ ] Domain contact verified at Namecheap, so the domain is not suspended
- [ ] `boyarbirthday/sendoncue` created as public and pushed
- [ ] Settings, Pages, Source set to GitHub Actions
- [ ] First workflow run finished green
- [ ] Namecheap on BasicDNS, parking records deleted, four A records plus the www CNAME added
- [ ] Custom domain confirmed on GitHub and the DNS check passed
- [ ] Enforce HTTPS ticked
- [ ] `hello@sendoncue.com` forwarding set up and tested
- [ ] All four pages load over https on a phone
