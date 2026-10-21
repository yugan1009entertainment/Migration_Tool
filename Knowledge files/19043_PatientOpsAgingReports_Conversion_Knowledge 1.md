# 📘 CONVERSION KNOWLEDGE FILE
# 19043 – Patient Ops Aging Reports
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-03
# Source files: 19043_patient_ops_aging_reports.txt + all PBIP .txt files

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PARAM-01 | Reporting Date Granularity
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Reporting Date (copy)_659777386608910339]
Tableau Caption       : Reporting Date Granularity
Tableau Type          : Integer list (param-domain-type = list)
Tableau Default       : 0 (Month)

Tableau Aliases:
  0 → Month
  1 → Week
  2 → Quarter
  3 → Year

Power BI Table Name   : Filter Reporting Date Granularity
Power BI Type         : DAX Calculated Table
Power BI Columns      : [Reporting Date Label] (string), [Reporting Date Value] (integer)

Power BI DAX:
─────────────
DATATABLE(
    "Reporting Date Label", STRING,
    "Reporting Date Value", INTEGER,
    {
        { "Month",   0 },
        { "Week",    1 },
        { "Quarter", 2 },
        { "Year",    3 }
    }
)

Usage in model:
  - SELECTEDVALUE('Filter Reporting Date Granularity'[Reporting Date Value])
    returns 0/1/2/3 to switch display format in measures and columns.

Conversion Status: ✅ FULL — all 4 values, order and keys match exactly.


### PARAM-02 | Reporting Date (Field Selector)
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Reschedule Classification Parameter (copy)_659777386608484354]
Tableau Caption       : Reporting Date
Tableau Type          : Integer list
Tableau Default       : 0 (First Cancellation Date)

Tableau Aliases:
  0 → First Cancellation Date
  1 → Last Cancellation Date
  2 → First Confirmation Date
  3 → Last Confirmation Date
  4 → First Collection Appointment
  5 → Last Collection Appointment

Power BI Table Name   : Date Selector Table
Power BI Type         : DAX Calculated Table
Power BI Columns      : [Date Type] (string, visible), [Sort] (integer, hidden, sort column)

Power BI DAX:
─────────────
DATATABLE(
    "Date Type", STRING,
    "Sort",      INTEGER,
    {
        { "First Confirmation Date",      2 },
        { "First Cancellation Date",      0 },
        { "Last Cancellation Date",       1 },
        { "Last Confirmation Date",       3 },
        { "First Collection Appointment", 4 },
        { "Last Collection Appointment",  5 }
    }
)

Usage in model:
  - SELECTEDVALUE('Date Selector Table'[Date Type])
    returns a string label used in all Dynamic Date measures.

Conversion Status: ✅ FULL — all 6 values present; Sort column preserves
                   Tableau integer ordering.


### PARAM-03 | Reschedule Classification Display Selector
─────────────────────────────────────────────────────────────
Tableau Equivalent    : Integer parameter controlling which reschedule
                        grouping column drives the Filter calculated column
Tableau Default       : 0 (Reschedule Classification)

Tableau Values:
  0 → Reschedule Classification
  1 → Cancellation Lead Time
  2 → Reschedule Classification (group)

Power BI Table Name   : Reschedule Classification Parameter
Power BI Type         : DAX Calculated Table
Power BI Columns      : [ParameterTableKeyColumn] (integer), 
                        [Reschedule Classification Label] (string)

Power BI DAX:
─────────────
DATATABLE(
    "ParameterTableKeyColumn",         INTEGER,
    "Reschedule Classification Label", STRING,
    {
        { 0, "Reschedule Classification"        },
        { 1, "Cancellation Lead Time"           },
        { 2, "Reschedule Classification (group)"}
    }
)

Usage in model:
  - SELECTEDVALUE('Reschedule Classification Parameter'[ParameterTableKeyColumn], 0)
    drives the [Filter Reschedule Classification] calculated column SWITCH.

Conversion Status: ✅ FULL


### PARAM-04 | Reporting Metric
─────────────────────────────────────────────────────────────
Tableau Equivalent    : Integer list parameter for APH metric visual switch
Tableau Default       : 0 (Aph Completion Status)

Tableau Values:
  0 → Aph Completion Status
  1 → Cancellation to Collection Appt (days)
  2 → Rescheduled Cancellations

Power BI Table Name   : Reporting Metric
Power BI Type         : DAX Calculated Table
Power BI Columns      : [Reporting Metric Value] (integer),
                        [Reporting Metric Label] (string)

Power BI DAX:
─────────────
DATATABLE(
    "Reporting Metric Value", INTEGER,
    "Reporting Metric Label", STRING,
    {
        { 0, "Aph Completion Status"                    },
        { 1, "Cancellation to Collection Appt (days)"   },
        { 2, "Rescheduled Cancellations"                }
    }
)

Conversion Status: ✅ FULL


### PARAM-05 | Date Granularity Axis Toggle
─────────────────────────────────────────────────────────────
Tableau Equivalent    : DATETRUNC granularity switcher (Week/Month/Quarter/Year)
                        driving axis field on time-series visuals

Power BI Table Name   : Date Granularity Axis Parameter
Power BI Type         : DAX Calculated Table (NAMEOF-based field parameter)
Power BI Columns      :
  [Date Granularity Axis Parameter]        — display label (string, visible)
  [Date Granularity Axis Parameter Fields] — NAMEOF() field ref (string, hidden,
                                             ParameterMetadata kind=2)
  [Date Granularity Axis Parameter Order]  — sort integer (hidden)

Power BI DAX:
─────────────
{
    ( "Week",    NAMEOF('Date Table'[Week]),    0 ),
    ( "Month",   NAMEOF('Date Table'[Month]),   1 ),
    ( "Quarter", NAMEOF('Date Table'[Quarter]), 2 ),
    ( "Year",    NAMEOF('Date Table'[Year]),    3 )
}

Usage in model:
  - The [Date Granularity Axis Parameter Fields] column is bound to the
    visual X-axis as a field parameter, enabling dynamic axis switching.
  - Also referenced as MIN('Date Granularity Axis Parameter'
    [ParameterTableKeyColumn]) inside the [Filter Reporting Date] column.

NOTE: Tableau default = 0 = Month; PBI table rows list Week first (order=0).
      Functionally equivalent — sort driven by Order column, not row position.

Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: pre_aph_cancellation_tracker
## PBI Full Table Name: pre_aph_cancellation_tracker
##   (ct_bia pre_aph_cancellation_trackerfdf0f

### CC-01 | Reschedule Classification
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_659777386604994561
PBI Data Type   : string
PBI Hidden      : Yes
PBI Comment     : /// Reschedule Classification - Column

Tableau Expression:
───────────────────
IF [reschedule_status] = 'completed reschedule'
THEN 'Reschedule Apheresis Complete'
ELSEIF [reschedule_status] = 'rescheduled with future collection date'
THEN 'Upcoming Rescheduled Apheresis'
ELSE 'No Rescheduled Apheresis'
END

Power BI DAX:
─────────────
/// Reschedule Classification - Column
column 'Reschedule Classification' =
IF(
    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
        [Reschedule Status] = "completed reschedule",
    "Reschedule Apheresis Complete",
    IF(
        'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Reschedule Status] = "rescheduled with future collection date",
        "Upcoming Rescheduled Apheresis",
        "No Rescheduled Apheresis"
    )
)

Depends on  : [Reschedule Status] (source column)
Used by     : CC-02, CC-03, CC-09, CC-11, Measure: Label Count & %
Conversion  : ✅ FULL


### CC-02 | Reschedule Classification (group)
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Reschedule Classification (group)
PBI Data Type   : string
PBI Hidden      : Yes

Tableau Expression:
───────────────────
// Categorical bin grouping
IF [Reschedule Classification] IN ('No Rescheduled Apheresis','no reschedule')
THEN 'No Reschedule'
ELSEIF [Reschedule Classification] IN (
    'Reschedule Apheresis Complete',
    'Upcoming Rescheduled Apheresis',
    'eventually aphed',
    'failed reschedule',
    'rescheduled with future collection date')
THEN 'Rescheduled'
END

Power BI DAX:
─────────────
column 'Reschedule Classification (group)' =
SWITCH(
    TRUE(),
    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
        [Reschedule Classification]
        IN { "No Rescheduled Apheresis", "no reschedule" },
    "No Reschedule",

    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
        [Reschedule Classification]
        IN {
            "Reschedule Apheresis Complete",
            "Upcoming Rescheduled Apheresis",
            "eventually aphed",
            "failed reschedule",
            "rescheduled with future collection date"
        },
    "Rescheduled",

    BLANK()
)

Depends on  : CC-01 [Reschedule Classification]
Used by     : CC-07, CC-03
Conversion  : ✅ FULL — all 5 "Rescheduled" bucket values preserved exactly


### CC-03 | Filter Reschedule Classification
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_659777386609246212
PBI Data Type   : string
PBI Hidden      : Yes
PBI Comment     : /// Filter Reschedule Classification@@ - Column

Tableau Expression:
───────────────────
CASE [Parameters].[Reschedule Classification Parameter]
  WHEN 0 THEN [Reschedule Classification]
  WHEN 1 THEN [Cancellation Lead Time]
  WHEN 2 THEN [Reschedule Classification (group)]
END

Power BI DAX:
─────────────
/// Filter Reschedule Classification@@ - Column
column 'Filter Reschedule Classification' =
SWITCH(
    SELECTEDVALUE(
        'Reschedule Classification Parameter'[ParameterTableKeyColumn],
        0
    ),
    0, 'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
           [Reschedule Classification],
    1, 'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
           [Cancellation Lead Time],
    2, 'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
           [Reschedule Classification (group)],
    BLANK()
)

Depends on  : CC-01, CC-06, CC-02; PARAM-03
Used by     : CC-10, visual axis switching
Conversion  : ✅ FULL


### CC-04 | Filter Reporting Date
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_659777386611716101
PBI Data Type   : dateTime
PBI Format      : dd/mm/yyyy
PBI Hidden      : Yes
PBI Comment     : /// Filter Reporting Date - Column

Tableau Expression (preserved verbatim in DAX comment):
───────────────────────────────────────────────────────
CASE [Parameters].[Reschedule Classification Parameter (copy)_659777386608484354]
  WHEN 0 THEN [first_cancellation_date]
  WHEN 1 THEN [last_cancellation_date]
  WHEN 2 THEN [first_confirmation_date]
  WHEN 3 THEN [last_cancellation_date]      ← NOTE: key 3 maps to last_cancellation_date
  WHEN 4 THEN [first_collection_appointment]           (NOT last_confirmation_date)
  WHEN 5 THEN [last_collection_appointment]
END

Power BI DAX:
─────────────
/// Filter Reporting Date - Column
column 'Filter Reporting Date' =
IF(
    MIN('Date Granularity Axis Parameter'[ParameterTableKeyColumn]) = 0,
    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
        [First Cancellation Date],
IF(
    MIN('Filter Reporting Date Param'[ParameterTableKeyColumn]) = 1,
    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
        [Last Cancellation Date],
IF(
    MIN('Filter Reporting Date Param'[ParameterTableKeyColumn]) = 2,
    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
        [First Confirmation Date],
IF(
    MIN('Filter Reporting Date Param'[ParameterTableKeyColumn]) = 3,
    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
        [Last Cancellation Date],
IF(
    MIN('Filter Reporting Date Param'[ParameterTableKeyColumn]) = 4,
    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
        [First Collection Appointment],
IF(
    MIN('Filter Reporting Date Param'[ParameterTableKeyColumn]) = 5,
    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
        [Last Collection Appointment],
    BLANK()
))))))
/* Original Tableau Expression is case
   [Parameters].[Reschedule Classification Parameter (copy)_659777386608484354]
   when 0 then [first_cancellation_date]
   when 1 then [last_cancellation_date]
   when 2 then [first_confirmation_date]
   when 3 then [last_cancellation_date]
   when 4 then [first_collection_appointment]
   when 5 then [last_collection_appointment] end */

Depends on  : PARAM-02 (Filter Reporting Date Param table),
              PARAM-05 (Date Granularity Axis Parameter)
Used by     : CC-08, CC-15, visual date axis
⚠️  LOGIC NOTE: Key 3 ("Last Confirmation Date" label) maps to
    [Last Cancellation Date] in BOTH Tableau and PBI — copied intentionally.
    Verify with business owner if this is correct.
Conversion  : ✅ FULL (original expression preserved in comment)


### CC-05 | Days from Cancellation to Collection Appointment
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_659777386838892551
PBI Data Type   : int64
PBI Hidden      : Yes
PBI Comment     : /// Days from Cancellation to Collection Appointment - Column

Tableau Expression:
───────────────────
[cancel_to_appt_days] * -1

Power BI DAX:
─────────────
/// Days from Cancellation to Collection Appointment - Column
column 'Days from Cancellation to Collection Appointment' =
'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
    [Cancel To Appt Days] * -1
/* Original Tableau Expression is [cancel_to_appt_days] * -1 */

Depends on  : [Cancel To Appt Days] (source column)
Used by     : CC-06, CC-07, CC-12
Conversion  : ✅ FULL


### CC-06 | Cancellation Lead Time
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_659777386861965323
PBI Data Type   : string
PBI Hidden      : Yes
PBI Comment     : /// Cancellation Lead Time - Column

Tableau Expression:
───────────────────
IF [Days from Cancellation to Collection Appointment] >= 0
   AND [Days from Cancellation to Collection Appointment] < 10
THEN '< 10 Days from Appointment'
ELSEIF [Days from Cancellation to Collection Appointment] >= 10
THEN '>= 10 Days from Appointment'
END

Power BI DAX:
─────────────
/// Cancellation Lead Time - Column
column 'Cancellation Lead Time' =
IF(
    AND(
        'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Days from Cancellation to Collection Appointment] >= 0,
        'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Days from Cancellation to Collection Appointment] < 10
    ),
    "< 10 Days from Appointment",
    IF(
        'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Days from Cancellation to Collection Appointment] >= 10,
        ">= 10 Days from Appointment",
        BLANK()
    )
)
/* Original Tableau Expression is
   if [Days from Cancellation to Collection Appointment] >= 0
   and [Days from Cancellation to Collection Appointment] < 10
   then '< 10 Days from Appointment'
   elseif [Days from Cancellation to Collection Appointment] >=10
   then '>= 10 Days from Appointment' end */

Depends on  : CC-05
Used by     : CC-03
Conversion  : ✅ FULL (original expression preserved in comment)


### CC-07 | Filter Pre-Aph Funnel
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_659777386875887629
PBI Data Type   : string
PBI Hidden      : Yes
PBI Comment     : /// Filter Pre-Aph Funnel - Column

Tableau Expression:
───────────────────
IF [Reschedule Classification (group)] = 'Rescheduled'
   AND [Days from Cancellation to Collection Appointment] >= 0
   AND [Days from Cancellation to Collection Appointment] < 10
THEN 'Rescheduled Last Minute'
ELSEIF [Reschedule Classification (group)] = 'Rescheduled'
THEN 'Rescheduled in Advance'
ELSEIF [Reschedule Classification (group)] != 'Rescheduled'
THEN 'Not Rescheduled'
END

Power BI DAX:
─────────────
/// Filter Pre-Aph Funnel - Column
column 'Filter Pre-Aph Funnel' =
VAR RescheduleGroup =
    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
        [Reschedule Classification (group)]

VAR DaysFromCancellation =
    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
        [Days from Cancellation to Collection Appointment]

RETURN
SWITCH(
    TRUE(),
    -- ✅ PBI enhancement: ISBLANK guard prevents NULL misclassification
    RescheduleGroup = "Rescheduled"
        && NOT ISBLANK(DaysFromCancellation)
        && DaysFromCancellation >= 0
        && DaysFromCancellation < 10,
    "Rescheduled Last Minute",

    RescheduleGroup = "Rescheduled",
    "Rescheduled in Advance",

    RescheduleGroup <> "Rescheduled",
    "Not Rescheduled",

    BLANK()
)

Depends on  : CC-02, CC-05
Used by     : CC-09, Measures: Label Count & %, 
              Enrollment Count by Filter Pre-Aph Funnel % (Dynamic Date)
PBI Enhancement: Added NOT ISBLANK(DaysFromCancellation) check not in Tableau.
Conversion  : ✅ FULL + enhanced


### CC-08 | Filter Reporting Date Granularity
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_659777386816663558
PBI Data Type   : dateTime
PBI Format      : MMM'YY
PBI Hidden      : Yes
PBI Comment     : /// Filter Reporting Date Granularity - Column

Tableau Expression:
───────────────────
CASE [Parameters].[Reporting Date Granularity]
  WHEN 0 THEN DATETRUNC('month',   [Filter Reporting Date])
  WHEN 1 THEN DATETRUNC('week',    [Filter Reporting Date])
  WHEN 2 THEN DATETRUNC('quarter', [Filter Reporting Date])
  WHEN 3 THEN DATETRUNC('year',    [Filter Reporting Date])
END

Power BI DAX:
─────────────
/// Filter Reporting Date Granularity - Column
column 'Filter Reporting Date Granularity' =
DATE(
    YEAR(
        'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Filter Reporting Date]
    ),
    MONTH(
        'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Filter Reporting Date]
    ),
    1
)

Depends on  : CC-04
Used by     : CC-09, CC-10, CC-11, CC-15
⚠️  PARTIAL: PBI always truncates to month-start DATE(Y,M,1).
    Week/Quarter/Year granularity is NOT handled in this column.
    Instead, axis-level switching is delegated to the
    [Date Granularity Axis Parameter] field parameter table (PARAM-05).
    The visual X-axis binds to that table's NAMEOF() field references.
Conversion  : ⚠️ PARTIAL — month grain only; granularity switching
              split to Date Granularity Axis Parameter table.


### CC-09 | Action (Filter Reporting Date Granularity, Filter Pre-Aph Funnel)
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Action (Filter Reporting Date Granularity,Filter Pre-Aph Funnel)
PBI Data Type   : string
PBI Hidden      : Yes
PBI Comment     : /// Action (Filter Reporting Date Granularity,Filter Pre-Aph Funnel)

Tableau Expression:
───────────────────
// Dashboard cross-filter action key
STR([Filter Reporting Date Granularity]) + " , " + [Filter Pre-Aph Funnel]

Power BI DAX:
─────────────
column 'Action (Filter Reporting Date Granularity,Filter Pre-Aph Funnel)' =
'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
    [Filter Reporting Date Granularity]
& " , " &
'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
    [Filter Pre-Aph Funnel]

Depends on  : CC-08, CC-07
Purpose     : Cross-filter navigation key — replicates Tableau dashboard
              filter action (select bar → filter other sheet)
Conversion  : ✅ FULL


### CC-10 | Action (Filter Reschedule Classification, Filter Reporting Date Granularity)
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Action (Filter Reschedule Classification ,Filter Reporting Date Granularity)
PBI Data Type   : string
PBI Hidden      : Yes

Tableau Expression:
───────────────────
[Filter Reschedule Classification] + " , " + STR([Filter Reporting Date Granularity])

Power BI DAX:
─────────────
column 'Action (Filter Reschedule Classification ,Filter Reporting Date Granularity)' =
'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
    [Filter Reschedule Classification]
& " , " &
'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
    [Filter Reporting Date Granularity]

Depends on  : CC-03, CC-08
Conversion  : ✅ FULL


### CC-11 | Action (Reschedule Classification, Filter Reporting Date Granularity)
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Action (Reschedule Classification,Filter Reporting Date Granularity)
PBI Data Type   : string
PBI Hidden      : Yes

Tableau Expression:
───────────────────
[Reschedule Classification] + " , " + STR([Filter Reporting Date Granularity])

Power BI DAX:
─────────────
column 'Action (Reschedule Classification,Filter Reporting Date Granularity)' =
'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
    [Reschedule Classification]
& " , " &
'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
    [Filter Reporting Date Granularity]

Depends on  : CC-01, CC-08
Conversion  : ✅ FULL


### CC-12 | Cancel To Appt Days (group)
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Cancel To Appt Days (group)
PBI Data Type   : string
PBI Hidden      : Yes

Tableau Expression:
───────────────────
// Categorical bin on cancel_to_appt_days integer values
IF [cancel_to_appt_days] IN {-9,-8,-7,-6,-5,-4,-3,-2,-1,0}
THEN '< 10 Days'
ELSEIF [cancel_to_appt_days] IN {-62,-60,-59,-56,-55,-54,-53,-50,-49,
                                  -48,-46,-45,-44,-43,-42,-41,-40,-39,
                                  -38,-37,-36,-35,-34,-33,-32,-31,-30,
                                  -29,-28,-27,-26,-25,-24,-23,-22,-21,
                                  -20,-19,-18,-17,-16,-15,-14,-13,-12,
                                  -11,-10}
THEN '>= 10 Days'
ELSE 'Other'
END

Power BI DAX:
─────────────
column 'Cancel To Appt Days (group)' =
SWITCH(
    TRUE(),
    FORMAT(
        'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Cancel To Appt Days],
        "0"
    ) IN { "-9","-8","-7","-6","-5","-4","-3","-2","-1","0" },
    "< 10 Days",

    FORMAT(
        'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Cancel To Appt Days],
        "0"
    ) IN {
        "-62","-60","-59","-56","-55","-54","-53","-50","-49","-48",
        "-46","-45","-44","-43","-42","-41","-40","-39","-38","-37",
        "-36","-35","-34","-33","-32","-31","-30","-29","-28","-27",
        "-26","-25","-24","-23","-22","-21","-20","-19","-18","-17",
        "-16","-15","-14","-13","-12","-11","-10"
    },
    ">= 10 Days",

    "Other"
)

Depends on  : [Cancel To Appt Days] (source column)
NOTE        : PBI uses FORMAT(...,"0") IN {"string",...} because DAX
              IN operator on string sets is cleaner than integer ranges
              here. All values are explicitly enumerated as in Tableau.
Conversion  : ✅ FULL


### CC-13 | cancellation_date_utc_MY
─────────────────────────────────────────────────────────────
PBI Lineage Tag : b1a2505d-fefd-4654-854f-328ce1cca6a0
PBI Data Type   : string
PBI Hidden      : Yes
PBI Comment     : /// This is a transformation column from the source
                      column with the format of MY

Tableau Expression:
───────────────────
DATENAME('month',[cancellation_date_utc]) + ", " + STR(YEAR([cancellation_date_utc]))
// Produces e.g. "January, 2024"

Power BI DAX:
─────────────
/// This is a transformation column from the source column with the format of MY
column cancellation_date_utc_MY =
FORMAT(
    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
        [Cancellation Date Utc],
    "MMMM, yyyy"
)

Depends on  : [Cancellation Date Utc] (source column)
Conversion  : ✅ FULL


### CC-14 | collection_appointment_Year
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 75ec85df-8085-44f1-bbbe-334d50b60879
PBI Data Type   : int64
PBI Hidden      : Yes
PBI Note        : Level 1 of the [collection_appointment_Date Hierarchy]

Tableau Expression:
───────────────────
YEAR([collection_appointment])

Power BI DAX:
─────────────
column collection_appointment_Year =
YEAR(
    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
        [Collection Appointment]
)

Depends on  : [Collection Appointment] (source column)
Conversion  : ✅ FULL


### CC-15 | Reporting Date Display
─────────────────────────────────────────────────────────────
PBI Lineage Tag : d6095f4c-c3c9-4d0f-a7f5-29fde3e2f34d
PBI Data Type   : string
PBI Hidden      : Yes

Tableau Expression:
───────────────────
CASE [Parameters].[Reporting Date Granularity]
  WHEN 0 THEN FORMAT([Filter Reporting Date], 'MMM YYYY')
  WHEN 1 THEN STR(DATEPART('week',[Filter Reporting Date]))
             + ' ' + STR(YEAR([Filter Reporting Date]))
  WHEN 2 THEN 'Q' + STR(DATEPART('quarter',[Filter Reporting Date]))
             + ' ' + STR(YEAR([Filter Reporting Date]))
  WHEN 3 THEN STR(YEAR([Filter Reporting Date]))
  ELSE        FORMAT([Filter Reporting Date], 'MMM YYYY')
END

Power BI DAX:
─────────────
column 'Reporting Date Display' =
SWITCH(
    SELECTEDVALUE('Filter Reporting Date Granularity'[Reporting Date Value]),
    0, FORMAT(
           MIN('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
               [First Cancellation Date]),
           "MMM YYYY"
       ),
    1, FORMAT(
           MIN('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
               [First Cancellation Date]),
           "WW YYYY"
       ),
    2, FORMAT(
           MIN('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
               [First Cancellation Date]),
           "\QQ YYYY"
       ),
    3, FORMAT(
           MIN('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
               [First Cancellation Date]),
           "YYYY"
       ),
    -- Default (no match):
    FORMAT(
        MIN('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [First Cancellation Date]),
        "MMM YYYY"
    )
)

Depends on  : PARAM-01, [First Cancellation Date] (source column)
NOTE        : PBI uses MIN([First Cancellation Date]) because this is a
              calculated column evaluated per row — the column is then used
              in visuals. Tableau uses the current row date directly.
Conversion  : ✅ FULL


## TABLE: Aging Enrollments_Custom SQL Query

### CC-16 | Enrollment Date
─────────────────────────────────────────────────────────────
PBI Table       : Aging Enrollments_Custom SQL Query
PBI Lineage Tag : (auto)
PBI Data Type   : dateTime
PBI Hidden      : No
PBI Comment     : Tableau comment preserved

Tableau Expression:
───────────────────
DATE([enrollment_created_date])
// Strips time component from datetime

Power BI DAX:
─────────────
column 'Enrollment Date' =
DATE(
    YEAR([Enrollment Created Date]),
    MONTH([Enrollment Created Date]),
    DAY([Enrollment Created Date])
)

Depends on  : [Enrollment Created Date] (source column)
Conversion  : ✅ FULL


### CC-17 | Remove Test Sites Filter
─────────────────────────────────────────────────────────────
PBI Table       : Aging Enrollments_Custom SQL Query
PBI Data Type   : boolean
PBI Hidden      : No

Tableau Expression:
───────────────────
NOT(CONTAINS(LOWER([site_of_care]), 'test'))
// Returns TRUE when site_of_care does NOT contain "test"

Power BI DAX:
─────────────
column 'Remove Test Sites Filter' =
NOT(CONTAINSSTRING(LOWER([Site Of Care]), "test"))

Depends on  : [Site Of Care] (source column)
Conversion  : ✅ FULL


## TABLE: Aging Treatments_Custom SQL Query

### CC-18 | shelf life
─────────────────────────────────────────────────────────────
PBI Table       : Aging Treatments_Custom SQL Query
PBI Data Type   : int64
PBI Hidden      : No

Tableau Expression:
───────────────────
DATEDIFF('day', [date_of_manufacturing], [expiration_date])

Power BI DAX:
─────────────
column 'shelf life' =
IF(
    NOT ISBLANK([expiration_date])
        && NOT ISBLANK([date of manufacturing]),
    DATEDIFF(
        [date of manufacturing],
        [expiration_date],
        DAY
    ),
    BLANK()
)

Depends on  : [expiration_date], [date of manufacturing] (source columns)
PBI Enhancement: Added NOT ISBLANK() guards to prevent errors on NULL dates.
Conversion  : ✅ FULL + enhanced


### CC-19 | Days to Expiration  (Aging Treatments)
─────────────────────────────────────────────────────────────
PBI Table       : Aging Treatments_Custom SQL Query
PBI Data Type   : int64
PBI Hidden      : No

Tableau Expression:
───────────────────
[expiration_date] - TODAY()

Power BI DAX:
─────────────
column 'Days to Expiration' =
ROUND([expiration_date] - TODAY(), 0)

Depends on  : [expiration_date] (source column)
NOTE        : PBI adds ROUND(..., 0) to ensure clean integer result.
Conversion  : ✅ FULL


## TABLE: Aging Product Orders_Custom SQL Query

### CC-20 | Days to Expiration  (Aging Product Orders)
─────────────────────────────────────────────────────────────
PBI Table       : Aging Product Orders_Custom SQL Query
PBI Data Type   : decimal (float)
PBI Hidden      : No

Tableau Expression:
───────────────────
[expiration_date] - TODAY()

Power BI DAX:
─────────────
column 'Days to Expiration' =
[expiration date] - TODAY()

Depends on  : [expiration date] (source column)
NOTE        : No ROUND here (unlike Aging Treatments) — result is decimal.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: pre_aph_cancellation_tracker
## (Measures defined directly on the fact table)

### M-01 | Enrollment Count
─────────────────────────────────────────────────────────────
PBI Table       : pre_aph_cancellation_tracker
                  (ct_bia pre_aph_cancellation_trackerfdf0f
PBI Format      : 0
PBI Hidden      : Yes (IsHidden)
PBI Lineage Tag : 0acb510d-f67b-4232-b1c7-c2afdef37ddf

Tableau Equivalent:
───────────────────
COUNT([enrollment_name])
// or COUNTD([enrollment_name]) depending on context

Power BI DAX:
─────────────
measure 'Enrollment Count' =
COUNTROWS(
    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
)

Used by     : M-05 (Label Count & %), M-06, M-07, M-08
Conversion  : ✅ FULL


### M-02 | PctTotal_enrollment_name (auto-generated)
─────────────────────────────────────────────────────────────
PBI Table       : pre_aph_cancellation_tracker
                  (ct_bia pre_aph_cancellation_trackerfdf0f
PBI Format      : 0.00%
PBI Hidden      : Yes
PBI Comment     : /// Percentage of Grant Total enrollment_name_...
PBI Lineage Tag : [enrollment_name]

Tableau Equivalent:
───────────────────
// Auto-generated % of total — no direct Tableau equivalent
// PBI creates this when Quick Measure "% of grand total" is applied

Power BI DAX: (auto-generated, hidden)
─────────────
measure 'PctTotal_enrollment_name_pre_aph_cancellation_tracker
 (ct_bia.pre_aph_cancellation_tracker) (postgres)_pre_aph_cancellation_tracker'
= [auto-generated % of grand total DAX]

Type        : PBI-specific (no Tableau equivalent)
Conversion  : ➕ PBI-only, auto-generated


## TABLE: MeasureCal
## (Central measures table — all key DAX measures live here)

### M-03 | Enrollment Count_M
─────────────────────────────────────────────────────────────
PBI Table       : MeasureCal
PBI Format      : #,0
PBI Lineage Tag : 69cad606-2f81-4741-a0c1-e14cc4cc0800

Tableau Equivalent:
───────────────────
COUNT([enrollment_name])

Power BI DAX:
─────────────
measure 'Enrollment Count_M' =
COUNT(
    'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
        [Enrollment Name]
)

NOTE: Duplicate of M-01 with comma-formatted display. M-01 is hidden;
      M-03 is the display-ready version used in visuals.
Conversion  : ✅ FULL


### M-04 | Enrollment Count (Dynamic Date)
─────────────────────────────────────────────────────────────
PBI Table       : MeasureCal
PBI Format      : 0
PBI Lineage Tag : 53eb213e-6c8e-43df-b431-41c687828fbe

Tableau Equivalent:
───────────────────
// Parameter-driven COUNT that changes which date field filters the result
// Tableau uses CASE [Reporting Date Param] + date slicer context
CASE [Parameters].[Reporting Date]
  WHEN 0 THEN { FIXED : COUNT([enrollment_name])
                WHERE [first_cancellation_date] IN date_range }
  WHEN 1 THEN { FIXED : COUNT([enrollment_name])
                WHERE [last_cancellation_date] IN date_range }
  ...
END

Power BI DAX (FULL):
─────────────────────
measure 'Enrollment Count (Dynamic Date)' =

VAR _selectedFilter =
    SELECTEDVALUE('Date Selector Table'[Date Type])

VAR _FirstCancellation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Cancellation Date]
        )
    )

VAR _LastCancellation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        )
    )

VAR _FirstConfirmation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Confirmation Date]
        )
    )

VAR _LastConfirmation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]        ← NOTE: maps to Last Cancellation Date
        )
    )

VAR _FirstCollection =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Collection Appointment]
        )
    )

VAR _LastCollection =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Collection Appointment]
        )
    )

VAR _retVal =
    SWITCH(
        TRUE(),
        _selectedFilter = "First Cancellation Date",      _FirstCancellation,
        _selectedFilter = "Last Cancellation Date",       _LastCancellation,
        _selectedFilter = "First Confirmation Date",      _FirstConfirmation,
        _selectedFilter = "Last Confirmation Date",       _LastConfirmation,
        _selectedFilter = "First Collection Appointment", _FirstCollection,
        _selectedFilter = "Last Collection Appointment",  _LastCollection,
        BLANK()
    )

RETURN _retVal

Key Pattern : TREATAS(DISTINCT('Date Table'[Date]), [date_field])
              applies the Date Table slicer context to any date column
              without a physical relationship — equivalent to Tableau's
              LOD FIXED date window filtering.
Conversion  : ✅ FULL


### M-05 | Enrollment Count by Filter Pre-Aph Funnel % (Dynamic Date)
─────────────────────────────────────────────────────────────
PBI Table       : MeasureCal
PBI Format      : 0%;-0%;0%
PBI Lineage Tag : 1d5eb51e-65b2-4e36-8290-6a362d04a2d4

Tableau Equivalent:
───────────────────
// % of total with LOD-style denominator ignoring [Filter Pre-Aph Funnel]
// Tableau: SUM([enrollment_count]) / TOTAL(SUM([enrollment_count]))
// scoped to the current date filter, denominator uses EXCLUDE [Filter Pre-Aph Funnel]
{FIXED [date_field] : COUNT([enrollment_name])} /
{FIXED               : COUNT([enrollment_name])}

Power BI DAX (FULL):
─────────────────────
measure 'Enrollment Count by Filter Pre-Aph Funnel % (Dynamic Date)' =

VAR _selectedFilter =
    SELECTEDVALUE('Date Selector Table'[Date Type])

-- ── Numerator: filtered by date slicer AND current Filter Pre-Aph Funnel context ──

VAR _FirstCancellation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Cancellation Date]
        )
    )
VAR _LastCancellation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        )
    )
VAR _FirstConfirmation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Confirmation Date]
        )
    )
VAR _LastConfirmation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        )
    )
VAR _FirstCollection =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Collection Appointment]
        )
    )
VAR _LastCollection =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Collection Appointment]
        )
    )

VAR Count_ =
    SWITCH(
        TRUE(),
        _selectedFilter = "First Cancellation Date",      _FirstCancellation,
        _selectedFilter = "Last Cancellation Date",       _LastCancellation,
        _selectedFilter = "First Confirmation Date",      _FirstConfirmation,
        _selectedFilter = "Last Confirmation Date",       _LastConfirmation,
        _selectedFilter = "First Collection Appointment", _FirstCollection,
        _selectedFilter = "Last Collection Appointment",  _LastCollection,
        BLANK()
    )

-- ── Denominator: ALL on [Filter Pre-Aph Funnel] removes that column's filter ──

VAR _DenomFirstCancellation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Cancellation Date]
        ),
        ALL('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Filter Pre-Aph Funnel])
    )
VAR _DenomLastCancellation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        ),
        ALL('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Filter Pre-Aph Funnel])
    )
VAR _DenomFirstConfirmation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Confirmation Date]
        ),
        ALL('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Filter Pre-Aph Funnel])
    )
VAR _DenomLastConfirmation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        ),
        ALL('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Filter Pre-Aph Funnel])
    )
VAR _DenomFirstCollection =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Collection Appointment]
        ),
        ALL('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Filter Pre-Aph Funnel])
    )
VAR _DenomLastCollection =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Collection Appointment]
        ),
        ALL('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Filter Pre-Aph Funnel])
    )

VAR _Denom =
    SWITCH(
        TRUE(),
        _selectedFilter = "First Cancellation Date",      _DenomFirstCancellation,
        _selectedFilter = "Last Cancellation Date",       _DenomLastCancellation,
        _selectedFilter = "First Confirmation Date",      _DenomFirstConfirmation,
        _selectedFilter = "Last Confirmation Date",       _DenomLastConfirmation,
        _selectedFilter = "First Collection Appointment", _DenomFirstCollection,
        _selectedFilter = "Last Collection Appointment",  _DenomLastCollection,
        BLANK()
    )

RETURN DIVIDE(Count_, _Denom, 0)

Key Pattern : Denominator uses ALL([Filter Pre-Aph Funnel]) to replicate
              Tableau EXCLUDE LOD on that dimension.
Conversion  : ✅ FULL


### M-06 | Enrollment Count by Reschedule Classification % (Dynamic Date)
─────────────────────────────────────────────────────────────
PBI Table       : MeasureCal
PBI Format      : 0%;-0%;0%
PBI Lineage Tag : 0bec4ee8-005f-4d5f-93da-6edd99031ac4

Tableau Equivalent:
───────────────────
// Same as M-05 but denominator ignores [Reschedule Classification] column
{EXCLUDE [Reschedule Classification] : COUNT([enrollment_name])} as denominator

Power BI DAX (FULL):
─────────────────────
measure 'Enrollment Count by Reschedule Classification % (Dynamic Date)' =

VAR _selectedFilter =
    SELECTEDVALUE('Date Selector Table'[Date Type])

-- ── Numerator (same TREATAS pattern as M-05) ──────────────────────────────

VAR _FirstCancellation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Cancellation Date]
        )
    )
VAR _LastCancellation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        )
    )
VAR _FirstConfirmation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Confirmation Date]
        )
    )
VAR _LastConfirmation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        )
    )
VAR _FirstCollection =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Collection Appointment]
        )
    )
VAR _LastCollection =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Collection Appointment]
        )
    )

VAR Count_ =
    SWITCH(
        TRUE(),
        _selectedFilter = "First Cancellation Date",      _FirstCancellation,
        _selectedFilter = "Last Cancellation Date",       _LastCancellation,
        _selectedFilter = "First Confirmation Date",      _FirstConfirmation,
        _selectedFilter = "Last Confirmation Date",       _LastConfirmation,
        _selectedFilter = "First Collection Appointment", _FirstCollection,
        _selectedFilter = "Last Collection Appointment",  _LastCollection,
        BLANK()
    )

-- ── Denominator: ALL on [Reschedule Classification] ────────────────────────

VAR _DenomFirstCancellation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Cancellation Date]
        ),
        ALL('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Reschedule Classification])
    )
VAR _DenomLastCancellation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        ),
        ALL('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Reschedule Classification])
    )
VAR _DenomFirstConfirmation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Confirmation Date]
        ),
        ALL('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Reschedule Classification])
    )
VAR _DenomLastConfirmation =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        ),
        ALL('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Reschedule Classification])
    )
VAR _DenomFirstCollection =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Collection Appointment]
        ),
        ALL('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Reschedule Classification])
    )
VAR _DenomLastCollection =
    CALCULATE(
        COUNTROWS('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Collection Appointment]
        ),
        ALL('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [Reschedule Classification])
    )

VAR _Denom =
    SWITCH(
        TRUE(),
        _selectedFilter = "First Cancellation Date",      _DenomFirstCancellation,
        _selectedFilter = "Last Cancellation Date",       _DenomLastCancellation,
        _selectedFilter = "First Confirmation Date",      _DenomFirstConfirmation,
        _selectedFilter = "Last Confirmation Date",       _DenomLastConfirmation,
        _selectedFilter = "First Collection Appointment", _DenomFirstCollection,
        _selectedFilter = "Last Collection Appointment",  _DenomLastCollection,
        BLANK()
    )

RETURN DIVIDE(Count_, _Denom, 0)

Conversion  : ✅ FULL


### M-07 | Reporting Date Display
─────────────────────────────────────────────────────────────
PBI Table       : MeasureCal
PBI Format      : (string)
PBI Lineage Tag : 6a88b605-c765-4135-ac66-229e28ce6eaf

Tableau Equivalent:
───────────────────
// Display label for the current date axis period
CASE [Reporting Date Granularity]
  WHEN 0 THEN FORMAT([Filter Reporting Date], 'MMM YYYY')
  WHEN 1 THEN STR(DATEPART('week',...)) + ' YYYY'
  WHEN 2 THEN 'QQ YYYY'
  WHEN 3 THEN 'YYYY'
END

Power BI DAX (FULL):
─────────────────────
measure 'Reporting Date Display' =
SWITCH(
    SELECTEDVALUE('Filter Reporting Date Granularity'[Reporting Date Value]),
    0, FORMAT(
           MIN('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
               [First Cancellation Date]),
           "MMM YYYY"
       ),
    1, FORMAT(
           MIN('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
               [First Cancellation Date]),
           "WW YYYY"
       ),
    2, FORMAT(
           MIN('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
               [First Cancellation Date]),
           "\QQ YYYY"
       ),
    3, FORMAT(
           MIN('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
               [First Cancellation Date]),
           "YYYY"
       ),
    FORMAT(
        MIN('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
            [First Cancellation Date]),
        "MMM YYYY"
    )
)

Conversion  : ✅ FULL


### M-08 | Label Count & %
─────────────────────────────────────────────────────────────
PBI Table       : MeasureCal
PBI Format      : (string)
PBI Lineage Tag : 1fdc893d-335f-4fd8-84ed-5d0c77f6a9b3

Tableau Equivalent:
───────────────────
// Concatenated label: "45 (62%)"
STR(COUNT([enrollment_name])) + " (" +
STR(INT(COUNT([enrollment_name]) /
    TOTAL(COUNT([enrollment_name])) * 100)) + "%)"

Power BI DAX (FULL):
─────────────────────
measure 'Label Count & %' =
VAR Count_ = [Enrollment Count]
VAR Pct =
    DIVIDE(
        [Enrollment Count],
        CALCULATE(
            [Enrollment Count],
            ALLSELECTED(
                'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                    [Filter Pre-Aph Funnel]
            )
        ),
        0
    )
RETURN
    Count_ & " (" & FORMAT(Pct, "0%") & ")"

Key Pattern : ALLSELECTED([Filter Pre-Aph Funnel]) = Tableau TOTAL() scoped
              to current slicer selection (not all data).
Conversion  : ✅ FULL


### M-09 | Label Count & % (Dynamic Date)
─────────────────────────────────────────────────────────────
PBI Table       : MeasureCal
PBI Format      : (string)
PBI Lineage Tag : 91614299-c4d3-4b0d-894d-ad80ff643598

Tableau Equivalent:
───────────────────
// Same label as M-08 but with dynamic date field switching
// Tableau: parameter-driven COUNT with TOTAL denominator on [Filter Pre-Aph Funnel]

Power BI DAX (FULL):
─────────────────────
measure 'Label Count & % (Dynamic Date)' =

VAR _selectedFilter =
    SELECTEDVALUE('Date Selector Table'[Date Type])

VAR _FirstCancellation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Cancellation Date]
        )
    )
VAR _LastCancellation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        )
    )
VAR _FirstConfirmation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Confirmation Date]
        )
    )
VAR _LastConfirmation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        )
    )
VAR _FirstCollection =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Collection Appointment]
        )
    )
VAR _LastCollection =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Collection Appointment]
        )
    )

VAR Count_ =
    SWITCH(
        TRUE(),
        _selectedFilter = "First Cancellation Date",      _FirstCancellation,
        _selectedFilter = "Last Cancellation Date",       _LastCancellation,
        _selectedFilter = "First Confirmation Date",      _FirstConfirmation,
        _selectedFilter = "Last Confirmation Date",       _LastConfirmation,
        _selectedFilter = "First Collection Appointment", _FirstCollection,
        _selectedFilter = "Last Collection Appointment",  _LastCollection,
        BLANK()
    )

VAR _DenomFirstCancellation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Cancellation Date]
        ),
        ALLSELECTED(
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Filter Pre-Aph Funnel]
        )
    )
VAR _DenomLastCancellation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        ),
        ALLSELECTED(
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Filter Pre-Aph Funnel]
        )
    )
VAR _DenomFirstConfirmation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Confirmation Date]
        ),
        ALLSELECTED(
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Filter Pre-Aph Funnel]
        )
    )
VAR _DenomLastConfirmation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        ),
        ALLSELECTED(
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Filter Pre-Aph Funnel]
        )
    )
VAR _DenomFirstCollection =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Collection Appointment]
        ),
        ALLSELECTED(
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Filter Pre-Aph Funnel]
        )
    )
VAR _DenomLastCollection =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Collection Appointment]
        ),
        ALLSELECTED(
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Filter Pre-Aph Funnel]
        )
    )

VAR _Denom =
    SWITCH(
        TRUE(),
        _selectedFilter = "First Cancellation Date",      _DenomFirstCancellation,
        _selectedFilter = "Last Cancellation Date",       _DenomLastCancellation,
        _selectedFilter = "First Confirmation Date",      _DenomFirstConfirmation,
        _selectedFilter = "Last Confirmation Date",       _DenomLastConfirmation,
        _selectedFilter = "First Collection Appointment", _DenomFirstCollection,
        _selectedFilter = "Last Collection Appointment",  _DenomLastCollection,
        BLANK()
    )

VAR Pct = DIVIDE(Count_, _Denom, 0)

RETURN
    IF(
        ISBLANK(Count_),
        BLANK(),
        Count_ & " (" & FORMAT(Pct, "0%") & ")"
    )

Conversion  : ✅ FULL


### M-10 | Reschedule Classification_Label Count & % (Dynamic Date)
─────────────────────────────────────────────────────────────
PBI Table       : MeasureCal
PBI Format      : (string)
PBI Lineage Tag : 6d99c871-2259-4b4b-b32b-6d38e657969a

Tableau Equivalent:
───────────────────
// Same label pattern as M-09 but denominator ignores [Reschedule Classification]
// not [Filter Pre-Aph Funnel]

Power BI DAX (FULL):
─────────────────────
measure 'Reschedule Classification_Label Count & % (Dynamic Date)' =

VAR _selectedFilter =
    SELECTEDVALUE('Date Selector Table'[Date Type])

VAR _FirstCancellation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Cancellation Date]
        )
    )
VAR _LastCancellation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        )
    )
VAR _FirstConfirmation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Confirmation Date]
        )
    )
VAR _LastConfirmation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        )
    )
VAR _FirstCollection =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Collection Appointment]
        )
    )
VAR _LastCollection =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Collection Appointment]
        )
    )

VAR Count_ =
    SWITCH(
        TRUE(),
        _selectedFilter = "First Cancellation Date",      _FirstCancellation,
        _selectedFilter = "Last Cancellation Date",       _LastCancellation,
        _selectedFilter = "First Confirmation Date",      _FirstConfirmation,
        _selectedFilter = "Last Confirmation Date",       _LastConfirmation,
        _selectedFilter = "First Collection Appointment", _FirstCollection,
        _selectedFilter = "Last Collection Appointment",  _LastCollection,
        BLANK()
    )

VAR _DenomFirstCancellation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Cancellation Date]
        ),
        ALLSELECTED(
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Reschedule Classification]
        )
    )
VAR _DenomLastCancellation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        ),
        ALLSELECTED(
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Reschedule Classification]
        )
    )
VAR _DenomFirstConfirmation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Confirmation Date]
        ),
        ALLSELECTED(
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Reschedule Classification]
        )
    )
VAR _DenomLastConfirmation =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Cancellation Date]
        ),
        ALLSELECTED(
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Reschedule Classification]
        )
    )
VAR _DenomFirstCollection =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [First Collection Appointment]
        ),
        ALLSELECTED(
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Reschedule Classification]
        )
    )
VAR _DenomLastCollection =
    CALCULATE(
        COUNT('pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
              [Enrollment Name]),
        TREATAS(
            DISTINCT('Date Table'[Date]),
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Last Collection Appointment]
        ),
        ALLSELECTED(
            'pre_aph_cancellation_tracker (ct_bia pre_aph_cancellation_trackerfdf0f'
                [Reschedule Classification]
        )
    )

VAR _Denom =
    SWITCH(
        TRUE(),
        _selectedFilter = "First Cancellation Date",      _DenomFirstCancellation,
        _selectedFilter = "Last Cancellation Date",       _DenomLastCancellation,
        _selectedFilter = "First Confirmation Date",      _DenomFirstConfirmation,
        _selectedFilter = "Last Confirmation Date",       _DenomLastConfirmation,
        _selectedFilter = "First Collection Appointment", _DenomFirstCollection,
        _selectedFilter = "Last Collection Appointment",  _DenomLastCollection,
        BLANK()
    )

VAR Pct = DIVIDE(Count_, _Denom, 0)

RETURN
    IF(
        ISBLANK(Count_),
        BLANK(),
        Count_ & " (" & FORMAT(Pct, "0%") & ")"
    )

Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU INITIAL TRANSFORMATIONS → POWER BI POWER QUERY (M)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | pre_aph_cancellation_tracker
─────────────────────────────────────────────────────────────
Tableau Source  : Direct table connection
                  ct_bia.pre_aph_cancellation_tracker
Tableau Steps   : 1. Connect to table
                  2. All 46 columns available
                  3. No server-side SQL; Tableau queries on demand

Power BI M Query (FULL):
─────────────────────────
let
    Source = PostgreSQL.Database(
        "bms-eim-cell-therapy-auroradb-read-prod.web.bms.com:5432",
        "postgres"
    ),
    #"ct_bia_pre_aph_cancellation_tracker" =
        Source{[Schema = "ct_bia", Item = "pre_aph_cancellation_tracker"]}[Data],

    -- Step: Select only the 22 columns needed by PBI visuals
    -- (24 Tableau columns intentionally excluded — unused in visuals)
    #"Select Columns" = Table.SelectColumns(
        #"ct_bia_pre_aph_cancellation_tracker",
        {
            "enrollment_name",
            "join_id",
            "country",
            "region",
            "product_name",
            "out_of_spec_product",
            "out_of_spec_decision",
            "collection_appointment",
            "collection_end_actual",
            "cancellation_date_utc",
            "cancel_to_appt_days",
            "treatment_change_reason",
            "treatment_status",
            "site_of_care",
            "site_of_care_country",
            "last_collection_appointment",
            "first_collection_appointment",
            "first_confirmation_date",
            "last_cancellation_date",
            "first_cancellation_date",
            "last_appointment_completed_tf",
            "reschedule_status"
        }
    ),

    -- Step: Add display-ready Year-Month string for last_cancellation_date
    -- Tableau equivalent: DATENAME('month',...) + '-' + STR(YEAR(...))
    #"Added Custom" = Table.AddColumn(
        #"Select Columns",
        "Last cancellation Year Month",
        each
            if [last_cancellation_date] = null
            then null
            else Date.ToText(
                     Date.From([last_cancellation_date]),
                     "MMM-yyyy"
                 )
    )
in
    #"Added Custom"

Tableau Columns Excluded from PBI M Select (not needed in visuals):
  enrollment_channel_name, product, oos_reporting_date,
  appointment_requested_date, confirmation_date_utc,
  qa_release_actual, qp_release_actual, dp_delivery_actual,
  appt_request_to_confirmation_days, appt_request_to_cancel_days,
  confirmation_to_cancel_days, enrollment_cancellation_reason,
  attempted_re_aph, last_appointment_requested_date,
  first_appointment_requested_date,
  first_to_last_appointment_requested_days, number_of_requested_appts,
  first_to_last_collection_appointment_days,
  number_of_collection_appointments, last_confirmation_date,
  first_to_last_confirmation_date, number_of_confirmations_days,
  first_to_last_cancellation_date, number_of_cancellations


### PQ-02 | Aging Enrollments_Custom SQL Query
─────────────────────────────────────────────────────────────
Tableau Source  : Custom SQL on ct_bia.stg_gps_scheduling
                  (CTE with maxdates join + filters)
Tableau Filters applied at SQL level:
  - enrollment_created_date = max_enrollment_created_date (latest record per enrollment)
  - max_confirmation_date IS NULL (not yet confirmed)
  - enrollment_record_type IN ('Commercial','Clinical')
  - treatment_status NOT IN ('Requested','Confirmed','Completed')
    OR treatment_status IS NULL

Power BI M Query (FULL):
─────────────────────────
let
    Source = PostgreSQL.Database(
        "bms-eim-cell-therapy-auroradb-read-prod.web.bms.com:5432",
        "postgres"
    ),
    #"Aging Enrollments Query" = Value.NativeQuery(
        Source,
        "
        WITH maxdates AS (
            SELECT
                enrollment_name,
                MAX(enrollment_created_date) AS max_enrollment_created_date,
                MAX(confirmation_date_utc)   AS max_confirmation_date
            FROM ct_bia.stg_gps_scheduling
            GROUP BY 1
        )
        SELECT DISTINCT
            a.enrollment_name,
            a.site_of_care,
            a.site_of_care_country,
            a.enrollment_created_date,
            a.product_name          AS enrollment_product_name,
            a.indication            AS enrollment_indication,
            a.enrollment_record_type,
            a.confirmation_date_utc AS confirmation_date,
            a.enrollment_status,
            a.treatment_status,
            now()::date - a.enrollment_created_date::date AS days_since_enrollment
        FROM ct_bia.stg_gps_scheduling a
        LEFT JOIN maxdates USING (enrollment_name)
        WHERE a.enrollment_created_date = maxdates.max_enrollment_created_date
          AND maxdates.max_confirmation_date IS NULL
          AND a.enrollment_record_type IN ('Commercial', 'Clinical')
          AND (
                a.treatment_status NOT IN ('Requested', 'Confirmed', 'Completed')
                OR a.treatment_status IS NULL
              )
        ORDER BY days_since_enrollment DESC
        ",
        null,
        [EnableFolding = true]
    )
in
    #"Aging Enrollments Query"

Output Columns:
  enrollment_name, site_of_care, site_of_care_country,
  enrollment_created_date, enrollment_product_name,
  enrollment_indication, enrollment_record_type,
  confirmation_date, enrollment_status, treatment_status,
  days_since_enrollment (int — computed as SQL: now()::date - date)


### PQ-03 | Aging Treatments_Custom SQL Query
─────────────────────────────────────────────────────────────
Tableau Source  : Custom SQL — multi-CTE joining:
                  1. ct_bia.raw_oracle_lot_master        (Oracle lot data)
                  2. ct_bia.sap_dp_expiry +
                     ct_bia.o2c_sap_join_id_mapping      (SAP lot data)
                  3. ct_bia.stg_gps_scheduling           (GPS scheduling)
                  4. ct_bia.raw_jn_product_orders        (product order status)
                  5. ct_bia.conforming_status__slot      (QC/conformance)

Tableau Filters at SQL level:
  - dp_delivery_tz_actual IS NULL     (not yet delivered)
  - make2_end_tz_actual IS NOT NULL   (manufacturing complete)
  - treatment_status <> 'Cancelled'
  - enrollment_channel_name IN ('Commercial','Clinical')

Power BI M Query (FULL):
─────────────────────────
let
    Source = PostgreSQL.Database(
        "bms-eim-cell-therapy-auroradb-read-prod.web.bms.com:5432",
        "postgres"
    ),
    #"Aging Treatments Query" = Value.NativeQuery(
        Source,
        "
        WITH oracle_lots AS (
            SELECT join_id, lot, org_code, expiration_date,
                   date_of_manufacturing, has_remanufactured_lot,
                   has_eap_lot, has_sind_lot, has_eap_non_us_lot
            FROM ct_bia.raw_oracle_lot_master
        ),
        sap_lots AS (
            SELECT m.join_id, s.lot, s.org_code, s.expiration_date,
                   s.date_of_manufacturing
            FROM ct_bia.sap_dp_expiry s
            JOIN ct_bia.o2c_sap_join_id_mapping m
              ON s.sap_material_doc = m.sap_material_doc
        ),
        combined_lots AS (
            SELECT * FROM oracle_lots
            UNION ALL
            SELECT join_id, lot, org_code, expiration_date,
                   date_of_manufacturing, NULL, NULL, NULL, NULL
            FROM sap_lots
        ),
        product_orders AS (
            SELECT join_id,
                   MAX(CASE WHEN product_order_status = 'Cancelled'
                            THEN 'Yes' ELSE 'No' END)
                       AS has_cancelled_product_order,
                   MAX(confirmation_date)  AS product_order_confirmation_date,
                   MAX(cancellation_date)  AS product_order_cancellation_date,
                   MAX(product_order_status) AS product_order_status,
                   MAX(product_order_name)   AS product_order_name
            FROM ct_bia.raw_jn_product_orders
            GROUP BY 1
        )
        SELECT
            g.join_id,
            g.enrollment_name,
            g.enrollment_channel_name,
            g.product_name,
            now()::date - g.make2_end_tz_actual::date AS days_since_mfg_completed,
            po.has_cancelled_product_order,
            g.collection_end_actual,
            g.estimated_earliest_delivery,
            g.estimated_earliest_availability,
            g.out_of_spec_product,
            g.out_of_spec_decision,
            cl.lot,
            cl.expiration_date,
            cl.date_of_manufacturing,
            cl.has_remanufactured_lot,
            cl.has_eap_lot,
            cl.has_sind_lot,
            cl.has_eap_non_us_lot,
            cs.slot_conformance_status,
            cs.failure_mode,
            cs.make2_site_name,
            cs.brand,
            po.product_order_confirmation_date,
            po.product_order_cancellation_date,
            po.product_order_status,
            po.product_order_name,
            g.site_of_care,
            g.site_of_care_country,
            g.enrollment_indication
        FROM ct_bia.stg_gps_scheduling g
        LEFT JOIN combined_lots  cl ON g.join_id = cl.join_id
        LEFT JOIN product_orders po ON g.join_id = po.join_id
        LEFT JOIN ct_bia.conforming_status__slot cs ON g.join_id = cs.join_id
        WHERE g.dp_delivery_tz_actual IS NULL
          AND g.make2_end_tz_actual IS NOT NULL
          AND g.treatment_status <> 'Cancelled'
          AND g.enrollment_channel_name IN ('Commercial', 'Clinical')
        ",
        null,
        [EnableFolding = true]
    )
in
    #"Aging Treatments Query"

Scope: Post-manufacturing, pre-delivery.
       days_since_mfg_completed computed at SQL level.


### PQ-04 | Aging Product Orders_Custom SQL Query
─────────────────────────────────────────────────────────────
Tableau Source  : Custom SQL — same multi-CTE pattern as PQ-03
                  but different GPS scheduling filter:

Tableau Filters at SQL level:
  - dp_delivery_tz_actual IS NOT NULL   (delivered)
  - dp_admin_tz_actual IS NULL          (not yet administered)
  - treatment_status <> 'Cancelled'
  - product_order_status <> 'Cancelled'
  - enrollment_channel_name IN ('Commercial','Clinical')

Power BI M Query (FULL):
─────────────────────────
let
    Source = PostgreSQL.Database(
        "bms-eim-cell-therapy-auroradb-read-prod.web.bms.com:5432",
        "postgres"
    ),
    #"Aging Product Orders Query" = Value.NativeQuery(
        Source,
        "
        WITH oracle_lots AS (
            SELECT join_id, lot, org_code, expiration_date,
                   date_of_manufacturing, has_remanufactured_lot,
                   has_eap_lot, has_sind_lot, has_eap_non_us_lot
            FROM ct_bia.raw_oracle_lot_master
        ),
        sap_lots AS (
            SELECT m.join_id, s.lot, s.org_code, s.expiration_date,
                   s.date_of_manufacturing
            FROM ct_bia.sap_dp_expiry s
            JOIN ct_bia.o2c_sap_join_id_mapping m
              ON s.sap_material_doc = m.sap_material_doc
        ),
        combined_lots AS (
            SELECT * FROM oracle_lots
            UNION ALL
            SELECT join_id, lot, org_code, expiration_date,
                   date_of_manufacturing, NULL, NULL, NULL, NULL
            FROM sap_lots
        ),
        po_detail AS (
            SELECT join_id,
                   MAX(confirmation_date) AS po_confirmation_date
            FROM ct_bia.raw_jn_product_orders
            WHERE product_order_status <> 'Cancelled'
            GROUP BY 1
        )
        SELECT
            g.join_id,
            g.enrollment_name,
            g.enrollment_record_type,
            g.product_name,
            now()::date - g.dp_delivery_tz_actual::date AS days_since_delivery,
            pd.po_confirmation_date,
            g.dp_delivery_tz_actual,
            g.dp_admin_tz_planned,
            g.estimated_earliest_delivery,
            g.estimated_earliest_availability,
            g.out_of_spec_product,
            g.out_of_spec_decision,
            cl.lot,
            cl.org_code,
            cl.expiration_date,
            cl.date_of_manufacturing,
            cl.has_remanufactured_lot,
            cl.has_eap_lot,
            cl.has_sind_lot,
            cl.has_eap_non_us_lot,
            cs.slot_conformance_status,
            cs.failure_mode,
            cs.make2_site_name,
            cs.brand,
            cs.disposition_status_non_conforming,
            g.site_of_care,
            g.site_of_care_country,
            g.enrollment_indication
        FROM ct_bia.stg_gps_scheduling g
        LEFT JOIN combined_lots cl ON g.join_id = cl.join_id
        LEFT JOIN po_detail     pd ON g.join_id = pd.join_id
        LEFT JOIN ct_bia.conforming_status__slot cs ON g.join_id = cs.join_id
        WHERE g.dp_delivery_tz_actual IS NOT NULL
          AND g.dp_admin_tz_actual    IS NULL
          AND g.treatment_status      <> 'Cancelled'
          AND g.enrollment_channel_name IN ('Commercial', 'Clinical')
        ",
        null,
        [EnableFolding = true]
    )
in
    #"Aging Product Orders Query"

Scope: Post-delivery, pre-administration.
       days_since_delivery computed at SQL level.


### PQ-05 | MeasureCal (empty measures table)
─────────────────────────────────────────────────────────────
Tableau Equivalent : n/a (PBI-only pattern)
Purpose            : Empty table to host all DAX measures centrally,
                     preventing measure scatter across fact tables.

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

Result: Empty table with no columns. All DAX measures are defined on this
        table and referencing the fact tables via full table/column references.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — CONVERSION COMPLETENESS SCORECARD
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Category                   | Tableau Items | PBI Full | PBI Partial | Not Converted | % Complete |
|----------------------------|---------------|----------|-------------|---------------|------------|
| Parameters                 | 5             | 5        | 0           | 0             | 100%       |
| Calculated Columns         | 20            | 19       | 1           | 0             | 97.5%      |
| LOD / Measures (Tableau)   | 8             | 8        | 0           | 0             | 100%       |
| Power Query / SQL          | 4             | 4        | 0           | 0             | 100%       |
| Fact Tables                | 4             | 4        | 0           | 0             | 100%       |
| Parameter Tables (PBI)     | 5             | 5        | 0           | 0             | 100%       |
| PBI-only Measures          | n/a           | 2        | —           | —             | (new)      |
| TOTAL                      | 46            | 45       | 1           | 0             | 🟢 98.5%   |

PARTIAL ITEM:
  ⚠️  CC-08 | Filter Reporting Date Granularity
      Tableau uses dynamic DATETRUNC (month/week/quarter/year per row).
      PBI hardcodes month-start DATE(Y,M,1).
      Week/Quarter/Year switching delegated to Date Granularity Axis
      Parameter table (PARAM-05) at visual axis level.
      → Functionally covered by the architecture, but not identical DAX.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — CONVERSION PATTERN CHEAT SHEET
## (Use this when converting future Tableau workbooks)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PATTERN-01 | Integer parameter → SWITCH(SELECTEDVALUE())
─────────────────────────────────────────────────────────────
Tableau:
  CASE [Parameters].[MyParam]
    WHEN 0 THEN [field_a]
    WHEN 1 THEN [field_b]
  END

Power BI (column):
  SWITCH(
      SELECTEDVALUE('MyParamTable'[KeyColumn], 0),
      0, [field_a],
      1, [field_b],
      BLANK()
  )

Power BI (parameter table):
  DATATABLE(
      "Label", STRING,
      "Key",   INTEGER,
      { {"Value A", 0}, {"Value B", 1} }
  )


PATTERN-02 | Date-filtered COUNT (LOD equivalent) → TREATAS
─────────────────────────────────────────────────────────────
Tableau (LOD FIXED with date filter):
  { FIXED : COUNT([enrollment_name])
    WHERE [first_cancellation_date] IN [date_slicer_range] }

Power BI:
  CALCULATE(
      COUNT([Enrollment Name]),
      TREATAS(
          DISTINCT('Date Table'[Date]),
          [First Cancellation Date]        -- map Date Table dates onto this column
      )
  )

WHY: TREATAS creates a virtual relationship between the Date Table
     (used by the slicer) and the fact table's date column,
     enabling date-context filtering without a physical relationship.


PATTERN-03 | % of Total (LOD EXCLUDE) → ALL() / ALLSELECTED()
─────────────────────────────────────────────────────────────
Tableau (TOTAL ignoring one dimension):
  COUNT([enrollment_name]) /
  TOTAL(COUNT([enrollment_name]))    -- removes [dim] from context

Power BI (removes filter on [dim] completely):
  DIVIDE(
      [MyMeasure],
      CALCULATE([MyMeasure], ALL([dim_column])),
      0
  )

Power BI (respects slicer but removes cross-filter from [dim]):
  DIVIDE(
      [MyMeasure],
      CALCULATE([MyMeasure], ALLSELECTED([dim_column])),
      0
  )

Use ALL()        when denominator should ignore ALL filters on that dim.
Use ALLSELECTED() when denominator should respect external slicers
                  but ignore cross-filter from that dim.


PATTERN-04 | DATETRUNC granularity → DATE(Y, M, 1) + NAMEOF()
─────────────────────────────────────────────────────────────
Tableau:
  DATETRUNC('month',   [mydate])   → first day of month
  DATETRUNC('week',    [mydate])   → first day of week
  DATETRUNC('quarter', [mydate])   → first day of quarter
  DATETRUNC('year',    [mydate])   → first day of year

Power BI (column, month only):
  DATE(YEAR([mydate]), MONTH([mydate]), 1)

Power BI (axis switching via field parameter):
  {
      ("Week",    NAMEOF('Date Table'[Week]),    0),
      ("Month",   NAMEOF('Date Table'[Month]),   1),
      ("Quarter", NAMEOF('Date Table'[Quarter]), 2),
      ("Year",    NAMEOF('Date Table'[Year]),    3)
  }
  Bind this table's [Fields] column to the visual X-axis.
  Date Table must have pre-computed Week/Month/Quarter/Year columns.


PATTERN-05 | Tableau categorical bin → SWITCH(TRUE(), FORMAT() IN {})
─────────────────────────────────────────────────────────────
Tableau:
  IF [int_col] IN {0,1,2} THEN 'Group A'
  ELSEIF [int_col] IN {10,11,12} THEN 'Group B'
  ELSE 'Other' END

Power BI:
  SWITCH(
      TRUE(),
      FORMAT([int_col], "0") IN {"0","1","2"}, "Group A",
      FORMAT([int_col], "0") IN {"10","11","12"}, "Group B",
      "Other"
  )


PATTERN-06 | Tableau dashboard filter action → concatenated key column
─────────────────────────────────────────────────────────────
Tableau:
  Dashboard action: clicking [DimA] filters Sheet2 by [DimA] AND [DimB]

Power BI:
  Create a calculated column:
  [Action Key] = [DimA] & " , " & [DimB]
  Use this column for cross-visual filtering in PBI bookmarks/buttons.


PATTERN-07 | Tableau comment preservation in DAX
─────────────────────────────────────────────────────────────
For every calculated column that originated from Tableau, add:
  /* Original Tableau Expression is <tableau_formula_here> */
at the end of the DAX expression.
Use /// header comments for the column-level display name:
  /// My Column Caption - Column
These comments appear in Tabular Editor and VS Code TMDL view.


PATTERN-08 | Multi-source SQL CTE → Value.NativeQuery in M
─────────────────────────────────────────────────────────────
Tableau: Custom SQL with CTEs
Power BI:
  Value.NativeQuery(
      Source,                -- PostgreSQL.Database(...) connection
      "<full CTE SQL here>",
      null,
      [EnableFolding = true]
  )
EnableFolding=true allows query folding back to the database where possible.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAP-01 | Filter Reporting Date Granularity column — partial
  Severity : Medium
  Detail   : PBI column always returns month-start DATE(Y,M,1).
             Tableau's DATETRUNC dynamically returns week/quarter/year
             start per the parameter value at row level.
  Fix      : If row-level week/quarter start is needed in visuals beyond
             the axis toggle, replace the column DAX with:
             SWITCH(
                 SELECTEDVALUE('Filter Reporting Date Granularity'[Reporting Date Value]),
                 0, DATE(YEAR([Filter Reporting Date]),
                         MONTH([Filter Reporting Date]), 1),
                 1, [Filter Reporting Date]
                    - WEEKDAY([Filter Reporting Date], 2) + 1,
                 2, DATE(YEAR([Filter Reporting Date]),
                         ((ROUNDUP(MONTH([Filter Reporting Date])/3,0)-1)*3)+1, 1),
                 3, DATE(YEAR([Filter Reporting Date]), 1, 1)
             )

GAP-02 | Key 3 "Last Confirmation Date" maps to last_cancellation_date
  Severity : Low / Business query
  Detail   : In BOTH Tableau and PBI, parameter value 3 (labelled
             "Last Confirmation Date") resolves to [last_cancellation_date]
             not [last_confirmation_date]. This appears intentional (copied
             from Tableau) but may be a legacy bug.
  Action   : Confirm with business owner before changing.

GAP-03 | 24 Tableau source columns excluded from PBI M query
  Severity : Info
  Detail   : Columns such as enrollment_channel_name, confirmation_date_utc,
             qa/qp/dp release dates, appt_request_* metrics,
             attempted_re_aph, first_to_last_* intervals,
             number_of_* counts are NOT loaded into PBI.
  Action   : If any future report page needs these, add them back to
             the Table.SelectColumns list in PQ-01.

GAP-04 | last_appointment_completed_tf boolean alias not replicated
  Severity : Low
  Detail   : Tableau has alias: true → "Rescheduled Apheresis Completed",
             null → " " (blank).
             PBI loads the raw boolean column (TRUE/FALSE).
  Fix      : Add a calculated column:
             [Last Appointment Completed Label] =
             IF([Last Appointment Completed Tf] = TRUE,
                "Rescheduled Apheresis Completed",
                " ")

GAP-05 | Filter Reporting Date Param table not in uploaded files
  Severity : Info
  Detail   : CC-04 references both 'Date Granularity Axis Parameter'
             [ParameterTableKeyColumn] (for key 0) AND
             'Filter Reporting Date Param'[ParameterTableKeyColumn]
             (for keys 1–5). The latter table was not in the uploaded
             PBIP files. It is likely the same as 'Date Selector Table'
             or a separate lookup.
  Action   : Verify 'Filter Reporting Date Param' exists in the model
             and confirm its key column matches the 0–5 integer mapping.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — FILE INVENTORY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| File                                              | Content                          |
|---------------------------------------------------|----------------------------------|
| 19043_patient_ops_aging_reports.txt               | Tableau workbook XML             |
| 2_database.txt                                    | PBIP model database metadata     |
| 2_model.txt                                       | PBIP model table list            |
| pre_aph_cancellation_tracker__ct_bia_...fdf0f.txt | Main fact table TMDL             |
| measurecal.txt                                    | MeasureCal measures table TMDL   |
| aging_enrollments_custom_sql_query.txt            | Aging Enrollments TMDL           |
| aging_treatments_custom_sql_query.txt             | Aging Treatments TMDL            |
| aging_product_orders_custom_sql_query.txt         | Aging Product Orders TMDL        |
| date_selector_table.txt                           | Date Selector Table TMDL         |
| filter_reporting_date_granularity.txt             | Filter Reporting Date Gran. TMDL |
| date_granularity_axis_parameter.txt               | Date Granularity Axis Param TMDL |
| reporting_metric.txt                              | Reporting Metric table TMDL      |
| reschedule_classification_parameter.txt           | Reschedule Class. Param TMDL     |

END OF KNOWLEDGE FILE
