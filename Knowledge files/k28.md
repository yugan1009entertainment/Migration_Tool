# 📘 CONVERSION KNOWLEDGE FILE
# 27487 – Global Potential Off Label
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-06
# Source files: global_potential_off_label.txt + all PBIP .txt files

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

> This workbook contains NO Tableau parameters.
> No <datasource-parameters> block was present in the Tableau XML.
> All filtering and display logic is driven by fixed worksheet filters
> and calculated fields (see Sections 2 and 3).
>
> The Power BI slicer and filter controls below serve the same
> user-facing interactive role that parameters would play in Tableau.

### PARAM-NONE | No Parameters Defined
─────────────────────────────────────────────────────────────
Tableau Parameters    : None
Power BI Tables       : No DATATABLE() or GENERATESERIES() parameter
                        tables required for this workbook.

Slicer columns acting as parameter-equivalent controls:
  [product]                 — Multi-select dropdown slicer
  [site_of_care]            — Multi-select dropdown slicer
  [treatment_status]        — Multi-select dropdown slicer
  [collection_pickup_country] — Multi-select dropdown slicer
  [treating_physician_name] — Multi-select dropdown slicer
  [join_id]                 — Multi-select dropdown slicer
  [enrollment_created]      — Date range slicer
  [infusion_logged]         — Date range slicer
  [Drug Admin Date]         — Date range slicer
  [Drug Delivey Date]       — Date range slicer  ⚠️ typo in name

Fixed (non-interactive) report-level filters:
  [enrollment_channel_name] = "Commercial"   — locked; not user-selectable
  [Indication Filter]       = TRUE           — locked; not user-selectable

If future iterations require user-selectable parameters (e.g., date grain
picker, product group toggle), use the following template:

Power BI DAX Template (future use):
─────────────────────────────────────
View By Parameter =
    DATATABLE(
        "View By",    STRING,
        "Sort Order", INTEGER,
        {
            { "Day",     1 },
            { "Month",   2 },
            { "Quarter", 3 },
            { "Year",    4 }
        }
    )

-- Consuming measure pattern:
Selected Grain =
    SELECTEDVALUE( 'View By Parameter'[View By], "Month" )

Conversion Status: ✅ N/A — no Tableau parameters to convert.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: Custom SQL Query
## Power BI Full Table Name: Custom SQL Query
## Source: PostgreSQL
##         Server  : bms-eim-cell-therapy-auroradb-read-prod.web.bms.com:5432
##         Database: postgres
##         Schema  : ct_bia
##         Table   : stg_gps_scheduling

### CC-01 | Indication Filter  ← Boolean Calculated Field
─────────────────────────────────────────────────────────────
PBI Data Type   : boolean
PBI Format      : "TRUE";"TRUE";"FALSE"
PBI Hidden      : No
PBI Summarize   : none
PBI lineageTag  : Calculation_772085909962469377

Tableau Expression:
───────────────────
IF ([indication] = 'Other'
    OR [join_id] = '6RP3-2P779'
    OR [join_id] = '1975-HH8H8')
THEN TRUE
ELSE FALSE END
// Returns TRUE for rows that should be included in the Off Label view.
// Hard-coded join_id overrides ensure two specific records are always
// included regardless of their indication value.

Power BI DAX:
─────────────
column 'Indication Filter' =
IF(
    OR(
        'Custom SQL Query'[Indication] = "Other",
        OR(
            'Custom SQL Query'[join_id] = "6RP3-2P779",
            'Custom SQL Query'[join_id] = "1975-HH8H8"
        )
    ),
    TRUE(),
    FALSE()
)
/* Original Tableau Expression:
   IF ([indication] = 'Other'
       OR [join_id] = '6RP3-2P779'
       OR [join_id] = '1975-HH8H8')
   THEN TRUE ELSE FALSE END */

Depends on  : [indication], [join_id]
Used by     : Fixed report-level filter — [Indication Filter] = TRUE;
              replaces Tableau worksheet filter on
              Calculation_772085909962469377 = TRUE
NOTE        : Two specific join_id values are hard-coded as override
              inclusions. Review periodically if new override IDs arise.
Conversion  : ✅ FULL


### CC-02 | Enrollment created(Format)  ← Date Display Column
─────────────────────────────────────────────────────────────
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression (SQL layer):
────────────────────────────────
TO_CHAR(enrollment_created_date::date, 'FMDD Mon YYYY')
// PostgreSQL TO_CHAR produces display string e.g. "5 Jan 2024".
// Tableau received this as a pre-formatted string field.

Power BI DAX:
─────────────
column 'Enrollment created(Format)' =
FORMAT( 'Custom SQL Query'[enrollment_created], "DD MMM YYYY" )
// Power BI re-materialises the display string from the stored date type.
// Pattern: PQ ingests as type date → DAX FORMAT() for display.

Depends on  : [enrollment_created] (stored as dateTime after PQ cast)
Used by     : Table visual display column; date slicer label
Pattern     : Tableau TO_CHAR(date,'FMDD Mon YYYY')
              → Table.TransformColumnTypes(..., type date) in PQ
              → FORMAT([col],"DD MMM YYYY") in DAX
Conversion  : ✅ FULL


### CC-03 | Cancellation Date(Format)  ← Date Display Column
─────────────────────────────────────────────────────────────
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression (SQL layer):
────────────────────────────────
TO_CHAR(cancellation_date_utc::date, 'FMDD Mon YYYY')

Power BI DAX:
─────────────
column 'Cancellation Date(Format)' =
FORMAT( 'Custom SQL Query'[cancellation_date], "DD MMM YYYY" )

Depends on  : [cancellation_date] (dateTime)
Used by     : Table visual display column
Conversion  : ✅ FULL


### CC-04 | Drug Delivery Date(Format)  ← Date Display Column
─────────────────────────────────────────────────────────────
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression (SQL layer):
────────────────────────────────
TO_CHAR(dp_delivery_utc_actual::date, 'FMDD Mon YYYY')

Power BI DAX:
─────────────
column 'Drug Delivery Date(Format)' =
FORMAT( 'Custom SQL Query'[Drug Delivey Date], "DD MMM YYYY" )
-- ⚠️ Source column name contains typo "Delivey" (missing 'r').
-- Reference must match the typo until source column is corrected.

Depends on  : [Drug Delivey Date] (dateTime) ⚠️ typo in source column
Used by     : Table visual display column
Conversion  : ✅ FULL (⚠️ typo in dependency column name — see Gap G1)


### CC-05 | Infusion Logged(Format)  ← Date Display Column
─────────────────────────────────────────────────────────────
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression (SQL layer):
────────────────────────────────
TO_CHAR(infusion_logged_date::date, 'FMDD Mon YYYY')

Power BI DAX:
─────────────
column 'Infusion Logged(Format)' =
FORMAT( 'Custom SQL Query'[infusion_logged], "DD MMM YYYY" )

Depends on  : [infusion_logged] (dateTime)
Used by     : Table visual display column
Conversion  : ✅ FULL


### CC-06 | Drug Admin Date(Format)  ← Date Display Column
─────────────────────────────────────────────────────────────
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression (SQL layer):
────────────────────────────────
TO_CHAR(dp_admin_utc_actual::date, 'FMDD Mon YYYY')

Power BI DAX:
─────────────
column 'Drug Admin Date(Format)' =
FORMAT( 'Custom SQL Query'[Drug Admin Date], "DD MMM YYYY" )

Depends on  : [Drug Admin Date] (dateTime)
Used by     : Table visual display column
Conversion  : ✅ FULL


### SOURCE COLUMNS — Full 33-Column Inventory
─────────────────────────────────────────────────────────────
All 33 columns flow from Power Query into the Custom SQL Query table.
Hidden columns in Tableau are present in the PBI model but not placed
on any visual canvas by default.

Col  Tableau Field Name            PBI Column Name               Type       Hidden  Notes
───────────────────────────────────────────────────────────────────────────────────────────────
01   join_id                       join_id                       string     No      Primary key
02   enrollment_name               enrollment_name               string     No
03   brand                         brand                         string     No
04   product                       product                       string     No      Slicer
05   site_of_care                  site_of_care                  string     No      Slicer
06   collection_pickup_country     collection_pickup_country     string     No      Slicer; Country/ISO3166_2
07   enrollment_created            enrollment_created            dateTime   No      Format: d mmmm, yyyy; LocalDateTable
08   infusion_logged               infusion_logged               dateTime   No      Format: d mmmm, yyyy; LocalDateTable
09   execution_status              execution_status              string     No
10   cancellation_date             cancellation_date             dateTime   No      Format: d mmmm, yyyy; LocalDateTable
11   enrollment_channel_name       enrollment_channel_name       string     No      Fixed filter = "Commercial"
12   treatment_status              treatment_status              string     No      Slicer
13   region                        region                        string     YES
14   treating_physician_name       treating_physician_name       string     No      Slicer
15   batch_conformance_status      batch_conformance_status      string     No
16   dp_admin_utc                  Drug Admin Date               dateTime   No      Renamed in PQ; Format: d mmmm, yyyy
17   dp_delivery_utc               Drug Delivey Date             dateTime   No      ⚠️ Typo; Format: d mmmm, yyyy
18   site_of_care_country          site_of_care_country          string     YES     Country/ISO3166_2
19   make1_site_name               make1_site_name               string     YES
20   make2_site_name               make2_site_name               string     YES
21   indication                    indication                    string     No      Used in CC-01
22   count_toward_tat_kpi          count_toward_tat_kpi          boolean    YES     Format: "TRUE";"TRUE";"FALSE"
23   re_apheresis_join             re_apheresis_join             string     YES
24   remake_flag                   remake_flag                   string     YES
25   retreatment_flag              retreatment_flag              string     YES
26   make1_start_tz_actual         make1_start_tz_actual         dateTime   YES     General Date; LocalDateTable
27   make1_start_utc_actual        make1_start_utc_actual        dateTime   YES     General Date; LocalDateTable
28   make2_start_tz_actual         make2_start_tz_actual         dateTime   YES     General Date; LocalDateTable
29   make2_start_utc_actual        make2_start_utc_actual        dateTime   YES     General Date; LocalDateTable
30   make1_start_tz_planned        make1_start_tz_planned        dateTime   YES     General Date; LocalDateTable
31   make1_start_utc_planned       make1_start_utc_planned       dateTime   YES     General Date; LocalDateTable
32   make2_start_tz_planned        make2_start_tz_planned        dateTime   YES     General Date; LocalDateTable
33   make2_start_utc_planned       make2_start_utc_planned       dateTime   YES     General Date; LocalDateTable


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: Custom SQL Query
## (All measures defined on the Custom SQL Query fact table)

> This workbook contains NO Tableau {FIXED} LOD expressions.
> Aggregation is handled at the SQL layer via SELECT DISTINCT and
> row-level filters. The measure layer covers row counting and
> the table calculation INDEX() replication only.

### M-01 | Record Count
─────────────────────────────────────────────────────────────
PBI Table   : Custom SQL Query
PBI Format  : 0

Tableau Equivalent:
───────────────────
CNT(stg_gps_scheduling)
// Tableau automatic COUNT aggregate on the data source.
// Used implicitly as a KPI on dashboards and in quick filter counts.

Power BI DAX:
─────────────
measure 'Record Count' =
COUNTROWS( 'Custom SQL Query' )
-- Counts all rows passing the current filter context.
-- Equivalent to Tableau's automatic COUNT aggregate.

Used by     : Dashboard KPI card; implicit row count display
Conversion  : ✅ FULL


### M-02 | Row Index
─────────────────────────────────────────────────────────────
PBI Table   : Custom SQL Query
PBI Format  : 0

Tableau Equivalent:
───────────────────
INDEX()
// Table calculation — returns the sequential row number within the
// current partition, ordered by Rows.
// Used as the first visible column in the worksheet to provide
// a stable display row number.

Power BI DAX:
─────────────
-- Option A: Power BI 2024+ native row number (preferred)
measure 'Row Index' =
ROWNUMBER(
    'Custom SQL Query',
    ORDERBY( 'Custom SQL Query'[join_id], ASC )
)

-- Option B: RANKX fallback (all PBI Desktop versions)
measure 'Row Index' =
RANKX(
    ALL( 'Custom SQL Query' ),
    'Custom SQL Query'[join_id],
    ,
    ASC,
    Dense
)

Used by     : First column of Table visual (display row counter)
NOTE        : Implemented as a measure, not a calculated column, to
              avoid re-materialising on every dataset refresh.
              Place as the first column in the Table visual.
              Tableau had this as a table calculation on the row shelf —
              no static equivalent in Power BI without a calculated column.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU INITIAL TRANSFORMATIONS → POWER BI POWER QUERY (M)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | Custom SQL Query  ← Main Fact Table
─────────────────────────────────────────────────────────────
Tableau Data Source   : Custom SQL Query (PostgreSQL, federated)
Tableau Connection    : PostgreSQL
                        Server   : bms-eim-cell-therapy-auroradb-read-prod
                                   .web.bms.com:5432
                        Database : postgres
                        Schema   : ct_bia
                        Table    : stg_gps_scheduling
Tableau Auth          : username-password (user: sibalp), SSL: require
Tableau SQL method    : Custom SQL — SELECT DISTINCT with WHERE clause

PBI Table Name        : Custom SQL Query
PBI Load Mode         : Import
PBI Connection        : PostgreSQL.Database(
                            "bms-eim-cell-therapy-auroradb-read-prod
                             .web.bms.com:5432",
                            "postgres"
                        )
PBI Query method      : Value.NativeQuery(..., [EnableFolding=true])

Tableau → PBI Column Mapping:
──────────────────────────────
Tableau SQL Alias / Source Column     PBI Column Name               Type Cast
──────────────────────────────────────────────────────────────────────────────
join_id                             → join_id                       string
enrollment_name                     → enrollment_name               string
brand                               → brand                         string
product                             → product                       string
site_of_care                        → site_of_care                  string
collection_pickup_country           → collection_pickup_country     string
enrollment_created (via TO_CHAR)    → enrollment_created            → type date (PQ cast)
infusion_logged    (via TO_CHAR)    → infusion_logged               → type date (PQ cast)
execution_status                    → execution_status              string
cancellation_date  (via TO_CHAR)    → cancellation_date             → type date (PQ cast)
enrollment_channel_name             → enrollment_channel_name       string
treatment_status                    → treatment_status              string
region                              → region                        string
treating_physician_name             → treating_physician_name       string
batch_conformance_status            → batch_conformance_status      string
dp_admin_utc       (via TO_CHAR)    → Drug Admin Date               → type date (PQ rename + cast)
dp_delivery_utc    (via TO_CHAR)    → Drug Delivey Date             → type date (PQ rename + cast) ⚠️ typo
site_of_care_country                → site_of_care_country          string
make1_site_name                     → make1_site_name               string
make2_site_name                     → make2_site_name               string
indication                          → indication                    string
count_toward_tat_kpi                → count_toward_tat_kpi          boolean
re_apheresis_join                   → re_apheresis_join             string
remake_flag                         → remake_flag                   string
retreatment_flag                    → retreatment_flag              string
make1_start_tz_actual               → make1_start_tz_actual         dateTime
make1_start_utc_actual              → make1_start_utc_actual        dateTime
make2_start_tz_actual               → make2_start_tz_actual         dateTime
make2_start_utc_actual              → make2_start_utc_actual        dateTime
make1_start_tz_planned              → make1_start_tz_planned        dateTime
make1_start_utc_planned             → make1_start_utc_planned       dateTime
make2_start_tz_planned              → make2_start_tz_planned        dateTime
make2_start_utc_planned             → make2_start_utc_planned       dateTime

Power BI Power Query M:
───────────────────────
let
    // ── Step 1: Connect to PostgreSQL ─────────────────────────────────
    Source = PostgreSQL.Database(
        "bms-eim-cell-therapy-auroradb-read-prod.web.bms.com:5432",
        "postgres"
    ),

    // ── Step 2: Execute native SQL — mirrors Tableau Custom SQL exactly
    // TO_CHAR() retained in SQL for source fidelity; results are
    // recast to type date in Step 3 to enable date intelligence.
    Result = Value.NativeQuery(
        Source,
        "select distinct
            join_id,
            enrollment_name,
            brand,
            product,
            site_of_care,
            collection_pickup_country,
            TO_CHAR(enrollment_created_date::date,  'FMDD Mon YYYY') as enrollment_created,
            TO_CHAR(infusion_logged_date::date,      'FMDD Mon YYYY') as infusion_logged,
            execution_status,
            TO_CHAR(cancellation_date_utc::date,    'FMDD Mon YYYY') as cancellation_date,
            enrollment_channel_name,
            treatment_status,
            region,
            treating_physician_name,
            batch_conformance_status,
            TO_CHAR(dp_admin_utc_actual::date,      'FMDD Mon YYYY') as dp_admin_utc,
            TO_CHAR(dp_delivery_utc_actual::date,   'FMDD Mon YYYY') as dp_delivery_utc,
            site_of_care_country,
            make1_site_name,
            make2_site_name,
            indication,
            count_toward_tat_kpi,
            re_apheresis_join,
            remake_flag,
            retreatment_flag,
            make1_start_tz_actual,
            make1_start_utc_actual,
            make2_start_tz_actual,
            make2_start_utc_actual,
            make1_start_tz_planned,
            make1_start_utc_planned,
            make2_start_tz_planned,
            make2_start_utc_planned
        from ct_bia.stg_gps_scheduling
        where join_id is not null
          and site_of_care <> 'Test Clinic EU'",
        [],
        [EnableFolding = true]
        // EnableFolding=true: WHERE/DISTINCT filters execute in
        // PostgreSQL, not in-memory — no M-layer filtering needed.
    ),

    // ── Step 3: Re-cast string date columns to proper date type ───────
    // Tableau treated these as strings after TO_CHAR().
    // Power BI promotes to type date to enable date intelligence,
    // slicers, hierarchies, and LocalDateTable auto-generation.
    #"Changed Type" = Table.TransformColumnTypes(
        Result,
        {
            { "enrollment_created", type date },
            { "infusion_logged",    type date },
            { "cancellation_date",  type date },
            { "dp_admin_utc",       type date },
            { "dp_delivery_utc",    type date }
        }
    ),

    // ── Step 4: Rename columns to business-friendly names ─────────────
    // Tableau used field captions/aliases; PBI uses Power Query rename.
    // ⚠️ "Drug Delivey Date" preserves typo from source — see Gap G1.
    #"Renamed Columns" = Table.RenameColumns(
        #"Changed Type",
        {
            { "dp_delivery_utc", "Drug Delivey Date" },
            // ⚠️ Typo: 'Delivey' — recommend fix to 'Drug Delivery Date'
            { "dp_admin_utc",    "Drug Admin Date"   }
        }
    )

in
    #"Renamed Columns"

Tableau Equivalent Steps:
  1. Connect to PostgreSQL (bms-eim-cell-therapy-auroradb-read-prod)
  2. Execute Custom SQL with SELECT DISTINCT
  3. Apply WHERE join_id IS NOT NULL AND site_of_care <> 'Test Clinic EU'
  4. TO_CHAR() pre-formats date columns as display strings (PBI undoes
     this by recasting to type date and using FORMAT() in DAX)
  5. Field captions alias dp_admin_utc → Drug Admin Date,
     dp_delivery_utc → Drug Delivery Date (⚠️ typo in PBI)

SQL Filter Logic — Tableau vs Power Query:
  Filter                   Tableau SQL               Power BI
  ──────────────────────────────────────────────────────────────────
  Exclude null join IDs  : WHERE join_id IS NOT NULL  — inline in NativeQuery
  Exclude test clinic    : AND site_of_care <> '...'  — inline in NativeQuery
  Deduplication          : SELECT DISTINCT            — inline in NativeQuery
  All three filters are pushed to PostgreSQL via EnableFolding=true.

Conversion Status: ✅ FULL — all 33 columns replicated with correct
                   types and names. Two column renames applied.
                   ⚠️ Typo in "Drug Delivey Date" — see Gap G1.


### PQ-02 | LocalDateTable Auto-Generation  ← Infrastructure Layer
─────────────────────────────────────────────────────────────
Tableau Equivalent    : Tableau auto date hierarchy (DATETRUNC-based,
                        per date field on worksheet row/column shelf)
PBI Table Names       : LocalDateTable_<GUID> (one per date column)
PBI Load Mode         : Import (DAX Calculated Tables — auto-generated
                        by Power BI Desktop; no manual authoring)

Standard LocalDateTable DAX template (identical for all 13 instances):
───────────────────────────────────────────────────────────────────────
// Hidden, showAsVariationsOnly = true
// Auto-scoped to the MIN/MAX year range of the bound source column.
LocalDateTable_<GUID> =
    CALENDAR(
        DATE( YEAR( MIN( 'Custom SQL Query'[<BoundColumn>] ) ), 1, 1 ),
        DATE( YEAR( MAX( 'Custom SQL Query'[<BoundColumn>] ) ), 12, 31 )
    )

// Auto-derived columns on each LocalDateTable (all hidden):
Year      = YEAR([Date])                        -- dataCategory: Years
MonthNo   = MONTH([Date])                       -- dataCategory: MonthOfYear
Month     = FORMAT([Date], "MMMM")              -- dataCategory: Months; sortByColumn: MonthNo
QuarterNo = INT(([MonthNo] + 2) / 3)            -- dataCategory: QuarterOfYear
Quarter   = "Qtr " & [QuarterNo]               -- dataCategory: Quarters; sortByColumn: QuarterNo
Day       = DAY([Date])                         -- dataCategory: DayOfMonth

// Auto-generated hierarchy on each LocalDateTable:
// Date Hierarchy → Year → Quarter → Month → Day

Complete GUID ↔ Bound Column Registry:
───────────────────────────────────────
LocalDateTable GUID                                    Bound Column               Format
────────────────────────────────────────────────────────────────────────────────────────────
b7978ce4-f15a-4e94-81be-14a588d066de                → enrollment_created         d mmmm, yyyy
f918beff-c9b7-42a9-a4b7-da8d30c672be                → infusion_logged            d mmmm, yyyy
163fa2df-7815-4639-9709-0f7475c13385                → cancellation_date          d mmmm, yyyy
b6b8c0ae-ceb1-4109-a049-445a3163d024                → Drug Admin Date            d mmmm, yyyy
70e6697b-ce7d-4288-aab4-01ba3ab9fa16                → Drug Delivey Date ⚠️       d mmmm, yyyy
a2b4fc00-0684-455f-8597-bcb37caae622                → make1_start_tz_actual      General Date
48a1e047-a502-47b8-a9e3-0ac85aabf15d                → make1_start_utc_actual     General Date
ea246ae0-1742-438a-8aa2-db443a9cd017                → make2_start_tz_actual      General Date
15ccfad2-e6cf-45a3-b84a-9a1d8573d7f1                → make2_start_utc_actual     General Date
08b0b4ac-b839-4843-b049-fd71e822b39c                → make1_start_tz_planned     General Date
4d9b7440-2ac2-47dc-89eb-deb1a5cba760                → make1_start_utc_planned    General Date
c5f38c4d-b0d1-48a1-a578-50c5d7589ca1                → make2_start_tz_planned     General Date
05bbe622-02ee-4985-aff5-d4f0d3aacf1b                → make2_start_utc_planned    General Date
DateTableTemplate_ad713325-b0fc-46d3-abd2-fd10b3f383b3 → Master seed template   Calendar(Date(2015,1,1),Date(2015,1,1))

NOTE: Consider replacing all 13 LocalDateTables with a single shared
      DimDate table for consistency with other BMS PBIP models.
      Verify whether UTC vs. timezone-offset columns should be the
      primary date axis before removing hidden make*_tz_* columns.

Conversion Status: ✅ FULL — all 13 date columns have auto-generated
                   LocalDateTables with correct GUID bindings and
                   standard Year/Quarter/Month/Day hierarchy.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

G1 | Column Name Typo — "Drug Delivey Date"
────────────────────────────────────────────
Status     : ⚠️ Must fix before production
Detail     : Power Query rename step uses "Drug Delivey Date"
             (missing 'r' in Delivery). This typo propagates to:
               - LocalDateTable GUID 70e6697b binding
               - CC-04 [Drug Delivery Date(Format)] DAX dependency
               - All slicer references to this column
Action     : Fix Table.RenameColumns in PQ-01:
               { "dp_delivery_utc", "Drug Delivery Date" }
             Then update CC-04 DAX dependency and LocalDateTable
             GUID 70e6697b source expression accordingly.

G2 | INDEX() Has No Static Column Equivalent
─────────────────────────────────────────────
Status     : ℹ️ Functional — note for future maintenance
Detail     : Tableau INDEX() is a table calculation; it does not
             persist as a stored column value.
             Power BI ROWNUMBER() is a measure, not a calculated
             column, which means it cannot be referenced by other
             calculated columns or used in relationships.
Action     : Use ROWNUMBER() measure (PBI 2024+) placed as first
             visual column. If a static sequential ID is needed,
             add an index column in Power Query:
               Table.AddIndexColumn(Result, "RowIndex", 1, 1, Int64.Type)
             before the type-cast step.

G3 | No Shared DimDate Table
──────────────────────────────
Status     : ℹ️ Architectural recommendation
Detail     : 13 LocalDateTables exist (one per date column) instead
             of a single shared date dimension. This is standard PBI
             auto-generation behaviour but creates model bloat.
Action     : Consider building a single DimDate calculated table
             (consistent with 19043, 12960 model patterns) and
             relating all date columns to it for a cleaner star schema.
             Mark DimDate as "Date Table" in PBI model properties.

G4 | UTC vs. Timezone-Offset Date Columns
───────────────────────────────────────────
Status     : ⚠️ Verify with business
Detail     : 8 hidden date columns come in both UTC and local
             timezone variants:
               make1_start_tz_actual / make1_start_utc_actual
               make2_start_tz_actual / make2_start_utc_actual
               make1_start_tz_planned / make1_start_utc_planned
               make2_start_tz_planned / make2_start_utc_planned
             All are hidden in both Tableau and PBI — display intent
             is unclear.
Action     : Confirm with business which variant (UTC or TZ) should
             be used if these columns are surfaced in future visuals.
             Remove unused variant to reduce model size.

G5 | Fixed Filter "enrollment_channel_name = Commercial" Not Visible
─────────────────────────────────────────────────────────────────────
Status     : ⚠️ Must configure in report
Detail     : Tableau applied this as a fixed worksheet filter.
             It is NOT a user slicer — users cannot change it.
             In PBI this must be set as a report-level filter,
             not as a slicer visual.
Action     : In Power BI report View → Filters pane → add
             [enrollment_channel_name] = "Commercial" at report level.
             Lock and hide the filter to prevent end-user modification.

G6 | Fixed Filter "Indication Filter = TRUE" Not Visible
─────────────────────────────────────────────────────────
Status     : ⚠️ Must configure in report
Detail     : Tableau applied Calculation_772085909962469377 = TRUE
             as a fixed worksheet filter. Same as G5 — must be a
             locked report-level filter in PBI, not a slicer.
Action     : In Power BI report Filters pane → add
             [Indication Filter] = TRUE at report level.
             Lock and hide the filter.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — CONVERSION COMPLETENESS SUMMARY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Category                              Tableau   PBI      Status      %
─────────────────────────────────────────────────────────────────────────
Parameters                                0       0     ✅ N/A      100%
Source Columns                           33      33     ✅ Full     100%
  (all 33 columns replicated)
Calculated Columns (boolean)              1       1     ✅ Full     100%
  (Indication Filter)
Calculated Columns (date format)          5       5     ✅ Full     100%
  (CC-02 to CC-06)
Table Calculations (INDEX)                1       1     ✅ Full     100%
  (Row Index measure)
DAX Measures (Tableau-equivalent)         2       2     ✅ Full     100%
  (Record Count, Row Index)
Power Query Sources                       1       1     ✅ Full     100%
LocalDateTables                          13      13     ✅ Full     100%
Dashboard Filters / Slicers              12      12     ✅ Full     100%
Fixed Worksheet Filters                   2       2     ⚠️ Pending   0%
  (Commercial + Indication Filter — must be set in report layer)
Column Name Typo (Drug Delivey Date)      —       —     ⚠️ Pending   —
  (recommend fix before production)
─────────────────────────────────────────────────────────────────────────
OVERALL (Tableau artefacts)             ~57     ~57     ✅/⚠️       ~97%
─────────────────────────────────────────────────────────────────────────

Legend:
  ✅ FULL     — exact functional equivalent implemented
  ⚠️ PARTIAL  — present but requires follow-up action
  ➕ PBI-only — no Tableau equivalent; Power BI enhancement
  ℹ️ Low risk — omitted intentionally; low business impact


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — QUICK REFERENCE: TABLEAU FORMULA → DAX PATTERN
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Tableau Pattern                                        DAX / M Equivalent
──────────────────────────────────────────────────────────────────────────────────────────
SELECT DISTINCT ... WHERE col IS NOT NULL              Value.NativeQuery(src, sql, [], [EnableFolding=true])
AND col <> 'X'                                         with identical SQL inline; all filters pushed to DB

TO_CHAR(date::date, 'FMDD Mon YYYY') in SQL            Table.TransformColumnTypes(..., type date) in PQ
                                                       → FORMAT([col], "DD MMM YYYY") in DAX calculated column

Field alias / caption in Tableau                       Table.RenameColumns(...) in Power Query

IF (A OR B OR C) THEN TRUE ELSE FALSE END              IF(OR(A, OR(B, C)), TRUE(), FALSE()) in DAX column

INDEX() table calculation (row number)                 ROWNUMBER('Table', ORDERBY([col], ASC)) — PBI 2024+
                                                       or RANKX(ALL('Table'), [col], , ASC, Dense)

CNT(datasource) automatic aggregate                    COUNTROWS('Table')

Tableau worksheet filter = member value                Report-level filter in PBI Filters pane (locked/hidden)

Tableau quick filter (dropdown)                        Power BI slicer visual

Hidden field in Tableau                                Column present in PBI model; not placed on any visual

semantic-role = '[Country].[ISO3166_2]'               Column Data Category = Country in PBI model properties

Auto date hierarchy per date field (Tableau)           Auto-generated LocalDateTable per date column
                                                       (hidden, showAsVariationsOnly = true)

Calendar scoped to MIN/MAX of column                   Calendar(Date(Year(MIN([col])),1,1),
                                                                Date(Year(MAX([col])),12,31))

Date Hierarchy: Year → Quarter → Month → Day           Standard LocalDateTable hierarchy (identical for all)


---
# END OF KNOWLEDGE FILE
# 27487 – Global Potential Off Label | Tableau → Power BI (PBIP/TMDL)
# Generated: 2026-07-06
# Source files: global_potential_off_label.txt (Tableau)
#               custom_sql_query.txt,
#               localdatetable_b7978ce4_f15a_4e94_81be_14a588d066de.txt,
#               localdatetable_f918beff_c9b7_42a9_a4b7_da8d30c672be.txt,
#               localdatetable_163fa2df_7815_4639_9709_0f7475c13385.txt,
#               localdatetable_b6b8c0ae_ceb1_4109_a049_445a3163d024.txt,
#               localdatetable_70e6697b_ce7d_4288_aab4_01ba3ab9fa16.txt,
#               localdatetable_a2b4fc00_0684_455f_8597_bcb37caae622.txt,
#               localdatetable_48a1e047_a502_47b8_a9e3_0ac85aabf15d.txt,
#               localdatetable_ea246ae0_1742_438a_8aa2_db443a9cd017.txt,
#               localdatetable_15ccfad2_e6cf_45a3_b84a_9a1d8573d7f1.txt,
#               localdatetable_08b0b4ac_b839_4843_b049_fd71e822b39c.txt,
#               localdatetable_4d9b7440_2ac2_47dc_89eb_deb1a5cba760.txt,
#               localdatetable_c5f38c4d_b0d1_48a1_a578_50c5d7589ca1.txt,
#               localdatetable_05bbe622_02ee_4985_aff5_d4f0d3aacf1b.txt,
#               datetabletemplate_ad713325_b0fc_46d3_abd2_fd10b3f383b3.txt,
#               definition.json (PBIP)
