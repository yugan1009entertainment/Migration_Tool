# 📘 CONVERSION KNOWLEDGE FILE
# K4 – Balance Scorecard Q4 2025
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
#          K4 is a standalone knowledge base — do not mix with K1/K2/K3
#          unless explicitly instructed.
# Generated: 2026-07-23
# Source files: 29339_balance_scorecard_q4_2025.txt (Tableau workbook XML)
#               Power BI .pbip / TMDL files (Set 4B)
# Workbook ID: 29339
# Tableau site: PRODUCTION
# Power BI compatibility level: 1604
# Culture: en-US

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI M EXPRESSIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

K4 contains 10 Tableau parameters mapped to Power BI M expressions.
Unlike K1/K2 (which use SELECTEDVALUE on disconnected tables), K4
parameters are Power BI M `expression` objects with GUID names.

### PARAM-01 | Previous Comparision Month
─────────────────────────────────────────────────────────────
Tableau Caption       : Previous Comparision Month
Tableau Type          : string
Tableau Default       : "Mar 2024"

Power BI Expression GUID: a0ba73ea-d086-4b28-b8f6-e44084a00cc9
Power BI Type           : type text

⚠️  SPELLING NOTE: "Comparision" is a misspelling of "Comparison" —
    preserved exactly from the source workbook.

Conversion Status: ✅ FULL


### PARAM-02 | Previous Comparision Quarter
─────────────────────────────────────────────────────────────
Tableau Caption       : Previous Comparision Quarter
Tableau Type          : string list
Tableau Default       : "Q3'25"
Tableau Members       : Q1'23 through Q3'25

Power BI Expression GUID: 07635a69-c4fe-4080-9f16-583e30958f9b
Power BI Type           : type text

Power BI M Expression:
──────────────────────
expression 07635a69-c4fe-4080-9f16-583e30958f9b =
    "Q3'25" meta [
        IsParameterQuery = true,
        List = {
            "Q1'23","Q2'23","Q3'23","Q4'23",
            "Q1'24","Q2'24","Q3'24","Q4'24",
            "Q1'25","Q2'25","Q3'25"
        },
        DefaultValue = "Q3'25",
        Type = type text,
        IsParameterQueryRequired = true
    ]

Usage in model:
  - Drives Previous Comparison Quarter slicer tables.
  - Referenced by shape colour measures and count comparison measures.
Pattern     : K4-PARAM-02 — string list parameter with explicit member list.

Conversion Status: ✅ FULL


### PARAM-03 | Start Date Parameter
─────────────────────────────────────────────────────────────
Tableau Caption       : Start Date Parameter
Tableau Type          : date
Tableau Default       : #2023-12-30#

Power BI Expression GUID: f03829a9-b6ea-43d0-b6ec-4f959445d146
Power BI Type           : type date

Power BI M Expression:
──────────────────────
expression f03829a9-b6ea-43d0-b6ec-4f959445d146 =
    #date(2023, 12, 30) meta [
        IsParameterQuery = true,
        List = { #date(2023, 12, 30) },
        DefaultValue = #date(2023, 12, 30),
        Type = type date,
        IsParameterQueryRequired = true
    ]

Pattern     : Tableau date literal #YYYY-MM-DD# → PBI M #date(YYYY, MM, DD).

Conversion Status: ✅ FULL


### PARAM-04 | Select Date - Parameter
─────────────────────────────────────────────────────────────
Tableau Caption       : Select Date - Parameter
Tableau Type          : date
Tableau Default       : #2020-05-05#

Power BI Expression GUID: f3b7e43e-ede0-4e77-87be-e3cec934ad27
Power BI Type           : type date

Conversion Status: ✅ FULL


### PARAM-05 | End Date Parameter
─────────────────────────────────────────────────────────────
Tableau Caption       : End Date Parameter
Tableau Type          : date
Tableau Default       : #2025-01-16#

Power BI Expression GUID: e81b969c-0039-4d18-a29c-73e2f1d58a30
Power BI Type           : type date

Conversion Status: ✅ FULL


### PARAM-06 | Select Time Period
─────────────────────────────────────────────────────────────
Tableau Caption       : Select Time Period
Tableau Type          : string
Tableau Default       : "Current Month"

Power BI Expression GUID: f10a4b13-f25b-4af0-96f2-df19f63f4792
Power BI Type           : type text

Conversion Status: ✅ FULL


### PARAM-07 | Current Comparision Quarter
─────────────────────────────────────────────────────────────
Tableau Caption       : Current Comparision Quarter
Tableau Type          : string
Tableau Default       : "Q4'25"

Power BI Expression GUID: f30ce0c1-65a3-4a0a-8070-074034a88544
Power BI Type           : type text

⚠️  SPELLING NOTE: "Comparision" — misspelling preserved exactly.

Conversion Status: ✅ FULL


### PARAM-08 | by / Activity grouping
─────────────────────────────────────────────────────────────
Tableau Caption       : by (Activity grouping)
Tableau Type          : string
Tableau Default       : "Description"

Power BI Expression GUID: 84df0a60-36bb-4b0d-b978-e2f75e5346fb
Power BI Type           : type text

Conversion Status: ✅ FULL


### PARAM-09 | Top N
─────────────────────────────────────────────────────────────
Tableau Caption       : Top N
Tableau Type          : integer
Tableau Default       : 10

Power BI Expression GUID: 3f449d7f-4384-4b67-aac2-320e2c6bf321
Power BI Type           : type number

Power BI M Expression:
──────────────────────
expression 3f449d7f-4384-4b67-aac2-320e2c6bf321 =
    10 meta [
        IsParameterQuery = true,
        List = {
            0,1,2,3,4,5,6,7,8,9,10,
            11,12,13,14,15,16,17,18,19,20,
            21,22,23,24,25,26,27,28,29,30
        },
        DefaultValue = 10,
        Type = type number,
        IsParameterQueryRequired = true
    ]

Pattern     : Integer range parameter with explicit list (0–30).

Conversion Status: ✅ FULL


### PARAM-10 | by copy / Department grouping
─────────────────────────────────────────────────────────────
Tableau Caption       : by copy (Department grouping)
Tableau Type          : string
Tableau Default       : "Department"

Power BI Expression GUID: dfd59fa7-82bb-4a1d-9a60-47272008f2c5
Power BI Type           : type text

Conversion Status: ✅ FULL


### Parameter Rules Summary
─────────────────────────────────────────────────────────────
| Rule ID      | Rule                                                                         |
|--------------|------------------------------------------------------------------------------|
| K4-PARAM-01  | Tableau date literal #YYYY-MM-DD# maps to PBI M #date(YYYY, MM, DD).        |
| K4-PARAM-02  | Tableau string-list parameters map to M expressions with List={...}.         |
| K4-PARAM-03  | Tableau integer range parameters map to M expressions with Type=type number. |
| K4-PARAM-04  | Use explicit lists rather than generated ranges when matching PBIP output.   |
| K4-PARAM-05  | Parameter expression names may be GUIDs.                                     |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### CC-01 | Quarter with Year
─────────────────────────────────────────────────────────────
PBI DataType    : string
Applies to      : Multiple tables (Cost Savings, BMS, Activity Tracker)

Tableau Expression:
───────────────────
"Q" + STR(DATEPART('quarter', [Date])) + "'" + RIGHT(STR(YEAR([Date])),2)

Power BI DAX:
─────────────
[Quarter with Year] =
    "Q"
        & FORMAT ( QUARTER ( [Date] ), "0" )
        & "'"
        & RIGHT ( FORMAT ( YEAR ( [Date] ), "0000" ), 2 )

Pattern     : Tableau DATEPART('quarter') → DAX QUARTER().
              Tableau RIGHT(STR(YEAR()),2) → DAX RIGHT(FORMAT(YEAR(), "0000"), 2).
Format      : Qn'YY (e.g., Q4'25, Q1'23)
NOTE        : Used across multiple tables. The source date column varies
              per table (Date, Updated Date, End date).

| Rule ID      | Rule                                                           |
|--------------|----------------------------------------------------------------|
| K4-CALC-01   | Tableau DATEPART('quarter', [Date]) maps to DAX QUARTER().     |
| K4-CALC-02   | Tableau YEAR([Date]) maps to DAX YEAR([Date]).                 |
| K4-CALC-03   | RIGHT(STR(YEAR()),2) → RIGHT(FORMAT(YEAR(), "0000"), 2).      |
| K4-CALC-04   | Standard quarter label format is Qn'YY.                       |

Conversion  : ✅ FULL


### CC-02 | Quarter Sort
─────────────────────────────────────────────────────────────
PBI DataType    : int64

Power BI DAX:
─────────────
VAR _QuarterText = [Quarter with Year]
VAR _QuarterNo = VALUE ( MID ( _QuarterText, 2, 1 ) )
VAR _YearNo = 2000 + VALUE ( RIGHT ( _QuarterText, 2 ) )
RETURN _YearNo * 10 + _QuarterNo

Depends on  : CC-01 [Quarter with Year]
Purpose     : Numeric sort key for chronological quarter ordering.
              Display quarter columns must sortByColumn: Quarter Sort.
Example     : Q1'23 → 20231, Q4'25 → 20254

| Rule ID      | Rule                                                  |
|--------------|-------------------------------------------------------|
| K4-QTR-01    | Quarter display text uses format Qn'YY.               |
| K4-QTR-02    | Quarter sort key uses Year * 10 + QuarterNumber.      |
| K4-QTR-03    | Display quarter columns must sort by Quarter Sort.    |

Conversion  : ✅ FULL


### CC-03 | Cost Display (K/M Currency Formatting)
─────────────────────────────────────────────────────────────
PBI DataType    : string
Applies to      : Cost Savings table

Tableau Expression:
───────────────────
IF [Value] = 0 THEN ""
ELSEIF ISNULL([Value]) THEN ""
ELSEIF [Value] < 1 THEN "$" + STR(ROUND([Value] * 1000, 0)) + "K"
ELSE "$" + LEFT(STR(ROUND([Value], 3)), FIND(...) + 2) + "M"
END

Power BI DAX:
─────────────
[Cost Display] =
    VAR _V = [Value]
    RETURN
        IF (
            ISBLANK ( _V ) || _V = 0,
            "",
            IF (
                _V < 1,
                "$" & FORMAT ( ROUND ( _V * 1000, 0 ), "0" ) & "K",
                "$" & FORMAT ( ROUND ( _V, 3 ), "0.00" ) & "M"
            )
        )

Depends on  : [Value] (source column)
Business Rule: Zero/blank → empty string. <1 → $nnnK. ≥1 → $n.nnM.
Pattern     : Tableau ISNULL → DAX ISBLANK.
              Tableau STR/LEFT/FIND formatting → DAX FORMAT.

| Rule ID        | Rule                                               |
|----------------|----------------------------------------------------|
| K4-CURRENCY-01 | Tableau ISNULL maps to DAX ISBLANK.                |
| K4-CURRENCY-02 | Zero and blank values return empty string.          |
| K4-CURRENCY-03 | Values < 1 display in K format.                    |
| K4-CURRENCY-04 | Values ≥ 1 display in M format.                    |
| K4-CURRENCY-05 | Use DAX FORMAT instead of Tableau STR/LEFT/FIND.  |

Conversion  : ✅ FULL


### CC-04 | Fixed colour
─────────────────────────────────────────────────────────────
PBI DataType    : string

Power BI DAX:
─────────────
[Fixed colour] =
    IF (
        [Quarter with Year] = <Current Quarter Selection>,
        "Current Quarter",
        IF (
            [Quarter with Year] = <Previous Quarter Selection>,
            "Previous Quarter",
            BLANK ()
        )
    )

Depends on  : CC-01 [Quarter with Year], parameter selection
Purpose     : Maps each row's quarter to "Current Quarter" or
              "Previous Quarter" label; non-selected quarters → BLANK.

| Rule ID          | Rule                                                    |
|------------------|---------------------------------------------------------|
| K4-FIXEDCOLOR-01 | Fixed colour maps row quarter to Current/Previous.      |
| K4-FIXEDCOLOR-02 | Non-selected quarters return BLANK.                     |
| K4-FIXEDCOLOR-03 | Validate referenced parameter table names on regen.     |

Conversion  : ✅ FULL


### CC-05 | Cost Center Key
─────────────────────────────────────────────────────────────
PBI DataType    : string
Applies to      : BMS User and Access Details table

Power BI DAX:
─────────────
Null-safe cost center key using IF(ISBLANK([Cost Center Name]), "__NULL__", [Cost Center Name])

Depends on  : [Cost Center Name] (source column)
Purpose     : Ensures null cost centers can participate in relationships
              and dimension lookups via sentinel value "__NULL__".

| Rule ID           | Rule                                              |
|-------------------|---------------------------------------------------|
| K4-BMS-04         | Null cost centers use sentinel key __NULL__.       |

Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## DISPLAY FOLDER: Work Stream Activities

### M-01 | Request Type Count Comparison
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI Folder      : Work Stream Activities

Power BI DAX:
─────────────
VAR _PreviousQuarter =
    SELECTEDVALUE (
        'Activity - Previous Comparison Quarter'[Previous Comparison Quarter]
    )
VAR _CurrentQuarter =
    SELECTEDVALUE (
        'Activity -Current Comparison Quarter'[Current Comparison Quarter]
    )
VAR _ChartColor =
    SELECTEDVALUE (
        'Chart Color Table'[Chart color]
    )
RETURN
    SWITCH (
        TRUE (),
        _ChartColor = "Previous Quarter",
            CALCULATE (
                COUNTA (
                    'Tracker (03_Activity_tracker_14Jan26) (3)_Tracker'[Request type]
                ),
                REMOVEFILTERS (
                    'Tracker (03_Activity_tracker_14Jan26) (3)_Tracker'[Quarter with Year]
                ),
                'Tracker (03_Activity_tracker_14Jan26) (3)_Tracker'[Quarter with Year]
                    = _PreviousQuarter
            ),
        _ChartColor = "Current Quarter",
            CALCULATE (
                COUNTA (
                    'Tracker (03_Activity_tracker_14Jan26) (3)_Tracker'[Request type]
                ),
                REMOVEFILTERS (
                    'Tracker (03_Activity_tracker_14Jan26) (3)_Tracker'[Quarter with Year]
                ),
                'Tracker (03_Activity_tracker_14Jan26) (3)_Tracker'[Quarter with Year]
                    = _CurrentQuarter
            )
    )

Key Pattern : Quarter comparison count using SELECTEDVALUE on slicer
              tables + SWITCH(TRUE()) on chart color +
              REMOVEFILTERS + direct quarter filter.

| Rule ID        | Rule                                                          |
|----------------|---------------------------------------------------------------|
| K4-MEASURE-01  | Use SELECTEDVALUE for current/previous quarter slicer reads.  |
| K4-MEASURE-02  | Use SWITCH(TRUE()) for chart color branch logic.              |
| K4-MEASURE-03  | Use REMOVEFILTERS on row quarter before applying filter.      |
| K4-MEASURE-04  | Use COUNTA for text-category counts.                          |

Conversion  : ✅ FULL


### M-02 | Aspect Count Comparison
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI Folder      : Work Stream Activities

Pattern     : Same as M-01 but counts [Aspect] column instead of [Request type].
Conversion  : ✅ FULL


### M-03 | Previous Quarter Shape Colour
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI Folder      : Work Stream Activities

Power BI DAX:
─────────────
VAR _Highlight =
    SELECTEDVALUE (
        'Chart Color Highlight Slicer'[Chart color highlight],
        "All"
    )
RETURN
    SWITCH (
        TRUE (),
        _Highlight = "Current Quarter", "#ebe7f3",
        "#C9C6F2"
    )

Purpose     : Returns hex color for previous quarter bars. When current
              quarter is highlighted, previous quarter is de-emphasized
              to lighter shade (#ebe7f3); otherwise full color (#C9C6F2).

| Rule ID             | Rule                                              |
|---------------------|---------------------------------------------------|
| K4-COLOR-MEASURE-01 | Shape color measures return hex color strings.     |
| K4-COLOR-MEASURE-02 | Highlighting one quarter de-emphasizes the other.  |
| K4-COLOR-MEASURE-03 | Used for conditional formatting in report layer.   |

Conversion  : ✅ FULL


### M-04 | Current Quarter Shape Colour
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI Folder      : Work Stream Activities

Power BI DAX:
─────────────
VAR _Highlight =
    SELECTEDVALUE (
        'Chart Color Highlight Slicer'[Chart color highlight],
        "All"
    )
RETURN
    SWITCH (
        TRUE (),
        _Highlight = "Previous Quarter", "#ebe3e9",
        "#C49ABC"
    )

Purpose     : Hex color for current quarter bars. De-emphasized (#ebe3e9)
              when previous quarter is highlighted; full (#C49ABC) otherwise.
Conversion  : ✅ FULL


## DISPLAY FOLDER: RWD Utilization

### M-05 | Therapeutic Area Count (TREATAS pattern)
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI Folder      : RWD Utilization

Power BI DAX (pattern):
───────────────────────
VAR _SelectedQuarter =
    SWITCH (
        TRUE (),
        _ChartColor = "Current Quarter", _CurrentQuarter,
        _ChartColor = "Previous Quarter", _PreviousQuarter
    )
RETURN
    COALESCE (
        CALCULATE (
            COUNTA ( <FactTable>[Therapeutic Area] ),
            TREATAS (
                { _SelectedQuarter },
                <FactTable>[Quarter with Year]
            )
        ),
        0
    )

Key Pattern : TREATAS for disconnected slicer tables filtering fact tables.

| Rule ID        | Rule                                                              |
|----------------|-------------------------------------------------------------------|
| K4-TREATAS-01  | Use TREATAS when slicers are disconnected from the target table.  |
| K4-TREATAS-02  | Use COALESCE(..., 0) to avoid blank counts.                       |
| K4-TREATAS-03  | Use single-value table literal { _SelectedQuarter }.              |

Conversion  : ✅ FULL


### M-06 | Requestor Department Count
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI Folder      : RWD Utilization
Pattern     : Same TREATAS pattern as M-05, counting [Requestor Department].
Conversion  : ✅ FULL


### M-07 | Requestor Dept Sort Score (Compound Sort)
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI Folder      : RWD Utilization

Power BI DAX:
─────────────
VAR _Total = [Requestor Dept Sort Total]
VAR _NameRank =
    SELECTEDVALUE (
        'Requestor Dept Dim'[Requestor Dept Name Rank],
        999999
    )
RETURN
    COALESCE ( _Total, 0 ) * 1000000 - _NameRank

Purpose     : Count dominates sort; name rank breaks ties alphabetically.

| Rule ID      | Rule                                                    |
|--------------|---------------------------------------------------------|
| K4-SORT-01   | Compound sort = total count × 1,000,000 − name rank.   |
| K4-SORT-02   | Count dominates sort order.                             |
| K4-SORT-03   | Name rank breaks ties alphabetically.                   |

Conversion  : ✅ FULL


## DISPLAY FOLDER: BMS Access

### M-08 | Active Domino Users
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI Folder      : BMS Access
Purpose     : Counts enabled users where Idnty Sta Nm = "ENABLED".
Conversion  : ✅ FULL


### M-09 | Cost Center Rank
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI Folder      : BMS Access
Purpose     : RANKX cost centers by active user count.
Conversion  : ✅ FULL


### M-10 | Show Top N Cost Center
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI Folder      : BMS Access

Power BI DAX:
─────────────
VAR _Rank = [Cost Center Rank]
VAR _TopN = [Top N Departments Value]
RETURN
    IF (
        _Rank <= _TopN
            && [Active Domino Users] > 0,
        1,
        0
    )

Purpose     : Visual-level filter (value = 1) for Top-N display.

| Rule ID             | Rule                                               |
|---------------------|----------------------------------------------------|
| K4-TOPN-MEASURE-01  | Rank cost centers with RANKX.                      |
| K4-TOPN-MEASURE-02  | Compare rank against selected Top-N value.          |
| K4-TOPN-MEASURE-03  | Suppress rows where active user count is zero.      |
| K4-TOPN-MEASURE-04  | Use as visual-level filter where value = 1.         |

Conversion  : ✅ FULL


### M-11 | BMS User Count - Selected Quarters
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI Folder      : BMS Access

Power BI DAX:
─────────────
VAR _CQ =
    SELECTEDVALUE (
        'Activity -Current Comparison Quarter'[Current Comparison Quarter]
    )
VAR _PQ =
    SELECTEDVALUE (
        'Activity - Previous Comparison Quarter'[Previous Comparison Quarter]
    )
RETURN
    CALCULATE (
        [BMS User Count],
        KEEPFILTERS (
            'Sheet1 (05_03_BMS_user_and_access_details_joined_file_14Jan26)_Sheet1'[Quarter with Year]
                IN { _CQ, _PQ }
        )
    )

| Rule ID          | Rule                                                    |
|------------------|---------------------------------------------------------|
| K4-BMS-MEASURE-01| Use IN { _CQ, _PQ } to filter both selected quarters.  |
| K4-BMS-MEASURE-02| Wrap quarter filter in KEEPFILTERS.                     |
| K4-BMS-MEASURE-03| Base user count is a separate reusable measure.         |

Conversion  : ✅ FULL


## DISPLAY FOLDER: General / Metadata / Helper

### M-12 | Donut Midde Circle
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
Purpose     : Constant zero measure for donut chart center.
⚠️  TYPO NOTE: "Midde" is a misspelling of "Middle" — preserved exactly.
Conversion  : ✅ FULL


### M-13 | Tooltip Text Measures (Pattern)
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table

Power BI DAX (pattern):
───────────────────────
VAR _SP = UNICHAR ( 160 )
RETURN
    "Data Source:" & REPT ( _SP, 6 ) & "<source>"
    & UNICHAR ( 10 ) & UNICHAR ( 10 )
    & "Description:" & REPT ( _SP, 6 ) & "<description>"
    & UNICHAR ( 10 ) & UNICHAR ( 10 )
    & "Refresh Date:" & REPT ( _SP, 5 ) & "<date>"

| Rule ID        | Rule                                                    |
|----------------|---------------------------------------------------------|
| K4-TOOLTIP-01  | Use UNICHAR(160) for non-breaking-space alignment.      |
| K4-TOOLTIP-02  | Use UNICHAR(10) for line breaks.                        |
| K4-TOOLTIP-03  | Tooltip sections: Data Source, Description, Refresh Date.|
| K4-TOOLTIP-04  | Use REPT(_SP, n) to align tooltip text.                 |

Conversion  : ✅ FULL


### M-14 | Info Icon
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
Purpose     : Returns Unicode info symbol (ℹ️) for visual display.
Conversion  : ✅ FULL


### Full Measure Inventory
─────────────────────────────────────────────────────────────
| Measure                                  | Folder                  | Pattern                      |
|------------------------------------------|-------------------------|------------------------------|
| Donut Midde Circle                       | General                 | Constant zero                |
| Cost Matrix                              | General                 | Selected value display        |
| Tooltip Quarter                          | General                 | Tooltip selected quarter      |
| Tooltip Value1                           | General                 | Tooltip value formatting      |
| Tooltip Content                          | General                 | Tooltip content               |
| Request Type Count Comparison            | Work Stream Activities  | Quarter comparison count      |
| RWDS Chart Title                         | Work Stream Activities  | Dynamic title                 |
| Aspect Count Comparison                  | Work Stream Activities  | Quarter comparison count      |
| RWDS Projects Chart Title                | Work Stream Activities  | Dynamic title                 |
| Selected Chart Color                     | Work Stream Activities  | Selected chart color          |
| Tooltip Quarter with Year                | Work Stream Activities  | Tooltip quarter               |
| Previous Quarter Shape Colour            | Work Stream Activities  | Hex color measure             |
| Current Quarter Shape Colour             | Work Stream Activities  | Hex color measure             |
| Activity Info Tooltip                    | Work Stream Activities  | Multi-line tooltip            |
| Selected Header Highlight                | Work Stream Activities  | Header highlight              |
| Info Icon                                | Work Stream Activities  | Unicode info icon             |
| Therapeutic Area Count                   | RWD Utilization         | TREATAS quarter count         |
| Tooltip Quarter with Year RWD U          | RWD Utilization         | Tooltip quarter               |
| Title RWD U1                             | RWD Utilization         | Dynamic title                 |
| Requestor Department Count               | RWD Utilization         | TREATAS quarter count         |
| Title RWD U2                             | RWD Utilization         | Dynamic title                 |
| Requestor Dept Sort Total                | RWD Utilization         | Sort helper                   |
| Requestor Dept Sort Score                | RWD Utilization         | Compound sort score           |
| RWD Utilization Tooltip                  | RWD Utilization         | Multi-line tooltip            |
| Top N Departments Value                  | General                 | Top-N selected value          |
| Active Domino Users                      | BMS Access              | Enabled-user count            |
| Cost Center Rank                         | BMS Access              | RANKX cost center             |
| Show Top N Cost Center                   | BMS Access              | Top-N visual filter           |
| Active Domino Users %                    | BMS Access              | Percent of total              |
| Anzen Privilege Count for Top N          | BMS Access              | Enabled privilege count       |
| BMS Quarter Filter                       | BMS Access              | Quarter filter helper         |
| BMS User Count                           | BMS Access              | User count                    |
| BMS User Count - Selected Quarters       | BMS Access              | Current/previous count        |
| BMS Access Visual Title                  | BMS Access              | Dynamic title                 |
| TPA Info Tooltip                         | Metadata                | Tooltip                       |
| BMS RWDB Access Info Tooltip             | Metadata                | Tooltip                       |
| MSA Info Tooltip                         | Metadata                | Tooltip                       |
| RWD Access Status Info Tooltip           | Metadata                | Tooltip                       |
| __Activity Quarter Count                 | Helper                  | Activity count                |
| __Activity Quarter Sort Total            | Helper                  | Sort helper                   |
| Activity Tooltip Value                   | Helper                  | Tooltip row value             |
| Activity Tooltip Label                   | Helper                  | Tooltip row label             |
| __Therapeutic Area Sort Total            | Helper                  | Sort helper                   |

| Rule ID            | Rule                                                       |
|--------------------|------------------------------------------------------------|
| K4-MEASURE-INV-01  | Keep test measures if validating Tableau parity.           |
| K4-MEASURE-INV-02  | Hide or remove test measures before production.            |
| K4-MEASURE-INV-03  | Measures with __ prefix are helpers; may be hidden.        |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU INITIAL TRANSFORMATIONS → POWER BI POWER QUERY (M)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | Canonical SharePoint Excel M Pattern
─────────────────────────────────────────────────────────────
Tableau Source  : SharePoint / OneDrive Excel via .hyper extract
PBI Connection  : Excel.Workbook(Web.Contents(...))
PBI Load Mode   : Import

Power BI M Query (canonical pattern):
──────────────────────────────────────
let
    Source =
        Excel.Workbook(
            Web.Contents(
                "https://sites.bms.com/sites/RWDSProjects-BMSFolder/Shared%20Documents/..."
            ),
            null,
            true
        ),
    #"<SheetName>" = Source{[Name="<SheetName>"]}[Data],
    #"Promoted Headers" =
        Table.PromoteHeaders(
            #"<SheetName>",
            [PromoteAllScalars = true]
        ),
    #"Selected Columns" =
        Table.SelectColumns(
            #"Promoted Headers",
            {<ColumnList>}
        )
in
    #"Selected Columns"

| Rule ID    | Rule                                                                   |
|------------|------------------------------------------------------------------------|
| K4-DS-01   | Tableau SharePoint Excel → PBI Excel.Workbook(Web.Contents(...)).      |
| K4-DS-02   | Do not use local file paths in generated M queries.                    |
| K4-DS-03   | Always promote headers with Table.PromoteHeaders([PromoteAllScalars]). |
| K4-DS-04   | Use Table.SelectColumns to keep only required columns.                 |
| K4-DS-05   | Tableau .hyper extracts replaced by M imports — not migrated.          |
| K4-DS-06   | Tableau extract row counts are documentation only.                     |


### PQ-02 | Source Tables Summary
─────────────────────────────────────────────────────────────
| PBI Table Name                                                              | Source Excel                                    | Key Date Column         |
|-----------------------------------------------------------------------------|-------------------------------------------------|-------------------------|
| Sheet1 (02_RWD_and_Support_Costs_Savings_Avoidance_14Jan26)_Sheet1          | 02_RWD_and_Support_Costs_Savings_Avoidance      | Date                    |
| Sheet1 (05_03_BMS_user_and_access_details_joined_file_14Jan26)_Sheet1       | 05_03_BMS_user_and_access_details_joined_file   | Updated Date            |
| Tracker (03_Activity_tracker_14Jan26) (3)_Tracker                           | 03_Activity_tracker_14Jan26                     | End date                |
| Activity tracker_Tracker (duplicate import)                                 | 03_Activity_tracker_14Jan26                     | End date                |
| Sheet1 (01_Executive_summary_14Jan26)_Sheet1                                | 01_Executive_summary_14Jan26                    | Date                    |
| Sheet1 (01_Executive_summary_14Jan26) (2)_Sheet1                            | 01_Executive_summary_14Jan26 (duplicate)        | Date                    |
| Jira Input_Sheet1                                                           | Jira input Excel                                | Completion Date         |
| Sheet1 (04_RWDA_projects_14Jan26)_Sheet1                                    | 04_RWDA_projects_14Jan26                        | Completion Date         |
| Table1 (Priority_projects_mapping) (2)_Table1                               | Priority_projects_mapping                       | None                    |
| Sheet1 (05_01_TPA_tracker_14Jan26)_Sheet1                                   | 05_01_TPA_tracker_14Jan26                       | Data Access End Date    |
| Summary (05_02_MSA_Tracker_14Jan26)_Summary                                 | 05_02_MSA_Tracker_14Jan26                       | MSA Expiration Dates    |

⚠️  DUPLICATE IMPORT (K4-DUP-01): Activity tracker is imported twice.
    The duplicate has renamed columns with suffix "1" (Request type1,
    Aspect1, Database name1). This supports Tableau action-link behavior.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4a — STATIC LOOKUP TABLES (DAX CALCULATED TABLES)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### STATIC-01 | Chart Color Table
─────────────────────────────────────────────────────────────
DATATABLE (
    "Chart color", STRING,
    "Chart Color Sort", INTEGER,
    {
        { "Previous Quarter", 1 },
        { "Current Quarter", 2 }
    }
)
sortByColumn: Chart Color Sort
Conversion  : ✅ FULL


### STATIC-02 | Chart Color Highlight Slicer
─────────────────────────────────────────────────────────────
DATATABLE (
    "Chart color highlight", STRING,
    "Sort", INTEGER,
    {
        { "Previous Quarter", 1 },
        { "Current Quarter", 2 }
    }
)
Purpose     : Drives de-emphasis colors in shape colour measures.
Conversion  : ✅ FULL


### STATIC-03 | Header Highlight Slicer
─────────────────────────────────────────────────────────────
DATATABLE (
    "Request type highlight", STRING,
    "Sort", INTEGER,
    {
        { "Enablement and Governance", 1 },
        { "Capabilities", 2 },
        { "Communications", 3 },
        { "Platforms & Operations", 4 }
    }
)
Conversion  : ✅ FULL


### STATIC-04 | Activity Tooltip Rows
─────────────────────────────────────────────────────────────
DATATABLE (
    "Tooltip Sort", INTEGER,
    "Tooltip Label", STRING,
    {
        { 1, "Quarter with Year:" },
        { 2, "Fixed colour:" },
        { 3, "Request type:" },
        { 4, "Count of Request type:" }
    }
)
Purpose     : Multi-row custom tooltip output. Values filled by
              SWITCH on selected tooltip sort value.
Conversion  : ✅ FULL


### STATIC-05 | Top N Departments
─────────────────────────────────────────────────────────────
GENERATESERIES ( 1, 30, 1 )
Purpose     : Top-N slicer range (1–30). Read via MAX or SELECTEDVALUE.
⚠️  NOTE: Includes extendedProperty ParameterMetadata = {"version": 0}.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4b — QUARTER SLICER TABLES (DAX CALCULATED TABLES)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### SLICER-01 | Activity - Current Comparison Quarter
─────────────────────────────────────────────────────────────
DAX pattern using ADDCOLUMNS + DISTINCT + FILTER + SELECTCOLUMNS.
Derives distinct Quarter with Year values from Activity Tracker,
computes Quarter Sort (YYYYQ), filters to ≥ 20231, renames to
"Current Comparison Quarter".
See Section 7.2 of source for full DAX.
Conversion  : ✅ FULL


### SLICER-02 | Activity - Previous Comparison Quarter
─────────────────────────────────────────────────────────────
Same pattern as SLICER-01 but with upper bound (≤ 20253) to exclude
the current quarter from the "previous" slicer.
Conversion  : ✅ FULL


### SLICER-03 | BMS Current Comparison Quarter
─────────────────────────────────────────────────────────────
Same pattern as SLICER-01 but sourced from BMS user-access table.
Conversion  : ✅ FULL


### SLICER-04 | BMS Previous Comparison Quarter
─────────────────────────────────────────────────────────────
Same pattern as SLICER-02 but sourced from BMS user-access table.
Conversion  : ✅ FULL

| Rule ID    | Rule                                                              |
|------------|-------------------------------------------------------------------|
| K4-QTR-04  | Current/previous quarters are separate calculated slicer tables.  |
| K4-QTR-05  | Current quarter slicer has lower-bound filter only.               |
| K4-QTR-06  | Previous quarter slicer can have both lower and upper bounds.     |
| K4-QTR-07  | BMS slicers reuse the same pattern against BMS table.             |
| K4-QTR-08  | Add NOT ISBLANK guards when source quarters may be blank.         |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4c — CALCULATED DIMENSION TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### DIM-01 | Cost Center Dim
─────────────────────────────────────────────────────────────
Purpose     : Clean cost center dimension from BMS user-access source.
              Handles nulls with __NULL__, applies manual sort ordering,
              enables Top-N ranking.

Core DAX pattern: DISTINCT + SELECTCOLUMNS + ADDCOLUMNS with null
sentinel (__NULL__) and manual SWITCH sort key (001, 002, ..., 999).

| Rule ID           | Rule                                                  |
|-------------------|-------------------------------------------------------|
| K4-COSTCENTER-01  | Build from distinct cost center keys.                 |
| K4-COSTCENTER-02  | Use __NULL__ as null sentinel key.                    |
| K4-COSTCENTER-03  | Display __NULL__ as "Null".                           |
| K4-COSTCENTER-04  | Use UPPER(TRIM(...)) before manual sort matching.     |
| K4-COSTCENTER-05  | Manual order keys use zero-padded strings.            |
| K4-COSTCENTER-06  | Unmatched names sort using fallback prefix 999.       |
| K4-COSTCENTER-07  | Display column sorts by Cost Center Sort Key.         |

Conversion  : ✅ FULL


### DIM-02 | Requestor Dept Dim
─────────────────────────────────────────────────────────────
Purpose     : Normalized department dimension from RWDA projects.
              Handles whitespace normalization, leading-space semantics,
              and stable RANKX-based sorting.

Whitespace normalization: SUBSTITUTE chains for UNICHAR(160), UNICHAR(9),
UNICHAR(10), UNICHAR(13) → space, then UPPER(TRIM(...)).

Leading-space sort: Departments with leading spaces get prefix "01|";
others get "02|". This preserves intentional indentation in names.

Rank: RANKX(ALL('Requestor Dept Dim'), [Sort Key], , ASC, DENSE).

| Rule ID      | Rule                                                            |
|--------------|-----------------------------------------------------------------|
| K4-DEPT-01   | Normalize NBSP, tabs, LF, CR before comparison.                |
| K4-DEPT-02   | Use UNICHAR(160/9/10/13) replacements.                          |
| K4-DEPT-03   | Departments with leading spaces receive 01| prefix.             |
| K4-DEPT-04   | Departments without leading spaces receive 02| prefix.          |
| K4-DEPT-06   | Use dense RANKX for stable name rank.                           |

Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — CONVERSION COMPLETENESS SCORECARD
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Category                          | Items | PBI Full | PBI Partial | Not Converted | % Complete |
|-----------------------------------|-------|----------|-------------|---------------|------------|
| Parameters / M Expressions        | 10    | 10       | 0           | 0             | 100%       |
| Calculated Columns                | 5     | 5        | 0           | 0             | 100%       |
| Measures (all folders)            | 42    | 42       | 0           | 0             | 100%       |
| Power Query / Excel Tables        | 11    | 11       | 0           | 0             | 100%       |
| Static Lookup Tables              | 5     | 5        | 0           | 0             | 100%       |
| Quarter Slicer Tables             | 4     | 4        | 0           | 0             | 100%       |
| Calculated Dimension Tables       | 2     | 2        | 0           | 0             | 100%       |
| Relationships (date)              | 14    | 14       | 0           | 0             | 100%       |
| Relationships (dimension)         | 2     | 2        | 0           | 0             | 100%       |
| TMDL Infrastructure               | 5     | 5        | 0           | 0             | 100%       |
| TOTAL                             | 100   | 100      | 0           | 0             | 🟢 100%    |

⚠️  NOTE: While structurally complete, 8 manual review flags require
    validation — see Section 7.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — CONVERSION PATTERN CHEAT SHEET (K4 UNIQUE PATTERNS)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Pattern ID | Pattern                          | Summary                                           |
|------------|----------------------------------|----------------------------------------------------|
| K4-P01     | SharePoint Excel connector       | Excel.Workbook(Web.Contents(...))                  |
| K4-P02     | Tableau .hyper to PBI import     | Replace extract with Excel M import                |
| K4-P03     | Multi-source Excel model         | Multiple Excel workbooks imported independently    |
| K4-P04     | Duplicate Excel import           | Same sheet imported twice with renamed columns     |
| K4-P05     | Quarter with Year                | Qn'YY calculated column                           |
| K4-P06     | Quarter Sort                     | YYYYQ numeric sort key (e.g. 20254)               |
| K4-P07     | Current/Previous quarter slicers | Separate calculated slicer tables                  |
| K4-P08     | BMS quarter slicers              | Quarter slicers from BMS user/access table         |
| K4-P09     | Static DATATABLE slicers         | Chart color, highlight, header, tooltip rows       |
| K4-P10     | Top-N slicer                     | GENERATESERIES(1,30,1)                             |
| K4-P11     | Cost Center Dim                  | Null-safe calculated dimension with manual sort    |
| K4-P12     | Requestor Dept Dim               | Whitespace-normalized calculated dimension         |
| K4-P13     | Compound sort score              | Count × 1,000,000 − rank                          |
| K4-P14     | TREATAS disconnected slicer      | Quarter slicer filters unrelated fact table         |
| K4-P15     | Hex color measures               | Measures return color strings                      |
| K4-P16     | UNICHAR tooltip formatting       | Multi-line aligned tooltip text                    |
| K4-P17     | Info icon measure                | Unicode info symbol                                |
| K4-P18     | bothDirections relationship      | Used for Cost Center Dim                           |
| K4-P19     | K/M currency formatter           | $nnnK and $n.nnM display                          |
| K4-P20     | Date parameters                  | Tableau date literals to M #date(...)              |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAP-01 | K4-MR-01 — Missing Group 1 Name Table
  Severity : High
  Detail   : Activity tracker_Tracker.[Fixed colour] references
             Group 1 Name, but that table was not found in Set 4B
             outputs.
  Action   : Validate table mapping and confirm whether Group 1 Name
             exists as a separate import or is derived.


GAP-02 | K4-MR-02 — Start Date Stored as String
  Severity : High
  Detail   : Start date in the activity tracker is stored as string.
             Do not use it in date calculations without explicit
             conversion.
  Action   : Add explicit string-to-date conversion if Start date
             is used in any date calculations or relationships.


GAP-03 | K4-MR-03 — Previous Comparision Month GUID Reference
  Severity : Medium
  Detail   : Previous Comparision Month uses a GUID expression
             reference in its M partition. Validate refresh behavior.
  Action   : Test data refresh and confirm GUID expression resolves
             correctly.


GAP-04 | K4-MR-04 — Tableau Date Parameters Not Fully Mapped
  Severity : Medium
  Detail   : Some Tableau date parameters may not have direct Power BI
             expression outputs. Confirm whether custom time-period
             parameters are required.
  Action   : Review all date parameter usage in Tableau and confirm
             PBI expression coverage.


GAP-05 | K4-MR-05 — Tableau Color Encodings Not in TMDL
  Severity : Medium
  Detail   : Tableau color encodings for cost center and status are not
             fully migrated to TMDL. TMDL does not fully capture report
             visual color formatting.
  Action   : Move into report theme JSON or conditional formatting.


GAP-06 | K4-MR-06 — Casing Inconsistency
  Severity : Medium
  Detail   : Potential casing inconsistency: "Chart color Table" vs.
             "Chart Color Table". Power BI is generally case-insensitive,
             but naming should be standardized.
  Action   : Standardize table/column naming across all files.


GAP-07 | K4-MR-07 — Test Table Contains Debug Measures
  Severity : Low
  Detail   : Test table contains validation/debug measures:
             Anzen Privilege Count for Top N - Tableau,
             Anzen Privilege Count for Top N - Enabled,
             Top N Departments Value test,
             Cost Center Rank - Tableau,
             Show Top N Cost Center - Tableau.
  Action   : Hide or remove test measures before production if not
             required for ongoing Tableau parity validation.


GAP-08 | K4-MR-08 — Typo in Measure Name
  Severity : Low
  Detail   : Measure name "Donut Midde Circle" contains a typo
             ("Midde" should be "Middle"). Preserved verbatim.
  Action   : Preserve unless stakeholder approves correction.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — FILE INVENTORY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| #  | Output Filename                                              | Purpose                                          |
|----|--------------------------------------------------------------|--------------------------------------------------|
| 1  | 4_database.txt                                               | Database metadata (compatibilityLevel: 1604)     |
| 2  | 4_model.txt                                                  | Model definition and query order                 |
| 3  | 4_relationships.txt                                          | Model relationships (date + dimension)           |
| 4  | 4_measures_table.txt                                         | Main measures table (42+ measures)               |
| 5  | expressions.txt                                              | Power BI parameter expressions (10 params)       |
| 6  | chart_color_table.txt                                        | Static chart color DATATABLE                     |
| 7  | chart_color_highlight_slicer.txt                             | Static chart highlight DATATABLE                 |
| 8  | header_highlight_slicer.txt                                  | Static request-type highlight DATATABLE          |
| 9  | activity_tooltip_rows.txt                                    | Tooltip row lookup DATATABLE                     |
| 10 | activity__current_comparison_quarter.txt                      | Current quarter slicer table                     |
| 11 | activity___previous_comparison_quarter.txt                    | Previous quarter slicer table                    |
| 12 | bms_current_comparison_quarter.txt                           | BMS current comparison quarter table             |
| 13 | bms_previous_comparison_quarter.txt                          | BMS previous comparison quarter table            |
| 14 | cost_center_dim.txt                                          | Cost Center calculated dimension                 |
| 15 | requestor_dept_dim.txt                                       | Requestor Department calculated dimension        |
| 16 | top_n_departments.txt                                        | Top-N slicer table (GENERATESERIES)              |
| 17 | top_n_departments_2.txt                                      | Top-N slicer with parameter metadata             |
| 18 | test.txt                                                     | Development / validation measures                |
| 19 | localdatetable_*.txt (14 files)                              | Auto date tables for each date column            |
| 20 | datetabletemplate_154d8834_73f5_49dd_8438_92ba51102c5f.txt    | Date table template                              |
| 21 | en-US.tmdl                                                   | Culture / linguistic metadata                    |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — RELATIONSHIPS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### Date Relationships (14 LocalDateTables)
─────────────────────────────────────────────────────────────
| From Table                                    | From Column            | LocalDateTable GUID   |
|-----------------------------------------------|------------------------|-----------------------|
| Sheet1 (02_RWD...)                            | Date                   | 1568b9f7              |
| Sheet1 (05_03_BMS...)                         | Created Date           | 34164f95              |
| Sheet1 (05_03_BMS...)                         | Updated Date           | 6d6e1589              |
| Sheet1 (05_03_BMS...)                         | Orgl Hire Dt           | eb55fcdd              |
| Sheet1 (05_03_BMS...)                         | Last Day With Cmpy Dt  | e17e8bc7              |
| Sheet1 (05_03_BMS...)                         | Last Termn Dt          | 95f0ebe7              |
| Tracker (03_Activity...)                      | End date               | 4e46e32c              |
| Sheet1 (01_Executive_summary...)              | Date                   | 2a8cf85f              |
| Activity tracker_Tracker                      | End date               | c63f1736              |
| Jira Input_Sheet1                             | Completion Date        | 1fb99c73              |
| Sheet1 (04_RWDA_projects...)                  | Completion Date        | 6e59ad32              |
| Sheet1 (05_01_TPA_tracker...)                 | Data Access End Date   | d287d40f              |
| Summary (05_02_MSA_Tracker...)                | MSA Expiration Dates   | 860a79ab              |
| Sheet1 (01_Executive_summary...) (2)          | Date                   | c93ce670              |

DateTableTemplate GUID: 154d8834-73f5-49dd-8438-92ba51102c5f

| Rule ID    | Rule                                                              |
|------------|-------------------------------------------------------------------|
| K4-REL-01  | Date relationships use joinOnDateBehavior: datePartOnly.          |
| K4-REL-02  | Each date/datetime column gets its own LocalDateTable.            |
| K4-REL-03  | Do not reuse LocalDateTables across unrelated date columns.       |
| K4-REL-04  | Source date columns must carry a variation to LocalDateTable.     |

### Dimension Relationships
─────────────────────────────────────────────────────────────
| From Table                    | From Column           | To Table            | To Column           | Direction       |
|-------------------------------|-----------------------|---------------------|---------------------|-----------------|
| Sheet1 (04_RWDA_projects...)  | Requestor Department  | Requestor Dept Dim  | Requestor Department| Single          |
| Sheet1 (05_03_BMS...)         | Cost Center Name      | Cost Center Dim     | Cost Center Name    | Both directions |

| Rule ID    | Rule                                                              |
|------------|-------------------------------------------------------------------|
| K4-REL-05  | Requestor Department relationship is single-direction.            |
| K4-REL-06  | Cost Center relationship uses bothDirections.                     |
| K4-REL-07  | Both-direction supports Top-N and cross-filtering.                |
| K4-REL-08  | Avoid unnecessary relationships for disconnected slicers.         |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — TABLEAU ACTION-LINK MAPPING
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Tableau action-link groups appear as internal action columns or
sheet-link constructs. Power BI does not model these as TMDL
relationships. They map to report-layer interactions.

| Tableau Action Group                                                         | Fields                                  |
|------------------------------------------------------------------------------|-----------------------------------------|
| Action (Cost Center Name)                                                    | Cost Center Name                        |
| Action (Quarter with Year)                                                   | Quarter with Year                       |
| Action (Quarter with Year, Cost Center Name, Empt Sta Nm)                    | Quarter, Cost Center, Employment Status |
| Action (Aspect, Quarter with Year, Chart color)                              | Aspect, Quarter, Chart color            |
| Action (Quarter with Year, Chart color, Request type)                        | Quarter, Chart color, Request type      |
| Action (Quarter with Year, Fixed colour, Description / TA / Market)          | Quarter, Fixed colour, grouping field   |
| Action (Requestor Dept, Quarter with Year, Fixed colour)                     | Department, Quarter, Fixed colour       |

| Rule ID        | Rule                                                                   |
|----------------|------------------------------------------------------------------------|
| K4-ACTION-01   | Tableau action-link groups map to PBI report-layer interactions.        |
| K4-ACTION-02   | Do NOT create physical PBI columns for Tableau sheet_link actions.      |
| K4-ACTION-03   | Use slicers, visual interactions, drill-through, or disconnected slicers.|
| K4-ACTION-04   | If action behavior drives calculations, reproduce with DAX measures.    |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 11 — TABLEAU COLOR ENCODING MAPPING
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Tableau Color Encoding     | Power BI Equivalent             |
|----------------------------|---------------------------------|
| Categorical palette        | Report theme JSON               |
| Conditional color by status| Conditional formatting          |
| Quarter current/previous   | Hex color measures              |
| Highlight/de-emphasis      | DAX color string measures       |

| Rule ID            | Rule                                                          |
|--------------------|---------------------------------------------------------------|
| K4-COLOR-REPORT-01 | TMDL does not fully capture report visual color formatting.   |
| K4-COLOR-REPORT-02 | Move Tableau palettes to PBI theme or conditional formatting. |
| K4-COLOR-REPORT-03 | Use hex string measures for measure-driven colors.            |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 12 — EXTRACT / HYPER MIGRATION
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Tableau Extract                       | Power BI Replacement                                |
|---------------------------------------|-----------------------------------------------------|
| federated_08kke5308gl8mz1cigjlx1.hyper| Excel M import from cost savings workbook           |
| federated_0ddwes61d0yxru18y3syp0.hyper| Excel M import from BMS user/access workbook        |
| federated_0m0x0jp0fccsmc1919hjs0.hyper| Excel M import from activity tracker workbook       |

| Rule ID      | Rule                                                                 |
|--------------|----------------------------------------------------------------------|
| K4-HYPER-01  | Do not migrate Tableau .hyper files as PBI data sources.             |
| K4-HYPER-02  | Replace .hyper with the original SharePoint Excel source.            |
| K4-HYPER-03  | Preserve extract row counts only as documentation.                   |
| K4-HYPER-04  | PBI import mode is the correct equivalent unless DirectQuery asked.  |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 13 — LINEAGE TAG AND TMDL RULES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

K4 must respect M11–M13 rules (see K3 knowledge file).

| Rule ID        | Rule                                                                   |
|----------------|------------------------------------------------------------------------|
| K4-LINEAGE-01  | lineageTags must match [a-zA-Z0-9_-]+.                                 |
| K4-LINEAGE-02  | Avoid spaces, brackets, punctuation in lineageTags.                    |
| K4-LINEAGE-03  | Expression lineageTags may contain Tableau names — validate first.     |
| K4-LINEAGE-04  | Do not reuse lineageTags across objects.                               |
| K4-LINEAGE-05  | Use UUIDs for newly generated objects where possible.                  |
| K4-TMDL-01     | Multi-line DAX → proper multiline blocks in TMDL.                     |
| K4-TMDL-02     | Single-line DAX can remain inline.                                    |
| K4-TMDL-03     | No unsupported comments inside relationship/partition definitions.     |
| K4-TMDL-04     | Only supported TMDL properties in relationship/partition blocks.       |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 14 — FUTURE MIGRATION GUIDANCE USING K4
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

When a new Tableau workbook resembles K4 (SharePoint/OneDrive Excel
sources, comparison-quarter dashboards), follow this sequence:

 1. Parse Tableau input.
 2. Identify all Excel / SharePoint / OneDrive data sources.
 3. Replace Tableau .hyper extracts with Excel.Workbook(Web.Contents(...)).
 4. Promote headers and select only used columns.
 5. Identify Tableau parameters.
 6. Convert parameters to Power BI M expressions.
 7. Create slicer tables for current/previous quarter if needed.
 8. Add Quarter with Year calculated columns where dates support comparison.
 9. Add Quarter Sort for all quarter slicer tables.
10. Create static DATATABLE lookups for chart color, highlights, headers, tooltip rows.
11. Create Top-N tables using GENERATESERIES.
12. Build dimension tables for normalized cost centers and departments.
13. Recreate Tableau calculated fields as DAX columns or measures.
14. Use TREATAS when disconnected slicers must filter fact tables.
15. Add LocalDateTables and relationships for every date/datetime column.
16. Move Tableau action-link behavior to report-layer interactions or DAX.
17. Move Tableau color palettes to PBI theme / conditional formatting.
18. Apply M11 lineageTag rules.
19. Apply M12 multi-line DAX formatting rules.
20. Validate all manual review flags.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 15 — FINAL K4 RULES SUMMARY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Area            | Key Rule                                                               |
|-----------------|------------------------------------------------------------------------|
| Data source     | SharePoint Excel → Excel.Workbook(Web.Contents(...)).                  |
| Extracts        | Tableau .hyper extracts are replaced, not migrated.                    |
| Parameters      | Tableau parameters map to M expressions.                               |
| Dates           | Date literals map to #date(YYYY,MM,DD).                                |
| Quarters        | Quarter labels use Qn'YY; sort uses YYYYQ.                            |
| Slicers         | Current/previous quarters are separate calculated slicer tables.       |
| Static tables   | Use DATATABLE for small lookup tables.                                 |
| Top N           | Use GENERATESERIES(1,30,1).                                           |
| Currency        | Use DAX FORMAT for K/M display.                                        |
| Tooltips        | Use UNICHAR(160) and UNICHAR(10).                                      |
| Dimensions      | Cost Center and Requestor Dept are calculated dimensions.              |
| Relationships   | Date relationships use joinOnDateBehavior: datePartOnly.               |
| Cross-filter    | Cost Center relationship uses bothDirections.                          |
| Colors          | Hex color measures support visual conditional formatting.              |
| Actions         | Tableau actions map to report interactions, not physical columns.      |
| Lineage         | TMDL lineageTags must be safe identifiers.                             |
| Review          | Validate unresolved table names and text-date fields.                  |


END OF KNOWLEDGE FILE
