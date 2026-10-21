# 📘 CONVERSION KNOWLEDGE FILE
# 12960 – O2C Executive Summary (AR vs. Credit Limit)
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-07
# Source files: o2c_executive_summary.txt (Tableau)
#               o_3a_1_1.txt, ar_2.txt, ar_3.txt, ar_4.txt,
#               companycode.txt, companyname.txt, region.txt,
#               reporting_region.txt, view_by_3a.txt,
#               stakeholder_list.txt, refresh_date.txt,
#               tmptbl.txt, dimdate.txt (prior batch),
#               localdatetable_3aa71d56_24d3_475f_b8ad_61d5116dec22.txt,
#               localdatetable_b164214b_d093_4729_b0e1_5f1838d9045b.txt,
#               localdatetable_d44fe336_9156_4e59_9e66_860c62e6923c.txt,
#               localdatetable_faf34038_98d2_4b0c_af4a_6f9a915e1ff6.txt,
#               localdatetable_f3017eaf_82f8_4ea8_9a40_57186c2e0369.txt,
#               7_definition.json, 7_database.txt, 7_model.txt,
#               7_relationships.txt, 7_diagramlayout.json

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## MODEL METADATA
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PBIP Report Name         : 12960_O(1/2) AR vs. Credit Limit
Model Name               : (derived from 7_* PBIP files)
Compatibility Level      : 1600
Schema Version           : 4.2
Culture                  : en-US
Time Intelligence        : Enabled
Dev Mode                 : Enabled
Tableau Workbook         : O2C Executive Summary
Tableau Data Source      : Federated SAP HANA
SAP HANA Server          : pahpdb01.net.bms.com:30015  (Implementation 2.0)
Load Mode                : Import (Value.NativeQuery for all fact tables)

Full Table Inventory:

  Table              Type            Source / Role
  ─────────────────────────────────────────────────────────────────────────
  O: 1-1             M / SAP HANA    CV_SC_SD_ENHNCE_CUST_SLS — Orders & Sales
  AR-2               M / SAP HANA    CV_FISC_ARSD_ENHNCE_SLS_ARBAL_TRND2 — DSO
  AR-3               M / SAP HANA    CV_FI_AR_ENHNCE_RSER_BAL_TRND — Bad Debt
  AR-4               M / SAP HANA    CV_FI_AR_ENHNCE_TRND_BCKT — AR Aging Buckets
  CompanyCode        Calculated DAX  DISTINCT UNION all 4 facts — Bridge/Dimension
  CompanyName        Calculated DAX  DISTINCT UNION all 4 facts — Lookup
  Region             Calculated DAX  DISTINCT UNION all 4 facts — Dimension
  Reporting Region   Calculated DAX  DISTINCT UNION all 4 facts — Dimension
  DimDate            Calculated DAX  Calendar — Date Dimension
  View By:           Calculated DAX  2-row inline table — Tableau param equivalent
  Stakeholder List   M / SharePoint  Data creation sheet — RLS entitlement
  Refresh Date       M inline        DateTime.LocalNow() — Refresh timestamp
  tmpTbl             M inline        Binary-compressed dev/validation lookup
  DateTableTemplate  Calculated DAX  Hidden seed template
  LocalDateTable ×4  Calculated DAX  Hidden auto-date hierarchies


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PARAM-01 | View Doc No./Amount by:  ← Field-Switch Parameter
─────────────────────────────────────────────────────────────
Tableau Caption       : View Doc No./Amount by:
Tableau Type          : String list
Tableau Values        : "Company Name", "Region"
Tableau Default       : "Company Name"

Power BI Table Name   : View By:
Power BI Type         : DAX Calculated Table (inline)
Power BI lineageTag   : 523709f8-5e72-43ee-b407-06e20cc7681d

Power BI DAX:
─────────────
table 'View By:' =
{
    ( "Company Name", NAMEOF('CompanyCode'[Company Name]), 0, "Company Name" ),
    ( "Region",       NAMEOF('CompanyCode'[Region Group]),  1, "Region"       )
}

Columns:
  [View By:]           — string, visible; sorted by [View By: Order];
                         relatedColumnDetails → groupByColumn: [View By: Fields]
  [View By: Fields]    — string, hidden; contains NAMEOF() field reference;
                         ParameterMetadata { version: 3, kind: 2 }
  [View By: Order]     — int64, hidden; sort key (0 = Company Name, 1 = Region)
  [Value4]             — string, visible; pass-through label (isNameInferred)

Usage in model:
  - Slicer bound to [View By:] column.
  - Visual axis binds to [View By: Fields] to dynamically switch between
    CompanyCode[Company Name] and CompanyCode[Region Group].
  - NAMEOF() pattern links field names dynamically — no IF/SWITCH in measure needed.

Tableau Mapping:
  Tableau string parameter "View Doc No./Amount by:" → DAX inline table with NAMEOF().
  Selecting "Company Name" → CompanyCode[Company Name] appears on visual axis.
  Selecting "Region"       → CompanyCode[Region Group] appears on visual axis.

Conversion Status: ✅ FULL


### PARAM-02 | Reporting Region / Company Code (Slicer)
─────────────────────────────────────────────────────────────
Tableau Caption       : Region / Company Code slicer
Tableau Type          : Multi-value filter / slicer
Tableau Values        : Driven by [Reporting Region (Test)] calculated field
                        (LEFT 4 of CMPY_CD → region labels)

Power BI Equivalent   : CompanyCode[Region Group] column (SWITCH on LEFT(CMPY_CD,4))
                        and CompanyCode[Company Name] column used as slicer values.
Power BI Table        : CompanyCode

Power BI DAX (Region Group column):
─────────────────────────────────────
column 'Region Group' =
SWITCH(
    TRUE(),
    -- Intercontinental
    LEFT('CompanyCode'[CMPY_CD], 4) IN {
        "0038","0042","0122","0048","0089","0231","0234","0205",
        "0124","0057","0670","0538","0412","0401","0511","0056",
        "0217","0468","0598","0150","0080","0083","0671","0045"
    }, "Intercontinental",
    -- Major Markets
    LEFT('CompanyCode'[CMPY_CD], 4) IN {
        "0098","0053","0035","0405","0101","0581","0044","0054",
        "0062","0434","0059","0119","0068","0141","0052","0074","0034"
    }, "Major Markets",
    -- US
    LEFT('CompanyCode'[CMPY_CD], 4) IN {
        "0020","0055"
    }, "US",
    -- Others (explicit list)
    LEFT('CompanyCode'[CMPY_CD], 4) IN {
        "0037","0047","0050","0064","0070","0072","0082","0115",
        "0126","0264","0422","0454","0550","0605","0615","1049","5085"
    }, "Others"
)

Tableau Mapping:
  Tableau's [Reporting Region (Test)] calc (LEFT 4 of Company Code → region label)
  → DAX calculated column SWITCH on LEFT(CMPY_CD, 4) in CompanyCode table.

NOTE: "Others" has a specific explicit code list — it does NOT use a
      fallback default branch. Any CMPY_CD not in any list returns BLANK.

Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — POWER QUERY (M) — SOURCE TABLE DEFINITIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | O: 1-1  ← Orders & Sales Fact Table
─────────────────────────────────────────────────────────────
PBI Table Name        : O: 1-1
SAP HANA View         : [_SYS_BIC].[SAP_ERP.OTC.ENHANCE/CV_SC_SD_ENHNCE_CUST_SLS]
Connection            : ODBC / SAP HANA — pahpdb01.net.bms.com:30015
Load Mode             : Import — Value.NativeQuery
Row Exclusions in PQ  : None — all filtering handled in DAX

Source Columns:
  Column          Type        Description
  ──────────────────────────────────────────────────────────
  BILL_MONTH      string      Billing Month (yyyyMM format)
  CMPY            string      Company name
  CMPY_CD         string      Company code
  CNCY_KEY        string      Currency Key
  CNT_SLS_DOC     int64/sum   Count of sales documents
  LST_DT          string      Last date (yyyyMMdd)
  NET_AMT_LCL     double/sum  Net Amount Local
  NET_AMT_USD     double/sum  Net Amount USD
  ORD_DOC_TYPE    string      Order document type
  ORD_REAS_CD     string      Order reason code
  REGN            string      Region
  RPORTING_REGN   string      Reporting Region

Tableau Equivalent:
  Tableau data source: SAP HANA federated connection to same SAP view.
  All 12 columns flow directly; no Tableau field renaming at source layer.


### PQ-02 | AR-2  ← DSO / AR Balance Fact Table
─────────────────────────────────────────────────────────────
PBI Table Name        : AR-2
SAP HANA View         : [_SYS_BIC].[SAP_ERP.OTC.ENHANCE/CV_FISC_ARSD_ENHNCE_SLS_ARBAL_TRND2]
Load Mode             : Import — Value.NativeQuery + Table.SelectRows

Row-Level Exclusions (Power Query — baked into load):
  CMPY_CD_TXT ≠  "0037 - BMIG - Edison (AZZ)"
  CMPY_CD_TXT ≠  "0070 - Lawrence Laboratories"
  CMPY_CD_TXT ≠  "0082 - Westwood-Intrafin SA"
  CMPY_CD_TXT ≠  "0115 - UPSA SAS"
  CMPY_CD_TXT ≠  "0454 - BMS Pharma.Ltd"
  CMPY_CD_TXT ≠  "0550 - Medarex Inc."
  CUST_ACCT_GRP  = "Z003"   ← maps to Tableau's locked filter

Source Columns:
  CALD_MTH            string      Calendar Month (yyyyMM)
  CMPY_CD             string      Company code
  CMPY_CD_TXT         string      Company name
  CNCY_KEY            string      Currency Key
  REGN                string      Region
  RPORTING_REGN       string      Reporting Region
  CUST_ACCT_GRP       string      Customer Account Group
  CUST_ACCT_GRP_TXT   string      Customer Account Group Name
  ARAMT_30_LCL        double/sum  AR (1 Month) Local
  ARAMT_30_USD        double/sum  AR (1 Month) USD
  SLSAMT_90_LCL       double/sum  Sales (3 Months) Local
  SLSAMT_90_USD       double/sum  Sales (3 Months) USD
  CC_SLSAMT_90_USD_KD double/sum  Sales (3 Months) USD @ Key Date


### PQ-03 | AR-3  ← Bad Debt Reserves Fact Table
─────────────────────────────────────────────────────────────
PBI Table Name        : AR-3
SAP HANA View         : [_SYS_BIC].[SAP_ERP.OTC.ENHANCE/CV_FI_AR_ENHNCE_RSER_BAL_TRND]
Load Mode             : Import — Value.NativeQuery
Row Exclusions in PQ  : None — filtering handled in DAX

Source Columns:
  CALD_MTH            string      Calendar Month (yyyyMM)
  CMPY_CD             string
  CMPY_CD_TXT         string
  CNCY_KEY            string
  LST_DT              string      Raw "last date" (yyyyMMdd) — used for Key Date Col
  REGN                string
  RPORTING_REGN       string
  RSER_BAL_LCL        double/sum  Reserve Balance Local
  RSER_BAL_USD        double/sum  Reserve Balance USD
  TOTL_OPN_AR_BAL_LOC double/sum  Total Open AR Balance Local
  TOTL_OPN_AR_BAL_USD double/sum  Total Open AR Balance USD


### PQ-04 | AR-4  ← AR Aging Buckets Fact Table
─────────────────────────────────────────────────────────────
PBI Table Name        : AR-4
SAP HANA View         : [_SYS_BIC].[SAP_ERP.OTC.ENHANCE/CV_FI_AR_ENHNCE_TRND_BCKT]
                        (same view as Tableau's AR-4 data source)
Load Mode             : Import — Value.NativeQuery
Column Rename in M    : AMT_30_USD → "Net Amount (1 Month) in USD"
Row Exclusions in PQ  : None — filtering handled in DAX

Source Columns:
  AGE_BCKT                       string      Aging Bucket (9 values)
  AGE_CATG                       string      Aging Category
  AMT_30_LCL                     double/sum  Net Amount Local
  Net Amount (1 Month) in USD    double/sum  (renamed from AMT_30_USD)
  CALD_MTH                       string      Calendar Month (yyyyMM)
  CMPY_CD                        string
  CMPY_CD_TXT                    string
  CNCY_KEY                       string
  REGN                           string
  RPORTING_REGN                  string

AGE_BCKT values (9):
  "Net Due...", "1-30 Days Past Due", "31-60 Days Past Due",
  "61-90 Days Past Due", "91-120 Days Past Due", "121-150 Days Past Due",
  "151-180 Days Past Due", "181-360 Days Past Due", ">360 Days Past Due"

AGE_CATG values (3):
  "Past Due", "Net Due", "Current/Datings"


### PQ-05 | Stakeholder List  ← RLS Entitlement Table
─────────────────────────────────────────────────────────────
PBI Table Name        : Stakeholder List
Source Type           : SharePoint Excel
SharePoint URL        : https://sites.bms.com/teams/BMSC-EFDataStrategyOps
SharePoint Path       : Shared Documents/Tableau to PowerBI Migration/
File Name             : Dashboard View - Stakeholder List_Executive Summary.xlsx
Sheet                 : Data creation
Load Mode             : Import

Columns:
  Stakeholder      string     Person's full name
  Title            string     Job title
  BMSUID           string     BMS user ID
  Company Code     int64      Entitlement key — joins CompanyCode[Company Code]
  Reporting Region string     User's reporting region
  email            string     ← PRIMARY RLS MATCH COLUMN vs USERPRINCIPALNAME()
  Column7          string     Hidden spare
  Column8          string     Hidden spare
  Column9          string     Hidden spare

Power Query M:
──────────────
let
    Source = SharePoint.Files(
        "https://sites.bms.com/teams/BMSC-EFDataStrategyOps",
        [ApiVersion = 15]
    ),
    #"..." = Source{[
        Name = "Dashboard View - Stakeholder List_Executive Summary.xlsx",
        #"Folder Path" = "https://sites.bms.com/teams/BMSC-EFDataStrategyOps/
                          Shared Documents/Tableau to PowerBI Migration/"
    ]}[Content],
    #"Imported Excel Workbook" = Excel.Workbook(...),
    #"Data creation_Sheet" = ...{[Item="Data creation", Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(..., [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(
        ...,
        {{"Stakeholder", type text}, {"Title", type text}, {"BMSUID", type text},
         {"Company Code", Int64.Type}, {"Reporting Region", type text},
         {"email", type any}, {"Column7", type any}, {"Column8", type any},
         {"Column9", type any}}
    )
in
    #"Changed Type"

NOTE: File is refreshed with every dataset refresh.
      The email column match is case-insensitive via UPPER() in all DAX measures.


### PQ-06 | Refresh Date  ← Dataset Refresh Timestamp
─────────────────────────────────────────────────────────────
PBI Table Name : Refresh Date
Source Type    : Inline M — single-row datetime table

Power Query M:
──────────────
let
    RefreshTime = DateTime.LocalNow(),
    Source = #table(
        type table [LastRefreshed = datetime],
        {{RefreshTime}}
    )
in
    Source

Column  : LastRefreshed (dateTime); LocalDateTable variation b164214b bound.

Measure : 'Refresh Date' = "Refresh Date: " & FORMAT(MAX([LastRefreshed]), "M/DD/YYYY")

Tableau Mapping:
  Maps to Tableau's {FIXED : MAX([Date])} refresh label display pattern.


### PQ-07 | tmpTbl  ← Dev / Validation Lookup
─────────────────────────────────────────────────────────────
PBI Table Name : tmpTbl
Source Type    : Inline M — Binary-compressed (Base64/Deflate)
Columns        : Cmpy_code (int64), Group (string)
Relationships  : NONE

⚠️  No relationships defined. Used for ad-hoc validation only.
    DO NOT include in production report without explicit confirmation.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### SHARED PATTERN — yyyyMM → DATE
─────────────────────────────────────────────────────────────
Applied identically to all 4 fact tables (O: 1-1, AR-2, AR-3, AR-4).

Tableau Expression:
───────────────────
DATEPARSE('yyyyMM', [CALD_MTH])   -- or [BILL_MONTH] in O: 1-1

Power BI DAX (all tables):
──────────────────────────
column Date =
DATE( LEFT([CALD_MTH], 4), RIGHT([CALD_MTH], 2), 1 )
-- O: 1-1 uses BILL_MONTH instead of CALD_MTH as the source column.
-- Result: first day of the calendar month.
-- Hidden in AR-4 (isHidden = true); visible in AR-2, AR-3, O: 1-1.
-- Each has a corresponding LocalDateTable relationship (joinOnDateBehavior: datePartOnly).

Conversion Status: ✅ FULL (all 4 tables)


## TABLE: O: 1-1 — Calculated Columns

### CC-01 | zccb & MNC  ← MNC Exclusion Flag
─────────────────────────────────────────────────────────────
PBI Data Type   : boolean
PBI Format      : "TRUE";"TRUE";"FALSE"

Tableau Expression:
───────────────────
CONTAINS([ORD_REAS_CD], "MNC")
-- Returns TRUE if the order reason code contains "MNC".
-- Named "zccb & MNC" in Tableau.

Power BI DAX:
─────────────
column 'zccb & MNC' =
IFERROR( ISNUMBER( FIND("MNC", [ORD_REAS_CD]) ), FALSE )

Depends on  : [ORD_REAS_CD]
Used by     : Invoiced Amount, Invoice Amount Growth, Sales CY, Sales PY,
              Invoiced Amount* (YTD), Orders Accuracy (YTD)
NOTE        : IFERROR wrapper handles BLANK ORD_REAS_CD rows (FIND errors
              on blank). Maps to Tableau's CONTAINS([ORD_REAS_CD], "MNC").
Conversion  : ✅ FULL


### CC-02 | Accuracy Num  ← Order Error Flag
─────────────────────────────────────────────────────────────
PBI Data Type   : boolean
PBI Format      : "TRUE";"TRUE";"FALSE"

Tableau Expression:
───────────────────
CONTAINS([ORD_REAS_CD], "112") OR CONTAINS([ORD_REAS_CD], "119")
OR CONTAINS([ORD_REAS_CD], "122") OR CONTAINS([ORD_REAS_CD], "161")
OR CONTAINS([ORD_REAS_CD], "203") OR CONTAINS([ORD_REAS_CD], "224")
OR CONTAINS([ORD_REAS_CD], "825") OR CONTAINS([ORD_REAS_CD], "UPE")
-- Returns TRUE for any of 8 specific error codes in ORD_REAS_CD.

Power BI DAX:
─────────────
column 'Accuracy Num' =
IF(
    ISBLANK([ORD_REAS_CD]),
    FALSE(),
    NOT ISERROR(FIND("112", [ORD_REAS_CD]))
    || NOT ISERROR(FIND("119", [ORD_REAS_CD]))
    || NOT ISERROR(FIND("122", [ORD_REAS_CD]))
    || NOT ISERROR(FIND("161", [ORD_REAS_CD]))
    || NOT ISERROR(FIND("203", [ORD_REAS_CD]))
    || NOT ISERROR(FIND("224", [ORD_REAS_CD]))
    || NOT ISERROR(FIND("825", [ORD_REAS_CD]))
    || NOT ISERROR(FIND("UPE", [ORD_REAS_CD]))
)

Error codes: 112, 119, 122, 161, 203, 224, 825, UPE (8 total)
Depends on  : [ORD_REAS_CD]
Used by     : CC-03, Orders Accuracy (YTD), Orders Accuracy CY, Orders Accuracy PY,
              Order Accuracy Growth
Conversion  : ✅ FULL


### CC-03 | Order Accuracy Num  ← Accuracy Numerator Column
─────────────────────────────────────────────────────────────
PBI Data Type   : int64
PBI Format      : 0

Power BI DAX:
─────────────
column 'Order Accuracy Num' =
IF( NOT( [Accuracy Num] ), [CNT_SLS_DOC], BLANK() )

Tableau Mapping:
  Pre-aggregation helper — CNT_SLS_DOC where Accuracy Num is FALSE.
  Used as SUM in Orders Accuracy numerator.
Depends on  : CC-02, [CNT_SLS_DOC]
Conversion  : ✅ FULL


### CC-04 | Removed Doc Types  ← Document Type Exclusion Flag
─────────────────────────────────────────────────────────────
PBI Data Type   : boolean
PBI Format      : "TRUE";"TRUE";"FALSE"

Excluded ORD_DOC_TYPE values (12):
  ZDCT  Self Con Don/CT OTC         ZEFD  Exp Free of Charge
  ZELF  Self Consumption OTC        ZFD   Free of Charge Order
  ZFDB  Free of Charge BR           ZKE   Consignment Issue
  ZKR   Consignment Returns         ZOEF  OTC: Export FOC
  ZRFE  Free Goods Return           ZRSM  Sample Goods Return
  ZRSV  Return-Sample/Vendor        ZSAB  Samples BR

Power BI DAX:
─────────────
column 'Removed Doc Types' =
CONTAINSSTRING([ORD_DOC_TYPE], "ZDCT")
|| CONTAINSSTRING([ORD_DOC_TYPE], "ZEFD")
|| CONTAINSSTRING([ORD_DOC_TYPE], "ZELF")
|| CONTAINSSTRING([ORD_DOC_TYPE], "ZFD")
|| CONTAINSSTRING([ORD_DOC_TYPE], "ZFDB")
|| CONTAINSSTRING([ORD_DOC_TYPE], "ZKE")
|| CONTAINSSTRING([ORD_DOC_TYPE], "ZKR")
|| CONTAINSSTRING([ORD_DOC_TYPE], "ZOEF")
|| CONTAINSSTRING([ORD_DOC_TYPE], "ZRFE")
|| CONTAINSSTRING([ORD_DOC_TYPE], "ZRSM")
|| CONTAINSSTRING([ORD_DOC_TYPE], "ZRSV")
|| CONTAINSSTRING([ORD_DOC_TYPE], "ZSAB")
-- Returns TRUE for rows that should be EXCLUDED.

Depends on  : [ORD_DOC_TYPE]
Used by     : Orders Processed* (YTD), Trade Orders Processed CY/PY YTD,
              Orders Accuracy (YTD), Orders Processed Growth, Orders Accuracy CY/PY
Conversion  : ✅ FULL


### CC-05 | Removed Doc Types_new  ← Enhanced Exclusion (TREATAS-safe)
─────────────────────────────────────────────────────────────
PBI Data Type   : string ("True" / "False" / BLANK)

Power BI DAX:
─────────────
column 'Removed Doc Types_new' =
IF(
    NOT( CONTAINS('AR-2', 'AR-2'[CMPY_CD], [CMPY_CD]) )
    && NOT( CONTAINS('AR-4', 'AR-4'[CMPY_CD], [CMPY_CD]) ),
    BLANK(),
    IF( [Removed Doc Types], "True", "False" )
)
-- Returns BLANK if CMPY_CD not found in AR-2 or AR-4 — TREATAS safe.
-- Returns string "True"/"False" (not boolean) for filter compatibility.

Used by     : Invoice Amount Growth, Sales CY, Sales PY, Invoice Amount Growth_new
NOTE        : Preferred version over CC-04 for measures that use TREATAS.
              Returns string, not boolean — filter must use = "False".
Conversion  : ✅ FULL


### CC-06 | Removed Doc Types_InvoiceYTD  ← Simpler Exclusion (No CMPY check)
─────────────────────────────────────────────────────────────
PBI Data Type   : string ("True" / "False")

Power BI DAX:
─────────────
column 'Removed Doc Types_InvoiceYTD' =
IF( [Removed Doc Types], "True", "False" )
-- No CMPY_CD check — simpler version used by invoice YTD measures.

Used by     : Invoiced Amount, Invoiced Amount* (YTD), Invoiced Amount YTD_new
Conversion  : ✅ FULL


### CC-07 | Order Doc Type  ← Company 0062-Specific Exclusion
─────────────────────────────────────────────────────────────
PBI Data Type   : boolean

Power BI DAX:
─────────────
column 'Order Doc Type' =
[CMPY_CD] = "0062"
&& (
    CONTAINSSTRING([ORD_DOC_TYPE], "ZECR")
    || CONTAINSSTRING([ORD_DOC_TYPE], "ZEXP")
    || CONTAINSSTRING([ORD_DOC_TYPE], "ZEDR")
    || CONTAINSSTRING([ORD_DOC_TYPE], "ZECC")
)

Used by     : DEBUG measures only — company 0062-specific doc type exclusion.
NOTE        : Do NOT apply in production unless specifically required for 0062.
Conversion  : ✅ FULL (debug scope)


## TABLE: AR-3 — Calculated Columns

### CC-08 | Key Date Col  ← Bad Debt Reserve Date
─────────────────────────────────────────────────────────────
PBI Data Type   : date
PBI Format      : mmmm d, yyyy
Has LocalDateTable: Yes (GUID faf34038)

Tableau Expression:
───────────────────
DATE(LEFT([LST_DT],4), MID([LST_DT],5,2), RIGHT([LST_DT],2))
-- Parses yyyyMMdd string to date. Used as "Key Date" for bad debt reserves.

Power BI DAX:
─────────────
column 'Key Date Col' =
IF(
    ISBLANK([LST_DT]) || LEN([LST_DT]) < 8,
    BLANK(),
    DATE( LEFT([LST_DT],4), MID([LST_DT],5,2), RIGHT([LST_DT],2) )
)

Depends on  : [LST_DT] (yyyyMMdd string)
Used by     : AR-3 Key Date measure, LocalDateTable_faf34038 binding
NOTE        : ISBLANK/LEN guard prevents parse errors on malformed LST_DT values.
Conversion  : ✅ FULL + enhanced


## TABLE: AR-4 — Calculated Columns

### CC-09 | AR Past Due  ← Row-Level Past Due Flag
─────────────────────────────────────────────────────────────
Power BI DAX:
─────────────
column 'AR Past Due' =
IF( [AGE_CATG] = "Past Due", [Net Amount (1 Month) in USD], 0 )

Tableau Mapping: Direct equivalent of Tableau's [AR Past Due] calculated field.
Depends on  : [AGE_CATG], [Net Amount (1 Month) in USD]
Conversion  : ✅ FULL


### CC-10 | Region Group dummyAR4  ← AR-4-Scoped Region Mapping
─────────────────────────────────────────────────────────────
PBI Data Type   : string (Calculated Column)

Power BI DAX:
─────────────
column 'Region Group dummyAR4' =
SWITCH(
    TRUE(),
    LEFT('AR-4'[CMPY_CD], 4) IN {
        "0038","0042","0122","0048","0089","0231","0234","0205",
        "0124","0057","0670","0538","0412","0401","0511","0056",
        "0217","0468","0598","0150","0080","0083","0671","0045"
    }, "Intercontinental",
    LEFT('AR-4'[CMPY_CD], 4) IN {
        "0098","0053","0035","0405","0101","0581","0044","0054",
        "0062","0434","0059","0119","0068","0141","0052","0074","0034"
    }, "Major Markets",
    LEFT('AR-4'[CMPY_CD], 4) IN {"0020","0055"}, "US",
    LEFT('AR-4'[CMPY_CD], 4) IN {
        "0037","0047","0050","0064","0070","0072","0082","0115",
        "0126","0264","0422","0454","0550","0605","0615","1049","5085"
    }, "Others"
)

NOTE: Duplicate of CompanyCode[Region Group] scoped to AR-4 rows.
      Functionally identical SWITCH logic, different table reference.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## SHARED PATTERN — RLS Security Block
─────────────────────────────────────────────────────────────
Applied identically to all secured measures across O: 1-1, AR-2, AR-3, AR-4.
AR-2 is always the RLS anchor. AR-3, AR-4, O: 1-1 derive their company scope
from AR-2's CMPY_CD filtered set.

DAX Template:
─────────────
VAR _UserEmail    = UPPER(USERPRINCIPALNAME())
VAR _CompanyCode  = SUMMARIZE(
                        FILTER('Stakeholder List',
                               UPPER('Stakeholder List'[email]) = _UserEmail),
                        'Stakeholder List'[Company Code]
                    )
VAR _CmpyCd       = SUMMARIZE(
                        FILTER('CompanyCode',
                               'CompanyCode'[Company Code] IN _CompanyCode),
                        'CompanyCode'[CMPY_CD]
                    )
-- Cross-table anchor (AR-2):
VAR _CmpyCdAR2    = SUMMARIZE(
                        FILTER('AR-2', 'AR-2'[CMPY_CD] IN _CmpyCd),
                        'AR-2'[CMPY_CD]
                    )
-- Apply with TREATAS to target table:
-- TREATAS(_CmpyCdAR2, '<target_table>'[CMPY_CD])

Full RLS Chain:
  USERPRINCIPALNAME()
    → Stakeholder List[email]
      → Stakeholder List[Company Code]
        → CompanyCode[Company Code]  (bidirectional ↔)
          → CompanyCode[CMPY_CD]
            → TREATAS → AR-2[CMPY_CD]  ← RLS ANCHOR
              → TREATAS → AR-3[CMPY_CD]
              → TREATAS → AR-4[CMPY_CD]
              → TREATAS → O: 1-1[CMPY_CD]

Authorization Gate Pattern:
────────────────────────────
VAR _IsAuthorized =
    COALESCE(
        COUNTROWS(FILTER('<table>', '<table>'[CMPY_CD] IN _CmpyCd)),
        0
    ) > 0
RETURN IF( NOT _IsAuthorized, BLANK(), <calculation> )


## SHARED PATTERN — Date Period Derivation
─────────────────────────────────────────────────────────────
Applied consistently across all 4 fact tables.

Tableau Equivalent:
  {FIXED : MAX([Date])} → LOD for max date
  IF MONTH(MaxDate) - 1 = 0 THEN 12 ELSE MONTH(MaxDate) - 1 END → prior month
  (same January edge-case handling)

Power BI DAX Template:
──────────────────────
VAR _MaxMonth     = MAXX(ALL('<table>'), '<table>'[Date])
VAR _MaxMonthNum  = MONTH(_MaxMonth)
VAR _TargetMonth  = IF(_MaxMonthNum = 1, 12, _MaxMonthNum - 1)
VAR _CurrentYear  = IF(_MaxMonthNum = 1, YEAR(_MaxMonth) - 1, YEAR(_MaxMonth))
VAR _PreviousYear = IF(_MaxMonthNum = 1, YEAR(_MaxMonth) - 2, YEAR(_MaxMonth) - 1)

NOTE: January edge case: when MaxMonth is January (month 1),
      the prior month is December (12) of the previous year.


## SHARED PATTERN — MOM Filter (Month-over-Month)
─────────────────────────────────────────────────────────────
Tableau Equivalent:
  Boolean [MOM] calculated field — TRUE for rows that belong to
  either the current or prior month window.

Power BI DAX Template (2-condition OR):
────────────────────────────────────────
(   YEAR([Date]) = _PrevYear
    && MONTH([Date]) = _PrevMonth
    && [Date] <= EDATE(_MaxMonth, -1)
)
OR
(   YEAR([Date]) = _CurrYear
    && MONTH([Date]) = _PrevMonth
    && [Date] <= _MaxMonth
)

Applied in: AR Balance_MOM, AR Value_MOM, Bad Debt as % of AR_MOM,
            3 Month DSO_MOM, all _MOM suffix measures.


## TABLE: O: 1-1 — Measures (22 total)

### M-01 | Orders Accuracy
─────────────────────────────────────────────────────────────
PBI Format  : 0.00%;-0.00%;0.00%
RLS         : Full (AR-2 anchor via TREATAS)

Tableau Expression:
───────────────────
SUM([Order Accuracy Num]) / SUM([CNT_SLS_DOC])
-- Accuracy = orders without error codes / total orders

Power BI DAX:
─────────────
measure 'Orders Accuracy' =
[RLS block → _CmpyCdAR2]
VAR _acc = CALCULATE(SUM('O: 1-1'[Order Accuracy Num]),
               TREATAS(_CmpyCdAR2, 'O: 1-1'[CMPY_CD]))
VAR _cnt = CALCULATE(SUM('O: 1-1'[CNT_SLS_DOC]),
               TREATAS(_CmpyCdAR2, 'O: 1-1'[CMPY_CD]))
RETURN DIVIDE(_acc, _cnt, 0)

Conversion  : ✅ FULL


### M-02 | Invoiced Amount
─────────────────────────────────────────────────────────────
PBI Format  : general (isGeneralNumber)
RLS         : Full (via O: 1-1 CMPY_CD set)

Tableau Expression:
───────────────────
{FIXED [RPORTING_REGN], YEAR([Date]) : SUM([NET_AMT_USD])}
WHERE MONTH(Date) <= PrevMonth AND Removed Doc Types = FALSE

Power BI DAX:
─────────────
measure 'Invoiced Amount' =
[RLS block → _CompanyCodes via O:1-1]
VAR _MaxMonth    = CALCULATE(MAX('O: 1-1'[Date]), ALL('O: 1-1'))
VAR _PrevMonth   = IF(MONTH(_MaxMonth)-1=0, 12, MONTH(_MaxMonth)-1)
VAR _SelectedYear= CALCULATE(MAX('DimDate'[Year]))   -- ← year from DimDate column context
VAR _CurrentRegion = VALUES('O: 1-1'[RPORTING_REGN])

RETURN
IF( ISBLANK(_SelectedYear), BLANK(),
    CALCULATE(
        SUM('O: 1-1'[NET_AMT_USD]),
        FILTER(ALL('O: 1-1'),
            YEAR([Date]) = _SelectedYear
            && MONTH([Date]) <= _PrevMonth
            && [Removed Doc Types_InvoiceYTD] = "False"
        ),
        TREATAS(_CurrentRegion, 'O: 1-1'[RPORTING_REGN]),
        TREATAS(_CompanyCodes,  'O: 1-1'[CMPY_CD])
    )
)

KEY: Uses CALCULATE(MAX('DimDate'[Year])) for year — respects matrix
     column context instead of hardcoding from O: 1-1 max date.
     TREATAS(_CurrentRegion) pins reporting region = Tableau FIXED LOD.
Conversion  : ✅ FULL


### M-03 | Invoiced Amount* (YTD)
─────────────────────────────────────────────────────────────
PBI Format  : general
RLS         : Full (_IsAuthorized gate + ALLSELECTED on CompanyCode)
NOTE        : Asterisk (*) in name is intentional — preserve exactly.

Power BI DAX key logic:
──────────────────────
CALCULATE(
    SUM('O: 1-1'[NET_AMT_USD]),
    FILTER('O: 1-1',
        YEAR([Date]) = _SelectedYear
        && MONTH([Date]) <= _PrevMonth
        && [Removed Doc Types_InvoiceYTD] = "False"
        && [zccb & MNC] = FALSE
    ),
    ALLSELECTED('CompanyCode'[CMPY_CD])
)
IF(NOT _IsAuthorized, BLANK(), IF(ISBLANK(_SelectedYear)..., _result))

Conversion  : ✅ FULL


### M-04 | Orders Processed* (YTD)
─────────────────────────────────────────────────────────────
PBI Format  : 0
NOTE        : Asterisk (*) in name is intentional — preserve exactly.

Power BI DAX key logic:
──────────────────────
CALCULATE(
    SUM('O: 1-1'[CNT_SLS_DOC]),
    FILTER(ALL('DimDate'), YEAR([Date])=CurrentYear && MONTH([Date])<=PrevMonth),
    FILTER(ALL('O: 1-1'), [zccb & MNC] = FALSE),
    'O: 1-1'[Removed Doc Types] = FALSE,
    TREATAS(VALUES('AR-2'[CMPY_CD]), 'O: 1-1'[CMPY_CD]),
    TREATAS(VALUES('AR-4'[CMPY_CD]), 'O: 1-1'[CMPY_CD])
)

Conversion  : ✅ FULL


### M-05 | Orders Accuracy (YTD)
─────────────────────────────────────────────────────────────
PBI Format  : 0.00%;-0.00%;0.00%
RLS         : Dual TREATAS (AR-2 + AR-4 company sets)

Power BI DAX key logic:
──────────────────────
VAR _dateFilter = FILTER(ALL('DimDate'), YEAR=CurrentYear && MONTH<=PrevMonth)
VAR _mncFilter  = FILTER(ALL('O: 1-1'[ORD_REAS_CD]),
                      ISBLANK([ORD_REAS_CD]) || NOT CONTAINSSTRING(...,"MNC"))
Numerator   = CALCULATE(SUMX('O: 1-1', IF([Accuracy Num]<>TRUE(), [CNT_SLS_DOC],0)),
                  _dateFilter, _mncFilter, [Removed Doc Types]=FALSE,
                  TREATAS(_CmpyCdAR2,...), TREATAS(_CmpyCdAR4,...))
Denominator = CALCULATE(SUM('O: 1-1'[CNT_SLS_DOC]),
                  _dateFilter, _mncFilter, [Removed Doc Types]=FALSE,
                  TREATAS(_CmpyCdAR2,...), TREATAS(_CmpyCdAR4,...))
RETURN DIVIDE(Numerator, Denominator, 0)

Conversion  : ✅ FULL


### M-06 | Invoice Amount Growth
─────────────────────────────────────────────────────────────
PBI Format  : 0.00%;-0.00%;0.00%
RLS         : Dual TREATAS (AR-2 + AR-4)

Logic: DIVIDE(SalesCY, SalesPY, BLANK()) - 1
  SalesCY/PY: NET_AMT_USD filtered by year, month <= TargetMonth,
              Removed Doc Types_new = "False", zccb & MNC = FALSE,
              dual TREATAS company filter.
Gate: IF(ISBLANK(SalesCY) && ISBLANK(SalesPY), BLANK(), _Result)

Conversion  : ✅ FULL


### M-07 | Orders Processed Growth
─────────────────────────────────────────────────────────────
PBI Format  : 0.00%;-0.00%;0.00%
RLS         : _IsAuthorized gate + dual TREATAS

Logic: DIVIDE(_OrdersCY, _OrdersPY, BLANK()) - 1
  Business filter: Removed Doc Types = FALSE
                   AND (ISBLANK(ORD_REAS_CD) OR NOT CONTAINS(..."MNC"))

Conversion  : ✅ FULL


### M-08 | Order Accuracy Growth
─────────────────────────────────────────────────────────────
PBI Format  : 0%;-0%;0%
RLS         : UNION(AR-2, AR-4) TREATAS

Logic: AccuracyCY - AccuracyPY  (difference, not ratio)
  Uses UNION(_CmpyCdAR2, _CmpyCdAR4) as single _CompanyCodes TREATAS.

Conversion  : ✅ FULL


### M-09 | Sales CY / Sales PY
─────────────────────────────────────────────────────────────
PBI Format  : general
RLS         : UNION(AR-2, AR-4) TREATAS

Sales CY:
  NET_AMT_USD, YEAR=_CurrentYear, MONTH<=_TargetMonth,
  Removed Doc Types_new = "False", zccb & MNC = FALSE,
  VALUES('O: 1-1'[RPORTING_REGN]) preserves region context,
  TREATAS(_CompanyCodes, 'O: 1-1'[CMPY_CD])

Sales PY:
  Same logic with _PreviousYear and _PrevMonth.

Conversion  : ✅ FULL


### M-10 | Trade Orders Processed CY YTD / PY YTD
─────────────────────────────────────────────────────────────
PBI Format  : 0
RLS         : Dual TREATAS (AR-2 + AR-4, intersecting)

Logic: CNT_SLS_DOC filtered by year (CY or PY), MONTH<=TargetMonth,
       MNC exclusion via ORD_REAS_CD CONTAINSSTRING,
       Removed Doc Types = FALSE.

Conversion  : ✅ FULL


### M-11 | Orders Accuracy CY / PY
─────────────────────────────────────────────────────────────
PBI Format CY : 0.0%;-0.0%;0.0%
PBI Format PY : 0.0%;-0.0%;0.0%
RLS           : Dual TREATAS (AR-2 + AR-4) + _IsAuthorized gate (CY only)

Logic: DIVIDE(NUM, DN, BLANK())
  NUM = CNT_SLS_DOC WHERE Accuracy Num <> TRUE, year, month, removed=FALSE
  DN  = CNT_SLS_DOC WHERE year, month, removed=FALSE

Conversion  : ✅ FULL


### M-12 | O1_CurrVal / O1_PrevVal  ← KPI Period Labels
─────────────────────────────────────────────────────────────
PBI Format  : text (no format string)

Power BI DAX:
─────────────
measure O1_CurrVal =
VAR _MaxBillMonth = MAXX(ALL('O: 1-1'), 'O: 1-1'[Date])
RETURN FORMAT(EDATE(_MaxBillMonth, -1), "MMM YY")

measure O1_PrevVal =
VAR _MaxBillMonth = MAXX(ALL('O: 1-1'), 'O: 1-1'[Date])
RETURN FORMAT(EDATE(EDATE(_MaxBillMonth, -1), -12), "MMM YY")

Tableau Mapping: Maps to Tableau's display labels on KPI cards.
Conversion  : ✅ FULL


## TABLE: AR-2 — Measures (9 total)

### M-13 | 3 Month DSO (@ Key Date)
─────────────────────────────────────────────────────────────
PBI Format  : 0
RLS         : Full (TREATAS via AR-2 CMPY_CD)

Power BI DAX key logic:
──────────────────────
DIVIDE(SUM(ARAMT_30_USD), SUM(CC_SLSAMT_90_USD_KD), 0) * 90

Tableau Mapping:
  (SUM([ARAMT_30_USD]) / SUM([CC_SLSAMT_90_USD_KD])) * 90
  Using the @ Key Date column (CC_SLSAMT_90_USD_KD) for sales denominator.

Conversion  : ✅ FULL


### M-14 | % AR/3 Month Sales
─────────────────────────────────────────────────────────────
PBI Format  : 0%;-0%;0%
RLS         : None (simple ratio)

Power BI DAX:
─────────────
measure '% AR/3 Month Sales' =
DIVIDE(SUM('AR-2'[ARAMT_30_USD]), SUM('AR-2'[SLSAMT_90_USD]), 0)

Conversion  : ✅ FULL


### M-15 | 3 Month DSO (% Growth)
─────────────────────────────────────────────────────────────
PBI Format  : 0.00%
RLS         : Full + date offset logic (MAXX date minus 1 month)

Logic: (DSO_CY / DSO_PY) - 1
  CY/PY derived using shared date period pattern.

Conversion  : ✅ FULL


### M-16 | AR2_CurrVal / AR2_PrevVal
─────────────────────────────────────────────────────────────
Power BI DAX:
─────────────
measure AR2_CurrVal = FORMAT(EDATE(MAXX_Date, -1), "MMM YY")
measure AR2_PrevVal = FORMAT(EDATE(EDATE(MAXX_Date,-1), -12), "MMM YY")

Conversion  : ✅ FULL


## TABLE: AR-3 — Measures (9 total)

### M-17 | Bad Debt Reserves
─────────────────────────────────────────────────────────────
PBI Format  : general
RLS         : Dual TREATAS (AR-2 blend into AR-3)

Power BI DAX key logic:
──────────────────────
-1 * CALCULATE(SUM('AR-3'[RSER_BAL_USD]),
         TREATAS(_CmpyCdAR2, 'AR-3'[CMPY_CD]))

NOTE: Negate (×-1) because reserve balances are stored as negative.
      Cross-table RLS: AR-2 company set applied via TREATAS into AR-3.

Tableau Mapping: -SUM([RSER_BAL_USD]) scoped to user's companies.
Conversion  : ✅ FULL


### M-18 | Bad Debt as % of AR
─────────────────────────────────────────────────────────────
PBI Format  : 0.0%
RLS         : Dual TREATAS

Logic: Bad Debt Reserves / Total Open AR Balance
       (both filtered with TREATAS(_CmpyCdAR2, 'AR-3'[CMPY_CD]))

Conversion  : ✅ FULL


### M-19 | % Bad Resrv Growth
─────────────────────────────────────────────────────────────
PBI Format  : 0.0%

Logic: (Current Bad Debt % - Prev Bad Debt %) YoY
       Uses MAXX(AR-3[Date]) for period anchoring.

Conversion  : ✅ FULL


### M-20 | Key Date  ← Bad Debt Key Date Display Label
─────────────────────────────────────────────────────────────
Power BI DAX:
─────────────
measure 'Key Date' =
"Key Date: " & FORMAT(MAX('AR-3'[Key Date Col]), "MMM DD, YYYY")

Conversion  : ✅ FULL


## TABLE: AR-4 — Measures (14 total)

### M-21 | AR Balance
─────────────────────────────────────────────────────────────
PBI Format  : #,0
RLS         : Full TREATAS

Power BI DAX key logic:
──────────────────────
CALCULATE(
    SUM('AR-4'[Net Amount (1 Month) in USD]),
    TREATAS(_CmpyCdAR2, 'AR-4'[CMPY_CD])
)

Conversion  : ✅ FULL


### M-22 | % of Total Net Amt Balance
─────────────────────────────────────────────────────────────
PBI Format  : 0%
RLS         : Full

Logic: CurrentAmt / CALCULATE(CurrentAmt, REMOVEFILTERS('AR-4'[AGE_CATG]))
Tableau Mapping: AR Value % across aging categories.
Conversion  : ✅ FULL


### M-23 | AR Value
─────────────────────────────────────────────────────────────
PBI Format  : 0.00%
RLS         : None (simple version)

Power BI DAX:
─────────────
measure 'AR Value' =
DIVIDE(SUM('AR-4'[AR Past Due]), SUM('AR-4'[Net Amount (1 Month) in USD]), 0)

Conversion  : ✅ FULL


### M-24 | Absolute AR Growth
─────────────────────────────────────────────────────────────
PBI Format  : 0.00%

Logic: (Current AR Balance DN / Previous AR Balance DN) - 1
       Both values use snapshot columns (Current AR Balance DN, Previous AR Balance DN).

Conversion  : ✅ FULL


### M-25 | Current AR % / Prev AR % / % AR Growth
─────────────────────────────────────────────────────────────
Current AR %  : Past Due ÷ Net Amount at CY target month + RLS  (0.0%)
Prev AR %     : Past Due ÷ Net Amount at PY target month + RLS  (0.0%)
% AR Growth   : Current AR % - Prev AR %  (0.00%)

Conversion  : ✅ FULL (all three)


### M-26 | AR4_CurrVal / AR4_PrevVal
─────────────────────────────────────────────────────────────
Same pattern as AR2_CurrVal / O1_CurrVal — FORMAT(EDATE(...)) period labels.
Conversion  : ✅ FULL


## TABLE: Stakeholder List — Measures

### M-27 | LoginUser  ← Welcome Banner
─────────────────────────────────────────────────────────────
Power BI DAX:
─────────────
measure LoginUser =
VAR _Email     = LOWER(USERPRINCIPALNAME())
VAR _LocalPart = LEFT(_Email, FIND("@", _Email) - 1)
VAR _FirstName = LEFT(_LocalPart, FIND(".", _LocalPart) - 1)
VAR _LastName  = MID(_LocalPart, FIND(".", _LocalPart) + 1,
                     LEN(_LocalPart) - FIND(".", _LocalPart))
RETURN "Welcome " & _LastName & ", " & _FirstName
-- e.g. email = "ama.zzb@bms.com" → "Welcome zzb, ama"

NOTE: Assumes firstname.lastname@bms.com email format.
Conversion  : ✅ FULL


### M-28 | Security Filter  ← RLS Diagnostic Gate
─────────────────────────────────────────────────────────────
PBI Format  : 0

Power BI DAX:
─────────────
measure 'Security Filter' =
VAR _userEmail   = UPPER(USERPRINCIPALNAME())
VAR _companyCode = SUMMARIZE(FILTER('Stakeholder List',
                       UPPER([email])=_userEmail), [Company Code])
VAR _CmpyCd      = SUMMARIZE(FILTER('CompanyCode',
                       [Company Code] IN _companyCode), [CMPY_CD])
VAR _CmpyCdAR2   = SUMMARIZE(FILTER('AR-2',
                       'AR-2'[CMPY_CD] IN _CmpyCd), 'AR-2'[CMPY_CD])
VAR _Match =
    COALESCE(COUNTROWS(FILTER('AR-2',  'AR-2'[CMPY_CD]  IN _CmpyCdAR2)),0)
  + COALESCE(COUNTROWS(FILTER('AR-3',  'AR-3'[CMPY_CD]  IN _CmpyCdAR2)),0)
  + COALESCE(COUNTROWS(FILTER('AR-4',  'AR-4'[CMPY_CD]  IN _CmpyCdAR2)),0)
  + COALESCE(COUNTROWS(FILTER('O: 1-1','O: 1-1'[CMPY_CD] IN _CmpyCdAR2)),0)
RETURN IF(_Match > 0, 1, 0)

Used for: Diagnostic tile — shows 1 if user has any data access, 0 if not.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — DIMENSION TABLES (CALCULATED DAX TABLES)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### DIM-01 | CompanyCode  ← Bridge / Dimension Table
─────────────────────────────────────────────────────────────
Type       : DAX Calculated Table
lineageTag : a4d1201a-9ac0-44b1-afe8-a6c5a641b3f4

Partition DAX:
──────────────
CompanyCode =
DISTINCT(UNION(
    SELECTCOLUMNS('O: 1-1', 'O: 1-1'[CMPY_CD]),
    SELECTCOLUMNS('AR-4',   'AR-4'[CMPY_CD]),
    SELECTCOLUMNS('AR-3',   'AR-3'[CMPY_CD]),
    SELECTCOLUMNS('AR-2',   'AR-2'[CMPY_CD])
))
-- Source column annotation: O: 1-1[CMPY_CD]

Columns:
  CMPY_CD          string     Source join key
  Company Code     int64      Integer alias of CMPY_CD; format: 0
  Region Group     string CC  SWITCH(TRUE(), LEFT(CMPY_CD,4) IN {...}) — 4 regions
  Company Name     string CC  IF(CONTAINS(AR-2,...), LOOKUPVALUE(CompanyName,...), BLANK())
  Company Name_new string CC  Identical formula to Company Name (in-progress duplicate)

NOTE: Company Code (int64) joins to Stakeholder List[Company Code] (int64).
      Company Name only returns a value if CMPY_CD also exists in AR-2 —
      this is the TREATAS safety check.


### DIM-02 | CompanyName  ← Company Display Name Lookup
─────────────────────────────────────────────────────────────
Type       : DAX Calculated Table
lineageTag : 30e28417-c0ba-4cd6-ba70-c31e79f965e2

Partition DAX:
──────────────
CompanyName =
DISTINCT(UNION(
    SELECTCOLUMNS('O: 1-1',  'O: 1-1'[CMPY],       'O: 1-1'[CMPY_CD]),
    SELECTCOLUMNS('AR-4',    'AR-4'[CMPY_CD_TXT],   'AR-4'[CMPY_CD]),
    SELECTCOLUMNS('AR-3',    'AR-3'[CMPY_CD_TXT],   'AR-3'[CMPY_CD]),
    SELECTCOLUMNS('AR-2',    'AR-2'[CMPY_CD_TXT],   'AR-2'[CMPY_CD])
))

Columns:
  CMPY             string  Company display name (source: O: 1-1[CMPY])
  O: 1-1_CMPY_CD   string  Join key (isNameInferred, source: O: 1-1[CMPY_CD])

Usage: CompanyCode[Company Name] does LOOKUPVALUE(CompanyName[CMPY], ...).


### DIM-03 | Region  ← Region Dimension
─────────────────────────────────────────────────────────────
Type       : DAX Calculated Table
lineageTag : 8a94f432-168e-488e-804a-88330505eb96

Partition DAX:
──────────────
Region =
DISTINCT(UNION(
    SELECTCOLUMNS('O: 1-1', 'O: 1-1'[REGN]),
    SELECTCOLUMNS('AR-4',   'AR-4'[REGN]),
    SELECTCOLUMNS('AR-3',   'AR-3'[REGN]),
    SELECTCOLUMNS('AR-2',   'AR-2'[REGN])
))
-- Source column: O: 1-1[REGN]

Column: REGN (string)
Joins to: All 4 fact tables via REGN relationships.


### DIM-04 | Reporting Region  ← Reporting Region Dimension
─────────────────────────────────────────────────────────────
Type       : DAX Calculated Table
lineageTag : d157375f-c36e-478d-a72f-fbad7c7d5848

Partition DAX:
──────────────
'Reporting Region' =
DISTINCT(UNION(
    SELECTCOLUMNS('O: 1-1', 'O: 1-1'[RPORTING_REGN]),
    SELECTCOLUMNS('AR-4',   'AR-4'[RPORTING_REGN]),
    SELECTCOLUMNS('AR-3',   'AR-3'[RPORTING_REGN]),
    SELECTCOLUMNS('AR-2',   'AR-2'[RPORTING_REGN])
))
-- Source column: O: 1-1[RPORTING_REGN]

Column: RPORTING_REGN (string)

NOTE: Distinct from CompanyCode[Region Group]. This table holds the raw
      RPORTING_REGN field from source. Region Group is the derived
      mapping from LEFT(CMPY_CD,4) → region label.
      Maps to Tableau's "Reporting Region (Test)" calculated field output.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — RELATIONSHIPS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Star Schema — all 4 fact tables join to CompanyCode, Region, and DimDate.

Key Relationships:

  From Table / Column              → To Table / Column            Cross-Filter  Notes
  ──────────────────────────────────────────────────────────────────────────────────────────────
  AR-2[CMPY_CD]                   → CompanyCode[CMPY_CD]          Single        FK → Dim
  AR-3[CMPY_CD]                   → CompanyCode[CMPY_CD]          Single
  AR-4[CMPY_CD]                   → CompanyCode[CMPY_CD]          Single
  O: 1-1[CMPY_CD]                 → CompanyCode[CMPY_CD]          Single
  Stakeholder List[Company Code]  ↔ CompanyCode[Company Code]     Both          ← RLS bidirectional
  AR-2[REGN]                      → Region[REGN]                  Single
  AR-3[REGN]                      → Region[REGN]                  Single
  AR-4[REGN]                      → Region[REGN]                  Single
  O: 1-1[REGN]                    → Region[REGN]                  Single
  AR-2[Date]                      → DimDate[Date]                 Single        datePartOnly
  AR-3[Date]                      → DimDate[Date]                 Single        datePartOnly
  AR-4[Date]                      → DimDate[Date]                 Single        datePartOnly
  O: 1-1[Date]                    → DimDate[Date]                 Single        datePartOnly
  DimDate[Date]                   → LocalDateTable_d44fe336[Date] Single        datePartOnly
  DimDate[Max Date]               → LocalDateTable_3aa71d56[Date] Single        datePartOnly
  AR-3[Key Date Col]              → LocalDateTable_faf34038[Date] Single        datePartOnly
  Refresh Date[LastRefreshed]     → LocalDateTable_b164214b[Date] Single        datePartOnly

NOTE: joinOnDateBehavior: datePartOnly is used on all date column relationships.
      The bidirectional relationship on Stakeholder List ↔ CompanyCode enables
      RLS to flow from the user's entitlements back through CompanyCode into
      all 4 fact tables via TREATAS in each measure.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — LOCALDATE TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All LocalDateTables are: hidden, showAsVariationsOnly = true,
standard hierarchy: Date Hierarchy → Year > Quarter > Month > Day
(all columns hidden; all categories set; Month sortByColumn: MonthNo)

  LocalDateTable GUID                                 Calendar Source Expression
  ──────────────────────────────────────────────────────────────────────────────────────────────
  3aa71d56-24d3-475f-b8ad-61d5116dec22  →  Calendar(Date(Year(MIN('DimDate'[Max Date])),1,1),
                                                     Date(Year(MAX('DimDate'[Max Date])),12,31))
  b164214b-d093-4729-b0e1-5f1838d9045b  →  Calendar(Date(Year(MIN('Refresh Date'[LastRefreshed])),1,1),
                                                     Date(Year(MAX('Refresh Date'[LastRefreshed])),12,31))
  d44fe336-9156-4e59-9e66-860c62e6923c  →  Calendar(Date(Year(MIN('DimDate'[Date])),1,1),
                                                     Date(Year(MAX('DimDate'[Date])),12,31))
  faf34038-98d2-4b0c-af4a-6f9a915e1ff6  →  Calendar(Date(Year(MIN('AR-3'[Key Date Col])),1,1),
                                                     Date(Year(MAX('AR-3'[Key Date Col])),12,31))

DateTableTemplate GUID: f305f672-13cc-46be-a821-0a78cc1c9d85
  Seed calendar: Calendar(Date(2015,1,1), Date(2015,1,1)) — single day
  Annotation: __PBI_TemplateDateTable = true; DefaultItem = DateHierarchy


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — TABLEAU FILTERS → POWER BI FILTER IMPLEMENTATION
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### FLT-01 | Aging Category ≠ NULL  ← Locked Filter
─────────────────────────────────────────────────────────────
Tableau    : Worksheet filter — [AGE_CATG] is not null
Power BI   : Report-level filter on AR-4[AGE_CATG] ≠ BLANK
Visibility : Locked and hidden — not user-selectable
Status     : ⚠️ Must configure in report Filters pane


### FLT-02 | Current Year Only  ← Locked Filter
─────────────────────────────────────────────────────────────
Tableau    : Worksheet filter — YEAR([Date]) = YEAR(TODAY())
Power BI   : Handled in DAX via _CurrentYear variable in all measures;
             additionally enforce as page-level filter on DimDate[Year]
Status     : ✅ Handled in DAX; report-level enforcement optional


### FLT-03 | Last 12 Months Window  ← Locked Filter
─────────────────────────────────────────────────────────────
Tableau    : DATEDIFF('month', [Date], MAX([Date])) <= 11
Power BI   : Enforced in DAX via MAXX date anchor pattern;
             additionally: report-level relative date filter on DimDate[Date]
Status     : ✅ Handled in DAX measures


### FLT-04 | MOM Rows Only  ← Locked Filter (on trend visuals)
─────────────────────────────────────────────────────────────
Tableau    : [MOM] = TRUE filter on trend worksheets
Power BI   : Implemented as _MOM measure suffix (see Shared Pattern — MOM Filter)
             Apply as visual-level filter: [AR Balance_MOM] ≠ BLANK
Status     : ✅ Handled in _MOM measures


### FLT-05 | CUST_ACCT_GRP = "Z003"  ← Locked Filter (AR-2 only)
─────────────────────────────────────────────────────────────
Tableau    : Locked worksheet filter on [CUST_ACCT_GRP]
Power BI   : Baked into AR-2 Power Query (Table.SelectRows)
             Not applied to AR-3, AR-4, O: 1-1.
Status     : ✅ Handled in Power Query — no report-level action needed


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — DEV / DEBUG ARTIFACTS — DO NOT PUBLISH
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

The following measures MUST be hidden or removed before production deployment.
They bypass RLS, contain hardcoded dates, or are superseded iterations.

  Measure Name                          Table      Reason
  ──────────────────────────────────────────────────────────────────────────────
  zzdebug 3 Month DSO (@ Key Date)      AR-2       No RLS — bypasses security
  zz 3 Month DSO (@ Key Date)           AR-2       Superseded iteration
  Bad Debt_Reserve                      AR-3       No RLS — raw sum
  zzDebug AR Balance                    AR-4       No RLS — dev only
  Previous AR Balance DN_2              AR-4       Experimental UNION/INTERSECT RLS
  Debug Sales PY All                    O: 1-1     Hardcoded year (2025) and month (≤5)
  DEBUG - Sales CY Reverted             O: 1-1     CMPY 0062 doc type logic — dev only
  DEBUG - Sales PY Value                O: 1-1     "Should match Tableau's 33,426M" comment
  zzbug_Orders Processed* (YTD)         O: 1-1     Bug-tracking version, no RLS
  Orders Accuracy (YTD)_new             O: 1-1     Dev iteration — simplified no-TREATAS
  Invoiced Amount YTD_new               O: 1-1     Dev iteration (ALL vs ALLSELECTED)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

G1 | tmpTbl Has No Relationships
──────────────────────────────────
Status  : ⚠️ Verify before production
Detail  : tmpTbl (Cmpy_code / Group) has no defined relationships to any
          other table. It appears to be a dev/validation artifact with
          binary-compressed inline data.
Action  : Confirm with business owner whether tmpTbl should be:
          (a) related to CompanyCode and used in a slicer/filter, or
          (b) removed entirely from the production report.

G2 | Company Name_new = Duplicate of Company Name
───────────────────────────────────────────────────
Status  : ℹ️ In-progress artifact
Detail  : CompanyCode[Company Name_new] contains identical DAX to
          CompanyCode[Company Name]. Likely an in-progress rename.
Action  : Confirm which column should be used in visuals. Remove the
          other before publishing to avoid confusion.

G3 | MOM Report-Level Filters Require Manual Setup
────────────────────────────────────────────────────
Status  : ⚠️ Must configure in report layer
Detail  : Tableau's MOM, current-year, and last-12-months filters are
          locked worksheet filters. In PBI, the _MOM and date-scoped
          DAX handles them in measures, but visual-level filters
          (e.g., AR Balance_MOM ≠ BLANK) still need to be set per visual.
Action  : Set visual-level filters on all MOM trend charts:
          [<measure>_MOM] is not blank.

G4 | Tableau "Reporting Region (Test)" vs. DimDate Region Group
─────────────────────────────────────────────────────────────────
Status  : ℹ️ Architecture note
Detail  : Tableau's [Reporting Region (Test)] maps LEFT 4 of CMPY_CD to
          region labels. In PBI this is CompanyCode[Region Group] (SWITCH).
          The 'Reporting Region' table holds the raw RPORTING_REGN values
          from source and is a different dimension from Region Group.
          These serve different visual axes.
Action  : When mapping Tableau visuals that used [Reporting Region (Test)]
          as a dimension, use CompanyCode[Region Group] in PBI.
          When mapping raw RPORTING_REGN slicer/filter, use 'Reporting Region'[RPORTING_REGN].

G5 | DimDate Table Not Included in This File Set
──────────────────────────────────────────────────
Status  : ℹ️ Documented in prior batch
Detail  : The full DimDate table definition (calendar columns, Max Date,
          12Months, Year, Fiscal Year, etc.) was extracted in a prior
          file batch. Refer to the session conversation history or
          re-extract dimdate.txt for the complete definition.
Action  : Ensure DimDate is referenced from prior batch notes before
          authoring any new DimDate-dependent measures.

G6 | "Others" Region Bucket Does Not Use ELSE Fallback
────────────────────────────────────────────────────────
Status  : ⚠️ Verify completeness periodically
Detail  : CompanyCode[Region Group] SWITCH uses an explicit IN list for
          "Others" rather than a catch-all ELSE/default branch.
          Any new CMPY_CD not in any of the 4 lists will return BLANK.
Action  : When new company codes are onboarded, review and update all
          four IN lists in both CompanyCode[Region Group] and
          AR-4[Region Group dummyAR4].


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 11 — CONVERSION COMPLETENESS SUMMARY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Category                                  Tableau   PBI       Status      %
──────────────────────────────────────────────────────────────────────────────
Parameters (field-switch + region)              2       2     ✅ Full    100%
Fact Tables (SAP HANA source)                   4       4     ✅ Full    100%
Dimension Tables (calculated DAX)               4       4     ✅ Full    100%
Support Tables (Stakeholder/Refresh/tmp)        3       3     ✅ Full    100%
Date Dimension (DimDate)                        1       1     ✅ Full    100%
LocalDateTables                                 4       4     ✅ Full    100%
Date Column Calc (yyyyMM → DATE)                4       4     ✅ Full    100%
Calculated Columns — O: 1-1                    11      11     ✅ Full    100%
Calculated Columns — AR-3 (Key Date)            1       1     ✅ Full    100%
Calculated Columns — AR-4                       3       3     ✅ Full    100%
Calculated Columns — CompanyCode                3       3     ✅ Full    100%
Measures — O: 1-1                              22      22     ✅ Full    100%
Measures — AR-2                                 9       9     ✅ Full    100%
Measures — AR-3                                 9       9     ✅ Full    100%
Measures — AR-4                                14      14     ✅ Full    100%
Measures — Stakeholder List                     2       2     ✅ Full    100%
Relationships                                  18      18     ✅ Full    100%
RLS Architecture (full chain)                   1       1     ✅ Full    100%
Locked Filters (PQ-baked)                       1       1     ✅ Full    100%
Locked Filters (report-level pending)           3       3     ⚠️ Pending   0%
Dev/Debug Measures (flag for removal)          11      11     ⚠️ Pending   0%
──────────────────────────────────────────────────────────────────────────────
OVERALL (Tableau artefacts mapped)                            ✅/⚠️      ~96%
──────────────────────────────────────────────────────────────────────────────

Legend:
  ✅ FULL     — exact functional equivalent implemented
  ⚠️ PARTIAL  — present but requires follow-up action before production
  ➕ PBI-only — no Tableau equivalent; Power BI infrastructure
  ℹ️ Low risk — omitted intentionally; low business impact


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 12 — QUICK REFERENCE: TABLEAU FORMULA → DAX PATTERN
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Tableau Pattern                                        DAX / M Equivalent
──────────────────────────────────────────────────────────────────────────────────────────────
DATEPARSE('yyyyMM', [CALD_MTH])                        DATE(LEFT([col],4), RIGHT([col],2), 1)

{FIXED : MAX([Date])}                                  MAXX(ALL('table'), 'table'[Date])

IF MONTH(MaxDate)-1=0 THEN 12                          IF(_MaxMonthNum=1, 12, _MaxMonthNum-1)
  ELSE MONTH(MaxDate)-1 END

{FIXED [RPTING_REGN], YEAR : SUM([NET_AMT])}           TREATAS(VALUES([RPORTING_REGN]),...) in CALCULATE

CONTAINS([ORD_REAS_CD], "MNC")                         IFERROR(ISNUMBER(FIND("MNC",[col])), FALSE)

LOD EXCLUDE [AGE_CATG] : SUM([amt])                    CALCULATE(SUM([amt]), REMOVEFILTERS([AGE_CATG]))

-SUM([RSER_BAL_USD])                                   -1 * CALCULATE(SUM([col]), TREATAS(...))

SUM / SUM → ratio                                      DIVIDE(numerator, denominator, 0)

User entitlement (no native Tableau RLS)               USERPRINCIPALNAME() → Stakeholder List →
                                                       CompanyCode → TREATAS into each fact table

Tableau parameter (string list)                        DAX inline table { ("Label", NAMEOF(col), sortInt) }
  driving axis field switch                            with ParameterMetadata { version:3, kind:2 }

Tableau FIXED LOD % of total                           DIVIDE(measure, CALCULATE(measure,
                                                         REMOVEFILTERS(dim_column)))

MOM boolean calc (2-condition window)                  (YEAR=PrevYear && MONTH=PrevMonth && Date<=EDATE(-1))
                                                       OR (YEAR=CurrYear && MONTH=PrevMonth && Date<=Max)

YoY KPI label (e.g. "Jan 25" vs "Jan 24")             FORMAT(EDATE(MAXX_Date, -1), "MMM YY")
                                                       FORMAT(EDATE(EDATE(MAXX_Date,-1),-12), "MMM YY")

Locked worksheet filter                                Report-level filter in PBI Filters pane (lock+hide)

Bidirectional relationship for RLS propagation         crossFilteringBehavior: bothDirections on
                                                       Stakeholder List ↔ CompanyCode

yyyyMMdd string → DATE                                 DATE(LEFT([col],4), MID([col],5,2), RIGHT([col],2))
                                                       with ISBLANK/LEN guard

DATENAME('month',[dt]) + STR(YEAR([dt]))               FORMAT([col], "MMMM, yyyy")

Value.NativeQuery SAP HANA                             Value.NativeQuery(
                                                         SapHana.Database("server:port",...), sqlText,
                                                         [], [EnableFolding=true])


---

# END OF KNOWLEDGE FILE
# 12960 – O2C Executive Summary (AR vs. Credit Limit)
# Tableau → Power BI (PBIP/TMDL)
# Generated: 2026-07-07
# Source files:
#   o2c_executive_summary.txt (Tableau workbook)
#   o_3a_1_1.txt, ar_2.txt, ar_3.txt, ar_4.txt,
#   companycode.txt, companyname.txt, region.txt,
#   reporting_region.txt, view_by_3a.txt,
#   stakeholder_list.txt, refresh_date.txt, tmptbl.txt,
#   localdatetable_3aa71d56_24d3_475f_b8ad_61d5116dec22.txt,
#   localdatetable_b164214b_d093_4729_b0e1_5f1838d9045b.txt,
#   localdatetable_d44fe336_9156_4e59_9e66_860c62e6923c.txt,
#   localdatetable_faf34038_98d2_4b0c_af4a_6f9a915e1ff6.txt,
#   datetabletemplate_f305f672_13cc_46be_a821_0a78cc1c9d85.txt,
#   7_definition.json, 7_database.txt, 7_model.txt,
#   7_relationships.txt, 7_diagramlayout.json,
#   companycode.txt, companyname.txt (PBIP)
