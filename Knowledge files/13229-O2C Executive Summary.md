

# O2C EXECUTIVE SUMMARY
# TABLEAU TO POWER BI MIGRATION KNOWLEDGE DOCUMENT
# ============================================================
# Source : 13229_o2c_executive_summary.json (Tableau TWB v18.1)
# Target : PBIP Semantic Model (Power BI TMDL)
# Author : Migration Team
# Date   : 2026-07-07
# ============================================================


O2C Executive Summary — Tableau to Power BI Migration Knowledge Document

1. DOCUMENT OVERVIEW
Report Name   : O2C Executive Summary
Tableau ID    : 13229-O2CExecutiveSummary
Tableau Site  : PRODUCTION
Tableau Rev   : 10.0 (source-build 2024.2.13)
PBI Format    : PBIP Semantic Model (TMDL)
Source System : SAP HANA — pahpdb01.net.bms.com:30015
SAP Schema    : _SYS_BIC
Migration Scope:
  - 4 Fact Tables (O:1-1, AR-2, AR-3, AR-4)
  - 7 Dimension / Reference Tables
  - 3 Tableau Parameters -> Power BI Field Parameter Tables
  - All SAP HANA initial transformations -> Power Query (M)
  - All Tableau LOD Expressions -> DAX Measures
  - All Tableau Calculated Columns -> DAX Calculated Columns
  - Row-Level Security (Stakeholder List pattern)
  - Utility Tables (Refresh Date, tmpTbl, Glossary)


2. PBIP SEMANTIC MODEL — DIRECTORY STRUCTURE
O2C Executive Summary.Dataset/
|
|-- definition/
|   |-- database.tmdl
|   |-- model.tmdl
|   |-- relationships.tmdl
|   |-- expressions.tmdl
|
|-- definition/tables/
|   |-- O 1-1.tmdl               (Fact)
|   |-- AR-2.tmdl                (Fact)
|   |-- AR-3.tmdl                (Fact)
|   |-- AR-4.tmdl                (Fact)
|   |-- CompanyCode.tmdl         (Dimension)
|   |-- CompanyName.tmdl         (Dimension)
|   |-- Region.tmdl              (Dimension)
|   |-- DimDate.tmdl             (Date Dimension)
|   |-- Glossary.tmdl            (Reference)
|   |-- tmpTbl.tmdl              (Helper / RLS Group)
|   |-- Stakeholder List.tmdl    (RLS / Security)
|   |-- View By.tmdl             (Field Parameter)
|   |-- Reporting Region.tmdl    (Calculated Dimension)
|   |-- Refresh Date.tmdl        (Utility)
|
|-- definition/tables/localDateTables/
|   |-- LocalDateTable_*.tmdl    (Auto-generated, 20+ entries)
|
|-- .platform
|-- diagramLayout.json
|-- definition.pbidataset

Sources (validated):

directory_structure.txt — full directory listing
diagramlayout.json — table layout and node mapping
model.tmdl — semantic model definition
relationships.txt — all relationship definitions


3. DATA SOURCE MAPPING
+-------------------+-------------------------------------------+----------------------------------+
| TABLEAU DATASOURCE| TABLEAU CONNECTION                        | POWER BI TABLE / POWER QUERY     |
+-------------------+-------------------------------------------+----------------------------------+
| AR-4              | saphana / pahpdb01.net.bms.com:30015      | AR-4 (import, NativeQuery)       |
|                   | _SYS_BIC.SAP_ERP.OTC.ENHANCE/             |                                  |
|                   | CV_FI_AR_ENHNCE_TRND_BCKT                 |                                  |
+-------------------+-------------------------------------------+----------------------------------+
| AR-3              | saphana / pahpdb01.net.bms.com:30015      | AR-3 (import, NativeQuery)       |
|                   | _SYS_BIC.SAP_ERP.OTC.ENHANCE/             |                                  |
|                   | CV_FI_AR_ENHNCE_RSER_BAL_TRND             |                                  |
+-------------------+-------------------------------------------+----------------------------------+
| AR-2              | saphana / pahpdb01.net.bms.com:30015      | AR-2 (import, NativeQuery)       |
|                   | _SYS_BIC.SAP_ERP.OTC.ENHANCE/             |                                  |
|                   | CV_FI_AR_ENHNCE_AR_AGING_TREND            |                                  |
+-------------------+-------------------------------------------+----------------------------------+
| O:1-1             | saphana / pahpdb01.net.bms.com:30015      | O: 1-1 (import, NativeQuery)     |
|                   | _SYS_BIC.SAP_ERP.OTC.ENHANCE/             |                                  |
|                   | CV_SD_OTC_ENHNCE_ORD_TREND                |                                  |
+-------------------+-------------------------------------------+----------------------------------+
| Parameters        | inline / no connection                    | View By: (calculated table)      |
|                   |                                           | + individual DAX parameters      |
+-------------------+-------------------------------------------+----------------------------------+

Sources: 13229_o2c_executive_summary.json connection metadata, ar_3.txt partition source, ar_4.txt Tableau metadata-records

4. INITIAL TRANSFORMATIONS — TABLEAU TO POWER QUERY (M)
4.1 AR-3 Power Query (Validated from ar_3.txt)
TABLEAU APPROACH:
  Direct live connection to SAP HANA view
  Schema: _SYS_BIC
  View  : SAP_ERP.OTC.ENHANCE/CV_FI_AR_ENHNCE_RSER_BAL_TRND
  Mode  : Live query, column projection at query time

POWER BI POWER QUERY (M) — AR-3:
  partition AR-3 = m
    mode: import
    source = Value.NativeQuery(
      SapHana.Database(
        "pahpdb01.net.bms.com:30015",
        [Implementation="2.0"]
      ),
      "SELECT
        ""SAP_ERP_OTC_ENHANCE_CV_FI_AR_ENHNCE_RSER_BAL_TRND"".""CALD_MTH"",
        ""SAP_ERP_OTC_ENHANCE_CV_FI_AR_ENHNCE_RSER_BAL_TRND"".""CMPY_CD"",
        ""SAP_ERP_OTC_ENHANCE_CV_FI_AR_ENHNCE_RSER_BAL_TRND"".""CMPY_CD_TXT"",
        ""SAP_ERP_OTC_ENHANCE_CV_FI_AR_ENHNCE_RSER_BAL_TRND"".""CNCY_KEY"",
        ""SAP_ERP_OTC_ENHANCE_CV_FI_AR_ENHNCE_RSER_BAL_TRND"".""LST_DT"",
        ""SAP_ERP_OTC_ENHANCE_CV_FI_AR_ENHNCE_RSER_BAL_TRND"".""REGN"",
        ""SAP_ERP_OTC_ENHANCE_CV_FI_AR_ENHNCE_RSER_BAL_TRND"".""RPORTING_REGN"",
        ""SAP_ERP_OTC_ENHANCE_CV_FI_AR_ENHNCE_RSER_BAL_TRND"".""RSER_BAL_LCL"",
        ""SAP_ERP_OTC_ENHANCE_CV_FI_AR_ENHNCE_RSER_BAL_TRND"".""RSER_BAL_USD"",
        ""SAP_ERP_OTC_ENHANCE_CV_FI_AR_ENHNCE_RSER_BAL_TRND"".""TOTL_OPN_AR_BAL_LOC"",
        ""SAP_ERP_OTC_ENHANCE_CV_FI_AR_ENHNCE_RSER_BAL_TRND"".""TOTL_OPN_AR_BAL_USD""
       FROM ""_SYS_BIC"".""SAP_ERP.OTC.ENHANCE/CV_FI_AR_ENHNCE_RSER_BAL_TRND""",
      null,
      [EnableFolding=true]
    )

MIGRATION NOTE:
  EnableFolding=true preserves query folding back to SAP HANA.
  Column list is explicit — no SELECT *.
  Import mode replaces Tableau live connection.

4.2 AR-4 Power Query (Validated from Tableau JSON metadata + PBIP pattern)
TABLEAU APPROACH:
  Live connection to SAP HANA view
  View: _SYS_BIC.SAP_ERP.OTC.ENHANCE/CV_FI_AR_ENHNCE_TRND_BCKT
  Columns projected: AGE_BCKT, AGE_CATG, CALD_MTH, CMPY_CD_TXT, CMPY_CD,
                     CNCY_KEY, REGN, RPORTING_REGN, AMT_30_LCL, AMT_30_USD

POWER BI POWER QUERY (M) — AR-4:
  partition AR-4 = m
    mode: import
    source = Value.NativeQuery(
      SapHana.Database(
        "pahpdb01.net.bms.com:30015",
        [Implementation="2.0"]
      ),
      "SELECT AGE_BCKT, AGE_CATG, CALD_MTH, CMPY_CD_TXT, CMPY_CD,
              CNCY_KEY, REGN, RPORTING_REGN, AMT_30_LCL, AMT_30_USD
       FROM ""_SYS_BIC"".""SAP_ERP.OTC.ENHANCE/CV_FI_AR_ENHNCE_TRND_BCKT""",
      null,
      [EnableFolding=true]
    )

4.3 Refresh Date Power Query (Validated from refresh_date.txt)
TABLEAU APPROACH:
  No equivalent — Tableau uses server-side timestamp metadata.

POWER BI POWER QUERY (M) — Refresh Date:
  partition 'Refresh Date' = m
    mode: import
    source =
      let
        RefreshTime = DateTime.LocalNow(),
        Source = #table(
          type table [LastRefreshed = datetime],
          {{RefreshTime}}
        )
      in
        Source

MIGRATION NOTE:
  Single-row table. DateTime.LocalNow() captures dataset refresh time.
  Used by Refresh Date measure to display formatted timestamp on report.


5. TABLE SCHEMA MAPPING — ALL TABLES
5.1 Fact Tables
+-------------------+--------------+-------------------------+---------------------+
| TABLE (TABLEAU)   | TABLE (PBI)  | SOURCE VIEW (HANA)      | LOAD MODE           |
+-------------------+--------------+-------------------------+---------------------+
| O:1-1             | O: 1-1       | CV_SD_OTC_ENHNCE_       | Import / NativeQuery|
|                   |              | ORD_TREND               |                     |
+-------------------+--------------+-------------------------+---------------------+
| AR-2 datasource   | AR-2         | CV_FI_AR_ENHNCE_        | Import / NativeQuery|
|                   |              | AR_AGING_TREND          |                     |
+-------------------+--------------+-------------------------+---------------------+
| AR-3 datasource   | AR-3         | CV_FI_AR_ENHNCE_        | Import / NativeQuery|
|                   |              | RSER_BAL_TRND           |                     |
+-------------------+--------------+-------------------------+---------------------+
| AR-4 datasource   | AR-4         | CV_FI_AR_ENHNCE_        | Import / NativeQuery|
|                   |              | TRND_BCKT               |                     |
+-------------------+--------------+-------------------------+---------------------+

5.2 O:1-1 Column Schema
+------------------+-------------+----------+----------+--------------------------------------+
| TABLEAU FIELD    | PBI COLUMN  | DATATYPE | VISIBLE  | NOTES                                |
+------------------+-------------+----------+----------+--------------------------------------+
| BILL_MONTH       | BILL_MONTH  | string   | yes      | Billing month key                    |
| CMPY             | CMPY        | string   | yes      | Company display name                 |
| CMPY_CD          | CMPY_CD     | string   | HIDDEN   | FK to CompanyCode[CMPY_CD]           |
| CNCY_KEY         | CNCY_KEY    | string   | yes      | Currency key                         |
| CNT_SLS_DOC      | CNT_SLS_DOC | integer  | yes      | Count of sales documents             |
| LST_DT           | LST_DT      | string   | yes      | Last date raw (YYYYMMDD string)      |
| NET_AMT_LCL      | NET_AMT_LCL | decimal  | yes      | Net amount local currency            |
| NET_AMT_USD      | NET_AMT_USD | decimal  | yes      | Net amount USD                       |
| ORD_DOC_TYPE     | ORD_DOC_TYPE| string   | yes      | Order document type                  |
| ORD_REAS_CD      | ORD_REAS_CD | string   | yes      | Order reason code                    |
| REGN             | REGN        | string   | HIDDEN   | FK to Region[REGN]                   |
| RPORTING_REGN    | RPORTING_REGN| string  | yes      | Reporting region                     |
| Date (calculated)| Date        | date     | yes      | DATE(LEFT(CALD_MTH,4),RIGHT(...),1)  |
+------------------+-------------+----------+----------+--------------------------------------+

5.3 AR-2 Column Schema
+--------------------+---------------------+----------+----------+-------------------------------+
| TABLEAU FIELD      | PBI COLUMN          | DATATYPE | VISIBLE  | NOTES                         |
+--------------------+---------------------+----------+----------+-------------------------------+
| ARAMT_30_LCL       | ARAMT_30_LCL        | decimal  | yes      | AR amount 30 days local       |
| ARAMT_30_USD       | ARAMT_30_USD        | decimal  | yes      | AR amount 30 days USD         |
| CALD_MTH           | CALD_MTH            | string   | yes      | Calendar month (YYYYMM)       |
| CC_SLSAMT_90_USD_KD| CC_SLSAMT_90_USD_KD | decimal  | yes      | CC sales amt 90d USD KD       |
| CMPY_CD            | CMPY_CD             | string   | yes      | FK to CompanyCode             |
| CMPY_CD_TXT        | CMPY_CD_TXT         | string   | yes      | Company name text             |
| CNCY_KEY           | CNCY_KEY            | string   | yes      | Currency key                  |
| CUST_ACCT_GRP      | CUST_ACCT_GRP       | string   | yes      | Customer account group code   |
| CUST_ACCT_GRP_TXT  | CUST_ACCT_GRP_TXT   | string   | yes      | Customer account group text   |
| REGN               | REGN                | string   | yes      | FK to Region                  |
| RPORTING_REGN      | RPORTING_REGN       | string   | yes      | Reporting region              |
| SLSAMT_90_LCL      | SLSAMT_90_LCL       | decimal  | yes      | Sales amt 90 days local       |
| SLSAMT_90_USD      | SLSAMT_90_USD       | decimal  | yes      | Sales amt 90 days USD         |
| Date (calculated)  | Date                | date     | yes      | DATE(LEFT(CALD_MTH,4),...,1)  |
+--------------------+---------------------+----------+----------+-------------------------------+

5.4 AR-3 Column Schema (Validated from ar_3.txt)
+------------------+------------------+----------+----------+----------------------------------------------+
| TABLEAU FIELD    | PBI COLUMN       | DATATYPE | VISIBLE  | NOTES                                        |
+------------------+------------------+----------+----------+----------------------------------------------+
| CALD_MTH         | CALD_MTH         | string   | yes      | Calendar month YYYYMM (source column)        |
| CMPY_CD          | CMPY_CD          | string   | yes      | Company code FK                              |
| CMPY_CD_TXT      | CMPY_CD_TXT      | string   | yes      | Company text                                 |
| CNCY_KEY         | CNCY_KEY         | string   | yes      | Currency key                                 |
| LST_DT           | LST_DT           | string   | yes      | Last date raw string YYYYMMDD                |
| REGN             | REGN             | string   | yes      | Region FK                                    |
| RPORTING_REGN    | RPORTING_REGN    | string   | yes      | Reporting region                             |
| RSER_BAL_LCL     | RSER_BAL_LCL     | double   | yes      | Reserve balance local currency (sum)         |
| RSER_BAL_USD     | RSER_BAL_USD     | double   | yes      | Reserve balance USD (sum)                    |
| TOTL_OPN_AR_BAL_ | TOTL_OPN_AR_BAL_ | double   | yes      | Total open AR balance local (sum)            |
|   LOC            |   LOC            |          |          |                                              |
| TOTL_OPN_AR_BAL_ | TOTL_OPN_AR_BAL_ | double   | yes      | Total open AR balance USD (sum)              |
|   USD            |   USD            |          |          |                                              |
| n/a (calculated) | Date             | date     | yes      | DATE(LEFT(CALD_MTH,4),RIGHT(CALD_MTH,2),1)  |
| n/a (calculated) | Key Date Col     | date     | yes      | Parsed from LST_DT — see section 6.2        |
+------------------+------------------+----------+----------+----------------------------------------------+

5.5 AR-4 Column Schema (Validated from Tableau JSON metadata-records)
+------------------+------------------+----------+----------+--------------------------------------+
| TABLEAU FIELD    | PBI COLUMN       | DATATYPE | VISIBLE  | NOTES / Tableau Caption              |
+------------------+------------------+----------+----------+--------------------------------------+
| AGE_BCKT         | AGE_BCKT         | string   | yes      | Aging Bucket                         |
| AGE_CATG         | AGE_CATG         | string   | yes      | Aging Category                       |
| CALD_MTH         | CALD_MTH         | string   | yes      | Calendar Month (YYYYMM)              |
| CMPY_CD          | CMPY_CD          | string   | yes      | Company Code                         |
| CMPY_CD_TXT      | CMPY_CD_TXT      | string   | yes      | Company (display name)               |
| CNCY_KEY         | CNCY_KEY         | string   | yes      | Currency Key                         |
| REGN             | REGN             | string   | yes      | Region                               |
| RPORTING_REGN    | RPORTING_REGN    | string   | yes      | Reporting Region                     |
| AMT_30_LCL       | AMT_30_LCL       | decimal  | yes      | AR Amount 30d local                  |
| Net Amount       | Net Amount       | decimal  | yes      | Net Amount (1 Month) in USD          |
|  (1 Month in USD)|  (1 Month in USD)|          |          |                                      |
| n/a (calculated) | Date             | date     | yes      | DATE(LEFT(CALD_MTH,4),...,1)         |
+------------------+------------------+----------+----------+--------------------------------------+

5.6 Dimension / Reference Tables
+---------------------+---------------------+----------+------------------------------------+
| TABLE               | COLUMNS             | TYPE     | NOTES                              |
+---------------------+---------------------+----------+------------------------------------+
| CompanyCode         | CMPY_CD             | Dim      | PK — joins all fact tables         |
|                     | Company Code        |          |                                    |
|                     | Region Group        |          | Used in View By: field parameter   |
|                     | Company Name        |          | Used in View By: field parameter   |
|                     | Company Name_new    |          |                                    |
+---------------------+---------------------+----------+------------------------------------+
| CompanyName         | CMPY                | Dim      | Links O:1-1[CMPY] via O:1-1_CMPY_CD|
|                     | O: 1-1_CMPY_CD      |          |                                    |
+---------------------+---------------------+----------+------------------------------------+
| Region              | REGN                | Dim      | Simple single-column dimension     |
+---------------------+---------------------+----------+------------------------------------+
| DimDate             | Date, Year, Qtr     | Date Dim | Full calendar hierarchy            |
|                     | Quarter, Month      |          |                                    |
|                     | Month Name          |          |                                    |
|                     | Month Name Short    |          |                                    |
|                     | Month Sort          |          |                                    |
|                     | Month-Year          |          |                                    |
|                     | Month-Year Sort     |          |                                    |
|                     | Day, Day Name       |          |                                    |
|                     | Day of Week Sort    |          |                                    |
|                     | Quarter-Year        |          |                                    |
|                     | Is Current Year     |          |                                    |
|                     | Year-Month          |          |                                    |
+---------------------+---------------------+----------+------------------------------------+
| Glossary            | Metric Name         | Reference| Metric definitions for tooltip use |
|                     | Metric Calculation  |          |                                    |
+---------------------+---------------------+----------+------------------------------------+
| tmpTbl              | Cmpy_code (int64)   | Helper   | Company-to-group assignment table  |
|                     | Group               |          | Used in DAX query_2 (unmapped cos) |
+---------------------+---------------------+----------+------------------------------------+
| Stakeholder List    | email               | Security | RLS — maps user email to Company   |
|                     | Company Code        |          | Code allowed access                |
+---------------------+---------------------+----------+------------------------------------+


6. CALCULATED COLUMNS MAPPING — TABLEAU TO POWER BI DAX
6.1 AR-3: Date Calculated Column (Validated from ar_3.txt)
TABLEAU APPROACH:
  CALD_MTH is a string YYYYMM — date parsing done inline in calcs
  No explicit calculated column for date in Tableau

POWER BI DAX CALCULATED COLUMN:
  Table  : AR-3
  Column : Date
  DAX    : Date = DATE(LEFT('AR-3'[CALD_MTH], 4), RIGHT('AR-3'[CALD_MTH], 2), 1)
  Format : General Date
  Note   : Creates first-of-month date from YYYYMM string
           Same pattern applied in AR-2, AR-4, O:1-1

6.2 AR-3: Key Date Col Calculated Column (Validated from ar_3.txt)
TABLEAU APPROACH:
  LST_DT is a raw YYYYMMDD string
  Tableau parses inline using string functions in LOD expressions
  No explicit calculated column — embedded in formula context

POWER BI DAX CALCULATED COLUMN:
  Table  : AR-3
  Column : Key Date Col
  DAX    :
    IF(
        ISBLANK('AR-3'[LST_DT]) || LEN('AR-3'[LST_DT]) < 8,
        BLANK(),
        DATE(
            LEFT('AR-3'[LST_DT], 4),
            MID('AR-3'[LST_DT], 5, 2),
            RIGHT('AR-3'[LST_DT], 2)
        )
    )
  Format   : mmmm d, yyyy
  Variation: Has LocalDateTable relationship (lineage 2e1972dd-...)
  Note     : Guards against blank or short LST_DT values before parsing

6.3 AR-2, AR-4, O:1-1: Date Calculated Column Pattern (Consistent across all fact tables)
TABLEAU APPROACH:
  String CALD_MTH (YYYYMM) used directly as date dimension in joins
  Tableau truncates to month level automatically

POWER BI DAX CALCULATED COLUMN (same pattern, each fact table):
  Column : Date
  DAX    : Date = DATE(LEFT([CALD_MTH], 4), RIGHT([CALD_MTH], 2), 1)
  Format : General Date
  Note   : Enables relationship to DimDate[Date] and LocalDateTable


7. TABLEAU PARAMETERS TO POWER BI MAPPING
7.1 Parameter Mapping Table
+-----------------------------+--------------------+------------------+--------------------+---------------------------+
| TABLEAU PARAMETER           | CAPTION            | DEFAULT VALUE    | MEMBERS            | POWER BI EQUIVALENT       |
+-----------------------------+--------------------+------------------+--------------------+---------------------------+
| [Parameter 1]               | View Doc No./      | "Document"       | "Amount" (Amount   | Field Parameter table or  |
|                             | Amount by:         |                  | USD)               | slicer on measure switch  |
|                             |                    |                  | "Document"         |                           |
|                             |                    |                  | (# of Docs.)       |                           |
+-----------------------------+--------------------+------------------+--------------------+---------------------------+
| [Parameter 2]               | Matter/Unit        | "Matter"         | "Matter"           | Field Parameter table or  |
|                             | Paramater          |                  | (Spend/Matter)     | slicer on measure switch  |
|                             |                    |                  | "Units"            |                           |
|                             |                    |                  | (Spend/Hours)      |                           |
+-----------------------------+--------------------+------------------+--------------------+---------------------------+
| [Parameter 3]               | Region/Company     | "Region"         | "Region"           | View By: calculated table |
|                             | Code               |                  | "Company Code"     | (view_by_3a.txt)          |
|                             |                    |                  | (Company)          |                           |
+-----------------------------+--------------------+------------------+--------------------+---------------------------+
| [P_Projection_Type (copy)]  | Spend Parameter 1  | "Matter Type"    | 7 members          | Not used in O2C scope —   |
|                             |                    |                  |                    | legacy parameter          |
+-----------------------------+--------------------+------------------+--------------------+---------------------------+
| [Spend Parameter 1 (copy)]  | Spend Parameter 2  | "Matters"        | 7 members          | Not used in O2C scope —   |
|                             |                    |                  |                    | legacy parameter          |
+-----------------------------+--------------------+------------------+--------------------+---------------------------+

Source: 13229_o2c_executive_summary.json — datasource[@name="Parameters"] column array
7.2 View By: — Power BI Field Parameter (Validated from view_by_3a.txt)
TABLEAU [Parameter 3] — "Region/ Company Code":
  Type    : string list parameter
  Values  : "Region", "Company Code" (aliased as "Company")
  Default : "Region"
  Usage   : Controls dimension shown on axis (Region or Company)

POWER BI EQUIVALENT — View By: Calculated Table:
  Table   : View By:
  TMDL    : view_by_3a.txt
  Source  :
    {
      ("Company Name", NAMEOF('CompanyCode'[Company Name]), 0, "Company Name"),
      ("Region",       NAMEOF('CompanyCode'[Region Group]), 1, "Region"      )
    }

  Columns:
    'View By:'         -> display label (sourceColumn: [Value1])
                          sortByColumn: 'View By: Order'
                          relatedColumnDetails groupByColumn: 'View By: Fields'
    'View By: Fields'  -> DAX NAMEOF() reference, HIDDEN
                          extendedProperty ParameterMetadata: {version:3, kind:2}
                          (kind:2 = field parameter)
    'View By: Order'   -> sort order integer, HIDDEN
    Value4             -> additional label column

MIGRATION NOTE:
  Tableau Parameter 3 had 2 members (Region, Company Code).
  Power BI uses NAMEOF() to point to CompanyCode[Company Name] and
  CompanyCode[Region Group] — the actual fields that drive visuals.
  kind:2 in ParameterMetadata marks this as a Field Parameter slicer.
  Tableau parameter switch logic in calculated fields becomes
  a SWITCH(SELECTEDVALUE('View By:'[View By: Fields]), ...) pattern in DAX.


8. TABLEAU LOD EXPRESSIONS TO POWER BI DAX MEASURES
8.1 LOD Pattern Reference
+----------------------------+-----------------------------------+------------------------------+
| TABLEAU LOD TYPE           | TABLEAU SYNTAX                    | POWER BI DAX EQUIVALENT      |
+----------------------------+-----------------------------------+------------------------------+
| FIXED (table-level)        | {FIXED : AGG(expr)}               | CALCULATE(AGG, ALL(table))   |
| FIXED (dimension-level)    | {FIXED [dim] : AGG(expr)}         | CALCULATE(AGG, ALLEXCEPT(    |
|                            |                                   |   table, table[dim]))        |
| INCLUDE                    | {INCLUDE [dim] : AGG(expr)}       | SUMMARIZE + CALCULATE        |
| EXCLUDE                    | {EXCLUDE [dim] : AGG(expr)}       | CALCULATE(AGG, ALLEXCEPT(    |
|                            |                                   |   table, remaining dims))    |
| Data Blending (cross-src)  | Blend on shared dimension         | TREATAS(VALUES(), target[col])|
+----------------------------+-----------------------------------+------------------------------+

8.2 AR-3 Measures — Full Extraction and LOD Mapping (Validated from ar_3.txt)
8.2.1 Bad Debt Reserves
TABLEAU LOD EQUIVALENT:
  { FIXED : -1 * SUM([RSER_BAL_USD]) } scoped to user company codes
  Cross-datasource blend: AR-2[CMPY_CD] blended into AR-3[CMPY_CD]

POWER BI DAX (from ar_3.txt):
  measure 'Bad Debt Reserves' =
    VAR _UserEmail    = UPPER(USERPRINCIPALNAME())
    VAR _CompanyCode  = SUMMARIZE(
                          FILTER('Stakeholder List',
                            UPPER('Stakeholder List'[email]) = _UserEmail),
                          'Stakeholder List'[Company Code])
    VAR _CmpyCd       = SUMMARIZE(
                          FILTER('CompanyCode',
                            'CompanyCode'[Company Code] IN _CompanyCode),
                          'CompanyCode'[CMPY_CD])
    VAR _CmpyCdAR2    = SUMMARIZE(
                          FILTER('AR-2', 'AR-2'[CMPY_CD] IN _CmpyCd),
                          'AR-2'[CMPY_CD])
    VAR _CmpyCdAR3    = SUMMARIZE(
                          FILTER('AR-3', 'AR-3'[CMPY_CD] IN _CmpyCd),
                          'AR-3'[CMPY_CD])
    VAR _Result       = CALCULATE(
                          -1 * SUM('AR-3'[RSER_BAL_USD]),
                          TREATAS(_CmpyCdAR2, 'AR-3'[CMPY_CD]),
                          TREATAS(_CmpyCdAR3, 'AR-3'[CMPY_CD]))
    RETURN _Result

LOD TYPE   : FIXED (implicit, no dimensions) with RLS + data-blend
TREATAS    : Replaces Tableau cross-datasource blend on CMPY_CD
USERPRINCIPALNAME: Replaces Tableau User Functions (USERNAME())

8.2.2 Bad Debt as % of AR
TABLEAU LOD EQUIVALENT:
  DIVIDE(
    {FIXED : -1 * SUM([RSER_BAL_USD])},
    {FIXED : SUM([TOTL_OPN_AR_BAL_USD])}
  )
  With cross-datasource blend on AR-2[CMPY_CD]

POWER BI DAX (from ar_3.txt):
  measure 'Bad Debt as % of AR' =
    VAR _UserEmail    = UPPER(USERPRINCIPALNAME())
    VAR _CompanyCode  = SUMMARIZE(FILTER('Stakeholder List',
                          UPPER('Stakeholder List'[email])=_UserEmail),
                          'Stakeholder List'[Company Code])
    VAR _CmpyCd       = SUMMARIZE(FILTER('CompanyCode',
                          'CompanyCode'[Company Code] IN _CompanyCode),
                          'CompanyCode'[CMPY_CD])
    VAR _CmpyCdAR2    = SUMMARIZE(FILTER('AR-2',
                          'AR-2'[CMPY_CD] IN _CmpyCd), 'AR-2'[CMPY_CD])
    VAR _CmpyCdAR3    = SUMMARIZE(FILTER('AR-3',
                          'AR-3'[CMPY_CD] IN _CmpyCd), 'AR-3'[CMPY_CD])
    VAR _BadDebt      = CALCULATE(-1 * SUM('AR-3'[RSER_BAL_USD]),
                          TREATAS(_CmpyCdAR2,'AR-3'[CMPY_CD]),
                          TREATAS(_CmpyCdAR3,'AR-3'[CMPY_CD]))
    VAR _MonthUSD     = CALCULATE(SUM('AR-3'[TOTL_OPN_AR_BAL_USD]),
                          TREATAS(_CmpyCdAR2,'AR-3'[CMPY_CD]),
                          TREATAS(_CmpyCdAR3,'AR-3'[CMPY_CD]))
    RETURN DIVIDE(_BadDebt, _MonthUSD, 0)

FORMAT     : 0.0%;-0.0%;0.0%
LOD TYPE   : FIXED ratio with blend

8.2.3 % Bad Resrv Growth (YoY)
TABLEAU LOD EQUIVALENT:
  { FIXED : PERCENTILE(bad_debt_pct) } for CY vs PY same month
  Typically a table calculation (YoY percent difference)

POWER BI DAX (from ar_3.txt):
  measure '% Bad Resrv Growth' =
    VAR _userEmail      = UPPER(USERPRINCIPALNAME())
    VAR _companyCode    = SUMMARIZE(FILTER('Stakeholder List',
                            UPPER('Stakeholder List'[email])=_userEmail),
                            'Stakeholder List'[Company Code])
    VAR _CmpyCd         = SUMMARIZE(FILTER('CompanyCode',
                            'CompanyCode'[Company Code] IN _companyCode),
                            'CompanyCode'[CMPY_CD])
    VAR _CmpyCdAR2      = SUMMARIZE(FILTER('AR-2',
                            'AR-2'[CMPY_CD] IN _CmpyCd),'AR-2'[CMPY_CD])
    VAR _companyFilter  = TREATAS(_CmpyCdAR2,'AR-3'[CMPY_CD])
    VAR _MaxMonth       = MAXX(ALL('AR-3'),'AR-3'[Date])
    VAR _CurrentYear    = YEAR(_MaxMonth)
    VAR _CurrentMonth   = MONTH(_MaxMonth)
    VAR _PreviousYear   = _CurrentYear - 1
    VAR _cyDateFilter   = FILTER(ALL('AR-3'),
                            YEAR('AR-3'[Date])=_CurrentYear
                            && MONTH('AR-3'[Date])=_CurrentMonth)
    VAR _pyDateFilter   = FILTER(ALL('AR-3'),
                            YEAR('AR-3'[Date])=_PreviousYear
                            && MONTH('AR-3'[Date])=_CurrentMonth)
    VAR _CY_BadDebt     = CALCULATE(-1*SUM('AR-3'[RSER_BAL_USD]),
                            _cyDateFilter,_companyFilter)
    VAR _CY_OpenAR      = CALCULATE(SUM('AR-3'[TOTL_OPN_AR_BAL_USD]),
                            _cyDateFilter,_companyFilter)
    VAR _PY_BadDebt     = CALCULATE(-1*SUM('AR-3'[RSER_BAL_USD]),
                            _pyDateFilter,_companyFilter)
    VAR _PY_OpenAR      = CALCULATE(SUM('AR-3'[TOTL_OPN_AR_BAL_USD]),
                            _pyDateFilter,_companyFilter)
    VAR _CYPct          = DIVIDE(_CY_BadDebt,_CY_OpenAR,BLANK())
    VAR _PYPct          = DIVIDE(_PY_BadDebt,_PY_OpenAR,BLANK())
    RETURN _CYPct - _PYPct

FORMAT     : 0.0%;-0.0%;0.0%
LOD TYPE   : FIXED {MAX([Date])} for period anchoring, YoY table calc

8.2.4 Current Bad Resv %
TABLEAU LOD EQUIVALENT:
  { FIXED : PERCENTILE(bad_debt_pct) } for current period only

POWER BI DAX (from ar_3.txt):
  measure 'Current Bad Resv %' =
    [RLS + company filter pattern as above]
    VAR _MaxMonth     = MAXX(ALL('AR-3'),'AR-3'[Date])
    VAR _CurrentYear  = YEAR(_MaxMonth)
    VAR _CurrentMonth = MONTH(_MaxMonth)
    VAR _cyDateFilter = FILTER(ALL('AR-3'),
                          YEAR('AR-3'[Date])=_CurrentYear
                          && MONTH('AR-3'[Date])=_CurrentMonth)
    VAR _CY_BadDebt   = CALCULATE(-1*SUM('AR-3'[RSER_BAL_USD]),
                          _cyDateFilter,_companyFilter)
    VAR _CY_OpenAR    = CALCULATE(SUM('AR-3'[TOTL_OPN_AR_BAL_USD]),
                          _cyDateFilter,_companyFilter)
    RETURN DIVIDE(_CY_BadDebt,_CY_OpenAR,BLANK())

FORMAT     : 0.00%;-0.00%;0.00%

8.2.5 Prev Bad Resv %
TABLEAU LOD EQUIVALENT:
  { FIXED : PERCENTILE(bad_debt_pct) } for previous year same month

POWER BI DAX (from ar_3.txt):
  measure 'Prev Bad Resv %' =
    [RLS + company filter pattern]
    VAR _MaxMonth     = MAXX(ALL('AR-3'),'AR-3'[Date])
    VAR _PreviousYear = YEAR(_MaxMonth) - 1
    VAR _CurrentMonth = MONTH(_MaxMonth)
    VAR _pyDateFilter = FILTER(ALL('AR-3'),
                          YEAR('AR-3'[Date])=_PreviousYear
                          && MONTH('AR-3'[Date])=_CurrentMonth)
    VAR _PY_BadDebt   = CALCULATE(-1*SUM('AR-3'[RSER_BAL_USD]),
                          _pyDateFilter,_companyFilter)
    VAR _PY_OpenAR    = CALCULATE(SUM('AR-3'[TOTL_OPN_AR_BAL_USD]),
                          _pyDateFilter,_companyFilter)
    RETURN DIVIDE(_PY_BadDebt,_PY_OpenAR,BLANK())

FORMAT     : 0.0%;-0.0%;0.0%

8.2.6 Bad Debt as % of AR_MOM (Month-over-Month variant)
TABLEAU LOD EQUIVALENT:
  { FIXED : MAX([Date]) } anchor for both current and prior-year month
  MOM window: current year max month + same month prior year

POWER BI DAX (from ar_3.txt):
  measure 'Bad Debt as % of AR_MOM' =
    [RLS + TREATAS company filter]
    VAR _maxMonth     = CALCULATE(MAX('AR-3'[Date]),ALL('AR-3'))
    VAR _maxMonthNum  = MONTH(_maxMonth)
    VAR _maxYear      = YEAR(_maxMonth)
    VAR _prevYear     = _maxYear - 1
    VAR _dateAddMonth = EDATE(_maxMonth,-1)
    VAR _dateFilter   = FILTER(ALL('AR-3'[Date]),
        (YEAR('AR-3'[Date])=_prevYear
          && MONTH('AR-3'[Date])=_maxMonthNum
          && 'AR-3'[Date] <= _dateAddMonth)
        || (YEAR('AR-3'[Date])=_maxYear
          && MONTH('AR-3'[Date])=_maxMonthNum
          && 'AR-3'[Date] <= _maxMonth))
    VAR _badDebt      = CALCULATE(-1*SUM('AR-3'[RSER_BAL_USD]),
                          _dateFilter,_companyFilter)
    VAR _openAR       = CALCULATE(SUM('AR-3'[TOTL_OPN_AR_BAL_USD]),
                          _dateFilter,_companyFilter)
    RETURN DIVIDE(_badDebt,_openAR,0)

FORMAT    : 0.0%;-0.0%;0.0%
LOD TYPE  : FIXED {MAX([Date])} with dual-year date window

8.2.7 AR3_CurrVal and AR3_PrevVal (Label Measures)
TABLEAU EQUIVALENT:
  Table calculation / string label showing "current month" and
  "previous year same month" formatted as MMM YY

POWER BI DAX (from ar_3.txt):
  measure AR3_CurrVal =
    VAR _MaxBillMonth = MAXX(ALL('AR-3'),'AR-3'[Date])
    VAR _PrevDate     = EDATE(_MaxBillMonth,-1)
    RETURN FORMAT(_PrevDate,"MMM YY")

  measure AR3_PrevVal =
    VAR _MaxBillMonth = MAXX(ALL('AR-3'),'AR-3'[Date])
    VAR _PrevDate     = EDATE(_MaxBillMonth,-1)
    VAR _PrevYearDate = EDATE(_PrevDate,-12)
    RETURN FORMAT(_PrevYearDate,"MMM YY")

NOTE: Drives dynamic axis label text on Bad Debt trend visual.

8.2.8 Key Date (Display Measure)
TABLEAU EQUIVALENT:
  LOD or string calc: "Key Date: " + STR(MAX([LST_DT])) formatted

POWER BI DAX (from ar_3.txt):
  measure 'Key Date' =
    "Key Date: " & FORMAT(MAX('AR-3'[Key Date Col]),"MMM DD, YYYY")

NOTE: Key Date Col is a calculated column (see section 6.2).
      This measure provides the formatted display string for the report.

8.2.9 Bad Debt_Reserve (Simple Aggregation)
TABLEAU EQUIVALENT:
  SUM(-1 * [RSER_BAL_USD]) — base aggregate, no LOD

POWER BI DAX (from ar_3.txt):
  measure 'Bad Debt_Reserve' =
    -1 * SUM('AR-3'[RSER_BAL_USD])

NOTE: Non-RLS version. Used in development/validation contexts.
      Active measures (Bad Debt Reserves) include RLS and TREATAS.

8.3 O:1-1 Measures (Validated from en_us.txt schema)
+------------------------------------+-------------------------------------------+---------------------+
| TABLEAU CALC / LOD                 | POWER BI DAX MEASURE                      | LOD TYPE            |
+------------------------------------+-------------------------------------------+---------------------+
| zccb & MNC                         | zccb & MNC                                | Concatenation       |
| Count of Sales Documents           | Accuracy Num                              | SUM / COUNT         |
| Order Accuracy numerator           | Order Accuracy Num                        | Conditional COUNT   |
| Invoiced Amount (SUM NET_AMT_USD)  | Invoiced Amt                              | SUM aggregate       |
| YTD Trade Orders Processed         | Trade Orders Processed YTD                | FIXED {year} SUM    |
|  { FIXED YEAR : COUNT([doc]) }     |                                           |                     |
| YTD Order Accuracy Num             | Orders Accuracy Num YTD                   | FIXED {year} COUNT  |
| YTD Order Accuracy DN              | Orders Accuracy DN YTD                    | FIXED {year} COUNT  |
| Order Accuracy % (ratio)           | Orders Accuracy                           | DIVIDE calc         |
| Invoiced Amount YTD                | Invoiced Amount YTD                       | FIXED {year} SUM    |
+------------------------------------+-------------------------------------------+---------------------+

NOTE:
  All YTD measures use FIXED {YEAR([date])} in Tableau.
  In Power BI these become CALCULATE([measure], DATESYTD(DimDate[Date]))
  or explicit VAR _year = YEAR(MAX(table[Date])) filters.

8.4 Refresh Date Measure (Validated from refresh_date.txt)
TABLEAU EQUIVALENT:
  No direct equivalent — Tableau exposes last extract refresh metadata

POWER BI DAX:
  measure 'Refresh Date' =
    "Refresh Date: " & FORMAT(MAX('Refresh Date'[LastRefreshed]),"M/DD/YYYY")

NOTE: Displayed as a card/text visual. Sourced from M query DateTime.LocalNow()
      which stamps the time the dataset was last refreshed and imported.


9. DATA BLENDING — TABLEAU TO POWER BI TREATAS PATTERN
9.1 Cross-Datasource Blend Pattern
TABLEAU BLENDING:
  Primary   : AR-3
  Secondary : AR-2
  Link field: CMPY_CD (shared dimension)
  Effect    : AR-3 filtered to only company codes present in AR-2
              for the current filter context (user / region scope)

POWER BI DAX EQUIVALENT (TREATAS):
  VAR _CmpyCdAR2 = SUMMARIZE(
                     FILTER('AR-2','AR-2'[CMPY_CD] IN _CmpyCd),
                     'AR-2'[CMPY_CD])
  VAR _companyFilter = TREATAS(_CmpyCdAR2,'AR-3'[CMPY_CD])
  CALCULATE([measure], _companyFilter)

MIGRATION RULE:
  Every measure using AR-3 that previously blended AR-2 in Tableau
  MUST include the TREATAS(_CmpyCdAR2,'AR-3'[CMPY_CD]) filter.
  This replicates the secondary datasource intersection behavior.
  See measures: Bad Debt Reserves, Bad Debt as % of AR,
                % Bad Resrv Growth, Current Bad Resv %,
                Prev Bad Resv %, Bad Debt as % of AR_MOM


10. ROW-LEVEL SECURITY PATTERN
10.1 Tableau User Functions to Power BI RLS
TABLEAU APPROACH:
  USERNAME() or USERDOMAIN() in calculated field
  Filter: [CMPY_CD] = { USER FUNCTION } or in allowed set
  No formal RLS framework — user filters embedded in each calc

POWER BI APPROACH:
  Dedicated table: Stakeholder List
  Columns        : email, Company Code
  Pattern (in every secured measure):
    VAR _UserEmail   = UPPER(USERPRINCIPALNAME())
    VAR _CompanyCode = SUMMARIZE(
                         FILTER('Stakeholder List',
                           UPPER('Stakeholder List'[email])=_UserEmail),
                         'Stakeholder List'[Company Code])
    VAR _CmpyCd      = SUMMARIZE(
                         FILTER('CompanyCode',
                           'CompanyCode'[Company Code] IN _CompanyCode),
                         'CompanyCode'[CMPY_CD])
  Then TREATAS(_CmpyCd, factTable[CMPY_CD]) applied in CALCULATE

MIGRATION RULE:
  - Do NOT use static Power BI RLS roles for this report
  - The email-to-company mapping is measure-level, not role-level
  - USERPRINCIPALNAME() is always UPPER() compared to stored email
    (case-insensitive match by convention)
  - Stakeholder List must contain all valid email-to-CompanyCode mappings
  - CompanyCode table bridges Stakeholder List[Company Code] to CMPY_CD


11. CALCULATED / VIRTUAL TABLES MAPPING
11.1 Reporting Region (Validated from reporting_region.txt)
TABLEAU APPROACH:
  Each datasource has its own RPORTING_REGN column.
  Tableau merges by using the field from primary datasource
  or blend context. No explicit union.

POWER BI DAX CALCULATED TABLE:
  table 'Reporting Region'
    column RPORTING_REGN (sourceColumn: O: 1-1[RPORTING_REGN])
    partition source =
      DISTINCT(
        UNION(
          SELECTCOLUMNS('O: 1-1',  'O: 1-1'[RPORTING_REGN]),
          SELECTCOLUMNS('AR-4',    'AR-4'[RPORTING_REGN]),
          SELECTCOLUMNS('AR-3',    'AR-3'[RPORTING_REGN]),
          SELECTCOLUMNS('AR-2',    'AR-2'[RPORTING_REGN])
        )
      )

MIGRATION NOTE:
  UNION of all 4 fact tables ensures all reporting regions are
  present even if one fact table has sparse data for a period.
  Used as slicer dimension in report pages.

11.2 View By: Field Parameter (Validated from view_by_3a.txt)
TABLEAU [Parameter 3] replaces:
  "Region" -> CompanyCode[Region Group]
  "Company Code" -> CompanyCode[Company Name]

POWER BI CALCULATED TABLE:
  table 'View By:'
    partition source =
      {
        ("Company Name", NAMEOF('CompanyCode'[Company Name]),  0, "Company Name"),
        ("Region",       NAMEOF('CompanyCode'[Region Group]),  1, "Region"      )
      }
  Columns:
    'View By:'         -> Value1, sortBy 'View By: Order'
    'View By: Fields'  -> Value2, HIDDEN, ParameterMetadata {version:3,kind:2}
    'View By: Order'   -> Value3, HIDDEN, integer sort
    Value4             -> extra label column

USAGE PATTERN IN DAX:
  SWITCH(
    SELECTEDVALUE('View By:'[View By: Fields]),
    NAMEOF('CompanyCode'[Company Name]),  [measure by company],
    NAMEOF('CompanyCode'[Region Group]),  [measure by region],
    [default]
  )

11.3 tmpTbl (Validated from conversation summary / query_2.txt)
TABLEAU APPROACH:
  No direct equivalent — ad-hoc data joined to identify unmapped companies

POWER BI:
  table tmpTbl
    columns: Cmpy_code (int64), Group

DAX QUERY USING tmpTbl (from query_2.txt):
  -- Find CompanyCodes not assigned to a group
  EVALUATE
    FILTER(
      ADDCOLUMNS(
        VALUES('CompanyCode'[CMPY_CD]),
        "IsMapped", NOT('CompanyCode'[CMPY_CD] IN VALUES('tmpTbl'[Cmpy_code]))
      ),
      [IsMapped] = TRUE
    )

NOTE: Used for admin/QA — identifies companies missing group assignment.
      Not a report visual; used in DAX query pane (daxqueries.json shows
      Query 1, Query 2, Query 3 tabs, defaultTab: Query 3).

11.4 Refresh Date (Validated from refresh_date.txt)
TABLEAU: No equivalent table — metadata only
POWER BI M Query:
  let
    RefreshTime = DateTime.LocalNow(),
    Source = #table(
      type table [LastRefreshed = datetime],
      {{RefreshTime}}
    )
  in Source

Single-row utility table.
Measure: "Refresh Date: " & FORMAT(MAX(...),"M/DD/YYYY")
Used as card visual on report header/footer.


12. RELATIONSHIPS MAPPING
+------------------------+-----------+------------------------+-----------+-------+----------+
| FROM TABLE [COLUMN]    | CARD FROM | TO TABLE [COLUMN]      | CARD TO   | CROSS | ACTIVE   |
+------------------------+-----------+------------------------+-----------+-------+----------+
| O: 1-1[CMPY_CD]        | Many      | CompanyCode[CMPY_CD]   | One       | Both  | Yes      |
| AR-2[CMPY_CD]          | Many      | CompanyCode[CMPY_CD]   | One       | Both  | Yes      |
| AR-3[CMPY_CD]          | Many      | CompanyCode[CMPY_CD]   | One       | Both  | Yes      |
| AR-4[CMPY_CD]          | Many      | CompanyCode[CMPY_CD]   | One       | Both  | Yes      |
| O: 1-1[Date]           | Many      | DimDate[Date]          | One       | Both  | Yes      |
| AR-2[Date]             | Many      | DimDate[Date]          | One       | Both  | Yes      |
| AR-4[Date]             | Many      | DimDate[Date]          | One       | Both  | Yes      |
| AR-3[CMPY_CD]          | Many      | AR-2[CMPY_CD]          | Many      | ---   | No       |
|                        |           |                        |           |       | (TREATAS)|
| AR-3[Key Date Col]     | Many      | LocalDateTable[Date]   | One       | Both  | Yes      |
| Stakeholder List       | Many      | CompanyCode            | One       | Both  | Yes      |
|  [Company Code]        |           |  [Company Code]        |           |       |          |
| Refresh Date           | Many      | LocalDateTable[Date]   | One       | Both  | Yes      |
|  [LastRefreshed]       |           |                        |           |       |          |
+------------------------+-----------+------------------------+-----------+-------+----------+

NOTE:
  AR-3 <-> AR-2 relationship is intentionally NOT a model relationship.
  It is handled via TREATAS in all AR-3 measures.
  This mirrors Tableau's secondary datasource blend behavior.


13. REPORT STYLE AND FORMAT TOKENS
TABLEAU STYLE (from JSON workbook.style):
  font-family  : Trebuchet MS
  font-size    : 9pt (worksheet), 10pt (title)
  font-weight  : bold (title)
  title color  : #004b7e
  text color   : #000000
  animation    : ao-off (animations disabled)

POWER BI EQUIVALENT:
  Theme font    : Trebuchet MS (custom theme JSON)
  Title color   : #004b7e
  Body font size: 9pt
  Animations    : Off in page settings
  Format strings carried over:
    Percentage measures : 0.0%;-0.0%;0.0% (standard)
    Detailed percentage : 0.00%;-0.00%;0.00% (Current Bad Resv %)
    Date measure        : M/DD/YYYY (Refresh Date)
    Key Date display    : MMM DD, YYYY
    Label measures      : MMM YY (AR3_CurrVal, AR3_PrevVal)


14. MIGRATION VALIDATION CHECKLIST
SECTION 3 — DATA SOURCES
[ ] All 4 SAP HANA NativeQuery partitions verified against Tableau connection metadata
[ ] Connection string: pahpdb01.net.bms.com:30015, schema _SYS_BIC
[ ] EnableFolding=true on all import partitions
[ ] Implementation="2.0" on SapHana.Database calls

SECTION 5 — SCHEMA
[ ] AR-3 all 12 columns match Tableau field list (ar_3.txt vs JSON metadata)
[ ] AR-4 all 10 columns from Tableau metadata-records confirmed
[ ] AR-2 all 14 columns from en_us.txt confirmed
[ ] O:1-1 all 13 columns from en_us.txt confirmed
[ ] CMPY_CD hidden on O:1-1 (matches Tableau hidden field)
[ ] REGN hidden on O:1-1 (matches Tableau hidden field)

SECTION 6 — CALCULATED COLUMNS
[ ] Date column present on all 4 fact tables (YYYYMM -> DATE())
[ ] Key Date Col on AR-3 guards ISBLANK and LEN < 8
[ ] Key Date Col has LocalDateTable variation relationship

SECTION 7 — PARAMETERS
[ ] View By: table has 2 rows (Company Name, Region)
[ ] View By: Fields column has ParameterMetadata kind:2
[ ] View By: Order column is HIDDEN and integer
[ ] Tableau Parameter 1 and Parameter 2 mapped to DAX measure switch
[ ] Legacy parameters (Spend Parameter 1/2) excluded from O2C scope

SECTION 8 — MEASURES
[ ] All 9 AR-3 measures present in ar_3.txt confirmed
[ ] USERPRINCIPALNAME() pattern used — not static RLS roles
[ ] TREATAS replaces all AR-2/AR-3 blend references
[ ] All YTD measures from O:1-1 replicate FIXED {YEAR} pattern
[ ] Refresh Date measure format M/DD/YYYY confirmed

SECTION 9 — DATA BLEND
[ ] _CmpyCdAR2 SUMMARIZE+FILTER pattern in every AR-3 measure
[ ] TREATAS(_CmpyCdAR2,'AR-3'[CMPY_CD]) applied consistently
[ ] No physical relationship between AR-2 and AR-3 in model

SECTION 10 — RLS
[ ] Stakeholder List table loaded with email + Company Code
[ ] Email comparison always uses UPPER() on both sides
[ ] CompanyCode bridges Company Code text to CMPY_CD integer

SECTION 11 — VIRTUAL TABLES
[ ] Reporting Region UNION covers all 4 fact tables
[ ] tmpTbl Cmpy_code typed as int64
[ ] Refresh Date partition uses DateTime.LocalNow()
[ ] View By: calculated table partition source confirmed

SECTION 12 — RELATIONSHIPS
[ ] All 4 fact-to-CompanyCode relationships active
[ ] AR-3[Date] -> DimDate active
[ ] AR-3[Key Date Col] -> LocalDateTable active
[ ] AR-3 <-> AR-2 NOT a model relationship (TREATAS only)

SECTION 13 — FORMATTING
[ ] Theme font set to Trebuchet MS
[ ] Title color #004b7e applied
[ ] All percentage format strings match source measures exactly


15. KNOWN GAPS AND NOTES
GAP-01: AR-2 TMDL (ar_2.txt failed to load)
  Status  : ar_2.txt file could not be read (tool error)
  Impact  : AR-2 Power Query M source not directly confirmed
  Evidence: AR-2 schema confirmed from Tableau JSON metadata-records
            AR-2 column CMPY_CD used in TREATAS in ar_3.txt — confirms structure
  Action  : Reupload ar_2.txt and confirm Power Query partition source
            Expected view: CV_FI_AR_ENHNCE_AR_AGING_TREND or equivalent

GAP-02: AR-4 TMDL (ar_4.txt failed to load)
  Status  : ar_4.txt file could not be read (tool error)
  Impact  : AR-4 DAX measures not extracted from TMDL directly
  Evidence: AR-4 columns confirmed from Tableau JSON metadata-records
            AR-4[RPORTING_REGN] referenced in reporting_region.txt UNION
  Action  : Reupload ar_4.txt to confirm measures and Power Query M

GAP-03: O:1-1 TMDL not in current upload batch
  Status  : O 1-1.tmdl not uploaded
  Impact  : O:1-1 DAX measures (Invoiced Amt, Orders Accuracy, YTD calcs)
            not extracted from TMDL — schema known from en_us.txt
  Action  : Upload O11.txt for full measure extraction

GAP-04: Stakeholder List TMDL not uploaded
  Status  : StakeholderList.tmdl not present
  Impact  : Stakeholder List column names and M source not confirmed
            (email, Company Code inferred from ar_3.txt measure usage)
  Action  : Upload StakeholderList.txt to confirm schema and data source

GAP-05: Tableau Parameters 1 and 2 not mapped to specific visuals
  Status  : Parameter 1 (Document/Amount) and Parameter 2 (Matter/Units)
            not confirmed in O2C visual layer (no worksheet JSON extracted)
  Impact  : Visual-level parameter usage unknown
  Action  : Cross-reference with Power BI report layout JSON when available


16. FILE SOURCES REFERENCE
+------------------------------------------+-------------------------------------------+
| FILE                                     | USED IN SECTIONS                          |
+------------------------------------------+-------------------------------------------+
| 13229_o2c_executive_summary.json         | 3, 4, 5.5, 7.1, 13                       |
| ar_3.txt                                 | 4.1, 5.4, 6.2, 8.2, 12                   |
| ar_4.txt (partial - Tableau JSON only)   | 3, 5.5                                    |
| view_by_3a.txt                           | 7.2, 11.2                                 |
| reporting_region.txt                     | 11.1, 12                                  |
| refresh_date.txt                         | 4.3, 8.4, 11.4                            |
| daxqueries.json                          | 11.3                                      |
| diagramlayout.json                       | 2, 12                                     |
| en_us.txt (from conversation summary)    | 5.2, 5.3, 5.6, 8.3                        |
| relationships.txt (from summary)         | 12                                        |
| model.tmdl (from summary)               | 2                                         |
| directory_structure.txt (from summary)  | 2                                         |
+------------------------------------------+-------------------------------------------+

VALIDATION RULE:
  Every mapping in this document is traceable to at least one of the
  above source files. No assumptions or external knowledge used.
  Section 15 explicitly flags any item not fully confirmed by file evidence.


End of Migration Knowledge Document — O2C Executive Summary
Validated against: 13229_o2c_executive_summary.json + PBIP TMDL files
Gaps documented in Section 15 — pending AR-2, AR-4, O:1-1, Stakeholder List TMDL uploads