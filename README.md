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
var SHEET_LOGS = 'logs';
var SHEET_ASSESS = 'assessments';
var SHEET_CONFIG = 'config';
var SHEET_META = 'meta';

function _ss() { return SpreadsheetApp.getActiveSpreadsheet(); }

function _getOrCreateSheet(name, headers) {
  var ss = _ss();
  var sh = ss.getSheetByName(name);
  if (!sh) {
    sh = ss.insertSheet(name);
    sh.appendRow(headers);
  }
  return sh;
}

function _ensureSheets() {
  _getOrCreateSheet(SHEET_LOGS, ['timestamp', 'date']);
  _getOrCreateSheet(SHEET_ASSESS, ['assessKey', 'ftnd', 'ecdi', 'date']);
  _getOrCreateSheet(SHEET_CONFIG, ['key', 'value']);
  _getOrCreateSheet(SHEET_META, ['date', 'mood', 'note']);
}

function doGet(e) {
  _ensureSheets();
  var out = { logs: [], assessments: {}, config: {}, mood: {}, notes: {} };
  
  var shLogs = _ss().getSheetByName(SHEET_LOGS).getDataRange().getValues();
  for (var i = 1; i < shLogs.length; i++) out.logs.push({ ts: String(shLogs[i][0]), date: String(shLogs[i][1]) });

  var shA = _ss().getSheetByName(SHEET_ASSESS).getDataRange().getValues();
  for (var j = 1; j < shA.length; j++) out.assessments[String(shA[j][0])] = { ftnd: shA[j][1], ecdi: shA[j][2], date: String(shA[j][3]) };

  var shC = _ss().getSheetByName(SHEET_CONFIG).getDataRange().getValues();
  for (var k = 1; k < shC.length; k++) out.config[String(shC[k][0])] = String(shC[k][1]);

  var shM = _ss().getSheetByName(SHEET_META).getDataRange().getValues();
  for (var m = 1; m < shM.length; m++) {
    out.mood[String(shM[m][0])] = String(shM[m][1]);
    out.notes[String(shM[m][0])] = String(shM[m][2]);
  }

  return ContentService.createTextOutput(JSON.stringify(out)).setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  _ensureSheets();
  var p = e.parameter;
  var action = p.action;
  
  try {
    if (action === 'log') _ss().getSheetByName(SHEET_LOGS).appendRow([p.ts, p.date]);
    else if (action === 'undo') {
      var sh = _ss().getSheetByName(SHEET_LOGS);
      var vals = sh.getDataRange().getValues();
      for (var i = vals.length - 1; i >= 1; i--) {
        if (String(vals[i][1]) === String(p.date)) { sh.deleteRow(i + 1); break; }
      }
    }
    return ContentService.createTextOutput(JSON.stringify({status: 'success'})).setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({status: 'error', message: err.toString()})).setMimeType(ContentService.MimeType.JSON);
  }
}
```

### 2. Frontend Setup
1. Copy the `index.html` file into your repository.
2. Update the configuration variable `const GAS_URL` with your unique Web App URL from the deployment step above.
3. Host the file via **GitHub Pages**.
4. Access the site on iOS Safari and use **"Add to Home Screen"** for an app-like experience.

## 🧠 Clinical Rationale & References

This protocol operates on the premise that pre-filled closed-system devices cannot be diluted. Therefore, dependence reduction relies strictly on modifying behavioral frequency, interval extension, and trigger decoupling.

1) Heatherton TF, et al. The Fagerström Test for Nicotine Dependence: a revision of the Fagerström Tolerance Questionnaire. Br J Addict. 1991.
2) Foulds J, et al. Development of a questionnaire for assessing dependence on electronic cigarettes. Nicotine Tob Res. 2015.
3) Tillery A, et al. Characterization of e-cigarette users according to device type, use behaviors... Tob Induc Dis. 2023.
4) Lindson N, et al. Smoking reduction interventions for smoking cessation. Cochrane Database Syst Rev. 2019.
5) US Preventive Services Task Force. Interventions for tobacco smoking cessation in adults. JAMA. 2021.
6) Batra A, et al. S3 guideline "Smoking and tobacco dependence: screening, diagnosis, and treatment". Eur Addict Res. 2022..

## ⚠️ Disclaimer

This software is provided as a data collection and structural tracking tool. It does not constitute diagnostic, psychiatric, or formal medical advice.
