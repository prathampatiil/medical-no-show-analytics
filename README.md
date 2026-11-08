# Healthcare Appointment No-Show Analysis

End-to-end SQL analysis of 110K+ medical appointments: data cleaning, no-show driver analysis, and a patient-level risk-scoring view built with window functions.

## Problem Statement

No-shows waste clinic slots and revenue. This project asks:

- What share of appointments end in a no-show?
- Which factors (lead time, reminders, age, day of week, neighbourhood) are linked to missed appointments?
- Can we flag high-risk appointments *before* they happen?

## Dataset

- **Source:** Medical Appointment No Shows dataset (Kaggle), file `Medical_Appointment.csv`
- **Size:** 110K+ appointments
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

## Patient-Level Risk Scoring

The view `v_appointment_risk` uses a CTE and window functions to compute, for each appointment, the patient's **prior appointments** and **prior no-shows** (using `ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING` so the current appointment never leaks into its own score). It then assigns a tier:

| Tier | Rule |
|---|---|
| New Patient - Monitor | No prior appointments |
| High Risk | Prior no-show rate >= 50% **or** lead time >= 8 days |
| Medium Risk | Prior no-show rate >= 20% **or** lead time of 4-7 days |
| Low Risk | Everything else |

## Key Findings

> Replace the placeholders below with the numbers from your query results.

- Overall no-show rate: **X%**
- Appointments booked 8+ days ahead had a **X%** no-show rate vs **Y%** for same-day appointments
- Highest-risk day of week: **___** (X%)
- Highest-risk age group: **___** (X%)
- Patients who received an SMS had a **X%** no-show rate vs **Y%** for those who did not
- Highest-risk neighbourhood (100+ appointments): **___** (X%)
- **Z** appointments flagged High Risk by the scoring view

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

