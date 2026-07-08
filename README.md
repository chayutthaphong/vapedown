# Step Down: Clinical Tapering Protocol & Longitudinal Tracker

A robust, web-based clinical tracking application designed to systematically monitor and manage Electronic Nicotine Delivery Systems (ENDS) tapering. This tool utilizes Cognitive Behavioral Therapy (CBT) principles and dynamic benchmarking to provide a structured 8-week step-down protocol, complete with automated data synchronization for longitudinal statistical analysis.

## 🌟 Core Clinical Features

* **Dual-Protocol Assessment:** Integrates both the **FTND-V (Vaping Adapted Standard)** and the **ECDI (Penn State Electronic Cigarette Dependence Index)**. Users can choose their preferred clinical protocol at the start of each week to recalibrate the tapering threshold.
* **Dynamic Tapering Algorithm:** Automatically calculates and adjusts daily puff benchmarks based on the clinical score obtained from the chosen assessment protocol.
* **Heatmap Analytics:** Features a 7-day longitudinal tracking chart with dynamic heatmap color-coding (from yellow to dark red) to visually represent adherence severity relative to the daily benchmark.
* **Comprehensive Clinical Reports (Weekly & Monthly):** Automatically generates end-of-week and end-of-month summaries. These reports calculate total puff volumes, evaluate adherence, and provide structured clinical feedback with synchronized visual cues.
* **Independent Mood & Trigger Tracking:** Features optional daily affective state logging (via emojis) and a dedicated text input for users to qualitatively record daily triggers, cravings, or withdrawal symptoms.
* **Relapse Management:** Automatically switches to a "Relapse" protocol once the daily target reaches zero, distinctively tagging post-abstinence usage for accurate survival analysis.
* **Cloud Data Synchronization:** Seamlessly logs multivariate data points directly into Google Sheets in real-time, creating a clean, structured dataset ready for advanced statistical evaluation.

## 📊 Dataset Structure (Google Sheets)

To utilize the automated cloud backup, set up a Google Sheet with the following headers in Row 1 (Columns A through I):

| Column | Header | Description |
| :--- | :--- | :--- |
| A | `Date` | Local date of the logged event (YYYY-MM-DD). |
| B | `Time` | Local time of the logged event (HH:MM:SS). |
| C | `Puff Count` | Cumulative number of puffs taken on that specific date. |
| D | `Daily Target` | The dynamically calculated benchmark for the day. |
| E | `Status` | Adherence status (`On Track`, `Over Limit`, or `Relapse`). |
| F | `Week` | The current active week of the 8-week protocol. |
| G | `FTND Score` | The score (0-10 or 0-20) from the chosen protocol. |
| H | `Mood` | The self-reported affective state. |
| I | `Notes` | Qualitative data regarding triggers, cravings, or context. |

## 🚀 Deployment Guide

### 1. Backend Setup (Google Apps Script)
1. Create a new Google Sheet matching the structure above.
2. Navigate to **Extensions > Apps Script**.
3. Deploy the following script as a **Web App** (Access: Anyone):

```javascript
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  sheet.appendRow([
    e.parameter.date, e.parameter.time, e.parameter.count, 
    e.parameter.target, e.parameter.status, e.parameter.week, 
    e.parameter.ftnd, e.parameter.mood, e.parameter.note
  ]);
  return ContentService.createTextOutput("Success").setMimeType(ContentService.MimeType.TEXT);
}
```

### 2. Frontend Setup
1. Copy the `index.html` file into your repository.
2. Update the configuration variable `const GAS_URL` with your unique Web App URL from the deployment step above.
3. Host the file via **GitHub Pages**.
4. Access the site on iOS Safari and use **"Add to Home Screen"** for an app-like experience.

## 🧠 Clinical Rationale & References

This protocol operates on the premise that pre-filled closed-system devices cannot be diluted. Therefore, dependence reduction relies strictly on modifying behavioral frequency, interval extension, and trigger decoupling.

* Fagerström, K. (2012). Determinants of tobacco use and renaming the FTND to the Fagerström Test for Cigarette Dependence. *Tobacco Control*, 21(1), 9-14.
* Foulds, J., et al. (2015). Development of a questionnaire for assessing dependence on electronic cigarettes. *Nicotine & Tobacco Research*, 17(2), 186-192.
* Truth Initiative. (2021). *Actionable strategies for e-cigarette cessation: Tapering and behavioral modification*.

## ⚠️ Disclaimer

This software is provided as a data collection and structural tracking tool. It does not constitute diagnostic, psychiatric, or formal medical advice.
