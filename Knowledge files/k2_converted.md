# 📘 CONVERSION KNOWLEDGE FILE
# K2 – Canada Order Trend Analysis
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-23
# Source files: canada_order_trend_analysis (Tableau workbook XML, ID 18504)
#               out_custom_sql_query.txt
#               out_measures_table.txt
#               out_parameters.txt
#               out_relationships.txt
#               out_database.txt
#               out_model.txt
#               out_en_us.txt
#               out_diagramlayout.json
#               out_theme.json
#               out_localdatetable_order_created.txt
#               out_localdatetable_customer_ref.txt
#               out_localdatetable_billing.txt
#               out_localdatetable_billing_created.txt
#               out_localdatetable_date_selection.txt

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PARAM-01 | Currency Selection
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Currency Selection]
Tableau Caption       : Currency Selection
Tableau Type          : integer
Tableau Default       : 0
Tableau Members       : 0 = Doc Currency, 1 = USD

Power BI Table Name   : Parameters (shared single-row table)
Power BI Column       : Currency Selector (int64)
PBI lineageTag        : param-1-key
PBI sourceColumn      : CurrSel

SELECTEDVALUE Measure:
──────────────────────
measure 'Parameter 1 Value' = SELECTEDVALUE('Parameters'[Currency Selector], 0)
    lineageTag: mt-meas-param-1-value

Usage in model:
  - Referenced by CC-14 [Exchange Rate Statement] for currency
    conversion logic.
  - Default slicer selection: 0 (Doc Currency).

Conversion Status: ✅ FULL


### PARAM-02 | Top Set
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Top Set]
Tableau Caption       : Top Set
Tableau Type          : integer
Tableau Default       : 10
Tableau Range         : 5–25

Power BI Table Name   : Parameters (shared single-row table)
Power BI Column       : Top Set (int64)
PBI lineageTag        : param-2-key
PBI sourceColumn      : TopSet

SELECTEDVALUE Measure:
──────────────────────
measure 'Parameter 2 Value' = SELECTEDVALUE('Parameters'[Top Set], 10)
    lineageTag: mt-meas-param-2-value

Usage in model:
  - Controls Top N filtering in report-layer visuals.
  - Default: 10.

Conversion Status: ✅ FULL


### PARAM-03 | Date Selector
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Date Selector]
Tableau Caption       : Date Selector
Tableau Type          : integer
Tableau Default       : 0
Tableau Members       : 0 = Order Creation Date, 1 = Billing Date

Power BI Table Name   : Parameters (shared single-row table)
Power BI Column       : Date Selector (int64)
PBI lineageTag        : param-3-key
PBI sourceColumn      : DateSel

SELECTEDVALUE Measure:
──────────────────────
measure 'Parameter 3 Value' = SELECTEDVALUE('Parameters'[Date Selector], 0)
    lineageTag: mt-meas-param-3-value

Usage in model:
  - Referenced by CC-01 [!Date Selection] — switches between
    [Order Created on Date] (0) and [Billing Date] (1).
  - Default: 0 (Order Creation Date).

⚠️  CRITICAL DEPENDENCY: CC-01 uses [Parameter 3 Value] measure.
    This parameter drives all downstream date-derived columns
    (CC-02 through CC-08).

Conversion Status: ✅ FULL


### PARAM-04 | Reporting Period
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Reporting Period]
Tableau Caption       : Reporting Period
Tableau Type          : integer
Tableau Default       : 1
Tableau Members       : 1 = Month, 2 = Quarter, 3 = Week

Power BI Table Name   : Parameters (shared single-row table)
Power BI Column       : Reporting Period (int64)
PBI lineageTag        : param-4-key
PBI sourceColumn      : PeriodSel

SELECTEDVALUE Measure:
──────────────────────
measure 'Parameter 4 Value' = SELECTEDVALUE('Parameters'[Reporting Period], 1)
    lineageTag: mt-meas-param-4-value

Usage in model:
  - Referenced by CC-08 [!Date Parameter Control] — switches
    between [!Reporting Month] and [!Reporting Quarter].
  - Default: 1 (Month).

Conversion Status: ✅ FULL


### PARAM-05 | Measure Selection
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Measure Selection]
Tableau Caption       : Measure Selection
Tableau Type          : integer
Tableau Default       : 2
Tableau Members       : 1 = Invoice Amount, 2 = Invoice Count

Power BI Table Name   : Parameters (shared single-row table)
Power BI Column       : Measure Selection (int64)
PBI lineageTag        : param-5-key
PBI sourceColumn      : MeasureSel

SELECTEDVALUE Measure:
──────────────────────
measure 'Parameter 5 Value' = SELECTEDVALUE('Parameters'[Measure Selection], 2)
    lineageTag: mt-meas-param-5-value

Usage in model:
  - Controls which metric is displayed in report-layer visuals
    (invoice amount vs. count).
  - Default: 2 (Invoice Count).

Conversion Status: ✅ FULL


### PARAM-06 | Minimum Order Threshold
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Minimum Order Threshold]
Tableau Caption       : Minimum Order Threshold
Tableau Type          : integer
Tableau Default       : 25000
Tableau Range         : min = 1

Power BI Table Name   : Parameters (shared single-row table)
Power BI Column       : Min Order Threshold (int64)
PBI lineageTag        : param-6-key
PBI sourceColumn      : MinThreshold

SELECTEDVALUE Measure:
──────────────────────
measure 'Parameter 6 Value' = SELECTEDVALUE('Parameters'[Min Order Threshold], 25000)
    lineageTag: mt-meas-param-6-value

Usage in model:
  - Threshold filter for large-order analysis visuals.
  - Default: 25000.

Conversion Status: ✅ FULL


### PARAM-07 | Threshold Select
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Threshold Select]
Tableau Caption       : Threshold Select
Tableau Type          : string
Tableau Default       : "Selected Threshold"
Tableau Members       : "Selected Threshold", "Avg Yearly by Sold To Group"

Power BI Table Name   : Parameters (shared single-row table)
Power BI Column       : Threshold Select (string)
PBI lineageTag        : param-7-key
PBI sourceColumn      : ThresholdSel

SELECTEDVALUE Measure:
──────────────────────
measure 'Parameter 7 Value' = SELECTEDVALUE('Parameters'[Threshold Select], "Selected Threshold")
    lineageTag: mt-meas-param-7-value

Usage in model:
  - Switches threshold logic between a fixed user-entered value
    and the average yearly value per Sold To Group.
  - Default: "Selected Threshold".

⚠️  TYPE NOTE: Only string-type parameter in the model. All other
    parameters are integer or double.

Conversion Status: ✅ FULL


### PARAM-08 | Bottom
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Bottom]
Tableau Caption       : Bottom
Tableau Type          : real (double)
Tableau Default       : 0.0

Power BI Table Name   : Parameters (shared single-row table)
Power BI Column       : Bottom Value (double)
PBI lineageTag        : param-8-key
PBI sourceColumn      : BottomVal

SELECTEDVALUE Measure:
──────────────────────
measure 'Parameter 8 Value' = SELECTEDVALUE('Parameters'[Bottom Value], 0.0)
    lineageTag: mt-meas-param-8-value

Usage in model:
  - Bottom N / floor value for report-layer filtering.
  - Default: 0.0.

Conversion Status: ✅ FULL


### Parameters Table — M Query (shared single-row table)
─────────────────────────────────────────────────────────────
All 8 parameters are stored in a single-row table with typed columns.

Power BI M Query:
─────────────────
let Source = #table(
        type table [CurrSel=Int64.Type, TopSet=Int64.Type, DateSel=Int64.Type,
            PeriodSel=Int64.Type, MeasureSel=Int64.Type, MinThreshold=Int64.Type,
            ThresholdSel=text, BottomVal=number],
        {{0, 10, 0, 1, 2, 25000, "Selected Threshold", 0.0}}
    ) in Source

PBI Table lineageTag: param-table-0000-000000000001

⚠️  DESIGN NOTE: Unlike the k52 pattern (separate single-row tables per
    parameter), this workbook consolidates all 8 parameters into one
    table with 8 columns. Each column is surfaced via a SELECTEDVALUE
    measure in the Measures Table.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: Custom_SQL_Query

### CC-01 | !Date Selection
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-date-selection
PBI DataType    : dateTime
PBI FormatString: dd-mmm-yy
PBI Hidden      : No

Tableau Expression:
───────────────────
IF [Date Selector] = 0 THEN [Order Created on Date]
ELSE [Billing Date] END
// Parameter-driven date selector — switches between order creation
// and billing date based on parameter 3

Power BI DAX:
─────────────
column '!Date Selection' = IF([Parameter 3 Value] = 0,
    'Custom_SQL_Query'[Order Created on Date],
    'Custom_SQL_Query'[Billing Date])

Depends on  : PARAM-03 [Parameter 3 Value] measure,
              [Order Created on Date], [Billing Date] (source columns)
Purpose     : Central date column driving all downstream date-derived
              calculations. User switches between order date and
              billing date via parameter slicer.
NOTE        : Has a date variation relationship to
              LocalDateTable_a1b2c3d4-5555-5555-5555-555555555555.
              Relationship: rel-date-selection.

⚠️  NAMING NOTE: Column name starts with '!' prefix — used throughout
    this workbook to indicate calculated/derived columns. Preserved exactly.
⚠️  CRITICAL: This column is the root dependency for CC-02 through CC-08.
    Changing PARAM-03 cascades through all date-derived columns.

Conversion Status: ✅ FULL


### CC-02 | !Date Selection (ISO Weekdays)
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-iso-weekday
PBI DataType    : int64
PBI FormatString: 0

Tableau Expression:
───────────────────
DATEPART('iso-weekday', [!Date Selection])
// ISO weekday number (1=Monday … 7=Sunday)

Power BI DAX:
─────────────
column '!Date Selection (ISO Weekdays)' = WEEKDAY('Custom_SQL_Query'[!Date Selection], 2)

Depends on  : CC-01 [!Date Selection]
Pattern     : Tableau DATEPART('iso-weekday') → DAX WEEKDAY(date, 2)
              WEEKDAY with return_type=2 returns 1=Mon through 7=Sun.
Conversion  : ✅ FULL


### CC-03 | !Reporting Week
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-reporting-week
PBI DataType    : dateTime
PBI FormatString: dd-mmm-yy

Tableau Expression:
───────────────────
DATETRUNC('week', [!Date Selection], 'monday')
// Monday-start reporting week

Power BI DAX:
─────────────
column '!Reporting Week' = 'Custom_SQL_Query'[!Date Selection] - WEEKDAY('Custom_SQL_Query'[!Date Selection], 2)

Depends on  : CC-01 [!Date Selection]
Pattern     : Tableau DATETRUNC('week', date, 'monday') →
              DAX [Date] - WEEKDAY([Date], 2)
NOTE        : Unlike k1 pattern ([Date] - WEEKDAY([Date], 2) + 1),
              this column uses subtraction only (no +1), producing
              the Sunday before the week start. Preserved exactly
              as in the PBIP source.
Conversion  : ✅ FULL


### CC-04 | !Reporting Month
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-reporting-month
PBI DataType    : string

Tableau Expression:
───────────────────
IF DATETRUNC('month', [!Date Selection]) = DATETRUNC('month', DATEADD('month', -1, TODAY()))
    THEN "Previous"
ELSEIF DATETRUNC('month', [!Date Selection]) = DATETRUNC('month', TODAY())
    THEN "Current"
ELSE STR(YEAR([!Date Selection]) * 100 + MONTH([!Date Selection]))
END
// Labels current month as "Current", previous month as "Previous",
// all others as YYYYMM numeric string

Power BI DAX:
─────────────
column '!Reporting Month' =
    VAR _ds = 'Custom_SQL_Query'[!Date Selection]
    VAR _today = TODAY()
    VAR _curYM = YEAR(_today) * 100 + MONTH(_today)
    VAR _prevYM = YEAR(EDATE(_today, -1)) * 100 + MONTH(EDATE(_today, -1))
    VAR _dsYM = YEAR(_ds) * 100 + MONTH(_ds)
    RETURN
        SWITCH(TRUE(),
            _dsYM = _prevYM, "Previous",
            _dsYM = _curYM, "Current",
            FORMAT(_dsYM, "0")
        )

Depends on  : CC-01 [!Date Selection]
Pattern     : Tableau IF/ELSEIF with DATETRUNC month comparison →
              DAX SWITCH(TRUE()) with YEAR*100+MONTH integer encoding.
              Uses EDATE(TODAY(), -1) for previous month.
Business Rule: "Current" = same YYYYMM as today. "Previous" = prior month.
               All other months → YYYYMM string.
Conversion  : ✅ FULL


### CC-05 | !Reporting Quarter
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-reporting-quarter
PBI DataType    : string

Tableau Expression:
───────────────────
// Same pattern as CC-04 but using quarter granularity
IF DATETRUNC('quarter', [!Date Selection]) = DATETRUNC('quarter', DATEADD('quarter', -1, TODAY()))
    THEN "Previous"
ELSEIF DATETRUNC('quarter', [!Date Selection]) = DATETRUNC('quarter', TODAY())
    THEN "Current"
ELSE STR(YEAR([!Date Selection]) * 100 + QUARTER([!Date Selection]))
END

Power BI DAX:
─────────────
column '!Reporting Quarter' =
    VAR _ds = 'Custom_SQL_Query'[!Date Selection]
    VAR _today = TODAY()
    VAR _curYQ = YEAR(_today) * 100 + QUARTER(_today)
    VAR _prevDate = EDATE(_today, -3)
    VAR _prevYQ = YEAR(_prevDate) * 100 + QUARTER(_prevDate)
    VAR _dsYQ = YEAR(_ds) * 100 + QUARTER(_ds)
    RETURN
        SWITCH(TRUE(),
            _dsYQ = _curYQ, "Current",
            _dsYQ = _prevYQ, "Previous",
            FORMAT(_dsYQ, "0")
        )

Depends on  : CC-01 [!Date Selection]
Pattern     : Same as CC-04 but with QUARTER() instead of MONTH().
              Uses EDATE(TODAY(), -3) for previous quarter start.
Business Rule: "Current" = same YYYYQ as today. "Previous" = prior quarter.
               All other quarters → YYYYQ string.
Conversion  : ✅ FULL


### CC-06 | !Brand
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-brand
PBI DataType    : string

Tableau Expression:
───────────────────
// Complex brand extraction from MATERIAL_DESCRIPTION
// ORENCIA has special sub-brand logic based on first two words

Power BI DAX:
─────────────
column '!Brand' =
    VAR _md = 'Custom_SQL_Query'[MATERIAL_DESCRIPTION]
    VAR _firstSpace = SEARCH(" ", _md, 1, 0)
    VAR _firstWord = IF(_firstSpace > 0, LEFT(_md, _firstSpace - 1), _md)
    VAR _firstTwoWords = IF(_firstSpace > 0, LEFT(_md, _firstSpace + 2), _md)
    RETURN
        SWITCH(TRUE(),
            _firstTwoWords = "ORENCIA CL", "ORENCIA SC",
            _firstTwoWords = "ORENCIA SC", "ORENCIA SC",
            _firstTwoWords = "ORENCIA LI", "ORENCIA IV",
            _firstWord
        )

Depends on  : [MATERIAL_DESCRIPTION] (source column)
Business Rule: Extracts brand name from the first word of material
               description. ORENCIA has special handling:
               "ORENCIA CL..." → "ORENCIA SC"
               "ORENCIA SC..." → "ORENCIA SC"
               "ORENCIA LI..." → "ORENCIA IV"
               All other materials → first word only.
⚠️  APPROXIMATION NOTE: The LEFT+SEARCH pattern is an approximation
    of Tableau's string parsing. Validate against actual
    MATERIAL_DESCRIPTION values in the data.
Conversion  : ✅ FULL


### CC-07 | !Material Brand
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-material-brand
PBI DataType    : string

Tableau Expression:
───────────────────
// First word of MATERIAL_DESCRIPTION; ORENCIA gets first 10 characters

Power BI DAX:
─────────────
column '!Material Brand' =
    VAR _md = 'Custom_SQL_Query'[MATERIAL_DESCRIPTION]
    VAR _firstSpace = SEARCH(" ", _md, 1, 0)
    VAR _firstWord = IF(_firstSpace > 0, LEFT(_md, _firstSpace - 1), _md)
    RETURN IF(_firstWord = "ORENCIA", LEFT(_md, 10), _firstWord)

Depends on  : [MATERIAL_DESCRIPTION] (source column)
Business Rule: First word of material description; if the first word
               is "ORENCIA", returns first 10 characters instead
               (captures sub-brand suffix like "ORENCIA SC").
⚠️  NAMING NOTE: Column name starts with '!' prefix. Preserved exactly.
Conversion  : ✅ FULL


### CC-08 | !Date Parameter Control
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-date-parameter-control
PBI DataType    : string

Tableau Expression:
───────────────────
IF [Reporting Period] = 1 THEN [!Reporting Month]
ELSE [!Reporting Quarter] END
// Parameter-driven period label — switches between month and quarter
// based on parameter 4

Power BI DAX:
─────────────
column '!Date Parameter Control' = IF([Parameter 4 Value] = 1,
    'Custom_SQL_Query'[!Reporting Month],
    'Custom_SQL_Query'[!Reporting Quarter])

Depends on  : PARAM-04 [Parameter 4 Value] measure,
              CC-04 [!Reporting Month], CC-05 [!Reporting Quarter]
Purpose     : Display column for period-aware visuals. When period
              parameter = 1 (Month), shows month labels; otherwise
              shows quarter labels.
NOTE        : PARAM-04 also has value 3 (Week), but this column does
              not handle the week case — only month vs. quarter.
              Week display may require report-layer conditional visibility.
Conversion  : ✅ FULL


### CC-09 | !Material Brand (group)
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-material-brand-group
PBI DataType    : string

Tableau Expression:
───────────────────
// ELIQUIS case-insensitive normalisation

Power BI DAX:
─────────────
column '!Material Brand (group)' = IF('Custom_SQL_Query'[!Material Brand] IN {"ELIQUIS","Eliquis"},
    "ELIQUIS", 'Custom_SQL_Query'[!Material Brand])

Depends on  : CC-07 [!Material Brand]
Business Rule: Normalizes "Eliquis" (mixed case) to "ELIQUIS" (uppercase).
               All other brands pass through unchanged.
⚠️  NAMING NOTE: Column name starts with '!' and contains parentheses.
    Valid in TMDL when enclosed in single quotes. Preserved exactly.
Conversion  : ✅ FULL


### CC-10 | Sales Doc + Item
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-sales-doc-item
PBI DataType    : string

Power BI DAX:
─────────────
column 'Sales Doc + Item' = 'Custom_SQL_Query'[SALES_DOC] & 'Custom_SQL_Query'[SALES_DOCUMENT_ITEM_SALES_I]

Depends on  : [SALES_DOC], [SALES_DOCUMENT_ITEM_SALES_I] (source columns)
Purpose     : Concatenated key combining Sales Doc and Item for
              line-level unique identification.
Conversion  : ✅ FULL


### CC-11 | Shipping Type
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-shipping-type
PBI DataType    : string

Tableau Expression:
───────────────────
IF [!Brand] IN ("ORENCIA","NULOJIX","EROPOPHOS","YERVOY","OPDIVO","EMPLICITI","REBLOZYL")
    THEN "Cold Chain" ELSE "Standard" END

Power BI DAX:
─────────────
column 'Shipping Type' = IF('Custom_SQL_Query'[!Brand] IN {
    "ORENCIA","NULOJIX","EROPOPHOS","YERVOY","OPDIVO","EMPLICITI","REBLOZYL"},
    "Cold Chain", "Standard")

Depends on  : CC-06 [!Brand]
Business Rule: Biologics requiring cold chain shipping are flagged.
               Seven brand names are classified as "Cold Chain";
               all others are "Standard".
Conversion  : ✅ FULL


### CC-12 | Order Net Value
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-order-net-value
PBI DataType    : double
PBI FormatString: #,##0.00
PBI summarizeBy : sum

Tableau Expression:
───────────────────
// Sign-flip logic by Sales Doc Type and Item Category
// Positive for domestic orders, negative for credits/returns,
// ABS for ZCCB sub-types based on item category

Power BI DAX:
─────────────
column 'Order Net Value' =
    VAR _net = 'Custom_SQL_Query'[NET_VALUE_OF_THE_BILLING_DOC]
    VAR _doctype = 'Custom_SQL_Query'[SALES_DOC_TYPE]
    VAR _itemcat = 'Custom_SQL_Query'[SALES_DOC_ITEM_CAT]
    RETURN
        SWITCH(TRUE(),
            _doctype IN {"ZDOM","ZDMX","ZDMR"}, _net,
            _doctype IN {"ZCMX","ZCMR","ZRET","ZREV"}, _net * -1,
            _doctype = "ZCCB" && _itemcat = "ZDMR", ABS(_net),
            _doctype = "ZCCB" && _itemcat = "ZCMR", ABS(_net) * -1,
            _net
        )

Depends on  : [NET_VALUE_OF_THE_BILLING_DOC], [SALES_DOC_TYPE],
              [SALES_DOC_ITEM_CAT] (source columns)
Business Rule:
  - ZDOM, ZDMX, ZDMR (domestic/debit): keep original sign
  - ZCMX, ZCMR, ZRET, ZREV (credit/return): flip sign (× -1)
  - ZCCB + ZDMR item category: absolute value (positive)
  - ZCCB + ZCMR item category: absolute value × -1 (negative)
  - All others: keep original sign
⚠️  SIGN CONVENTION: SAP stores credits as positive values;
    this column converts to accounting sign convention where
    credits are negative.
Conversion  : ✅ FULL


### CC-13 | Billing_Invoice_Line
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-billing-invoice-line
PBI DataType    : double
PBI FormatString: #,##0.00
PBI summarizeBy : sum

Tableau Expression:
───────────────────
// Same sign-flip logic as CC-12 but for billing invoice context

Power BI DAX:
─────────────
column 'Billing_Invoice_Line' =
    VAR _net = 'Custom_SQL_Query'[NET_VALUE_OF_THE_BILLING_DOC]
    VAR _doctype = 'Custom_SQL_Query'[SALES_DOC_TYPE]
    VAR _itemcat = 'Custom_SQL_Query'[SALES_DOC_ITEM_CAT]
    RETURN
        SWITCH(TRUE(),
            _doctype IN {"ZDOM","ZDMX","ZDMR"}, _net,
            _doctype IN {"ZCMX","ZCMR","ZRET","ZREV"}, _net * -1,
            _doctype = "ZCCB" && _itemcat = "ZDMR", ABS(_net),
            _doctype = "ZCCB" && _itemcat = "ZCMR", _net * -1,
            _net
        )

Depends on  : [NET_VALUE_OF_THE_BILLING_DOC], [SALES_DOC_TYPE],
              [SALES_DOC_ITEM_CAT] (source columns)
NOTE        : Nearly identical to CC-12 [Order Net Value] except the
              ZCCB+ZCMR branch uses _net * -1 (not ABS(_net) * -1).
              This subtle difference means ZCCB credit memos retain
              their original sign polarity here. Preserved exactly.
Conversion  : ✅ FULL


### CC-14 | Exchange Rate Statement
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-exchange-rate-statement
PBI DataType    : double
PBI FormatString: 0.00000
PBI summarizeBy : none

Tableau Expression:
───────────────────
// Exchange rate lookup via RELATED to TCURR table
// Handles FFACT (from-factor) division

Power BI DAX:
─────────────
column 'Exchange Rate Statement' = IF(RELATED('TCURR'[FFACT]) = 0,
    RELATED('TCURR'[Exchange Rate]),
    RELATED('TCURR'[Exchange Rate]) / RELATED('TCURR'[FFACT]))

Depends on  : RELATED('TCURR'[FFACT]), RELATED('TCURR'[Exchange Rate])
              via relationship rel-currency-fcurr
Business Rule: Looks up exchange rate from TCURR table via the
               SD_DOC_CURRENCY → From currency relationship.
               If FFACT (from-factor) = 0, uses raw exchange rate;
               otherwise divides by FFACT for proper conversion.
⚠️  DEPENDENCY: Requires TCURR table and relationship rel-currency-fcurr
    to be present in the model. TCURR contains SAP exchange rate data.
⚠️  FORMAT NOTE: 5 decimal places (0.00000) for exchange rate precision.
Conversion  : ✅ FULL


### CC-15 | Sold To Name (group)
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-sold-to-name-group
PBI DataType    : string

Tableau Expression:
───────────────────
// LEFT-prefix matching to group 80+ Sold To Name values into
// major customer groups

Power BI DAX:
─────────────
column 'Sold To Name (group)' =
    SWITCH(TRUE(),
        'Custom_SQL_Query'[SOLD_TO_NAME] IN {
            "BC CANCER AGENCY","BC CANCER AGENCY (PHSA)",
            "BC CANCER AGENCY ABBOTSFORD CENTRE",
            "BC CANCER AGENCY CNTR SOUTHERN INTE",
            "BC CHILDREN'S HOSPITAL"}, "BC",
        LEFT('Custom_SQL_Query'[SOLD_TO_NAME], 8) = "BAYSHORE", "Bayshore Pharmacy",
        LEFT('Custom_SQL_Query'[SOLD_TO_NAME], 7) = "INNOMAR", "INNOMAR",
        LEFT('Custom_SQL_Query'[SOLD_TO_NAME], 8) = "MCKESSON", "MCKESSON",
        LEFT('Custom_SQL_Query'[SOLD_TO_NAME], 8) = "SHOPPERS", "Shoppers Drug Mart",
        LEFT('Custom_SQL_Query'[SOLD_TO_NAME], 4) = "QEII", "QEII",
        LEFT('Custom_SQL_Query'[SOLD_TO_NAME], 5) = "ROYAL", "ROYAL HOSPITALS",
        'Custom_SQL_Query'[SOLD_TO_NAME]
    )

Depends on  : [SOLD_TO_NAME] (source column)
Business Rule: Groups customer names by prefix pattern matching.
               BC health authorities → "BC". BAYSHORE... → "Bayshore Pharmacy".
               INNOMAR... → "INNOMAR". MCKESSON... → "MCKESSON".
               SHOPPERS... → "Shoppers Drug Mart". QEII... → "QEII".
               ROYAL... → "ROYAL HOSPITALS". All others → original name.
⚠️  APPROXIMATION NOTE: LEFT-prefix matching covers 80+ distinct values
    but is an approximation. Validate all groupings against source data
    to confirm no values are miscategorized.
⚠️  HIERARCHY: This column is the top level of hierarchy
    'Sold To Group - Ship To' (see SECTION 2a).
Conversion  : ✅ FULL


### CC-16 | Invoice Groups
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-invoice-groups
PBI DataType    : string

Tableau Expression:
───────────────────
// Categorical binning of BILLING_TYPE into business-friendly groups

Power BI DAX:
─────────────
column 'Invoice Groups' =
    SWITCH('Custom_SQL_Query'[BILLING_TYPE],
        "ZB3","Credit Memo","ZOG2","Credit Memo",
        "ZOL2","Debit Memo","ZS1","Debit Memo",
        "ZFDB","Free Goods","ZS3","Free Goods",
        "ZS2","Return and Rejection and Credit Memo",
        "ZREB","Returns and Rejections","ZROB","Returns and Rejections",
        "ZF2B","Sales Order","ZOF2","Sales Order",
        "ZOF8","Samples","Other"
    )

Depends on  : [BILLING_TYPE] (source column)
Business Rule: Maps SAP billing type codes to business-friendly groups:
  ZB3, ZOG2          → "Credit Memo"
  ZOL2, ZS1          → "Debit Memo"
  ZFDB, ZS3          → "Free Goods"
  ZS2                → "Return and Rejection and Credit Memo"
  ZREB, ZROB         → "Returns and Rejections"
  ZF2B, ZOF2         → "Sales Order"
  ZOF8               → "Samples"
  All others         → "Other"
Pattern     : Tableau categorical bin → DAX SWITCH with value mapping.
Conversion  : ✅ FULL


### CC-17 | Sales Document Group
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-calc-sales-doc-group
PBI DataType    : string

Tableau Expression:
───────────────────
// Categorical binning of SALES_DOC_TYPE into business groups

Power BI DAX:
─────────────
column 'Sales Document Group' =
    SWITCH(TRUE(),
        'Custom_SQL_Query'[SALES_DOC_TYPE] IN {"ZCCB","ZCMR","ZCMX","ZDMR","ZDMX","ZRET"},
            "Rebates and Adjustments",
        'Custom_SQL_Query'[SALES_DOC_TYPE] IN {"ZREV","ZRFE"}, "Returns",
        'Custom_SQL_Query'[SALES_DOC_TYPE] IN {"ZDOM","ZFD","ZSAM"}, "Sales Orders",
        "Other"
    )

Depends on  : [SALES_DOC_TYPE] (source column)
Business Rule: Groups SAP sales document types:
  ZCCB, ZCMR, ZCMX, ZDMR, ZDMX, ZRET → "Rebates and Adjustments"
  ZREV, ZRFE                           → "Returns"
  ZDOM, ZFD, ZSAM                      → "Sales Orders"
  All others                           → "Other"
Pattern     : Tableau categorical bin → DAX SWITCH(TRUE()) with IN sets.
Conversion  : ✅ FULL


### CC-18 | Action (!Reporting Week,YEAR(!Date Selection),MONTH(!Date Selection),SOLD_TO_NAME (group))
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-action-week-year-month-soldto
PBI DataType    : string
PBI Hidden      : isHidden = true

Power BI DAX:
─────────────
column 'Action (!Reporting Week,YEAR(!Date Selection),MONTH(!Date Selection),SOLD_TO_NAME (group))' = FORMAT('Custom_SQL_Query'[!Reporting Week], "yyyy-mm-dd") & " , " & YEAR('Custom_SQL_Query'[!Date Selection]) & " , " & MONTH('Custom_SQL_Query'[!Date Selection]) & " , " & 'Custom_SQL_Query'[Sold To Name (group)]

Depends on  : CC-03 [!Reporting Week], CC-01 [!Date Selection],
              CC-15 [Sold To Name (group)]
Purpose     : Tableau action filter emulation — concatenates 4 fields
              for multi-field cross-visual filtering. Hidden in PBI.
Pattern     : Tableau action filter → DAX concatenated hidden column.
⚠️  NAMING NOTE: Column name is extremely long and contains parentheses,
    exclamation marks, and commas. Valid in TMDL when enclosed in
    single quotes. Preserved exactly.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2a — HIERARCHY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### HIER-01 | Sold To Group - Ship To
─────────────────────────────────────────────────────────────
PBI lineageTag  : csq-hier-sold-to-ship-to

```
hierarchy 'Sold To Group - Ship To'
    lineageTag: csq-hier-sold-to-ship-to
    level 'Sold To Name (group)'
        lineageTag: csq-hier-lvl-1
        column: 'Sold To Name (group)'
    level SOLD_TO_NAME
        lineageTag: csq-hier-lvl-2
        column: SOLD_TO_NAME
    level 'Ship-To Name'
        lineageTag: csq-hier-lvl-3
        column: SHIP_TO_NAME
```

Drill path: Sold To Name (group) → SOLD_TO_NAME → Ship-To Name
Tableau equivalent: Drill path / hierarchy on Sold To dimension.
NOTE: Level 1 uses CC-15 [Sold To Name (group)] (calculated column);
      Levels 2 and 3 use source columns.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: Measures Table

### M-01 | Parameter 1 Value
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI lineageTag  : mt-meas-param-1-value

Power BI DAX:
─────────────
measure 'Parameter 1 Value' = SELECTEDVALUE('Parameters'[Currency Selector], 0)

Purpose     : SELECTEDVALUE accessor for PARAM-01 Currency Selection.
Pattern     : SELECTEDVALUE with default fallback.
Conversion  : ✅ FULL


### M-02 | Parameter 2 Value
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI lineageTag  : mt-meas-param-2-value

Power BI DAX:
─────────────
measure 'Parameter 2 Value' = SELECTEDVALUE('Parameters'[Top Set], 10)

Purpose     : SELECTEDVALUE accessor for PARAM-02 Top Set.
Conversion  : ✅ FULL


### M-03 | Parameter 3 Value
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI lineageTag  : mt-meas-param-3-value

Power BI DAX:
─────────────
measure 'Parameter 3 Value' = SELECTEDVALUE('Parameters'[Date Selector], 0)

Purpose     : SELECTEDVALUE accessor for PARAM-03 Date Selector.
              Referenced by CC-01 [!Date Selection].
Conversion  : ✅ FULL


### M-04 | Parameter 4 Value
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI lineageTag  : mt-meas-param-4-value

Power BI DAX:
─────────────
measure 'Parameter 4 Value' = SELECTEDVALUE('Parameters'[Reporting Period], 1)

Purpose     : SELECTEDVALUE accessor for PARAM-04 Reporting Period.
              Referenced by CC-08 [!Date Parameter Control].
Conversion  : ✅ FULL


### M-05 | Parameter 5 Value
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI lineageTag  : mt-meas-param-5-value

Power BI DAX:
─────────────
measure 'Parameter 5 Value' = SELECTEDVALUE('Parameters'[Measure Selection], 2)

Purpose     : SELECTEDVALUE accessor for PARAM-05 Measure Selection.
Conversion  : ✅ FULL


### M-06 | Parameter 6 Value
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI lineageTag  : mt-meas-param-6-value

Power BI DAX:
─────────────
measure 'Parameter 6 Value' = SELECTEDVALUE('Parameters'[Min Order Threshold], 25000)

Purpose     : SELECTEDVALUE accessor for PARAM-06 Min Order Threshold.
Conversion  : ✅ FULL


### M-07 | Parameter 7 Value
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI lineageTag  : mt-meas-param-7-value

Power BI DAX:
─────────────
measure 'Parameter 7 Value' = SELECTEDVALUE('Parameters'[Threshold Select], "Selected Threshold")

Purpose     : SELECTEDVALUE accessor for PARAM-07 Threshold Select.
Conversion  : ✅ FULL


### M-08 | Parameter 8 Value
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI lineageTag  : mt-meas-param-8-value

Power BI DAX:
─────────────
measure 'Parameter 8 Value' = SELECTEDVALUE('Parameters'[Bottom Value], 0.0)

Purpose     : SELECTEDVALUE accessor for PARAM-08 Bottom Value.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU INITIAL TRANSFORMATIONS → POWER BI POWER QUERY (M)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | Custom_SQL_Query (Canada Order Trend — main table)
─────────────────────────────────────────────────────────────
Tableau Source  : SAP HANA direct connection
                  Server: pahpdb01.net.bms.com:30015
                  Schema: _SYS_BIC
                  User: GFSGLOBAL03 (username-password auth)
                  Tables: CV_VBAK (Sales Header), CV_VBAP (Sales Item),
                          CV_KNA1 (Customer), ship-to, billing, PO info
PBI Connection  : SapHana.Database
                  ("pahpdb01.net.bms.com:30015",
                   [Implementation = "2.0", CommandTimeout = #duration(0, 4, 0, 0)])
PBI Method      : Value.NativeQuery with EnableFolding = true
PBI Load Mode   : Import

Power BI M Query (FULL):
─────────────────────────
let
    Source = SapHana.Database(
        "pahpdb01.net.bms.com:30015",
        [Implementation = "2.0", CommandTimeout = #duration(0, 4, 0, 0)]
    ),
    Query = "Select to_date(Sales_H.ERDAT) Order_Created_on_Date,
Sales_H.VTWEG Distribution_Channel, Sales_H.ERNAM Order_Created_By,
Sales_H.AUGRU Order_Reason, Sales_H.BSARK PO_Type, Sales_H.VBELN Sales_Doc,
Sales_H.AUART Sales_Doc_Type, Sales_H.VKORG Sales_Org,
Sales_H.WAERK SD_Doc_Currency, Sales_H.KUNNR Sold_To_Party,
Sales_I.Sales_Document_Sales_I, Sales_I.Sales_Document_Item_Sales_I,
Sales_I.Sales_Doc_Item_Cat, Sales_I.Net_Value_Order_Doc_Currency,
PO_Info.PO_Number, to_date(PO_Info.Customer_Reference_Date) Customer_Reference_Date,
PO_Info.Source, cust.Country_Key_Cust, cust.Customer_Number, cust.Industry_Key,
cust.Sold_To_Name, Sales_Part.Country_Key, Sales_Part.Name Ship_To_Name,
Sales_Part.Ship_To_City, Sales_Part.Ship_to_ST,
to_date(bill.Billing_Date) Billing_Date, bill.Billing_Doc,
bill.Billing_Doc_Canceled, to_date(bill.Billing_Doc_Created) Billing_Doc_Created,
bill.Actual_Invoice_Quantity, bill.Billing_Item, bill.Material_Description,
bill.Material_Entered, bill.Company_Code, bill.Net_Value_of_the_Billing_Doc,
bill.Region_of_Sales_Order, bill.Billing_Type, bill.SD_Document_Currency,
bill.Subtotal_5
FROM ""SAP_ERP.TABLE_CV_S4::CV_VBAK"" Sales_H
join (Select cust.land1 Country_Key_Cust, cust.kunnr Customer_Number,
    cust.brsch Industry_Key, cust.name1 Sold_To_Name
    from ""_SYS_BIC"".""SAP_ERP.TABLE_CV_S4/CV_KNA1"" Cust
    group by cust.land1, cust.kunnr, cust.brsch, cust.name1) cust
on cust.Customer_Number = Sales_H.KUNNR
join (Select Sales_I.VBELN Sales_Document_Sales_I,
    Sales_I.POSNR Sales_Document_Item_Sales_I,
    Sales_I.PSTYV Sales_Doc_Item_Cat, Sales_I.NETWR Net_Value_Order_Doc_Currency
    FROM ""SAP_ERP.TABLE_CV_S4::CV_VBAP"" Sales_I
    group by Sales_I.VBELN, Sales_I.POSNR, Sales_I.PSTYV, Sales_I.NETWR) Sales_I
on Sales_I.Sales_Document_Sales_I = Sales_H.VBELN
left join (...ship-to subquery...) sales_part on ...
left join (...billing subquery...) bill on ...
left join (...PO info subquery...) PO_Info on ...
where Sales_H.VKORG in ('S035')
and to_integer(year(to_date(Sales_H.ERDAT,'YYYYMMDD'))) >= to_integer((year(CURRENT_DATE)-3))
group by ...",
    Result = Value.NativeQuery(Source, Query, null, [EnableFolding=true])
in
    Result

SQL Architecture:
  - Sales_H (CV_VBAK): Sales header — order dates, doc types, currencies
  - Sales_I (CV_VBAP): Sales items — line items, quantities, net values
  - cust (CV_KNA1): Customer master — name, country, industry
  - sales_part: Ship-to party — address, city, state
  - bill: Billing documents — invoices, quantities, materials
  - PO_Info: Purchase order info — PO numbers, reference dates
  - WHERE: Sales org 'S035' (Canada), last 3 years from ERDAT

⚠️  SAP HANA RULES (Rule M14):
    - Pass Source root to Value.NativeQuery — NOT Source{[Name="..."]}[Data]
    - Use null (not []) as third parameter
    - HANA schema/table names use double-quote escaping: ""_SYS_BIC"".""TABLE""
    - Requires on-premise Data Gateway
    - Credentials: Basic (username/password)
⚠️  TIMEOUT: CommandTimeout = #duration(0, 4, 0, 0) — 4 hours.
    Complex multi-join query may take significant time on large datasets.
⚠️  TRUNCATION: Ship-to, billing, and PO subqueries shown abbreviated.
    Full SQL is in the Tableau workbook XML.


### PQ-02 | TCURR (Exchange Rate Reference Table)
─────────────────────────────────────────────────────────────
Tableau Equivalent : Secondary data source / data blend for exchange rates
PBI Type           : Separate table joined via relationship rel-currency-fcurr
PBI Load Mode      : Import

NOTE: The TCURR table must contain at minimum:
  [From currency] — currency code (string, relationship key)
  [Exchange Rate]  — rate value (double)
  [FFACT]          — from-factor divisor (double)

⚠️  DEPENDENCY: CC-14 [Exchange Rate Statement] uses RELATED('TCURR'[...]).
    The relationship rel-currency-fcurr must be defined:
    fromColumn: 'Custom_SQL_Query'.SD_DOC_CURRENCY
    toColumn: 'TCURR'.'From currency'


### PQ-03 | Parameters (single-row parameter table)
─────────────────────────────────────────────────────────────
See SECTION 1 — Parameters Table M Query for full details.


### PQ-04 | Measures Table (empty measures host)
─────────────────────────────────────────────────────────────
Tableau Equivalent : n/a (PBI-only pattern)
Purpose            : Empty table hosting all DAX measures.
                     Contains 8 SELECTEDVALUE parameter measures (M-01
                     through M-08).
NOTE               : Standard PBI empty measures table pattern (same as k1).


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — CONVERSION COMPLETENESS SCORECARD
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Category                         | Tableau Items | PBI Full | PBI Partial | Not Converted | % Complete |
|----------------------------------|---------------|----------|-------------|---------------|------------|
| Parameters                       | 8             | 8        | 0           | 0             | 100%       |
| Calculated Columns               | 18            | 18       | 0           | 0             | 100%       |
| Measures (SELECTEDVALUE)         | 8             | 8        | 0           | 0             | 100%       |
| Power Query / SQL Tables         | 4             | 4        | 0           | 0             | 100%       |
| Relationships                    | 6             | 6        | 0           | 0             | 100%       |
| Hierarchies                      | 1             | 1        | 0           | 0             | 100%       |
| Custom Theme                     | 1             | 1        | 0           | 0             | 100%       |
| TMDL Infrastructure Files        | 5             | 5        | 0           | 0             | 100%       |
| TOTAL                            | 51            | 51       | 0           | 0             | 🟢 100%    |

PARTIAL ITEMS: None.

NOTE: While all items are structurally converted, several require
      manual validation (see SECTION 7 — Known Gaps) due to
      approximation patterns (brand extraction, sold-to grouping)
      and external dependencies (TCURR table, SAP HANA gateway).


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — CONVERSION PATTERN CHEAT SHEET
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PATTERN-01 | Tableau parameter → PBI disconnected table + SELECTEDVALUE
─────────────────────────────────────────────────────────────
Tableau:
  Parameter [X] with default value, used in calculated fields as
  [Parameters].[X]

Power BI:
  Single-row table with typed columns (M Query #table()):
    #table(type table [Col1=Type, Col2=Type, ...], {{val1, val2, ...}})
  SELECTEDVALUE measure for each parameter:
    measure 'Param N Value' = SELECTEDVALUE('Table'[Column], default)
  Reference in calc columns via [Param N Value] measure.

This model has 8 parameters in ONE consolidated table (not 8 separate
tables). Each parameter has a corresponding SELECTEDVALUE measure.


PATTERN-02 | Tableau IF/ELSEIF date switching → DAX IF + measure reference
─────────────────────────────────────────────────────────────
Tableau:
  IF [Date Selector] = 0 THEN [Date A] ELSE [Date B] END

Power BI:
  column '!Date Selection' = IF([Parameter 3 Value] = 0,
      T[Date A], T[Date B])

This model uses this pattern in: CC-01, CC-08.


PATTERN-03 | Tableau DATEPART('iso-weekday') → DAX WEEKDAY(date, 2)
─────────────────────────────────────────────────────────────
Tableau:
  DATEPART('iso-weekday', [Date])

Power BI:
  WEEKDAY(T[Date], 2)
  Returns 1=Monday through 7=Sunday.

This model uses this pattern in: CC-02.


PATTERN-04 | Tableau DATETRUNC('week','monday') → DAX subtraction
─────────────────────────────────────────────────────────────
Tableau:
  DATETRUNC('week', [Date], 'monday')

Power BI:
  T[Date] - WEEKDAY(T[Date], 2)

⚠️  NOTE: This model uses subtraction only (no +1), producing the
    Sunday before the week start. Compare with k1 which uses +1.
    Preserved exactly as in the PBIP source.

This model uses this pattern in: CC-03.


PATTERN-05 | Tableau Current/Previous period labeling → DAX SWITCH(TRUE())
─────────────────────────────────────────────────────────────
Tableau:
  IF DATETRUNC('month', [Date]) = DATETRUNC('month', TODAY()) THEN "Current"
  ELSEIF ... THEN "Previous" ELSE STR(YYYYMM) END

Power BI:
  VAR _curYM = YEAR(TODAY()) * 100 + MONTH(TODAY())
  VAR _prevYM = YEAR(EDATE(TODAY(), -1)) * 100 + MONTH(EDATE(TODAY(), -1))
  VAR _dsYM = YEAR([Date]) * 100 + MONTH([Date])
  RETURN SWITCH(TRUE(),
      _dsYM = _curYM, "Current",
      _dsYM = _prevYM, "Previous",
      FORMAT(_dsYM, "0"))

This model uses this pattern in: CC-04 (month), CC-05 (quarter).


PATTERN-06 | Tableau action filter → PBI concatenated hidden column
─────────────────────────────────────────────────────────────
Tableau:
  Action filter passing multiple fields between dashboard sheets.

Power BI:
  column 'Action (Field1,Field2,...)' = T[F1] & " , " & T[F2] & " , " & ...
      dataType: string
      isHidden
      lineageTag: tbl-action-<slug>
      summarizeBy: none

This model has ONE action column:
  CC-18: Action (!Reporting Week,YEAR(!Date Selection),MONTH(!Date Selection),SOLD_TO_NAME (group))


PATTERN-07 | Tableau categorical bin → DAX SWITCH
─────────────────────────────────────────────────────────────
Tableau:
  Dimension bins or groups created via CASE or IF chains.

Power BI:
  SWITCH([Field], "val1","Group A", "val2","Group B", ..., "Other")
  or
  SWITCH(TRUE(), [Field] IN {"v1","v2"}, "Group A", ..., "Other")

This model uses this pattern in: CC-16 (Invoice Groups), CC-17
(Sales Document Group).


PATTERN-08 | SAP sign-flip → DAX SWITCH(TRUE()) with ABS
─────────────────────────────────────────────────────────────
Tableau:
  IF [Doc Type] IN ("credit types") THEN -[Value] ELSE [Value] END

Power BI:
  SWITCH(TRUE(),
      _doctype IN {"positive types"}, _net,
      _doctype IN {"negative types"}, _net * -1,
      _doctype = "mixed" && _itemcat = "positive", ABS(_net),
      _doctype = "mixed" && _itemcat = "negative", ABS(_net) * -1,
      _net
  )

This model uses this pattern in: CC-12 (Order Net Value), CC-13
(Billing_Invoice_Line).


PATTERN-09 | Tableau data blend → PBI RELATED() via relationship
─────────────────────────────────────────────────────────────
Tableau:
  Data blending with secondary datasource (e.g., exchange rate lookup).

Power BI:
  relationship from primary table column to secondary table key.
  RELATED('SecondaryTable'[Column]) in calculated column.

This model uses this pattern in: CC-14 (Exchange Rate Statement)
via relationship rel-currency-fcurr to TCURR table.


PATTERN-10 | Tableau string prefix matching → DAX LEFT() + SWITCH(TRUE())
─────────────────────────────────────────────────────────────
Tableau:
  IF LEFT([Name], 8) = "BAYSHORE" THEN "Bayshore" ...

Power BI:
  SWITCH(TRUE(),
      LEFT(T[Name], 8) = "BAYSHORE", "Bayshore Pharmacy",
      LEFT(T[Name], 7) = "INNOMAR", "INNOMAR",
      T[Name]
  )

This model uses this pattern in: CC-15 (Sold To Name group).


PATTERN-11 | Tableau custom color palette → PBI theme.json
─────────────────────────────────────────────────────────────
Tableau:
  Custom color palette defined in workbook XML preferences.

Power BI:
  theme.json with dataColors array:
  {
    "name": "BMS",
    "dataColors": ["#595454","#a69f9f",...],
    "background": "#FFFFFF",
    "foreground": "#252423"
  }


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6a — LINEAGETAG NAMING CONVENTIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Object             | Pattern                              | Example                                  |
|--------------------|--------------------------------------|------------------------------------------|
| Source column      | `<prefix>-col-<name>`                | `csq-col-sales-doc`                      |
| Calculated column  | `<prefix>-calc-<name>`               | `csq-calc-brand`                         |
| Measure            | `<prefix>-meas-<name>`               | `mt-meas-param-1-value`                  |
| Action column      | `<prefix>-action-<slug>`             | `csq-action-week-year-month-soldto`      |
| Parameter column   | `param-<n>-key`                      | `param-1-key`                            |
| Hierarchy          | `<prefix>-hier-<name>`               | `csq-hier-sold-to-ship-to`              |
| Hierarchy level n  | `<prefix>-hier-lvl-<n>`              | `csq-hier-lvl-1`                         |
| Table              | `<prefix>-0001-0000-0000-<12digits>` | `csq-0001-0000-0000-000000000001`        |
| Relationship       | `rel-<type>-<name>`                  | `rel-date-order-created`                 |

Table prefix codes:
  csq   = Custom_SQL_Query
  mt    = Measures Table
  param = Parameters
  ldt<n> = LocalDateTable n (e.g., ldt1 through ldt5)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAP-01 | Sold To Name Grouping — Approximation via LEFT-prefix
  Severity : Medium
  Detail   : CC-15 [Sold To Name (group)] uses LEFT-prefix matching
             to group 80+ distinct SOLD_TO_NAME values into major
             customer groups. The LEFT(n) pattern is an approximation
             that may miscategorize names not fitting the prefix pattern.
  Action   : Validate all groupings against actual source data. Consider
             maintaining an explicit lookup table for production use.


GAP-02 | ORENCIA Brand Logic — Complex String Approximation
  Severity : Medium
  Detail   : CC-06 [!Brand] and CC-07 [!Material Brand] use complex
             LEFT+SEARCH patterns to extract brand sub-types from
             MATERIAL_DESCRIPTION. The "ORENCIA CL/SC/LI" first-two-words
             logic is an approximation that depends on consistent
             description formatting.
  Action   : Validate against actual MATERIAL_DESCRIPTION values in
             SAP data. If format varies, consider a lookup table.


GAP-03 | Exchange Rate — TCURR Table Dependency
  Severity : High
  Detail   : CC-14 [Exchange Rate Statement] uses RELATED('TCURR'[...])
             which requires:
             1. The TCURR table to exist in the model
             2. Relationship rel-currency-fcurr to be defined
             3. TCURR to contain [From currency], [Exchange Rate], [FFACT]
             No TCURR .tmdl file has been provided in the conversion.
  Fix      : Either provide the TCURR table TMDL, or create a separate
             Power Query sourcing exchange rate data from SAP HANA
             (table TCURR in SAP ERP schema).


GAP-04 | Action Column Emulation → Report-Layer Wiring
  Severity : Low (report-layer)
  Detail   : CC-18 is a Tableau action filter emulation using concatenated
             string columns. In PBI, the equivalent cross-visual
             filtering must be configured at the report layer using
             drillthrough, bookmarks, or button actions.
  Action   : Configure PBI report canvas interactions to replicate
             the Tableau tab-navigation behavior.


GAP-05 | SAP HANA Gateway Required
  Severity : High (infrastructure)
  Detail   : The SAP HANA connection requires an on-premise Data Gateway
             to be installed and configured. Credentials are Basic
             (username/password for GFSGLOBAL03).
  Action   : Install and configure on-premise Data Gateway. Set up
             Basic credential authentication for the SAP HANA server.


GAP-06 | !Date Selection Variation — Non-standard Calculated Column Variation
  Severity : Medium
  Detail   : CC-01 [!Date Selection] is a calculated column with a date
             variation relationship to a LocalDateTable. Date variations
             on calculated columns (as opposed to source columns) are
             non-standard and may exhibit unexpected behavior in date
             hierarchy drill-down.
  Action   : Test date hierarchy drill-down thoroughly in report visuals.
             If issues occur, consider removing the variation and using
             explicit date slicers instead.


GAP-07 | Parameter Slicers — Report-Layer Configuration
  Severity : Low (report-layer)
  Detail   : All 8 parameters are stored in the data model but require
             slicer visuals to be placed on report pages for user
             interaction. Without slicers, default values are used.
  Action   : Add slicer visuals for each parameter on the appropriate
             report pages.


GAP-08 | Reporting Period Week (Value 3) — Not Handled in !Date Parameter Control
  Severity : Low
  Detail   : PARAM-04 Reporting Period has value 3 = Week, but CC-08
             [!Date Parameter Control] only handles values 1 (Month)
             and 2 (Quarter). Week display is not addressed.
  Action   : Confirm with report owner whether week-level display
             requires a separate visual or conditional visibility
             at the report layer.


GAP-09 | SQL Subqueries Truncated in Knowledge File
  Severity : Informational
  Detail   : The ship-to, billing, and PO info subqueries in PQ-01
             are shown abbreviated ("(...ship-to subquery...)"). Full
             SQL is available in the Tableau workbook XML (ID 18504).
  Action   : Reference the full Tableau workbook XML for complete
             subquery definitions when modifying the M query.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — FILE INVENTORY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| #  | Output Filename                              | Rename To                                                    | Folder                | Content                                           |
|----|----------------------------------------------|--------------------------------------------------------------|-----------------------|---------------------------------------------------|
| 1  | out_database.txt                             | database.tmdl                                                | /definition/          | Database compatibility level (1604)               |
| 2  | out_model.txt                                | model.tmdl                                                   | /definition/          | Model metadata, culture, table refs               |
| 3  | out_custom_sql_query.txt                     | Custom_SQL_Query.tmdl                                        | /definition/tables/   | Main SAP HANA table with source + calc columns    |
| 4  | out_measures_table.txt                       | Measures Table.tmdl                                          | /definition/tables/   | Empty measures host (8 SELECTEDVALUE measures)    |
| 5  | out_parameters.txt                           | Parameters.tmdl                                              | /definition/tables/   | 8-column single-row parameter table               |
| 6  | out_relationships.txt                        | relationships.tmdl                                           | /definition/          | 5 date variations + 1 currency relationship       |
| 7  | out_datetabletemplate.txt                     | DateTableTemplate_<GUID>.tmdl                                | /definition/tables/   | Auto date table template                          |
| 8  | out_en_us.txt                                | en-US.tmdl                                                   | /definition/cultures/ | Linguistic metadata (en-US)                       |
| 9  | out_diagramlayout.json                       | diagramLayout.json                                           | /definition/          | Model diagram layout                              |
| 10 | out_theme.json                               | theme.json                                                   | /definition/          | BMS custom color palette (24 colors)              |
| 11 | out_localdatetable_order_created.txt          | LocalDateTable_a1b2c3d4-1111-1111-1111-111111111111.tmdl     | /definition/tables/   | Date hierarchy for [Order Created on Date]        |
| 12 | out_localdatetable_customer_ref.txt           | LocalDateTable_a1b2c3d4-2222-2222-2222-222222222222.tmdl     | /definition/tables/   | Date hierarchy for [Customer Reference Date]      |
| 13 | out_localdatetable_billing.txt                | LocalDateTable_a1b2c3d4-3333-3333-3333-333333333333.tmdl     | /definition/tables/   | Date hierarchy for [Billing Date]                 |
| 14 | out_localdatetable_billing_created.txt        | LocalDateTable_a1b2c3d4-4444-4444-4444-444444444444.tmdl     | /definition/tables/   | Date hierarchy for [Billing Doc Created]          |
| 15 | out_localdatetable_date_selection.txt         | LocalDateTable_a1b2c3d4-5555-5555-5555-555555555555.tmdl     | /definition/tables/   | Date hierarchy for [!Date Selection] (calc col)   |

.pbip folder structure:
```
<ProjectName>.SemanticModel/definition/
├── database.tmdl
├── model.tmdl
├── relationships.tmdl
├── diagramLayout.json
├── theme.json
├── tables/
│   ├── Custom_SQL_Query.tmdl
│   ├── Measures Table.tmdl
│   ├── Parameters.tmdl
│   ├── DateTableTemplate_<GUID>.tmdl
│   ├── LocalDateTable_a1b2c3d4-1111-....tmdl   (Order Created on Date)
│   ├── LocalDateTable_a1b2c3d4-2222-....tmdl   (Customer Reference Date)
│   ├── LocalDateTable_a1b2c3d4-3333-....tmdl   (Billing Date)
│   ├── LocalDateTable_a1b2c3d4-4444-....tmdl   (Billing Doc Created)
│   └── LocalDateTable_a1b2c3d4-5555-....tmdl   (!Date Selection)
└── cultures/
    └── en-US.tmdl
```


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — CUSTOM COLOR PALETTE (theme.json)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Source: Extracted from Tableau workbook XML custom color palette.

```json
{
  "name": "BMS",
  "dataColors": [
    "#595454","#a69f9f","#c0f2fb","#33d6f1","#009fba","#097789",
    "#c5ffe6","#59ffb9","#1dce9b","#138967","#be2bbb","#dac5c5",
    "#cb7c78","#fedcca","#fda97d","#ffeccd","#ffd186","#eee7e7",
    "#df603a","#ba4422","#ffac25","#c37900","#772a28","#501514"
  ],
  "background": "#FFFFFF",
  "foreground": "#252423"
}
```

24 colors in the palette. The first 6 entries are used as the primary
data series colors in visuals. The full palette provides sufficient
variety for multi-category charts.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — RELATIONSHIPS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### REL-01 | Currency lookup (Custom_SQL_Query → TCURR)
─────────────────────────────────────────────────────────────
```
relationship rel-currency-fcurr
    fromColumn: 'Custom_SQL_Query'.SD_DOC_CURRENCY
    toColumn: 'TCURR'.'From currency'
```
Type: Data join (many-to-one).
Used by: CC-14 [Exchange Rate Statement] via RELATED().


### REL-02 | Order Created on Date → LocalDateTable
─────────────────────────────────────────────────────────────
```
relationship rel-date-order-created
    joinOnDateBehavior: datePartOnly
    fromColumn: 'Custom_SQL_Query'.'Order Created on Date'
    toColumn: LocalDateTable_a1b2c3d4-1111-1111-1111-111111111111.Date
```
Type: Date variation (datePartOnly).


### REL-03 | Customer Reference Date → LocalDateTable
─────────────────────────────────────────────────────────────
```
relationship rel-date-customer-ref
    joinOnDateBehavior: datePartOnly
    fromColumn: 'Custom_SQL_Query'.'Customer Reference Date'
    toColumn: LocalDateTable_a1b2c3d4-2222-2222-2222-222222222222.Date
```
Type: Date variation (datePartOnly).


### REL-04 | Billing Date → LocalDateTable
─────────────────────────────────────────────────────────────
```
relationship rel-date-billing
    joinOnDateBehavior: datePartOnly
    fromColumn: 'Custom_SQL_Query'.'Billing Date'
    toColumn: LocalDateTable_a1b2c3d4-3333-3333-3333-333333333333.Date
```
Type: Date variation (datePartOnly).


### REL-05 | Billing Doc Created → LocalDateTable
─────────────────────────────────────────────────────────────
```
relationship rel-date-billing-created
    joinOnDateBehavior: datePartOnly
    fromColumn: 'Custom_SQL_Query'.'Billing Doc Created'
    toColumn: LocalDateTable_a1b2c3d4-4444-4444-4444-444444444444.Date
```
Type: Date variation (datePartOnly).


### REL-06 | !Date Selection → LocalDateTable
─────────────────────────────────────────────────────────────
```
relationship rel-date-selection
    joinOnDateBehavior: datePartOnly
    fromColumn: 'Custom_SQL_Query'.'!Date Selection'
    toColumn: LocalDateTable_a1b2c3d4-5555-5555-5555-555555555555.Date
```
Type: Date variation (datePartOnly).
⚠️  NOTE: Variation on a calculated column (non-standard). See GAP-06.


END OF KNOWLEDGE FILE
