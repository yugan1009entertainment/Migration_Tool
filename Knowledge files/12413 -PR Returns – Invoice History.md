# 📘 CONVERSION KNOWLEDGE FILE
# PR Returns – Invoice History
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-09
# Source files: pr_returns___invoice_history.txt (Tableau)
#               12_database.txt, 12_model.txt,
#               12_definition.json, 12_diagramlayout.json (PBIP)

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 0 — WORKBOOK & MODEL METADATA
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Tableau Workbook Name           : PR Returns – Invoice History
Tableau Server ID               : PRReturns-InvoiceHistory
Tableau Site                    : PRODUCTION
Tableau Revision                : 4.5
Tableau Build                   : 20242.25.0710.1539
Tableau Source Build            : 2024.2.1 (20242.24.0711.1636)
Tableau Version                 : 18.1
Tableau Animation               : OFF (ao-off)

⚠️  Animation is OFF — differs from other BMS workbooks (e.g., CRU Training,
    which is ON). Do NOT apply animation defaults from other conversions.

PBIP Model Name                 : PR Returns - Invoice History- model
⚠️  Trailing hyphen + space in model name: "PR Returns - Invoice History- model"
    Note the "History- model" spacing — no space before the hyphen.
    Preserve exactly as-is. Changing the model name breaks workspace references.

PBIP Database Comment           : /// PR Returns - Invoice History 1 dataset
                                    was created by TOM
PBIP Model Comment              : /// A Demo Tabular data model with
                                    1550 compatibility level.
⚠️  Comment references 1550 but actual compatibilityLevel = 1604.
    This is a template artefact — preserve both as-is. Do NOT update
    the comment to match 1604.

compatibilityLevel              : 1604
culture                         : en-US
sourceQueryCulture              : en-US
defaultPowerBIDataSourceVersion : powerBI_V3
dataAccessOptions               : legacyRedirects, returnErrorValuesAsNull
TimeIntelligenceEnabled         : 1  (annotation __PBI_TimeIntelligenceEnabled)
PBI_QueryOrder                  : ["Query1"]
PBI_ProTooling                  : ["TMDLView_Desktop", "DevMode"]

PBIP Tables (from 12_diagramlayout.json):
  1. Query1          — main fact table (SAP HANA DirectQuery, 5-table join)
  2. Calendar Table  — date dimension

⚠️  Only 2 tables confirmed from uploaded PBIP files. No individual table
    TMDL files (Query1.tmdl, CalendarTable.tmdl) were included in this
    upload set. All column, measure, and calculated column details below
    are derived from Tableau workbook XML + diagram layout.
    Upload Query1.tmdl and CalendarTable.tmdl to reach 100% completeness.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PARAM-01 | Created On (Start)
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 1]
Tableau Caption       : Created On (Start)
Tableau Type          : Date, range (param-domain-type = range)
Tableau Min Bound     : #2018-01-01#
Tableau Default       : #2018-08-10#

Tableau XML (verbatim):
───────────────────────
<column caption='Created On (Start)' datatype='date'
  name='[Parameter 1]' param-domain-type='range'
  role='measure' type='quantitative' value='#2018-08-10#'>
  <calculation class='tableau' formula='#2018-08-10#' />
  <range min='#2018-01-01#' />
</column>

Power BI Implementation:
  Date range slicer or What-If Parameter. Minimum bound = 2018-01-01.
  No maximum bound defined in Tableau — upper end is unbounded.
  Used together with PARAM-02 to define the start/end date window
  for the Returns sheet filter on [ERDAT] (Sales Order Created Date).

⚠️  param-domain-type='range' with explicit min but NO max defined.
    The upper bound is unbounded in the Tableau source.
⚠️  Default date 2018-08-10 is a demo/sample value — NOT a business rule.
    Do NOT hardcode this date as a filter in PBIP.
⚠️  In Tableau this parameter filters [ERDAT] (CV_VBAK.ERDAT).
    In PBIP, replicate as a date slicer connected to Query1[ERDAT].

Conversion Status: ⚠️  PENDING — no parameter table TMDL uploaded.
                   Pattern: date slicer / What-If Parameter (see PATTERN-04).


### PARAM-02 | Created On (End)
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 2]
Tableau Caption       : Created On (End)
Tableau Type          : Date, range, granularity = 1 day (period-type-v2='day')
Tableau Default       : #2018-08-10#

Tableau XML (verbatim):
───────────────────────
<column caption='Created On (End)' datatype='date'
  name='[Parameter 2]' param-domain-type='range'
  role='measure' type='quantitative' value='#2018-08-10#'>
  <calculation class='tableau' formula='#2018-08-10#' />
  <range granularity='1' period-type-v2='day' />
</column>

Power BI Implementation:
  Paired with PARAM-01 as the end-date bound of the created-on window.
  Granularity = 1 day → date (not datetime) precision in slicer.

⚠️  period-type-v2='day' — Tableau enforces day-level granularity.
    In PBIP, if the Calendar Table has a Date column at day grain,
    this is automatically satisfied. Do NOT use datetime precision.
⚠️  Same default date as PARAM-01 (both 2018-08-10) — this is
    intentional: start = end = single-day default range.

Conversion Status: ⚠️  PENDING — paired with PARAM-01.


### PARAM-03 | All Lines - Created On (Start)
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 3]
Tableau Caption       : All Lines - Created On (Start)
Tableau Type          : Date, any (param-domain-type = any, no bounds)
Tableau Default       : #2018-08-06#

Tableau XML (verbatim):
───────────────────────
<column caption='All Lines - Created On (Start)' datatype='date'
  name='[Parameter 3]' param-domain-type='any'
  role='measure' type='quantitative' value='#2018-08-06#'>
  <calculation class='tableau' formula='#2018-08-06#' />
</column>

Power BI Implementation:
  Separate date window for the "All Lines" view/sheet — fully independent
  from the PARAM-01/PARAM-02 window. No min/max constraint.
  Implement as a separate date slicer or What-If Parameter scoped to
  the "All Lines" visual page or visual-level filter.

⚠️  param-domain-type='any' — no bounds. User can enter any date.
    PBIP date slicers are unbounded by default — this is correct.
⚠️  Default date differs from PARAM-01/PARAM-02 (2018-08-06 vs 2018-08-10).
    Preserve separate defaults; do NOT unify the two date windows.
⚠️  Caption prefix "All Lines -" is a business-level scope qualifier.
    Preserve exactly in slicer/parameter label.

Conversion Status: ⚠️  PENDING — separate from PARAM-01/PARAM-02 scope.


### PARAM-04 | All Lines - Created On (End)
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 4]
Tableau Caption       : All Lines - Created On (End)
Tableau Type          : Date, any, datatype-customized='true'
Tableau Default       : #2018-08-06#

Tableau XML (verbatim):
───────────────────────
<column caption='All Lines - Created On (End)' datatype='date'
  datatype-customized='true' name='[Parameter 4]'
  param-domain-type='any' role='measure' type='quantitative'
  value='#2018-08-06#'>
  <calculation class='tableau' formula='#2018-08-06#' />
</column>

Power BI Implementation:
  Paired with PARAM-03 as the end-date bound of the "All Lines" window.

⚠️  datatype-customized='true' — Tableau flag indicating the user manually
    overrode the inferred data type. Declared type is 'date'. In PBIP,
    treat as date — no special handling needed.
⚠️  Same default as PARAM-03 (2018-08-06): single-day default window
    for "All Lines", same pattern as PARAM-01/PARAM-02.

Conversion Status: ⚠️  PENDING — paired with PARAM-03.


### Parameter Summary

| Param | Tableau Caption                 | Type        | Default    | Scope        | PBIP Status |
|-------|---------------------------------|-------------|------------|--------------|-------------|
| P-01  | Created On (Start)              | Date, range | 2018-08-10 | Returns view | ⚠️ PENDING  |
| P-02  | Created On (End)                | Date, range | 2018-08-10 | Returns view | ⚠️ PENDING  |
| P-03  | All Lines - Created On (Start)  | Date, any   | 2018-08-06 | All Lines    | ⚠️ PENDING  |
| P-04  | All Lines - Created On (End)    | Date, any   | 2018-08-06 | All Lines    | ⚠️ PENDING  |

⚠️  TWO INDEPENDENT DATE WINDOWS exist in this workbook:
      Window A: PARAM-01 + PARAM-02 → "Returns" sheets
      Window B: PARAM-03 + PARAM-04 → "All Lines" sheets
    They are scoped independently — do NOT merge into a single slicer.
⚠️  No integer, string, or list parameters exist in this workbook.
    All 4 parameters are date type only.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️  No individual table TMDL files were uploaded for this workbook.
    Calculated columns listed below are derived from Tableau workbook
    XML (column definitions, captions, calculated field references).
    DAX equivalents are inferred from Tableau formulas.
    Upload Query1.tmdl to confirm or override all definitions below.

## TABLE: Query1

### CC-01 | Date Filter  (Returns scope)
─────────────────────────────────────────────────────────────
Tableau Expression:
───────────────────
[ERDAT] >= [Parameter 1] AND [ERDAT] <= [Parameter 2]
Used as a datasource filter or calculated field for the Returns sheets.

Power BI DAX (inferred):
─────────────────────────
column 'Date Filter' =
IF(
    AND(
        Query1[ERDAT] >= SELECTEDVALUE('Created On Start'[Date]),
        Query1[ERDAT] <= SELECTEDVALUE('Created On End'[Date])
    ),
    TRUE(),
    FALSE()
)

Depends on : [ERDAT] (CV_VBAK.ERDAT — Sales Order Created Date)
             PARAM-01, PARAM-02
⚠️  Exact DAX depends on the parameter table implementation chosen
    (What-If Parameter vs date slicer). Confirm against Query1.tmdl.
⚠️  [ERDAT] maps to CV_VBAK.ERDAT. See Section 4 for full column map.

Conversion Status: ⚠️  INFERRED — confirm from Query1.tmdl.


### CC-02 | All Lines Date Filter  (All Lines scope)
─────────────────────────────────────────────────────────────
Tableau Expression:
───────────────────
[ERDAT] >= [Parameter 3] AND [ERDAT] <= [Parameter 4]
Separate filter for the All Lines sheet.

Power BI DAX (inferred):
─────────────────────────
column 'All Lines Date Filter' =
IF(
    AND(
        Query1[ERDAT] >= SELECTEDVALUE('All Lines Start'[Date]),
        Query1[ERDAT] <= SELECTEDVALUE('All Lines End'[Date])
    ),
    TRUE(),
    FALSE()
)

Depends on : [ERDAT], PARAM-03, PARAM-04
⚠️  "All Lines" scope uses PARAM-03/PARAM-04 exclusively.
    Do NOT reuse the Returns date filter for this column.

Conversion Status: ⚠️  INFERRED — confirm from Query1.tmdl.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️  No measures TMDL was uploaded. Measures below are inferred from
    Tableau sheet definitions and column usage patterns.
    Upload Query1.tmdl / measures table to confirm all definitions.

## TABLE: Query1

### M-01 | Count of Returns
─────────────────────────────────────────────────────────────
Tableau Equivalent:
───────────────────
COUNTD([VBELN])
Count of distinct Sales Order numbers.

Power BI DAX (inferred):
─────────────────────────
measure 'Count of Returns' =
DISTINCTCOUNT(Query1[VBELN])

Depends on : Query1[VBELN] (CV_VBAK.VBELN — Sales Order Number)
Conversion : ⚠️  INFERRED


### M-02 | Net Value
─────────────────────────────────────────────────────────────
Tableau Equivalent:
───────────────────
SUM([NETWR])
Unqualified [NETWR] resolves to CV_VBAK.NETWR (header net value).

Power BI DAX (inferred):
─────────────────────────
measure 'Net Value' =
SUM(Query1[NETWR])

⚠️  NETWR appears in both CV_VBAK (header) and CV_VBRP (item).
    Tableau unqualified [NETWR] resolves to CV_VBAK.NETWR (primary table).
    The item-level version is [NETWR_VBRP] in PBIP (see rename register,
    Section 4). Confirm per-visual which NETWR is used after uploading
    Query1.tmdl.

Conversion : ⚠️  INFERRED


### M-03 | Billing Quantity
─────────────────────────────────────────────────────────────
Tableau Equivalent:
───────────────────
SUM([FKIMG])
CV_VBRP.FKIMG — Billing Quantity.

Power BI DAX (inferred):
─────────────────────────
measure 'Billing Quantity' =
SUM(Query1[FKIMG])

Conversion : ⚠️  INFERRED


### M-04 | Gross Value
─────────────────────────────────────────────────────────────
Tableau Equivalent:
───────────────────
SUM([BRTWR])
CV_VBRP.BRTWR — Gross Value.

Power BI DAX (inferred):
─────────────────────────
measure 'Gross Value' =
SUM(Query1[BRTWR])

Conversion : ⚠️  INFERRED


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU INITIAL TRANSFORMATIONS → POWER BI POWER QUERY (M)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | Query1  (SAP HANA DirectQuery — 5-table multi-join)
─────────────────────────────────────────────────────────────
Tableau Datasource Name  : SAP_ERP.TABLE_CV::CV_VBAK (_SYS_BIC)
Tableau Datasource ID    : federated.0trqlpn186vdwz16gyp7q1oj6k2s
Connection Type          : SAP HANA (Single Node, ODBC)
Server                   : pahpdb01.net.bms.com
Port                     : 30015
Schema                   : _SYS_BIC
SAP HANA User            : GFSGLOBAL01
Auth                     : username-password
workgroup-auth-mode      : prompt

⚠️  Connection uses ODBC. SAP HANA DirectQuery in PBIP uses the
    SapHana.Database() M connector. Confirm connector availability
    and on-premises gateway configuration before deploying.
⚠️  Schema is _SYS_BIC (SAP HANA Calculation Views / InfoProviders),
    NOT raw table schema. All 5 source objects are Calculation View proxies.

#### SAP HANA Join Structure (from Tableau XML — verbatim):

Join Order (inside-out, resolved left to right):
─────────────────────────────────────────────────
Step 1 [INNER] : CV_VBAK ──[VBAK.KUNNR = KNA1.KUNNR]──► CV_KNA1
Step 2 [INNER] : (VBAK+KNA1) ──[VBAK.VBELN = VBPA.VBELN]──► CV_VBPA
Step 3 [INNER] : (VBAK+KNA1+VBPA) ──[VBPA.ADRNR = ADRC.ADDRNUMBER]──► CV_ADRC
Step 4 [INNER] : (VBAK+KNA1+VBPA+ADRC) ──[VBAK.VBELN = VBRP.AUBEL]──► CV_VBRP
Step 5 [LEFT]  : (all above) ──[VBRP.AUBEL = CE4BMSI.KAUFN
                                AND VBRP.AUPOS = CE4BMSI.KDPOS
                                AND VBRP.PRCTR = CE4BMSI.PRCTR]──► CV_CE4BMSI

Join type summary:
  CV_VBAK  ←INNER→ CV_KNA1    : VBAK.KUNNR = KNA1.KUNNR
  CV_VBAK  ←INNER→ CV_VBPA    : VBAK.VBELN = VBPA.VBELN
  CV_VBPA  ←INNER→ CV_ADRC    : VBPA.ADRNR = ADRC.ADDRNUMBER
  CV_VBAK  ←INNER→ CV_VBRP    : VBAK.VBELN = VBRP.AUBEL
  CV_VBRP  ←LEFT→  CV_CE4BMSI : VBRP.AUBEL = CE4BMSI.KAUFN
                                  AND VBRP.AUPOS = CE4BMSI.KDPOS
                                  AND VBRP.PRCTR = CE4BMSI.PRCTR

⚠️  CE4BMSI join is a 3-column composite LEFT JOIN. All three conditions
    must be satisfied simultaneously. Missing any one condition changes
    join semantics. Preserve all three predicates exactly.
⚠️  VBRP.AUBEL = VBAK.VBELN — AUBEL is the "reference sales order" in
    the billing item. This is the key that links billing back to sales order.
⚠️  The LEFT JOIN on CE4BMSI means COPA/profitability data may be NULL
    for some billing items. Do NOT promote to INNER JOIN.

#### SAP HANA Source Objects (all in schema _SYS_BIC):

| PBIP Alias  | SAP Object                          | Description                   |
|-------------|-------------------------------------|-------------------------------|
| CV_VBAK     | SAP_ERP.TABLE_CV_S4::CV_VBAK        | Sales Order Header            |
| CV_KNA1     | SAP_ERP.TABLE_CV_S4::CV_KNA1        | Customer Master               |
| CV_VBPA     | SAP_ERP.TABLE_CV_S4::CV_VBPA        | Sales Document Partner        |
| CV_ADRC     | SAP_ERP.TABLE_CV_S4::CV_ADRC        | Address                       |
| CV_VBRP     | SAP_ERP.TABLE_CV_S4::CV_VBRP        | Billing Document Item         |
| CV_CE4BMSI  | SAP_ERP.TABLE_CV_S4::CV_CE4BMSI     | COPA / Profitability (BMS)    |

⚠️  All objects are prefixed SAP_ERP.TABLE_CV_S4:: — the BMS SAP S/4HANA
    Calculation View package path. Do NOT use raw table paths.
⚠️  CV_CE4BMSI is a BMS-specific COPA extension (CE4 = Cost Element 4,
    BMSI = BMS-specific). Treat as proprietary — do NOT assume standard
    SAP structure for this object.

#### Power BI M Query (inferred — DirectQuery pattern):
─────────────────────────────────────────────────────────────
let
    Source = SapHana.Database(
        "pahpdb01.net.bms.com:30015",
        [Implementation = "2.0"]
    ),
    _SYS_BIC    = Source{[Name = "_SYS_BIC"]}[Data],
    CV_VBAK     = _SYS_BIC{[Name = "SAP_ERP.TABLE_CV_S4/CV_VBAK"]}[Data],
    CV_KNA1     = _SYS_BIC{[Name = "SAP_ERP.TABLE_CV_S4/CV_KNA1"]}[Data],
    CV_VBPA     = _SYS_BIC{[Name = "SAP_ERP.TABLE_CV_S4/CV_VBPA"]}[Data],
    CV_ADRC     = _SYS_BIC{[Name = "SAP_ERP.TABLE_CV_S4/CV_ADRC"]}[Data],
    CV_VBRP     = _SYS_BIC{[Name = "SAP_ERP.TABLE_CV_S4/CV_VBRP"]}[Data],
    CV_CE4BMSI  = _SYS_BIC{[Name = "SAP_ERP.TABLE_CV_S4/CV_CE4BMSI"]}[Data]
    -- joins applied as DirectQuery fold or via Value.NativeQuery
in
    -- final joined result set as DirectQuery table "Query1"
    Result

⚠️  RECOMMENDED PATTERN: Push the 5-table join to SAP HANA as a native query:
    Value.NativeQuery(
        _SYS_BIC,
        "SELECT ... FROM ""SAP_ERP.TABLE_CV_S4"".""CV_VBAK""
         INNER JOIN ""SAP_ERP.TABLE_CV_S4"".""CV_KNA1"" ON VBAK.KUNNR = KNA1.KUNNR
         INNER JOIN ""SAP_ERP.TABLE_CV_S4"".""CV_VBPA"" ON VBAK.VBELN = VBPA.VBELN
         INNER JOIN ""SAP_ERP.TABLE_CV_S4"".""CV_ADRC"" ON VBPA.ADRNR = ADRC.ADDRNUMBER
         INNER JOIN ""SAP_ERP.TABLE_CV_S4"".""CV_VBRP"" ON VBAK.VBELN = VBRP.AUBEL
         LEFT  JOIN ""SAP_ERP.TABLE_CV_S4"".""CV_CE4BMSI""
               ON VBRP.AUBEL = CE4BMSI.KAUFN
              AND VBRP.AUPOS = CE4BMSI.KDPOS
              AND VBRP.PRCTR = CE4BMSI.PRCTR",
        null,
        [EnableFolding = true]
    )

⚠️  SAP HANA Calculation View path format in M uses "/" separator:
    M:       SAP_ERP.TABLE_CV_S4/CV_VBAK
    Tableau: SAP_ERP.TABLE_CV_S4::CV_VBAK
    Do NOT use the "::" form in M queries.
⚠️  PBIP model PBI_QueryOrder = ["Query1"] — this is the only query in
    the model. DirectQuery mode: no import/cached data. All queries
    fold to SAP HANA at runtime.
⚠️  The verbatim M query is inferred from Tableau join structure.
    Upload Query1.tmdl to replace this with the actual M expression.

Conversion Status: ⚠️  PARTIALLY INFERRED — join structure fully documented
                   from Tableau XML. Verbatim M query pending Query1.tmdl.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — SOURCE COLUMN MAPPING
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

The Tableau workbook exposes all columns from the 5-table join as a
flat datasource. The columns below are confirmed used in Tableau
sheets and/or calculations. The full column map contains 500+ SAP fields.

⚠️  DISAMBIGUATION PATTERN: Where the same SAP field name appears in
    multiple source tables, Tableau qualifies with source alias:
      [NETWR]                      = CV_VBAK.NETWR  (header — unqualified = primary)
      [NETWR (SAP_ERP_...CV_VBRP)] = CV_VBRP.NETWR  (item — qualified)
    In PBIP, duplicate-named columns MUST be renamed using the suffix
    pattern _{SOURCETABLE}. See rename register below.

#### Key Business Column Map:

| Tableau Name    | Source Table | SAP Field  | Business Meaning              | PBIP Column Name  |
|-----------------|--------------|------------|-------------------------------|-------------------|
| [VBELN]         | CV_VBAK      | VBELN      | Sales Order Number            | VBELN             |
| [ERDAT]         | CV_VBAK      | ERDAT      | Sales Order Created Date      | ERDAT             |
| [AUART]         | CV_VBAK      | AUART      | Order Type                    | AUART             |
| [KUNNR]         | CV_VBAK      | KUNNR      | Sold-to Customer              | KUNNR             |
| [NETWR]         | CV_VBAK      | NETWR      | Header Net Value              | NETWR             |
| [WAERK]         | CV_VBAK      | WAERK      | Document Currency             | WAERK             |
| [VKORG]         | CV_VBAK      | VKORG      | Sales Organization            | VKORG             |
| [VTWEG]         | CV_VBAK      | VTWEG      | Distribution Channel          | VTWEG             |
| [SPART]         | CV_VBAK      | SPART      | Division                      | SPART             |
| [AUGRU]         | CV_VBAK      | AUGRU      | Order Reason                  | AUGRU             |
| [BSTNK]         | CV_VBAK      | BSTNK      | Customer PO Number            | BSTNK             |
| [GSBER]         | CV_VBAK      | GSBER      | Business Area                 | GSBER             |
| [BUKRS]         | CV_VBAK      | BUKRS      | Company Code                  | BUKRS             |
| [NETWR (VBRP)]  | CV_VBRP      | NETWR      | Item Net Value                | NETWR_VBRP        |
| [FKIMG]         | CV_VBRP      | FKIMG      | Billing Quantity              | FKIMG             |
| [BRTWR]         | CV_VBRP      | BRTWR      | Gross Value                   | BRTWR             |
| [MATNR]         | CV_VBRP      | MATNR      | Material Number               | MATNR             |
| [ARKTX]         | CV_VBRP      | ARKTX      | Item Description              | ARKTX             |
| [AUBEL]         | CV_VBRP      | AUBEL      | Reference Sales Order         | AUBEL             |
| [AUPOS]         | CV_VBRP      | AUPOS      | Reference Item                | AUPOS             |
| [VBELN (VBRP)]  | CV_VBRP      | VBELN      | Billing Document Number       | VBELN_VBRP        |
| [WERKS]         | CV_VBRP      | WERKS      | Plant                         | WERKS             |
| [PRCTR]         | CV_VBRP      | PRCTR      | Profit Center                 | PRCTR             |
| [FKART_ANA]     | CV_VBRP      | FKART_ANA  | Billing Type (analytics)      | FKART_ANA         |
| [NAME1 (KNA1)]  | CV_KNA1      | NAME1      | Customer Name                 | NAME1_KNA1        |
| [ORT01]         | CV_KNA1      | ORT01      | City (customer master)        | ORT01             |
| [LAND1 (KNA1)]  | CV_KNA1      | LAND1      | Country (customer master)     | LAND1_KNA1        |
| [KUNNR (KNA1)]  | CV_KNA1      | KUNNR      | Customer Number (master)      | KUNNR_KNA1        |
| [ADRNR]         | CV_VBPA      | ADRNR      | Address Reference Number      | ADRNR             |
| [PARVW]         | CV_VBPA      | PARVW      | Partner Function              | PARVW             |
| [NAME1]         | CV_ADRC      | NAME1      | Address Name                  | NAME1_ADRC        |
| [CITY1]         | CV_ADRC      | CITY1      | Address City                  | CITY1             |
| [POST_CODE1]    | CV_ADRC      | POST_CODE1 | Postal Code                   | POST_CODE1        |
| [COUNTRY]       | CV_ADRC      | COUNTRY    | Address Country               | COUNTRY_ADRC      |
| [KAUFN]         | CV_CE4BMSI   | KAUFN      | COPA Reference Order          | KAUFN             |
| [KDPOS]         | CV_CE4BMSI   | KDPOS      | COPA Reference Item           | KDPOS             |
| [PRCTR (CE4)]   | CV_CE4BMSI   | PRCTR      | COPA Profit Center            | PRCTR_CE4BMSI     |
| [VKORG (CE4)]   | CV_CE4BMSI   | VKORG      | Sales Org (COPA)              | VKORG_CE4BMSI     |
| [WERKS (CE4)]   | CV_CE4BMSI   | WERKS      | Plant (COPA)                  | WERKS_CE4BMSI     |
| [BUKRS (CE4)]   | CV_CE4BMSI   | BUKRS      | Company Code (COPA)           | BUKRS_CE4BMSI     |

#### Renamed Column Register (Tableau qualified name → PBIP column name):

  [NETWR (SAP_ERP_TABLE_CV_S4__CV_VBRP)]    → NETWR_VBRP
  [VBELN (SAP_ERP_TABLE_CV_S4__CV_VBRP)]    → VBELN_VBRP
  [VBELN (SAP_ERP_TABLE_CV_S4__CV_VBPA)]    → VBELN_VBPA
  [KUNNR (SAP_ERP_TABLE_CV_S4__CV_KNA1)]    → KUNNR_KNA1
  [KUNNR (SAP_ERP_TABLE_CV_S4__CV_VBPA)]    → KUNNR_VBPA
  [NAME1 (SAP_ERP_TABLE_CV_S4__CV_KNA1)]    → NAME1_KNA1
  [NAME1] (CV_ADRC)                          → NAME1_ADRC
  [LAND1 (SAP_ERP_TABLE_CV_S4__CV_KNA1)]    → LAND1_KNA1
  [LAND1 (SAP_ERP_TABLE_CV_S4__CV_CE4BMSI)] → LAND1_CE4BMSI
  [PRCTR (SAP_ERP_TABLE_CV_S4__CV_CE4BMSI)] → PRCTR_CE4BMSI
  [VKORG (SAP_ERP_TABLE_CV_S4__CV_CE4BMSI)] → VKORG_CE4BMSI
  [WERKS (SAP_ERP_TABLE_CV_S4__CV_KNA1)]    → WERKS_KNA1
  [WERKS (SAP_ERP_TABLE_CV_S4__CV_CE4BMSI)] → WERKS_CE4BMSI
  [BUKRS (SAP_ERP_TABLE_CV_S4__CV_CE4BMSI)] → BUKRS_CE4BMSI
  [ADRNR (SAP_ERP_TABLE_CV_S4__CV_KNA1)]    → ADRNR_KNA1
  [ERDAT (SAP_ERP_TABLE_CV_S4__CV_KNA1)]    → ERDAT_KNA1
  [ERDAT (SAP_ERP_TABLE_CV_S4__CV_VBRP)]    → ERDAT_VBRP
  [GSBER (SAP_ERP_TABLE_CV_S4__CV_VBRP)]    → GSBER_VBRP
  [GSBER (SAP_ERP_TABLE_CV_S4__CV_CE4BMSI)] → GSBER_CE4BMSI
  [SPART (SAP_ERP_TABLE_CV_S4__CV_VBRP)]    → SPART_VBRP
  [SPART (SAP_ERP_TABLE_CV_S4__CV_CE4BMSI)] → SPART_CE4BMSI
  [MANDT] (VBAK — unqualified)               → MANDT
  [MANDT (SAP_ERP_TABLE_CV_S4__CV_KNA1)]    → MANDT_KNA1
  [MANDT (SAP_ERP_TABLE_CV_S4__CV_VBPA)]    → MANDT_VBPA
  [MANDT (SAP_ERP_TABLE_CV_S4__CV_VBRP)]    → MANDT_VBRP
  [MANDT (SAP_ERP_TABLE_CV_S4__CV_CE4BMSI)] → MANDT_CE4BMSI
  [AUFNR] (VBAK — unqualified)               → AUFNR
  [AUFNR (SAP_ERP_TABLE_CV_S4__CV_VBRP)]    → AUFNR_VBRP
  [AUFNR (SAP_ERP_TABLE_CV_S4__CV_CE4BMSI)] → AUFNR_CE4BMSI
  [ABRVW] (VBAK — unqualified)               → ABRVW
  [ABRVW (SAP_ERP_TABLE_CV_S4__CV_KNA1)]    → ABRVW_KNA1
  [ABRVW (SAP_ERP_TABLE_CV_S4__CV_VBRP)]    → ABRVW_VBRP
  [KOKRS] (VBAK — unqualified)               → KOKRS
  [KOKRS (SAP_ERP_TABLE_CV_S4__CV_KNA1)]    → KOKRS_KNA1
  [KOKRS (SAP_ERP_TABLE_CV_S4__CV_VBRP)]    → KOKRS_VBRP
  [KOKRS (SAP_ERP_TABLE_CV_S4__CV_CE4BMSI)] → KOKRS_CE4BMSI
  [HZUOR] (VBPA — unqualified)               → HZUOR
  [HZUOR (SAP_ERP_TABLE_CV_S4__CV_KNA1)]    → HZUOR_KNA1
  [ZETLTIMSTMP] (VBAK — unqualified)         → ZETLTIMSTMP
  [ZETLTIMSTMP (all others)]                 → ZETLTIMSTMP_{SOURCE}

⚠️  Unqualified field names ([NETWR], [VBELN], [ERDAT], [KUNNR], [NAME1],
    [LAND1], [PRCTR], [VKORG], [WERKS], [AUFNR], [MANDT], [GSBER],
    [SPART], [ABRVW], [KOKRS], [ZETLTIMSTMP]) all resolve to CV_VBAK
    as the primary/driving table, EXCEPT where those fields don't exist
    in VBAK — in which case they resolve to the table where they are
    defined. Preserve the default-table resolution exactly.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — CALENDAR TABLE
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PBIP Table Name     : Calendar Table
Source              : Confirmed from 12_diagramlayout.json (node label)
Expected Columns    : Date, Year, Month, MonthNo, Quarter, etc.
                      (standard BMS Calendar Table pattern)
Expected Relationship : Calendar Table[Date] → Query1[ERDAT]
                        (Many:1, single active relationship)

⚠️  CalendarTable.tmdl not uploaded — exact columns and DAX/M definition
    are unknown. This table follows the standard PBIP Calendar Table
    pattern used across BMS PBIP models.
⚠️  [ERDAT] (Sales Order Created Date from CV_VBAK) is the primary date
    key for the fact table. Additional date fields in Query1 (e.g.,
    AUDAT, AEDAT, FKDAT_ANA from CV_VBRP) may require inactive
    relationships or role-playing date tables.

Conversion Status: ⚠️  PENDING — CalendarTable.tmdl not uploaded.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — CONVERSION COMPLETENESS SCORECARD
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Category                      | Tableau Items | PBI Full | PBI Partial | Not Converted | % Complete |
|-------------------------------|---------------|----------|-------------|---------------|------------|
| Parameters                    | 4             | 0        | 4           | 0             | 50%        |
| Calculated Columns            | 2 (inferred)  | 0        | 2           | 0             | 30%        |
| DAX Measures                  | 4 (inferred)  | 0        | 4           | 0             | 30%        |
| Power Query / Join Structure  | 1             | 0        | 1           | 0             | 70%        |
| Source Column Map             | 500+          | 1        | 0           | 0             | 80%        |
| Calendar Table                | 1             | 0        | 1           | 0             | 40%        |
| Model Metadata                | 1             | 1        | 0           | 0             | 100%       |
| TOTAL                         | ~513          | 2        | 12          | 0             | 🟡 ~55%    |

PENDING — to reach 100%:
  → Query1.tmdl     (verbatim M query, all columns, all measures,
                      all calculated columns, relationships)
  → CalendarTable.tmdl (date dimension definition)
  → Confirmed parameter table TMDL files (date slicers / What-If)
  → Visual/sheet-level filter documentation (Returns vs All Lines scope)
  → Per-measure confirmation of which NETWR column (header vs item) applies


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — CONVERSION PATTERN CHEAT SHEET
## (SAP HANA DirectQuery — PR Returns specific patterns)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PATTERN-01 | SAP HANA DirectQuery connection in PBIP M
─────────────────────────────────────────────────────────────
Tableau: Named connection → saphana class → server:port, schema=_SYS_BIC

Power BI M:
  Source = SapHana.Database(
      "pahpdb01.net.bms.com:30015",
      [Implementation = "2.0"]
  ),
  Schema = Source{[Name = "_SYS_BIC"]}[Data],
  MyView = Schema{[Name = "SAP_ERP.TABLE_CV_S4/CV_VBAK"]}[Data]

⚠️  SAP HANA Calculation View path format uses "/" separator in M:
    M (correct):      SAP_ERP.TABLE_CV_S4/CV_VBAK
    Tableau (source): SAP_ERP.TABLE_CV_S4::CV_VBAK
    Never use "::" in M query paths.


PATTERN-02 | Multi-table join → single DirectQuery table (Query1)
─────────────────────────────────────────────────────────────
Tableau exposes all join columns as a single flat datasource.
PBIP replicates this as a single "Query1" table in DirectQuery mode.
The 5-table join logic is encapsulated in the M query or as a
native SAP HANA query using Value.NativeQuery.

Recommended approach (pushdown to SAP HANA):
  Result = Value.NativeQuery(
      Schema,
      "SELECT ...
       FROM ""SAP_ERP.TABLE_CV_S4"".""CV_VBAK""
       INNER JOIN ""SAP_ERP.TABLE_CV_S4"".""CV_KNA1""
             ON VBAK.KUNNR = KNA1.KUNNR
       INNER JOIN ""SAP_ERP.TABLE_CV_S4"".""CV_VBPA""
             ON VBAK.VBELN = VBPA.VBELN
       INNER JOIN ""SAP_ERP.TABLE_CV_S4"".""CV_ADRC""
             ON VBPA.ADRNR = ADRC.ADDRNUMBER
       INNER JOIN ""SAP_ERP.TABLE_CV_S4"".""CV_VBRP""
             ON VBAK.VBELN = VBRP.AUBEL
       LEFT  JOIN ""SAP_ERP.TABLE_CV_S4"".""CV_CE4BMSI""
             ON VBRP.AUBEL = CE4BMSI.KAUFN
            AND VBRP.AUPOS = CE4BMSI.KDPOS
            AND VBRP.PRCTR = CE4BMSI.PRCTR",
      null,
      [EnableFolding = true]
  )

⚠️  CE4BMSI LEFT JOIN must be preserved. Do NOT promote to INNER JOIN.
⚠️  3-column composite join on CE4BMSI — all three conditions required.
    The PRCTR = PRCTR condition references CV_VBRP.PRCTR = CV_CE4BMSI.PRCTR.


PATTERN-03 | Disambiguating duplicate SAP field names
─────────────────────────────────────────────────────────────
Tableau: [NETWR] = VBAK (primary); [NETWR (SAP_ERP_TABLE_CV_S4__CV_VBRP)]
         = VBRP (qualified).

Power BI: Rename all secondary occurrences with _{SOURCETABLE} suffix.
          Primary table (VBAK) retains the bare field name.
          See Section 5 rename register for the full list.


PATTERN-04 | Tableau date parameter → PBIP date slicer
─────────────────────────────────────────────────────────────
Tableau param-domain-type='range' → PBIP between-dates slicer
Tableau param-domain-type='any'   → PBIP unconstrained date slicer

For date range filtering in DAX measures:
  CALCULATE(
      [MyMeasure],
      Query1[ERDAT] >= MIN('DateParamStart'[Date]),
      Query1[ERDAT] <= MAX('DateParamEnd'[Date])
  )


PATTERN-05 | Two independent date windows
─────────────────────────────────────────────────────────────
This workbook uses TWO separate date parameter pairs:
  Window A: PARAM-01 (Start) + PARAM-02 (End) → "Returns" sheets
  Window B: PARAM-03 (Start) + PARAM-04 (End) → "All Lines" sheets

In PBIP, implement as two independent pairs of page-scoped or
visual-scoped slicers. Use report-page-level slicer sync to contain
each pair to its respective report page. Do NOT share slicers between
the two scopes.


PATTERN-06 | CE4BMSI NULL guard for COPA measures
─────────────────────────────────────────────────────────────
The LEFT JOIN on CV_CE4BMSI means COPA fields (KAUFN, KDPOS, PRCTR, BUKRS,
VKORG_CE4, WERKS_CE4, etc.) will be NULL for billing items with no COPA
record. Add NULL guards in any DAX measure using CE4BMSI columns:

  measure 'My COPA Measure' =
  CALCULATE(
      [MyMeasure],
      NOT ISBLANK(Query1[KAUFN])
  )

Use this pattern consistently for all CE4BMSI-sourced columns.


PATTERN-07 | Tableau comment preservation in DAX
─────────────────────────────────────────────────────────────
For every calculated column that originated from Tableau, add:
  /* Original Tableau Expression is <tableau_formula_here> */
at the end of the DAX expression.
Use /// header comments for column-level display names:
  /// My Column Caption - Column
These comments appear in Tabular Editor and VS Code TMDL view.


PATTERN-08 | SAP HANA ODBC capability flags (audit only)
─────────────────────────────────────────────────────────────
The Tableau XML contains 100+ CAP_ ODBC capability customization flags.
Notable non-default values (for audit trail only — no PBIP action needed):
  CAP_ODBC_METADATA_SUPPRESS_SQLSTATISTICS_API = yes
  CAP_ODBC_UNBIND_EACH = yes
  CAP_ODBC_QUERY_USE_PREPARE_PARAMETER_MARKER = yes
  CAP_PERSIST_SQL_RELATION_ON_LEAF_CONNECTION = yes
These affect Tableau's ODBC driver behavior only. No equivalent setting
exists in the PBIP SapHana.Database() connector.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAP-01 | Query1.tmdl not uploaded — CRITICAL
  Severity : Critical
  Detail   : The main fact table TMDL was not included in the uploaded
             PBIP files. All column definitions, M query, measures,
             calculated columns, and relationships are currently
             inferred from Tableau XML only.
  Action   : Upload Query1.tmdl to replace all inferred items with
             verified, verbatim TMDL definitions.


GAP-02 | CalendarTable.tmdl not uploaded
  Severity : High
  Detail   : The date dimension table TMDL is missing. Relationship
             to Query1[ERDAT] is inferred from diagram layout only.
  Action   : Upload CalendarTable.tmdl to confirm date dimension
             columns, DAX/M definition, and relationship binding.


GAP-03 | 500+ SAP columns — no explicit PBIP column selection confirmed
  Severity : Medium
  Detail   : Tableau exposes all columns from the 5-table join.
             PBIP may load a subset. Without Query1.tmdl, the exact
             column set is unknown.
  Action   : After uploading Query1.tmdl, cross-reference against the
             Section 5 column map and document any columns dropped.


GAP-04 | Two date windows — scope separation not confirmed in PBIP
  Severity : Medium
  Detail   : Tableau uses two independent date parameter pairs scoping
             two different sheet groups ("Returns" and "All Lines").
             PBIP page/visual scope architecture has not been confirmed.
  Action   : Confirm which report pages correspond to each Tableau sheet
             scope. Implement page-scoped slicers with slicer sync.


GAP-05 | CE4BMSI LEFT JOIN — NULL handling in measures not confirmed
  Severity : Medium
  Detail   : The 3-column composite LEFT JOIN on CV_CE4BMSI means COPA
             fields (KAUFN, KDPOS, PRCTR_CE4BMSI, BUKRS_CE4BMSI, etc.)
             will be NULL for billing items with no COPA record.
             Whether the PBIP model handles these NULLs in measures
             is not confirmed without Query1.tmdl.
  Action   : Add ISBLANK guards (see PATTERN-06) to all DAX measures
             that reference CE4BMSI-sourced columns.


GAP-06 | NETWR ambiguity — header vs item level
  Severity : Medium
  Detail   : CV_VBAK.NETWR (header net value) and CV_VBRP.NETWR (item
             net value) are both present in Query1. Tableau sheets may
             use either depending on context. Without Query1.tmdl,
             the exact column used per measure/visual is unknown.
  Action   : After uploading Query1.tmdl, document per measure which
             NETWR is referenced and add to Section 3 measure notes.


GAP-07 | compatibilityLevel comment vs actual discrepancy
  Severity : Info
  Detail   : 12_model.txt comment states "1550 compatibility level".
             12_database.txt declares compatibilityLevel: 1604.
             Effective level is 1604.
  Action   : Preserve both as-is. The comment is a template artefact.
             Do NOT update the comment to match 1604.


GAP-08 | Model name trailing hyphen-space artefact
  Severity : Info
  Detail   : Model name is "PR Returns - Invoice History- model"
             with no space before the final hyphen: "History- model".
             This is likely an accidental authoring artefact.
  Action   : Preserve exactly — do NOT "correct" the spacing.
             Changing the model name breaks all workspace references.


GAP-09 | SAP HANA ODBC capability flags — audit record only
  Severity : Info
  Detail   : The Tableau XML contains an extensive SAP HANA ODBC
             capability customization block (100+ CAP_ flags).
             These affect Tableau's ODBC driver behavior only and
             have no equivalent in the PBIP SapHana.Database() M connector.
  Action   : No PBIP action required. Documented for audit trail only.
             See PATTERN-08 for the list of notable non-default values.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — FILE INVENTORY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| File                             | Content                                          |
|----------------------------------|--------------------------------------------------|
| pr_returns___invoice_history.txt | Tableau workbook XML (full, rev 4.5)             |
| 12_database.txt                  | PBIP database.tmdl (compatibilityLevel: 1604)    |
| 12_model.txt                     | PBIP model.tmdl (culture, DQ version, QueryOrder)|
| 12_definition.json               | PBIP definition.json (schema version 4.2)        |
| 12_diagramlayout.json            | PBIP diagram layout (Query1 + Calendar Table)    |

Pending uploads:
| Query1.tmdl                      | Main fact table: M query, all columns, measures, |
|                                  | calculated columns, and relationships             |
| CalendarTable.tmdl               | Date dimension table definition                  |
| [parameter tables].tmdl          | Date slicer / What-If parameter definitions      |

END OF KNOWLEDGE FILE
# PR Returns – Invoice History
# Tableau → Power BI (PBIP/TMDL)
# Generated: 2026-07-09
# Completeness: ~55% | Critical pending: Query1.tmdl, CalendarTable.tmdl
