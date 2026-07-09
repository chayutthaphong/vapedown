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
* **Longitudinal Tracking & Analytics:** Visualizes progress with a 7-day adherence bar chart and a comprehensive 28-day monthly review (identifying reduction trends and adherence quality).
* **Trigger & Mood Journaling:** Allows users to log their emotional states and specific cravings/triggers alongside their puff data.
* **Offline-First & Cloud Sync:** Puffs are instantly logged to the device's Local Storage (preventing data loss during offline periods) and silently synced to Google Sheets in the background. Each puff is assigned a Unique ID to prevent duplicate logs.
* **Auto-Healing System:** Built-in date parsing safeguards ensure that iOS/Safari users do not encounter the classic `NaN` (Not a Number) bug.

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

*(Note: If you have old data containing `NaN` in your Google Sheet, please delete those specific rows before using the app to ensure accurate calculations.)*

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
   const GAS_URL = "YOUR_WEB_APP_URL_HERE";
