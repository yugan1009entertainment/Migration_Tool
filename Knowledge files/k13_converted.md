# 📘 CONVERSION KNOWLEDGE FILE
# K13 – CL-1: Customer Credit Management Updates
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-23
# Document Version: 1.0
# Source files: cl_1__customer_credit_management_updates.txt (Tableau TWB)
#               + 19 Power BI PBIP output files
# All content derived exclusively from uploaded source files

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PARAM-01 | View by (Parameter 1)
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 1]
Tableau Caption       : View by
Tableau Type          : string list
Tableau Default       : "Company"
Tableau Members       : "Company", "Customer", "User"

Power BI Table Name   : Parameter
Power BI lineageTag   : 96b9f96c-c7ba-43ba-848f-ba0a4234aa2f
Power BI Type         : Calculated DATATABLE (disconnected slicer)
Power BI Columns      :
  [Parameter]         — display label (string, visible)
                        sortByColumn: 'Parameter Order'
  [Parameter Fields]  — view key value (string)
  [Parameter Order]   — sort integer

Power BI DAX:
─────────────
DATATABLE(
    "Parameter", STRING,
    "Parameter Fields", STRING,
    "Parameter Order", INTEGER,
    {
        {"View By Company",  "Company",  1},
        {"View By Customer", "Customer", 2},
        {"View By User",     "User",     3}
    }
)

Usage in model:
  - SELECTEDVALUE('Parameter'[Parameter Fields]) drives SWITCH in
    CC-10 [View] calculated column, dynamically switching the display
    dimension between Company, Customer, and User.
  - Tableau CASE [Parameter 1] WHEN "Customer" THEN... →
    DAX SWITCH(SELECTEDVALUE('Parameter'[Parameter Fields]), ...)

Conversion Status: ✅ FULL


### PARAM-02 | Currency (Parameter 2)
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 2]
Tableau Caption       : Currency
Tableau Type          : string list
Tableau Default       : "USD"
Tableau Members       : "USD", "Local"

Power BI Table Name   : Currency
Power BI lineageTag   : fe7d88dc-1d44-4377-8641-550e532cc5e8
Power BI Type         : Calculated DATATABLE (disconnected slicer)

Power BI DAX:
─────────────
DATATABLE(
    "Currency", STRING,
    {
        {"USD"},
        {"Local"}
    }
)

Usage in model:
  - SELECTEDVALUE('Currency'[Currency]) used in ALL currency-switched
    measures: Value New, Value Old, Value New_tp, Value Old_tp,
    Value New 1, Displayed Currency, Amount Change_slicer,
    Value New Text 1, Value Old Text 1.
  - Tableau CASE [Parameter 2] WHEN "USD" THEN... →
    DAX SWITCH(SELECTEDVALUE('Currency'[Currency]), ...)

Conversion Status: ✅ FULL


### PARAM-03 | PastMonthParameter (Time Bucket Slicer)
─────────────────────────────────────────────────────────────
Power BI Table Name   : PastMonthParameter
Power BI lineageTag   : b66bd261-daf7-4647-a8e8-a06511a2b101
Power BI Type         : Calculated DATATABLE
Power BI Sort         : Past Month Indicator sorted by Sort_Order

Power BI DAX:
─────────────
DATATABLE(
    "Past Month Indicator", STRING,
    "Sort_Order", INTEGER,
    {
        {"Past 6 Months",       1},
        {"Past 7 - 12 Months",  2},
        {"Past 13 - 18 Months", 3},
        {"Past 19 - 24 Months", 4}
    }
)

Usage in model:
  - Slicer for time period filtering aligned with CC-09 [Time Period]
    classification column.

Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: CVQ_SC_SD_GOTC_CRLMT_UPDT (Fact Table)

### CC-01 | Customer 1
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1097189499375738887
PBI DataType    : string

Tableau Expression:
───────────────────
[CUST]+' '+ IFNULL([CUST_NM1],'')

Power BI DAX:
─────────────
[Customer] & " " & COALESCE([Customer Name1], "")

Pattern     : Tableau IFNULL → DAX COALESCE
Conversion  : ✅ FULL


### CC-02 | Company
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1097189499377860623

Tableau Expression: [CMPY]+' '+[CMPY_TXT]
Power BI DAX: [Company Code] & " " & [Company Code Description]
Conversion  : ✅ FULL


### CC-03 | Changed By
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1097189499377184781

Tableau Expression: [CHG_BY_ID]+' - '+ IFNULL(([CHG_BY_NM]), ' No NAME')
Power BI DAX: [ChangedBy] & " - " & COALESCE([ChangedByName], " No NAME")
Pattern     : Tableau IFNULL with sentinel value → DAX COALESCE
Conversion  : ✅ FULL


### CC-04 | Field 1
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1097189499376123913
Tableau Expression: [FLD]+' ' + [FLD_DESC]
Power BI DAX: [Field] & " " & [Field Description]
Conversion  : ✅ FULL


### CC-05 | Credit Segment 1
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_50102552973942784
Tableau Expression: [CR_SEG]+'-'+[CR_SEG_TXT]
Power BI DAX: [Credit Segment] & "-" & [Credit Segment Description]
Conversion  : ✅ FULL


### CC-06 | Table Name 1
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_50102552974745601
Tableau Expression: [TBL_NM]+'-'+[TBL_NM_TXT]
Power BI DAX: [Table Name] & "-" & [Table Description]
Conversion  : ✅ FULL


### CC-07 | Customer Group 1
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1097189499375370245
Tableau Expression: [CUST_GRP]+' '+[CUST_GRP_TXT]
Power BI DAX: [Customer group] & " " & [Customer Group Description]
Conversion  : ✅ FULL


### CC-08 | Business Partner
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1097189499373989888
Tableau Expression: [PTNR]+'-'+[PTNR_NM] with null handling

Power BI DAX:
─────────────
VAR Partner = [Partner]
VAR PartnerName = [Partner Name]
RETURN
IF(
    ISBLANK(Partner) || ISBLANK(PartnerName) || Partner = "" || PartnerName = "",
    "Null",
    Partner & "-" & PartnerName
)

Pattern     : Extended null/blank handling — both ISBLANK and empty string checks.
Conversion  : ✅ FULL


### CC-09 | Time Period
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1137440398163296257

Tableau Expression:
───────────────────
IF [PST_IND]='1 - 6 Months' THEN 'Past 6 Months'
ELSEIF [PST_IND]='7 - 12 Months' THEN 'Past 7 - 12 Months'
ELSEIF [PST_IND]='13 - 15 Months' THEN 'Past 13 - 15 Months'
ELSE '' END

Power BI DAX:
─────────────
IF([Past Month Indicator] = "1 - 6 Months", "Past 6 Months",
    IF([Past Month Indicator] = "7 - 12 Months", "Past 7 - 12 Months",
        IF([Past Month Indicator] = "13 - 15 Months", "Past 13 - 15 Months", "")))

Pattern     : Tableau IF/ELSEIF → nested DAX IF
Conversion  : ✅ FULL


### CC-10 | View (Parameter-Driven)
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1097189499376717835

Tableau Expression:
───────────────────
CASE [Parameter 1]
  WHEN "Customer" THEN [Customer]
  WHEN "User" THEN [Changed By]
  WHEN "Company" THEN [Company]
END

Power BI DAX:
─────────────
VAR _p = SELECTEDVALUE('Parameter'[Parameter Fields])
RETURN
SWITCH(
    _p,
    "Customer", [Customer],
    "User",     [Changed By],
    "Company",  [Company],
    BLANK()
)

Depends on  : PARAM-01 [Parameter Fields], CC-01, CC-02, CC-03
Pattern     : Tableau CASE [Parameter] → DAX SWITCH(SELECTEDVALUE(...))
Conversion  : ✅ FULL


### CC-11 | Key Date 1 (Date Conversion)
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1097189499376312330
PBI DataType    : dateTime

Tableau Expression: DATE(DATEPARSE("yyyyMMdd", [LST_DT]))
Power BI DAX:
DATE(VALUE(LEFT([Key Date], 4)), VALUE(MID([Key Date], 5, 2)), VALUE(RIGHT([Key Date], 2)))

Pattern     : Tableau DATEPARSE("yyyyMMdd") → DAX DATE(VALUE(LEFT/MID/RIGHT))
NOTE        : Has date variation to LocalDateTable_154c7730.
Conversion  : ✅ FULL


### CC-12 through CC-16 | Change Type Flag Columns
─────────────────────────────────────────────────────────────
Five CASE-based flag columns that filter Change ID by Field type:

| CC   | Column Name                              | lineageTag                                | Field Filter                |
|------|------------------------------------------|-------------------------------------------|-----------------------------|
| CC-12| # of Credit Limit Changes                | Calculation_526639724764233729             | Field = "CREDIT_LIMIT"      |
| CC-13| # of Check Rule Changes                  | # of Credit Limit Changes (copy 2)        | Field = "CHECK_RULE"        |
| CC-14| # of Risk Class Changes                  | # of Credit Limit Changes (copy)          | Field = "RISK_CLASS"        |
| CC-15| # of New Credit Segment Assignments      | Calculation_312155807596904448             | Field Description = "New..."  |
| CC-16| # of Deleted Credit Segment Assignments  | # of New Credit Segment Assignments (copy)| Field Description = "Deleted..."|

Tableau Pattern: CASE [FLD] WHEN "X" THEN [CHG_ID] ELSE null END
DAX Pattern:     IF([Field] = "X", [Change ID], BLANK())
Conversion  : ✅ FULL (all 5)


### CC-17 | Value New Text 1 (Currency-Switched)
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1079456550524813312
Uses currency switching pattern with SELECTEDVALUE + SWITCH + FIRSTNONBLANK.
Conversion  : ✅ FULL


### CC-18 | Value Old Text 1 (Currency-Switched)
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1079456550525214721
Same currency switching pattern as CC-17 for old values.
Conversion  : ✅ FULL


### CC-19 | Percent Change1
─────────────────────────────────────────────────────────────
PBI lineageTag  : 07cfef7b-9190-4caa-a492-ee5a4bc50e29
Uses field-type guard pattern: BLANK for CHECK_RULE and RISK_CLASS.
Pattern     : DIVIDE([Value New] - [Value Old], [Value Old])
Conversion  : ✅ FULL


### CC-20 | Amount Change_slicer
─────────────────────────────────────────────────────────────
PBI lineageTag  : 5f6a2a25-44d7-4141-8872-8de490478870
Combines currency switching AND field-type guard patterns.
Conversion  : ✅ FULL


### CC-21 | Payer Cust. Acct. Group
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1097189499375095811
Tableau Expression: [CUST_ACCT_GRP_TXT]
Power BI DAX: [Customer Account Group Description]
Conversion  : ✅ FULL


### CC-22 through CC-24 | Hidden Action Columns
─────────────────────────────────────────────────────────────
| CC   | Column Name                | lineageTag                  | DAX                                | Hidden |
|------|----------------------------|-----------------------------|-------------------------------------|--------|
| CC-22| Action (Change Date,View)  | Action (Change Date,View)   | [Change Date] & " , " & [View]     | Yes    |
| CC-23| Action (Key Date )         | Action (Key Date )          | [Key Date 1]                        | Yes    |
| CC-24| Action (View)              | Action (View)               | [View]                              | Yes    |

Pattern     : Tableau action filter → hidden concatenated columns.
⚠️  NOTE: CC-23 [Action (Key Date )] has a trailing space in the name.
    Preserved exactly. Has date variation to LocalDateTable_33f92f42.
Conversion  : ✅ FULL (all 3)


## TABLE: Lookup Dimension (CV_SC_OTC_SD22766)

### CC-25 | Credit Control Area 1
─────────────────────────────────────────────────────────────
[Credit Control Area] & " " & [Credit Control Area Text]
Conversion  : ✅ FULL

### CC-26 | Credit Segment (Lookup)
─────────────────────────────────────────────────────────────
[Ctrdit Segment] & " " & [Credit Segment Text]
⚠️  TYPO NOTE: Source column is "Ctrdit Segment" (misspelling) — preserved exactly.
Conversion  : ✅ FULL

### CC-27 | Company (Lookup)
─────────────────────────────────────────────────────────────
[Company Code] & " " & [Company Code Text]
Conversion  : ✅ FULL


## TABLE: V_LEAP_SECURITY

### CC-28 | Company Code (Security)
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_769271118858903554
Tableau Formula : [COMPANY_CD]
Power BI DAX    : [COMPANY_CD]
Purpose     : Pass-through for RLS relationship join.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Fact Table Measures

### M-01 | Amount Change
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1097189499379068948
PBI Format      : 0.0
Tableau Origin  : [ValueNew] - [ValueOld]
Power BI DAX    : [Value New] - [Value Old]
Depends on      : M-05 [Value New], M-04 [Value Old]
Conversion  : ✅ FULL


### M-02 | Avg. Changes per Customer
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1097189499379515414
PBI Format      : 0.0
Tableau Origin  : COUNTD([CHG_ID]) / COUNTD([CUST_NM1])
Power BI DAX    : DIVIDE(DISTINCTCOUNT([Change ID]), DISTINCTCOUNT([Customer Name1]))
Pattern         : Tableau COUNTD/COUNTD → DAX DIVIDE(DISTINCTCOUNT, DISTINCTCOUNT)
Conversion  : ✅ FULL


### M-03 | Percent_Change1 (Measure)
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_1097189499380039704
Uses field-type guard + DIVIDE(ValueNew - ValueOld, ValueOld).
Conversion  : ✅ FULL


### M-04 | Value Old (Currency-Switched)
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_700309755790241793
PBI Format      : 0.00
Tableau Origin  : CASE [Parameter 2] WHEN 'USD' THEN [VAL_OLD_NMRC_USD] WHEN 'Local' THEN [VAL_OLD_NMRC]
Power BI DAX    : SELECTEDVALUE('Currency'[Currency]) → SWITCH → SUM([Value Old Numeric USD]) or SUM([Value Old Numeric])
Conversion  : ✅ FULL


### M-05 | Value New (Currency-Switched)
─────────────────────────────────────────────────────────────
PBI lineageTag  : Calculation_700309755790512130
PBI Format      : 0.00
Same currency switching pattern as M-04 for new values.
Conversion  : ✅ FULL


### M-06 | Displayed Currency
─────────────────────────────────────────────────────────────
PBI lineageTag  : 4fc610cb-e895-4d59-a3b3-71207ed39da1
Tableau Origin  : IF [Parameter 2]='USD' THEN 'USD' ELSE 'Local'
Power BI DAX    : if(SELECTEDVALUE('Currency'[Currency])="USD","USD","Local")
Conversion  : ✅ FULL


### M-07 through M-11 | Change Type Count Measures
─────────────────────────────────────────────────────────────
| M    | Measure Name                              | lineageTag                               | Filter                              |
|------|-------------------------------------------|------------------------------------------|--------------------------------------|
| M-07 | # of Check Rule Changes                   | 4ba686eb-57db-4c51-bd38-31d52cdd71e8     | Field = "CHECK_RULE"                |
| M-08 | # of Credit Limit Changes                 | 405b3fc4-9247-44af-b97c-4e9cf3725ce7     | Field = "CREDIT_LIMIT"              |
| M-09 | # of Deleted Credit Segment Assignments   | d142eb84-05d4-4dc6-abc9-e41e9ad1986d     | Field Description = "Deleted..."    |
| M-10 | # of New Credit Segment Assignments       | 5495ca2e-18d2-4c42-8333-d34e27f9585b     | Field Description = "New..."        |
| M-11 | # of Risk Class Changes                   | 929eca71-36eb-4f22-b563-a63663cc8c0b     | Field = "RISK_CLASS"                |

Pattern: CALCULATE(DISTINCTCOUNT([Change ID]), [Field] = "X")
Conversion  : ✅ FULL (all 5)


### M-12 through M-20 | Remaining Measures
─────────────────────────────────────────────────────────────
| M    | Measure Name         | lineageTag                               | DAX Summary                              |
|------|----------------------|------------------------------------------|------------------------------------------|
| M-12 | Value New 1          | 519704d8-9130-431c-9dc3-dfd4c0d532ed     | Currency-switched SUM                    |
| M-13 | Value New_tp         | f4cb93ef-0bdd-4022-a158-5e4754101799     | Currency-switched SUM (format #,0)       |
| M-14 | Value Old_tp         | 56cb0252-43fe-4258-ac60-583d0b430a64     | Currency-switched SUM (format #,0)       |
| M-15 | Amount Change_tp     | a333fca4-cd78-41d3-b1d2-712779adee1f     | [Value New] - [Value Old]                |
| M-16 | % amount change      | fc23c5e7-dd39-4242-b8c6-74cdcc8c7e9c     | SUM([Percent Change])                    |
| M-17 | # change id          | fda99f9a-a3a9-4970-a284-3150a8194bb5     | DISTINCTCOUNT([Change ID])               |
| M-18 | # Customers          | ebc534c0-93cc-4c05-aed5-63a4c4117ba6     | DISTINCTCOUNTNOBLANK([customer])          |
| M-19 | key date             | a68bafeb-fc98-4b68-bf33-993bea4d9e8f     | MAX([Key Date 1])                        |
| M-20 | min/max_change_date  | 7dd5b92a / 22ec2817                      | MIN/MAX([Change Date])                   |

### Measures Table Measures
─────────────────────────────────────────────────────────────
| M    | Measure Name       | lineageTag                               | DAX                                |
|------|--------------------|------------------------------------------|------------------------------------|
| M-21 | [Number of Records]| Number of Records                        | 1                                  |
| M-22 | Blank              | Calculation_1097189499374125057           | "" (hidden)                        |
| M-23 | Download           | Calculation_1097189499378315281           | "DL" (hidden)                      |
| M-24 | Index              | Calculation_1097189499379838999           | ROW("INDEX", 1)                    |
| M-25 | Index 1            | Calculation_940970882875871235            | RANKX(ALL('Measures Table'), 1)    |
| M-26 | blank_CVQ          | a967f67b-7da7-4729-ad79-4fd50f13ea86     | ""                                 |
| M-27 | date (Refresh)     | (RefreshTable)                           | MAX('RefreshTable'[RefreshTable])  |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU INITIAL TRANSFORMATIONS → POWER BI POWER QUERY (M)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | Fact Table — SAP HANA
─────────────────────────────────────────────────────────────
Tableau Source  : SAP HANA (federated.1avm7vn0royxgl15k2jrc0tr52gc)
                  _SYS_BIC.QUERIES.CORPORATE_FUNCTIONS.GLOBAL_FINANCE.LEAPFROG/CVQ_SC_SD_GOTC_CRLMT_UPDT
PBI Connection  : SapHana.Database("pahpdb01.net.bms.com:30015", [Implementation = "2.0"])
PBI Method      : Value.NativeQuery(Source, SqlQuery, null, [EnableFolding = false])
PBI Load Mode   : Import
PBI lineageTag  : 748cdb9d-17af-4f3d-bffd-84fc3ac1592b

M Query steps:
  1. SapHana.Database connection
  2. SELECT all 36 columns from HANA view
  3. Table.RenameColumns (SAP technical → business names, 36 renames)
  4. Table.AddColumn "sort_order_tp" (integer sort for Past Month Indicator)

⚠️  NOTE: Uses EnableFolding = false (unlike k1/k2 which use true).

### SAP HANA → Power BI Field Rename Map (36 columns)
─────────────────────────────────────────────────────────────
| SAP Technical Name | Power BI Column Name              | Data Type  |
|--------------------|-----------------------------------|------------|
| AMT_CHG            | Amount Change                     | double     |
| AMT_CHG_ABSL       | Absolute Amount Change            | double     |
| CHG_BY_ID          | ChangedBy                         | string     |
| CHG_BY_NM          | ChangedByName                     | string     |
| CHG_DT             | Change Date                       | dateTime   |
| CHG_ID             | Change ID                         | string     |
| CHG_TIME           | Change Time                       | dateTime   |
| CLNT               | Client                            | string     |
| CMPY               | Company Code                      | string     |
| CMPY_TXT           | Company Code Description          | string     |
| CNCY_KEY           | Currency Key                      | string     |
| CR_SEG             | Credit Segment                    | string     |
| CR_SEG_TXT         | Credit Segment Description        | string     |
| CUST               | Customer                          | string     |
| CUST_ACCT_GRP      | Customer Account Group            | string     |
| CUST_ACCT_GRP_TXT  | Customer Account Group Description| string     |
| CUST_GRP           | Customer Group                    | string     |
| CUST_GRP_TXT       | Customer Group Description        | string     |
| CUST_NM1           | Customer Name1                    | string     |
| FLD                | Field                             | string     |
| FLD_DESC           | Field Description                 | string     |
| LST_DT             | Key Date                          | string     |
| PCTG_CHG           | Percent Change                    | double     |
| PCTG_CHG_ABS       | Absolute Percent Change           | double     |
| PST_IND            | Past Month Indicator              | string     |
| PTNR               | Partner                           | string     |
| PTNR_NM            | Partner Name                      | string     |
| REGN               | Region                            | string     |
| RLTP_CAT           | Relationship Category             | string     |
| TBL_NM             | Table Name                        | string     |
| TBL_NM_TXT         | Table Description                 | string     |
| VAL_NEW_NMRC       | Value New Numeric                 | double     |
| VAL_NEW_NMRC_USD   | Value New Numeric USD             | double     |
| VAL_NEW_TXT        | Value New Text                    | string     |
| VAL_NEW_TXT_USD    | Value New Text USD                | string     |
| VAL_OLD_NMRC       | Value Old Numeric                 | double     |
| VAL_OLD_NMRC_USD   | Value Old Numeric USD             | double     |
| VAL_OLD_TXT        | Value Old Text                    | string     |
| VAL_OLD_TXT_USD    | Value Old Text USD                | string     |

PQ-added column: sort_order_tp (integer, not from SAP)


### PQ-02 | Lookup Table — SAP HANA (Cube/Analytic)
─────────────────────────────────────────────────────────────
PBI Table       : QUERIES CORPORATE_FUNCTIONS GLOBAL_FINANCE LEAPFROG CV_SC_OTC_SD22766
PBI lineageTag  : 7e51a772-00a7-4a4a-94bc-2bd2a70cdfd1
HANA Object     : CV_SC_OTC_SD_CCA_CS_CC
Connection Type : SapHana.Database with Cube.Transform
Fields loaded   : CREDIT_SGMNT, CREDIT_SGMNT_TXT, KKBER, KKBTX, BUKRS, BUTXT


### PQ-03 | Security Table — Oracle
─────────────────────────────────────────────────────────────
PBI Table       : V_LEAP_SECURITY
PBI lineageTag  : 73a4cb28-016b-419b-8e96-1d6dd9490607
Oracle Server   : uslvlbmsasp013.net.bms.com:1521/IHUBP
Oracle Schema   : LEAPFROG
Selected columns: BMSUID, COMPANY_CD, COUNTRY


### PQ-04 | RefreshTable (Utility)
─────────────────────────────────────────────────────────────
M Query: DateTimeZone.UtcNow() → RemoveZone → DateTime.ToText("MM/dd/yyyy hh:mm:ss tt")
Measure: date = MAX('RefreshTable'[RefreshTable])


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — CONVERSION COMPLETENESS SCORECARD
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Category                         | Items | PBI Full | PBI Partial | Not Converted | % Complete |
|----------------------------------|-------|----------|-------------|---------------|------------|
| Parameters / Slicer Tables       | 3     | 3        | 0           | 0             | 100%       |
| Calculated Columns (Fact)        | 24    | 24       | 0           | 0             | 100%       |
| Calculated Columns (Lookup)      | 3     | 3        | 0           | 0             | 100%       |
| Calculated Columns (Security)    | 1     | 1        | 0           | 0             | 100%       |
| Measures                         | 27    | 27       | 0           | 0             | 100%       |
| Power Query / Data Sources       | 4     | 4        | 0           | 0             | 100%       |
| Relationships                    | 5+    | 5+       | 0           | 0             | 100%       |
| RLS Roles                        | 1     | 1        | 0           | 0             | 100%       |
| Hierarchies                      | 1     | 1        | 0           | 0             | 100%       |
| TMDL Infrastructure              | 7     | 7        | 0           | 0             | 100%       |
| TOTAL                            | 76+   | 76+      | 0           | 0             | 🟢 100%    |

Table Count Summary:
  Fact Tables                 : 1
  Lookup / Dimension Tables   : 2
  Disconnected Parameter Tables: 3
  Utility Tables              : 1
  Auto Date Tables (hidden)   : 4
  Measure Tables              : 1
  Total Tables in Model       : 12


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — CONVERSION PATTERN CHEAT SHEET
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PATTERN-01 | Currency Switching Pattern
─────────────────────────────────────────────────────────────
Used in: Value New, Value Old, Value New_tp, Value Old_tp,
         Value New 1, Value New Text 1, Value Old Text 1, Amount Change_slicer

VAR SelectedCurrency = SELECTEDVALUE('Currency'[Currency], "USD")
RETURN SWITCH(
    TRUE(),
    SelectedCurrency = "USD",   SUM([...Numeric USD]),
    SelectedCurrency = "Local", SUM([...Numeric]),
    BLANK()
)


PATTERN-02 | Field-Type Guard Pattern
─────────────────────────────────────────────────────────────
Used in: Percent_Change1, Percent Change1 (column), Amount Change_slicer

IF(
    [Field] = "CHECK_RULE Rule for Credit Check" ||
    [Field] = "RISK_CLASS Risk Class",
    BLANK(),
    <actual calculation>
)

Purpose: Excludes non-numeric change types from percentage/amount calculations.


PATTERN-03 | Null Handling Migration
─────────────────────────────────────────────────────────────
Tableau IFNULL([field], '')             → DAX COALESCE([column], "")
Tableau IFNULL([field], ' No NAME')    → DAX COALESCE([column], " No NAME")
Tableau ZN([value])                    → DAX IF(ISBLANK(val) || val=0, BLANK(), val)
Tableau null in CASE/WHEN              → DAX BLANK()


PATTERN-04 | Aggregation Migration
─────────────────────────────────────────────────────────────
Tableau COUNTD([CHG_ID])        → DAX DISTINCTCOUNT([Change ID])
Tableau COUNTD([CUST])          → DAX DISTINCTCOUNTNOBLANK([customer])
Tableau COUNTD([CUST_NM1])      → DAX DISTINCTCOUNT([Customer Name1])
Tableau SUM([VAL_NEW_NMRC_USD]) → DAX SUM([Value New Numeric USD])
Tableau Index() table calc      → DAX ROW("INDEX", 1) or RANKX(ALL('Measures Table'), 1)


PATTERN-05 | Date Handling Migration
─────────────────────────────────────────────────────────────
Tableau DATEPARSE("yyyyMMdd", [LST_DT])
    → DAX DATE(VALUE(LEFT(...,4)), VALUE(MID(...,5,2)), VALUE(RIGHT(...,2)))

Tableau Year drill on CHG_DT   → LocalDateTable_57416a29 Date Hierarchy
Tableau Key Date as date       → Key Date 1 column → LocalDateTable_154c7730


PATTERN-06 | Sort Order Patterns
─────────────────────────────────────────────────────────────
Past Month Indicator (Fact)         sorted by sort_order_tp (PQ column)
Past Month Indicator (PastMonthParam) sorted by Sort_Order
Parameter                           sorted by Parameter Order
Month (all LocalDateTables)         sorted by MonthNo
Quarter (all LocalDateTables)       sorted by QuarterNo


PATTERN-07 | Action Filter Migration
─────────────────────────────────────────────────────────────
Tableau: Action (Change Date,View) → PBI: Hidden column [Change Date] & " , " & [View]
Tableau: Action (Key Date )        → PBI: Hidden column [Key Date 1] with date variation
Tableau: Action (View)             → PBI: Hidden column [View]


PATTERN-08 | View-by Parameter → DATATABLE + SWITCH
─────────────────────────────────────────────────────────────
Tableau CASE [Parameter 1] WHEN "Customer" THEN...
    → DAX SWITCH(SELECTEDVALUE('Parameter'[Parameter Fields]), "Customer", ..., "User", ..., "Company", ...)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAP-01 | Compatibility Level Mismatch
  Severity : Low (informational)
  Detail   : model.txt shows compatibilityLevel 1550 while database.txt
             shows 1604. The database-level value (1604) takes precedence.
  Action   : Standardize to 1604 in both files.


GAP-02 | Lookup Table Column Typo
  Severity : Low
  Detail   : Lookup table column "Ctrdit Segment" is a misspelling of
             "Credit Segment". Preserved exactly from PBIP source.
  Action   : Preserve unless stakeholder approves correction.


GAP-03 | Action Column lineageTags Contain Special Characters
  Severity : Medium
  Detail   : Action columns use lineageTags like "Action (Change Date,View)"
             and "Action (Key Date )" which contain spaces, parentheses,
             and commas — violating M11 rules.
  Action   : Replace with kebab-case lineageTags per K3 M11 rules
             before loading into PBI Desktop.


GAP-04 | SAP HANA Gateway Required
  Severity : High (infrastructure)
  Detail   : SAP HANA connection requires on-premise Data Gateway.
             Credentials: LEAPFROG user (username/password).
  Action   : Configure gateway + Basic credentials.


GAP-05 | Oracle Gateway Required
  Severity : High (infrastructure)
  Detail   : Oracle V_LEAP_SECURITY connection requires on-premise
             Data Gateway to uslvlbmsasp013.net.bms.com:1521/IHUBP.
  Action   : Configure gateway for Oracle LEAPFROG schema.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — FILE INVENTORY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| #  | Role   | File Name                                                                           | Purpose                                          |
|----|--------|-------------------------------------------------------------------------------------|--------------------------------------------------|
| 1  | INPUT  | cl_1__customer_credit_management_updates.txt                                        | Tableau TWB source workbook                      |
| 2  | OUTPUT | cvq_sc_sd_gotc_crlmt_updt_queries_corporate_functions_global_finaff79d.txt          | Fact table (columns, calc cols, measures, M)      |
| 3  | OUTPUT | currency.txt                                                                        | Currency disconnected slicer + measures           |
| 4  | OUTPUT | queries_corporate_functions_global_finance_leapfrog_cv_sc_otc_sd22766.txt           | Lookup dimension table                            |
| 5  | OUTPUT | v_leap_security.txt                                                                 | Security table (Oracle)                           |
| 6  | OUTPUT | measures_table.txt                                                                  | Centralized DAX measures table                    |
| 7  | OUTPUT | parameter.txt                                                                       | View-by parameter table                           |
| 8  | OUTPUT | pastmonthparameter.txt                                                              | Past Month time bucket slicer                     |
| 9  | OUTPUT | refreshtable.txt                                                                    | Data refresh timestamp table                      |
| 10 | OUTPUT | relationships.txt                                                                   | All model relationships                           |
| 11 | OUTPUT | security_groups.txt                                                                 | RLS role definition                               |
| 12 | OUTPUT | expressions.txt                                                                     | M expression parameters                          |
| 13 | OUTPUT | model.txt                                                                           | Model-level metadata                              |
| 14 | OUTPUT | database.txt                                                                        | Database-level metadata                           |
| 15 | OUTPUT | diagramlayout.txt                                                                   | Model diagram layout                              |
| 16 | OUTPUT | datetabletemplate_3be456a8_f031_41d4_ba1a_ad89b5682415.txt                           | Date table template                               |
| 17 | OUTPUT | localdatetable_33f92f42_ebb9_4d43_a332_b924cc3ba913.txt                              | LocalDateTable for Action (Key Date)              |
| 18 | OUTPUT | localdatetable_57416a29_12df_4518_a315_07dd6ce0689d.txt                              | LocalDateTable for Change Date                    |
| 19 | OUTPUT | localdatetable_154c7730_899d_45e0_aa81_cc8955b3dc41.txt                              | LocalDateTable for Key Date 1                     |
| 20 | OUTPUT | en_us.txt                                                                           | Linguistic metadata for Q&A                       |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — SECURITY / ROW-LEVEL SECURITY (RLS)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### RLS-01 | User-Level Filtering
─────────────────────────────────────────────────────────────
Tableau Source  : Published datasource V_LEAP_SECURITY_0 (revision 1.7)
                  with user-level filtering
PBI Table       : V_LEAP_SECURITY
PBI Role File   : security_groups.txt
PBI Pattern     : USERPRINCIPALNAME() filtering

Security Architecture:
  - V_LEAP_SECURITY table loaded from Oracle (BMSUID, COMPANY_CD, COUNTRY)
  - CC-28 [Company Code] pass-through column for join
  - RLS role uses USERPRINCIPALNAME() to match BMSUID
  - Company Code relationship filters fact table rows by user's company access

⚠️  NOTE: Tableau user-level filtering migrated to Power BI using
    USERPRINCIPALNAME() in the RLS role definition. Test with
    actual user accounts before production deployment.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — TABLEAU TO POWER BI FIELD MAPPING (ALL CALCULATED FIELDS)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Tableau Internal Name                          | Tableau Caption                         | Power BI Name                           | PBI Type             |
|------------------------------------------------|-----------------------------------------|-----------------------------------------|----------------------|
| Calculation_1097189499375738887                 | Customer                                | Customer 1                              | Calc Column (Fact)   |
| Calculation_1097189499377860623                 | Company                                 | Company                                 | Calc Column (Fact)   |
| Calculation_1097189499377184781                 | Changed By                              | Changed By                              | Calc Column (Fact)   |
| Calculation_1097189499376123913                 | Field                                   | Field 1                                 | Calc Column (Fact)   |
| Calculation_50102552973942784                   | Credit Segment                          | Credit Segment 1                        | Calc Column (Fact)   |
| Calculation_50102552974745601                   | Table Name                              | Table Name 1                            | Calc Column (Fact)   |
| Calculation_1097189499375370245                 | Customer Group                          | Customer Group 1                        | Calc Column (Fact)   |
| Calculation_1097189499373989888                 | Business Partner                        | Business Partner                        | Calc Column (Fact)   |
| Calculation_1097189499375095811                 | Payer Cust. Acct. Group                 | Payer Cust. Acct. Group                 | Calc Column (Fact)   |
| Calculation_700309755802419204                  | Credit Control Area                     | Credit Control Area 1                   | Calc Column (Lookup) |
| Calculation_940970882875174912                  | Credit Segment                          | Credit Segment                          | Calc Column (Lookup) |
| Calculation_940970882875666434                  | Company                                 | Company                                 | Calc Column (Lookup) |
| Calculation_1097189499376312330                 | Key Date                                | Key Date 1                              | Calc Column (Fact)   |
| Calculation_1137440398163296257                 | Time Period                             | Time Period                             | Calc Column (Fact)   |
| Calculation_1097189499376717835                 | View                                    | View                                    | Calc Column (Fact)   |
| Calculation_526639724764233729                  | # of Credit Limit Changes               | # of Credit Limit Changes               | Calc Column (Fact)   |
| # of Credit Limit Changes (copy 2)             | # of Check Rule Changes                 | # of Check Rule Changes                 | Calc Column (Fact)   |
| # of Credit Limit Changes (copy)               | # of Risk Class Changes                 | # of Risk Class Changes                 | Calc Column (Fact)   |
| Calculation_312155807596904448                  | # of New Credit Segment Assignments     | # of New Credit Segment Assignments     | Calc Column (Fact)   |
| # of New Credit Segment Assignments (copy)     | # of Deleted Credit Segment Assignments | # of Deleted Credit Segment Assignments | Calc Column (Fact)   |
| Calculation_1079456550524813312                 | Value New Text                          | Value New Text 1                        | Calc Column (Fact)   |
| Calculation_1079456550525214721                 | Value Old Text                          | Value Old Text 1                        | Calc Column (Fact)   |
| Calculation_1097189499379068948                 | Amount Change                           | Amount Change                           | Measure (Fact)       |
| Calculation_1097189499379515414                 | Avg. Changes per Cust.                  | Avg. Changes per Customer               | Measure (Fact)       |
| Calculation_1097189499380039704                 | Percent_Change1                         | Percent_Change1                         | Measure (Fact)       |
| Calculation_700309755790241793                  | Value Old                               | Value Old                               | Measure (Fact)       |
| Calculation_700309755790512130                  | Value New                               | Value New                               | Measure (Fact)       |
| Calculation_877076032733638656                  | Displayed Currency                      | Displayed Currency                      | Measure (Currency)   |
| Calculation_1097189499374125057                 | Blank                                   | Blank                                   | Measure (Measures)   |
| Calculation_1097189499378315281                 | Download                                | Download                                | Measure (Measures)   |
| Calculation_1097189499379838999                 | Index                                   | Index                                   | Measure (Measures)   |
| Calculation_769271118858903554                  | Company Code                            | Company Code                            | Calc Column (Security)|


END OF KNOWLEDGE FILE
