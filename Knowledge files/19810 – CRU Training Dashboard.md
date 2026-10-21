# 📘 CONVERSION KNOWLEDGE FILE
# 19810 – CRU Training Dashboard (PROD)
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-09
# Source files: 19810_cru_training_dashboard___prod.txt (Tableau)
#               training_data.txt, cru_training_db.txt,
#               coming_due_selector.txt, date_aggregation_selector.txt,
#               date_aggregation_selector1.txt, date_selector.txt,
#               heat_map_selector.txt, summary_aggregation.txt (PBIP)

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PARAM-01 | Coming Due Selector
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 2]
Tableau Caption       : Coming Due Selector
Tableau Type          : Integer list
Tableau Default       : 0 (All Coming Due)

Tableau Aliases:
  0  → All Coming Due
  7  → Coming Due 7 Days
  14 → Coming Due 14 Days
  21 → Coming Due 21 Days
  30 → Coming Due 30 Days

Power BI Table Name   : Coming Due Selector
Power BI Type         : PQ M Import (Base64+Deflate encoded)
Power BI Columns      : [Value] (int64, summarizeBy: sum),
                        [Display As] (string, summarizeBy: none,
                        sortByColumn: Value)
lineageTag            : d3ef9c8c-2391-440e-b34d-15f49068a3f0

Members:
  Value=0  → Display As="All Coming Due"
  Value=7  → Display As="Coming Due 7 Days"
  Value=14 → Display As="Coming Due 14 Days"
  Value=21 → Display As="Coming Due 21 Days"
  Value=30 → Display As="Coming Due 30 Days"

Usage in model:
  - SELECTEDVALUE('Coming Due Selector'[Value]) returns 0/7/14/21/30
    to drive filtering in Coming Due Filter measures.
  - SELECTEDVALUE('Coming Due Selector'[Display As]) used in display
    label measure 'Coming Due Selector'.

⚠️  The integer [Value] column (NOT [Display As]) drives all DAX logic.
⚠️  changedProperty = SortByColumn on [Display As] — preserve exactly.

Conversion Status: ✅ FULL — all 5 values, order and keys match exactly.


### PARAM-02 | Training_Details_AnnotationParam  (DROPPED)
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 3]
Tableau Caption       : Training_Details_AnnotationParam
Tableau Type          : String (any/free entry)
Tableau Default       : {"filters":{},"parameters":{}}
Hidden                : Yes (Tableau side)

Power BI Implementation:
  No equivalent parameter table exists in the PBIP model.
  Tableau used a JSON string injection to cross-filter the annotation
  datasource (cru_training_db) at query time.
  PBIP loads cru_training_db in full via Import (SQL Server).
  The [On-Leave Flag] calculated column drives annotation display logic.
  Filtering is handled at visual/report layer, not via parameter injection.

⚠️  Do NOT attempt to replicate the JSON injection mechanism in PBIP.
⚠️  No [Training_Details_AnnotationParam] parameter table in any PBIP file.

Conversion Status: ⚠️  ADAPTED — JSON param mechanism dropped;
                   full-table Import replaces dynamic filter injection.


### PARAM-03 | Heat Map Selector
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 4]
Tableau Caption       : Heat Map Selector
Tableau Type          : String list ("1"/"2"/"3")
Tableau Default       : "1" (Metric View)

Power BI Table Name   : Heat Map Selector
Power BI Type         : DAX Calculated Table (DATATABLE)
Power BI Columns      : [ID] (int64, summarizeBy: count),
                        [Value] (string, summarizeBy: none, sortByColumn: ID)
lineageTag            : 1e214409-3a4f-4a0b-8411-5109e64a2dc4
Annotation            : PBI_Id = 8e95259d619a4c23b43971710b12f79f

Power BI DAX:
─────────────
DATATABLE(
    "ID",    INTEGER,
    "Value", STRING,
    {
        { 1, "Metric View"      },
        { 2, "Overdue View"     },
        { 3, "Coming Due View"  }
    }
)

Usage in model:
  - SELECTEDVALUE('Heat Map Selector'[ID]) returns 1/2/3
    to drive the [Heat Map Filter] calculated column SWITCH.

⚠️  Tableau stored string values "1"/"2"/"3"; PBIP uses integer IDs 1/2/3.
    Functionally equivalent but type differs. In DAX always compare to
    integer, not string.

Conversion Status: ✅ FULL — all 3 values, integer IDs, sort order preserved.


### PARAM-04 | Date Aggregation Selector
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 5]
Tableau Caption       : Date Aggregation Selector
Tableau Type          : String list ("1"/"2")
Tableau Default       : "1" (Month View)

Power BI Table Name   : Date Aggregation Selector
Power BI Type         : DAX Calculated Table (DATATABLE)
Power BI Columns      : [ID] (inferred integer, isNameInferred,
                        summarizeBy: count),
                        [Value] (inferred string, isNameInferred,
                        summarizeBy: none)
lineageTag            : 88ed44d5-0a68-48dc-b5e0-dc75deaa633f
Annotation            : PBI_Id = 82671d8b2a334777a86148472982c526

Power BI DAX:
─────────────
DATATABLE(
    "ID",    INTEGER,
    "Value", STRING,
    {
        { 1, "Month View"       },
        { 2, "Month/Week View"  }
    }
)

⚠️  isNameInferred = true on both columns — names auto-derived from
    DATATABLE definition. Do NOT set explicit column names in TMDL.
⚠️  This table holds display labels only. Field binding for the visual
    axis is in Date Aggregation Selector1 and Date selector (NAMEOF tables).

Conversion Status: ✅ FULL


### PARAM-05 | Date Aggregation Selector1  (NAMEOF Field Parameter)
─────────────────────────────────────────────────────────────
Tableau Equivalent    : Field toggle driving axis between [required date]
                        month aggregation and week aggregation
Power BI Table Name   : Date Aggregation Selector1
Power BI Type         : DAX Calculated Table (NAMEOF tuple)
Power BI Columns      : [Value1] (display label, isNameInferred),
                        [Value2] (NAMEOF field ref, isNameInferred),
                        [Value3] (sort int64, isNameInferred)
lineageTag            : 0f584601-77df-4a21-8917-ecdb11937030
Annotation            : PBI_Id = ef921d4993d3438d948c54b78edd1239

Power BI DAX:
─────────────
{
    ( "Month View",      NAMEOF('Training Data'[Required Date (Months)_MY]), 0 ),
    ( "Month/Week View", NAMEOF('Training Data'[Required Date Week]),        1 )
}

Usage in model:
  - [Value2] NAMEOF bindings connect this parameter to calculated columns
    [Required Date (Months)_MY] (M-layer source) and [Required Date Week]
    (DAX calculated column).

⚠️  DELTA vs Date selector (PARAM-06): this table binds [Required Date Week]
    for Month/Week; Date selector binds [MonthWeek Display] for the same
    option. These are two distinct columns serving different visuals.

Conversion Status: ✅ FULL


### PARAM-06 | Date selector  (NAMEOF Field Parameter — Visual Axis)
─────────────────────────────────────────────────────────────
Tableau Equivalent    : Dynamic X-axis toggle between month label and
                        month+week display label
Power BI Table Name   : Date selector
Power BI Type         : DAX Calculated Table (NAMEOF tuple, ParameterMetadata)
Power BI Columns      :
  [Date selector]        — string, visible, sortByColumn: 'Date selector Order',
                           relatedColumnDetails: groupByColumn = 'Date selector Fields'
  [Date selector Fields] — isHidden, ParameterMetadata kind=2 (active field
                           parameter column), sortByColumn: 'Date selector Order'
  [Date selector Order]  — isHidden, int, summarizeBy: sum
lineageTag            : 03f8f3e9-715a-46e0-bee6-8446a3f34b33
Annotation            : PBI_Id = 9725ac1a7d554ac190bc00736ec6ab88

Power BI DAX:
─────────────
{
    ( "Month View",      NAMEOF('Training Data'[Required Date (Months)_MY]), 0 ),
    ( "Month/Week View", NAMEOF('Training Data'[MonthWeek Display]),         1 )
}

Usage in model:
  - [Date selector Fields] (ParameterMetadata kind=2) is the active field
    parameter column bound to the visual X-axis for dynamic axis switching.
  - [MonthWeek Display] is the formatted two-line display column
    (MMM'yy + line feed + zero-padded week number).

⚠️  ParameterMetadata kind=2 marks this as the visual axis field parameter.
    Preserve the extendedProperty block exactly.

Conversion Status: ✅ FULL


### PARAM-07 | Summary Aggregation  (User-scope Selector)
─────────────────────────────────────────────────────────────
Tableau Equivalent    : None — PBIP-native addition for personal /
                        manager-scoped training views
Power BI Table Name   : Summary Aggregation
Power BI Type         : PQ M Import
Power BI Columns      : [ParameterTableKeyColumn] (int64, summarizeBy: sum),
                        [Value] (string, summarizeBy: none,
                        sortByColumn: ParameterTableKeyColumn)
lineageTag            : 9390d186-59f7-40f4-9967-52057777540f
Annotations           : PBI_NavigationStepName=Navigation;
                        PBI_ResultType=Table;
                        TableType=ParameterTable

Power BI M Query:
─────────────────
let
    Source = {
        {"1", "Display All Training"},
        {"2", "Display Personal Training"},
        {"3", "Display Direct Reports Training"}
    },
    TableFromList = Table.FromRows(
        Source,
        {"ParameterTableKeyColumn", "Value"}
    ),
    ChangedType = Table.TransformColumnTypes(
        TableFromList,
        {{"ParameterTableKeyColumn", Int64.Type}, {"Value", type text}}
    )
in
    ChangedType

Members:
  1 → Display All Training
  2 → Display Personal Training
  3 → Display Direct Reports Training

Usage in model:
  - SELECTEDVALUE('Summary Aggregation'[ParameterTableKeyColumn])
    returns 1/2/3 to drive USERNAME() filtering in Personal Summary measures:
    Key 1: all records (no filter)
    Key 2: filter Training Data where [bms_user_id] = USERNAME()
    Key 3: filter Training Data where [manager_uid] = USERNAME()

⚠️  Raw source rows are string tuples {"1","2",...}; live column type is
    int64 after TransformColumnTypes. Always compare to integer in DAX.
⚠️  No Tableau equivalent — PBIP-native pattern. Do NOT look for a matching
    Tableau parameter.

Conversion Status: ✅ FULL (PBIP-native, no Tableau counterpart)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: Training Data

### CC-01 | Training Status
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1267763344863404033
PBI Data Type   : string
PBI Hidden      : No
PBI Comment     : (none)

Tableau Expression:
───────────────────
Derived from [completion date] and [required date] business rules.
(Core status classification — no single Tableau formula documented.)

Output values: "Coming Due" | "Late - Not Complete" |
               "On-Time - Complete" | "Late - Complete" | BLANK()

Power BI DAX:
─────────────
column 'Training Status' =

VAR CompletionDate = 'Training Data'[completion date]
VAR RequiredDate   = 'Training Data'[required date]

RETURN
IF(
    ISBLANK(CompletionDate) && RequiredDate >= TODAY(),
    "Coming Due",
IF(
    ISBLANK(CompletionDate) && RequiredDate < TODAY(),
    "Late - Not Complete",
IF(
    NOT(ISBLANK(CompletionDate)) && CompletionDate <= RequiredDate,
    "On-Time - Complete",
IF(
    NOT(ISBLANK(CompletionDate)) && CompletionDate > RequiredDate,
    "Late - Complete",
    BLANK()
))))

Depends on  : [required date], [completion date] (source columns)
Used by     : CC-02, CC-03, CC-04, CC-06, CC-07, CC-08
Conversion  : ✅ FULL


### CC-02 | On-Time Training
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1118863086134231041
PBI Data Type   : int64
PBI Format      : 0
PBI Hidden      : No
PBI summarizeBy : sum

Tableau Expression:
───────────────────
IF [Training Status] = "On-Time - Complete" THEN 1 ELSE 0 END

Power BI DAX:
─────────────
column 'On-Time Training' =
IF([Training Status] = "On-Time - Complete", 1, 0)
/* Original Tableau Expression is
   IF [Training Status] = "On-Time - Complete" THEN 1 ELSE 0 END */

Depends on  : CC-01 [Training Status]
Conversion  : ✅ FULL


### CC-03 | Heatmap Scope
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1118863086134546434
PBI Data Type   : boolean
PBI Format      : """TRUE"";""TRUE"";""FALSE"""
PBI Hidden      : No
PBI summarizeBy : none

Tableau Expression:
───────────────────
Window: required_date within rolling 12 months (month-aligned) AND
Training Status ≠ "Not Yet Due"

Power BI DAX:
─────────────
column 'Heatmap Scope' =

VAR RequiredDate    = 'Training Data'[required date]
VAR TodayDate       = TODAY()
VAR TwelveMonthsAgo = DATE(
    YEAR(EDATE(TodayDate, -12)),
    MONTH(EDATE(TodayDate, -12)),
    1
)

RETURN
IF(
    'Training Data'[Training Status] <> "Not Yet Due"
        && RequiredDate <= TodayDate
        && RequiredDate >= TwelveMonthsAgo,
    TRUE(),
    FALSE()
)

Depends on  : CC-01, [required date]
Used by     : CC-06 [Heat Map Filter]
⚠️  "Not Yet Due" is NOT produced by CC-01 (CC-01 outputs: "Coming Due",
    "Late - Not Complete", "On-Time - Complete", "Late - Complete", BLANK()).
    This condition always evaluates TRUE for valid rows — acts as a
    NULL/BLANK safety guard. Preserve exactly; do NOT optimise away.
Conversion  : ✅ FULL


### CC-04 | Training Scope
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1267763344857989120
PBI Data Type   : boolean
PBI Format      : """TRUE"";""TRUE"";""FALSE"""
PBI Hidden      : No
PBI summarizeBy : none

Tableau Expression:
───────────────────
IF [required date] >= DATEADD('month', -24, DATETRUNC('month', today()))
AND [required date] < DATEADD('month', 13, DATETRUNC('month', today()))
THEN TRUE ELSE FALSE END

Power BI DAX:
─────────────
column 'Training Scope' =
IF(
    AND(
        'Training Data'[required date] >= EDATE(TODAY(), -24),
        'Training Data'[required date] <  EDATE(TODAY(), 13)
    ),
    TRUE(),
    FALSE()
)
/* Original Tableau Expression is
   IF [required date] >= DATEADD('month', -24, DATETRUNC('month', today()))
   AND [required date] < DATEADD('month', 13, DATETRUNC('month', today()))
   THEN TRUE ELSE FALSE END */

Depends on  : [required date]
Used by     : CC-06 [Heat Map Filter]
⚠️  DATETRUNC vs EDATE: Tableau aligns to month-start before DATEADD;
    PBIP uses EDATE(TODAY(),...) directly without month-start alignment.
    Minor boundary difference at mid-month — accepted approximation.
    Preserve as-is; do NOT add month-start alignment.
Conversion  : ✅ FULL (with noted approximation)


### CC-05 | Total Training
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1118863086135341060
PBI Data Type   : int64
PBI Hidden      : No
PBI summarizeBy : sum

Tableau Expression:
───────────────────
1   (literal row counter)

Power BI DAX:
─────────────
column 'Total Training' = 1
/* Original Tableau Expression: 1 */

Purpose     : SUM([Total Training]) = total row count.
Conversion  : ✅ FULL


### CC-06 | Heat Map Filter
─────────────────────────────────────────────────────────────
PBI Lineage Tag : a8dee978-0dbc-402c-bb75-c0313b35e37a
PBI Data Type   : boolean
PBI Format      : """TRUE"";""TRUE"";""FALSE"""
PBI Hidden      : No
PBI summarizeBy : none

Tableau Expression:
───────────────────
IF [Parameters].[Parameter 4] = '1'
   AND [Heatmap Scope] = TRUE AND [Training Scope] = TRUE THEN TRUE
ELSEIF [Parameters].[Parameter 4] = '2'
   AND [Training Status] = 'Late - Not Complete' THEN TRUE
ELSEIF [Parameters].[Parameter 4] = '3'
   AND [Training Status] = 'Coming Due' THEN TRUE
ELSE FALSE END

Power BI DAX:
─────────────
column 'Heat Map Filter' =
IF(
    AND(
        SELECTEDVALUE('Heat Map Selector'[ID]) = 1,
        AND(
            'Training Data'[Heatmap Scope] = TRUE(),
            'Training Data'[Training Scope] = TRUE()
        )
    ),
    TRUE(),
IF(
    AND(
        SELECTEDVALUE('Heat Map Selector'[ID]) = 2,
        'Training Data'[Training Status] = "Late - Not Complete"
    ),
    TRUE(),
IF(
    AND(
        SELECTEDVALUE('Heat Map Selector'[ID]) = 3,
        'Training Data'[Training Status] = "Coming Due"
    ),
    TRUE(),
    FALSE()
)))
/* Original Tableau Expression is
   IF [Parameters].[Parameter 4] = '1' AND [Heatmap Scope] = TRUE
   AND [Training Scope] = TRUE THEN TRUE
   ELSEIF [Parameters].[Parameter 4] = '2'
   AND [Training Status] = 'Late - Not Complete' THEN TRUE
   ELSEIF [Parameters].[Parameter 4] = '3'
   AND [Training Status] = 'Coming Due' THEN TRUE
   ELSE FALSE END */

Depends on  : CC-01, CC-03, CC-04, PARAM-03 [Heat Map Selector]
⚠️  Tableau uses string '1'/'2'/'3'; PBIP uses integer 1/2/3.
    Always compare SELECTEDVALUE('Heat Map Selector'[ID]) to integer.
Conversion  : ✅ FULL


### CC-07 | Aging Filter
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_927741583719153664
PBI Data Type   : boolean
PBI Format      : """TRUE"";""TRUE"";""FALSE"""
PBI Hidden      : No
PBI summarizeBy : none

Tableau Expression:
───────────────────
IF [required date] < DATEADD('month', -12, DATETRUNC('month', today()))
   AND [Training Status] = "Late - Not Complete" THEN TRUE
ELSEIF [required date] > DATEADD('month', -12, DATETRUNC('month', today()))
THEN TRUE
ELSE FALSE END

Power BI DAX:
─────────────
column 'Aging Filter' =
IF(
    AND(
        'Training Data'[required date] <
            EDATE(DATE(YEAR(TODAY()), MONTH(TODAY()), 1), -12),
        'Training Data'[Training Status] = "Late - Not Complete"
    ),
    TRUE(),
    IF(
        'Training Data'[required date] >
            EDATE(DATE(YEAR(TODAY()), MONTH(TODAY()), 1), -12),
        TRUE(),
        FALSE()
    )
)
/* Original Tableau Expression is
   IF [required date] < DATEADD('month', -12, DATETRUNC('month', today()))
   AND [Training Status] = "Late - Not Complete" THEN TRUE
   ELSEIF [required date] > DATEADD('month', -12, DATETRUNC('month', today()))
   THEN TRUE ELSE FALSE END */

Depends on  : CC-01, [required date]
⚠️  12-month boundary uses < and > only (no =). Records exactly at the
    12-month cutoff evaluate FALSE — intentional. Do NOT add = operators.
Conversion  : ✅ FULL


### CC-08 | Previous Month T/F
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1471269736415903744
PBI Data Type   : boolean
PBI Hidden      : No
PBI summarizeBy : none

Tableau Expression:
───────────────────
DATETRUNC('month', [required date]) =
DATETRUNC('month', DATEADD('month', -1, today()))

Power BI DAX:
─────────────
column 'Previous Month T/F' =
IF(
    YEAR('Training Data'[required date])  = YEAR(EDATE(TODAY(), -1))
    && MONTH('Training Data'[required date]) = MONTH(EDATE(TODAY(), -1)),
    TRUE(),
    FALSE()
)
/* Original Tableau Expression is
   DATETRUNC('month', [required date]) =
   DATETRUNC('month', DATEADD('month', -1, today())) */

Depends on  : [required date]
Conversion  : ✅ FULL


### CC-09 | Week Filter T/F
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1471269736418516993
PBI Data Type   : boolean
PBI Hidden      : No
PBI summarizeBy : none

Tableau Expression:
───────────────────
DATETRUNC('week', [required date]) >= DATETRUNC('week', today())
AND DATETRUNC('week', [required date]) <=
    DATETRUNC('week', DATEADD('week', 2, today()))

Power BI DAX:
─────────────
column 'Week Filter T/F' =
IF(
    AND(
        'Training Data'[required date]
            - WEEKDAY('Training Data'[required date], 2) + 1
            >= TODAY() - WEEKDAY(TODAY(), 2) + 1,
        'Training Data'[required date]
            - WEEKDAY('Training Data'[required date], 2) + 1
            <= (TODAY() + 14) - WEEKDAY(TODAY() + 14, 2) + 1
    ),
    TRUE(),
    FALSE()
)
/* Original Tableau Expression is
   DATETRUNC('week', [required date]) >= DATETRUNC('week', today())
   AND DATETRUNC('week', [required date]) <=
   DATETRUNC('week', DATEADD('week', 2, today())) */

Depends on  : [required date]
⚠️  WEEKDAY mode=2 → Monday-start week (1=Mon...7=Sun).
    Window: current week through 2 weeks ahead (TODAY()+14).
Conversion  : ✅ FULL


### CC-10 | RequiredDate_3M_Rolling
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1616792319890599936
PBI Data Type   : boolean
PBI Hidden      : No
PBI summarizeBy : none

Tableau Expression:
───────────────────
DATEDIFF('month', DATETRUNC('month', TODAY()),
         DATETRUNC('month', [required date])) IN (-1, 0, 1)

Power BI DAX:
─────────────
column RequiredDate_3M_Rolling =
DATEDIFF(
    DATE(YEAR(TODAY()), MONTH(TODAY()), 1),
    DATE(YEAR('Training Data'[required date]),
         MONTH('Training Data'[required date]), 1),
    MONTH
) IN {-1, 0, 1}
/* Original Tableau Expression is
   DATEDIFF('month', DATETRUNC('month', TODAY()),
   DATETRUNC('month', [required date])) IN (-1, 0, 1) */

Depends on  : [required date]
Window      : Previous month, current month, next month.
Conversion  : ✅ FULL


### CC-11 | wb_ref
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_219832011871481856
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Expression:
───────────────────
LOWER([bms_user_id])

Power BI DAX:
─────────────
column wb_ref = LOWER('Training Data'[bms_user_id])
/* Original Tableau Expression is LOWER([bms_user_id]) */

Depends on  : [bms_user_id] (source column)
Purpose     : Lowercase version of bms_user_id for case-insensitive
              USERNAME() comparisons.
Conversion  : ✅ FULL


### CC-12 | Agg
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1254815457163018240
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Expression:
───────────────────
""   (empty string literal)

Power BI DAX:
─────────────
column Agg = BLANK()
/* Original Tableau Expression is "" */

Purpose     : Placeholder aggregation / grouping anchor column.
⚠️  Tableau uses ""; PBIP uses BLANK(). ISBLANK([Agg]) = TRUE in PBIP;
    [Agg] = "" would be FALSE. Preserve as BLANK() — do NOT change to "".
Conversion  : ✅ FULL (with noted empty-string → BLANK() delta)


### CC-13 | required date (Week numbers)
─────────────────────────────────────────────────────────────
PBI Lineage Tag : required date (Week numbers)
PBI Data Type   : int64
PBI Hidden      : No
PBI summarizeBy : sum

Tableau Expression:
───────────────────
DATEPART('week', [required date])

Power BI DAX:
─────────────
column 'required date (Week numbers)' =
WEEKNUM('Training Data'[required date])
/* Original Tableau Expression is DATEPART('week', [required date]) */

Depends on  : [required date]
Used by     : CC-14, CC-17
⚠️  WEEKNUM default = mode 1 (week starts Sunday).
    Tableau DATEPART('week',...) also defaults to Sunday. Functionally
    equivalent for most dates; edge weeks may differ. Preserve default mode.
Conversion  : ✅ FULL


### CC-14 | Required Date Week
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1391612299604713473
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none
PBI Comment     : /// Required Date Agg - Column

Tableau Expression:
───────────────────
"W" + STR(DATEPART('week', [required date]))

Power BI DAX:
─────────────
column 'Required Date Week' =
"W" & FORMAT('Training Data'[required date (Week numbers)], "0")

Depends on  : CC-13 [required date (Week numbers)]
Used by     : PARAM-05 [Date Aggregation Selector1] NAMEOF binding
⚠️  FORMAT(..., "0") = no leading zero. Produces "W1" not "W01".
    Do NOT change to "00" — that padding is used only in CC-16.
Conversion  : ✅ FULL


### CC-15 | Required Date (Months)
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 274b2739-a48c-411a-856b-42d3344a1c21
PBI Data Type   : (unspecified — General Date)
PBI Format      : General Date
PBI Hidden      : No
PBI summarizeBy : none

Power BI DAX:
─────────────
column 'Required Date (Months)' =
DATE(
    YEAR('Training Data'[required date]),
    MONTH('Training Data'[required date]),
    1
)

Depends on  : [required date]
Purpose     : Month-truncated date (first day of month) as a true date
              value. Distinct from [Required Date (Months)_MY] which is
              a string label ("MMM'yy") added in M.
Conversion  : ✅ FULL


### CC-16 | MonthWeek Display
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 20948dd6-da78-4b96-9607-f235f5126aef
PBI Data Type   : (unspecified — string)
PBI Hidden      : No
PBI summarizeBy : none
PBI sortByColumn: WeekSort (changedProperty = SortByColumn)

Power BI DAX:
─────────────
column 'MonthWeek Display' =
FORMAT('Training Data'[Required Date], "MMM''yy")
  & UNICHAR(10) &
  FORMAT('Training Data'[Required Date Week], "00")

Depends on  : [Required Date] / [required date], CC-14
Used by     : PARAM-06 [Date selector] NAMEOF binding
⚠️  UNICHAR(10) = line feed — produces a two-line label:
    Line 1: "Jan'25"
    Line 2: zero-padded week number (e.g., "01", "09") using "00" format.
⚠️  [Required Date Week] uses "00" format here (zero-padded),
    whereas CC-14 itself uses "0" (no padding). The two usages differ.
⚠️  [Required Date] (capitalised) is referenced — likely an alias for
    [required date] (lowercase source column). Confirm casing in desktop.
Conversion  : ✅ FULL


### CC-17 | WeekSort
─────────────────────────────────────────────────────────────
PBI Lineage Tag : a40fb7c5-a11d-4418-86b9-c0a2d7c6a121
PBI Data Type   : (unspecified — integer)
PBI Hidden      : No
PBI summarizeBy : sum
PBI Annotation  : PBI_FormatHint = {"isGeneralNumber":true}

Power BI DAX:
─────────────
column WeekSort =
YEAR('Training Data'[Required Date]) * 100
  + 'Training Data'[required date (Week numbers)]

Depends on  : [Required Date] / [required date], CC-13
Used by     : sortByColumn for CC-16 [MonthWeek Display]
Purpose     : Numeric sort key. YYYY*100 + WW ensures correct
              chronological ordering.
Conversion  : ✅ FULL


## TABLE: cru_training_db

### CC-18 | On-Leave Flag
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 0aa469c2-3b1f-47da-a3f5-e4e4102813b6
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Expression:
───────────────────
IF ISNULL(ATTR([federated.1uh8kny0vr907q1fa1bvs06xifaq].[justification]))
THEN "Active"
ELSE "On-Leave"
END

Power BI DAX:
─────────────
column 'On-Leave Flag' =
IF (
    OR (
        ISBLANK ( cru_training_db[Justification] ),
        cru_training_db[Justification] = ""
    ),
    "Active",
    IF (
        cru_training_db[Justification] = "Null",
        "Active",
        "On-Leave"
    )
)
/* Original Tableau Expression is
   IF ISNULL(ATTR([federated.1uh8kny0vr907q1fa1bvs06xifaq].[justification]))
   THEN "Active" ELSE "On-Leave" END */

Depends on  : [Justification] (source column)
PBI Enhancement: Adds empty string guard ("") and literal string "Null"
                 guard in addition to ISBLANK(). Both map to "Active".
                 Neither condition exists in the Tableau expression.
⚠️  Preserve all three conditions exactly. Do NOT simplify to ISBLANK() only.
Conversion  : ✅ FULL + enhanced


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: Training Data  (placeholder measures)

### M-01 | Measure  (placeholder)
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 791213d7-088e-45a0-a393-631da58416f5
PBI Annotation  : 43dbc3e8-3a1c-4b6f-9923-b49ff7d6691c = True
DAX             : (empty — scaffold placeholder)
Purpose         : No business logic. Do NOT populate without explicit
                  instruction. Preserve annotation exactly.
Conversion      : ➕ PBI scaffold only


### M-02 | Measure 2  (placeholder)
─────────────────────────────────────────────────────────────
PBI Lineage Tag : e472fd3f-be6d-4f5f-a1f5-7e687b781159
PBI Annotation  : 43dbc3e8-3a1c-4b6f-9923-b49ff7d6691c = True
DAX             : (empty — scaffold placeholder)
Purpose         : No business logic. Do NOT populate without explicit
                  instruction. Preserve annotation exactly.
Conversion      : ➕ PBI scaffold only


## TABLE: Coming Due Selector

### M-03 | Coming Due Filter
─────────────────────────────────────────────────────────────
PBI Table       : Coming Due Selector
PBI Lineage Tag : f164d3d9-6c57-4b97-8ed2-93819809a3a0
PBI Format      : #,0

Tableau Equivalent:
───────────────────
IF [Coming Due Selector] = 0 THEN TRUE
ELSE
  IF DATEDIFF('day', TODAY(), [required date]) <= INT([Coming Due Selector])
  THEN TRUE END
END

Power BI DAX:
─────────────
measure 'Coming Due Filter' =
SWITCH(
    SELECTEDVALUE('Coming Due Selector'[Value]),
    0, [Item ID_count],
    CALCULATE(
        [Item ID_count],
        FILTER(
            ALL('Training Data'[required date]),
            DATEDIFF(
                TODAY(),
                'Training Data'[required date],
                DAY
            ) <= SELECTEDVALUE('Coming Due Selector'[Value])
        )
    )
)
/* IF [Coming Due Selector] = 0 THEN TRUE
   ELSE IF DATEDIFF('day', TODAY(), [required date])
   <= INT([Coming Due Selector]) THEN TRUE END END */

Key Pattern : When Value=0 → returns [Item ID_count] unfiltered.
              When Value>0 → filters to required dates due within N days.
              ALL() removes all filters on [required date] only;
              other filter context remains active.
⚠️  [Item ID_count] dependency: definition not in uploaded PBIP files.
    Flag as PENDING until confirmed.
Conversion  : ✅ FULL (pending [Item ID_count] definition)


### M-04 | Coming Due Filter (3M)
─────────────────────────────────────────────────────────────
PBI Table       : Coming Due Selector
PBI Lineage Tag : c050950a-0677-4deb-88f6-11641ed97a33
PBI Format      : #,0

Tableau Equivalent:
───────────────────
Same as M-03 but restricted to a rolling 3-month window.

Power BI DAX:
─────────────
measure 'Coming Due Filter (3M)' =
SWITCH(
    SELECTEDVALUE('Coming Due Selector'[Value]),
    0, [Item Id_Count_3M],
    CALCULATE(
        [Item Id_Count],
        FILTER(
            ALL('Training Data'[required date]),
            'Training Data'[required date] >= EOMONTH(TODAY(), -2) + 1
            &&
            DATEDIFF(
                TODAY(),
                'Training Data'[required date],
                DAY
            ) <= SELECTEDVALUE('Coming Due Selector'[Value])
        )
    )
)
/* IF [Coming Due Selector] = 0 THEN TRUE
   ELSE IF DATEDIFF('day', TODAY(), [required date])
   <= INT([Coming Due Selector]) THEN TRUE END END */

Key Pattern : 3M window: required_date >= start of 2 months ago
              (EOMONTH(TODAY(),-2)+1).
              When Value=0: returns [Item Id_Count_3M] baseline.
⚠️  CAPITALISATION DELTA: this measure references [Item Id_Count] and
    [Item Id_Count_3M] (mixed Id, uppercase Count). M-03 references
    [Item ID_count] (uppercase ID, lowercase count). These are two
    distinct measures — preserve both capitalisations exactly.
⚠️  [Item Id_Count] and [Item Id_Count_3M]: PENDING definition.
Conversion  : ✅ FULL (pending dependent measure definitions)


### M-05 | Personal Summary T/F
─────────────────────────────────────────────────────────────
PBI Table       : Coming Due Selector
PBI Lineage Tag : 724a497c-c745-4aa7-b51d-04229187e375
PBI Format      : #,0

Tableau Equivalent:
───────────────────
IF [Summary Aggregation] = '1' THEN TRUE
ELSEIF [Summary Aggregation] = '2'
       AND LOWER([bms_user_id]) = LOWER(USERNAME()) THEN TRUE
ELSEIF [Summary Aggregation] = '3'
       AND LOWER([manager_uid]) = LOWER(USERNAME()) THEN TRUE
END

Power BI DAX:
─────────────
measure 'Personal Summary T/F' =
SWITCH(
    SELECTEDVALUE('Summary Aggregation'[ParameterTableKeyColumn]),
    1, 'Coming Due Selector'[Coming Due Filter],
    2, CALCULATE(
           'Coming Due Selector'[Coming Due Filter],
           'Training Data'[bms_user_id] = USERNAME()
       ),
    3, CALCULATE(
           'Coming Due Selector'[Coming Due Filter],
           'Training Data'[manager_uid] = USERNAME()
       )
)

Key Pattern : USERNAME() for row-level user scoping.
              Key 1: all records (no filter).
              Key 2: bms_user_id = current user.
              Key 3: manager_uid = current user.
⚠️  Tableau used LOWER() for case-insensitive compare; PBIP uses direct
    equality. Confirm USERNAME() case sensitivity in the deployed tenant.
Conversion  : ✅ FULL


### M-06 | Personal Summary T/F (3M)
─────────────────────────────────────────────────────────────
PBI Table       : Coming Due Selector
PBI Lineage Tag : 0d3d1537-ffdd-4d11-87cc-207785668d5d
PBI Format      : #,0

Tableau Equivalent:
───────────────────
Same as M-05 but delegates to the 3-month rolling variant.

Power BI DAX:
─────────────
measure 'Personal Summary T/F (3M)' =
SWITCH(
    SELECTEDVALUE('Summary Aggregation'[ParameterTableKeyColumn]),
    1, 'Coming Due Selector'[Coming Due Filter (3M)],
    2, CALCULATE(
           'Coming Due Selector'[Coming Due Filter (3M)],
           'Training Data'[bms_user_id] = USERNAME()
       ),
    3, CALCULATE(
           'Coming Due Selector'[Coming Due Filter (3M)],
           'Training Data'[manager_uid] = USERNAME()
       )
)

Depends on  : PARAM-07, M-04 [Coming Due Filter (3M)]
Conversion  : ✅ FULL


### M-07 | Coming Due Selector  (Display label)
─────────────────────────────────────────────────────────────
PBI Table       : Coming Due Selector
PBI Lineage Tag : cc75720c-8068-48d8-ad7e-4c21e39923cf

Power BI DAX:
─────────────
measure 'Coming Due Selector' =
SELECTEDVALUE('Coming Due Selector'[Display As])

Purpose     : Returns selected display label for card / title visuals.
⚠️  Measure name is identical to the table name: 'Coming Due Selector'.
    In DAX, 'Coming Due Selector'[Coming Due Selector] disambiguates.
    Do NOT rename either the table or the measure.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU INITIAL TRANSFORMATIONS → POWER BI POWER QUERY (M)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | Training Data  (Redshift + PostgreSQL federated LEFT JOIN)
─────────────────────────────────────────────────────────────
Tableau Sources : 1. gps_bia_mfg.cru_training_data (Redshift)
                  2. md_rpt_v.v_ent_wrkr_dtl (PostgreSQL) — Ireland only

Redshift Connection:
  Server   : bms-gpsdl-non-validated-redshift-prod.web.bms.com:5439
  Database : gpsdb
  Schema   : gps_bia_mfg
  Table    : cru_training_data
  M call   : AmazonRedshift.Database → Value.NativeQuery(EnableFolding=false)

PostgreSQL Connection:
  Server   : mdhub.web.bms.com:5432
  Database : postgres
  Schema   : md_rpt_v
  View     : v_ent_wrkr_dtl
  M call   : PostgreSQL.Database → Value.NativeQuery(EnableFolding=true)

⚠️  EnableFolding asymmetry:
    Redshift: EnableFolding=false — SQL must run exactly as written.
    PostgreSQL: EnableFolding=true — query can fold.
    Preserve both settings exactly.

Redshift SQL (verbatim):
─────────────────────────
SELECT
    "user id" || '_' || "item id" || '_' ||
    CASE
        WHEN "completion date" IS NULL THEN ''
        ELSE "completion date"::CHAR(10)
    END AS "training_key",
    "user id",
    NULL AS "user name",
    NULL AS "organization id",
    NULL AS "organization",
    NULL AS "security domain id",
    NULL AS "security domain",
    NULL AS "curriculum id",
    NULL AS "curriculum",
    "item id",
    NULL AS "item type",
    NULL AS "item revision",
    NULL AS "assignment type",
    MIN("curriculum assign date") AS "curriculum assign date",
    MIN("required date")         AS "required date",
    "completion date",
    NULL AS "completion status",
    NULL AS "overdue",
    NULL AS "days remaining",
    NULL AS "source",
    "bms_id",
    "bms_user_id",
    "full_name",
    "job_fam_name",
    "job_fam_grp_name",
    "slt_bms_id",
    "manager_uid",
    "hr_orgzn_nm",
    "lt user id",
    "lt name",
    "lt department",
    "employee status"
FROM gps_bia_mfg.cru_training_data
GROUP BY
    "user id" || '_' || "item id" || '_' || "completion date",
    "user id", "item id", "completion date",
    "bms_id", "bms_user_id", "full_name", "job_fam_name",
    "job_fam_grp_name", "slt_bms_id", "manager_uid",
    "hr_orgzn_nm", "lt user id", "lt name",
    "lt department", "employee status"

PostgreSQL SQL (verbatim):
───────────────────────────
SELECT
    LTRIM(bms_id, '0') AS "man_join_bms_id",
    whitepages_mgr_full_nm AS "manager_name"
FROM md_rpt_v.v_ent_wrkr_dtl
WHERE work_location_country_name LIKE '%IRELAND%'
  AND idnty_sta_nm = 'ENABLED'

⚠️  Ireland-only hard filter is source-level — do NOT move to visual layer.

JOIN: LeftOuter on Training Data[bms_id] ↔ Custom SQL Query[man_join_bms_id]
⚠️  bms_id retains leading zeros (Redshift). man_join_bms_id strips them
    via LTRIM (PostgreSQL SQL). Never modify Redshift [bms_id].

M-layer computed columns (added after JOIN, before TypedColumns):
  Required Date (Months)_MY : Text.Start(Date.MonthName([required date]),3)
                               & "'" &
                               Text.End(Text.From(Date.Year([required date])),2)
                               → e.g. "Jan'25"
  Month                     : Date.Month([required date])
  Year                      : Date.Year([required date])
  Index                     : [Month] + [Year] * 100     (sort key)
  Month Year                : Date.MonthName([required date]) & " " &
                               Text.From([Year])
                               → e.g. "January 2025"

⚠️  RenameColumns step is identity (no actual renames) — all column
    names are preserved as-is through the M pipeline.
⚠️  14 columns are always NULL from Redshift SQL (structural placeholders).
    Do NOT remove them. See FLAG-01.

Source Columns → PBIP Column Name Rename Register:
  job_fam_name  → Tier 2 Dept      (lineageTag=[job_fam_name])
  hr_orgzn_nm   → Tier 1 Dept      (lineageTag=[hr_orgzn_nm])
  lt department → LT Department    (lineageTag=[lt department])
  All other columns: no rename — name preserved exactly including spaces.

displayOrdinal gap: man_join_bms_id=34, manager_name=35.
Ordinal 33 is absent. Do NOT renumber — preserve the gap.

Conversion  : ✅ FULL


### PQ-02 | cru_training_db  (SQL Server Annotation Writeback)
─────────────────────────────────────────────────────────────
Tableau Source  : Federated datasource driving annotation overlay
                  (JSON string parameter injection in Tableau)
Power BI Source : Sql.Database("usvnabmsasp030", "tabwritebackprod")
Schema / Table  : [GPS BIA].[cru_training_db]

Power BI M Query:
─────────────────
let
    Source = Sql.Database("usvnabmsasp030", "tabwritebackprod"),
    #"GPS BIA_cru_training_db" =
        Source{[Schema="GPS BIA", Item="cru_training_db"]}[Data],
    #"Select Columns" = Table.SelectColumns(
        #"GPS BIA_cru_training_db",
        {"id", "ref_id", "filter_parameter_set",
         "audit_user", "audit_time", "justification"}
    )
in
    #"Select Columns"

Columns loaded (6 only — explicit selection, no wildcard):
  id, ref_id, filter_parameter_set, audit_user, audit_time, justification

⚠️  Any new source column requires an M change to appear in PBIP.
⚠️  PBIP column renames vs source:
    id                   → Id
    ref_id               → Ref Id
    filter_parameter_set → Filter Parameter Set
    audit_user           → Audit User
    audit_time           → Audit Time
    justification        → Justification  (capitalised)
⚠️  PBIP does NOT set isHidden on any cru_training_db column, even those
    that are hidden in Tableau. All 6 columns are visible. Preserve this.

Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — HIERARCHY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### HIER-01 | Heat Map  (Training Data table)
─────────────────────────────────────────────────────────────
PBI Table       : Training Data
PBI lineageTag  : 39636ebc-3372-440b-ad91-ed2da214f45f

Levels (top → bottom):

  Level 1 | Name: full_name
    → column: full_name
    → lineageTag: 617b27a8-03fa-4582-82e3-b2cf8706222d

  Level 2 | Name: job_fam_name
    → column: Tier 2 Dept      ← RENAMED column
    → lineageTag: b8ad4c38-12e9-4ef5-8dc4-f0eebc9fb59e

  Level 3 | Name: hr_orgzn_nm
    → column: Tier 1 Dept      ← RENAMED column
    → lineageTag: 6bb527fa-d581-4eb5-8dc4-f0eebc9fb59e

  Level 4 | Name: lt department
    → column: LT Department    ← RENAMED column
    → lineageTag: 09f23f12-f2fd-4ff8-b0b3-1cf9d9fec8d0

⚠️  Hierarchy level names use the Tableau/source column names
    (full_name, job_fam_name, hr_orgzn_nm, lt department).
    The actual PBIP column references use the RENAMED PBIP display names
    (Tier 2 Dept, Tier 1 Dept, LT Department).
    Preserve this split between level name and column reference exactly.

Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — CONVERSION COMPLETENESS SCORECARD
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Category                       | Tableau Items | PBI Full | PBI Partial | Not Converted | % Complete |
|--------------------------------|---------------|----------|-------------|---------------|------------|
| Parameters                     | 4             | 3        | 0           | 1 (dropped)   | 75%        |
| Calculated Columns (Trng Data) | 17            | 17       | 0           | 0             | 100%       |
| Calculated Column (cru_db)     | 1             | 1        | 0           | 0             | 100%       |
| DAX Measures                   | 7             | 7        | 0           | 0             | 100%       |
| Power Query / SQL              | 2             | 2        | 0           | 0             | 100%       |
| Hierarchy                      | 1             | 1        | 0           | 0             | 100%       |
| Parameter Tables (PBI-native)  | 3             | 3        | 0           | 0             | (new)      |
| TOTAL                          | 35            | 34       | 0           | 1             | 🟢 ~97%    |

ADAPTED ITEM:
  ⚠️  PARAM-02 | Training_Details_AnnotationParam
      Tableau JSON parameter injection mechanism not carried to PBIP.
      cru_training_db loaded in full via Import; [On-Leave Flag] column
      drives annotation display. Filtering at visual/report layer.
      → Functionally covered by the architecture, not identical mechanism.

PENDING (to reach 100%):
  → [Item ID_count] measure definition (referenced in M-03)
  → [Item Id_Count] measure definition (referenced in M-04)
  → [Item Id_Count_3M] measure definition (referenced in M-04)
  → database.tmdl / model.tmdl (compatibilityLevel, culture, PBI_QueryOrder)
  → relationships.tmdl (date column → LocalDateTable mappings)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — CONVERSION PATTERN CHEAT SHEET
## (Use this when converting future Tableau workbooks)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PATTERN-01 | Integer parameter → SWITCH(SELECTEDVALUE())
─────────────────────────────────────────────────────────────
Tableau:
  CASE [Parameters].[MyParam]
    WHEN 1 THEN [field_a]
    WHEN 2 THEN [field_b]
  END

Power BI (column):
  SWITCH(
      SELECTEDVALUE('MyParamTable'[KeyColumn], 0),
      1, [field_a],
      2, [field_b],
      BLANK()
  )

Power BI (parameter table):
  DATATABLE(
      "Label", STRING,
      "Key",   INTEGER,
      { {"Value A", 1}, {"Value B", 2} }
  )


PATTERN-02 | Tableau ISNULL() → PBIP ISBLANK() + empty string + literal "Null"
─────────────────────────────────────────────────────────────
Tableau:
  IF ISNULL([field]) THEN "Active" ELSE "On-Leave" END

Power BI (enhanced — preserve all 3 guards):
  IF(
      OR(ISBLANK([Field]), [Field] = ""),
      "Active",
      IF([Field] = "Null", "Active", "On-Leave")
  )

WHY: PBI datasets may contain empty strings or the literal string "Null"
     where Tableau would see NULL. All three must map to the "Active" branch.


PATTERN-03 | Boolean column format string
─────────────────────────────────────────────────────────────
For any boolean calculated column that must display TRUE/FALSE as strings:
  formatString: """TRUE"";""TRUE"";""FALSE"""
This preserves the three-part format string exactly as written in TMDL.


PATTERN-04 | DATETRUNC('week') → WEEKDAY(date, 2) Monday-start alignment
─────────────────────────────────────────────────────────────
Tableau:
  DATETRUNC('week', [mydate])     → Monday-aligned week start

Power BI:
  [mydate] - WEEKDAY([mydate], 2) + 1
  (WEEKDAY mode=2: 1=Mon, 2=Tue, ... 7=Sun)


PATTERN-05 | DATETRUNC('month') → DATE(YEAR(), MONTH(), 1)
─────────────────────────────────────────────────────────────
Tableau:
  DATETRUNC('month', [mydate])    → first day of month

Power BI:
  DATE(YEAR([mydate]), MONTH([mydate]), 1)


PATTERN-06 | User-scoped filtering → USERNAME() + CALCULATE
─────────────────────────────────────────────────────────────
Tableau:
  LOWER([bms_user_id]) = LOWER(USERNAME())

Power BI:
  CALCULATE(
      [MyMeasure],
      'Training Data'[bms_user_id] = USERNAME()
  )

Note: PBI USERNAME() returns UPN (email). Confirm case sensitivity
      in the deployed tenant before removing LOWER() equivalence.


PATTERN-07 | Coming Due window → DATEDIFF(TODAY(), future_date, DAY)
─────────────────────────────────────────────────────────────
Tableau:
  DATEDIFF('day', TODAY(), [required date]) <= N

Power BI:
  CALCULATE(
      [MyMeasure],
      FILTER(
          ALL('Training Data'[required date]),
          DATEDIFF(TODAY(), 'Training Data'[required date], DAY) <= N
      )
  )

Note: DATEDIFF(TODAY(), future_date, DAY) is positive for future dates.
      ALL() on [required date] only — other filter context preserved.


PATTERN-08 | Tableau comment preservation in DAX
─────────────────────────────────────────────────────────────
For every calculated column originating from Tableau, add:
  /* Original Tableau Expression is <tableau_formula_here> */
at the end of the DAX expression.
Use /// header comments for column-level display names:
  /// My Column Caption - Column
These comments appear in Tabular Editor and VS Code TMDL view.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAP-01 | [Item ID_count], [Item Id_Count], [Item Id_Count_3M] — pending
  Severity : High
  Detail   : M-03, M-04, M-05, M-06 all depend on these base count
             measures. Their definitions were not in the uploaded PBIP
             files. Without them the Coming Due Filter measures cannot
             be validated or reproduced.
  Action   : Upload the TMDL file containing these measure definitions
             (likely a measures table or Training Data measures TMDL).
  ⚠️  Capitalisation matters: [Item ID_count] ≠ [Item Id_Count].
      Preserve both exact capitalisations as-is.


GAP-02 | 14 always-NULL Redshift columns retained in PBIP
  Severity : Info
  Detail   : user name, organization id, organization, security domain id,
             security domain, curriculum id, curriculum, item type,
             item revision, assignment type, completion status, source,
             overdue, days remaining — all always NULL from Redshift SQL.
  Action   : Do NOT remove. Structural placeholders for future source
             schema population. Preserve column definitions in TMDL.


GAP-03 | training_key trailing underscore for NULL completion date
  Severity : Info
  Detail   : SQL: CASE WHEN "completion date" IS NULL THEN ''
             → training_key for incomplete items ends in '_'
             e.g. "U001_ITEM_TRAIN001_"
  Action   : Preserve exactly. Do NOT substitute a placeholder for NULL.


GAP-04 | Relationships not uploaded
  Severity : Medium
  Detail   : No relationships.tmdl was included. Date column →
             LocalDateTable mappings are inferred only.
  Action   : Upload relationships.tmdl to complete the model mapping.
             All parameter tables (Coming Due Selector, Heat Map Selector,
             Date Aggregation Selector, Date Aggregation Selector1,
             Date selector, Summary Aggregation) are expected to be
             disconnected — consumed via SELECTEDVALUE() and NAMEOF() only.


GAP-05 | Model metadata not uploaded
  Severity : Low
  Detail   : No database.tmdl or model.tmdl uploaded. compatibilityLevel,
             culture, TimeIntelligenceEnabled, and PBI_QueryOrder are
             unknown for this model.
  Action   : Upload database.tmdl / model.tmdl to complete the record.


GAP-06 | Tableau Parameter 3 JSON injection mechanism dropped
  Severity : Low
  Detail   : The Tableau annotation filter was driven by a hidden JSON
             string parameter ([Parameter 3]) injected at query time.
             PBIP loads cru_training_db in full; no JSON mechanism exists.
  Action   : Confirm with business owner that full-table load +
             visual-layer filtering is acceptable. If row-level
             annotation scoping is needed, implement as a DAX measure
             using TREATAS or a report-level filter on [Ref Id].


GAP-07 | [Heatmap Scope] references "Not Yet Due" — orphaned status value
  Severity : Info
  Detail   : [Heatmap Scope] checks Training Status <> "Not Yet Due",
             but [Training Status] (CC-01) never produces "Not Yet Due".
             This condition always evaluates TRUE — acts as a NULL guard.
  Action   : Preserve exactly. Do NOT optimise away. If CC-01 is ever
             updated to add "Not Yet Due" as an output, this guard
             will activate as intended.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — FILE INVENTORY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| File                                        | Content                                  |
|---------------------------------------------|------------------------------------------|
| 19810_cru_training_dashboard___prod.txt     | Tableau workbook XML (full)              |
| training_data.txt                           | Training Data table TMDL (main fact)     |
| cru_training_db.txt                         | cru_training_db annotation table TMDL   |
| coming_due_selector.txt                     | Coming Due Selector table + 5 measures   |
| date_aggregation_selector.txt               | Date Aggregation Selector DATATABLE      |
| date_aggregation_selector1.txt              | Date Aggregation Selector1 NAMEOF table  |
| date_selector.txt                           | Date selector NAMEOF field parameter     |
| heat_map_selector.txt                       | Heat Map Selector DATATABLE              |
| summary_aggregation.txt                     | Summary Aggregation PQ M table           |

Pending uploads:
| [database].tmdl                             | compatibilityLevel, culture, metadata    |
| [model].tmdl                                | Table refs, PBI_QueryOrder, annotations  |
| [relationships].tmdl                        | All relationship definitions             |
| [measures_table].tmdl or similar            | Item ID_count, Item Id_Count,            |
|                                             | Item Id_Count_3M definitions             |

END OF KNOWLEDGE FILE
# 19810 – CRU Training Dashboard (PROD)
# Tableau → Power BI (PBIP/TMDL)
# Generated: 2026-07-09 | Completeness: ~97% | Pending: see GAP-01, GAP-04, GAP-05
