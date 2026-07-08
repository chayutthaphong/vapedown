# Step Down: Clinical Tapering Protocol & Longitudinal Tracker

A robust, web-based clinical tracking application designed to systematically monitor and manage Electronic Nicotine Delivery Systems (ENDS) tapering. This tool utilizes Cognitive Behavioral Therapy (CBT) principles and dynamic benchmarking to provide a structured 8-week step-down protocol, complete with automated data synchronization for longitudinal statistical analysis.

## 🌟 Core Clinical Features

*   **Automated FTND-V Assessment:** Integrates a vaping-adapted Fagerström Test for Nicotine Dependence (FTND-V) at the beginning of each week to evaluate physiological dependence.
*   **Dynamic Tapering Algorithm:** Automatically calculates and adjusts daily puff benchmarks based on the user's weekly FTND-V score (High, Moderate, Low dependence).
*   **Independent Mood Tracking:** Allows for daily affective state logging using visual indicators (emojis), completely independent of vaping events, ensuring continuous psychological monitoring even during the abstinence phase (Week 8+).
*   **Relapse Management & Tracking:** Automatically switches to a "Relapse" protocol once the daily target reaches zero, distinctively tagging post-abstinence usage for accurate survival analysis.
*   **Cloud Data Synchronization:** Seamlessly logs multivariate data points directly into Google Sheets in real-time, creating a clean, structured dataset ready for advanced statistical evaluation.

## 📊 Dataset Structure (Google Sheets)

To utilize the automated cloud backup, set up a Google Sheet with the following headers in Row 1 (Columns A through H):

| Column | Header | Description |
| :--- | :--- | :--- |
| A | `Date` | Local date of the logged event (YYYY-MM-DD). |
| B | `Time` | Local time of the logged event (HH:MM:SS). |
| C | `Puff Count` | Cumulative number of puffs taken on that specific date. |
| D | `Daily Target` | The dynamically calculated benchmark for the day. |
| E | `Status` | Adherence status (`On Track`, `Over Limit`, or `Relapse`). |
| F | `Week` | The current active week of the 8-week protocol. |
| G | `FTND Score` | The most recent Fagerström score (0-10). |
| H | `Mood` | The self-reported affective state (`Awful`, `Bad`, `Neutral`, `Good`, `Great`). |

## 🚀 Deployment Guide

### 1. Backend Setup (Google Apps Script)
1. Create a new Google Sheet matching the structure above.
2. Navigate to **Extensions > Apps Script**.
3. Deploy the following script as a **Web App** (Access: Anyone):

```javascript
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  
  var date = e.parameter.date;
  var time = e.parameter.time;
  var count = e.parameter.count;
  var target = e.parameter.target;
  var status = e.parameter.status;
  var week = e.parameter.week || "-";
  var ftnd = e.parameter.ftnd || "-";
  var mood = e.parameter.mood || "Not Recorded";
  
  sheet.appendRow([date, time, count, target, status, week, ftnd, mood]);
  
  return ContentService.createTextOutput("Success").setMimeType(ContentService.MimeType.TEXT);
}
```

### 2. Frontend Setup
1. Copy the provided `index.html` file into your repository.
2. Locate the configuration section in the script (around line 260):
   `const GAS_URL = "YOUR_WEB_APP_URL_HERE";`
3. Replace the placeholder with your active Google Apps Script Web App URL.
4. Host the file via **GitHub Pages** (or any static hosting service).
5. Open the hosted URL on a mobile device and select **"Add to Home Screen"** for optimal UI rendering.

## 🧠 Clinical Rationale & References

This protocol operates on the premise that pre-filled closed-system devices (e.g., 3% or 30mg/mL Nicotine) cannot be diluted. Therefore, dependence reduction relies strictly on modifying behavioral frequency, extending inter-puff intervals, and decoupling environmental triggers based on established CBT methodologies.

*   Fagerström, K. (2012). Determinants of tobacco use and renaming the FTND to the Fagerström Test for Cigarette Dependence. *Tobacco Control*, 21(1), 9-14.
*   Foulds, J., Veldheer, S., Yingst, J., Hrabovsky, S., Wilson, S. J., Nichols, T. T., & Eissenberg, T. (2015). Development of a questionnaire for assessing dependence on electronic cigarettes among a large sample of ex-smoking E-cigarette users. *Nicotine & Tobacco Research*, 17(2), 186-192.
*   Truth Initiative. (2021). *Actionable strategies for e-cigarette cessation: Tapering and behavioral modification*.

## ⚠️ Disclaimer

This software is provided as a data collection and structural tracking tool. It does not constitute diagnostic, psychiatric, or formal medical advice.
