# Step Down - Clinical Protocol Vape Tracker

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Mobile First](https://img.shields.io/badge/UI-Mobile_First-brightgreen.svg)]()
[![Backend](https://img.shields.io/badge/Backend-Google_Apps_Script-yellow.svg)]()

**Step Down** is a mobile-first web application designed to facilitate structured e-cigarette tapering based on clinical protocols (CBT principles). It tracks nicotine dependence and calculates a dynamic, personalized daily puff ceiling to help users gradually step down their consumption to zero.

## ✨ Key Features

* **Clinical Assessments:** Evaluates nicotine dependence weekly using standard instruments:
  * *Fagerström Test for Nicotine Dependence (FTND)*
  * *Penn State E-Cigarette Dependence Index (PS-ECDI)*
* **Dynamic Tapering Algorithm:** Automatically calculates a daily puff ceiling based on the user's dependence scores and selected program length (4, 6, or 8 weeks).
* **Longitudinal Tracking & Analytics:** Visualizes progress with a 7-day adherence bar chart and a 28-day monthly review line chart.
* **Trigger & Mood Journaling:** Allows users to log their emotional states and specific cravings/triggers alongside their puff data.
* **Offline-First & Cloud Sync:** Puffs are instantly logged to the device's Local Storage (preventing data loss during offline periods) and silently synced to Google Sheets in the background.

## 🛠️ Technology Stack

* **Frontend:** HTML5, Vanilla JavaScript
* **Styling:** Tailwind CSS (via CDN)
* **Data Visualization:** Chart.js
* **Backend / Database:** Google Apps Script (GAS) & Google Sheets

## ⚙️ Installation & Setup

To deploy your own instance of Step Down, you need to set up a Google Sheet as the database and update the frontend configuration.

### Step 1: Database Setup (Google Sheets)
1. Create a new Google Sheet.
2. Create **4 specific tabs (sheets)** exactly as named below, and populate the **first row** of each with these exact headers:
   * Sheet Name: `logs`
     * Headers: `A1: log_id` | `B1: timestamp` | `C1: date`
   * Sheet Name: `assessments`
     * Headers: `A1: assessKey` | `B1: ftnd` | `C1: ecdi` | `D1: date`
   * Sheet Name: `config`
     * Headers: `A1: key` | `B1: value` | `C1: date`
   * Sheet Name: `meta`
     * Headers: `A1: date` | `B1: mood` | `C1: note`

### Step 2: Backend Setup (Google Apps Script)
1. In your Google Sheet, go to **Extensions > Apps Script**.
2. Replace the default `Code.gs` content with your backend script (handling `doGet` and `doPost`).
3. Click **Deploy > New deployment**.
4. Select type **Web App**.
   * *Execute as:* Me
   * *Who has access:* Anyone
5. Click **Deploy** and copy the generated **Web App URL**.

### Step 3: Frontend Setup
1. Open `index.html`.
2. Locate the configuration section in the `<script>` tag.
3. Replace the `GAS_URL` variable with the Web App URL you obtained in Step 2:
   ```javascript
   const GAS_URL = "YOUR_WEB_APP_URL_HERE";3. Deploy the following script as a **Web App** (Access: Anyone):

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
