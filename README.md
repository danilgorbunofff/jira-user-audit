# jira-user-audit

A market review for a Jira Cloud app, written 2026-10-06. Everything below was checked live
against the Atlassian Marketplace API, Atlassian's own pricing pages and GitHub. Raw scans are
in `docs/evidence.md`; the datasets are in `data/`.

This is a research repository. No app code yet.

---

## The verdict

Build a **user and licence audit app for Jira Cloud**. It shows which paid seats belong to people
who never log in, what those seats cost, and lets an admin deactivate them in bulk.

The pitch is money, not convenience. Jira costs **$7.91 per user per month** on Standard and
**$14.54** on Premium. A 300-person company spends **$28k–$52k a year** on seats, and typically
10–20% of those seats belong to people who never log in. The app finds them and hands the buyer a
number they can take to their finance team.

---

## Why this market, and not the ones I checked first

Five things make the Atlassian Marketplace work where the others did not.

**1. Small shops win here.** 241 of the top 300 Jira Cloud apps are published by small teams or
individuals. codefortynine has five apps in the top 300 by itself; Broken Build Labs about ten.
The big vendors — Appfire, Tempo, Adaptavist, SmartBear, Xblend — hold only 59 of the top 300.

This is the opposite of the JetBrains Marketplace, where zero paid plugins reach the top 100 and
the median paid plugin released this year has 171 lifetime downloads.

**2. It is not winner-take-all.** The top 10 apps hold just **18.2%** of all installs across the
top 300. App #100 has 1,691 installs. App #300 still has 685. There is a real middle class here.

**3. Prices are real.** Verified per-user-per-month prices from small shops:

| App | Vendor | Installs | Price |
|---|---|---|---|
| Time in Status Reports | votazz | 1,460 | $6.60 |
| Work Time Calendar | BrizoIT | 1,092 | $7.70 |
| Manage Users for Jira Cloud | Bloompeak | 942 | $2.20 |
| Column Order Gadget | Communardo | 833 | $1.10 |
| Agile Retrospectives | Catapult Labs | 870 | $18.15 |
| Discord for Jira | Firnity | 682 | $5 → $11 |
| Magic Estimations | Magic Apps | 1,444 | $11.00 |
| Bulk Attachment Download | Pristine Bytes | 702 | $5.50 |
| eSign Signatures | Digital Rose | 682 | $14.30 |
| Risk Register | ProjectBalm | 2,189 | $5 → $19.25 |

Same customer as a JetBrains plugin, but one to two orders of magnitude more revenue per year.

**4. Almost no open-source competition.** GitHub searches for free Jira apps: planning poker —
8 repositories, best 4 stars. Time-in-status — 6 repositories, best 8 stars. Checklists — 3
repositories, all 0 stars. Nobody writes free Jira Cloud apps because they only run on Atlassian's
locked Forge platform. That platform is the moat, and it is the reason the usual "a free tool
already does this" rule does not kill the idea here.

**5. Solo developers demonstrably earn a living here.** Broken Build Labs publishes about ten
small chart gadgets, each with 750–2,244 installs. votazz has 5 apps between 604 and 1,641
installs. Seibert, BrizoIT, Futurate, Bloompeak, Agilis LT, Teamlead and codefortynine are all
small shops, each app sitting at 500–2,200 installs.

---

## The specific idea

Three small shops already do this job, and none of them dominates it:

- **Manage Users for Jira Cloud** — Bloompeak Labs — 942 installs — $2.20/user/month
- **User Management for Jira** (bulk and timed inactive-user deactivation) — TechTime Initiative
  Group — 836 installs
- **User/Group Export & License Optimizer for Jira** — Aety admin apps — 798 installs

No open-source equivalent exists. The free rival is Jira's own admin console, which does show
last-login and does let you deactivate an account — but one at a time, with no cost figure and no
scheduling. It is weak in exactly the places that matter.

### Honest limits

This is a **thin feature, not a moat**. It is copyable in a fortnight, and Atlassian could add it
to the admin console at any time. Treat it as the way in, not the destination.

The same customers go on to buy bulk field management, permission auditing and project-limit
warnings. A suite of small admin tools is a business. One is a weekend.

---

## Names to avoid

The recent failures in this store share one naming shape, and every one of them sits at 0–6
installs:

Label Doctor (1), Field Scout (12), Field Hygiene (0), Stale Radar (1), Queue Aging Heatmap (1),
Tacet (1), Orphaned-Owner Cleanup (0), Notification Hygiene (0), Inactive-User Hygiene (0),
Release Hygiene Guard (0), Filter Hygiene (2), Data Health (1).

The apps that live lead with the literal job: Manage Users for Jira Cloud (942), Bulk Attachment
Download (702), Time in Status (5,286), Column Order Gadget (833), Reminder for Jira (1,215),
Default Values for "Create Issue" screen (2,462).

Naming is not what killed those apps — the saturated lane did. But marketplace search matches on
the name and summary, so it is worth naming the app like a search query anyway.

codefortynine's five top-300 apps contain no brand word at all. Their brand lives in the **vendor
field**, which Atlassian prints under the app name on every listing.

**Avoid the words Hygiene, Cleanup and Guard.** They are the most crowded words in the store, and
they signal "occasional chore" to the buyer — the single best predictor of 0 installs found in
this research.

## Proposed listing name

**User Audit for Jira — Inactive Users, Licence Cost & Bulk Deactivate**

Alternatives: *Inactive Users for Jira* · *Jira Seat Cost Optimiser* · *Unused Seats for Jira*.

The repository name is a separate decision from the listing name. Keep the two in sync so they do
not drift apart.

---

## Revenue estimate

An app at roughly 1,000 installs priced at $5/user/month: in the region of **$5,000–$40,000 per
month**. This is an estimate, not a reported figure. The floor assumes 5% of installs convert to
paid at about 15 seats.

For reference, only 67.5% of startups earn anything at all in their first months, and the median
Stripe startup has billed $260 lifetime. Most of the number above is hope; the install curve and
the prices in `data/` are not.

---

## Method and data

- `data/jira-top300-apps.csv` — the top 300 Jira Cloud apps by installs: name, vendor, installs,
  categories, review count and average rating.
- `data/jira-marketplace-top30.csv` — the top 30 with verified per-user-per-month prices.
- `docs/evidence.md` — the raw scans behind every number above, plus the API quirks needed to
  reproduce them.

Source: `https://marketplace.atlassian.com/rest/2/addons` (no authentication needed), plus each
app's `?tab=pricing` page for prices and creation dates.

## Unknowns

- No verified figure exists for what an individual app author earns. Any specific number you see
  anywhere is an estimate.
- Atlassian does not publish a paid-versus-free split per category.
- Forge apps are JavaScript/TypeScript, not Java — a platform change from the IntelliJ plugins.
- Payout mechanics from inside Russia are unverified and are a real risk. Check this before
  writing code.
