# 📘 CONVERSION KNOWLEDGE FILE
# 24670 — GRS vNavigator Data Quality GSP
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Definitive reference for converting and maintaining the
#          GRS vNavigator Data Quality GSP report in PBIP.
#          Covers all data sources, parameters, calculated columns,
#          measures, filters, slicers, and DAX/Power Query logic.
# Generated: 2026-07-06
# Source files:
#   Tableau : 24670_grs_vnavigator_data_quality_gsp.txt
#   PBIP    : bargraph2.txt, dateparameter.txt, dim_date.txt,
#             kpi_labels.txt, kpi_labels_new.txt, kpi_summary.txt,
#             param_country_theraparea.txt, query1.txt, topn.txt

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 0 — WORKBOOK OVERVIEW
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Tableau Workbook Name   : GRS vNavigator Data Quality GSP
Tableau URL             : https://tableau.bms.com/t/PRODUCTION/workbooks/GRSvNavigatorDataQualityGSP
Tableau Revision        : 1.0 (derived from 1.2)
Tableau Source Build    : 2024.2.13 (20242.25.0710.1539)
Tableau Version         : 18.1
Tableau Site            : PRODUCTION

PBIP Report Name        : GRS vNavigator Data Quality GSP
Model Compatibility     : 1600, schema v4.2, culture en-US
Load Mode               : Import (all tables)

Data Source             : Amazon Redshift Serverless (UAT → PROD)
DB Server (UAT, in M)   : vnavigator.456939526755.us-east-1.redshift-serverless.amazonaws.com:5439
DB Server (PROD target) : vnavigator.311406940810.us-east-1.redshift-serverless.amazonaws.com
Database                : vnavigator_uat (M) → vnavigator_prod (PROD)
Schema                  : vnavigator_v
View/Table              : v_dq_mf_hai_gsp
Auth                    : auth-user-pass (username: vnavigator_analyst)

⚠️ IMPORTANT: The M partition currently points to the UAT endpoint
   (vnavigator.456939526755.us-east-1...vnavigator_uat).
   Before production deployment, update to:
   Server  : vnavigator.311406940810.us-east-1.redshift-serverless.amazonaws.com
   Database: vnavigator_prod

PBIP Tables             : 7
  1. Query1              — Main fact table (v_dq_mf_hai_gsp), all KPI measures
  2. Dim_Date            — Master date dimension (2015–2050), fiscal year logic
  3. DateParameter       — Disconnected date slicer (2020–2030)
  4. BarGraph2           — Toggle: Missing Records vs Non-Adherence view
  5. TopN                — Dynamic Top-N slicer (1–50)
  6. Param_Country_TherapArea — Field parameter: Country vs Therapeutic Area
  7. KPI_Labels / KPI_Labels_New / KPI_Summary — KPI display metadata tables

LocalDateTables         : Auto-generated per dateTime column, hidden, GUID-bound
Tableau Parameters      : 5 (fully mapped — see Section 1)

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PARAM-01 | Date
─────────────────────────────────────────────────────────────
Tableau Caption  : Date
Tableau Type     : date
Default Value    : 2024-12-31
Domain           : Continuous date range

Business Purpose :
  Acts as the primary "as-of" date for all 18-month rolling
  window calculations. All KPI measures resolve their record
  universe relative to this selected date.

Power BI Implementation:
  Table    : Dim_Date
  Column   : Date (CALENDAR(DATE(2015,1,1), DATE(2050,12,31)))
  Column   : date1 = FORMAT(Dim_Date[Date], "MM/DD/YYYY")
             (used as the slicer-visible display label)

  In measures, the selected date is captured as:
    VAR SelectedDate = DATEVALUE(MAX(Dim_Date[Date1]))
    -- or --
    VAR SelectedDate = MAX(Dim_Date[Date])
    -- or --
    VAR SelectedDate = SELECTEDVALUE(Dim_Date[Date], DATE(2024,12,31))

  ⚠️ NOTE: Three variants of the SelectedDate pattern exist across
  measures. The [Date1] (string column) variant uses DATEVALUE(MAX(...))
  while the [Date] (datetime column) variants use MAX() or SELECTEDVALUE().
  Both resolve to the same value — but ensure the slicer is bound to
  [date1] for display and filter propagation as designed.

Default Slicer   : DATE(2024, 12, 31)
Conversion Status: ✅ FULL


### PARAM-02 | TopN
─────────────────────────────────────────────────────────────
Tableau Caption  : TopN (used for "Top N defaulters by Country/TA")
Tableau Type     : integer
Default Value    : 10

Business Purpose :
  Controls how many bars appear in the "Top N Defaulters"
  bar chart (by Country or Therapeutic Area). Only the top N
  items by TotalMissing are shown; others are blanked out.

Power BI Implementation:
  Table   : TopN
  DAX     : GENERATESERIES(1, 50, 1)
  Column  : TopN (integer, 1–50)
  Measure : 'TopN Value' = SELECTEDVALUE('TopN'[TopN], 10)
            -- default = 10 when no selection

  Usage pattern in bar measures:
    VAR Limit = [TopN Value]
    VAR InLimit = NOT ISBLANK(RankNo) && RankNo >= 1 && RankNo <= Limit
    RETURN IF(InLimit, value, BLANK())

Default Slicer   : 10
Conversion Status: ✅ FULL


### PARAM-03 | BarGraph (Country / Therapeutic Area)
─────────────────────────────────────────────────────────────
Tableau Caption  : BarGraph (or "Country / Therapeutic Area")
Tableau Type     : string / boolean toggle
Default Value    : "Country" (order = 0)

Business Purpose :
  Switches the bar chart dimension between Country (filing_country)
  and Therapeutic Area (disease_area_standardized).
  Drives ranking, bar measure display, and dynamic title generation.

Power BI Implementation:
  Table   : Param_Country_TherapArea
  Type    : Field Parameter (Power BI native)
  Columns :
    Parameter        — Display label ("Country" or "Therapeutic Area")
                       sourceColumn: [Value1]; sortBy: Parameter Order
    Parameter Fields — Hidden field reference (NAMEOF column)
                       sourceColumn: [Value2]; marked isHidden
    Parameter Order  — Sort key (0 = Country, 1 = Therapeutic Area)
                       sourceColumn: [Value3]; marked isHidden

  DAX partition:
    {
      ("Country",           NAMEOF('Query1'[filing_country]),              0),
      ("Therapeutic Area",  NAMEOF('Query1'[disease_area_standardized]),   1)
    }

  Measure : 'Param_Country_TherapArea Fields' =
    SELECTEDVALUE('Param_Country_TherapArea'[Parameter Order], 0)
    -- Returns 0 for Country (default), 1 for Therapeutic Area

  Usage pattern in measures:
    VAR SelectedField = [Param_Country_TherapArea Fields]
    VAR RankNo = IF(SelectedField = 0, [Country_RankNo], [TherapArea_RankNo])

Default Slicer   : "Country" (Parameter Order = 0)
Conversion Status: ✅ FULL


### PARAM-04 | Date month (Month slicer for trend chart)
─────────────────────────────────────────────────────────────
Tableau Caption  : Date month (used for "Monthly Trend" view)
Tableau Type     : date (month granularity)
Default Value    : n/a (all months shown)

Business Purpose :
  Provides month-level filtering for the 12-month trend line.
  In PBIP this is the DateParameter table used as an additional
  date slicer for the monthly trend visuals.

Power BI Implementation:
  Table   : DateParameter
  DAX     : CALENDAR(DATE(2020,1,1), DATE(2030,12,31))
  Column  : Date

  ⚠️ NOTE: DateParameter is a SEPARATE disconnected slicer table.
  It is NOT the same as Dim_Date. Use DateParameter[Date] for the
  monthly trend slicer and Dim_Date[date1] for the main KPI date picker.

Conversion Status: ✅ FULL


### PARAM-05 | BarGraph2 (Missing Records / Non-Adherence)
─────────────────────────────────────────────────────────────
Tableau Caption  : BarGraph2 (or "Deviations / Non-Adherence" toggle)
Tableau Type     : boolean / list toggle
Members          : 1 → "Missing Records", 2 → "Non-Adherence" (or similar)

Business Purpose :
  Toggles the bar chart display between the "Missing Records"
  KPI view (KPI 1–5 flag measures) and the "Non-Adherence"
  view (NonAdhKPI1–NonAdhKPI5 measures).

Power BI Implementation:
  Table   : BarGraph2
  Type    : Inline parameter table (Base64-compressed Table.FromRows)
  Columns :
    Value        — Integer key (1 or 2)
    ' Display As' — String label (⚠️ leading space in column name)

  Measures :
    BarGraph2_Selected = SELECTEDVALUE(BarGraph2[Value], 1)
      -- returns 1 (Missing Records) if nothing selected

    BarGraphTop = IF([BarGraph2_Selected] = 1, TRUE, FALSE)
      -- format: "TRUE";"TRUE";"FALSE"
      -- used as a visual filter flag

    BarChartTop_KPI% = SWITCH(
        SELECTEDVALUE(KPI_Labels[KPI_Name]),
        "KPI1",  COALESCE(1 - DIVIDE([PlannedSubDateNull?], [Record], 0), 0),
        "KPI2",  COALESCE(1 - DIVIDE([ActualSubNull?],      [Record], 0), 0),
        "KPI3",  COALESCE(1 - DIVIDE([ApplicationIDNull?],  [Record], 0), 0),
        "KPI4",  COALESCE(1 - DIVIDE([RegObjNameNull?],     [Record], 0), 0),
        "KPI5",  COALESCE(1 - DIVIDE([ActualApprovalNull?], [Record], 0), 0)
    )
      -- keyed on KPI_Labels[KPI_Name] string

    BarChartTop_KPI% new = SWITCH(
        SELECTEDVALUE(KPI_Summary[KPI_Order]),
        1,  COALESCE(1 - DIVIDE([PlannedSubDateNull?], [Record], 0), 0),
        2,  COALESCE(1 - DIVIDE([ActualSubNull?],      [Record], 0), 0),
        3,  COALESCE(1 - DIVIDE([ApplicationIDNull?],  [Record], 0), 0),
        4,  COALESCE(1 - DIVIDE([RegObjNameNull?],     [Record], 0), 0),
        5,  COALESCE(1 - DIVIDE([ActualApprovalNull?], [Record], 0), 0)
    )
      -- keyed on KPI_Summary[KPI_Order] integer (preferred version)

  ⚠️ CRITICAL: ' Display As' has a leading space. Reference it as:
    BarGraph2[ Display As]  -- NOT BarGraph2[Display As]

Default Slicer   : Value = 1 ("Missing Records")
Conversion Status: ✅ FULL

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — DATA SOURCE AND POWER QUERY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### TABLE: Query1 (main fact table)
─────────────────────────────────────────────────────────────
Source View  : vnavigator_v.v_dq_mf_hai_gsp
Connector    : AmazonRedshift.Database
Server (UAT) : vnavigator.456939526755.us-east-1.redshift-serverless.amazonaws.com:5439
Database     : vnavigator_uat
Load Mode    : Import
EnableFolding: true

#### Source Columns (37 total from SQL SELECT):

#   DB Column                       PBI Column Name                       PBI Type    Notes
──  ──────────────────────────────  ────────────────────────────────────  ──────────  ─────────────────────────────────────────────
1   acceleration_mechanism_confirmed acceleration_mechanism_confirmed     string      CAST AS TEXT
2   acceleration_mechanism_potential acceleration_mechanism_potential     string      CAST AS TEXT
3   activity_comments               activity_comments                     string      CAST AS TEXT
4   activity_id                     activity_id                           string
5   activity_name                   activity_name                         string      CAST AS TEXT
6   activity_status                 activity_status                       string      Key filter field in NonAdhKPI measures
7   activity_type                   activity_type                         string
8   actual_approval_date            actual_approval_date                  dateTime    Long Date format; UnderlyingDateType = Date
9   actual_submission_date          actual_submission_date                dateTime    Long Date format; UnderlyingDateType = Date
10  application_id                  application_id                        string      CAST AS TEXT; NULL check = KPI3 flag
11  base_case_approval_date         base_case_approval_date               dateTime    Long Date format; used in ApplicationIDNull?
12  best_case_approval_date         best_case_approval_date               dateTime    Long Date format; used in NonAdhKPI2
13  country                         country                               string      Country of record (vs. filing_country)
14  data_gap_cat                    data_gap_cat                          string      Used in Filtered_Rows measure
15  description                     description                           string      CAST AS TEXT
16  disease_area_standardized       disease_area_standardized             string      CAST AS TEXT; TA dimension for bar chart
17  event                           event                                 string      CAST AS TEXT
18  event_name                      event_name                            string      CAST AS TEXT
19  event_status                    event_status                          string      Used in NonAdhKPI5 ("Cancelled" check)
20  filing_country                  filing_country                        string      Country dimension for bar chart
21  generic_name                    generic_name                          string      CAST AS TEXT; basis for generic_name (groups)
22  ha_interaction_date             ha_interaction_date                   dateTime    Long Date format
23  ha_interaction_name             ha_interaction_name                   string      CAST AS TEXT
24  ha_interaction_status           ha_interaction_status                 string      CAST AS TEXT
25  ha_interaction_type             ha_interaction_type                   string      CAST AS TEXT
26  is_major_filing                 is_major_filing                       integer     Used in RegObjNameNull? (= 1 gate)
27  market                          market                                string      CAST AS TEXT
28  planned_approval_date           planned_approval_date                 dateTime    Long Date format
29  planned_short_indication        planned_short_indication              string      CAST AS TEXT
30  planned_submission_date         planned_submission_date               dateTime    Long Date format; KEY date field
31  population_scope                population_scope                      string
32  primary_region                  primary_region                        string
33  product_group                   product_group                         string
34  project_orbis                   project_orbis                         string
35  reg_objective_name              reg_objective_name                    string      NULL check = KPI4 flag
36  regulatory_objective_status     regulatory_objective_status           string      Used in NonAdhKPI1/3/4
37  unique_key                      unique_key                            string      CAST AS TEXT

#### Power Query M — Post-load transformations:
──────────────────────────────────────────────────
  1. Duplicate planned_submission_date → extract Year
     → new column: "planned submission date year" (Int64)
  2. Duplicate planned_submission_date → extract Month
     → new column: "planned_submission_date  Month" (Int64)
     ⚠️ NOTE: Double space in "planned_submission_date  Month" —
        must be preserved exactly.

#### Full Power Query M:
────────────────────────
let
    Source = Value.NativeQuery(
        AmazonRedshift.Database(
            "vnavigator.456939526755.us-east-1.redshift-serverless.amazonaws.com:5439",
            "vnavigator_uat"
        ),
        "SELECT
          CAST(""v_dq_mf_hai_gsp"".""acceleration_mechanism_confirmed"" AS TEXT) AS ""acceleration_mechanism_confirmed"",
          CAST(""v_dq_mf_hai_gsp"".""acceleration_mechanism_potential"" AS TEXT) AS ""acceleration_mechanism_potential"",
          CAST(""v_dq_mf_hai_gsp"".""activity_comments"" AS TEXT) AS ""activity_comments"",
          ""v_dq_mf_hai_gsp"".""activity_id"" AS ""activity_id"",
          CAST(""v_dq_mf_hai_gsp"".""activity_name"" AS TEXT) AS ""activity_name"",
          ""v_dq_mf_hai_gsp"".""activity_status"" AS ""activity_status"",
          ""v_dq_mf_hai_gsp"".""activity_type"" AS ""activity_type"",
          ""v_dq_mf_hai_gsp"".""actual_approval_date"" AS ""actual_approval_date"",
          ""v_dq_mf_hai_gsp"".""actual_submission_date"" AS ""actual_submission_date"",
          CAST(""v_dq_mf_hai_gsp"".""application_id"" AS TEXT) AS ""application_id"",
          ""v_dq_mf_hai_gsp"".""base_case_approval_date"" AS ""base_case_approval_date"",
          ""v_dq_mf_hai_gsp"".""best_case_approval_date"" AS ""best_case_approval_date"",
          ""v_dq_mf_hai_gsp"".""country"" AS ""country"",
          ""v_dq_mf_hai_gsp"".""data_gap_cat"" AS ""data_gap_cat"",
          CAST(""v_dq_mf_hai_gsp"".""description"" AS TEXT) AS ""description"",
          CAST(""v_dq_mf_hai_gsp"".""disease_area_standardized"" AS TEXT) AS ""disease_area_standardized"",
          CAST(""v_dq_mf_hai_gsp"".""event"" AS TEXT) AS ""event"",
          CAST(""v_dq_mf_hai_gsp"".""event_name"" AS TEXT) AS ""event_name"",
          ""v_dq_mf_hai_gsp"".""event_status"" AS ""event_status"",
          ""v_dq_mf_hai_gsp"".""filing_country"" AS ""filing_country"",
          CAST(""v_dq_mf_hai_gsp"".""generic_name"" AS TEXT) AS ""generic_name"",
          ""v_dq_mf_hai_gsp"".""ha_interaction_date"" AS ""ha_interaction_date"",
          CAST(""v_dq_mf_hai_gsp"".""ha_interaction_name"" AS TEXT) AS ""ha_interaction_name"",
          CAST(""v_dq_mf_hai_gsp"".""ha_interaction_status"" AS TEXT) AS ""ha_interaction_status"",
          CAST(""v_dq_mf_hai_gsp"".""ha_interaction_type"" AS TEXT) AS ""ha_interaction_type"",
          ""v_dq_mf_hai_gsp"".""is_major_filing"" AS ""is_major_filing"",
          CAST(""v_dq_mf_hai_gsp"".""market"" AS TEXT) AS ""market"",
          ""v_dq_mf_hai_gsp"".""planned_approval_date"" AS ""planned_approval_date"",
          CAST(""v_dq_mf_hai_gsp"".""planned_short_indication"" AS TEXT) AS ""planned_short_indication"",
          ""v_dq_mf_hai_gsp"".""planned_submission_date"" AS ""planned_submission_date"",
          ""v_dq_mf_hai_gsp"".""population_scope"" AS ""population_scope"",
          ""v_dq_mf_hai_gsp"".""primary_region"" AS ""primary_region"",
          ""v_dq_mf_hai_gsp"".""product_group"" AS ""product_group"",
          ""v_dq_mf_hai_gsp"".""project_orbis"" AS ""project_orbis"",
          ""v_dq_mf_hai_gsp"".""reg_objective_name"" AS ""reg_objective_name"",
          ""v_dq_mf_hai_gsp"".""regulatory_objective_status"" AS ""regulatory_objective_status"",
          CAST(""v_dq_mf_hai_gsp"".""unique_key"" AS TEXT) AS ""unique_key""
        FROM ""vnavigator_v"".""v_dq_mf_hai_gsp"" ""v_dq_mf_hai_gsp""",
        null,
        [EnableFolding = true]
    ),
    #"Duplicated Column"  = Table.DuplicateColumn(Source, "planned_submission_date", "planned_submission_date - Copy"),
    #"Extracted Year"     = Table.TransformColumns(#"Duplicated Column",  {{"planned_submission_date - Copy", Date.Year,  Int64.Type}}),
    #"Renamed Columns"    = Table.RenameColumns(#"Extracted Year",   {{"planned_submission_date - Copy", "planned submission date year"}}),
    #"Duplicated Column1" = Table.DuplicateColumn(#"Renamed Columns", "planned_submission_date", "planned_submission_date - Copy"),
    #"Extracted Month"    = Table.TransformColumns(#"Duplicated Column1", {{"planned_submission_date - Copy", Date.Month, Int64.Type}}),
    #"Renamed Columns1"   = Table.RenameColumns(#"Extracted Month",  {{"planned_submission_date - Copy", "planned_submission_date  Month"}})
in
    #"Renamed Columns1"

Conversion Status: ✅ FULL

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — DIM_DATE TABLE (Date Dimension)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

DAX Source:
  Dim_Date = ADDCOLUMNS(
      CALENDAR(DATE(2015,1,1), DATE(2050,12,31)),
      "Day",             DAY([Date]),
      "Month Number",    MONTH([Date]),
      "Day Name",        FORMAT([Date], "dddd"),
      "Month Name",      FORMAT([Date], "MMMM"),
      "Month Short",     FORMAT([Date], "MMM"),
      "Year",            YEAR([Date]),
      "Quarter",         "Q" & QUARTER([Date]),
      "Fiscal Year",     "FY" & IF(MONTH([Date]) >= 7, YEAR([Date]) + 1, YEAR([Date])),
      "Month Year",      FORMAT([Date], "MMM-yyyy"),
      "Month Year Sort", FORMAT([Date], "yyyyMM"),
      "date1",           FORMAT(Dim_Date[Date], "MM/DD/YYYY")
  )

Key Columns:
  Date           : Base dateTime; primary join target for LocalDateTables
  date1          : String display label "MM/DD/YYYY"; used as slicer face value;
                   sortByColumn = "Month Year Sort"
  Fiscal Year    : "FY" prefix; BMS fiscal year starts July 1
  Month Year Sort: Numeric "yyyyMM" — use as sort key for Month Year column

Measures on Dim_Date:
  Selected Date     = SELECTEDVALUE(Dim_Date[Date], MAX(Dim_Date[Date]))
  Selected Date new = MAX(Dim_Date[Date])

⚠️ NOTE: Measures that use DATEVALUE(MAX(Dim_Date[Date1])) expect
  [Date1] to be the string column "MM/DD/YYYY". Ensure DATEVALUE
  can parse this format in your locale. If locale causes issues,
  consider switching to: SELECTEDVALUE(Dim_Date[Date], DATE(2024,12,31)).

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TOPN TABLE
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

DAX Source  : GENERATESERIES(1, 50, 1)
Column      : TopN (integer 1–50; summarizeBy: none)
Measure     : 'TopN Value' = SELECTEDVALUE('TopN'[TopN], 10)
              -- default = 10 when no selection made

Usage: All "_Bar" measures use [TopN Value] as the upper rank limit:
  VAR Limit = [TopN Value]
  VAR InLimit = RankNo >= 1 && RankNo <= Limit
  RETURN IF(InLimit, value, BLANK())

Slicer Type : Single-select numeric slicer (or dropdown)
Default     : 10

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — CORE MEASURES: RECORD UNIVERSE (18-MONTH WINDOW)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

The RECORD UNIVERSE is the foundational set of rows included in all
KPI calculations. It applies an 18-month backward-looking window from
the selected date, with a three-tier fallback for date assignment:

  Tier 1: planned_submission_date is NOT NULL → use it
  Tier 2: planned_submission_date IS NULL + actual_submission_date NOT NULL → use actual_submission
  Tier 3: planned_submission_date IS NULL + actual_submission IS NULL
          + actual_approval_date NOT NULL → use actual_approval

Each date is month-normalized (DATE(YEAR, MONTH, 1)) before the
DATEDIFF comparison against SelectedMonth.

### M-R-01 | Record (primary denominator measure)
─────────────────────────────────────────────────────────────
VAR SelectedDate  = DATEVALUE(MAX(Dim_Date[date1]))
VAR SelectedMonth = DATE(YEAR(SelectedDate), MONTH(SelectedDate), 1)
RETURN
COUNTROWS(
    FILTER(
        Query1,
        SWITCH(
            TRUE(),
            -- Tier 1: Planned date within 18 months
            NOT ISBLANK(Query1[planned_submission_date])
                && DATEDIFF(
                    DATE(YEAR(Query1[planned_submission_date]),
                         MONTH(Query1[planned_submission_date]), 1),
                    SelectedMonth, MONTH) <= 18,
            TRUE(),
            -- Tier 2: Planned NULL, Actual Sub within 18 months
            ISBLANK(Query1[planned_submission_date])
                && NOT ISBLANK(Query1[actual_submission_date])
                && DATEDIFF(
                    DATE(YEAR(Query1[actual_submission_date]),
                         MONTH(Query1[actual_submission_date]), 1),
                    SelectedMonth, MONTH) <= 18,
            TRUE(),
            -- Tier 3: Both NULL, Actual Approval within 18 months
            ISBLANK(Query1[planned_submission_date])
                && ISBLANK(Query1[actual_submission_date])
                && NOT ISBLANK(Query1[actual_approval_date])
                && DATEDIFF(
                    DATE(YEAR(Query1[actual_approval_date]),
                         MONTH(Query1[actual_approval_date]), 1),
                    SelectedMonth, MONTH) <= 18,
            TRUE(),
            FALSE()
        )
    )
)

Format: 0

⚠️ NOTE: 'Record_New' and 'Record_New version 2' are variant measures
using SELECTEDVALUE(Dim_Date[Date], DATE(2024,12,31)) instead of
DATEVALUE(MAX(Dim_Date[date1])). They also add a >= 0 lower bound
(0–18 months) rather than just <= 18.
  • Record       : backward-only (no lower bound), uses Date1 string
  • Record_New   : SELECTEDVALUE; <= 18 only
  • Record_New v2: SELECTEDVALUE; 0–18 window (stricter)
USE: 'Record' is the primary denominator for all KPI% measures.


### M-R-02 | Record_Final
─────────────────────────────────────────────────────────────
= CALCULATE(COUNTROWS(Query1), Query1[Is_In_Record] = 1)
Format: 0
Purpose: Uses the Is_In_Record calculated column (hardcoded to 2024-12-31)
         as a static pre-calculated record flag. For development/debug only.
⚠️ NOTE: Is_In_Record is hardcoded to DATE(2024,12,31). Not recommended
         for live use — use [Record] measure for dynamic date selection.


### M-R-03 | PlannedSubDatePresent
─────────────────────────────────────────────────────────────
SUMX(FILTER(Query1, [18-month gate]), IF(NOT ISBLANK([planned_submission_date]), 1, 0))
Format: 0
Purpose: Count of rows within the 18-month universe that have a planned
         submission date. Used as denominator for KPI5 (Donut5).

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — MISSING RECORDS KPI FLAG MEASURES (KPI 1–5)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All five "Missing Records" KPIs follow the same pattern:
  VAR _Flag   = [FlagMeasure]    -- count of rows with the specific null
  VAR _Record = [Record]         -- denominator (18-month universe)
  RETURN IF(ISBLANK(_Record) || _Record = 0, 0, 1 - DIVIDE(_Flag, _Record, 0))

Format: 0.00%;-0.00%;0.00%

Interpretation: Higher % = better data quality (fewer nulls)
  95%+ → Green (#00B050)
  61–94% → Yellow (#FFC000)
  <61% → Red (#FF0000)

### M-KPI1 | KPI1% — Missing Planned Submission Date
─────────────────────────────────────────────────────────────
Flag Measure: PlannedSubDateNull?
Logic: COUNTROWS of rows where:
  - planned_submission_date IS BLANK
  - AND at least one of: actual_submission or actual_approval
    is within 0–18 months of SelectedDate

DAX (PlannedSubDateNull?):
  VAR SelectedDate = DATEVALUE(MAX(Dim_Date[date1]))
  RETURN COUNTROWS(FILTER(Query1,
    ISBLANK(Query1[planned_submission_date])
    && (
        (NOT ISBLANK(Query1[actual_submission_date])
         && DATEDIFF(Query1[actual_submission_date], SelectedDate, MONTH) <= 18
         && DATEDIFF(Query1[actual_submission_date], SelectedDate, MONTH) >= 0)
        ||
        (NOT ISBLANK(Query1[actual_approval_date])
         && DATEDIFF(Query1[actual_approval_date], SelectedDate, MONTH) <= 18
         && DATEDIFF(Query1[actual_approval_date], SelectedDate, MONTH) >= 0)
    )
  ))

KPI1% = IF(ISBLANK([Record]) || [Record] = 0, 0,
            1 - DIVIDE([PlannedSubDateNull?], [Record], 0))


### M-KPI2 | KPI2% — Missing Actual Submission Date
─────────────────────────────────────────────────────────────
Flag Measure: ActualSubDateNull? (also aliased as ActualSubNull?)
Logic: COUNTROWS of rows where:
  - actual_submission_date IS BLANK
  - AND actual_approval_date is NOT BLANK
  - AND planned_submission_date (if present) is within 0–18 months

Two branches:
  Branch 1: planned NOT NULL + actual_approval NOT NULL → planned in range
  Branch 2: planned NULL + actual_approval NOT NULL → actual_approval in range

KPI2% = 1 - DIVIDE([ActualSubDateNull?], [Record], 0)


### M-KPI3 | KPI3% — Missing Application ID
─────────────────────────────────────────────────────────────
Flag Measure: ApplicationIDNull?
Logic: COUNTROWS of rows where:
  - application_id IS BLANK
  - AND (planned sub is in future 0–18 months OR base_case_approval < TODAY()
         OR, if planned NULL: actual_sub / actual_approval in range
         OR base_case_approval < TODAY())

⚠️ NOTE: ApplicationIDNull? uses MIXED DATEDIFF directions in places.
  Some branches compute DATEDIFF(SelectedDate, Query1[date], MONTH) (future)
  while others use DATEDIFF(Query1[date], SelectedDate, MONTH) (past).
  These are preserved exactly as translated from Tableau.

KPI3% = 1 - DIVIDE([ApplicationIDNull?], [Record], 0)


### M-KPI4 | KPI4% — Missing Regulatory Objective Name
─────────────────────────────────────────────────────────────
Flag Measure: RegObjNameNull?
Logic: COUNTROWS of rows where:
  - reg_objective_name IS BLANK
  - AND is_major_filing = 1
  - AND (planned sub + actual_approval combination in range
         OR base_case_approval < TODAY()
         OR, if planned NULL: actual sub / approval in range)

⚠️ NOTE: Also uses mixed DATEDIFF directions (exact Tableau translation).

KPI4% = 1 - DIVIDE([RegObjNameNull?], [Record], 0)


### M-KPI5 | KPI5% — Missing Actual Approval Date
─────────────────────────────────────────────────────────────
Flag Measure: ActualApprovalNull? / ActualAppDateNull?
Logic: COUNTROWS of rows where:
  - actual_approval_date IS BLANK
  - Branch 1: actual_submission NOT NULL + planned in 0–18 months past
  - Branch 2: planned NULL + actual_submission in 0–18 months past

KPI5% = 1 - DIVIDE([ActualAppDateNull?], [Record], 0)


### M-TM-01 | TotalMissing
─────────────────────────────────────────────────────────────
= COALESCE(
    [PlannedSubDateNull?] + [ActualSubNull?] + [ApplicationIDNull?]
    + [RegObjNameNull?]   + [ActualApprovalNull?],
    0
  )
Format: 0
Purpose: Sum of all 5 missing-record flags per row context.
         Used for ranking countries/TAs in the bar chart.


### M-TM-02 | Total%
─────────────────────────────────────────────────────────────
= COALESCE(
    DIVIDE([KPI1%]+[KPI2%]+[KPI3%]+[KPI4%]+[KPI5%]+[KPI6%]
           +[KPI7%]+[KPI8%]+[KPI9%]+[KPI10%], 10, 0),
    0
  )
Format: 0.00%;-0.00%;0.00%
Purpose: Overall data quality score = average of all 10 KPI%.
         Used for the summary scorecard card.

Color measure:
  Data_Quality_ColorStatus =
  IF([Total%] >= 0.95, "#69ac65",
      IF([Total%] >= 0.61, "yellow", "red"))


### M-TM-03 | DetailsTableFlag
─────────────────────────────────────────────────────────────
= IF(
    [ActualAppDateNull?] >= 1 || [ActualSubDateNull?] >= 1
    || [ApplicationIDNull?] >= 1 || [PlannedSubDateNull?] >= 1
    || [RegObjNameNull?] >= 1    || [NonAdhKPI1] >= 1
    || [NonAdhKPI2] >= 1         || [NonAdhKPI3] >= 1
    || [NonAdhKPI4] >= 1         || [NonAdhKPI5] >= 1,
    1, 0
  )
Format: 0
Purpose: Row-level flag for detail table visibility —
         shows row only if any KPI flag fires.

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — NON-ADHERENCE KPI MEASURES (KPI 6–10)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All Non-Adherence KPIs use VAR SelectedDate = MAX(Dim_Date[Date])
(not the Date1 string column). They count rows failing specific
business-rule adherence checks, subject to the same 18-month
date gate as the Missing Records KPIs.

KPI%  formula: 1 - DIVIDE([NonAdhKPIn], [Record], 0)

### M-NA-01 | NonAdhKPI1 → KPI6%
─────────────────────────────────────────────────────────────
Status condition:
  regulatory_objective_status = "Planned"
  AND activity_status IN {"Closed","Opt Out","Cancelled","Submission Ready"}

Business rule: A planned objective should NOT be closed/cancelled/etc.
Date gate: Standard 18-month window (Branch A: planned; Branch B: fallback)
Format: 0


### M-NA-02 | NonAdhKPI2 → KPI7%
─────────────────────────────────────────────────────────────
Status condition (at least ONE of A, B, C):
  A: activity_status = "Cancelled" AND actual_approval NOT NULL
  B: activity_status <> "Cancelled" AND base_case_approval IS NULL
  C: activity_status <> "Cancelled" AND best_case_approval IS NULL

⚠️ NOTE: Condition B and C are an OR — meaning any non-cancelled
  record with a missing base or best case approval date fires.
Date gate: Standard 18-month window
Format: 0


### M-NA-03 | NonAdhKPI3 → KPI8% (via Donut3)
─────────────────────────────────────────────────────────────
Status conditions (A OR B OR C OR D OR E):
  A: reg_objective_status IS NULL + actual_submission NOT NULL + "Opt Out"
  B: reg_objective_status = "Decision Not to File" + actual_sub NOT NULL + "Opt Out"
  C: reg_objective_status = "Decision Not to File" + actual_sub NOT NULL + "Closed"
  D: reg_objective_status IN {"HA Notified","Rejected"} + actual_sub IS NULL
  E: reg_objective_status = "HA Notified" + actual_approval NOT NULL

Business rule: Regulatory Objective Status must be consistent
  with actual date presence and activity status.
Date gate: Month-normalized 18-month window (uses DATE(YEAR,MONTH,1))
Format: 0; COALESCE → 0


### M-NA-04 | NonAdhKPI4 → KPI9%
─────────────────────────────────────────────────────────────
Status conditions (ALL THREE must be true):
  A: actual_submission_date NOT NULL
  B: actual_approval_date IS NULL
  C: activity_status <> "Submission Ready"

Business rule: If submission has occurred and no approval exists,
  activity should not be "Submission Ready" status.
Date gate: Standard 18-month window
Format: 0; COALESCE → 0


### M-NA-05 | NonAdhKPI5 → KPI10%
─────────────────────────────────────────────────────────────
Status condition:
  event_status = "Cancelled"
  AND (activity_status <> "Cancelled" OR activity_status <> "Closed"
       OR activity_status <> "Opt Out")

⚠️ NOTE: The OR block (A <> x || A <> y || A <> z) is ALWAYS TRUE
  for any value of activity_status. This effectively means the only
  real condition is event_status = "Cancelled". Preserved exactly
  as translated from Tableau.
Date gate: Standard 18-month window
Format: 0; COALESCE → 0

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — DONUT CHART MEASURES (KPI3–KPI6 detail view)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Donut measures are non-adherence variants using month-normalized
date gates (DATE(YEAR,MONTH,1)) instead of raw DATEDIFF on date fields.
They map to KPI8–KPI10 and a combined Donut6.

### M-D-01 | Donut3 → AdherentRecordsKPI3
  Conditions A+B+C from NonAdhKPI3 (Opt Out / Decision Not to File only)
  Date: Month-normalized. Used as: AdherentRecordsKPI3 = [Record] - [Donut3]

### M-D-02 | Donut4 → AdherentRecordsKPI4
  Conditions: reg_obj_status IN {"HA Notified","Rejected"} + actual_sub NULL
  Date: Month-normalized. Used as: AdherentRecordsKPI4 = [Record] - [Donut4]

### M-D-03 | Donut5 → AdherentRecordsKPI5
  Conditions: planned NOT NULL + actual_submission IS NULL
  Date: Month-normalized, planned gate only (fallback branch structurally
        present for Tableau parity but never fires since planned NOT NULL)
  Denominator: [PlannedSubDatePresent] (not [Record])
  Used as: AdherentRecordsKPI5 = [PlannedSubDatePresent] - [Donut5]

### M-D-04 | NonAdhKPI_Donut6 → AdherentRecordsKPI6
  Combined flag — 4 branches covering:
    Branch 1: ApplicationIDNull + future planned/base_case past TODAY
    Branch 2: ApplicationIDNull + planned NULL + sub/approval window + base_case past
    Branch 3: RegObjName NULL + is_major = 1 + planned window + approval check
    Branch 4: RegObjName NULL + planned NULL + is_major = 1 + sub/approval window
  Scope gate: Full 18-month universe (SWITCH pattern, month-normalized)
  Date: Month-normalized throughout
  Used as: AdherentRecordsKPI6 = [Record] - [NonAdhKPI_Donut6]

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — 12-MONTH TREND MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

These SUM_KPI*_12 measures power the monthly trend line chart.
They use a 12-month (not 18-month) DATEDIFF window.
VAR SelectedDate = MAX(dim_date[date])

### M-T-01 | SUM_KPI1_12 — Planned Sub NULL, 12-month window
  Planned IS NULL + (actual_sub or actual_approval within 12 months)

### M-T-02 | SUM_KPI2_12 — Actual Sub NULL, 12-month window
  Actual_sub IS NULL + actual_approval NOT NULL + planned/approval within 12 months

### M-T-03 | SUM_KPI3_12 — Application ID NULL, 12-month window
  Application_id IS NULL + (planned within 12m OR fallback to actual_sub/approval)

### M-T-04 | SUM_KPI4_12 — Reg Obj Name NULL, 12-month window
  reg_objective_name IS NULL + is_major = 1 + date in range

### M-T-05 | SUM_KPI5_12 — Actual Approval NULL, 12-month window
  actual_approval IS NULL + actual_sub NOT NULL + planned/actual within 12 months

### M-T-06 | Total KPI (12-month sum)
  = [SUM_KPI1_12] + [SUM_KPI2_12] + [SUM_KPI3_12]
    + [SUM_KPI4_12] + [SUM_KPI5_12]
Format: 0

### M-T-07 | Monthly Trend Title (dynamic)
  = "Monthly Trend of Missing Records Over Past 12 Months from "
    & FORMAT(SELECTEDVALUE('Dim_Date'[date1], DATE(2024,12,31)), "MMM YYYY")

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — RANKING AND BAR CHART MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### M-RK-01 | Country_RankNo
  Dense manual ranking (COUNTROWS + 1 pattern):
  Counts other valid countries with higher TotalMissing, or same TM
  but alphabetically earlier. +1 = current rank.
  Blank/empty country names → BLANK() (never ranked).
  Zero-value countries ARE ranked (previous bug: they were blanked).

### M-RK-02 | TherapArea_RankNo
  Same dense ranking pattern applied to disease_area_standardized.
  Blank/empty TA names ARE ranked (blank TA with TM=227 must be Rank 1).
  No exclusion of blank TA unlike Country ranking.

### M-RK-03 | Country_Rank (RANKX variant)
  = RANKX(ALLSELECTED(Query1[filing_country]), CALCULATE([TotalMissing]),, DESC, Skip)
  Skip mode = tied items share a rank with a gap (matches Tableau RANK_UNIQUE desc).

### M-RK-04 | Rank_Country (dense RANKX variant)
  = IF(NOT ISBLANK([TotalMissing]),
       RANKX(ALLSELECTED(Query1[filing_country]), [TotalMissing],, DESC, Dense),
       BLANK())
  Dense = no gaps. Used in some visuals.

### M-RK-05 | Dynamic_Rank
  Switches ranking dimension based on Param_Country_TherapArea:
  SelectedField = 0 → ranks by filing_country
  SelectedField = 1 → ranks by disease_area_standardized

### Bar Chart Measures (one per KPI, shared pattern):
─────────────────────────────────────────────────────────────
Each *_Bar measure:
  1. Gets SelectedField (0=Country, 1=TA) and Limit (TopN Value)
  2. Computes the appropriate RankNo (Country_RankNo or TherapArea_RankNo)
  3. For Country: adds CountryGuard (non-blank, non-empty name)
  4. Returns COALESCE(flag_measure, 0) + 0.0001 if InLimit, else BLANK()
     ⚠️ The +0.0001 offset ensures zero-value bars still render in Power BI

Measures:
  PlannedSubMissing_Bar  → [PlannedSubDateNull?]
  ActualSubMissing_Bar   → [ActualSubDateNull?]
  ApplIDMissing_Bar      → [ApplicationIDNull?]
  RegObjMissing_Bar      → [RegObjNameNull?]
  ActualApprMissing_Bar  → [ActualAppDateNull?]
  TotalMissing_Bar       → [TotalMissing] (rank 1 only; used for reference line)
  'TotalMissing_Bar SORT NEW' → [TotalMissing] (all TopN; for sort axis)

### M-RK-06 | TotalMissing_Sort
  = IF(NOT ISBLANK([TotalMissing]), [TotalMissing], 0)
  Format: 0; used as sort field on bar chart axis.

### M-RK-07 | TotalMissing_Label / TotalMissing_TA_Only
  = IF([Param_Country_TherapArea Fields] = 1, [TotalMissing], BLANK())
  Shows TotalMissing only when TA view is active.

### M-RK-08 | TA_Count_With_Missing
  COUNTROWS of distinct TAs (non-blank) with TotalMissing > 0.
  Uses ALLSELECTED to respect active slicer context.

### M-RK-09 | total_bar (diagnostic sum)
  = [ActualApprMissing_Bar] + [ActualSubMissing_Bar] + [ApplIDMissing_Bar]
    + [PlannedSubMissing_Bar] + [RegObjMissing_Bar]
    + [RegObjMissing_Bar]   ← ⚠️ DUPLICATE (RegObj counted twice)
    + [TotalMissing_Bar]
  Format: 0
  ⚠️ NOTE: RegObjMissing_Bar appears twice — likely a copy-paste error.
  Correct before production.

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 11 — KPI COLOR MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All KPI color measures follow:
  SWITCH(TRUE(), _Val >= 0.95, "#00B050", _Val >= 0.61, "#FFC000", "#FF0000")

  KPI1Color   : uses [KPI1%]; also checks ISBLANK → "#FF0000"
  KPI2Color   : [KPI2%]
  KPI3Color   : [KPI3%]
  KPI4Color   : [KPI4%]
  KPI5Color   : [KPI5%]
  KPI6Color   : [KPI6%]
  KPI7Color   : [KPI7%]
  KPI8Color   : [KPI8%]
  KPI9Color   : [KPI9%]
  KPI10Color  : [KPI10%]
  Total%Color : [Total%]

Color thresholds:
  ≥ 95% → Green   (#00B050)
  61–94% → Yellow (#FFC000)
  < 61% → Red     (#FF0000)

⚠️ NOTE: KPI1Color explicitly handles ISBLANK(_Val) → Red.
  Other color measures do not — add ISBLANK guard to all if
  no-data cells should display red rather than transparent.

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 12 — DYNAMIC TITLE AND LABEL MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### 'Dynamic Title'
  "Top " & SELECTEDVALUE('TopN'[TopN], 3)
  & " Defaulters by "
  & IF(SELECTEDVALUE('Param_Country_TherapArea'[Parameter Order],0) = 0,
        "Country", "Therapeutic Area")

  ⚠️ NOTE: Uses default TopN = 3 here (not 10 as in TopN Value).
  Align with [TopN Value] default if chart title should match.

### 'Dynamic Tooltip'
  IF(SelectedOrder = 0, "Country", "Therapeutic Area")
  & "-wise missing records for each KPI"

### 'Data Quality Title'
  "Data Quality Percentage for all "
  & SELECTEDVALUE('BarGraph2'[ Display As], "Missing Records")   ← leading space
  & " KPIs"

### 'Monthly Trend Title'
  "Monthly Trend of Missing Records Over Past 12 Months from "
  & FORMAT(SELECTEDVALUE('Dim_Date'[date1], DATE(2024,12,31)), "MMM YYYY")

### 'Planned Sub Month Name' (measure)
  = FORMAT(MAX(Query1[planned_submission_date]), "MMMM")

### 'Selected Value' / 'Selected Dimension Label'
  Use NAMEOF() to map Param_Country_TherapArea[Parameter Fields]
  to actual column values and display labels dynamically.

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 13 — CALCULATED COLUMNS ON Query1
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### CC-Q1-01 | Is_In_Record
─────────────────────────────────────────────────────────────
⚠️ HARDCODED to DATE(2024, 12, 31) — static column.
Same 3-tier logic as [Record] measure but evaluated at row level
with a fixed date. Returns 1 (in universe) or 0 (out).
Used in [Record_Final] measure.
Recommend: use [Record] measure for dynamic filtering instead.

### CC-Q1-02 | generic_name (groups)
─────────────────────────────────────────────────────────────
A Power BI "group" column created from generic_name.
Maps ~24 specific drug names to individual display values,
15 additional drugs to "Others", and all remaining to pass-through.

BMS drugs explicitly named:
  Abatacept, Adagrasib, Admilparant, Arlocabtagene Autoleucel,
  Bms-986012-01/Nivolumab, BMS-986504-05, Cendakimab,
  Deucravacitinib, Ezobresib, Golcadomide,
  Golcadomide Hydrochloride, Iberdomide Hydrochloride,
  Ipilimumab, Izalontamab Brengitecan, Mezigdomide, Milvexian,
  Nivolumab, Nivolumab Sc, Nivolumab/Relatlimab, Ozanimod,
  Repotrectinib, Trotabresib, Xanomeline/Trospium Chloride

"Others" group:
  Afimetoran, Apixaban, Azacitidine, Belatacept, BMS-986325-01,
  Dasatinib, Elotuzumab, Farletuzumab-Ecteribulin, Fedratinib,
  Idecabtagene Vicleucel, Lenalidomide, Lisocabtagene Maraleucel,
  Luspatercept, Mavacamten, Pomalidomide

sortByColumn: (none specified — sorts alphabetically by default)
summarizeBy : none

### CC-Q1-03 | Error Flag
─────────────────────────────────────────────────────────────
Row-level version of NonAdherenceToBusinessRule (measure).
Evaluated at row context — returns the first-priority error string
or BLANK() if no issue. Same 10-priority SWITCH logic.
⚠️ NOTE: The commented-out version (using COALESCE(...) = 1)
  is preserved in the DAX source as a dev artifact. The active
  version uses measure references (>= 1 threshold).

### CC-Q1-04 | Planned Sub Month Name col
  = IF(NOT ISBLANK([planned_submission_date]), FORMAT([planned_submission_date], "MMMM"), BLANK())
  sortByColumn: Planned Sub Month Number

### CC-Q1-05 | Planned Sub Month Number
  = IF(NOT ISBLANK([planned_submission_date]), MONTH([planned_submission_date]), BLANK())
  Format: 0; used to sort [Planned Sub Month Name col]

### CC-Q1-06 | Planned Sub Year
  = IF(NOT ISBLANK([planned_submission_date]), YEAR([planned_submission_date]), BLANK())
  Format: 0

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 14 — NON-ADHERENCE LABEL MEASURE
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### NonAdherenceToBusinessRule (measure)
─────────────────────────────────────────────────────────────
Priority SWITCH returning the first-firing issue label:

  Priority | Condition            | Label
  ─────────────────────────────────────────────────────────────────────────────
  1        | PlannedSubDateNull?  | "Missing Planned Submission Date"
  2        | ActualSubDateNull?   | "Missing Actual Submission Date"
  3        | ApplicationIDNull?   | "Missing Application ID"
  4        | RegObjNameNull?      | "Missing Regulatory Objective Name"
  5        | ActualAppDateNull?   | "Missing Actual Approval Date"
  6        | NonAdhKPI1           | "Invalid Activity Status with Planned Reg. Obj. Status"
  7        | NonAdhKPI2           | "Inconsistent Activity Status"
  8        | NonAdhKPI3           | "Invalid Reg. Obj. Status as per Actual Dates"
  9        | NonAdhKPI4           | "Invalid Activity Status having Actual Submission Date Present and Approval Date blank"
  10       | NonAdhKPI5           | "Invalid Activity Status for Cancelled GSP Events"
  default  | (none)               | BLANK()

Used in the detail table to show the highest-priority issue per row.
The Error Flag calculated column mirrors this logic at row level.

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 15 — DIAGNOSTIC / DEBUG MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

These measures are present in the PBIP model but are for
development/validation only. Remove before production:

  Filtered_Rows          = CALCULATE(COUNTROWS(Query1), [data_gap_cat] = "Major Filings")
  Diag_NonBlankAsset     = COUNTROWS(FILTER(Query1, NOT ISBLANK([generic_name])))
  Diag_MajorFilings_Asset= COUNTROWS(FILTER(..., NOT ISBLANK([generic_name]) && [data_gap_cat] = "Major Filings"))
  Diag_DateOnly          = COUNTROWS(FILTER(..., planned NOT NULL && DATEDIFF <= 18))
  Diag_BlankAsset        = COUNTROWS(FILTER(..., ISBLANK([generic_name]) || [generic_name] = ""))
  Diag_BlankAssetTotal   = CALCULATE(COUNTROWS, ALL([generic_name]), ISBLANK([generic_name]))
  Diag_TotalIgnoreSlicers= CALCULATE(COUNTROWS, ALL(Query1))
  Diag_BoundaryRows      = COUNTROWS of rows exactly at 18-month boundary
  Boundary_Flag          = 1 if row is exactly at 18-month boundary
  Record_New             = [Record] variant using SELECTEDVALUE
  Record_New version 2   = [Record] variant with 0–18 floor
  Measure / Measure 2–10 = Stub/empty placeholder measures

  Total Dates Count      = Sum of all 5 null-date flag counts
                           (diagnostic cross-check against TotalMissing)
  AdherentRecordsKpi1–6  = [Record] - [flag_measure] variants for donut charts

  'Test_KPI1'            = [KPI1%] (alias/passthrough)

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 16 — SLICERS AND INTERACTIVITY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Tableau Control                    | PBI Implementation                                      |
|────────────────────────────────────|─────────────────────────────────────────────────────────|
| Date parameter (as-of date)        | Slicer on Dim_Date[date1] (string display, DATEVALUE)   |
| Date month (trend chart)           | Slicer on DateParameter[Date]                           |
| TopN parameter                     | Slicer on TopN[TopN] (single-select)                    |
| BarGraph (Country/TA toggle)       | Slicer on Param_Country_TherapArea[Parameter]           |
| BarGraph2 (Missing/Non-Adh toggle) | Slicer on BarGraph2[ Display As] (leading space!)       |
| filing_country filter              | Slicer on Query1[filing_country]                        |
| disease_area filter                | Slicer on Query1[disease_area_standardized]             |
| generic_name filter                | Slicer on Query1[generic_name (groups)]                 |
| data_gap_cat filter                | Slicer on Query1[data_gap_cat]                          |

⚠️ NOTE: Dim_Date is a disconnected slicer — it has NO relationship
  to Query1. All measures read the slicer value via MAX(Dim_Date[Date])
  or SELECTEDVALUE(Dim_Date[Date]). Do NOT create a relationship
  between Dim_Date and Query1.

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 17 — KNOWN ISSUES, GAPS, AND RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

G1 | UAT endpoint in Power Query M (CRITICAL)
   Current: vnavigator.456939526755.us-east-1...vnavigator_uat
   Required: vnavigator.311406940810.us-east-1...vnavigator_prod
   Must update before production deployment.

G2 | total_bar has RegObjMissing_Bar doubled
   [RegObjMissing_Bar] appears twice in the total_bar measure formula.
   Likely a copy-paste error. Correct to a single reference before production.

G3 | Is_In_Record calculated column is hardcoded to 2024-12-31
   This static column should not be used for dynamic KPI calculations.
   Use the [Record] measure instead. The column is retained for dev parity.

G4 | NonAdhKPI5 OR block is always TRUE
   The condition (activity_status <> "Cancelled" || <> "Closed" || <> "Opt Out")
   is logically equivalent to TRUE for any value. Preserved exactly from
   Tableau. Review with business owner whether the intent was AND or ELSE.

G5 | Mixed DATEDIFF directions in ApplicationIDNull? and RegObjNameNull?
   Some branches swap START/END in DATEDIFF. These are preserved as translated
   from Tableau. Validate against expected row counts before production.

G6 | BarGraph2[' Display As'] leading space
   The column name has a leading space: ' Display As'.
   All DAX references must include the space: BarGraph2[ Display As]
   Flag for renaming before production.

G7 | KPI color measures lack ISBLANK guard (except KPI1Color)
   Only KPI1Color handles ISBLANK → Red. Add to all KPIxColor measures
   for consistent no-data behavior.

G8 | Stub measures (Measure, Measure 2–10) should be removed
   These are empty placeholders with annotation True flags.
   Remove before production to keep the model clean.

G9 | 'Dynamic Title' uses TopN default = 3, not 10
   SELECTEDVALUE('TopN'[TopN], 3) — the fallback default is 3.
   [TopN Value] uses default 10. Align if title should match chart.

G10 | Error Flag calculated column uses measure references
   Calculated columns calling measures can cause circular dependency
   risks and performance issues. Monitor in large datasets.

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 18 — QUICK REFERENCE: CONVERSION PATTERNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Tableau Pattern                               | DAX / Power Query Equivalent                            |
|───────────────────────────────────────────────|─────────────────────────────────────────────────────────|
| Parameter (date)                              | Disconnected Dim_Date slicer; MAX(Dim_Date[Date])       |
| Parameter (TopN integer)                      | GENERATESERIES table; SELECTEDVALUE with default        |
| Parameter (dimension toggle)                  | Field Parameter table; SELECTEDVALUE on Parameter Order |
| Parameter (view toggle)                       | Inline DATATABLE; SELECTEDVALUE + SWITCH                |
| ISNULL([col])                                 | ISBLANK([col])                                          |
| ZN([measure])                                 | COALESCE([measure], 0)                                  |
| IF condition THEN 1 ELSE 0 → COUNT            | COUNTROWS(FILTER(table, condition))                     |
| RANK_UNIQUE([field], 'desc')                  | RANKX(ALLSELECTED(...), ...,, DESC, Skip)               |
| Dense rank                                    | COUNTROWS(FILTER(..., other > current)) + 1             |
| DATEDIFF in months (Tableau)                  | DATEDIFF(start, end, MONTH) in DAX                     |
| DATE_TRUNC('month', date)                     | DATE(YEAR(date), MONTH(date), 1)                       |
| NULL check on date with range filter          | NOT ISBLANK(date) && DATEDIFF(..., MONTH) <= 18        |
| Tableau "group" on field                      | SWITCH(TRUE, col IN {"val1"}, "Group1", ...) column    |
| Calculated field (row-level)                  | DAX calculated column on table                         |
| Measure                                       | DAX measure                                            |
| Parameter → SWITCH visual logic               | SWITCH(SELECTEDVALUE(ParamTable[Col]), ...)            |
| Dynamic chart title                           | DAX measure returning string for card/title visual     |
| Color coding threshold                        | SWITCH(TRUE(), val >= 0.95, "#00B050", ...) measure    |

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 19 — COMPLETENESS SCORECARD
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Component                              Status       Coverage
─────────────────────────────────────  ───────────  ────────────────────
Tableau Parameters → PBI tables        ✅ FULL       5 / 5 (100%)
Data Source → Power Query M            ✅ FULL       1 / 1 (100%)
Source Column Mapping (Query1)         ✅ FULL       37 / 37 (100%)
Power Query transformations            ✅ FULL       Year + Month extract
Date Dimension (Dim_Date)              ✅ FULL       13 columns, 2 measures
TopN Table                             ✅ FULL       1 measure
Field Parameter (Country/TA)           ✅ FULL       3 columns, 1 measure
BarGraph2 Toggle                       ✅ FULL       2 columns, 4 measures
Record Universe Measures               ✅ FULL       3 variants documented
Missing Records KPIs (KPI 1–5)         ✅ FULL       5 flag + 5 % measures
Non-Adherence KPIs (KPI 6–10)          ✅ FULL       5 measures
Donut Chart Measures                   ✅ FULL       4 measures
12-Month Trend Measures                ✅ FULL       5 KPI + 1 total + 1 title
Ranking Measures                       ✅ FULL       6 measures
Bar Chart *_Bar Measures               ✅ FULL       7 measures
KPI Color Measures                     ✅ FULL       11 measures
Dynamic Titles / Labels                ✅ FULL       5 measures
Calculated Columns on Query1           ✅ FULL       6 columns
Non-Adherence Label Measure            ✅ FULL       10-priority SWITCH
Diagnostic / Debug Measures            ✅ NOTED      14 items flagged for removal
Known Issues Documented                ✅ FULL       10 gaps documented
Overall Completeness                   ✅ ~97%       (pending G1, G2, G4 fixes)
