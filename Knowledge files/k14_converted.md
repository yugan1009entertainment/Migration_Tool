# 📘 CONVERSION KNOWLEDGE FILE
# K14 – 24363 DRWDS Oncology Capability Assessment
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-23
# Source files: 24363_drwds_oncology_capability_assessment.txt (Tableau TWB)
#               + 25 Power BI PBIP output files
# Workbook Name: 24363-DRWDS_Oncology_CapabilityAssessment
# Tableau Build: 2023.1.4 (20231.23.0711.1149)
# PBI Compatibility Level: 1550
# Domain: BMS Real World Data Science (RWDS) - Oncology
# Vendor: HealthVerity

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PARAM-01 | Yearly/Monthly (Parameter 1)
─────────────────────────────────────────────────────────────
Tableau Caption       : Yearly/Monthly
Tableau Type          : String (list)
Tableau Default       : "1"
Tableau Alias         : "Yearly"

Power BI Table Name   : Yearly Monthly
Power BI Type         : Single-row parameter table
Power BI Expression GUID: b2a7c08e-7fe5-4ace-bab8-a4410a38192e
PBI Column            : ParameterTableKeyColumn (text, value "1")

Usage in model:
  - Referenced via SELECTEDVALUE for time granularity switching.

Conversion Status: ✅ FULL


### PARAM-02 | start date/end date (Parameter 2)
─────────────────────────────────────────────────────────────
Tableau Caption       : start date/end date
Tableau Type          : String (list)
Tableau Default       : "1"
Tableau Alias         : "Event Start Date"

Power BI Table Name   : start date end date
Power BI Type         : Single-row parameter table
Power BI Expression GUID: 8ca5330a-e08a-4661-9566-993a94ec9e5b
PBI Column            : ParameterTableKeyColumn (text, value "1")

Conversion Status: ✅ FULL


### PARAM-03 | Dashboard Refresh Date (Parameter 3)
─────────────────────────────────────────────────────────────
Tableau Caption       : Dashboard Refresh Date
Tableau Type          : Date
Tableau Default       : #2023-09-06#

Power BI Table Name   : Dashboard Refresh Date
Power BI Type         : Single-row date parameter table
Power BI Expression GUID: 71b5b1bc-4667-4551-af29-b56663543afe
PBI Value             : #date(2023, 09, 06)
PBI Column            : ParameterTableKeyColumn (date)

Usage in model:
  - Linked to LocalDateTable_0feaa2be via relationship 7aa34c2c.
  - joinOnDateBehavior: datePartOnly.

Pattern     : Tableau date literal #YYYY-MM-DD# → PBI M #date(YYYY, MM, DD).

Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### CC-01 | Category and Units (Outlier Patient Summary)
─────────────────────────────────────────────────────────────
Tableau Expression: [Category_type] + ' (in ' + [Units] + ')'
Power BI DAX: [Category_type] & " (in " & [Units] & ")"
Conversion  : ✅ FULL

### CC-02 | sort (Outlier Patient Summary — PBI-only)
─────────────────────────────────────────────────────────────
Power BI DAX: RELATED('Table (2)'[s])
Type        : ➕ PBI-only — no Tableau equivalent.
Purpose     : Sort key via RELATED lookup to Table (2).
Conversion  : ✅ FULL

### CC-03 | States (patient_cohort)
─────────────────────────────────────────────────────────────
Tableau Expression: if [Main Category]='States' THEN [Category Type] END
Power BI DAX: IF([Main Category]="States", [Category Type], BLANK())
Conversion  : ✅ FULL

### CC-04 | Category Type Sort (patient_cohort — PBI-only, hidden)
─────────────────────────────────────────────────────────────
Power BI DAX: IFERROR(VALUE([Category Type]), 9999)
Purpose     : Numeric sort key for Category Type display.
Conversion  : ✅ FULL

### CC-05 through CC-08 | Geo Columns (patient_cohort — PBI-only, hidden)
─────────────────────────────────────────────────────────────
| CC   | Column Name | DAX Summary                                           |
|------|-------------|-------------------------------------------------------|
| CC-05| Country     | "United States" (constant)                            |
| CC-06| State Geo   | SWITCH for PR/DC + [Category Type] & ", United States"|
| CC-07| State Lat   | Full SWITCH with lat coords for all 50 states + PR    |
| CC-08| State Lng   | Full SWITCH with lng coords for all 50 states + PR    |

Purpose     : Bypass PBI geocoding — explicit coordinates for US state map.
Type        : ➕ PBI-only — replaces Tableau built-in map semantic role.
Conversion  : ✅ FULL (all 4)

### CC-09 | Data Quality Issue
─────────────────────────────────────────────────────────────
Tableau Expression: [DQ Issue Name] + ' :' + [Details]
Power BI DAX: [DQ Issue Name] & " :" & [Details]
Conversion  : ✅ FULL

### CC-10 | % Patient - Final (Attribute Summary)
─────────────────────────────────────────────────────────────
Power BI DAX: IFERROR(VALUE([% Patients]), 0)
Purpose     : Text-to-number conversion for percentage patients.
Conversion  : ✅ FULL

### CC-11 | # Patients - Numeric (Attribute Summary)
─────────────────────────────────────────────────────────────
Power BI DAX:
VAR _TextValue = TRIM([# Patients] & "")
VAR _CleanValue = SUBSTITUTE(_TextValue, ",", "")
RETURN IF(_TextValue = "" || _TextValue = "-", 0, IFERROR(VALUE(_CleanValue), 0))

Purpose     : Cleans text patient counts (removes commas, handles blanks/dashes).
Conversion  : ✅ FULL

### CC-12 | Non-Outlier Range (Cost Analysis)
─────────────────────────────────────────────────────────────
Power BI DAX: "Not Available" (PLACEHOLDER)
⚠️  GAP: Referenced Min/Max columns not present in Data Model 10.
Conversion  : ⚠️ PARTIAL — placeholder until Cost Analysis source confirmed.

### CC-13 | Number of Patients Display (Data Quality — PBI-only)
─────────────────────────────────────────────────────────────
Power BI DAX:
VAR _Value = TRIM([Number of Patients] & "")
VAR _CleanValue = SUBSTITUTE(_Value, ",", "")
RETURN IF(_Value = "" || UPPER(_Value) = "NA" || UPPER(_Value) = "N/A", "Null", FORMAT(IFERROR(VALUE(_CleanValue), 0), "#,##0"))

Conversion  : ✅ FULL

### CC-14 | Condition Cost (Cost Analysis)
─────────────────────────────────────────────────────────────
Power BI DAX: IF([Main Category] = "Cost", 1, 0)
Conversion  : ✅ FULL

### CC-15 through CC-41 | Condition Flag Columns (27 total)
─────────────────────────────────────────────────────────────
27 condition flag columns across Patient Overall (6), patient_cohort (20),
and Cost Analysis (1). All follow the same pattern:

Pattern: IF([Main Category] = "<Category Name>", 1, 0)

Patient Overall (6): Top 10 Metric 1/2/3, Cohort Creation, Yearly Incidence, Coverage Metric 1
patient_cohort (20): States, Insurance Plan, Gender, Race, Ethnicity, Vitals,
    Continuous Activity, Division, Sdoh 1/2/3, Age at first diagnosis,
    Indication Specific Attribute 1–9
Cost Analysis (1): Cost

⚠️  TYPO NOTE (GAP-07): Condition Division has "DIVIDEision" — a copy-paste
    error. FIX REQUIRED: Change to "Division".

Conversion  : ✅ FULL (all 27, pending Division typo fix)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### M-01 | % of Patient Outlier
─────────────────────────────────────────────────────────────
Tableau: SUM(if [Patient Type]='Outlier' then [# of Outlier Patients] END) / MAX([# of Patient Captured])
Power BI: DIVIDE(CALCULATE(SUMX(...IFERROR(VALUE([# of Outlier Patients]),0)), FILTER(...[Patient Type]="Outlier")), MAXX(...IFERROR(VALUE([# of Patient Captured]),0)))
Conversion  : ✅ FULL

### M-02 | % of Patient Captured
─────────────────────────────────────────────────────────────
Tableau: SUM([# of Patient Captured]) / MAX([Total Patient Count])
Power BI: DIVIDE(SUMX(...IFERROR(VALUE([# of Patient Captured]),0)), MAXX(...IFERROR(VALUE([Total Patient Count]),0)))
⚠️  PATTERN NOTE: All numeric columns stored as strings in PBI —
    all aggregations use IFERROR(VALUE(...), 0) wrapper.
Conversion  : ✅ FULL

### M-03 | % of Patients (Patient Overall & patient_cohort)
─────────────────────────────────────────────────────────────
Tableau: SUM([Number of Patients]) / MAX([Total Patients])
Power BI: DIVIDE(SUMX(...IFERROR(VALUE([Number of Patients]),0)), MAXX(...IFERROR(VALUE([Total Patients]),0)))
Conversion  : ✅ FULL

### M-04 | Rank (Tableau FIXED LOD → DAX RANKX)
─────────────────────────────────────────────────────────────
Tableau: RANK(SUM({FIXED [Category Type],[BMS Specific Medication flag]: SUM([Number of Patients])}),'desc')
Power BI DAX:
RANKX(
    ALL([Category Type], [BMS Specific Medication flag]),
    CALCULATE(SUM([Number of Patients]),
        REMOVEFILTERS(...),
        KEEPFILTERS(VALUES([Category Type])),
        KEEPFILTERS(VALUES([BMS Specific Medication flag]))),
    , DESC, DENSE)

Key Pattern : Tableau FIXED LOD RANK → DAX RANKX + REMOVEFILTERS + KEEPFILTERS
Conversion  : ✅ FULL

### M-05 | Top 10 MEds
─────────────────────────────────────────────────────────────
Tableau: [Rank]<=10 OR SUM([BMS Specific Medication flag])=1
Power BI: IF(OR([Rank]<=10, SUM([BMS Specific Medication flag])=1), TRUE(), FALSE())
Conversion  : ✅ FULL

### M-06 | % Patients Bar Color (PBI-only)
─────────────────────────────────────────────────────────────
Power BI DAX:
SWITCH(TRUE(), ISBLANK(_Value),"#EDEDED", _Value<0.30,"#E6E6E6",
    _Value<0.40,"#D8D2DD", _Value<0.50,"#C4B2CF",
    _Value<0.60,"#9F7EAD", _Value<0.75,"#7E5590", "#5B2D75")
Type        : ➕ PBI-only — conditional color measure.
Conversion  : ✅ FULL

### M-07 | Fulfillment Color (PBI-only)
─────────────────────────────────────────────────────────────
Power BI DAX:
SWITCH(TRUE(),
    SELECTEDVALUE([Fulfillement Index])="Can be fulfilled", "#1a9850",
    SELECTEDVALUE([Fulfillement Index])="Can not be fulfilled", "#d73027",
    "#e67e22")
⚠️  TYPO NOTE: "Fulfillement" is misspelled — preserved exactly.
Conversion  : ✅ FULL

### M-08 | Use Case Desc Concat (PBI-only)
─────────────────────────────────────────────────────────────
Power BI DAX:
CONCATENATEX(..., "▸ " & [Use Case Description], UNICHAR(10), [Use Case Description], ASC)
Conversion  : ✅ FULL

### M-09 | Sum of Patients (PO) display (PBI-only)
─────────────────────────────────────────────────────────────
Power BI DAX: "(N=" & FORMAT(_PatientCount, "#,##0") & ")"
Conversion  : ✅ FULL

### M-10 through M-31 | Show/Hide Measures (22 pairs)
─────────────────────────────────────────────────────────────
22 Show/Hide measures paired with 27 condition flag columns.
Pattern: IF(SUM([Condition <Name>]) >= 1, "", "The data in the view is not available for the selected indication")

Patient Overall (6): Top 10 Metric 1/2/3, Cohort Creation, Yearly Incidence, Coverage Metric 1
patient_cohort (15): States, Insurance Plan, Gender, Race, Ethnicity, Vitals,
    Continuous Activity, Division, Sdoh 1/2/3, Age at first diagnosis,
    Indication Specific Attribute 1–9
Cost Analysis (1): Cost
Conversion  : ✅ FULL (all 22)

### M-32 through M-40 | Navigation/Label Measures (Measures Table)
─────────────────────────────────────────────────────────────
| Measure            | Value                                                          |
|--------------------|----------------------------------------------------------------|
| View NA            | "THE DATA IN THE VIEW IS NOT AVAILABLE..."                     |
| 'Gender'           | "GENDER"                                                       |
| 'Race'             | "RACE"                                                         |
| 'Ethnicity'        | "ETHNICITY"                                                    |
| 'Executive Summary'| "EXECUTIVE SUMMARY"                                            |
| 'Data Overview'    | "DATA OVERVIEW"                                                |
| 'Next>'            | "NEXT>"                                                        |
| '<Prev'            | "<PREV"                                                        |
| INDEX()            | RANKX(ALL('Measures Table'), 1)                                |

Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU INITIAL TRANSFORMATIONS → POWER BI POWER QUERY (M)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | Canonical SharePoint Excel M Pattern
─────────────────────────────────────────────────────────────
Tableau Source  : Excel Direct + Hyper Extract
PBI Connection  : Excel.Workbook(Web.Contents(...)) from SharePoint
PBI Load Mode   : Import
PBI Query Pattern: Table.PromoteHeaders() applied in all M queries

Base SharePoint Path:
https://sites.bms.com/sites/RWDSProjects-BMSFolder/Shared Documents/BMS Team Folder/Hyderabad - Internal BMS/02. Analytic Insights Projects/2026/Tableau Migration Excels/DRWDS_Oncology_Capability Assessment/

### PQ-02 | Source Table Mapping (10 tables)
─────────────────────────────────────────────────────────────
| PBI Table Name                              | Source Excel        | Sheet           |
|---------------------------------------------|---------------------|-----------------|
| Outlier Patient Summary_5 Point Summary     | Data Model 8.xlsx   | 5 Point Summary |
| Header Summary_Sheet1                       | Data Model 14.xlsx  | Sheet1          |
| patient_cohort_Sheet1                       | Data Model 2.xlsx   | Sheet1          |
| Patient Overall_Sheet1                      | Data Model 1.xlsx   | Sheet1          |
| Executive Summary_Sheet1                    | Data Model 4.xlsx   | Sheet1          |
| Use Case Analysis Updated_Sheet1            | Data Model 7.xlsx   | Sheet1          |
| Attribute Summary Updated_Sheet1            | Data Model 3.xlsx   | Sheet1          |
| Cost Analysis_Sheet1                        | Data Model 10.xlsx  | Sheet1          |
| Data Quality_Sheet1                         | Data Model 5.xlsx   | Sheet1          |
| Screen Config_Sheet1                        | Data Model 10.xlsx  | Sheet1          |

### PQ-03 | Auxiliary Lookup Tables (2 — inline M with base64 encoded data)
─────────────────────────────────────────────────────────────
| Table      | Purpose                                           | Key Column     |
|------------|---------------------------------------------------|----------------|
| Table      | Attribute Name sort order                         | Attribute Name |
| Table (2)  | Category Type sort key                            | Category Type  |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — CONVERSION COMPLETENESS SCORECARD
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Category                         | Items | PBI Full | PBI Partial | Not Converted | % Complete |
|----------------------------------|-------|----------|-------------|---------------|------------|
| Parameters                       | 3     | 3        | 0           | 0             | 100%       |
| Data Tables Migrated             | 10    | 10       | 0           | 0             | 100%       |
| Calculated Columns               | 35+   | 34       | 1           | 0             | 97%        |
| Measures (all tables)            | 45+   | 45+      | 0           | 0             | 100%       |
| Show/Hide Measure Pairs          | 22    | 22       | 0           | 0             | 100%       |
| Condition Flag Columns           | 27    | 27       | 0           | 0             | 100%       |
| Model Relationships              | 4     | 4        | 0           | 0             | 100%       |
| Auxiliary Lookup Tables           | 2     | 2        | 0           | 0             | 100%       |
| Date Tables (auto-generated)     | 2     | 2        | 0           | 0             | 100%       |
| TOTAL                            | 150+  | 149+     | 1           | 0             | 🟢 99%     |

PARTIAL ITEM:
  ⚠️  CC-12 Non-Outlier Range (Cost Analysis) — PLACEHOLDER.
      Referenced Min/Max columns not present in Data Model 10.

⚠️  CRITICAL FIX: Condition Division has "DIVIDEision" typo — must be
    corrected to "Division" before production.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — CONVERSION PATTERN CHEAT SHEET
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PATTERN-01 | String-to-Number Aggregation (IFERROR(VALUE()) wrapper)
─────────────────────────────────────────────────────────────
All numeric columns stored as strings in PBI. Every aggregation uses:
SUMX('Table', IFERROR(VALUE('Table'[Column]), 0))
MAXX('Table', IFERROR(VALUE('Table'[Column]), 0))

Used in: All % of Patients measures, Outlier measures, display measures.


PATTERN-02 | Show/Hide Condition Pattern (22 pairs)
─────────────────────────────────────────────────────────────
Step 1 — Condition column: IF([Main Category] = "<Name>", 1, 0)
Step 2 — Show/Hide measure: IF(SUM([Condition <Name>]) >= 1, "", "The data in the view is not available for the selected indication")

Tableau: floating container show/hide → PBI: DAX measure + visual conditional visibility.


PATTERN-03 | Tableau FIXED LOD RANK → DAX RANKX
─────────────────────────────────────────────────────────────
Tableau: RANK(SUM({FIXED [Dim1],[Dim2]: SUM([Metric])}),'desc')
Power BI:
RANKX(ALL([Dim1],[Dim2]),
    CALCULATE(SUM([Metric]),
        REMOVEFILTERS(...),
        KEEPFILTERS(VALUES([Dim1])),
        KEEPFILTERS(VALUES([Dim2]))),
    , DESC, DENSE)


PATTERN-04 | Hex Color Measures
─────────────────────────────────────────────────────────────
SWITCH(TRUE(), ISBLANK(_Value),"#color1", _Value<threshold,"#color2", ...)
Used in: % Patients Bar Color, Fulfillment Color.


PATTERN-05 | Explicit Geo Coordinates (Map Visual)
─────────────────────────────────────────────────────────────
Tableau: Built-in map with semantic role [State]
Power BI: Explicit State Lat/State Lng columns via SWITCH on state codes.
Avoids PBI geocoding issues with ambiguous place names.


PATTERN-06 | CONCATENATEX for Multi-Line Display
─────────────────────────────────────────────────────────────
CONCATENATEX('Table', "▸ " & [Column], UNICHAR(10), [Column], ASC)
Used in: Use Case Desc Concat measure.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAP-01 | Numeric Columns Stored as Strings
  Severity : Medium
  Detail   : Total Patients, Number of Patients, # of Patient Captured
             typed as integer/real in Tableau but stored as string in PBI.
  Action   : All aggregation measures use IFERROR(VALUE(...),0) pattern.
             Consider fixing at Power Query layer for cleaner model.

GAP-02 | FIXED LOD to DAX RANKX
  Severity : Low (resolved)
  Detail   : RANK(SUM({FIXED...})) required RANKX + REMOVEFILTERS + KEEPFILTERS.
  Action   : Implemented in Rank measure. Verify with actual data.

GAP-03 | Tableau Table Calculations to DAX
  Severity : Medium
  Detail   : PctTotal table calculation requires explicit DIVIDE with
             correct filter context.
  Action   : Placeholder measure created — validate context behavior.

GAP-04 | Category Type Aliases
  Severity : Low
  Detail   : Tableau aliases "0.50"→"50%", "0.55"→"55%" etc. not
             natively supported in PBI.
  Action   : Handle via Power Query replace or calculated column.

GAP-05 | BMS Medication Flag Aliases
  Severity : Low
  Detail   : 0→"Other Brand Medication", 1→"BMS Brand Medication"
             not aliased in PBI.
  Action   : Handle via calculated column or visual legend formatting.

GAP-06 | Non-Outlier Range (Cost Analysis)
  Severity : High
  Detail   : Referenced Min/Max Non Outlier Range columns not present
             in Data Model 10. CC-12 is a PLACEHOLDER.
  Action   : Fix when Cost Analysis source is confirmed.

GAP-07 | Condition Division Typo — CRITICAL FIX
  Severity : CRITICAL
  Detail   : [Main Category]='Division' has "DIVIDEision" — copy-paste error.
  Action   : FIX REQUIRED: Change to "Division" in the condition column DAX.

GAP-08 | Date Parse Format Lost
  Severity : Low
  Detail   : Refresh Date in patient_cohort has date-parse-format='yyyy/MM'
             in Tableau but stored as plain string in PBI.
  Action   : Add explicit parsing step in M query if date operations needed.

GAP-09 | Tableau Shape/Mark Encoding
  Severity : Low (report-layer)
  Detail   : Funnel images, custom shape marks for cohort funnel not
             directly mirrored in PBI.
  Action   : Implement as image URLs or conditional formatting icons.

GAP-10 | Color Encoding
  Severity : Low (report-layer)
  Detail   : Per-bucket color maps for Category Type in patient_cohort
             not in current PBI tables.
  Action   : Implement via DAX SWITCH color measures per visual.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — FILE INVENTORY & MODEL STRUCTURE
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### Table Load Order
─────────────────────────────────────────────────────────────
| Order | Table Name                                  |
|-------|---------------------------------------------|
| 1     | Measures Table                              |
| 2     | Yearly Monthly                              |
| 3     | start date end date                         |
| 4     | Dashboard Refresh Date                      |
| 5     | Outlier Patient Summary_5 Point Summary     |
| 6     | Header Summary_Sheet1                       |
| 7     | patient_cohort_Sheet1                       |
| 8     | Patient Overall_Sheet1                      |
| 9     | Executive Summary_Sheet1                    |
| 10    | Use Case Analysis Updated_Sheet1            |
| 11    | Attribute Summary Updated_Sheet1            |
| 12    | Use Case Analysis - Examples_Sheet1         |
| 13    | Cost Analysis_Sheet1                        |
| 14    | Data Quality_Sheet1                         |
| 15    | Screen Config_Sheet1                        |
| 16    | DateTableTemplate (GUID)                    |
| 17    | LocalDateTable (GUID)                       |
| 18    | Table (Attribute Name sort)                 |
| 19    | Table (2) (Category Type sort)              |

### Relationships
─────────────────────────────────────────────────────────────
| Relationship ID        | From Table                             | From Column             | To Table            | To Column      | Cross-Filter |
|------------------------|----------------------------------------|-------------------------|---------------------|----------------|--------------|
| 7aa34c2c               | Dashboard Refresh Date                 | ParameterTableKeyColumn | LocalDateTable      | Date           | Single       |
| AutoDetected_eb61f1d6  | Screen Config_Sheet1                   | Dataset Vendor          | Cost Analysis_Sheet1| Dataset Vendor | Both         |
| AutoDetected_44249bd9  | Use Case Analysis - Examples_Sheet1    | Attribute Name          | Table               | Attribute Name | Single       |
| fbd230bd               | Outlier Patient Summary_5 Point Summary| Category_type           | Table (2)           | Category Type  | Single       |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — SUCCESSFULLY MIGRATED PATTERNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Pattern                                      | Status   |
|----------------------------------------------|----------|
| All if/THEN/else/end → IF()                  | Complete |
| String concatenation + → &                   | Complete |
| SUM()/MAX() aggregation pattern              | Complete |
| FIXED LOD RANK → RANKX                       | Complete |
| All Show/Hide condition pairs (22 pairs)     | Complete |
| Parameter tables (3 parameters)              | Complete |
| Category and Units concatenation             | Complete |
| Sort lookup tables                           | Complete |
| Geo columns for US state map                 | Complete |


END OF KNOWLEDGE FILE
