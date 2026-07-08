# Step Down: Vape Tapering Tracker

A minimalist, web-based tracking application designed to help users systematically taper their electronic nicotine delivery systems (ENDS) usage. Built with a focus on cognitive behavioral principles, this tool provides a structured 8-week protocol for reducing nicotine dependence, specifically modeled for closed-system devices (e.g., 3% or 30mg/mL nicotine).

## 🌟 Features

*   **Clinical Tapering Strategy:** Integrated 8-week step-down protocol utilizing interval extension, puff limitation, and trigger decoupling.
*   **Minimalist Interface:** Clean, distraction-free UI built with Tailwind CSS to keep the focus on behavioral control.
*   **Dynamic Analytics:** Real-time 7-day historical data visualization (via Chart.js) comparing actual usage against daily benchmarks.
*   **Cloud Synchronization:** Automated backend logging to Google Sheets via Google Apps Script for secure, long-term data retention and personalized analysis.
*   **Adaptive Feedback:** Contextual motivational messaging based on daily progress and benchmark limits.

## 🧠 Protocol Basis

Because dilution is impossible in pre-filled disposable pods, this application relies on modifying user behavior to systematically reduce plasma nicotine concentration and manage withdrawal symptoms. The protocol is divided into four distinct phases:

1.  **Weeks 1-2 (Baseline & Interval Extension):** Establishing a baseline and delaying the first puff of the day.
2.  **Weeks 3-4 (Dose & Frequency Decoupling):** Extending intervals between sessions and limiting puffs per session.
3.  **Weeks 5-6 (Strict Scheduled Dosing):** Restricting usage to specific, predetermined times (e.g., post-meal) to break automatic behavioral triggers.
4.  **Weeks 7-8 (SOS Mode to Zero):** Extreme limitation reserved only for severe cravings, culminating in a complete cessation date.

## 🚀 Getting Started

### Prerequisites
*   A modern web browser (Safari, Chrome, etc.)
*   A Google account (for setting up the Google Sheets database)

### Installation & Setup

1.  **Clone or Fork the Repository:**
    Copy the `index.html` file to your own environment or host it directly via [GitHub Pages](https://pages.github.com/).

2.  **Set Up the Google Sheets Database:**
    *   Create a new Google Sheet.
    *   Name the columns in Row 1: `Date`, `Time`, `Puff Count`, `Daily Target`, and `Status`.
    *   Navigate to **Extensions > Apps Script** and deploy the following backend script as a Web App:

    ```javascript
    function doPost(e) {
      var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
      var date = e.parameter.date;
      var time = e.parameter.time;
      var count = e.parameter.count;
      var target = e.parameter.target;
      var status = e.parameter.status;
      
      sheet.appendRow([date, time, count, target, status]);
      return ContentService.createTextOutput("Success").setMimeType(ContentService.MimeType.TEXT);
    }
    ```

3.  **Connect the App:**
    *   Copy the Web App URL generated from the Apps Script deployment.
    *   Open `index.html` and replace `YOUR_WEB_APP_URL_HERE` with your actual URL.
    *   Save and commit the changes.

4.  **Mobile Usage:**
    For the best experience, open the hosted URL in your mobile browser and select **"Add to Home Screen"** to use it as a native web app.

## ⚠️ Disclaimer

This application provides a structural framework for nicotine tapering and is intended for personal tracking and data collection. It does not constitute formal medical advice, diagnosis, or behavioral therapy. Users should consult healthcare professionals for personalized medical guidance regarding smoking cessation.

## 📚 References

The methodologies implemented in this tracker are inspired by behavioral cessation strategies documented in:
*   Lindson, N., et al. (2019). Gradual versus abrupt quitting for smokers. *Cochrane Database of Systematic Reviews*.
*   Truth Initiative. (2021). Actionable strategies for e-cigarette cessation: Tapering and behavioral modification.
*   Foulds, J., et al. (2022). Effect of electronic nicotine delivery systems on quitting. *JAMA Network Open*.
