# Step Down — Clinical Vape Tapering Tracker

**Step Down** is a single-file web app that helps people quit or reduce closed-system (pod) e-cigarette use. Because pre-filled pods can't be diluted, the app works through **behavioural frequency modification** — logging every puff, setting a quit-oriented daily ceiling, and stepping it down to zero over a chosen program length. Alongside the numbers, it adds **real-time craving support**: an interval/pacing timer with binge detection, a guided 6-step CBT loop, and a coping-plan library. It records data locally and syncs to Google Sheets, and presents progress through Daily, Weekly, and Monthly reports.

> **Disclaimer:** This tool is a framework for personal data collection and longitudinal tracking. It is not medical advice, diagnosis, or behavioural therapy. Please consult a healthcare professional for personalised guidance.

---

## Table of Contents

- [What the app does](#what-the-app-does)
- [Logging puffs](#logging-puffs)
- [Dependence assessment](#dependence-assessment)
- [The weekly ceiling](#the-weekly-ceiling)
- [The dynamic daily challenge](#the-dynamic-daily-challenge)
- [Interval / pacing timer & binge detection](#interval--pacing-timer--binge-detection)
- [Working through a craving (CBT loop)](#working-through-a-craving-cbt-loop)
- [Coping plan library](#coping-plan-library)
- [Reports](#reports)
- [Date picker (historical viewing)](#date-picker-historical-viewing)
- [Heat-coloured percentages](#heat-coloured-percentages)
- [Mood & notes](#mood--notes)
- [Evening reminder](#evening-reminder)
- [Design philosophy](#design-philosophy)
- [Google Sheets sync](#google-sheets-sync)
- [Installation](#installation-github-pages)
- [References](#references)
- [Stack](#stack)

---

## What the app does

At a glance, Step Down lets a user:

- Log each puff (individually or in batches) with a real timestamp
- Take a weekly nicotine-dependence assessment (FTND + Penn State ECDI)
- See two targets side by side every day: a fixed **weekly ceiling** and a tighter **dynamic daily challenge**
- Get a live **pacing timer** that widens the gap between puffs and flags **binges**
- See a **journey card** with total puffs, day count, and an **average per day** (total puffs ÷ all days since tracking started — days with no puffs count as zero, so the average drops as you have clean days)
- Defuse an urge in the moment with a guided **6-step CBT loop** and an **urge-surf timer**
- Browse a **coping-plan library** of evidence-based craving strategies
- Track consumption over time in **Daily / Weekly / Monthly** report tabs
- Scroll back through history with a **date picker**
- Record overall daily **mood** and free-text **triggers/notes**
- Sync everything to **Google Sheets**, with a local fallback when offline

---

## Logging puffs

- **Log 1 Puff** — the primary button, adds a single puff
- **+3 / +5** — batch buttons for catching up quickly; each puff is still written as its own timestamped record (so hourly charts stay accurate)
- **Undo** — removes the most recent puff of the day

Every logged puff is saved locally and pushed to Google Sheets immediately, one row per puff.

---

## Dependence assessment

Once per program week the app prompts a two-part questionnaire:

- **FTND** — Fagerström Test for Nicotine Dependence (score 0–10), adapted for vaping
- **Penn State ECDI** — Electronic Cigarette Dependence Index (score 0–20)

The higher of the two dependence bands sets the starting ceiling (see below). Assessment scores are also plotted over time in the Monthly report.

---

## The weekly ceiling

The **ceiling** is a fixed hard limit for the current program week. Its starting value (week 1) is derived from the dependence assessment:

| Dependence level | FTND baseline | ECDI baseline |
|---|---|---|
| Low | 24 | 24 |
| Moderate / Medium | 42 | 42 |
| High | 60 | 60 |
| Very High (ECDI only) | — | 78 |

The baseline is the **higher** of the FTND and ECDI values. The maximum starting ceiling is **60 puffs/day** (High) — well below typical real-world POD use (~123 puffs/day, EMIT study) — deliberately set to push toward cessation.

The ceiling then steps down linearly each week to zero:

```
ceiling(week) = ceil( baseTarget × [1 − (week−1)/(programWeeks−1)] )
final week = 0 (quit)
```

**Example** (baseTarget = 60, 6-week program):

| Week | Ceiling (puffs/day) |
|---|---|
| 1 | 60 |
| 2 | ~48 |
| 3 | ~36 |
| 4 | ~24 |
| 5 | ~12 |
| 6 | 0 (quit) |

Program length is selectable: **4 / 6 / 8 weeks**.

---

## The dynamic daily challenge

Alongside the fixed ceiling, the app shows a tighter, self-adjusting **daily challenge** that reacts to your own recent behaviour:

```
challenge = min( weekly ceiling, yesterday's actual puffs × 0.9 )
```

Behaviour in detail:

- **First day** (no previous day recorded): challenge = ceiling
- **If yesterday stayed below the ceiling:** today's challenge is **10% lower than what you actually used** — so it ratchets down as you improve (e.g. used 50 → challenge 45 → 41 → 37 …)
- **If yesterday reached or exceeded the ceiling:** the challenge **resets to the ceiling** rather than rewarding the overshoot
- **When a new week lowers the ceiling:** the lower ceiling automatically **caps** the challenge, so the challenge can never exceed the current week's ceiling

The two targets play different roles: the **ceiling guarantees a minimum rate of reduction**, while the **challenge encourages going faster when you can**. Both are shown together on the main screen and in the Daily report. (Both are pragmatic design choices, not validated dosing protocols.)

---

## Interval / pacing timer & binge detection

Below the log buttons, a live card shows **the earliest time you should take your next puff** — turning "cut down" into a concrete, moment-to-moment target. It only appears during waking hours (**07:00–23:00**) and once there's at least one logged puff.

### Base interval

The base gap is learned from **your own** recent behaviour:

```
base interval (min) = mean real inter-session gap over the last 7 days × 1.1
```

Puffs less than **2 minutes** apart are treated as one continuous *session*, so those tiny within-session gaps are **excluded** from the average — otherwise a rapid burst would collapse the interval toward zero. With too little data (fewer than 5 inter-session gaps), it falls back to spreading today's **challenge** evenly across the 16-hour waking window.

### Session & binge detection

- **Session size** = how many recent puffs are chained together with < 2 min gaps.
- **Binge threshold** = `min( 3, round(average session size) + 1 )`. It's a *hybrid*: it can **learn downward** (if your typical session is a single puff, the threshold tightens to 2) but is **hard-capped at 3**, so a run of habitual large sessions can never quietly normalise a high bar.
- A session **over the threshold** is flagged as a binge — the card turns red and explains that many puffs in a row deliver a nicotine spike comparable to several separate uses.
- **Peak hours** (which stretch the interval further) are derived from the **last 7 real days ending today** — or fewer if less data exists — using the mean hourly rate across active hours, *not* just the current day. This is anchored to today regardless of which historical date you're viewing in the reports.

### Interval expansion

When a session exceeds the binge threshold, the required gap **accumulates** on top of whatever time was still owed from a prior binge — it doesn't reset just because a new, smaller session starts in between:

```
for each session, in time order:
  remaining = time still left on the standing deadline, if any (else 0)
  if this session is a binge (size > threshold):
      new deadline = this session's last puff + remaining + (base × multiplier for this session's size)
  else (session size ≤ threshold):
      if remaining > 0: deadline is left unchanged (a normal-sized session neither adds to nor cancels a standing binge deadline)
      if remaining = 0: deadline = this session's last puff + base × 1 (a fresh, ordinary wait)

multiplier(size) = 1 + 0.5 × max(0, size − threshold), then ×1.3 more if that session's hour was a personal peak hour
```

In practice: binge 5 puffs and you're told to wait, say, 2 hours. Puff 2 more within the wait window and the 2 hours **doesn't restart from zero and doesn't just vanish** — since that session is within threshold, the standing 2-hour deadline carries through unchanged. Binge again (another 5+ puff session) while time is still owed, and the new binge's own required wait is added on top of whatever was left, pushing the deadline further out. Once the original deadline has actually elapsed, a fresh session is judged purely on its own size — nothing carries over indefinitely. A long break (over 4 hours between sessions) is treated as genuine recovery and also clears any carried-over deadline, so accumulation never spans across sleep or a multi-hour gap.

A progress bar fills as the interval elapses, with a live countdown. Colours: **indigo** while waiting, **green** once you may pace a puff, **red** while a binge — current or still carrying over from a recent one — governs the wait.

### Early-log nudge & urge surfing

Logging **before** the interval is up isn't blocked — the app is supportive, not punitive — but it gently notes how many minutes early you were and offers a **5-minute urge-surf timer** (a countdown that helps you ride the craving's peak, which usually fades within a few minutes).

---

## Working through a craving (CBT loop)

The **🧠 Work through a craving** button opens a guided **6-step** worksheet based on standard cognitive-behavioural craving management. It walks you from the automatic reach-for-the-pod to a considered choice:

1. **Notice the urge** — pick the trigger (stress/anxiety, boredom, others vaping/offered, a habit cue like coffee/meal/drive, low mood, or physical withdrawal). *Required to continue.*
2. **Catch the thought** — free-text the automatic thought the craving is using (e.g. "I can't cope without it").
3. **Rate the craving** — a 0–10 slider for the strength of **this urge right now** (explicitly *not* your overall daily mood). Naming the number itself loosens its grip.
4. **Talk back to it** — tap a ready-made reframe or write your own kinder, truer response.
5. **Pick a coping action** — suggestions are **tailored to the trigger you chose**, with a link into the full coping library. *Required to continue.*
6. **Re-rate the urge** — score it again; the app shows the change and responds accordingly (encouragement if it dropped, a nudge to try another action or step away from the cue if it held or rose).

**What gets saved:** CBT is kept **completely separate from the user's own Clinical Notes** — it never writes into the notes field. Each completed loop is stored as a structured record in a local `cbtHistory[date]` array, and posted to Google Sheets as a **structured `cbt` record** (`action: 'cbt'`) that writes each field to its **own column** in a dedicated `cbt` sheet — `date`, `timestamp`, `time`, `trigger_id`, `trigger_label`, `thought`, `urge_before`, `urge_after`, `urge_delta`, `reframe`, `action_taken` — so the data is ready for pivots and analysis without parsing text.

The **content you enter is not shown back** on the main screen; instead a small badge on the "Work through a craving" button shows **how many CBT loops you've done today** (e.g. "2× today"), resetting each day.

The structured columns require a small Apps Script handler (see `apps_script_cbt_addon.md`); until it's added, the `cbt` posts are harmlessly ignored. The `cbtHistory` structure stays on-device and is never overwritten by the sync merge.

---

## Coping plan library

The **📄 Coping plans** button opens a filterable library of **12** evidence-based craving strategies, grounded in CBT for smoking cessation — the "4 Ds" (delay, deep-breathe, drink water, distract), urge surfing, stimulus control, and trigger decoupling. Tap any plan to expand a short how-to.

Filter by trigger category: **Stress / low mood · Boredom · Social / others vaping · Habit / routine · Withdrawal**. The library is reachable directly from the main screen or from step 5 of the CBT loop, and the CBT loop pre-filters it to match your selected trigger.

---

## Reports

Three switchable tabs, each anchored to the date chosen in the picker (defaults to today). All hourly charts are line charts (x = hour of day, y = puffs/hour).

### Daily
- **Puffs**, **ceiling**, and **% of ceiling** for the selected day
- **Challenge** value and **% of challenge**, both heat-coloured
- **Line chart** of puffs per hour
- Peak-hour and heaviest-window (morning/afternoon/evening/night) summary

### Weekly (7 days ending on the selected date)
- **FTND and PS-ECDI** shown as numbers with **↑/↓ arrows vs the previous assessment** (a single assessment is labelled "first assessment")
- Total puffs, daily average, **average vs ceiling %** (heat-coloured), on-target days
- **Daily puffs bar chart** overlaid with both the **ceiling** and the **challenge** line
- Puffs-per-hour line chart for the week
- Comparison vs the previous week

### Monthly (28 days ending on the selected date)
- **FTND and PS-ECDI trend line** (dual axis) from all assessments in the window
- Total puffs, daily average, **average vs ceiling %** (heat-coloured), zero-puff days, on-target days
- Daily puffs trend line vs the ceiling
- Puffs-per-hour line chart for the month
- Comparison vs the previous 28 days

> All ceilings, challenges, and adherence figures are computed **per day**, using the values that applied on that specific day — so historical reports stay accurate even as the ceiling steps down over time.

> **Sparse data:** Trend lines only look like lines once several days / multiple assessments exist. With a single data point (e.g. on your first day) a chart shows a single dot rather than a line — this is expected, not a bug.

---

## Date picker (historical viewing)

A date field under the report tabs lets you view any past **Daily**, **Weekly**, or **Monthly** report:

- **Daily** → that specific day
- **Weekly** → the 7 days ending on that date
- **Monthly** → the 28 days ending on that date

A **Today** button jumps back to the present, and logging a new puff automatically snaps the view back to today. The picker can't select future dates or dates before the program start.

---

## Heat-coloured percentages

Any "% of ceiling" or "% of challenge" figure is colour-graded so risk is visible at a glance — it shifts from cool to hot as consumption approaches or passes the limit:

| % of target | Colour |
|---|---|
| 0% (none) | green |
| ≤40% | lime |
| ≤60% | yellow |
| ≤80% | amber |
| ≤99% | orange |
| 100–120% | red |
| >120% | dark red |

---

## Mood & notes

- **Overall mood today** — a 5-point emoji scale (Awful → Great) capturing how you feel about the **day as a whole**. This is deliberately distinct from the 0–10 craving rating inside the CBT loop; the UI labels and a helper line make the difference explicit so the two aren't confused.
- **Clinical notes / triggers** — a free-text field for recording cravings, triggers, or withdrawal symptoms. This is the user's own field; CBT loops are kept separate and do **not** write here.

Both are stored per day and synced to Google Sheets.

---

## Evening reminder

An optional daily check-in (**21:30**) nudges you to log the day's mood and notes if you haven't yet. When enabled it can raise a browser notification (with permission) and shows an in-app banner. It never fires once you've already logged mood or a note for the day, and it can be dismissed for the day.

---

## Design philosophy

The ceiling is a **hard limit, not a target to fill**. The app is deliberately quit-oriented:

- Baselines are set **far below real-world use**
- In-app messaging never says "you have puffs left to use." It uses a **health-warning + challenge** tone — reminding the user that every avoided puff spares the lungs more nicotine and toxicants, and challenging them to beat the daily target
- The pacing timer, CBT loop, and coping library shift the app from passive counting toward **active craving management** — spacing puffs out, decoupling triggers, and defusing urges in the moment
- Percentages turn hotter as consumption nears or passes the limit
- The ceiling steps down to **0 (quit)** in the final week
- When behavioural reduction alone stalls, in-app feedback points to clinical practice guidelines (USPSTF 2021; German S3 2022) recommending approved cessation medication plus counselling

---

## Google Sheets sync

The app talks to a Google Apps Script (GAS) web app bound to a Google Sheet.

- Set `GAS_URL` in `index.html` to your GAS deployment URL
- The `logs` sheet uses the columns: `log_id`, `timestamp` (ISO / UTC), `date` (`YYYY-MM-DD`), `time` (`HH:MM:SS`, local GMT+7)

**Data flow:**

- **On app open** — `doGet` returns JSON (`logs`, `assessments`, `config`, `mood`, `notes`); the app counts daily puffs from `logs`
- **On each logged puff (incl. +3 / +5)** — `doPost` writes a new log row immediately, one per puff
- **Also synced** — assessments, mood/notes, and program settings. CBT loops are posted separately to their own `cbt` sheet (see `apps_script_cbt_addon.md`).

**Dates:** handled as `YYYY-MM-DD` strings throughout — for the sheet's `date`, `timestamp`, and `config.startDate`, and when comparing assessment dates to the report anchor. This string-based comparison avoids timezone drift (a UTC `Date` can land on the wrong local day), which otherwise made counts and dependence values disappear.

> **Timezone caveat:** `timestamp` is UTC. Used in the evening (GMT+07:00), the date derived from UTC may roll over to the next day. To anchor strictly to local (Thai) time, use the `date` column from GAS instead.

**Offline:** all data is mirrored in `localStorage` (key `clinical_vape_tracker_v4`), so the app keeps working without a connection and re-syncs when it returns. The local-only `cbtHistory` structure is preserved across syncs.

---

## Installation (GitHub Pages)

1. Place `index.html` at the root of your repository
2. Go to **Settings → Pages**, select a branch (e.g. `main`) and the `/root` folder
3. Open the URL GitHub Pages generates (e.g. `https://<username>.github.io/<repo>/`)

It's a single self-contained HTML file loading Tailwind and Chart.js via CDN — no build step.

> **After pushing an update:** GitHub Pages may serve a cached copy. Wait for the deploy to finish, then hard-refresh or append a cache-buster to the URL (e.g. `?v=18`).

> **Note on baselines:** ceiling baselines (24 / 42 / 60 / 78) are computed **client-side** from the FTND/ECDI scores; they are not stored in the Sheet. Updating the app code takes effect on reload — no need to re-take the assessment.

---

## References

The full reference list (Vancouver style) is in the app's **Disclaimer & References** section. **Every citation has been verified against PubMed** — both that the article exists and that the app's summary of it is accurate. Coverage:

- **Dependence assessment instruments** — FTND (Heatherton 1991), Penn State ECDI (Foulds 2015)
- **Consumption / frequency benchmarks** — EMIT study (Tillery 2023, POD ~123 puffs/day), Yingst 2020, Kosmider 2018 (~156 puffs/day), Aherrera 2020 (~365 puffs/day, MOD/tank users)
- **Tapering strategy & program length** — Cochrane review (Lindson 2019), Lindson-Hawley 2016, Skelton 2022 (gradual arm: 25% reduction over 4 weeks), RedPharm (Farley 2017; 4- vs 16-week reduction, participants stayed ~6 weeks on average), Cinciripini 2023
- **Craving management** — the interval-extension, urge-surfing, stimulus-control, and trigger-decoupling strategies draw on established CBT for smoking cessation (the "4 Ds" and relapse-prevention literature)
- **Clinical practice guidelines** — USPSTF 2021 (JAMA), German S3 Tobacco Guideline (Batra 2022)

> Real-world usage figures describe actual behaviour, not safe limits. The ceiling values, daily-challenge formula, linear taper schedule, and the interval/binge thresholds are pragmatic design choices informed by the literature above, not validated dosing protocols.

---

## Stack

- HTML + Tailwind CSS (CDN)
- Chart.js (CDN)
- Google Apps Script + Google Sheets (backend / sync)
- localStorage (offline on-device storage)
