# 📘 CONVERSION KNOWLEDGE FILE
# K1 – Enterprise Exchange Dashboard
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-23
# Source files: enterprise_exchange_submissions (Tableau workbook XML)
#               out_custom_sql_query.txt
#               out_measures_table.txt
#               out_relationships.txt
#               out_database.txt
#               out_model.txt
#               out_en_us.txt
#               out_diagramlayout.json
#               out_localdatetable_submission_week.txt
#               out_localdatetable_submission_date.txt
#               out_localdatetable_week_start.txt

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

No Tableau parameters were identified in the Enterprise Exchange
workbook for pair_1. The conversion framework supports parameters
as a conditional output (out_parameters.txt → Parameters.tmdl) if
future workbook revisions introduce them.

⚠️  NOTE: If Tableau parameters are added in a future workbook update,
    follow the standard conversion patterns:
    - String list parameter → DATATABLE or NAMEOF field parameter
    - Date range parameter → single-row parameter table with M Query
    - Boolean parameter → single-row boolean parameter table

Conversion Status: ✅ N/A (no parameters in source workbook)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: Custom SQL Query

### CC-01 | Territory
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-territory
PBI DataType    : string
PBI Hidden      : changedProperty = IsHidden
PBI Comment     : // Territory display name (concatenate name + form override)

Tableau Expression:
───────────────────
[territory_name] + IF TRIM(LOWER([Territory (form)])) = TRIM(LOWER([territory_name]))
                   THEN "" ELSE "(" + [Territory (form)] + ")" END
// Appends the form territory name in parentheses when it differs
// from the standard territory name

Power BI DAX:
─────────────
column Territory = 'Custom SQL Query'[territory_name] & IF(
    TRIM(LOWER('Custom SQL Query'[Territory (form)])) = TRIM(LOWER('Custom SQL Query'[territory_name])),
    "",
    "(" & 'Custom SQL Query'[Territory (form)] & ")")

Depends on  : [territory_name], [Territory (form)] (source columns)
Business Rule: Displays territory_name alone when both fields match
               (case-insensitive, trimmed); otherwise appends form
               override in parentheses for disambiguation.
Conversion  : ✅ FULL


### CC-02 | #ARMs
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-arms
PBI DataType    : string
PBI Hidden      : changedProperty = IsHidden
PBI Comment     : // ARM identifier string

Tableau Expression:
───────────────────
[Submitter] + CHAR(13) + [first_name] + " " + [last_name]
// Combines submitter email with roster name, separated by carriage return

Power BI DAX:
─────────────
column '#ARMs' = 'Custom SQL Query'[Submitter] & UNICHAR(13) & 'Custom SQL Query'[first_name] & " " & 'Custom SQL Query'[last_name]

Depends on  : [Submitter], [first_name], [last_name] (source columns)
Pattern     : Tableau CHAR(13) → DAX UNICHAR(13) (carriage return)
⚠️  NAMING NOTE: Column name starts with '#' — valid in TMDL when
    enclosed in single quotes. Preserved exactly.
Conversion  : ✅ FULL


### CC-03 | Submission Week
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-submission-week
PBI DataType    : dateTime
PBI FormatString: Short Date
PBI Hidden      : changedProperty = IsHidden

Tableau Expression:
───────────────────
DATETRUNC('week', [Submission Date], 'monday')
// Truncates submission date to Monday-start ISO week

Power BI DAX:
─────────────
column 'Submission Week' = 'Custom SQL Query'[Submission Date] - WEEKDAY('Custom SQL Query'[Submission Date], 2) + 1

Depends on  : [Submission Date] (source dateTime column)
Pattern     : Tableau DATETRUNC('week', date, 'monday') →
              DAX [Date] - WEEKDAY([Date], 2) + 1
NOTE        : Has a date variation relationship to
              LocalDateTable_2e8b88fa-14b2-4993-9f15-905d8ee79573.
              GUID: 141f5ec5-fd3c-4a31-96a5-7768a11081d5.
⚠️  ANNOTATION: UnderlyingDateTimeDataType = Date (date-only semantics).
Conversion  : ✅ FULL


### CC-04 | Action (Category,Submission Date)
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-action-category-submission-date
PBI DataType    : string
PBI Hidden      : isHidden = true

Tableau Expression:
───────────────────
// Dashboard cross-sheet action filter key — concatenates Category and
// Submission Date for use as an action filter target

Power BI DAX:
─────────────
column 'Action (Category,Submission Date)' = 'Custom SQL Query'[Category] & " , " & 'Custom SQL Query'[Submission Date]

Depends on  : [Category], [Submission Date] (source columns)
Purpose     : Tableau action filter emulation — concatenated string column
              for cross-visual filtering. Hidden in PBI field list.
Pattern     : Tableau action filter → DAX concatenated hidden column
              (see PATTERN-06 in Section 6).
Conversion  : ✅ FULL


### CC-05 | Action (Category,Topic)
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-action-category-topic
PBI DataType    : string
PBI Hidden      : isHidden = true

Power BI DAX:
─────────────
column 'Action (Category,Topic)' = 'Custom SQL Query'[Category] & " , " & 'Custom SQL Query'[Topic]

Depends on  : [Category], [Topic] (source columns)
Purpose     : Tableau action filter emulation.
Conversion  : ✅ FULL


### CC-06 | Action (Submission Week,Category)
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-action-submission-week-category
PBI DataType    : string
PBI Hidden      : isHidden = true

Power BI DAX:
─────────────
column 'Action (Submission Week,Category)' = 'Custom SQL Query'[Submission Week] & " , " & 'Custom SQL Query'[Category]

Depends on  : CC-03 [Submission Week], [Category] (source column)
Purpose     : Tableau action filter emulation.
⚠️  DEPENDENCY NOTE: References calculated column [Submission Week] (CC-03),
    not a source column. DAX evaluation order handles this automatically
    in calculated columns.
Conversion  : ✅ FULL


### CC-07 | Week Start
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-week-start
PBI FormatString: dd-mmm-yy
PBI Hidden      : (not explicitly hidden)

Tableau Expression:
───────────────────
DATETRUNC('week', [Submission Date], 'monday')
// Same logic as CC-03 Submission Week, different display format

Power BI DAX:
─────────────
column 'Week Start' = 'Custom SQL Query'[Submission Date] - WEEKDAY('Custom SQL Query'[Submission Date], 2) + 1

Depends on  : [Submission Date] (source dateTime column)
Pattern     : Tableau DATETRUNC('week', date, 'monday') →
              DAX [Date] - WEEKDAY([Date], 2) + 1
NOTE        : Same calculation as CC-03 [Submission Week] but with
              format "dd-mmm-yy" instead of "Short Date". Has a date
              variation relationship to
              LocalDateTable_37266a76-90b5-4078-942d-75a59988e0d3.
              GUID: 3fdd7d34-51e1-40ff-b66e-7a36643a4824.
⚠️  ANNOTATION: PBI_FormatHint = {"isDateTimeCustom":true}
Conversion  : ✅ FULL


### CC-08 | Topic2
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-topic2
PBI DataType    : string (implicit)

Tableau Expression:
───────────────────
IFNULL([Topic], "Null")
// Replaces null Topic values with the string "Null"

Power BI DAX:
─────────────
column Topic2 = COALESCE('Custom SQL Query'[Topic], "Null")

Depends on  : [Topic] (source column)
Pattern     : Tableau IFNULL → DAX COALESCE
NOTE        : The PQ-level M query already performs null replacement
              on the [region] column via Table.ReplaceValue. This
              calculated column handles the same pattern for [Topic]
              at the DAX layer.
Conversion  : ✅ FULL


### CC-09 | Submission Week dmy
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-submission-week-dmy
PBI DataType    : string (implicit)

Tableau Expression:
───────────────────
// Formatted display string of Submission Week as "dd-MMM-yyyy"

Power BI DAX:
─────────────
column 'Submission Week dmy' = FORMAT('Custom SQL Query'[Submission Week], "dd-MMM-yyyy")

Depends on  : CC-03 [Submission Week]
Purpose     : Human-readable date label for slicers/visuals, sorted by
              CC-10 [Submission Week dmy sort] for chronological ordering.
              sortByColumn: 'Submission Week dmy sort'
Conversion  : ✅ FULL


### CC-10 | Submission Week dmy sort
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-submission-week-dmy-sort
PBI DataType    : string (implicit)

Power BI DAX:
─────────────
column 'Submission Week dmy sort' = FORMAT('Custom SQL Query'[Submission Week], "yyyymmdd")

Depends on  : CC-03 [Submission Week]
Purpose     : Sort key for CC-09 [Submission Week dmy]. FORMAT returns a
              string in "yyyymmdd" format for correct lexicographic
              (chronological) sorting.
NOTE        : CC-09 has sortByColumn referencing this column. Both are
              string type — sorting is lexicographic on the "yyyymmdd"
              pattern.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: Measures Table

### M-01 | Total Submissions
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI lineageTag  : mt-meas-total-submissions
PBI Format      : #,##0

Tableau Equivalent:
───────────────────
COUNT([Status])
// Counts all rows via the Status field (non-null status = valid submission)

Power BI DAX:
─────────────
measure 'Total Submissions' = COUNT('Custom SQL Query'[Status])

Business Rule: Total number of submissions in the Enterprise Exchange
               system, counted by non-null Status values.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU INITIAL TRANSFORMATIONS → POWER BI POWER QUERY (M)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | Custom SQL Query (Enterprise Exchange main table)
─────────────────────────────────────────────────────────────
Tableau Source  : PostgreSQL direct connection
                  cdra_stg_pass.enterprise_exchange_submissions
                  cdra_stage.align1_dal_roster_pass
PBI Connection  : PostgreSQL.Database
                  (primary-passcdraprod-rds-db-instance-7ijwssk.cvegddsdpypp.us-east-1.rds.amazonaws.com:5432,
                   passcdraprod)
PBI Method      : Value.NativeQuery with EnableFolding = true
PBI Load Mode   : Import
PBI Partition   : 402bc123-6bbc-426d-a942-ac7c92535218_Partition

Power BI M Query (FULL):
─────────────────────────
let
    Source = PostgreSQL.Database(
        "primary-passcdraprod-rds-db-instance-7ijwssk.cvegddsdpypp.us-east-1.rds.amazonaws.com:5432",
        "passcdraprod"
    ),
    Query =
        "with r as (
    select first_name, last_name, email, district, district_name, territory_code,
           territory_name, role_code
    from cdra_stage.align1_dal_roster_pass
    where sales_force_id in ('009', '109')
    order by sales_force_id desc, territory_code, role_code desc
),
d as (
    select * from r
    where right(r.territory_code, 4) = '0000'
      and r.role_code = 'PRIM'
)
SELECT
    ees.status, submission_date, submitter,
    ees.territory_name AS territory_name_xl,
    therapeutic_area, category, topic, description,
    adro_response, adro_comment, ees.region, ees.director,
    roster_idx, idx, r.*,
    d.first_name || ' ' || d.last_name AS director_roster
FROM ""cdra_stg_pass"".""enterprise_exchange_submissions"" ees
LEFT OUTER JOIN r ON r.email = ees.submitter AND r.role_code = 'PRIM'
LEFT OUTER JOIN d ON d.territory_code = left(r.territory_code, 4) || '0000'
    AND d.role_code = 'PRIM'",
    Result = Value.NativeQuery(Source, Query, [], [EnableFolding=true]),
    #"Filtered Rows" = Table.SelectRows(Result, each true),
    #"Replaced Value"  = Table.ReplaceValue(#"Filtered Rows","","Null",Replacer.ReplaceValue,{"region"}),
    #"Replaced Value1" = Table.ReplaceValue(#"Replaced Value",null,"Null",Replacer.ReplaceValue,{"region"})
in
    #"Replaced Value1"

SQL Architecture:
  - CTE r: Selects roster data from align1_dal_roster_pass for sales
    forces '009' and '109', filtered to PRIM role.
  - CTE d: Filters r to district-level records (territory_code ends '0000',
    PRIM role) — provides director names.
  - Main SELECT: Joins enterprise_exchange_submissions with roster (r) on
    submitter email and with director (d) on left-4-digits of territory_code.
    Concatenates director first + last name as director_roster.

⚠️  M STEP NOTE: Two sequential Table.ReplaceValue steps on the [region]
    column — first replaces empty string "" with "Null", then replaces
    null with "Null". This ensures all blank/null regions display as
    the string "Null" consistently.
⚠️  SCHEMA NOTE: The SQL uses double-quoted schema/table names
    (""cdra_stg_pass"".""enterprise_exchange_submissions"") — this is
    required for case-sensitive PostgreSQL identifiers.


### PQ-02 | Measures Table (empty measures host)
─────────────────────────────────────────────────────────────
Tableau Equivalent : n/a (PBI-only pattern)
Purpose            : Empty table hosting all DAX measures.
                     Enterprise Exchange has a single measure (M-01)
                     on this table.

Power BI M Query:
─────────────────
let
    Source = Table.FromRows(
        Json.Document(
            Binary.Decompress(
                Binary.FromText("i44FAA==", BinaryEncoding.Base64),
                Compression.Deflate
            )
        ),
        let _t = ((type nullable text) meta [Serialized.Text = true])
        in type table [Column1 = _t]
    ),
    #"Removed Columns" = Table.RemoveColumns(Source, {"Column1"})
in
    #"Removed Columns"

NOTE: Standard PBI empty measures table pattern using Base64-compressed
      empty JSON. The Column1 is created then immediately removed,
      leaving a zero-column table that serves only as a host for measures.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — CONVERSION COMPLETENESS SCORECARD
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Category                         | Tableau Items | PBI Full | PBI Partial | Not Converted | % Complete |
|----------------------------------|---------------|----------|-------------|---------------|------------|
| Parameters                       | 0             | 0        | 0           | 0             | N/A        |
| Calculated Columns               | 10            | 10       | 0           | 0             | 100%       |
| Measures                         | 1             | 1        | 0           | 0             | 100%       |
| Power Query / SQL Tables         | 2             | 2        | 0           | 0             | 100%       |
| Relationships                    | 3             | 3        | 0           | 0             | 100%       |
| TMDL Infrastructure Files        | 5             | 5        | 0           | 0             | 100%       |
| TOTAL                            | 21            | 21       | 0           | 0             | 🟢 100%    |

PARTIAL ITEMS: None.

NOTE: The Enterprise Exchange workbook (pair_1) is a relatively simple
      single-table model with one data source, 10 calculated columns,
      1 measure, and 3 date-variation relationships. All items fully
      converted.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — CONVERSION PATTERN CHEAT SHEET
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PATTERN-01 | Tableau IF/ELSEIF → DAX SWITCH
─────────────────────────────────────────────────────────────
Tableau:
  IF [x]=1 THEN "A" ELSEIF [x]=2 THEN "B" ELSE "C" END

Power BI:
  SWITCH([x], 1, "A", 2, "B", "C")

For inequality chains:
  IF [x]>10 THEN "Hi" ELSEIF [x]>5 THEN "Mid" ELSE "Lo" END
  →
  SWITCH(TRUE(), [x]>10, "Hi", [x]>5, "Mid", "Lo")


PATTERN-02 | Tableau LOD FIXED → DAX CALCULATE + ALLEXCEPT
─────────────────────────────────────────────────────────────
Tableau:
  { FIXED [Dim1], [Dim2] : COUNTD([Key]) }

Power BI:
  CALCULATE(
      DISTINCTCOUNT([Key]),
      ALLEXCEPT('Table', 'Table'[Dim1], 'Table'[Dim2])
  )

WHY: ALLEXCEPT removes all column filters except the named ones,
     replicating Tableau's FIXED scope.

Alternative as measure:
  { FIXED [A] : MAX([Val]) }
  →
  SUMX(SUMMARIZE(T, T[A]), CALCULATE(MAX(T[Val])))


PATTERN-03 | Tableau LOD EXCLUDE → DAX REMOVEFILTERS
─────────────────────────────────────────────────────────────
Tableau:
  { EXCLUDE [Cat] : SUM([Sales]) }

Power BI:
  CALCULATE(SUM(T[Sales]), REMOVEFILTERS(T[Cat]))


PATTERN-04 | Tableau LOD INCLUDE → DAX SUMX + VALUES
─────────────────────────────────────────────────────────────
Tableau:
  { INCLUDE [Sub] : SUM([Sales]) }

Power BI:
  SUMX(VALUES(T[Sub]), CALCULATE(SUM(T[Sales])))


PATTERN-05 | Tableau DATETRUNC → DAX week start calculation
─────────────────────────────────────────────────────────────
Tableau:
  DATETRUNC('week', [Date], 'monday')

Power BI:
  T[Date] - WEEKDAY(T[Date], 2) + 1

This model uses this pattern in: CC-03, CC-07.

Other date patterns:
  DATEPART('year', [Date])      → YEAR(T[Date])
  DATEPART('month', [Date])     → MONTH(T[Date])
  DATEPART('iso-weekday', [Date]) → WEEKDAY(T[Date], 2)
  DATETRUNC('month', [Date])    → DATE(YEAR(T[Date]), MONTH(T[Date]), 1)
  DATEDIFF('day', [S], [E])     → DATEDIFF(T[S], T[E], DAY)
  DATEADD('month', -1, TODAY()) → EDATE(TODAY(), -1)


PATTERN-06 | Tableau action filter → PBI concatenated hidden column
─────────────────────────────────────────────────────────────
Tableau:
  Action filter passing multiple fields between dashboard sheets.

Power BI:
  column 'Action (Field1,Field2)' = T[Field1] & " , " & T[Field2]
      dataType: string
      isHidden
      lineageTag: tbl-action-field1-field2   // kebab-case only
      summarizeBy: none
      annotation SummarizationSetBy = Automatic

This model has THREE action columns:
  CC-04: Action (Category,Submission Date)
  CC-05: Action (Category,Topic)
  CC-06: Action (Submission Week,Category)

All follow the same pattern: concatenated string, hidden, kebab-case
lineageTag per naming convention (see Section 6a).


PATTERN-07 | Tableau ISNULL → DAX ISBLANK / COALESCE
─────────────────────────────────────────────────────────────
Tableau:
  ISNULL([field])

Power BI:
  ISBLANK([field])

Note: DAX ISBLANK() returns TRUE for both NULL and empty string "".
      If empty string ≠ NULL matters, use:
      [field] = BLANK() (NULL only) or LEN([field]) = 0 (empty string only).

Alternative: Tableau ZN([x]) → DAX IF(ISBLANK(T[x]), 0, T[x])
             Tableau IFNULL([x], "val") → DAX COALESCE(T[x], "val")

This model uses COALESCE in: CC-08 (Topic2).


PATTERN-08 | Tableau string functions → DAX equivalents
─────────────────────────────────────────────────────────────
  FIND([S], "sub")          → SEARCH("sub", T[S], 1, 0)
  LEFT([S], n)              → LEFT(T[S], n)
  CONTAINS([F], "v")        → SEARCH("v", T[F], 1, 0) > 0
  STR([N])                  → FORMAT(T[N], "0")
  TRIM([S])                 → TRIM(T[S])
  LOWER([S])                → LOWER(T[S])

This model uses TRIM + LOWER in: CC-01 (Territory comparison).


PATTERN-09 | Tableau aggregation functions → DAX equivalents
─────────────────────────────────────────────────────────────
  SUM([Sales])      → SUM(T[Sales])
  COUNT([ID])       → COUNT(T[ID])
  COUNTD([ID])      → DISTINCTCOUNT(T[ID])
  AVG([Sales])      → AVERAGE(T[Sales])

This model uses COUNT in: M-01 (Total Submissions).


PATTERN-10 | Tableau comment preservation in DAX
─────────────────────────────────────────────────────────────
For every calculated column that originated from a Tableau expression, add:
  /* Original Tableau Expression is <formula> */
at the end of the DAX expression.
Use /// header comment for column display name:
  /// My Column Name - Column
Use // line comments for inline documentation:
  // Territory display name (concatenate name + form override)
These comments appear in Tabular Editor and VS Code TMDL view.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6a — LINEAGETAG NAMING CONVENTIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Object             | Pattern                              | Example                                  |
|--------------------|--------------------------------------|------------------------------------------|
| Source column      | `<prefix>-col-<name>`                | `csq-col-status`                         |
| Calculated column  | `<prefix>-calc-<name>`               | `csq-calc-territory`                     |
| Measure            | `<prefix>-meas-<name>`               | `mt-meas-total-submissions`              |
| Action column      | `<prefix>-action-<slug>`             | `csq-action-category-topic`              |
| Hierarchy          | `<prefix>-hier-<name>`               | `csq-hier-territory`                     |
| Hierarchy level n  | `<prefix>-hier-lvl-<n>`              | `csq-hier-lvl-1`                         |
| Table              | `<prefix>-0001-0000-0000-<12digits>` | `csq-0001-0000-0000-000000000001`        |

Table prefix codes:
  csq  = Custom SQL Query
  mt   = Measures Table
  param = Parameters (if applicable)
  ldt<n> = LocalDateTable n (e.g., ldt1, ldt2, ldt3)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAP-01 | Duplicate Week Calculation (CC-03 and CC-07)
  Severity : Low (informational)
  Detail   : Both [Submission Week] (CC-03) and [Week Start] (CC-07)
             use identical DAX logic:
             [Submission Date] - WEEKDAY([Submission Date], 2) + 1
             They differ only in formatString (Short Date vs. dd-mmm-yy)
             and each has its own LocalDateTable date variation.
  Action   : Confirm with report owner whether both columns are needed
             or if one can be removed. If both are retained, ensure
             visuals reference the correct column for their format needs.


GAP-02 | Region Null Replacement at Two Layers
  Severity : Low (informational)
  Detail   : The [region] column has null replacement at the Power Query
             level (two Table.ReplaceValue steps: "" → "Null" and
             null → "Null"), while [Topic] null replacement is handled
             at the DAX layer (CC-08 Topic2 = COALESCE). This
             inconsistency is preserved from the original PBIP.
  Action   : Consider standardizing null handling to one layer (either
             all in PQ or all in DAX) for maintainability.


GAP-03 | Tableau Action Columns → PBI Cross-Visual Interactions
  Severity : Low (report-layer)
  Detail   : CC-04 [Action (Category,Submission Date)], CC-05 [Action
             (Category,Topic)], and CC-06 [Action (Submission Week,
             Category)] are Tableau cross-sheet navigation action columns.
             In PBI these require report-layer bookmark navigation or
             button actions configured on the report canvas. No TMDL
             equivalent for the navigation behavior itself.
  Action   : Configure PBI report canvas buttons/bookmarks to replicate
             the Tableau tab-navigation behavior using these action columns
             as filter targets.


GAP-04 | Sales Force ID Hardcoded in SQL
  Severity : Informational
  Detail   : The CTE r in PQ-01 SQL filters on
             sales_force_id IN ('009', '109'). If additional sales
             forces need to be included, the M query SQL must be updated
             manually.
  Action   : Consider externalizing to a parameter table if the sales
             force list changes frequently.


GAP-05 | Territory Code District Logic Hardcoded
  Severity : Informational
  Detail   : The CTE d identifies district-level records by checking
             right(territory_code, 4) = '0000'. This convention
             (trailing 4 zeros = district) is a business rule embedded
             in SQL. If territory code structure changes, both CTEs
             must be updated.
  Action   : Document the territory code structure with data owner.


GAP-06 | No RLS (Row-Level Security) Defined
  Severity : Informational
  Detail   : The Enterprise Exchange workbook has no Tableau RLS
             or PBI roles defined. If data access restrictions are
             needed, roles.tmdl must be created.
  Action   : Confirm with report owner whether RLS is required
             for Enterprise Exchange data.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — FILE INVENTORY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| #  | Output Filename                              | Rename To                                            | Folder                | Content                                           |
|----|----------------------------------------------|------------------------------------------------------|-----------------------|---------------------------------------------------|
| 1  | out_database.txt                             | database.tmdl                                        | /definition/          | Database compatibility level (1604)               |
| 2  | out_model.txt                                | model.tmdl                                           | /definition/          | Model metadata, culture, table refs               |
| 3  | out_custom_sql_query.txt                     | Custom SQL Query.tmdl                                | /definition/tables/   | Main data table with source + calculated columns  |
| 4  | out_measures_table.txt                       | Measures Table.tmdl                                  | /definition/tables/   | Empty measures host table (M-01)                  |
| 5  | out_relationships.txt                        | relationships.tmdl                                   | /definition/          | 3 date-variation relationships                    |
| 6  | out_datetabletemplate.txt                     | DateTableTemplate_<GUID>.tmdl                        | /definition/tables/   | Auto date table template                          |
| 7  | out_en_us.txt                                | en-US.tmdl                                           | /definition/cultures/ | Linguistic metadata (en-US)                       |
| 8  | out_diagramlayout.json                       | diagramLayout.json                                   | /definition/          | Model diagram layout (Custom SQL Query only)      |
| 9  | out_localdatetable_submission_week.txt        | LocalDateTable_2e8b88fa-14b2-4993-9f15-905d8ee79573.tmdl | /definition/tables/ | Date hierarchy for [Submission Week]              |
| 10 | out_localdatetable_submission_date.txt        | LocalDateTable_572388b4-2112-40ad-9630-af36b8afc93e.tmdl | /definition/tables/ | Date hierarchy for [Submission Date]              |
| 11 | out_localdatetable_week_start.txt             | LocalDateTable_37266a76-90b5-4078-942d-75a59988e0d3.tmdl | /definition/tables/ | Date hierarchy for [Week Start]                   |

Conditional (not generated for pair_1):
  - out_parameters.txt → Parameters.tmdl (if Tableau parameters exist)
  - out_roles.txt → roles.tmdl (if Tableau RLS exists)
  - out_theme.json → theme.json (if custom color palette exists)

.pbip folder structure:
```
<ProjectName>.SemanticModel/definition/
├── database.tmdl
├── model.tmdl
├── relationships.tmdl
├── diagramLayout.json
├── tables/
│   ├── Custom SQL Query.tmdl
│   ├── Measures Table.tmdl
│   ├── DateTableTemplate_<GUID>.tmdl
│   ├── LocalDateTable_2e8b88fa-....tmdl   (Submission Week)
│   ├── LocalDateTable_572388b4-....tmdl   (Submission Date)
│   └── LocalDateTable_37266a76-....tmdl   (Week Start)
└── cultures/
    └── en-US.tmdl
```


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — TMDL FILE TEMPLATES & INFRASTRUCTURE
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### TMPL-01 | database.tmdl
─────────────────────────────────────────────────────────────
```
database
    compatibilityLevel: 1604
```


### TMPL-02 | model.tmdl
─────────────────────────────────────────────────────────────
```
model Model
    culture: en-US
    defaultPowerBIDataSourceVersion: powerBI_V3
    sourceQueryCulture: en-US
    dataAccessOptions
        legacyRedirects
        returnErrorValuesAsNull

    annotation __TEdtr = 1
    annotation __PBI_TimeIntelligenceEnabled = 1
    annotation PBI_QueryOrder = ["Measures Table","Custom SQL Query"]
    annotation PBI_ProTooling = ["DevMode"]

    ref table 'Measures Table'
    ref table 'Custom SQL Query'
    ref table DateTableTemplate_<GUID>
    ref table LocalDateTable_<GUID>
    ref cultureInfo en-US
```


### TMPL-03 | relationships.tmdl (pair_1 actual)
─────────────────────────────────────────────────────────────
```
// Week Start date variation
relationship 3fdd7d34-51e1-40ff-b66e-7a36643a4824
    joinOnDateBehavior: datePartOnly
    fromColumn: 'Custom SQL Query'.'Week Start'
    toColumn: LocalDateTable_37266a76-90b5-4078-942d-75a59988e0d3.Date

// Submission Date date variation
relationship bbe4a354-0f5e-4c9f-8381-bd4fafcd17df
    joinOnDateBehavior: datePartOnly
    fromColumn: 'Custom SQL Query'.'Submission Date'
    toColumn: LocalDateTable_572388b4-2112-40ad-9630-af36b8afc93e.Date

// Submission Week date variation
relationship 141f5ec5-fd3c-4a31-96a5-7768a11081d5
    joinOnDateBehavior: datePartOnly
    fromColumn: 'Custom SQL Query'.'Submission Week'
    toColumn: LocalDateTable_2e8b88fa-14b2-4993-9f15-905d8ee79573.Date
```

NOTE: All three relationships are date-variation relationships
      (joinOnDateBehavior: datePartOnly) connecting calculated or
      source date columns to their respective LocalDateTable.


### TMPL-04 | en-US.tmdl
─────────────────────────────────────────────────────────────
```
cultureInfo en-US
    linguisticMetadata = ```
            {
              "Version": "1.0.0",
              "Language": "en-US"
            }
            ```
        annotation TopLevelSchema = True
```


### TMPL-05 | diagramLayout.json
─────────────────────────────────────────────────────────────
```json
{
  "version": "1.1",
  "tables": [
    { "id": "Custom SQL Query", "name": "Custom SQL Query",
      "x": 0, "y": 0, "width": 260, "height": 400 }
  ],
  "relationships": []
}
```


### TMPL-06 | LocalDateTable.tmdl (template — one per date column)
─────────────────────────────────────────────────────────────
```
table LocalDateTable_<GUID>
    isHidden
    showAsVariationsOnly
    lineageTag: <GUID>

    column Date
        dataType: dateTime
        isHidden
        formatString: General Date
        lineageTag: ldt<n>-date
        dataCategory: PaddedDateTableDates
        summarizeBy: none
        isNameInferred
        sourceColumn: [Date]
    column Year = YEAR([Date])
        dataType: int64
        isHidden
        formatString: 0
        lineageTag: ldt<n>-year
        dataCategory: Years
        summarizeBy: none
    column MonthNo = MONTH([Date])
        dataType: int64
        isHidden
        formatString: 0
        lineageTag: ldt<n>-monthno
        dataCategory: MonthOfYear
        summarizeBy: none
    column Month = FORMAT([Date], "MMMM")
        dataType: string
        isHidden
        lineageTag: ldt<n>-month
        dataCategory: Months
        summarizeBy: none
        sortByColumn: MonthNo
    column QuarterNo = INT(([MonthNo] + 2) / 3)
        dataType: int64
        isHidden
        formatString: 0
        lineageTag: ldt<n>-qno
        dataCategory: QuarterOfYear
        summarizeBy: none
    column Quarter = "Qtr " & [QuarterNo]
        dataType: string
        isHidden
        lineageTag: ldt<n>-q
        dataCategory: Quarters
        summarizeBy: none
        sortByColumn: QuarterNo
    column Day = DAY([Date])
        dataType: int64
        isHidden
        formatString: 0
        lineageTag: ldt<n>-day
        dataCategory: DayOfMonth
        summarizeBy: none

    hierarchy 'Date Hierarchy'
        lineageTag: ldt<n>-hier
        level Year
            lineageTag: ldt<n>-hier-y
            column: Year
        level Quarter
            lineageTag: ldt<n>-hier-q
            column: Quarter
        level Month
            lineageTag: ldt<n>-hier-m
            column: Month
        level Day
            lineageTag: ldt<n>-hier-d
            column: Day

    partition LocalDateTable_<GUID> = calculated
        mode: import
        source = Calendar(Date(Year(MIN('TableName'[DateCol])),1,1), Date(Year(MAX('TableName'[DateCol])),12,31))

    annotation __PBI_LocalDateTable = true
```

GUID assignment for this model:
  Date col 1 (Submission Week)  → LocalDateTable_2e8b88fa-14b2-4993-9f15-905d8ee79573, prefix ldt1
  Date col 2 (Submission Date)  → LocalDateTable_572388b4-2112-40ad-9630-af36b8afc93e, prefix ldt2
  Date col 3 (Week Start)       → LocalDateTable_37266a76-90b5-4078-942d-75a59988e0d3, prefix ldt3


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — SOURCE COLUMN REFERENCE (Custom SQL Query)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### SCOL-01 | Status
─────────────────────────────────────────────────────────────
PBI Column      : Status
PBI DataType    : string
PBI displayOrd  : 1
PBI lineageTag  : csq-col-status
PBI sourceColumn: status
PBI Hidden      : changedProperty = IsHidden
PBI summarizeBy : none

### SCOL-02 | Submission Date
─────────────────────────────────────────────────────────────
PBI Column      : Submission Date
PBI DataType    : dateTime
PBI FormatString: dd-mmm-yy
PBI displayOrd  : 2
PBI lineageTag  : csq-col-submission-date
PBI sourceColumn: submission_date
PBI Hidden      : changedProperty = IsHidden
PBI summarizeBy : none
Date Variation  : relationship bbe4a354-0f5e-4c9f-8381-bd4fafcd17df →
                  LocalDateTable_572388b4-2112-40ad-9630-af36b8afc93e
⚠️  ANNOTATIONS: UnderlyingDateTimeDataType = Date,
    PBI_FormatHint = {"isDateTimeCustom":true}

NOTE: Remaining source columns (submitter, territory_name,
      territory_name_xl, therapeutic_area, category, topic,
      description, adro_response, adro_comment, region, director,
      roster_idx, idx, first_name, last_name, email, district,
      district_name, territory_code, role_code, director_roster,
      Territory (form)) follow the standard source column pattern:
      dataType: string, sourceColumn: <sql_alias>, lineageTag:
      csq-col-<kebab-name>, summarizeBy: none.
      Full source column list is available in the TMDL table file.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 11 — TABLEAU FEATURES → POWER BI MAPPING
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Tableau Feature        | Power BI Equivalent                        | Handled In       |
|------------------------|--------------------------------------------|------------------|
| Dimension Alias        | SWITCH calculated column                   | TMDL             |
| Categorical Bin        | SWITCH(TRUE(), col IN {...}, ...)           | TMDL             |
| LOD {FIXED}            | CALCULATE + ALLEXCEPT                      | DAX              |
| Parameters             | Disconnected table + SELECTEDVALUE measure  | Parameters.tmdl  |
| Action Filter          | Concatenated string column (isHidden)      | TMDL             |
| Custom Color Palette   | theme.json dataColors array                | theme.json       |
| RLS                    | role + tablePermission TMDL                | roles.tmdl       |
| Drill Path             | hierarchy block                            | TMDL             |
| Tooltip Sheet          | Tooltip report page                        | Report layer only|
| Dashboard              | Report page                                | Report layer only|

For Enterprise Exchange pair_1, the following features are actively used:
  ✅ Action Filter (CC-04, CC-05, CC-06)
  ✅ Dimension concatenation (CC-01 Territory)
  ✅ Date truncation / week start (CC-03, CC-07)
  ❌ Parameters — not present in source workbook
  ❌ LOD {FIXED} — not present in source workbook
  ❌ RLS — not defined
  ❌ Custom Color Palette — not defined


END OF KNOWLEDGE FILE
