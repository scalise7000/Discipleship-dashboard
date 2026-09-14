# Discipleship Dashboard — Church at The Mill

A static, single-file dashboard for tracking discipleship relationships across small groups, discipleship groups and Equip.

**Everything is in `index.html`.** No build step, no dependencies, no server. Open it locally or drop it on GitHub Pages.

---

## Deploying to GitHub Pages

1. Create a repository (public or private with Pages enabled).
2. Commit `index.html` at the repo root.
3. Settings → Pages → Source: "Deploy from a branch" → `main` / `/ (root)`.
4. The dashboard is live at `https://<org>.github.io/<repo>/` in about a minute.

Commit `sample-attendance-import.csv` too if you want the import format handy in the repo.

---

## Data in this version

The file ships with fabricated demo data: 76 active leaders, 18 available mentors, 125 disciples, and six weeks of attendance. **None of these are real people.** Replace before anyone outside the staff sees it.

Three ways to change the data:

| Method | What it does |
|---|---|
| Import data → `.json` | Replaces the whole dataset. Export all (JSON) first to see the shape. |
| Import data → `.csv` | Replaces one table. The headers decide which: a file with `leaderId` **and** `date` loads as attendance; `leaderId` without a date loads as disciples; neither loads as leaders. |
| Edit `index.html` | Find `var SEED =` near the top of the script block and replace the object. This is what makes the change permanent in the repo. |

Imports are held in memory only. Refreshing the page returns to whatever is baked into `index.html`. That is deliberate: there is no hidden browser storage, so nobody's contact details linger on a shared machine.

---

## The ministry calendar

The "% complete in yr" denominator is computed, not typed in. Groups run the second week of August through December, then January through May. Weeks containing Thanksgiving, Christmas Day, New Year's Day or Easter are dropped.

For 2026–27 that produces **38 meeting weeks**, running Aug 10, 2026 through the week of May 24, 2027.

`buildWeeks(year)` in the script regenerates the list for any ministry year, including a correct Easter date. To change which holidays are excluded, edit the `skip` list inside that function.

---

## Metrics

| Metric | Definition |
|---|---|
| Meetings reported this week | Leaders with attendance in ÷ active leaders |
| Meetings % (per leader) | Meetings held ÷ ministry weeks since that leader's start date |
| % complete in yr | Meetings held ÷ 38 ministry weeks |
| Quiet 3+ weeks | No attendance reported for three or more consecutive ministry weeks |

`QUIET_WEEKS` at the top of the script changes the quiet threshold everywhere at once.

---

## Messaging

Select any rows, then "Message selected."

**Email** opens the staff member's own mail client with the selection in BCC. Nothing is sent by this page: a real person at the church presses send, and replies come back to them rather than to a no-reply address. Past about 60 addresses a mailto link gets silently truncated by the browser, so the dashboard stops and offers to copy the list for pasting into Outlook's BCC field instead. `MAILTO_MAX` at the top of the script sets that ceiling.

**Text** does not send. Pressing it opens a screen explaining that sending would use the same system the church already uses for the weekly attendance texts, and what connecting it would take, then copies the numbers for pasting into that platform.

To make texting live later: confirm which platform sends the weekly attendance texts, since it is already A2P 10DLC registered and adding a second vendor means a second registration, a second bill and a second number the congregation does not recognize. Then stand up one endpoint holding that platform's API token as an environment secret, gate it behind staff sign-in, and change `explainTexting()` to post `{to: [...], body: "..."}` instead of copying. Never put the token in `index.html`: anything in this file is readable by anyone who can open the page.

## Wiring up the live TouchPoint feed

The capture flow is already in place: automated texts go to small group, discipleship group and Equip leaders, they tap the link, and attendance lands in TouchPoint. This dashboard is the read side of that.

Two workable paths:

**Scheduled export (simplest).** A GitHub Action runs nightly, calls the TouchPoint API, writes a fresh `data.json` to the repo, and Pages redeploys. Credentials live in GitHub Secrets. The dashboard reads the file instead of the inline seed. Data is up to a day stale, which is fine for a weekly rhythm.

**Live proxy.** A small serverless function holds the TouchPoint credentials and serves JSON to the dashboard on load. Always current, and it can be locked to church staff sign-in.

Either way, replace the seed assignment with a fetch and call `render()`:

```js
fetch("data.json")
  .then(r => r.json())
  .then(d => { DATA = d; TODAY = new Date(); route(); });
```

---

## Access control

GitHub Pages hosts and serves the file. It does not decide who may read it: a public repo means a public dashboard.

Cloudflare Access sits in front of Pages rather than replacing it. Point a domain you control at the Pages site, then set a rule such as "only addresses ending in @churchatthemill.com may pass." Everyone else gets a sign-in screen and never reaches the page. Free up to 50 users. The alternative is a private repo on a paid GitHub plan, which also works.

Either one is a decision for before the first real import, not for today.

## Privacy note

This file contains names, email addresses and phone numbers. On a public GitHub Pages site, every one of them is public and indexable. Use a private repo with Pages restricted to organization members, or put the dashboard behind the church's existing staff sign-in, before loading real congregants.
