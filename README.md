# Healthcare Appointment No-Show Analysis

End-to-end SQL analysis of 110,521 medical appointments: data cleaning, no-show driver analysis, and a patient-level risk-scoring view built with window functions.

## Problem Statement

No-shows waste clinic slots and revenue. This project asks:

- What share of appointments end in a no-show?
- Which factors (lead time, reminders, age, day of week, neighbourhood) are linked to missed appointments?
- Can we flag high-risk appointments *before* they happen?

## Dataset

- **Source:** Medical Appointment No Shows dataset (Kaggle), file `Medical_Appointment.csv`
- **Size:** 110,521 appointments from 62,298 unique patients (after cleaning)
- **Key columns:** `PatientId`, `AppointmentID`, `ScheduledDay`, `AppointmentDay`, `Age`, `Neighbourhood`, `Scholarship`, `Hipertension`, `Diabetes`, `Alcoholism`, `Handcap`, `SMS_received`, `No-show`

## Data Cleaning

| Step | What was done |
|---|---|
| Column names | Renamed `Hipertension` to `hypertension`, `Handcap` to `disability_count`, `No-show` to `no_show` |
| Date formats | Converted ISO strings in `ScheduledDay` and `AppointmentDay` into proper `DATETIME` / `DATE` columns |
| Invalid ages | Removed rows with `Age = -1` |
| Invalid lead times | Removed rows where the appointment date was before the scheduled date |
| New feature | Added `lead_time_days` (days between scheduling and appointment) |

## Questions Answered

1. Overall no-show rate
2. No-show rate by day of week
3. No-show rate by lead-time bucket (same day, 1-3 days, 4-7 days, 8+ days)
4. No-show rate by age group (child, teen, young adult, adult, senior)
5. Do SMS reminders reduce no-shows?
6. Top 15 highest-risk neighbourhoods (minimum 100 appointments)

## Key Findings

| Factor | Finding |
|---|---|
| **Overall** | **20.19%** of appointments are no-shows (22,314 of 110,521), about 1 in 5 |
| **Lead time** | Strongest driver. No-show rate rises from **4.65%** (same day) to 22.89% (1-3 days), 25.20% (4-7 days) and **32.06%** (8+ days), about 7x higher for 8+ day bookings than same-day |
| **Age** | Teens (13-19) have the highest rate at **25.95%**; seniors (60+) the lowest at **15.31%** |
| **Day of week** | Weak predictor. Friday is highest (21.23%) and Thursday lowest (19.35%). Saturday (23.08%) has only 39 appointments, so it is excluded from conclusions |
| **SMS reminders** | Patients who received an SMS had a **27.57%** no-show rate vs **16.70%** for those who did not. This is likely confounded by lead time, since reminders are mostly sent for appointments booked further ahead. TODO: add the result of the lead-time-controlled comparison |
| **Neighbourhood** | Highest rate: **Santos Dumont** (28.92%). Itarare (26.27%) has the most appointments among the top-ranked areas (3,514). Rates among the top 15 range from 22.28% to 28.92% |
| **Patient history** | 56% of appointments are a patient's first, so no history is available. Among returning patients, those with a prior no-show rate of 50% or more had a **33.43%** no-show rate vs **17.28%** for those under 20% |

**Caveats:** all findings are associations, not causes. Same-day appointments are often urgent visits, and the lead-time effect may partly explain the SMS and neighbourhood results.

## Patient-Level Risk Scoring

The view `v_appointment_risk` uses a CTE and window functions to compute, for each appointment, the patient's **prior appointments** and **prior no-shows** (using `ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING` so the current appointment never leaks into its own score). Ties on the same appointment day are broken by `AppointmentID`. It then assigns a tier:

| Tier | Rule |
|---|---|
| New Patient - Monitor | No prior appointments |
| High Risk | Prior no-show rate >= 50% **or** lead time >= 8 days |
| Medium Risk | Prior no-show rate >= 20% **or** lead time of 4-7 days |
| Low Risk | Everything else |

The thresholds were checked against observed outcomes: longer lead times and higher prior no-show rates both corresponded to higher no-show rates.

**Tier counts:** TODO (add High / Medium / Low / New Patient counts from the view)

## Tech Stack

- MySQL (DDL/DML, `STR_TO_DATE`, `DATEDIFF`)
- Window functions (`RANK`, `COUNT/SUM OVER`), CTEs, views
- Tableau (in progress)

## Dashboards

Tableau dashboards coming soon. The Tableau Public link and screenshots will be added here.

## Repository Structure

```
.
├── README.md
├── data/
│   └── Medical_Appointment.csv
├── sql/
│   ├── 01_cleaning.sql
│   ├── 02_exploration.sql
│   └── 03_risk_scoring_view.sql
├── dashboards/
│   └── README.md
└── images/
```

## How to Run

1. Create the database: `CREATE DATABASE healthcare_analytics;`
2. Import `Medical_Appointment.csv` into a table named `medicalappointment` (MySQL Workbench: Table Data Import Wizard).
3. Run the scripts in order:
   - `sql/01_cleaning.sql`
   - `sql/02_exploration.sql`
   - `sql/03_risk_scoring_view.sql`
4. Query the view: `SELECT * FROM v_appointment_risk WHERE risk_tier = 'High Risk' LIMIT 50;`
