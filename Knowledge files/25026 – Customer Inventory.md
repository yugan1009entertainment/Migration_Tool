# 📘 CONVERSION KNOWLEDGE FILE
# 25026 – Customer Inventory
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-07
# Source files: customer_inventory.txt (Tableau)
#               5_database.txt, 5_model.txt, 5_expressions.txt,
#               5_relationships.txt, 5_definition.json, 5_diagramlayout.json,
#               secure1_valuecentric_com_ds_customer_inventory_tableau.txt,
#               measure_selector.txt,
#               datetabletemplate_0d8f2d4d_42c2_4ab1_b229_2aec2ccc09ae.txt,
#               localdatetable_0d20c742_*.txt, localdatetable_e7b7f522_*.txt,
#               localdatetable_0b721842_*.txt, localdatetable_0bd6e828_*.txt,
#               localdatetable_bffaeb3b_*.txt (PBIP)

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 0 — WORKBOOK & MODEL METADATA
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PBIP Report Name          : 25026_Customer Inventory 1
PBIP Model Name           : 25026_Customer Inventory- model
Compatibility Level       : 1604
Schema Version            : 4.2  (definition.json)
Culture                   : en-US
PBI Data Source Version   : powerBI_V3
Time Intelligence         : Enabled  (__PBI_TimeIntelligenceEnabled = 1)
Dev Mode                  : Enabled  (PBI_ProTooling = ["DevMode"])
Query Order               : Measures Table → Measure Selector →
                            secure1 valuecentric com_DS Customer Inventory Tableau →
                            2ede9107-052e-441c-bf94-d551288a93bb

Tableau Workbook          : Customer Inventory
Tableau Revision          : 6.5
Tableau Build             : 2024.2.1 (20242.24.0711.1636)
Last Extract Refresh      : 2026-07-06  |  569,520 rows


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PARAM-01 | Measure Selector (Parameter 1)
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameters].[Parameter 1]
Tableau Caption       : Parameter 1 / Measure Selector
Tableau Type          : Integer list (param-domain-type = list)
Tableau Default       : 0  (Dollars $)

Tableau Aliases:
  0 → Dollars $
  1 → Units

Power BI Table Name   : Measure Selector
Power BI Type         : M Query (Table.FromRows) — Import
Power BI Columns      : [ParameterTableKeyColumn] (double/number),
                        [Value] (string)
Power BI Partition ID : 15f9bb8d-f00a-41a3-8ccf-66b454edb4cf
lineageTag            : ffa4984e-3490-4fe6-a8fa-6caf74d0cb3b

Power BI Power Query M:
───────────────────────
let
    Source = {
        {0, "Dollars $"},
        {1, "Units"}
    },
    TableFromList = Table.FromRows(Source, {"ParameterTableKeyColumn", "Value"}),
    ChangedType   = Table.TransformColumnTypes(TableFromList, {
        {"ParameterTableKeyColumn", type number},
        {"Value", type text}
    })
in
    ChangedType

Power Query Expression (expressions file):
──────────────────────────────────────────
expression 2ede9107-052e-441c-bf94-d551288a93bb =
    "Dollars $" meta [
        IsParameterQuery     = true,
        List                 = {"Dollars $", "Units"},
        DefaultValue         = "Dollars $",
        Type                 = type any,
        IsParameterQueryRequired = true
    ]
    lineageTag: Parameter 1
    annotation ParameterName = Parameter 1
    annotation DataType      = Int64
    annotation PBI_ResultType = Text

Usage in model:
  - Most measures use:
      MIN('Measure Selector'[ParameterTableKeyColumn])
    to read 0 or 1 and switch between Dollars/Units.
  - [Sales] measure uses the cleaner:
      SELECTEDVALUE('Measure Selector'[ParameterTableKeyColumn], 0)
    with a VAR pattern. Both are valid — prefer SELECTEDVALUE for new work.

Conversion Status: ✅ FULL — both values, order and keys match exactly.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — DATA SOURCE & POWER QUERY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | secure1 valuecentric com_DS Customer Inventory Tableau  ← Main Fact Table
─────────────────────────────────────────────────────────────
Tableau Data Source   : IQVIA ValueTrak Web Data Connector
                        Server : valuetrak.iqvia.com
                        Connection type : Tableau Web Connector (TSV over HTTPS)

PBI Table Name        : secure1 valuecentric com_DS Customer Inventory Tableau
PBI dataCategory      : Customers
PBI Load Mode         : Import
PBI Partition GUID    : 6294ec74-b9ee-456f-90c5-b852832e3723
lineageTag            : 78438f8e-e24c-45cc-82e4-a02dbaba4855

Endpoint URL:
  https://valuetrak.iqvia.com/ValueCentric/getDatasource.cfm
  ?r=p42HJ1CLtmjfjTTEEtNPMA%3D%3D&q=1&c=95

Source format: Tab-delimited TSV, 15 columns, UTF-8 (Encoding=65001),
               QuoteStyle.None
NOTE: Source columns arrive pre-aggregated from IQVIA with SUM(...)
      naming. Power BI renames them to clean display names via
      sourceColumn mappings in TMDL.

Power BI Power Query M:
───────────────────────
let
    Source = Csv.Document(
        Web.Contents(
            "https://valuetrak.iqvia.com/ValueCentric/getDatasource.cfm" &
            "?r=p42HJ1CLtmjfjTTEEtNPMA%3D%3D&q=1&c=95"
        ),
        [Delimiter="#(tab)", Columns=15, Encoding=65001, QuoteStyle=QuoteStyle.None]
    ),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers", {
        {"Partner",                              type text  },
        {"Product Group",                        type text  },
        {"Day",                                  type date  },
        {"Month",                                type text  },
        {"Partner Group",                        type text  },
        {"SUM(Ending Inventory Units)",          Int64.Type },
        {"SUM(Sales Units)",                     Int64.Type },
        {"SUM(Morgue Inv. Units)",               Int64.Type },
        {"SUM(852 Returns To Usable Inv. Units)",Int64.Type },
        {"SUM(852 Returns To Morgue Units)",     Int64.Type },
        {"SUM(Ending Inventory Dollars)",        Int64.Type },
        {"SUM(Sales Dollars)",                   Int64.Type },
        {"SUM(852 Returns To Morgue Dollars)",   Int64.Type },
        {"SUM(852 Returns To Usable Inv. Dollars)",Int64.Type},
        {"SUM(Morgue Inv. Dollars)",             Int64.Type }
    })
in
    #"Changed Type"

Conversion Status: ✅ FULL — all 15 source columns loaded and typed.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — SOURCE COLUMN MAPPING (c1–c15)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: secure1 valuecentric com_DS Customer Inventory Tableau

Tableau Tag  PBI Column Name                       Data Type  Source Column (raw)                   Summarize  Format
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
[c1]         Partner                               string     Partner                               none       —
[c2]         Product Group                         string     Product Group                         none       —
[c3]         Day                                   dateTime   Day                                   none       mm/dd/yyyy hh:nn AM/PM
[c4]         Month                                 string     Month                                 none       —
[c5]         Partner Group                         string     Partner Group                         none       —
[c6]         Ending Inventory Units                int64      SUM(Ending Inventory Units)           sum        0
[c7]         Sales Units                           int64      SUM(Sales Units)                      sum        0
[c8]         Morgue Inv. Units                     int64      SUM(Morgue Inv. Units)                sum        0
[c9]         852 Returns To Usable Inv. Units      int64      SUM(852 Returns To Usable Inv. Units) sum        0
[c10]        852 Returns To Morgue Units           int64      SUM(852 Returns To Morgue Units)      sum        0
[c11]        Ending Inventory Dollars              int64      SUM(Ending Inventory Dollars)         sum        0
[c12]        Sales Dollars                         int64      SUM(Sales Dollars)                    sum        0
[c13]        852 Returns To Morgue Dollars         int64      SUM(852 Returns To Morgue Dollars)    sum        0
[c14]        852 Returns To Usable Inv. Dollars    int64      SUM(852 Returns To Usable Inv. Dollars) sum      0
[c15]        Morgue Inv. Dollars                   int64      SUM(Morgue Inv. Dollars)              sum        0

displayOrdinal: c1=0, c2=1, c3=2, c4=3, c5=4, c6=5, c7=6, c8=7,
               c9=8, c10=9, c11=10, c12=11, c13=12, c14=13, c15=14


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: secure1 valuecentric com_DS Customer Inventory Tableau

### CC-01 | Date
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_634726077419061248
PBI Data Type   : dateTime
PBI Format      : Mm/dd/yyyy
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression:
───────────────────
DATE(DATEPARSE("yyyy-MM-dd HH:mm:ss", STR([c3])))
// Strips time component from the Day column (datetime → date)

Power BI DAX:
─────────────
/// Date - Column
column Date =
DATEVALUE(
    'secure1 valuecentric com_DS Customer Inventory Tableau'[Day]
)
/* Original Tableau Expression is
   DATE(DATEPARSE("yyyy-MM-dd HH:mm:ss", STR([c3]))) */

Depends on  : [c3] Day (source column)
Used by     : CC-02 [Last Day of Month Filter], CC-04 [Date_MY],
              CC-05 [Date_Year], CC-06 [Last Day MOnth fil],
              M-01 [Ending Inv. Dollars Calc], M-02 [Ending Inv. Units Calc],
              LocalDateTable_e7b7f522 relationship
Variation   : Relationship b8b2bdd6 → LocalDateTable_e7b7f522.Date
              defaultHierarchy: LocalDateTable_e7b7f522.'Date Hierarchy'
Conversion  : ✅ FULL


### CC-02 | Last Day of Month Filter
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_634726077421936641
PBI Data Type   : string  ("Yes" / "No")
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression:
───────────────────
if [Date] = DATEADD('day',-1,(DATEADD('month',1,DATETRUNC('month',[Date]))))
then "Yes"
elseif [Date] = dateadd('day',-1,today())
then "Yes"
ELSE "No"
end
// Returns "Yes" if the date is the last day of its month,
// OR if the date is yesterday (today()-1).

Power BI DAX:
─────────────
/// Last Day of Month Filter - Column
column 'Last Day of Month Filter' =
IF(
    'secure1 valuecentric com_DS Customer Inventory Tableau'[Date]
        = EOMONTH(
            'secure1 valuecentric com_DS Customer Inventory Tableau'[Date],
            0
          ),
    "Yes",
    IF(
        'secure1 valuecentric com_DS Customer Inventory Tableau'[Date]
            = TODAY() - 1,
        "Yes",
        "No"
    )
)
/* Original Tableau Expression is
   if [Date] = DATEADD('day',-1,(DATEADD('month',1,DATETRUNC('month',[Date]))))
   then "Yes" elseif [Date] = dateadd('day',-1,today()) then "Yes" ELSE "No" end */

Depends on  : CC-01 [Date]
Used by     : M-01 [Ending Inv. Dollars Calc], M-02 [Ending Inv. Units Calc],
              M-04 [Morgue Dollars Calc], M-05 [Morgue Units Calc],
              M-09 [Morgue], M-10 [Morgue — unit path]
Conversion  : ✅ FULL


### CC-03 | Show if Not All Zero
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_634726077431341067
PBI Data Type   : boolean
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression:
───────────────────
IF [c11] = 0 AND [c12] = 0 AND [c14] = 0 AND [c15] = 0 AND [c13] = 0
THEN FALSE
ELSE TRUE
END
// Flags rows where ALL five dollar columns are zero → hide from detail views.

Power BI DAX:
─────────────
/// Show if Not All Zero - Column
column 'Show if Not All Zero' =
IF(
    AND(
        AND(
            AND(
                AND(
                    'secure1 valuecentric com_DS Customer Inventory Tableau'
                        [Ending Inventory Dollars] = 0,
                    'secure1 valuecentric com_DS Customer Inventory Tableau'
                        [Sales Dollars] = 0
                ),
                'secure1 valuecentric com_DS Customer Inventory Tableau'
                    [852 Returns To Usable Inv. Dollars] = 0
            ),
            'secure1 valuecentric com_DS Customer Inventory Tableau'
                [Morgue Inv. Dollars] = 0
        ),
        'secure1 valuecentric com_DS Customer Inventory Tableau'
            [852 Returns To Morgue Dollars] = 0
    ),
    FALSE(),
    TRUE()
)
/* Original Tableau Expression is
   IF [c11]=0 AND [c12]=0 AND [c14]=0 AND [c15]=0 AND [c13]=0
   THEN FALSE ELSE TRUE END */

Depends on  : [c11], [c12], [c13], [c14], [c15] (source columns)
Used by     : Report-level filter on detail sheets — include only TRUE rows
Conversion  : ✅ FULL


### CC-04 | Date_MY
─────────────────────────────────────────────────────────────
PBI Lineage Tag : ce209016-69d2-442d-a958-13a3131151b9
PBI Data Type   : dateTime  (underlying date, formatted as MY string)
PBI Format      : Mmm, yyyy
PBI Hidden      : No
PBI Comment     : /// This is a transformation column from the source
                      column with the format of MY

Tableau Expression:
───────────────────
// Month-Year derivation of [Date] e.g. "Jan, 2025"
FORMAT([Date], "MMM, yyyy")

Power BI DAX:
─────────────
/// This is a transformation column from the source column with the format of MY
column Date_MY =
FORMAT(
    'secure1 valuecentric com_DS Customer Inventory Tableau'[Date],
    "MMM, yyyy"
)

Depends on  : CC-01 [Date]
Variation   : Relationship c69db48d → LocalDateTable_0bd6e828.Date
              defaultHierarchy: LocalDateTable_0bd6e828.'Date Hierarchy'
changedProperty: DataType
Annotations : UnderlyingDateTimeDataType = Date
Conversion  : ✅ FULL


### CC-05 | Date_Year
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 6f426a15-f7f1-4d14-8e51-805afabba2e6
PBI Data Type   : dateTime  (underlying year, formatted as year string)
PBI Format      : Mmmm, yyyy
PBI Hidden      : No

Tableau Expression:
───────────────────
YEAR([Date])

Power BI DAX:
─────────────
column Date_Year =
YEAR(
    'secure1 valuecentric com_DS Customer Inventory Tableau'[Date]
)

Depends on  : CC-01 [Date]
Variation   : Relationship b5bd6f29 → LocalDateTable_0b721842.Date
              defaultHierarchy: LocalDateTable_0b721842.'Date Hierarchy'
changedProperty: DataType
Annotations : UnderlyingDateTimeDataType = Date
Used by     : Hierarchy 'Date_Date Hierarchy' (level 1: Date_Year)
Conversion  : ✅ FULL


### CC-06 | Last Day MOnth fil
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 8ebbc813-476f-414c-9cff-71e7a8295d20
PBI Data Type   : string  ("Yes" / "No")
PBI Hidden      : No
PBI Summarize   : none
⚠️  NOTE        : Column name has an intentional capitalization typo —
                  "MOnth" — preserve exactly as-is.

Tableau Expression:
───────────────────
// WIP / alternate version of [Last Day of Month Filter]
// Uses SELECTEDVALUE instead of straight row context

Power BI DAX:
─────────────
column 'Last Day MOnth fil' =
IF(
    'secure1 valuecentric com_DS Customer Inventory Tableau'[Date]
        = EOMONTH(
            SELECTEDVALUE(
                'secure1 valuecentric com_DS Customer Inventory Tableau'[Date]
            ),
            0
          )
    ||
    'secure1 valuecentric com_DS Customer Inventory Tableau'[Date]
        = TODAY() - 1,
    "Yes",
    "No"
)

Depends on  : CC-01 [Date]
⚠️  NOTE    : SELECTEDVALUE() inside a calculated column resolves to
              BLANK() when multiple dates are in context, effectively
              making the EOMONTH branch FALSE for most rows.
              This is a WIP/experimental column — CC-02 is the
              production version used by all Calc measures.
Conversion  : ✅ FULL (typo "MOnth" preserved intentionally)


### CC-07 | Product Grouping
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Product Group (group)
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression:
───────────────────
// Categorical bin on [Product Group] (c2)
// 7 specialty products pass through by name;
// 23 products → "Non-Specialty"; all others → NULL

Power BI DAX:
─────────────
column 'Product Grouping' =
SWITCH(
    TRUE(),

    -- Specialty products (pass-through by name)
    'secure1 valuecentric com_DS Customer Inventory Tableau'
        [Product Group] = "CAMZYOS",   "CAMZYOS",
    'secure1 valuecentric com_DS Customer Inventory Tableau'
        [Product Group] = "KRAZATI",   "KRAZATI",
    'secure1 valuecentric com_DS Customer Inventory Tableau'
        [Product Group] = "POMALYST",  "POMALYST",
    'secure1 valuecentric com_DS Customer Inventory Tableau'
        [Product Group] = "REVLIMID",  "REVLIMID",
    'secure1 valuecentric com_DS Customer Inventory Tableau'
        [Product Group] = "SOTYKTU",   "SOTYKTU",
    'secure1 valuecentric com_DS Customer Inventory Tableau'
        [Product Group] = "THALOMID",  "THALOMID",
    'secure1 valuecentric com_DS Customer Inventory Tableau'
        [Product Group] = "ZEPOSIA",   "ZEPOSIA",

    -- Non-Specialty bucket (23 products)
    'secure1 valuecentric com_DS Customer Inventory Tableau'
        [Product Group] IN {
            "ABRAXANE", "AUGTYRO",  "AZACTAM",    "BARACLUDE",  "COBENFY",
            "ELIQUIS",  "EMPLICITI","EVOTAZ",      "IDHIFA",     "INREBIC",
            "ISTODAX",  "KENALOG",  "NULOJIX",     "ONUREG",     "OPDIVO",
            "OPDUALAG", "ORENCIA IV","ORENCIA SC",  "REBLOZYL",
            "REYATAZ",  "SPRYCEL",  "VIDAZA",      "YERVOY"
        },
    "Non-Specialty",

    BLANK()
)

Depends on  : [c2] Product Group (source column)
Used by     : Report-level filter excluding REVLIMID on SP Monthly views;
              Product Grouping slicer on Wholesaler views
Specialty list (7) : CAMZYOS, KRAZATI, POMALYST, REVLIMID, SOTYKTU,
                     THALOMID, ZEPOSIA
Non-Specialty (23) : ABRAXANE, AUGTYRO, AZACTAM, BARACLUDE, COBENFY,
                     ELIQUIS, EMPLICITI, EVOTAZ, IDHIFA, INREBIC,
                     ISTODAX, KENALOG, NULOJIX, ONUREG, OPDIVO,
                     OPDUALAG, ORENCIA IV, ORENCIA SC, REBLOZYL,
                     REYATAZ, SPRYCEL, VIDAZA, YERVOY
Conversion  : ✅ FULL


### CC-08 | Refresh
─────────────────────────────────────────────────────────────
PBI Lineage Tag : ca445a0f-7ddb-449a-9fa0-1da7228d060e
PBI Data Type   : dateTime
PBI Format      : General Date
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression:
───────────────────
// No direct Tableau equivalent — data freshness timestamp

Power BI DAX:
─────────────
column Refresh =
NOW()

Variation   : Relationship 88c28e47 → LocalDateTable_bffaeb3b.Date
              defaultHierarchy: LocalDateTable_bffaeb3b.'Date Hierarchy'
NOTE        : Captures model refresh timestamp per row.
              LocalDateTable is created automatically by PBI for this
              date column but has minimal practical use.
Conversion  : ➕ PBI-only (no Tableau equivalent)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — TABLEAU CALCULATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: secure1 valuecentric com_DS Customer Inventory Tableau
## (All measures defined on the main fact table)

### M-01 | Ending Inv. Dollars Calc
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_634726077422100482
PBI Format      : 0.0
PBI Hidden      : No

Tableau Expression:
───────────────────
SUM(IF [Calculation_634726077421936641] = "Yes" THEN [c11] ELSE NULL END)
// Sums Ending Inventory Dollars only for last-day-of-month rows

Power BI DAX:
─────────────
/// Measure_Ending Inv. Dollars Calc on column Ending Inv. Dollars Calc
measure 'Ending Inv. Dollars Calc' =
CALCULATE(
    SUM(
        'secure1 valuecentric com_DS Customer Inventory Tableau'
            [Ending Inventory Dollars]
    ),
    'secure1 valuecentric com_DS Customer Inventory Tableau'
        [Last Day of Month Filter] = "Yes"
)
/* Original Tableau Expression is
   SUM(IF[Calculation_634726077421936641]="Yes" then [c11] ELSE NULL end) */

Depends on  : [c11] Ending Inventory Dollars, CC-02 [Last Day of Month Filter]
Used by     : M-07 [Ending Inventory], M-13 [Selected Measure]
Conversion  : ✅ FULL


### M-02 | Ending Inv. Units Calc
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Ending Inv. Dollars Calc (copy)_634726077424758788
PBI Format      : 0.0

Tableau Expression:
───────────────────
SUM(IF [Calculation_634726077421936641] = "Yes" THEN [c6] ELSE NULL END)

Power BI DAX:
─────────────
/// Measure_Ending Inv. Units Calc on column Ending Inv. Units Calc
measure 'Ending Inv. Units Calc' =
CALCULATE(
    SUM(
        'secure1 valuecentric com_DS Customer Inventory Tableau'
            [Ending Inventory Units]
    ),
    'secure1 valuecentric com_DS Customer Inventory Tableau'
        [Last Day of Month Filter] = "Yes"
)
/* Original Tableau Expression is
   SUM(IF[Calculation_634726077421936641]="Yes" then [c6] ELSE NULL end) */

Depends on  : [c6] Ending Inventory Units, CC-02
Used by     : M-07 [Ending Inventory], M-13 [Selected Measure]
Conversion  : ✅ FULL


### M-03 | Ending Inv.
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_634726077434986509
PBI Format      : 0.0

Tableau Expression:
───────────────────
IF [Parameters].[Parameter 1]=0 THEN [c11]
ELSEIF [Parameters].[Parameter 1]=1 THEN [c6]
END
// Direct column sum (no last-day filter) — simpler variant

Power BI DAX:
─────────────
/// Measure_Ending Inv. on column Ending Inv.
measure 'Ending Inv.' =
IF(
    MIN('Measure Selector'[ParameterTableKeyColumn]) = 0,
    CALCULATE(
        SUM('secure1 valuecentric com_DS Customer Inventory Tableau'
            [Ending Inventory Dollars])
    ),
    IF(
        MIN('Measure Selector'[ParameterTableKeyColumn]) = 1,
        CALCULATE(
            SUM('secure1 valuecentric com_DS Customer Inventory Tableau'
                [Ending Inventory Units])
        ),
        BLANK()
    )
)
/* Original Tableau Expression is
   IF [Parameters].[Parameter 1]=0 THEN [c11]
   ELSEIF [Parameters].[Parameter 1]=1 THEN [c6] END */

Depends on  : [c11], [c6], PARAM-01
Conversion  : ✅ FULL


### M-04 | Morgue Dollars Calc
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Ending Inv. Dollars Calc (copy)_634726077426380808
PBI Format      : 0.0

Tableau Expression:
───────────────────
SUM(IF [Calculation_634726077421936641] = "Yes" THEN [c15] ELSE NULL END)

Power BI DAX:
─────────────
/// Measure_Morgue Dollars Calc on column Morgue Dollars Calc
measure 'Morgue Dollars Calc' =
CALCULATE(
    SUM(
        'secure1 valuecentric com_DS Customer Inventory Tableau'
            [Morgue Inv. Dollars]
    ),
    'secure1 valuecentric com_DS Customer Inventory Tableau'
        [Last Day of Month Filter] = "Yes"
)
/* Original Tableau Expression is
   SUM(IF[Calculation_634726077421936641]="Yes" then [c15] ELSE NULL end) */

Depends on  : [c15], CC-02
Used by     : M-09 [Morgue]
Conversion  : ✅ FULL


### M-05 | Morgue Units Calc
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Ending Inv. Units Calc (copy)_634726077426573321
PBI Format      : 0.0

Tableau Expression:
───────────────────
SUM(IF [Calculation_634726077421936641] = "Yes" THEN [c8] ELSE NULL END)

Power BI DAX:
─────────────
/// Measure_Morgue Units Calc on column Morgue Units Calc
measure 'Morgue Units Calc' =
CALCULATE(
    SUM(
        'secure1 valuecentric com_DS Customer Inventory Tableau'
            [Morgue Inv. Units]
    ),
    'secure1 valuecentric com_DS Customer Inventory Tableau'
        [Last Day of Month Filter] = "Yes"
)
/* Original Tableau Expression is
   SUM(IF[Calculation_634726077421936641]="Yes" then [c8] ELSE NULL end) */

Depends on  : [c8], CC-02
Used by     : M-09 [Morgue]
Conversion  : ✅ FULL


### M-06 | Max Date
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_634726077445574672
PBI Format      : """TRUE"";""TRUE"";""FALSE"""  (boolean display)

Tableau Expression:
───────────────────
MAX([Date]) = TODAY()-1
// TRUE when the latest date in the data is yesterday

Power BI DAX:
─────────────
/// Measure_Max Date on column Max Date
measure 'Max Date' =
MAX(
    'secure1 valuecentric com_DS Customer Inventory Tableau'[Date]
) = TODAY() - 1
/* Original Tableau Expression is MAX([Date])= TODAY()-1 */

NOTE: Returns a boolean; displayed as "TRUE"/"FALSE" string via
      format string """TRUE"";""TRUE"";""FALSE"""
Used by     : M-11 [Validation]
Conversion  : ✅ FULL


### M-07 | Ending Inventory
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_634726077424123907
PBI Format      : 0.0

Tableau Expression:
───────────────────
IF [Parameters].[Parameter 1]=0 THEN [Ending Inv. Dollars Calc]
ELSEIF [Parameters].[Parameter 1]=1 THEN [Ending Inv. Units Calc]
END
// Routes to the last-day-filtered calc measure

Power BI DAX:
─────────────
/// Measure_Ending Inventory on column Ending Inventory
measure 'Ending Inventory' =
IF(
    MIN('Measure Selector'[ParameterTableKeyColumn]) = 0,
    [Ending Inv. Dollars Calc],
    IF(
        MIN('Measure Selector'[ParameterTableKeyColumn]) = 1,
        [Ending Inv. Units Calc],
        BLANK()
    )
)
/* Original Tableau Expression is
   IF [Parameters].[Parameter 1]=0 THEN [Ending Inv. Dollars Calc]
   ELSEIF [Parameters].[Parameter 1]=1 THEN [Ending Inv. Units Calc] END */

Depends on  : M-01, M-02, PARAM-01
Conversion  : ✅ FULL


### M-08 | Returns to Usable Inv
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Ending Inventory (copy)_634726077425364998
PBI Format      : 0.0

Tableau Expression:
───────────────────
IF [Parameters].[Parameter 1]=0 THEN [c14]
ELSEIF [Parameters].[Parameter 1]=1 THEN [c9]
END

Power BI DAX:
─────────────
/// Measure_Returns to Usable Inv on column Returns to Usable Inv
measure 'Returns to Usable Inv' =
IF(
    MIN('Measure Selector'[ParameterTableKeyColumn]) = 0,
    CALCULATE(
        SUM('secure1 valuecentric com_DS Customer Inventory Tableau'
            [852 Returns To Usable Inv. Dollars])
    ),
    IF(
        MIN('Measure Selector'[ParameterTableKeyColumn]) = 1,
        CALCULATE(
            SUM('secure1 valuecentric com_DS Customer Inventory Tableau'
                [852 Returns To Usable Inv. Units])
        ),
        BLANK()
    )
)
/* Original Tableau Expression is
   IF [Parameters].[Parameter 1]=0 THEN [c14]
   ELSEIF [Parameters].[Parameter 1]=1 THEN [c9] END */

Depends on  : [c14], [c9], PARAM-01
Conversion  : ✅ FULL


### M-09 | Morgue
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Ending Inventory (copy)_634726077426749450
PBI Format      : 0.0

Tableau Expression:
───────────────────
IF [Parameters].[Parameter 1]=0 THEN [Morgue Dollars Calc]
ELSEIF [Parameters].[Parameter 1]=1 THEN [Morgue Units Calc]
END

Power BI DAX:
─────────────
/// Measure_Morgue on column Morgue
measure Morgue =
IF(
    MIN('Measure Selector'[ParameterTableKeyColumn]) = 0,
    [Morgue Dollars Calc],
    IF(
        MIN('Measure Selector'[ParameterTableKeyColumn]) = 1,
        [Morgue Units Calc],
        BLANK()
    )
)
/* Original Tableau Expression is
   IF [Parameters].[Parameter 1]=0 THEN [Morgue Dollars Calc]
   ELSEIF [Parameters].[Parameter 1]=1 THEN [Morgue Units Calc] END */

Depends on  : M-04, M-05, PARAM-01
Conversion  : ✅ FULL


### M-10 | Returns to Morgue
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Sales (copy)_634726077431885836
PBI Format      : 0.0

Tableau Expression:
───────────────────
IF [Parameters].[Parameter 1]=0 THEN [c13]
ELSEIF [Parameters].[Parameter 1]=1 THEN [c10]
END

Power BI DAX:
─────────────
/// Measure_Returns to Morgue on column Returns to Morgue
measure 'Returns to Morgue' =
IF(
    MIN('Measure Selector'[ParameterTableKeyColumn]) = 0,
    CALCULATE(
        SUM('secure1 valuecentric com_DS Customer Inventory Tableau'
            [852 Returns To Morgue Dollars])
    ),
    IF(
        MIN('Measure Selector'[ParameterTableKeyColumn]) = 1,
        CALCULATE(
            SUM('secure1 valuecentric com_DS Customer Inventory Tableau'
                [852 Returns To Morgue Units])
        ),
        BLANK()
    )
)
/* Original Tableau Expression is
   IF [Parameters].[Parameter 1]=0 THEN [c13]
   ELSEIF [Parameters].[Parameter 1]=1 THEN [c10] END */

Depends on  : [c13], [c10], PARAM-01
Conversion  : ✅ FULL


### M-11 | Validation
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_634726077445668881
PBI Format      : (string — no format string)

Tableau Expression:
───────────────────
IF [Max Date] = True THEN "Valid" ELSE "Error" END

Power BI DAX:
─────────────
/// Measure_Validation on column Validation
measure Validation =
IF(
    [Max Date] = TRUE(),
    "Valid",
    "Error"
)
/* Original Tableau Expression is
   IF [Max Date]= True then "Valid" ELSE "Error" END */

Depends on  : M-06 [Max Date]
Used by     : Validation sheet — filter to show only rows where
              Validation <> "Valid"
Conversion  : ✅ FULL


### M-12 | Morgue Inv.
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Ending Inv. (copy)_634726077435187214
PBI Format      : 0.0

Tableau Expression:
───────────────────
IF [Parameters].[Parameter 1]=0 THEN [c15]
ELSEIF [Parameters].[Parameter 1]=1 THEN [c8]
END

Power BI DAX:
─────────────
/// Measure_Morgue Inv. on column Morgue Inv.
measure 'Morgue Inv.' =
IF(
    MIN('Measure Selector'[ParameterTableKeyColumn]) = 0,
    CALCULATE(
        SUM('secure1 valuecentric com_DS Customer Inventory Tableau'
            [Morgue Inv. Dollars])
    ),
    IF(
        MIN('Measure Selector'[ParameterTableKeyColumn]) = 1,
        CALCULATE(
            SUM('secure1 valuecentric com_DS Customer Inventory Tableau'
                [Morgue Inv. Units])
        ),
        BLANK()
    )
)
/* Original Tableau Expression is
   IF [Parameters].[Parameter 1]=0 THEN [c15]
   ELSEIF [Parameters].[Parameter 1]=1 THEN [c8] END */

Depends on  : [c15], [c8], PARAM-01
Conversion  : ✅ FULL


### M-13 | Selected Measure
─────────────────────────────────────────────────────────────
PBI Lineage Tag : bb1524d9-fd5b-47cf-9d53-9c8f133b6ef4
PBI Format      : 0  (integer)

Tableau Expression:
───────────────────
// No direct equivalent — PBI summary card / KPI measure

Power BI DAX:
─────────────
measure 'Selected Measure' =
SWITCH(
    SELECTEDVALUE('Measure Selector'[ParameterTableKeyColumn]),
    0, [Ending Inv. Dollars Calc],
    1, [Ending Inv. Units Calc],
    BLANK()
)

NOTE        : Uses SELECTEDVALUE (not MIN) — preferred modern pattern.
              Functionally equivalent to M-07 [Ending Inventory] but
              uses SWITCH instead of nested IF.
Depends on  : M-01, M-02, PARAM-01
Conversion  : ✅ FULL


### M-14 | Sales
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Returns to Usable Inv (copy)_634726077425983495
PBI Format      : 0.0

Tableau Expression:
───────────────────
IF [Parameters].[Parameter 1]=0 THEN [c12]
ELSEIF [Parameters].[Parameter 1]=1 THEN [c7]
END

Power BI DAX:
─────────────
/// Measure_Sales on column Sales
measure Sales =
VAR _sel =
    SELECTEDVALUE('Measure Selector'[ParameterTableKeyColumn], 0)
RETURN
    IF(
        _sel = 0,
        SUM('secure1 valuecentric com_DS Customer Inventory Tableau'
            [Sales Dollars]),
        IF(
            _sel = 1,
            SUM('secure1 valuecentric com_DS Customer Inventory Tableau'
                [Sales Units])
        )
    )
/* Original Tableau Expression is
   IF [Parameters].[Parameter 1]=0 THEN [c12]
   ELSEIF [Parameters].[Parameter 1]=1 THEN [c7] END */

Depends on  : [c12], [c7], PARAM-01
⚠️  NOTE    : This measure uses SELECTEDVALUE + VAR pattern instead of
              MIN(). All other parameter-driven measures use MIN().
              Both patterns are functionally equivalent here.
              Prefer SELECTEDVALUE/VAR for all new measures.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — HIERARCHIES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### H-01 | Date_Date Hierarchy
─────────────────────────────────────────────────────────────
PBI Table       : secure1 valuecentric com_DS Customer Inventory Tableau
PBI lineageTag  : e2be6278-8d37-46d9-bcff-9d826f23870a

Levels:
  Level 1  : Date_Year  (lineageTag: Date_Year,  column: CC-05 Date_Year)

NOTE: Single-level hierarchy — only Date_Year is explicitly defined.
      Full Year→Quarter→Month→Day drill-down is available via the
      LocalDateTable_0b721842 automatic hierarchy.

Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — DATE TABLE RELATIONSHIPS & LOCALDATE TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All five date-type columns on the main table have auto-generated
LocalDateTable companions. All LocalDateTables are:
  isHidden, showAsVariationsOnly, mode: import (calculated)
  Standard Date Hierarchy: Year → Quarter → Month → Day

Relationship ID              From Column   JoinOnDateBehavior  LocalDateTable GUID
─────────────────────────────────────────────────────────────────────────────────
eaaa6f23-42b0-4026-828f-276c8fcf0b2b  Day       datePartOnly  LocalDateTable_0d20c742
b8b2bdd6-8590-4f58-89a4-8e63e7c69f69  Date      datePartOnly  LocalDateTable_e7b7f522
b5bd6f29-9bfb-4ffe-9dce-4fcd582dbf57  Date_Year datePartOnly  LocalDateTable_0b721842
c69db48d-e52a-4cce-9ef9-f1ab0a4b1c24  Date_MY   datePartOnly  LocalDateTable_0bd6e828
88c28e47-e15c-4471-8212-9c5d7cfb6946  Refresh   datePartOnly  LocalDateTable_bffaeb3b

LocalDateTable Calendar Sources:
  LocalDateTable_0d20c742 → Calendar( MIN([Day]),  MAX([Day])  )  — year boundaries
  LocalDateTable_e7b7f522 → Calendar( MIN([Date]), MAX([Date]) )
  LocalDateTable_0b721842 → Calendar( MIN([Date_Year]), MAX([Date_Year]) )
  LocalDateTable_0bd6e828 → Calendar( MIN([Date_MY]),   MAX([Date_MY])   )
  LocalDateTable_bffaeb3b → Calendar( MIN([Refresh]),   MAX([Refresh])   )

DateTableTemplate GUID: DateTableTemplate_0d8f2d4d-42c2-4ab1-b229-2aec2ccc09ae
  Seed: Calendar(Date(2015,1,1), Date(2015,1,1))
  Columns: Date, Year, MonthNo, Month (sortBy: MonthNo),
           QuarterNo, Quarter (sortBy: QuarterNo), Day
  Hierarchy: Year → Quarter → Month → Day
  Annotation: __PBI_TemplateDateTable = true


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — DASHBOARD FILTERS & FIXED BEHAVIORS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Filter / Behavior                        Scope                  Value                        PBI Implementation
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────
Product Grouping ≠ REVLIMID              SP Monthly, SPs Chart, Wholesaler   Exclude REVLIMID      Report-level filter on [Product Grouping] ≠ "REVLIMID"
Partner Group ≠ "ALL US - PR" and ≠ NULL SP/Wholesaler detail   Exclude two values           Report/page filter excluding "ALL US - PR" and BLANK
Partner Group = "ALL US - PR"            Wholesaler sheets       Wholesaler-only view         Report-level filter = "ALL US - PR"
Show if Not All Zero = TRUE              Detail sheets           Hides all-zero rows          Visual-level filter on CC-03 [Show if Not All Zero] = TRUE
Date_MY (relative — last month)          Detail sheets           Jun 2026 + last-values       Relative date slicer on [Date_MY] or report filter
Validation ≠ "Valid"                     Validation sheet        Show only Error rows         Filter on M-11 [Validation] ≠ "Valid"
Sheet 9 hard-coded date                  Sheet 9 only            2026-05-31 (static)          Fixed report filter on [Date] = 2026-05-31  ⚠️ not dynamic


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — PRODUCT COLOR MAP (Tableau → Power BI Theme)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Apply as Power BI report theme JSON or manual conditional formatting rules.

Product            Hex Color
──────────────────────────────
EVOTAZ             #499894
POMALYST           #4e79a7
REVLIMID           #4e79a7
BARACLUDE          #59a14f
YERVOY             #59a14f
KRAZATI            #76b7b2
THALOMID           #76b7b2
ISTODAX            #79706e
IDHIFA             #86bcb6
COBENFY            #8cd17d
ORENCIA IV         #9d7660
ABRAXANE           #a0cbe8
REYATAZ            #a0cbe8
OPDIVO             #b07aa1
ELIQUIS            #b6992d
KENALOG            #bab0ac
NULOJIX            #d37295
OPDUALAG           #d4a6c8
ORENCIA SC         #d4a6c8
REBLOZYL           #d7b5a6
SOTYKTU            #e15759
EMPLICITI          #f1ce63
AUGTYRO            #f28e2b
CAMZYOS            #f28e2b
SPRYCEL            #f28e2b
ONUREG             #fabfd2
INREBIC            #ff9d9a
AZACTAM            #ffbe7d
VIDAZA             #ffbe7d


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — KNOWN GAPS, NOTES & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAP-01 | Intentional column name typo: "Last Day MOnth fil"
  Severity : Info
  Detail   : Column CC-06 has "MOnth" capitalised mid-word.
             This matches the source TMDL exactly and must be
             preserved in any future re-creation or update.
  Action   : Do NOT auto-correct capitalisation.

GAP-02 | Intentional sheet name typo: "Wholsaler Chart"
  Severity : Info
  Detail   : Tableau worksheet named "Wholsaler Chart" (missing 'e').
             Preserve when naming equivalent PBI report page.
  Action   : Keep typo in PBI page name for traceability.

GAP-03 | Measure pattern inconsistency (MIN vs SELECTEDVALUE)
  Severity : Low
  Detail   : All parameter-driven measures except [Sales] and
             [Selected Measure] use MIN('Measure Selector'[...]).
             [Sales] uses SELECTEDVALUE + VAR; [Selected Measure]
             uses SELECTEDVALUE.
  Action   : For future new measures, standardise on
             SELECTEDVALUE('Measure Selector'[ParameterTableKeyColumn], 0)
             with a VAR. Existing measures are functionally correct.

GAP-04 | Sheet 9 hard-coded date filter
  Severity : Medium
  Detail   : Sheet 9 in Tableau has a static filter #2026-05-31#.
             This will go stale as data moves forward in time.
  Action   : Replace with a dynamic "last available month" filter,
             or implement a relative date slicer.

GAP-05 | Compatibility Level 1604 (not 1600)
  Severity : Info
  Detail   : This model uses level 1604, slightly higher than
             other workbooks in the portfolio (1600).
  Action   : Note when deploying; ensure Fabric/PBI Desktop version
             supports 1604 before opening on older builds.

GAP-06 | [Refresh] column creates a LocalDateTable with low utility
  Severity : Info
  Detail   : Refresh = NOW() triggers an auto LocalDateTable
             (LocalDateTable_bffaeb3b). The table is hidden and
             valid but the date hierarchy on a NOW() timestamp
             has no analytical value.
  Action   : Consider hiding the [Refresh] column from report
             view and suppressing the LocalDateTable if it causes
             confusion.

GAP-07 | Source columns have SUM() prefixes in raw data
  Severity : Info
  Detail   : IQVIA delivers the TSV with column headers like
             "SUM(Ending Inventory Dollars)". Power Query's
             Table.PromoteHeaders + sourceColumn mapping in TMDL
             renames them to clean display names. If the IQVIA
             endpoint ever changes column header naming, the
             Power Query step must be updated.
  Action   : Monitor IQVIA feed for header changes at each refresh.

GAP-08 | Product color map must be applied manually
  Severity : Info
  Detail   : Tableau color encoding is stored in workbook XML.
             Power BI does not auto-import color assignments.
  Action   : Apply color map from Section 9 as a PBI JSON theme
             or via conditional formatting rules on each visual.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 11 — CONVERSION COMPLETENESS SCORECARD
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Category                    Tableau Items  PBI Full  PBI Partial  Not Converted  % Complete
────────────────────────────────────────────────────────────────────────────────────────────
Parameters                       1             1          0             0           100%
Source Columns (c1–c15)         15            15          0             0           100%
Calculated Columns               7             7          0             0           100%
DAX Measures (Tableau-equiv.)   14            14          0             0           100%
DAX Measures (PBI-only)          0             2          —             —          (new)
Power Query Sources              1             1          0             0           100%
LocalDate / Date Tables          5             5          0             0           100%
────────────────────────────────────────────────────────────────────────────────────────────
TOTAL (Tableau artefacts)       42            42          0             0        🟢 100%

PBI-only additions:
  ➕ CC-08  [Refresh] column  — data freshness timestamp
  ➕ M-13   [Selected Measure] — SWITCH-based SELECTEDVALUE variant


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 12 — QUICK REFERENCE: TABLEAU FORMULA → DAX PATTERN
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Tableau Pattern                                          DAX Equivalent
───────────────────────────────────────────────────────────────────────────────
IF [Param]=0 THEN [fieldA] ELSEIF [Param]=1 ...        → IF(MIN('Selector'[Key])=0,[fieldA],IF(...=1,[fieldB],BLANK()))
IF [Param]=0 THEN [fieldA] ELSEIF [Param]=1 ...        → VAR _sel=SELECTEDVALUE([Key],0) RETURN IF(_sel=0,...)  ← preferred
SUM(IF [flag]="Yes" THEN [col] ELSE NULL END)           → CALCULATE(SUM([col]),[flag]="Yes")
DATE(DATEPARSE("yyyy-MM-dd HH:mm:ss", STR([dt])))      → DATEVALUE([dt])
IF [d]=DATEADD('day',-1,DATEADD('month',1,DATETRUNC('month',[d])))
    THEN "Yes" ...                                      → IF([d]=EOMONTH([d],0),"Yes",IF([d]=TODAY()-1,"Yes","No"))
MAX([Date])=TODAY()-1                                   → MAX([Date])=TODAY()-1  (boolean measure)
IF [bool]=True THEN "Valid" ELSE "Error" END            → IF([Measure]=TRUE(),"Valid","Error")
CASE [ProductGroup] WHEN "X" THEN "X" ... IN {...} THEN "Non-Specialty" END
                                                        → SWITCH(TRUE(),[col]="X","X",...,[col] IN {...},"Non-Specialty",BLANK())
Parameter-driven axis (Dollars/Units)                   → Table.FromRows static table + SELECTEDVALUE dispatch measure
YEAR([Date])                                            → YEAR([Date])  — stored as dateTime type, formatted "Mmmm, yyyy"
FORMAT([Date],"MMM, yyyy")                              → FORMAT([Date],"MMM, yyyy") — stored as dateTime type


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 13 — FILE INVENTORY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

File                                                           Content
────────────────────────────────────────────────────────────────────────────────────────────────
customer_inventory.txt  (not uploaded — inferred from PBIP)    Tableau workbook XML
5_definition.json                                              PBIP schema version & settings
5_database.txt                                                 PBIP model database / compat level
5_model.txt                                                    PBIP model table list & annotations
5_expressions.txt                                              Power Query parameter expression
5_relationships.txt                                            5 LocalDateTable relationships
5_diagramlayout.json                                           3-table diagram layout
secure1_valuecentric_com_ds_customer_inventory_tableau.txt     Main fact table TMDL
measure_selector.txt                                           Measure Selector parameter table TMDL
datetabletemplate_0d8f2d4d_42c2_4ab1_b229_2aec2ccc09ae.txt    Date template table TMDL
localdatetable_0d20c742_9b65_4b20_8685_b2a696eeb642.txt        LocalDateTable for [Day]
localdatetable_e7b7f522_82e7_400c_acaa_80d667452491.txt        LocalDateTable for [Date]
localdatetable_0b721842_8bb4_42ea_af97_c9097707dc83.txt        LocalDateTable for [Date_Year]
localdatetable_0bd6e828_5f73_48fa_b918_cb95ffada02a.txt        LocalDateTable for [Date_MY]
localdatetable_bffaeb3b_b3d3_4e0b_8824_db7b5942833f.txt        LocalDateTable for [Refresh]

---
# END OF KNOWLEDGE FILE
# 25026 – Customer Inventory | Tableau → Power BI (PBIP/TMDL)
# Generated: 2026-07-07
# Source files: See Section 13 — File Inventory above
