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

⚠️ Medical Disclaimer
This application provides a structural framework for nicotine tapering and is intended for personal data collection and longitudinal tracking. It does not constitute formal medical advice, diagnosis, or behavioral therapy. When self-directed reduction is not enough, clinical practice guidelines (e.g., USPSTF 2021; German S3 Tobacco Guideline 2022) recommend seeing a clinician/pharmacist for behavioural support plus approved cessation medication. If you experience severe withdrawal symptoms, please consult healthcare professionals for personalized guidance.

📚 References (Vancouver style)
	1.	Heatherton TF, Kozlowski LT, Frecker RC, Fagerström KO. The Fagerström Test for Nicotine Dependence: a revision of the Fagerström Tolerance Questionnaire. Br J Addict. 1991;86(9):1119–27. doi:10.1111/j.1360-0443.1991.tb01879.x
	2.	Foulds J, Veldheer S, Yingst J, Hrabovsky S, Wilson SJ, Nichols TT, et al. Development of a questionnaire for assessing dependence on electronic cigarettes among a large sample of ex-smoking E-cigarette users. Nicotine Tob Res. 2015;17(2):186–92. doi:10.1093/ntr/ntu204
	3.	Yingst J, Foulds J, Veldheer S, Cobb CO, Yen MS, Hrabovsky S, et al. Measurement of electronic cigarette frequency of use among smokers participating in a randomized controlled trial. Nicotine Tob Res. 2020;22(5):699–704. doi:10.1093/ntr/nty233
	4.	Tillery A, Aherrera A, Chen R, Lin JJY, Tehrani M, Moustafa D, et al. Characterization of e-cigarette users according to device type, use behaviors, and self-reported health outcomes: findings from the EMIT study. Tob Induc Dis. 2023;21:159. doi:10.18332/tid/174710
	5.	Kosmider L, Jackson A, Leigh N, O’Connor R, Goniewicz ML. Circadian puffing behavior and topography among e-cigarette users. Tob Regul Sci. 2018;4(5):41–9. doi:10.18001/TRS.4.5.4
	6.	Aherrera A, Aravindakshan A, Jarmul S, Olmedo P, Chen R, Cohen JE, et al. E-cigarette use behaviors and device characteristics of daily exclusive e-cigarette users in Maryland: implications for product toxicity. Tob Induc Dis. 2020;18:93. doi:10.18332/tid/128319
	7.	Lindson N, Klemperer E, Hong B, Ordóñez-Mena JM, Aveyard P. Smoking reduction interventions for smoking cessation. Cochrane Database Syst Rev. 2019;9(9):CD013183. doi:10.1002/14651858.CD013183.pub2
	8.	Lindson-Hawley N, Banting M, West R, Michie S, Shinkins B, Aveyard P. Gradual versus abrupt smoking cessation: a randomized, controlled noninferiority trial. Ann Intern Med. 2016;164(9):585–92. doi:10.7326/M14-2805
	9.	Skelton E, Lum A, Robinson M, Dunlop A, Guillaumier A, Baker A, et al. A pilot randomised controlled trial of abrupt versus gradual smoking cessation in combination with vaporised nicotine products for people receiving alcohol and other drug treatment. Addict Behav. 2022;131:107328. doi:10.1016/j.addbeh.2022.107328
	10.	Farley A, Tearne S, Taskila T, Williams RH, MacAskill S, Etter JF, et al. A mixed methods feasibility study of nicotine-assisted smoking reduction programmes delivered by community pharmacists — the RedPharm study. BMC Public Health. 2017;17(1):210. doi:10.1186/s12889-017-4116-z
	11.	Cinciripini PM, Minnix JA, Robinson JD, Kypriotakis G, Cui Y, Blalock JA, et al. The effects of scheduled smoking reduction and precessation nicotine replacement therapy on smoking cessation: randomized controlled trial with compliance. JMIR Form Res. 2023;7:e39487. doi:10.2196/39487
	12.	US Preventive Services Task Force; Krist AH, Davidson KW, Mangione CM, Barry MJ, Cabana M, et al. Interventions for tobacco smoking cessation in adults, including pregnant persons: US Preventive Services Task Force recommendation statement. JAMA. 2021;325(3):265–79. doi:10.1001/jama.2020.25019
	13.	Batra A, Kiefer F, Andreas S, Gohlke H, Klein M, Kotz D, et al. S3 guideline "Smoking and tobacco dependence: screening, diagnosis, and treatment" — short version. Eur Addict Res. 2022;28(5):382–400. doi:10.1159/000525265

📄 License
This project is licensed under the MIT License.
