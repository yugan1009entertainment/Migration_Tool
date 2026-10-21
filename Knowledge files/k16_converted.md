# 📘 CONVERSION KNOWLEDGE FILE
# K16 – Medical on Call Dashboard
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-23
# Source: K16 1A (Tableau TWB, revision 10.4) · K16 2B (Power BI PBIP)
# Model Name: Medical on Call - Dashboard
# PBI Compatibility Level: 1604
# Culture: en-US
# Tableau Site: PRODUCTION (tableau.bms.com)

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PARAM-01 | Start Date (Parameter 1)
─────────────────────────────────────────────────────────────
Tableau Caption       : Start Date
Tableau Type          : Date
Tableau Default       : 2025-01-01

Power BI Implementation: Date slicer / report-layer filter.
Usage in model:
  - Drives [Daterange] calculated field: Parameter 1 ≤ DATE-MOC+INT ≤ Parameter 2.

Conversion Status: ✅ FULL


### PARAM-02 | End Date (Parameter 2)
─────────────────────────────────────────────────────────────
Tableau Caption       : End Date
Tableau Type          : Date
Tableau Default       : 2025-12-31

Power BI Implementation: Date slicer / report-layer filter.
Conversion Status: ✅ FULL


### PARAM-03 | Start Date - AI (Parameter 3)
─────────────────────────────────────────────────────────────
Tableau Caption       : Start Date - AI
Tableau Type          : Date
Tableau Default       : 2025-01-01

Power BI Implementation: Date slicer for AI feedback views.
Conversion Status: ✅ FULL


### PARAM-04 | End Date - AI (Parameter 4)
─────────────────────────────────────────────────────────────
Tableau Caption       : End Date - AI
Tableau Type          : Date
Tableau Default       : 2025-12-31

Power BI Implementation: Date slicer for AI feedback views.
Conversion Status: ✅ FULL


### PARAM-05 | SFHC-Parameter (Parameter 5)
─────────────────────────────────────────────────────────────
Tableau Caption       : SFHC-Parameter
Tableau Type          : String (list)
Tableau Default       : "Login History"
Tableau Members       : "Login History", "User Presence",
                        "User Case Handle Time" (alias: UCHT)

Power BI Table Name   : Parameter Table
Power BI Type         : DAX Calculated Table (DATATABLE)

Power BI DAX:
─────────────
DATATABLE(
    "Parameter", STRING,
    "SortOrder", INTEGER,
    {
        {"Login History", 1},
        {"User Presence", 2},
        {"User Case Handle Time", 3}
    }
)

Usage in model:
  - Drives SFHC tab view toggle between Login History, User Presence,
    and User Case Handle Time views.

Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: MoC Request Interactions Data_Custom SQL Query (Fact)

### CC-01 | Region-MOC+Int
─────────────────────────────────────────────────────────────
Tableau Expression: Maps hcp_country/country codes → Germany, Italy, Japan, Puerto Rico, USA, Others
Power BI: SQL-level CASE mapping in nested query.
Conversion  : ✅ FULL

### CC-02 | DATE - MOC+INT (Unified Date)
─────────────────────────────────────────────────────────────
Tableau Expression: IF ISNULL(created_date) THEN interaction_dt ELSE created_date END
Power BI: SQL-level COALESCE(created_date, interaction_dt)
Purpose     : Unifies MoC (created_date) and Interaction (interaction_dt) dates.
Conversion  : ✅ FULL

### CC-03 | GMT-EST - FINAL (DST-Aware Timezone Conversion)
─────────────────────────────────────────────────────────────
Tableau Expression: DST-aware conversion from GMT Createddate per region
Power BI: SQL-level `adj_date` column with DST logic

DST Timezone Rules:
| Region       | Standard Offset | DST Offset | DST Period                    |
|--------------|-----------------|------------|-------------------------------|
| USA          | UTC-5 (EST)     | UTC-4 (EDT)| Mar 2nd Sun → Nov 1st Sun     |
| Germany      | UTC+1 (CET)     | UTC+2 (CEST)| Last Sun Mar → Last Sun Oct  |
| Japan        | UTC+9           | — (no DST) | Always +9                     |
| Others       | No change       | —          | Pass-through                  |

⚠️  CRITICAL: This is the primary time column for all analysis.
    Any timezone rule changes must be updated in the SQL query.
Conversion  : ✅ FULL

### CC-04 | TIME BUCKET
─────────────────────────────────────────────────────────────
Tableau Expression: Buckets GMT-EST - FINAL hour into time bands
Power BI: SQL-level CASE on hour of adj_date

| Hour Range   | Label       | Sort |
|--------------|-------------|------|
| 8 AM–12 PM   | 8 AM–12 PM  | 1    |
| 12 PM–4 PM   | 12 PM–4 PM  | 2    |
| 4 PM–8 PM    | 4 PM–8 PM   | 3    |
| Other        | Other       | 4    |

Conversion  : ✅ FULL

### CC-05 | GMT-Local Time
─────────────────────────────────────────────────────────────
Tableau Expression: HH:MM string from GMT-EST - FINAL
Power BI: SQL-level formatted string from adj_date.
Conversion  : ✅ FULL

### CC-06 | ITRTN_TA + Product
─────────────────────────────────────────────────────────────
Tableau Expression: [itrtn_ta] + '-' + [product]
Power BI: SQL COALESCE(itrtn_ta,'') || '-' || COALESCE(product,'')
Conversion  : ✅ FULL

### CC-07 | queue_skill_all_mocs
─────────────────────────────────────────────────────────────
Maps queue_skill_bms__c + hcp_country → standardized MoC queue names
(Germany MoCs, Japan MoCs, USA CV MoCs, etc.)
Conversion  : ✅ FULL

### CC-08 | interactions_cal_field
─────────────────────────────────────────────────────────────
Maps fld_team_name + country (or ITRTN_TA+Product for intl) →
interaction bucket labels (Germany CV Interactions, Japan Abecma, etc.)
Conversion  : ✅ FULL

### CC-09 | Brand/Team Name filter
─────────────────────────────────────────────────────────────
Unifies queue + interaction bucket → brand/team label.
8 segments: Germany CV, Japan Abecma, USA Cobenfy, USA CV, USA Derm,
USA Myeloid, USA HCC, USA Zeposia.
Conversion  : ✅ FULL

### CC-10 | Team Name Filter MOC+INT
─────────────────────────────────────────────────────────────
High-level team: CV, Dermatology, Neuropsychiatry, MYELOID, Zeposia,
NEUROLOGY, Cell Therapy.
Conversion  : ✅ FULL

### CC-11 | TA-Itrtn+MOC
─────────────────────────────────────────────────────────────
Unified TA across MoC + Interaction: CARDIOVASCULAR, IMMUNOLOGY,
ONCOLOGY, Neurology, HEMATOLOGY, Others.
Conversion  : ✅ FULL

### CC-12 through CC-14 | AI Feedback Freshness Columns
─────────────────────────────────────────────────────────────
| CC   | Column                     | Cutoff Date  | Logic                                    |
|------|----------------------------|--------------|------------------------------------------|
| CC-12| Feedback-AI Response       | 2025-09-22   | BMS_AI_Feedback__c or 'Data Not Refreshed'|
| CC-13| Feedback-AI Response(Copy) | 2026-01-05   | Same pattern, newer cutoff               |
| CC-14| Feedback: AI Response      | 2026-01-05   | Null-safe (returns 'Null' if null)       |

⚠️  FRESHNESS GATE: Records created AFTER the cutoff display
    'Data Not Refreshed' instead of actual feedback values.
Conversion  : ✅ FULL (all 3)

### CC-15 | Cancelled MI Name
─────────────────────────────────────────────────────────────
Tableau: IF [status]='Cancelled' THEN [mi_name] END
Power BI: SQL CASE WHEN status = 'Cancelled' THEN mi_name ELSE NULL
Conversion  : ✅ FULL

### CC-16 through CC-31 | Power Query Time Label/Sort Columns (16 pairs)
─────────────────────────────────────────────────────────────
| CC      | Label Column  | Sort Column   | Granularity | Format Example           |
|---------|---------------|---------------|-------------|--------------------------|
| CC-16/17| Year Label    | (year int)    | Year        | "2025"                   |
| CC-18/19| Quarter Label | Quarter Sort  | Quarter     | "Q1 2025" / 20251        |
| CC-20/21| Month Label   | Month Sort    | Month       | "January 2025" / 202501  |
| CC-22/23| Week Label    | Week Sort     | Week        | "January 5, 2025" / YYYYMMDD|
| CC-24/25| Day Label     | Day Sort      | Day         | "January 5, 2025" / YYYYMMDD|
| CC-26/27| Hour Label    | Hour Sort     | Hour        | "January 5, 2025 at 3 PM"|
| CC-28/29| Minute Label  | Minute Sort   | Minute      | "...at 3:30 PM"          |
| CC-30/31| Second Label  | Second Sort   | Second      | "1/5/2025 3:30:00 PM"    |

Also: Weekday Name, Weekday Sort (Monday=1), Time Bucket Sort.

### CC-32 | Year Label Hierarchy
─────────────────────────────────────────────────────────────
8-level hierarchy: Year → Quarter → Month → Week → Day → Hour → Minute → Second
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### M-01 | Avg Duration
─────────────────────────────────────────────────────────────
Tableau: { FIXED [interaction_id] : AVG([interaction_duration])}
Power BI DAX:
CALCULATE(
    AVERAGEX(VALUES([Interaction ID]), CALCULATE(AVERAGE([Interaction Duration]))),
    REMOVEFILTERS([Status (copy)])
)

Pattern     : Tableau LOD FIXED → DAX AVERAGEX(VALUES([ID]), CALCULATE(AVERAGE(...)))
NOTE        : REMOVEFILTERS on [Status (copy)] ensures metric is not
              affected by status slicers.
Conversion  : ✅ FULL

### M-02 | nuL
─────────────────────────────────────────────────────────────
Tableau: '' (empty string placeholder)
Power BI DAX: BLANK()
PBI Format  : 0
Conversion  : ✅ FULL

### M-03 | Max Date
─────────────────────────────────────────────────────────────
Power BI DAX:
CALCULATE(MAX([refresh_date]), REMOVEFILTERS('SFHC Login Information Data...'))

Purpose     : Latest refresh date, ignoring SFHC filters.
Conversion  : ✅ FULL

### M-04 | Email Action
─────────────────────────────────────────────────────────────
Power BI DAX:
"mailto:Madhuprabha.Sivakumar@bms.com;sachin.bhardwaj@bms.com?cc=Mounika.Bommareddy@bms.com&Subject=Help%20-%20Medical%20on%20Call%20Dashboard"

Purpose     : Static mailto link for dashboard help requests.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU INITIAL TRANSFORMATIONS → POWER BI POWER QUERY (M)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | MoC Request Interactions Data_Custom SQL Query (Main Fact)
─────────────────────────────────────────────────────────────
Tableau Source  : Published Data Source (SQL Proxy) — MoCRequestInteractionsData rev 2.2
PBI Connection  : PostgreSQL (imap-dl-datamart.web.bms.com:5432, postgres)
PBI Method      : 3-level nested SQL query
PBI Load Mode   : Import

SQL Architecture:
  - Level 1: SELECT from bia_sandbox.ms_fnl_moc_scrd_ads (alias a)
             FULL OUTER JOIN bia_sandbox.ms_ai_responce_msl_feedback (alias b)
             ON a.id = b."MIRF_SF_Id__c"
  - Level 2: Adds SQL-calculated columns (Region-MOC+Int, queue_skill_all_mocs,
             interactions_cal_field, Brand/Team Name filter, adj_date, TIME BUCKET, etc.)
  - Level 3: Adds Power Query post-processing (Weekday/Month/Quarter/Year labels + sorts)

Source columns: 36+ from ms_fnl_moc_scrd_ads + 28 joined from ms_ai_responce_msl_feedback.

⚠️  FULL OUTER JOIN: Retains unmatched rows from both sides. MoC records
    without AI feedback and AI feedback without MoC records both appear.
⚠️  SPELLING NOTE: "Responce" is misspelled (should be "Response") —
    preserved exactly from source table name.


### PQ-02 | AI Responce Feedback (local copy)
─────────────────────────────────────────────────────────────
Source      : Local copy import (file-based)
Purpose     : Standalone AI response feedback table for date partitioning
              and reference.
Columns     : 30 columns including CreatedDate_Only (datetime), Day_Sort, Weekday_Sort.


### PQ-03 | MoC Request Interactions Data_moc_ai_response
─────────────────────────────────────────────────────────────
Source      : PostgreSQL bia_sandbox.moc_ai_response
Purpose     : Standalone AI response table (separate from joined version in PQ-01).
Columns     : 29 columns.


### PQ-04 | SFHC Login Information Data (local copy)
─────────────────────────────────────────────────────────────
Source      : Local copy import (file-based)
Purpose     : SFHC login/presence data for MSLs. Powers the Parameter Table
              view toggle (Login History / User Presence / UCHT).
Key column  : CreatedDate_Only (datetime) — LocalDateTable_a4c1c799.


### PQ-05 | Calendar
─────────────────────────────────────────────────────────────
Source      : Referenced table (defined in calendar.txt)
Purpose     : Shared calendar table for time intelligence across the model.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — CONVERSION COMPLETENESS SCORECARD
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Category                         | Items | PBI Full | PBI Partial | Not Converted | % Complete |
|----------------------------------|-------|----------|-------------|---------------|------------|
| Parameters                       | 5     | 5        | 0           | 0             | 100%       |
| SQL-Level Calculated Columns     | 15+   | 15+      | 0           | 0             | 100%       |
| PQ Post-Processing Columns       | 18    | 18       | 0           | 0             | 100%       |
| Hierarchies                      | 1     | 1        | 0           | 0             | 100%       |
| Measures                         | 4     | 4        | 0           | 0             | 100%       |
| Power Query / Data Sources       | 5     | 5        | 0           | 0             | 100%       |
| LocalDateTables                  | 12    | 12       | 0           | 0             | 100%       |
| Relationships                    | 14    | 14       | 0           | 0             | 100%       |
| TMDL Infrastructure              | 3     | 3        | 0           | 0             | 100%       |
| TOTAL                            | 77+   | 77+      | 0           | 0             | 🟢 100%    |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — CONVERSION PATTERN CHEAT SHEET
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PATTERN-01 | DST-Aware Timezone Conversion (SQL-level)
─────────────────────────────────────────────────────────────
Tableau: DST-aware calculated field on Createddate per region
PBI: SQL CASE with DST boundary detection (Mar 2nd Sun, Nov 1st Sun
for USA; last Sun of Mar/Oct for Germany; fixed +9 for Japan).
Column: adj_date / GMT-EST - FINAL.

PATTERN-02 | Unified Date Field (MoC + Interaction)
─────────────────────────────────────────────────────────────
Tableau: IF ISNULL(created_date) THEN interaction_dt ELSE created_date END
PBI: SQL COALESCE(created_date, interaction_dt) → DATE - MOC+INT.

PATTERN-03 | Time Bucket Classification
─────────────────────────────────────────────────────────────
CASE on hour of adj_date:
  8–11 → "8 AM–12 PM" (sort 1)
  12–15 → "12 PM–4 PM" (sort 2)
  16–19 → "4 PM–8 PM" (sort 3)
  else → "Other" (sort 4)

PATTERN-04 | Multi-Granularity Label/Sort Pairs (PQ-level)
─────────────────────────────────────────────────────────────
For each time granularity (Year through Second), PQ generates both
a display label (formatted string) and a numeric sort key. These
feed an 8-level hierarchy: Year → Quarter → Month → Week → Day →
Hour → Minute → Second.

PATTERN-05 | AI Feedback Freshness Gate
─────────────────────────────────────────────────────────────
IF Createddate > cutoff_date THEN 'Data Not Refreshed' ELSE BMS_AI_Feedback__c
Two cutoff versions exist (2025-09-22 and 2026-01-05).

PATTERN-06 | Brand/Team Segmentation (SQL-level CASE chains)
─────────────────────────────────────────────────────────────
Multi-step derivation:
  1. queue_skill_all_mocs: queue_skill_bms__c + hcp_country → MoC queue
  2. interactions_cal_field: fld_team_name + country → interaction bucket
  3. Brand/Team Name filter: Unifies queue + interaction → 8 segments
  4. Team Name Filter MOC+INT: High-level team labels
  5. TA-Itrtn+MOC: Unified therapeutic area

PATTERN-07 | LOD FIXED AVG → DAX AVERAGEX(VALUES())
─────────────────────────────────────────────────────────────
Tableau: { FIXED [interaction_id] : AVG([interaction_duration])}
PBI: AVERAGEX(VALUES([Interaction ID]), CALCULATE(AVERAGE([Interaction Duration])))
     with REMOVEFILTERS on status column.

PATTERN-08 | FULL OUTER JOIN (SQL-level)
─────────────────────────────────────────────────────────────
Main fact joined to AI feedback via FULL OUTER JOIN on
a.id = b."MIRF_SF_Id__c". Retains unmatched rows from both sides.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAP-01 | Local Copy File-Based Imports
  Severity : Medium
  Detail   : AI Responce Feedback and SFHC Login tables use local file
             imports rather than live database connections.
  Action   : Confirm whether these should be migrated to PostgreSQL
             queries for automated refresh.

GAP-02 | AI Feedback Freshness Cutoff Dates Hardcoded
  Severity : Medium
  Detail   : CC-12/CC-13/CC-14 have hardcoded cutoff dates (2025-09-22
             and 2026-01-05). These must be manually updated when
             the AI feedback pipeline extends.
  Action   : Consider parameterizing the cutoff date or using
             a dynamic MAX date from the feedback table.

GAP-03 | Spelling: "Responce" Throughout
  Severity : Low (informational)
  Detail   : Table name "ms_ai_responce_msl_feedback" and column
             references contain "Responce" (misspelling of "Response").
             Preserved exactly from PostgreSQL source.
  Action   : Preserve unless source table is renamed.

GAP-04 | DST Rule Maintenance
  Severity : Low (annual)
  Detail   : DST boundaries (Mar 2nd Sunday, Nov 1st Sunday for USA;
             last Sunday of Mar/Oct for Germany) are embedded in SQL.
             If DST rules change, the SQL must be updated.
  Action   : Document annual DST rule review process.

GAP-05 | 12 LocalDateTables — Potential Simplification
  Severity : Low
  Detail   : 12 auto-generated LocalDateTables + 1 DateTableTemplate.
             Some may be redundant (e.g., adj_date and GMT-EST - FINAL
             are the same column with different names).
  Action   : Review and consolidate if possible. Do not collapse without
             verifying all date hierarchy drill-down paths.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — FILE INVENTORY & RELATIONSHIPS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### Model Tables
─────────────────────────────────────────────────────────────
| Table                                                            | Type              | Source              |
|------------------------------------------------------------------|-------------------|---------------------|
| MoC Request Interactions Data_Custom SQL Query                   | Main Fact         | PostgreSQL (joined) |
| AI Responce Feedback (local copy)_ms_ai_responce_msl_feedback    | Reference         | Local file import   |
| MoC Request Interactions Data_moc_ai_response                    | AI Response       | PostgreSQL          |
| SFHC Login Information Data (local copy)_ms_sfhc_msl_login_history| Login History    | Local file import   |
| Parameter Table                                                  | Parameter         | DATATABLE           |
| Calendar                                                         | Calendar          | Referenced table    |
| LocalDateTable (×12)                                             | Hidden date tables| Auto-generated      |
| DateTableTemplate_28fd16d6                                       | Template          | Auto-generated      |

### LocalDateTables (12)
─────────────────────────────────────────────────────────────
| LocalDateTable GUID | Source Table                        | Source Column        |
|---------------------|-------------------------------------|----------------------|
| 0d809bf2            | MoC Request Interactions (Fact)     | interaction_dt       |
| 2b664752            | MoC Request Interactions (Fact)     | adj_date             |
| 6ebfccf3            | MoC Request Interactions (Fact)     | refresh_date         |
| 84e4bf22            | MoC Request Interactions (Fact)     | created_date         |
| 96b1d7c8            | MoC Request Interactions (Fact)     | Createddate          |
| 733e5d97            | MoC Request Interactions (Fact)     | Interaction Date     |
| 6260e384            | MoC Request Interactions (Fact)     | GMT-EST - FINAL      |
| f68c6a5d            | MoC Request Interactions (Fact)     | Last Modified Date   |
| 6e51ab4e            | AI Responce Feedback (local copy)   | CreatedDate_Only     |
| 465cb8e3            | AI Responce Feedback (local copy)   | Weekday_Sort         |
| 45340033            | AI Responce Feedback (local copy)   | Day_Sort             |
| a4c1c799            | SFHC Login (local copy)             | CreatedDate_Only     |

### Relationships (14)
─────────────────────────────────────────────────────────────
12 date-variation relationships (LocalDateTable.Date → source column)
+ 2 data relationships (MoC Fact → AI Feedback on MIRF_SF_Id__c,
  MoC Fact → SFHC Login on join key).
All date relationships use joinOnDateBehavior: datePartOnly.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — BUSINESS LOGIC & KEY RULES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### 9.1 Date/Time Handling
  - All MoC records created in GMT. Converted to local time via DST logic.
  - Primary time column: GMT-EST - FINAL (adj_date).
  - DATE - MOC+INT unifies MoC and Interaction dates.

### 9.2 AI Feedback Freshness
  - Two cutoff dates (2025-09-22, 2026-01-05).
  - Records after cutoff → 'Data Not Refreshed'.

### 9.3 Brand/Team Segmentation (8 segments)
  - Germany CV, Japan Abecma, USA Cobenfy, USA CV, USA Derm,
    USA Myeloid, USA HCC, USA Zeposia.

### 9.4 MoC vs. Interaction Records
  - FULL OUTER JOIN combines both. DATE - MOC+INT handles both date types.

### 9.5 SFHC Login Data
  - Separate table with Parameter Table driving the view toggle.

### 9.6 Avg Duration
  - Per unique Interaction ID via AVERAGEX(VALUES()).
  - REMOVEFILTERS on Status (copy) for independence from status slicers.

### 9.7 Email Support Link
  - Static mailto measure for dashboard help.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — GLOSSARY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Term                    | Definition                                                     |
|-------------------------|-----------------------------------------------------------------|
| MoC                     | Medical on Call — Salesforce-based medical inquiry/case         |
| MSL                     | Medical Science Liaison                                        |
| HCP                     | Healthcare Professional                                        |
| MIRF                    | Medical Information Request Form                               |
| MI_Name                 | Medical Inquiry Name (MoC case identifier)                     |
| Createddate             | GMT timestamp when MoC case was created in Salesforce          |
| GMT-EST - FINAL         | DST-aware local time conversion of Createddate                 |
| adj_date                | Internal alias for GMT-EST - FINAL (SQL subquery)              |
| DATE - MOC+INT          | Unified date: created_date (MoC) or interaction_dt (Interaction)|
| interaction_dt          | Date/time of MSL-HCP interaction                               |
| refresh_date            | Date data was last refreshed in the datamart                   |
| queue_skill_bms__c      | Salesforce field for MoC queue skill / therapeutic area         |
| fld_team_name           | Field team name from interaction data                          |
| ITRTN_TA + Product      | Concatenated therapeutic area + product                        |
| BMS_AI_Feedback__c      | Salesforce AI response feedback field                          |
| TIME BUCKET             | Hour-of-day bucket: 8AM–12PM, 12PM–4PM, 4PM–8PM, Other        |
| SFHC                    | Salesforce Health Cloud — login/presence tracking              |
| bia_sandbox             | PostgreSQL schema for BMS BIA sandbox tables                   |
| ms_fnl_moc_scrd_ads     | Main MoC scored/ADS table                                      |
| ms_ai_responce_msl_feedback | AI response feedback table (note: "Responce" misspelling)  |
| FULL OUTER JOIN         | SQL join retaining unmatched rows from both sides              |
| TAS                     | Territory Alignment / Therapeutic Area Specialist               |


END OF KNOWLEDGE FILE
