# mutabloom.com

The website for Mutabloom, served by GitHub Pages. Four pages, one stylesheet, no build step,
no framework and no JavaScript: everything here is the file that gets served.

This is also the **only** copy. It deliberately does not live inside the game repository under
`site/`, because the two would have drifted the first time either was edited — which matters
more than usual here, because three of these URLs are compiled into the app binary and two of
these files are documents Apple and Google hold you to.

| Path | Why it exists |
|---|---|
| `index.html` | Landing page. Also the App Store Connect **Marketing URL**, which is the path ad-network crawlers walk to reach `app-ads.txt` |
| `privacy/index.html` | App Store Connect **Privacy Policy URL**. Required before the app can be submitted, and required by the ad network before it will serve |
| `support/index.html` | App Store Connect **Support URL**. Also required at submission |
| `terms/index.html` | Linked from the game's Settings screen. Goes into no App Store Connect field |
| `404.html` | Served by GitHub Pages for anything else |
| `app-ads.txt` | Must sit at the domain root, as `text/plain`. Holds `OWNERDOMAIN` and Google's seller line — see below |
| `style.css` | Every page's styles, matching the game's art direction (plan §5.2, §5.4) |
| `CNAME` | The custom domain, for GitHub Pages |
| `_config.yml` | The list of things this repository is **not** allowed to publish |

## Why the pages are directories

`privacy/index.html` rather than `privacy.html`, because the game links to
`https://mutabloom.com/privacy` with no extension and that string is compiled into the binary.
A directory with an `index.html` serves at that URL on every static host there is.
Extensionless serving of `privacy.html` is a per-host behaviour, and this is not a thing to be
clever about: if these URLs 404, the app ships with three dead links in Settings, which is a
guideline 2.1 rejection.

## The two files that are not decoration

**`app-ads.txt`** is how advertising networks confirm that whoever is selling advertising
inside the game is allowed to. It must be served from the root of the domain, as `text/plain`,
HTTP 200, with no redirect to another domain. Google crawls it by following the **Marketing
URL** on the App Store listing, so verification cannot even begin until both the listing and
this site are live. Allow about 24 hours after that. Until it verifies, the game's ad
inventory is unverified, and most programmatic demand discounts it or refuses to bid.

**It is complete.** Until 2026-09-17 it held `OWNERDOMAIN=mutabloom.com` and no seller line,
because there was no AdMob app for Mutabloom and so no publisher ID to authorise. Mutabloom was
added to AdMob that day (app `ca-app-pub-5706069574815967~1321980592`), and the Google line was
copied from AdMob's own "Set up app-ads.txt" dialog, not typed or adapted from another property.
It is the same publisher account as cratergut.com. A wrong `DIRECT` line authorises a seller
that is not used, which is worse than a missing one, because it looks verified and is not. Same
rule for every network added later — paste what that network's dashboard generates, and only
once that network is genuinely enabled in the shipping build.

**`privacy/index.html`** describes what the game actually does, not what a plan once said it
might do. AdMob rewarded and interstitial, UMP consent, App Tracking Transparency, Firebase
Analytics/Crashlytics/Remote Config, StoreKit purchases, Game Center, local notifications,
Live Activities and their push token, iCloud sync. **If any of that changes, this page changes
first**, before the build that changes it ships, and the "Last updated" line moves.

## Words that must never appear on this site

Mutabloom is its own game and owes nothing to anybody else's. Naming the thing it will
inevitably be compared to — in prose, in a comment, in a file name, in an alt attribute — is
how a clean-room product stops being one, and a public web page is the worst place for it,
because a web page is what a search engine reads and what a lawyer is shown. The plan document
in the game repository names a reference title in its market analysis; nothing from that
analysis belongs on this domain.

The same goes for the owner's other games: this site names none of them and links to none of
them. Cross-promotion between apps belongs in AdMob house campaigns, not on a legal page.

If a lint script is ever added here to enforce that, it goes in `scripts/`, which `_config.yml`
already excludes from publishing — read "What this repository publishes" below before adding
one, because on another property the check written to keep those words off the site was the
one thing on it publishing them.

## Setting it up

**All of this is DONE as of 12 September 2026** and the site is live and serving over HTTPS.
The steps are kept because they are how it gets rebuilt, and because step 3 is the one that
is invisible from inside this repository.

What was actually done, in this order: the two Namecheap parking records were deleted (a
`CNAME www → parkingpage.namecheap.com` and a `URL Redirect @ → http://www.mutabloom.com/`),
the nine records in the table below were added, Pages was enabled on `main` at the root, and
`https_enforced` was set once GitHub's certificate reached `approved` — which took about a
minute here, not the twenty-four hours the checkbox warns about.

**Namecheap's email-forwarding SPF record was deliberately left alone.** It is
`TXT @ v=spf1 include:spf.efwd.registrar-servers.com ~all`, it is not GitHub's business, and
nothing here asked for it to be deleted. Namecheap's API has no "add one record" call —
`setHosts` replaces every host record on the domain — so anything that ever automates this
must read, merge and write, not send nine records and hope. `cratergut-website/scripts/dns.py`
already does exactly that and is the thing to copy.

1. **Settings → Pages** in this repository: source `Deploy from a branch`, branch `main`,
   folder `/ (root)`.
2. **Settings → Pages → Custom domain**: `mutabloom.com`. GitHub writes a `CNAME` file; one is
   already committed here, so it should simply agree.
3. **DNS**, at whoever holds mutabloom.com. An apex domain needs A records rather than a
   CNAME:

   | Type | Name | Value |
   |---|---|---|
   | A | @ | 185.199.108.153 |
   | A | @ | 185.199.109.153 |
   | A | @ | 185.199.110.153 |
   | A | @ | 185.199.111.153 |
   | AAAA | @ | 2606:50c0:8000::153 |
   | AAAA | @ | 2606:50c0:8001::153 |
   | AAAA | @ | 2606:50c0:8002::153 |
   | AAAA | @ | 2606:50c0:8003::153 |
   | CNAME | www | therealcreynold.github.io |

   All four A records and all four AAAA records, not one of each: they are GitHub's edge
   addresses and dropping some of them costs availability rather than breaking the site
   outright, which is the sort of fault that shows up as an intermittent App Review failure.

4. **Enforce HTTPS**, back in Settings → Pages. GitHub has to issue a certificate first, so the
   checkbox can take up to 24 hours to become available. Apple will not accept a plain HTTP
   privacy policy URL.

## Checking it actually works

Once DNS has propagated, all six of these must return 200, and the last must be `text/plain`:

    curl -sSI https://mutabloom.com/
    curl -sSI https://mutabloom.com/privacy
    curl -sSI https://mutabloom.com/terms
    curl -sSI https://mutabloom.com/support
    curl -sSI https://mutabloom.com/nope          # expect 404, and the styled 404 page
    curl -sS  https://mutabloom.com/app-ads.txt

Also check that `www` and plain HTTP both land on the canonical HTTPS apex:

    curl -sSI http://mutabloom.com/ | grep -i location
    curl -sSI https://www.mutabloom.com/ | grep -i location

The privacy, terms and support URLs are linked from the game's Settings screen and are
compiled into the binary, so all three must serve. Only **two** of them are App Store Connect
fields, and the third field is not one of these pages at all:

| App Store Connect field | Value |
|---|---|
| Privacy Policy URL | `https://mutabloom.com/privacy` |
| Support URL | `https://mutabloom.com/support` |
| Marketing URL | `https://mutabloom.com` — the **apex**, not `/terms` and not `/privacy` |

`/terms` goes into no App Store Connect field.

### Why Marketing URL must be the apex

Because that field is not decoration: it is the entry point of the `app-ads.txt` crawl. An ad
network starts at the store listing, reads the Marketing URL, takes the **domain** of it, and
fetches `app-ads.txt` from the root of that domain. If Marketing URL is set to
`https://mutabloom.com/terms`, the crawler is being pointed at a page in a subdirectory, and
the whole verification depends on the crawler correctly reducing that to the apex rather than
looking for `https://mutabloom.com/terms/app-ads.txt` — which is a file that does not exist
here and never will, because `app-ads.txt` is only valid at the root.

Set it to the apex and there is nothing to reduce and nothing to get wrong. Getting this wrong
does not produce an error anywhere: the pages all still work, the listing looks fine, and
AdMob verification simply never completes, quietly, while launch week burns.

The same reasoning is why `OWNERDOMAIN` inside `app-ads.txt` is `mutabloom.com` with no
scheme and no path.

## What this repository publishes

**Only what `_config.yml` allows.** GitHub Pages deploys every file in a repository by
default, so without that file this site would publish this README along with the four pages.
`.nojekyll` does not help and cannot: with Jekyll off there is no ignore list at all, so there
is no way to keep a file in the repository and out of the public directory. Dotfiles are not a
way round it either — only `.git` is special-cased by GitHub.

That has bitten before, on another property, and badly: a checked-in list of terms that a lint
script existed to keep *off* the public site was itself served as prose, HTTP 200, on the
domain the App Store listing pointed at. The exact thing the "Words that must never appear"
section above exists to prevent, published by the tooling written to prevent it.

So Jekyll is on, and `exclude:` names `README.md` and `scripts`. `scripts` does not exist yet;
the exclusion is there so that whenever it is created it is already private. Excluding a
directory that does not exist costs nothing.

Jekyll is safe for these particular files because it renders only files carrying YAML front
matter, and every page here starts with `<!doctype html>` and has none, so each is copied byte
for byte. There is no Liquid syntax anywhere in the site, which was verified before the first
commit and is worth re-verifying after any page is edited.

**Anything added to this repository is public the moment it is pushed unless it is named in
`_config.yml`. Add it there first.** And after any change to that file, re-check every URL in
the section above: an excluded page and a broken page look identical from a terminal, and
three of those URLs are compiled into the shipping app.

## Left for a human, deliberately

- **The App Store link on `index.html`.** There is none, on purpose. As of 2026-09-20 the app
  is *submitted and waiting for review*, which is not the same as live, and a product URL that
  404s on the Marketing URL page is both a dead link and a bad look. Add it the day the listing
  goes live. The hero note and the "Coming soon" section both say "waiting for review" and both
  have to change at the same moment.
- **Any quote from a player.** Still absent on purpose: nobody outside has played it.

## The launch-day pass, done 2026-09-20

The third item that used to sit in the list above. All three "Last updated" dates moved to
20 September in the same commit as the corrections, which is the only honest way to move them:
a fresh date over stale prose asserts a re-read that did not happen.

**What it found is why the pass exists.** The pages were written on 12 September against the
plan, and the build had moved:

- **The privacy policy described a server that does not exist.** It said the only infrastructure
  the developer runs is a push-token service for Live Activities, and described what that
  service stores. There is no such service and there never was: the app contains no
  `URLSession`, no push token and no network call of its own, and the Live Activity is drawn on
  the device from the local save. Corrected to say there is no back end at all, which is both
  true and a stronger claim.
- **It described the pre-decision-14 advertising posture**, promising non-personalised adverts
  to everyone who declines consent, and never saying what happens to somebody who *allows*
  tracking — while the published App Store privacy label declares Device ID and Advertising
  Data as used to track you. Disclosing less than the label is the dangerous direction.
- **Both it and the terms page sold a Gardener's Pass subscription.** 1.0 sells eleven in-app
  purchases and no subscription of any kind.
- **The support page had six factual errors**, including the offline cap (it caps the whole
  garden, not only pets), which harvests keep a streak alive (a pet's do not), and switches for
  haptics and motion that the game does not have.
- **`app-ads.txt` named another of the owner's properties in a comment.** That file is served
  from the root as `text/plain` and ad networks are guaranteed to fetch it, which makes it the
  worst place on the domain to break the rule above — and the same shape as the cautionary tale
  in "What this repository publishes".

The lesson for the next pass: read the pages against the code, not against the plan. Every one
of those claims was true of the design at some point.
