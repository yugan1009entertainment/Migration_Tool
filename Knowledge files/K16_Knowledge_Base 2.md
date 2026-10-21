# K16 Knowledge Base — Medical on Call Dashboard
## Power BI Semantic Model (K16 2B) + Tableau (K16 1A)
**Generated:** 2026-06-12  
**Model Name:** Medical on Call - Dashboard  
**Compatibility Level:** 1604  
**Culture:** en-US  
**Power BI Data Source Version:** powerBI_V3  
**Tableau Workbook Revision:** 10.4 (PRODUCTION site)

---

## Table of Contents
1. [Model Overview](#1-model-overview)
2. [Data Sources](#2-data-sources)
3. [Tables](#3-tables)
   - 3.1 [MoC Request Interactions Data_Custom SQL Query (Main Fact)](#31-moc-request-interactions-data_custom-sql-query-main-fact)
   - 3.2 [AI Responce Feedback (local copy)_ms_ai_responce_msl_feedback](#32-ai-responce-feedback-local-copy_ms_ai_responce_msl_feedback)
   - 3.3 [MoC Request Interactions Data_moc_ai_response](#33-moc-request-interactions-data_moc_ai_response)
   - 3.4 [SFHC Login Information Data (local copy)_ms_sfhc_msl_login_history](#34-sfhc-login-information-data-local-copy_ms_sfhc_msl_login_history)
   - 3.5 [Parameter Table](#35-parameter-table)
   - 3.6 [Calendar](#36-calendar)
4. [Local Date Tables](#4-local-date-tables)
5. [Relationships](#5-relationships)
6. [DAX Measures](#6-dax-measures)
7. [Calculated / SQL Columns](#7-calculated--sql-columns)
8. [Tableau (K16 1A) — Parameters & Calculated Fields](#8-tableau-k16-1a--parameters--calculated-fields)
9. [Business Logic & Key Rules](#9-business-logic--key-rules)
10. [Glossary](#10-glossary)

---

## 1. Model Overview

| Property | Value |
|---|---|
| Report Name | Medical on Call - Dashboard |
| Power BI Model Name | Medical on Call - Dashboard- model |
| Compatibility Level | 1604 |
| Culture | en-US |
| Tableau Workbook | MOC-Dashboard-WIP (revision 10.4) |
| Tableau Site | PRODUCTION |
| Tableau Server | tableau.bms.com |
| PBI Query Order | MoC Request Interactions Data_Custom SQL Query → SFHC Login Info → AI Responce Feedback → MoC Request Interactions Data_moc_ai_response |
| Dev Mode | Enabled (PBI_ProTooling = DevMode) |
| Time Intelligence | Enabled (__PBI_TimeIntelligenceEnabled = 1) |

---

## 2. Data Sources

### Power BI (K16 2B)
| Source | Connection | Schema / Table |
|---|---|---|
| Main Fact + AI Feedback (joined) | PostgreSQL: `imap-dl-datamart.web.bms.com:5432` / `postgres` | `bia_sandbox.ms_fnl_moc_scrd_ads` FULL OUTER JOIN `bia_sandbox.ms_ai_responce_msl_feedback` ON `a.id = b."MIRF_SF_Id__c"` |
| AI Response (standalone) | Same PostgreSQL | `bia_sandbox.moc_ai_response` |
| SFHC Login History | Local copy (file-based import) | `ms_sfhc_msl_login_history` |
| AI Response Feedback | Local copy (file-based import) | `ms_ai_responce_msl_feedback` |

### Tableau (K16 1A)
| Source | Connection | Details |
|---|---|---|
| Main Data Source | Tableau Published Data Source (SQL Proxy) | `MoCRequestInteractionsData` rev 2.2 on `tableau.bms.com` |
| Parameters | Inline (no connection) | Start Date, End Date, Start Date-AI, End Date-AI, SFHC-Parameter |

---

## 3. Tables

### 3.1 MoC Request Interactions Data_Custom SQL Query (Main Fact)

**Source:** PostgreSQL 3-level nested SQL query (`bia_sandbox.ms_fnl_moc_scrd_ads` FULL OUTER JOIN `bia_sandbox.ms_ai_responce_msl_feedback`)  
**Note:** Described in the model as "combination of other tables"

#### Raw Source Columns (from `ms_fnl_moc_scrd_ads` — alias `a`)
| Column | Data Type | Description |
|---|---|---|
| id | string | MoC record ID (primary key) |
| mi_name | string | Medical Inquiry name |
| bms_worker_name | string | BMS Worker Name |
| inquiry_text | string | Inquiry text |
| status | string | MoC status (Open / Closed / Cancelled) |
| moc_state | string | MoC state (mapped to `State_1`) |
| queue_skill_bms__c | string | Queue skill / therapeutic area code |
| reason_for_cancellation | string | Reason for cancellation |
| hcp_name | string | HCP name |
| hcp_id | string | HCP ID |
| hcp_country | string | HCP country code (US, DE, JP, IT, PR) |
| ta | string | Therapeutic Area (MoC-level) |
| moc_region | string | MoC region (mapped to `TAS Region`) |
| createddate | datetime | MoC created datetime (GMT) |
| created_date | date | Interaction created date |
| lastmodifieddate | datetime | Last modified datetime |
| interaction_dt | datetime | Interaction datetime |
| interaction_name | string | Interaction name |
| interaction_id | string | Interaction ID |
| itrtn_hcp_id | string | Interaction HCP ID |
| itrtn_hcp_name | string | Interaction HCP name |
| country | string | Interaction country |
| itrtn_ta | string | Interaction therapeutic area |
| state | string | Interaction state |
| interaction_duration | int64 | Duration in minutes |
| fld_team_name | string | Field team name (CARDIOVASCULAR, DERMATOLOGY, etc.) |
| intraction_type | string | Interaction type |
| interaction_intent | string | Interaction intent |
| worker_bms_id | string | Worker BMS ID (mapped to FM ID) |
| msm_full_name | string | MSL/FM full name |
| product | string | Product name |
| itrtn_sub_topic | string | Medical Communication Topic |
| da | string | Disease area / Indication |
| itrtn_topic | string | Interaction topic |
| refresh_date | datetime | Data refresh date |
| account_vod__c | string | Veeva account ID |
| acpn_by_ind_derv | string | Derived field |
| product_id | string | Product ID |
| product_name | string | Product name (full) |
| region | string | Region |
| sub_region | string | Sub-region |

#### Joined Columns (from `ms_ai_responce_msl_feedback` — alias `b`, suffixed `(ms_ai_responce_msl_feedback)`)
| Column | Data Type |
|---|---|
| Account.Name | string |
| AccountId | string |
| Agent_Work__c | string |
| BMS_AI_Feedback__c | string |
| BMS_Reason_for_Cancellation__c | string |
| Case_Region__c | string |
| ClosedDate | string |
| CreatedBy.Name | string |
| CreatedById | string |
| CreatedDate | string |
| Description | string |
| Id | string |
| IsClosed | boolean |
| LastModifiedById | string |
| LastModifiedDate | string |
| Medical_Skill_Area__c | string |
| MIRF_Link__c | string |
| MIRF_SF_Id__c | string |
| Origin | string |
| Owner.Name | string |
| OwnerId | string |
| Priority | string |
| Remote_Meeting_URL__c | string |
| Status | string |
| Subject | string |
| TAS_Email__c | string |
| TAS_Name__c | string |
| Therapeutic_Area__c | string |
| Veeva_CRM_Org__c | string |
| CaseNumber | int64 |

#### SQL-Level Calculated Columns (Level 1–3)
| Column | Logic |
|---|---|
| `Region-MOC+Int` | Derives region from `hcp_country`/`country`: Germany, Italy, Japan, Puerto Rico, USA, Others |
| `queue_skill_all_mocs` | Maps `queue_skill_bms__c` + `hcp_country` to standardized MoC queue names (Germany MoCs, Japan MoCs, USA CV MoCs, etc.) |
| `ITRTN_TA + Product` | `COALESCE(itrtn_ta,'') \|\| '-' \|\| COALESCE(product,'')` |
| `interactions_cal_field` | Maps `fld_team_name` + `country` (or `ITRTN_TA+Product` for intl) to interaction bucket labels (Germany CV Interactions, Japan Abecma Interactions, USA CV Interactions, etc.) |
| `adj_date` | DST-aware timezone conversion of `Createddate` to local time (USA: UTC-4/UTC-5; Germany: UTC+2/UTC+1; Japan: UTC+9) — internal use |
| `GMT-EST - FINAL` | Same as `adj_date` — exposed as Power BI visible column |
| `TIME BUCKET` | Buckets `adj_date` hour: 8 AM–12 PM, 12 PM–4 PM, 4 PM–8 PM, Other |
| `GMT-Local Time` | `HH:MM` string from `adj_date` |
| `Feedback-AI Response` | `BMS_AI_Feedback__c` if `Createddate` ≤ 2025-09-22, else `'Data Not Refreshed'` |
| `Feedback-AI Response(Copy)` | Same logic but cutoff is 2026-01-05 |
| `Feedback: AI Response` | Null-safe version of `Feedback-AI Response(Copy)` (returns `'Null'` if NULL) |
| `Brand/Team Name filter` | Combined queue/interaction label: Germany CV, Japan Abecma, USA Cobenfy, USA CV, USA Derm, USA Myeloid, USA HCC, USA Zeposia |
| `Brand/Team Name filter Copy` | Duplicate of above |
| `Team Name Filter MOC+INT` | Maps `queue_skill_bms__c`/`fld_team_name` to: CV, Dermatology, Neuropsychiatry, MYELOID, Zeposia, NEUROLOGY, Cell Therapy |
| `Cancelled MI Name` | `mi_name` if `status = 'Cancelled'`, else NULL |
| `DATE - MOC+INT` | `created_date` if not null, else `interaction_dt` |

#### Power Query (M) Post-Processing Columns
| Column | Logic |
|---|---|
| `Weekday Name` | Day of week name from `GMT-EST - FINAL` |
| `Weekday Sort` | Numeric weekday (Monday=1) from `GMT-EST - FINAL` |
| `Year Label` | Year string from `GMT-EST - FINAL` |
| `Quarter Label` | `"Q{n} {yyyy}"` from `GMT-EST - FINAL` |
| `Quarter Sort` | `Year * 10 + Quarter` |
| `Month Label` | `"MMMM yyyy"` from `GMT-EST - FINAL` |
| `Month Sort` | `Year * 100 + Month` |
| `Week Label` | Sunday-starting week start date as `"MMMM d, yyyy"` |
| `Week Sort` | `YYYYMMDD` of week start |
| `Day Label` | `"MMMM d, yyyy"` from `GMT-EST - FINAL` |
| `Day Sort` | `YYYYMMDD` numeric |
| `Hour Label` | `"MMMM d, yyyy at H AM/PM"` |
| `Hour Sort` | `YYYYMMDDH` numeric |
| `Minute Label` | `"MMMM d, yyyy at H:MM AM/PM"` |
| `Minute Sort` | `YYYYMMDDHHmm` numeric |
| `Second Label` | `"M/D/YYYY H:MM:SS AM/PM"` |
| `Second Sort` | `YYYYMMDDHHmmss` numeric |
| `Time Bucket Sort` | 8AM-12PM=1, 12PM-4PM=2, 4PM-8PM=3, Other=4, else 99 |

#### Hierarchy
| Hierarchy | Levels |
|---|---|
| Year Label Hierarchy | Year Label → Quarter Label → Month Label → Week Label → Day Label → Hour Label → Minute Label → Second Label |

---

### 3.2 AI Responce Feedback (local copy)_ms_ai_responce_msl_feedback

**Source:** Local copy import (file-based)  
**Purpose:** Standalone AI response feedback table (used for date table partitioning and as a reference)

| Column | Data Type |
|---|---|
| Id | string |
| CaseNumber | int64 |
| BMS_AI_Feedback__c | string |
| AccountId | string |
| RecordTypeId | string |
| Status | string |
| IsClosed | boolean |
| ClosedDate | string |
| OwnerId | string |
| CreatedDate_Only | datetime |
| CreatedById | string |
| LastModifiedDate | string |
| LastModifiedById | string |
| Case_Region__c | string |
| MIRF_SF_Id__c | string |
| Medical_Skill_Area__c | string |
| Account.Name | string |
| Agent_Work__c | string |
| BMS_Reason_for_Cancellation__c | string |
| Description | string |
| MIRF_Link__c | string |
| Origin | string |
| Owner.Name | string |
| Priority | string |
| Remote_Meeting_URL__c | string |
| Subject | string |
| TAS_Email__c | string |
| TAS_Name__c | string |
| Therapeutic_Area__c | string |
| Veeva_CRM_Org__c | string |
| Weekday_Sort | datetime |
| Day_Sort | datetime |

---

### 3.3 MoC Request Interactions Data_moc_ai_response

**Source:** PostgreSQL `bia_sandbox.moc_ai_response`  
**Purpose:** Standalone AI response/feedback table (separate from joined version in main fact)

| Column | Data Type |
|---|---|
| Id | string |
| CaseNumber | int64 |
| BMS_AI_Feedback__c | string |
| AccountId | string |
| RecordTypeId | string |
| Status | string |
| IsClosed | boolean |
| ClosedDate | string |
| OwnerId | string |
| CreatedDate | string |
| CreatedById | string |
| LastModifiedDate | string |
| LastModifiedById | string |
| Case_Region__c | string |
| MIRF_SF_Id__c | string |
| Medical_Skill_Area__c | string |
| Account.Name | string |
| Agent_Work__c | string |
| BMS_Reason_for_Cancellation__c | string |
| Description | string |
| MIRF_Link__c | string |
| Origin | string |
| Owner.Username | string |
| Pending_Assignment__c | boolean |
| Priority | string |
| Remote_Meeting_URL__c | string |
| Subject | string |
| TAS_Email__c | string |
| TAS_Name__c | string |
| Therapeutic_Area__c | string |
| Veeva_CRM_Org__c | string |

---

### 3.4 SFHC Login Information Data (local copy)_ms_sfhc_msl_login_history

**Source:** Local copy import  
**Purpose:** SFHC login/presence data for MSLs; powers the Parameter Table view toggle

Key columns include:
- `CreatedDate_Only` (datetime) — used for LocalDateTable_a4c1c799 partitioning

---

### 3.5 Parameter Table

**Source:** DAX Calculated Table (`DATATABLE`)  
**Purpose:** Drives the SFHC tab view-by selector (Login History / User Presence / User Case Handle Time)

| Parameter | SortOrder |
|---|---|
| Login History | 1 |
| User Presence | 2 |
| User Case Handle Time | 3 |

---

### 3.6 Calendar

**Source:** Referenced table (defined separately in `calendar.txt`)  
**Purpose:** Shared calendar table for time intelligence across the model

---

## 4. Local Date Tables

All local date tables are **hidden**, auto-generated (`showAsVariationsOnly`), and follow the standard Power BI pattern with columns: `Date`, `Year`, `MonthNo`, `Month`, `QuarterNo`, `Quarter`, `Day` and a `Date Hierarchy` (Year → Quarter → Month → Day).

| Table ID | Source Table | Source Column | Description |
|---|---|---|---|
| LocalDateTable_0d809bf2 | MoC Request Interactions Data_Custom SQL Query | `interaction_dt` | MoC interaction datetime |
| LocalDateTable_2b664752 | MoC Request Interactions Data_Custom SQL Query | `adj_date` | Timezone-adjusted datetime |
| LocalDateTable_6e51ab4e | AI Responce Feedback (local copy) | `CreatedDate_Only` | AI feedback created date |
| LocalDateTable_6ebfccf3 | MoC Request Interactions Data_Custom SQL Query | `refresh_date` | Data refresh date |
| LocalDateTable_84e4bf22 | MoC Request Interactions Data_Custom SQL Query | `created_date` | MoC/interaction created date |
| LocalDateTable_96b1d7c8 | MoC Request Interactions Data_Custom SQL Query | `Createddate` | MoC created datetime |
| LocalDateTable_465cb8e3 | AI Responce Feedback (local copy) | `Weekday_Sort` | AI feedback weekday sort field |
| LocalDateTable_45340033 | AI Responce Feedback (local copy) | `Day_Sort` | AI feedback day sort field |
| LocalDateTable_733e5d97 | MoC Request Interactions Data_Custom SQL Query | `Interaction Date` | Interaction date |
| LocalDateTable_6260e384 | MoC Request Interactions Data_Custom SQL Query | `GMT-EST - FINAL` | Final local-time datetime |
| LocalDateTable_f68c6a5d | MoC Request Interactions Data_Custom SQL Query | `Last Modified Date` | Record last modified date |
| LocalDateTable_a4c1c799 | SFHC Login Information Data (local copy) | `CreatedDate_Only` | SFHC login created date |
| DateTableTemplate_28fd16d6 | (Template) | — | Shared date table template |

---

## 5. Relationships

All relationships originate **from** the local date tables **to** the fact/detail tables.

| From Table | From Column | To Table | To Column | Type |
|---|---|---|---|---|
| LocalDateTable_0d809bf2 | Date | MoC Request Interactions Data_Custom SQL Query | interaction_dt | Many-to-One |
| LocalDateTable_2b664752 | Date | MoC Request Interactions Data_Custom SQL Query | adj_date | Many-to-One |
| LocalDateTable_6e51ab4e | Date | AI Responce Feedback (local copy) | CreatedDate_Only | Many-to-One |
| LocalDateTable_6ebfccf3 | Date | MoC Request Interactions Data_Custom SQL Query | refresh_date | Many-to-One |
| LocalDateTable_84e4bf22 | Date | MoC Request Interactions Data_Custom SQL Query | created_date | Many-to-One |
| LocalDateTable_96b1d7c8 | Date | MoC Request Interactions Data_Custom SQL Query | Createddate | Many-to-One |
| LocalDateTable_465cb8e3 | Date | AI Responce Feedback (local copy) | Weekday_Sort | Many-to-One |
| LocalDateTable_45340033 | Date | AI Responce Feedback (local copy) | Day_Sort | Many-to-One |
| LocalDateTable_733e5d97 | Date | MoC Request Interactions Data_Custom SQL Query | Interaction Date | Many-to-One |
| LocalDateTable_6260e384 | Date | MoC Request Interactions Data_Custom SQL Query | GMT-EST - FINAL | Many-to-One |
| LocalDateTable_f68c6a5d | Date | MoC Request Interactions Data_Custom SQL Query | Last Modified Date | Many-to-One |
| LocalDateTable_a4c1c799 | Date | SFHC Login Information Data (local copy) | CreatedDate_Only | Many-to-One |
| MoC Request Interactions Data_Custom SQL Query | MIRF_SF_Id__c (ms_ai_responce_msl_feedback) | AI Responce Feedback (local copy) | [join key] | Many-to-One |
| MoC Request Interactions Data_Custom SQL Query | [join key] | SFHC Login Information Data (local copy) | [join key] | Many-to-One |

---

## 6. DAX Measures

All measures reside on the `MoC Request Interactions Data_Custom SQL Query` table.

| Measure | DAX Formula | Notes |
|---|---|---|
| `Avg Duration` | `CALCULATE(AVERAGEX(VALUES([Interaction ID]), CALCULATE(AVERAGE([Interaction Duration]))), REMOVEFILTERS([Status (copy)]))` | Average duration per unique Interaction ID, ignoring Status (copy) filter |
| `nuL` | `BLANK()` | Blank placeholder (formatString: 0) |
| `Max Date` | `CALCULATE(MAX([refresh_date]), REMOVEFILTERS('SFHC Login Information Data (local copy)_ms_sfhc_msl_login_history'))` | Latest refresh date, ignoring SFHC filters |
| `Email Action` | `"mailto:Madhuprabha.Sivakumar@bms.com;sachin.bhardwaj@bms.com?cc=Mounika.Bommareddy@bms.com&Subject=Help%20-%20Medical%20on%20Call%20Dashboard"` | Static email mailto link for dashboard help |

---

## 7. Calculated / SQL Columns

### Timezone Conversion Logic (`GMT-EST - FINAL` / `adj_date`)

The model applies DST-aware timezone offsets based on `Region-MOC+Int`:

| Region | Standard Offset | DST Offset | DST Period |
|---|---|---|---|
| USA | UTC-5 (EST) | UTC-4 (EDT) | Mar 2nd Sun → Nov 1st Sun |
| Germany | UTC+1 (CET) | UTC+2 (CEST) | Last Sun of Mar → Last Sun of Oct |
| Japan | UTC+9 | — (no DST) | Always +9 |
| Others | No change | — | Pass-through |

### AI Feedback Freshness Columns

| Column | Cutoff Date | Logic |
|---|---|---|
| `Feedback-AI Response` | 2025-09-22 | Returns `BMS_AI_Feedback__c` or `'Data Not Refreshed'` |
| `Feedback-AI Response(Copy)` | 2026-01-05 | Same pattern, newer cutoff |
| `Feedback: AI Response` | 2026-01-05 | Null-safe (returns `'Null'` if null) |

### Region & Team Mapping

| Derived Column | Logic Summary |
|---|---|
| `Region-MOC+Int` | Maps `hcp_country`/`country` codes → Germany, Italy, Japan, Puerto Rico, USA, Others |
| `queue_skill_all_mocs` | Maps `queue_skill_bms__c` + `hcp_country` → standardized MoC queue name |
| `interactions_cal_field` | Maps `fld_team_name` + `country` (or `ITRTN_TA+Product`) → interaction bucket |
| `Brand/Team Name filter` | Unifies MoC queue + interaction bucket → brand/team label |
| `Team Name Filter MOC+INT` | High-level team label (CV, Dermatology, Neuropsychiatry, MYELOID, Zeposia, etc.) |
| `TA-Itrtn+MOC` | Unified TA across MoC + Interaction: CARDIOVASCULAR, IMMUNOLOGY, ONCOLOGY, Neurology, HEMATOLOGY, Others |

---

## 8. Tableau (K16 1A) — Parameters & Calculated Fields

### Parameters (Inline — no connection)

| Parameter | Caption | Data Type | Default Value | Domain |
|---|---|---|---|---|
| Parameter 1 | Start Date | Date | 2025-01-01 | Any |
| Parameter 2 | End Date | Date | 2025-12-31 | Any |
| Parameter 3 | Start Date - AI | Date | 2025-01-01 | Any |
| Parameter 4 | End Date - AI | Date | 2025-12-31 | Any |
| Parameter 5 | SFHC-Parameter | String | "Login History" | List: Login History, User Presence, User Case Handle Time (alias: UCHT) |

### Key Calculated Fields

| Field Caption | Formula Summary |
|---|---|
| `Region-MOC+Int` | `hcp_country`/`country` → Germany, Italy, Japan, Puerto Rico, USA, Others |
| `DATE - MOC+INT` | `IF ISNULL(created_date) THEN interaction_dt ELSE created_date END` |
| `Daterange` | True/False based on Parameter 1 ≤ DATE-MOC+INT ≤ Parameter 2 |
| `Daterange AI` | Same logic as Daterange (copy) |
| `GMT-EST - FINAL` | DST-aware timezone conversion from `createddate` per region (USA EDT/EST, Germany CEST/CET, Japan +9) |
| `GMT to EST` | Earlier version of timezone conversion |
| `TIME BUCKET` | Buckets `GMT-EST - FINAL` hour: 8 AM–12 PM, 12 PM–4 PM, 4 PM–8 PM, Other |
| `GMT-Local Time` | HH:MM string from `GMT-EST - FINAL` |
| `ITRTN_TA + Product` | `[itrtn_ta] + '-' + [product]` |
| `Interactions Team/Product Name` | Maps `fld_team_name`/`ITRTN_TA+Product` → CARDIOVASCULAR, DERMATOLOGY, MYELOID, NEUROPSYCHIATRY, NEUROLOGY, Cell Therapy |
| `Queue Skill - All MOCs` | Maps `queue_skill_bms__c` + `hcp_country` → standardized MoC queue labels |
| `INTERACTIONS-CAL FIELD` | Maps `fld_team_name`/`country`/`ITRTN_TA+Product` → interaction bucket labels |
| `Brand/Team Name filter` | Unifies Queue + Interaction bucket → Germany CV, Japan Abecma, USA Cobenfy/CV/Derm/Myeloid/HCC/Zeposia |
| `Team Name FIlter MOC+INT` | High-level team: CV, Dermatology, Neuropsychiatry, MYELOID, Zeposia, NEUROLOGY, Cell Therapy |
| `TA-Itrtn+MOC` | Unified TA: CARDIOVASCULAR, IMMUNOLOGY, ONCOLOGY, Neurology, HEMATOLOGY, Others |
| `Feedback-AI Response` | `IF DATE(createddate) > '2025-09-22' THEN 'Data Not Refreshed' ELSE BMS_AI_Feedback__c END` |
| `Feedback-AI Response (copy)` | Same with cutoff 2026-01-05 |
| `Feedback: AI Response` | Null-safe version using copy field |
| `Avg Duration` | `{ FIXED [interaction_id] : AVG([interaction_duration])}` |
| `Cancelled MoC` | `IF [status]='Cancelled' THEN [mi_name] END` |
| `Closed MoC` | `IF [status]='Closed' THEN [mi_name] END` |
| `nuL` | `''` (empty string placeholder) |
| `NULL` | `""` (empty string placeholder) |
| `CreatedDate-DUP` | `STR(DATEPART('hour', createddate)) + ':' + RIGHT('00' + STR(DATEPART('minute', createddate)), 2)` |

---

## 9. Business Logic & Key Rules

### 9.1 Date/Time Handling
- All MoC records are created in **GMT**. The model converts to **local time** per region using DST-aware logic.
- The primary time column for analysis is **`GMT-EST - FINAL`** (Power BI) / **`GMT-EST - FINAL` calculated field** (Tableau).
- The **`DATE - MOC+INT`** column unifies MoC (`created_date`) and Interaction (`interaction_dt`) dates into a single date axis.

### 9.2 AI Feedback Freshness
- The model includes two freshness-gated feedback columns with different cutoff dates:
  - **`Feedback-AI Response`**: Cutoff 2025-09-22
  - **`Feedback-AI Response(Copy)` / `Feedback: AI Response`**: Cutoff 2026-01-05
- Records created after the cutoff display `'Data Not Refreshed'` instead of the actual feedback value.

### 9.3 Brand/Team Segmentation
- The dashboard supports **8 brand/team segments**: Germany CV, Japan Abecma, USA Cobenfy, USA CV, USA Derm, USA Myeloid, USA HCC, USA Zeposia.
- Segmentation is determined by a combination of `queue_skill_bms__c`, `fld_team_name`, `hcp_country`, `country`, and `ITRTN_TA + Product`.

### 9.4 MoC vs. Interaction Records
- The main table contains **both MoC records** (from `ms_fnl_moc_scrd_ads`) and their **linked interactions** (via full outer join).
- MoC records have `created_date`; interaction records have `interaction_dt`. The `DATE - MOC+INT` field handles both.

### 9.5 SFHC Login Data
- A separate table (`SFHC Login Information Data`) tracks MSL login history/presence.
- The **Parameter Table** (`Login History`, `User Presence`, `User Case Handle Time`) drives the SFHC view toggle.

### 9.6 Avg Duration Calculation
- Average duration is calculated **per unique Interaction ID** (not per row) using `AVERAGEX` over `VALUES([Interaction ID])`.
- The `Status (copy)` filter is removed to ensure the metric is not affected by status slicers.

### 9.7 Email Support Link
- The `Email Action` measure provides a pre-formatted `mailto:` link for dashboard help requests directed to the model team.

---

## 10. Glossary

| Term | Definition |
|---|---|
| MoC | Medical on Call — a Salesforce-based medical inquiry/case |
| MSL | Medical Science Liaison |
| HCP | Healthcare Professional |
| MIRF | Medical Information Request Form |
| MI_Name | Medical Inquiry Name (the MoC case identifier) |
| Createddate | GMT timestamp when the MoC case was created in Salesforce |
| GMT-EST - FINAL | DST-aware local time conversion of Createddate |
| adj_date | Internal alias for GMT-EST - FINAL (used in SQL subquery) |
| DATE - MOC+INT | Unified date field: created_date (MoC) or interaction_dt (Interaction) |
| interaction_dt | Date/time of the MSL-HCP interaction |
| created_date | Date the interaction record was created |
| refresh_date | Date the data was last refreshed in the datamart |
| queue_skill_bms__c | Salesforce field for the MoC queue skill / therapeutic area |
| fld_team_name | Field team name from interaction data |
| ITRTN_TA + Product | Concatenated therapeutic area + product for interaction classification |
| BMS_AI_Feedback__c | Salesforce field capturing AI response feedback (thumbs up/down, etc.) |
| queue_skill_all_mocs | Derived standardized MoC queue label per brand/region |
| interactions_cal_field | Derived interaction category per brand/country |
| Brand/Team Name filter | Unified brand-level filter across MoCs and Interactions |
| TIME BUCKET | Hour-of-day bucket: 8AM–12PM, 12PM–4PM, 4PM–8PM, Other |
| SFHC | Salesforce Health Cloud — login/presence tracking system |
| bia_sandbox | PostgreSQL schema for BMS BIA sandbox tables |
| ms_fnl_moc_scrd_ads | Main MoC scored/ADS (analytics data store) table |
| ms_ai_responce_msl_feedback | AI response feedback table (joined to main fact) |
| moc_ai_response | Standalone AI response table (separate query) |
| ms_sfhc_msl_login_history | SFHC MSL login history table |
| imap-dl-datamart | BMS internal PostgreSQL datamart host |
| TAS | Territory Alignment / Therapeutic Area Specialist |
| DA | Disease Area (maps to Indication) |
| Dev Mode | Power BI Desktop Developer Mode (PBI_ProTooling annotation) |
| FULL OUTER JOIN | SQL join type used to combine MoC and AI feedback rows, retaining unmatched rows from both sides |
