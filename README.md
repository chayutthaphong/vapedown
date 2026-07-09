# Step Down — Clinical Vape Tapering Tracker

A tracker for quitting or reducing closed-system (pod) e-cigarette use through behavioural frequency modification rather than nicotine dilution. It records daily data and syncs to Google Sheets via Google Apps Script.

> **Disclaimer:** This tool is a framework for personal data collection and longitudinal tracking. It is not medical advice, diagnosis, or behavioural therapy. Please consult a healthcare professional for personalised guidance.

---

## Key Features

- **Daily puff counter** with Log / Undo buttons
- **Nicotine dependence assessment** using FTND (0–10) and the Penn State ECDI (0–20), taken weekly
- **Automatic taper plan** — computes a daily puff ceiling from dependence scores, then steps it down linearly to zero over the chosen program length
- **Selectable program length** of 4 / 6 / 8 weeks
- **Progress charts** — 7-day tracking and a 28-day monthly review (trend, daily average, zero-puff days)
- **Daily mood and trigger notes**
- **Google Sheets sync** with a local (localStorage) fallback when offline

---

## Design Philosophy

The ceiling is a **hard limit, not a target to fill**. The app is deliberately quit-oriented:

- Baseline ceilings are set **far below typical real-world use** (POD users average ~123 puffs/day, EMIT study)
- In-app messaging never says "you have puffs left to use." Instead it uses a **health-warning + challenge** tone — reminding the user that every avoided puff spares the lungs more nicotine and toxicants, and challenging them to stay below the ceiling
- The ceiling steps down to **0 (quit)** in the final week

---

## How the Ceiling Is Calculated

The baseline ceiling is taken from whichever dependence level is higher between FTND and ECDI:

| Dependence level | FTND baseline | ECDI baseline |
|---|---|---|
| Low | 24 | 24 |
| Moderate / Medium | 42 | 42 |
| High | 60 | 60 |
| Very High (ECDI only) | — | 78 |

> The maximum starting ceiling is **60 puffs/day** (High dependence). This is well below real-world POD use and has been tightened over successive revisions to push more firmly toward cessation.

The ceiling then steps down linearly each week:

```
ceiling(week) = ceil( baseTarget × [1 − (week−1)/(programWeeks−1)] )
final week = 0 (quit)
```

**Example** (High dependence → baseTarget = 60, 6-week program):

| Week | Ceiling (puffs/day) |
|---|---|
| 1 | 60 |
| 2 | ~48 |
| 3 | ~36 |
| 4 | ~24 |
| 5 | ~12 |
| 6 | 0 (quit) |

---

## Installation (GitHub Pages)

1. Place `index.html` at the root of your repository
2. Go to **Settings → Pages**, select a branch (e.g. `main`) and the `/root` folder
3. Open the URL GitHub Pages generates (e.g. `https://<username>.github.io/<repo>/`)

The app is a single-file HTML page that loads Tailwind and Chart.js via CDN — no build step required.

> **After pushing an update:** GitHub Pages may serve a cached copy. Wait for the deploy to finish, then hard-refresh or append a cache-buster to the URL (e.g. `?v=5`).

---

## Google Sheets Integration

The app exchanges data with a Google Apps Script (GAS) web app bound to a Google Sheet.

- Set `GAS_URL` in `index.html` to your GAS deployment URL
- The `logs` sheet uses the columns: `log_id`, `timestamp` (ISO), `date`

### Data flow

- **On app open:** `doGet` returns JSON (`logs`, `assessments`, `config`, `mood`, `notes`), and the app counts daily puffs from `logs`
- **On Log 1 Puff:** `doPost` writes a new log to the sheet immediately (`action: log`)
- **Also synced:** assessments (`assess`), mood/notes (`meta`), and program settings (`config`)

### Note on dates (important)

The app normalises the sheet's `date`/`timestamp` values to `YYYY-MM-DD` so the keys always match the current local date. Without this step, the puff count shows 0 every time the app opens.

> **Timezone caveat:** `timestamp` is in UTC. If used in the evening (GMT+07:00), the date derived from UTC may roll over to the next day. To anchor strictly to local (Thai) time, use the `date` column from GAS instead.

### Note on baseline values

Baseline ceilings (24 / 42 / 60 / 78) are computed **client-side** from the FTND/ECDI scores; they are not stored in the Sheet. When the app code is updated, existing assessments still apply and the new ceilings take effect on reload — no need to re-take the assessment.

---

## References

The full reference list (Vancouver style, verified against PubMed) is in the **Disclaimer & References** section inside the app. It covers:

- **Dependence assessment instruments** — FTND (Heatherton 1991), Penn State ECDI (Foulds 2015)
- **Consumption / frequency benchmarks** — EMIT study (Tillery 2023), Yingst 2020, Kosmider 2018, Aherrera 2020
- **Tapering strategy and program length** — Cochrane review (Lindson 2019), Lindson-Hawley 2016, Skelton 2022, RedPharm (Farley 2017), Cinciripini 2023
- **Clinical practice guidelines** — USPSTF 2021 (JAMA), German S3 Tobacco Guideline 2022

> Real-world usage figures (refs 3–6) describe actual behaviour, not safe limits. The ceiling values and linear taper schedule in this app are a pragmatic design choice informed by the literature above, not a validated dosing protocol.

---

## Stack

- HTML + Tailwind CSS (CDN)
- Chart.js (CDN)
- Google Apps Script + Google Sheets (backend / sync)
- localStorage (offline on-device storage)
