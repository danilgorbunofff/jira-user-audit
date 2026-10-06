# Evidence

Raw scans behind the conclusions in the [README](../README.md). All data collected 2026-10-06.
Nothing here is copied from secondary sources — every figure was pulled live and can be reproduced
with the commands below.

---

## 1. Method and API quirks

Base endpoint, no authentication required:

```
https://marketplace.atlassian.com/rest/2/addons
```

Parameters: `application`, `hosting`, `cost` (`free`|`paid`), `category*`, `text`, `sortBy`,
`order`, `offset`, `limit`, `withPricingInfo`, `includePrivate`, `withVersion`.

Response shape: `{_links, _embedded:{addons:[…]}, count}`.

Install counts live at `_embedded.distribution.totalInstalls`, alongside `downloads`,
`totalUsers`, `bundled` and `bundledCloud`. `totalInstalls` is the meaningful metric;
`downloads` over-counts badly.

### Quirks that cost real time

| Problem | Reality |
|---|---|
| `limit=100` returns no install counts | The `_embedded.distribution` block is **silently dropped** above `limit=50`. Always use `limit=50`. |
| `text=` search returns no install counts | Same drop. Fetch `GET /rest/2/addons/{key}/distribution` per addon instead. Confirmed reliable. |
| `sortBy=created` / `newest` / `updated` / `trending` | **Entirely ignored.** All return the identical top 3. Only `sortBy=downloads&order=desc` does anything, and it is already the default. There is no date-sorted or cohort query available. |
| No creation date in the API | There is no `dateCreated` field. Only `_embedded.lastModified`. Creation dates must be scraped from the app's `?tab=pricing` page. |
| No pricing API | `/rest/2/addons/{key}/pricing`, `/pricingInfo`, `/prices` → 404. `/rest/1|3|4/…` → 404. `/gateway/api/marketplace/catalog/*` → 401. |

### The working price route

1. `GET https://marketplace.atlassian.com/rest/2/addons/{key}` → read `_links.alternate.href`,
   for example `/apps/1226187/time-in-status-reports-for-jira?tab=overview`. **Take the numeric ID
   from here** — a wrong ID returns a generic page with zero USD.
2. Build `https://marketplace.atlassian.com` + that href, then replace `?tab=overview` with
   `?tab=pricing`.
3. Fetch, then **strip all whitespace before matching**. The price JSON fields are sometimes one
   per line, and a naive regex on the raw text fails because the newline-to-space replacement
   breaks the literal.

Fields to extract:

```
"@type":"AggregateOffer","price":…,"lowPrice":…,"highPrice":…
"dateCreated":"…"
"floor":1,"ceiling":<n>,"amount":<price>     <- the free tier ceiling
```

**Diagnostic tell:** if `dateCreated` matches but `AggregateOffer` does not, you fetched the
overview page (the `?tab=overview` → `?tab=pricing` replacement silently no-ops when the href has
no query string) or the app is genuinely free.

Do not scrape vendor sites for prices. codefortynine.com, herocoders.com and saasjet.com are
SPA-rendered with no price strings and no links containing `pricing`.

---

## 2. Market concentration — the top 300

Six pages of `limit=50`. Total **959,328 installs** across the top 300 Jira Cloud apps.

| Cut | Share of top-300 installs | Installs at that rank |
|---|---|---|
| Top 10 | 18.2% | 11,728 |
| Top 25 | 26.6% | 3,631 |
| Top 50 | 41.7% | — |
| Top 100 | 54.3% | 1,691 |
| Top 300 | 100% | 685 |

**241 of the top 300 come from outside the large vendors** (Appfire, Adaptavist, Tempo, SmartBear,
Xblend, Atlassian). **217 of the top 300 are small shops at a narrower threshold.**

Roughly 182 of those apps sit between 350 and 2,500 installs — that band is the actual addressable
market, and it is where every one of the reference prices below was taken from.

### Repeated small-shop portfolios

| Vendor | Apps in top 300 | Installs |
|---|---|---|
| Broken Build Labs | ~10 chart gadgets | 750–2,244 each |
| votazz | 5 | 604–1,641 |
| Seibert | 3 (Awesome Custom Fields 1,105 · Templating.app 1,073 · Didit Checklists 506) | 506–1,105 |
| codefortynine | 5 | — |
| Others | BrizoIT, Futurate, Agilis LT, Bloompeak, Teamlead, DevSamurai, Getint, Move Work Forward, Apwide | — |

### Category map

| Category | Apps | Installs | Median installs |
|---|---|---|---|
| Project management | 186 | 457,739 | 1,163 |
| Workflow | 65 | 367,932 | 1,441 |
| Reports | 110 | 278,820 | 1,336 |
| Dashboard gadgets | 71 | 247,119 | 1,114 |
| Integrations | 55 | 226,902 | 1,020 |
| Charts & diagramming | 58 | 221,774 | 1,995 |
| Custom fields | 52 | 186,052 | 1,105 |
| Source code | 6 | 181,049 | 8,411 |
| Time tracking | 52 | 161,263 | 1,436 |
| Admin tools | 48 | 130,282 | 956 |
| Testing & QA | 22 | 118,923 | 1,980 |
| Themes & styles | 6 | 8,509 | 1,517 |
| Dependency management | 9 | 14,575 | 849 |
| Security | 11 | 8,562 | 956 |

---

## 3. Reference prices

Verified from each app's live `?tab=pricing` page. Free tier is **"up to 10 users" on 21 of the
23 apps checked** — the universal freemium norm on this marketplace.

| App | Vendor | Installs | Price, USD/user/month |
|---|---|---|---|
| Time in Status Reports | votazz | 1,460 | $0 → $6.60 |
| Work Time Calendar | BrizoIT | 1,092 | $0 → $7.70 |
| eSign Signatures | Digital Rose | 682 | $0 → $14.30 |
| Column Order Gadget | Communardo | 833 | $0 → $1.10 |
| Agile Retrospectives | Catapult Labs | 870 | $0 → $18.15 |
| Discord for Jira | Firnity | 682 | $5 → $11.00 |
| Manage Users for Jira Cloud | Bloompeak | 942 | $2 → $2.20 |
| Magic Estimations | Magic Apps | 1,444 | $0 → $11.00 |
| Bulk Attachment Download | Pristine Bytes | 702 | $0 → $5.50 |
| Risk Register | ProjectBalm | 2,189 | $5 → $19.25 |

Jira's own seat price, read from the JSON on `atlassian.com/software/jira/pricing`:

```
"price":"0        (Free)
"price":"7.91     (Standard)
"price":"14.54    (Premium)
```

The page is about 1.4 MB and contains no `$`-prefixed or `user/month` strings — parse the
`"price":"…"` JSON instead.

---

## 4. The dead lane: cleanup, hygiene and stale-data apps

28 apps screened in the cleanup / hygiene / duplicate / archive / stale-data lane. **21 of them sit
at 0–6 installs.**

| App | Installs |
|---|---|
| Label Doctor | 1 |
| Field Scout | 12 |
| Field Hygiene | 0 |
| Stale Radar | 1 |
| Queue Aging Heatmap | 1 |
| Tacet | 1 |
| Orphaned-Owner Cleanup | 0 |
| Notification Hygiene | 0 |
| Inactive-User Hygiene | 0 |
| Release Hygiene Guard | 0 |
| Filter Hygiene | 2 |
| Data Health | 1 |

One real winner in the whole lane:

- **Optimizer for Jira (Health Checks & Cleanup)** — key `co.uk.jackgraves.jira-optimiser` —
  a solo developer — created **2017-05-24** — **1,124 installs**, 55,203 downloads —
  **$5.00 → $5.83** per user per month.

The mechanism, and the single most useful finding in this research: **apps people open daily or
every sprint accumulate 500–2,500 installs. Apps for rare admin chores get 0–10 installs no matter
how good the name is.** The dead pile is all "fix this once and never think about it again" work,
sitting behind one nine-year-old incumbent.

---

## 5. Tool connectors

40 popular tools screened for an existing Jira app. Big SaaS vendors ship their own and dominate:

| Tool | Installs |
|---|---|
| Figma | 48,642 |
| Sentry | 16,294 |
| Miro | 12,325 |
| Snyk | 5,585 |
| PagerDuty | 4,095 |
| Notion | 2,822 |

Third-party connectors to mid-size tools stay small:

| Tool | Vendor | Installs |
|---|---|---|
| HubSpot | Reichert | 479 |
| Monday.com | Getint | 454 |
| Freshdesk | — | 386 |
| Asana | Getint | 355 |
| ClickUp | — | 200 |
| JFrog | — | 193 |
| NetSuite | — | 203 |
| Databricks | — | 37 |
| Firebase | — | 43 |
| Terraform | Shiranui Software | 16 |
| Qualys | — | 14 |

**Rule:** if a SaaS company ships its own Jira app, do not try to beat it.

**Flagged, not concluded:** observability (Datadog, Grafana, Prometheus, Zabbix, New Relic) has no
consequential Jira app. That looks like a gap, but those vendors all ship free webhook
integrations, so the gap is probably illusory. Untested — do not build on it without checking
first.

---

## 6. Open-source competition check

GitHub, 2026-10-06:

| Search | Repositories | Best |
|---|---|---|
| Planning poker for Jira | 8 | 4 stars |
| Time-in-status for Jira | 6 | 8 stars |
| Checklists for Jira | 3 | 0 stars |
| Issue templates for Jira | 47 | top hit unrelated (583 stars, a security tool) |
| Dashboard gadgets for Jira | 27 | top hit unrelated (189 stars) |

Essentially zero free competition in any Jira app shape. The reason is structural: Jira Cloud apps
run only on Atlassian's locked Forge or Connect platform, so nobody writes free open-source for the
ecosystem. **The platform itself is the moat** — that is why the free-competitor rule that killed
almost every other lane in this research does not apply here.

Search gotcha: `q=jira+app+atlassian+in:name,description,readme+stars:>200` returns meaningless
awesome-lists. Use `q=jira+language:java+stars:>150`, `q=topic:jira+stars:>100` or
`q=atlassian-connect+stars:>50`.

---

## 7. Marketplace economics

- ~9,121 Atlassian apps total: 5,752 paid, 3,369 free.
- Jira Cloud: 4,668 apps — 3,238 paid, 1,916 free.

Revenue share, quoted verbatim from
`developer.atlassian.com/platform/marketplace/pricing-payment-and-billing/`:

> "Forge apps: 84% Connect apps: 80% Data Center apps: 75%. This pricing is effective 1 April
> 2026. On 1 October 2026, the revenue share rate will change to the following: Forge apps: 83%
> Connect apps: 75% Data Center apps: 75%… As of 1 January 2026, partners will receive 100% of
> gross revenue earned on Forge, up to $1 million in lifetime Forge revenue… calculated in
> aggregate across all Forge apps."

Two things to note. The Forge rate **drops from 84% to 83% on 1 October 2026** — today. And the
100%-up-to-$1M Forge deal **started 1 January 2026**, so it is live now and favours Forge over
Connect for a new solo developer.

---

## 8. The free rival, stated plainly

Jira's own organisation admin page already shows last-login dates and lets an admin deactivate an
account.

It cannot do bulk operations, it shows no money figure, and it cannot schedule or automate. That is
the whole opening. The app is worth paying for because of those three gaps, not because the admin
page does not exist.

---

## 9. Why the alternatives were rejected

| Lane | Verdict | Reason |
|---|---|---|
| JetBrains paid plugins | FAIL | 15% commission, zero paid plugins in the top 100, median 2026-cohort plugin 171 lifetime downloads, median price $19/yr |
| Perforce IntelliJ plugin | THIN | The real competitor is the free P4V client; `id 69` has 13.5M downloads at 2.7/5 and is not bundled |
| Stagecraft (Jenkins + IntelliJ) | FAIL as described | — |
| Log-Smith (IntelliJ log tool) | THIN PASS | Best commercial evidence of the three, but still the wrong marketplace |
| AI/agent tooling gaps | FAIL | Every agent runtime ships human-in-the-loop free; Cloudflare's `isitagentready.com` commoditised the rest |
| Connectors to observability tools | UNTESTED, doubtful | Vendors' own free webhooks probably cover it |

---

## 10. Reproducing all of this

```powershell
# Top 300 by installs (6 pages, ~5 minutes)
1..6 | ForEach-Object {
  $off = ($_ - 1) * 50
  curl.exe -s "https://marketplace.atlassian.com/rest/2/addons?application=jira&hosting=cloud&limit=50&offset=$off"
}
```

Then, per app key, fetch the alternate href and swap the tab to `pricing` as described in section 1.
Strip whitespace before matching.
