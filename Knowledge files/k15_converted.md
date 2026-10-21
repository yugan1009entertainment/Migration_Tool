# 📘 CONVERSION KNOWLEDGE FILE
# K15 – CA-2: Cash Clearing
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-23
# Source: K15 1A (Tableau TWB) · K15 2B (Power BI PBIP)
# Domain: BMS Global Finance – Order to Cash (OTC)
# Scope: Accounts Receivable / Cash Clearing

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PARAM-01 | Currency (Parameter 1)
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 1]
Tableau Caption       : Currency
Tableau Type          : string list
Tableau Default       : "USD"
Tableau Members       : "USD", "Local"

Power BI Table Name   : Currency
Power BI Type         : Calculated DATATABLE (disconnected slicer)
Power BI Values       : USD, Local

Usage in model:
  - Drives Amount in USD/LCL measure:
    Tableau: CASE [Parameter 1] WHEN 'Local' THEN [AMT_LCL] WHEN 'USD' THEN [AMT_USD] END
    PBI: SELECTEDVALUE('Currency'[Currency]) → SWITCH → SUM([Amount in USD]) or SUM([Amount in LCL])
  - Drives Displayed Currency measure:
    Tableau: CASE [Parameter 1] WHEN 'USD' THEN 'USD' ELSE 'Local' END
    PBI: IF(SELECTEDVALUE('Currency'[Currency]) = "USD", "USD", "Local")

Conversion Status: ✅ FULL


### PARAM-02 | ViewBy (Parameter 2)
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 2]
Tableau Caption       : ViewBy
Tableau Type          : string list
Tableau Default       : "Company"
Tableau Members       : Company, Customer, Debit/Credit Indicator, Doc. Type,
                        GL Account, GL Account Group, Posting Key

Power BI Table Name   : ViewBy Parameter
Power BI Type         : Calculated table (NAMEOF field parameter or DATATABLE)

Power BI Values & Mapped Fields:
────────────────────────────────
| Label                    | Mapped Column                            |
|--------------------------|------------------------------------------|
| Company                  | CVQ_FI_AR_GOTC_CUST_CLRD_CASH[company]   |
| Customer                 | CVQ_FI_AR_GOTC_CUST_CLRD_CASH[Customer_cal] |
| Debit/Credit Indicator   | CVQ_FI_AR_GOTC_CUST_CLRD_CASH[Debit/Credit Indicator] |
| Doc. Type                | CVQ_FI_AR_GOTC_CUST_CLRD_CASH[Doc Type]  |
| GL Account               | CVQ_FI_AR_GOTC_CUST_CLRD_CASH[GL_account]|
| GL Account Group         | CVQ_FI_AR_GOTC_CUST_CLRD_CASH[GL_account_group] |
| Posting Key              | CVQ_FI_AR_GOTC_CUST_CLRD_CASH[posting_key] |

Usage in model:
  - Tableau: CASE [Parameter 2] WHEN "Company" THEN [Company] ...
    PBI: SWITCH(SELECTEDVALUE('ViewBy Parameter'[Parameter Fields]), ...)
  - Tooltip Dimension measure uses SWITCH/ISINSCOPE pattern returning
    "[Label]: [Selected Value]" for each dimension.

Conversion Status: ✅ FULL


### PARAM-03 | ViewBy2 (Parameter 3)
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 3]
Tableau Caption       : ViewBy2
Tableau Type          : string list
Tableau Default       : "AR Debit Amount" (alias: Net AR Impact)
Tableau Members       : "AR Debit Amount" (Net AR Impact),
                        "Cash Cleared Amount" (Net Cash Impact)

Power BI Table Name   : ViewBy2 Parameter
Power BI Type         : Calculated table

Power BI Values:
────────────────
| Label           | Mapped Measure                              |
|-----------------|---------------------------------------------|
| Net AR Impact   | CVQ_FI_AR_GOTC_CUST_CLRD_CASH[AR Debit Amount] |
| Net Cash Impact | CVQ_FI_AR_GOTC_CUST_CLRD_CASH[Cash Cleared Amount] |

Usage in model:
  - Drives trend chart title:
    "Monthly Trend of " & IF(SELECTEDVALUE(order) = 0, "Net AR Impact", "Net Cash Impact")
  - Tableau: CASE [Parameter 3] WHEN 'Cash Cleared Amount' THEN... WHEN 'AR Debit Amount' THEN...

Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: CVQ_FI_AR_GOTC_CUST_CLRD_CASH (Fact Table)

### CC-01 | Posting Date (Date Conversion)
─────────────────────────────────────────────────────────────
Tableau Expression: DATE(DATETRUNC('day', DATEPARSE("yyyyMMdd", [POST_DT])))
Power BI DAX: Calculated column parsing raw yyyyMMdd string to dateTime.
Pattern     : Tableau DATEPARSE("yyyyMMdd") → DAX DATE(VALUE(LEFT/MID/RIGHT))
NOTE        : Has date variation to LocalDateTable_56b40e09.
Conversion  : ✅ FULL

### CC-02 | Posting_date (Alternate parsed date)
─────────────────────────────────────────────────────────────
Alternate date column with variation to LocalDateTable_b33fc482.
Conversion  : ✅ FULL

### CC-03 | company (Display Label)
─────────────────────────────────────────────────────────────
Tableau Expression: [CMPY_CD] + ' ' + [CMPY_CD_TXT]
Power BI DAX: [Company Code] & " " & [Company Code Description]
Conversion  : ✅ FULL

### CC-04 | Customer_cal / Customer (Display Labels)
─────────────────────────────────────────────────────────────
Tableau Expression: [CUST] + ' ' + [CUST_NM1]
Power BI DAX: [Customer] & " " & [Customer Name1]
Conversion  : ✅ FULL

### CC-05 | GL_account (Display Label)
─────────────────────────────────────────────────────────────
Tableau: [GL_AC] + ' ' + [GL_AC_TXT]
Power BI: [GL Account] & " " & [GL Account Description]
Conversion  : ✅ FULL

### CC-06 | GL_account_group (Display Label)
─────────────────────────────────────────────────────────────
Tableau: [GL_AC_GRP] + ' ' + [GL_AC_GRP_TXT]
Power BI: [GL Account Group] & " " & [GL Account Group Description]
Conversion  : ✅ FULL

### CC-07 | Doc Type (Display Label)
─────────────────────────────────────────────────────────────
Tableau: [DOC_TYPE] + ' ' + [DOC_TYPE_TXT]
Power BI: [Document Type] & " " & [Document Type Description]
Conversion  : ✅ FULL

### CC-08 | posting_key (Display Label)
─────────────────────────────────────────────────────────────
Tableau: [PSTNG_KY] + ' ' + [PSTNG_KY_TXT]
Power BI: [Posting Key] & " " & [Posting Key Description]
Conversion  : ✅ FULL

### CC-09 | Cust. Acct. Group (Display Label)
─────────────────────────────────────────────────────────────
Tableau: [CUST_ACCT_GRP] + ' - ' + [CUST_ACCT_GRP_TXT]
Power BI: [Customer Account Group] & " - " & [Customer Account Group Name]
⚠️  NOTE: Uses " - " (space-dash-space) separator, unlike other display
    labels that use " " (space). Preserved exactly.
Conversion  : ✅ FULL

### CC-10 | Customer Group (Display Label)
─────────────────────────────────────────────────────────────
Tableau: [CUST_GRP] + ' - ' + [CUST_GRP_TXT]
Power BI: [Customer Group] & " - " & [Customer Group Description]
Conversion  : ✅ FULL

### CC-11 | Debit/Credit Indicator
─────────────────────────────────────────────────────────────
Power BI: Debit/Credit indicator text column.
Conversion  : ✅ FULL

### CC-12 | AR Debit Amount (Pre-calculated Column)
─────────────────────────────────────────────────────────────
Tableau Expression:
CASE [GL_AC_GRP] WHEN 'RECO' THEN [Amount] ELSE 0 END
(where Amount = signed value based on Debit/Credit indicator)

Power BI: Pre-calculated AR debit column for RECO GL group.
Conversion  : ✅ FULL

### CC-13 | Cash Cleared Amount (Pre-calculated Column)
─────────────────────────────────────────────────────────────
Tableau Expression:
CASE [GL_AC_GRP] WHEN 'FUND' THEN [Amount] ELSE 0 END

Power BI: Pre-calculated cash clearing column for FUND GL group.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### M-01 | Amount in USD/LCL (Currency-Switched)
─────────────────────────────────────────────────────────────
Tableau: CASE [Parameter 1] WHEN 'Local' THEN [AMT_LCL] WHEN 'USD' THEN [AMT_USD] END
Power BI: SELECTEDVALUE('Currency'[Currency]) → SWITCH → SUM([Amount in USD]) or SUM([Amount in LCL])
Pattern  : Currency switching via SELECTEDVALUE (same as K13).
Conversion  : ✅ FULL

### M-02 | Amount (Signed Base Amount)
─────────────────────────────────────────────────────────────
Tableau: CASE [DBT_CR_IND_TXT] WHEN 'Debit' THEN [USD/LCL] WHEN 'Credit' THEN -1*[USD/LCL] END
Power BI: Debit positive / Credit negative sign convention.
Conversion  : ✅ FULL

### M-03 | AR Debit Amount (Measure)
─────────────────────────────────────────────────────────────
Net AR impact from RECO GL account group entries.
Conversion  : ✅ FULL

### M-04 | Cash Cleared Amount (Measure)
─────────────────────────────────────────────────────────────
Net cash cleared from FUND GL account group entries.
Conversion  : ✅ FULL

### M-05 | # Acct Docs
─────────────────────────────────────────────────────────────
Tableau: {INCLUDE [Company],[ACCT_DOC_NO],YEAR([Posting Date]): COUNTD([ACCT_DOC_NO])}
Power BI: DISTINCTCOUNT([Accounting Document Number])
Pattern  : Tableau LOD INCLUDE COUNTD → DAX DISTINCTCOUNT
Conversion  : ✅ FULL

### M-06 | # Acct Doc Line Item
─────────────────────────────────────────────────────────────
Tableau: {INCLUDE [Company],[ACCT_DOC_LN],[ACCT_DOC_NO],YEAR([Posting Date]): COUNTD([ACCT_DOC_LN])}
Power BI: DISTINCTCOUNT([Accounting Document Line Item])
Conversion  : ✅ FULL

### M-07 | Displayed Currency
─────────────────────────────────────────────────────────────
Tableau: CASE [Parameter 1] WHEN 'USD' THEN 'USD' ELSE 'Local' END
Power BI: IF(SELECTEDVALUE('Currency'[Currency]) = "USD", "USD", "Local")
Conversion  : ✅ FULL

### M-08 | Tooltip Dimension
─────────────────────────────────────────────────────────────
Power BI: SWITCH/ISINSCOPE pattern returning "[Label]: [Selected Value]"
          for each of the 7 ViewBy dimensions.
Type    : ➕ PBI-only enhancement — replaces Tableau tooltip field behavior.
Conversion  : ✅ FULL

### M-09 through M-22 | Additional Measures
─────────────────────────────────────────────────────────────
| M    | Measure Name                    | Description                              |
|------|---------------------------------|------------------------------------------|
| M-09 | AR Debit Amount matrix          | Matrix visual variant                    |
| M-10 | Cash Cleared Amount Matrix      | Matrix visual variant                    |
| M-11 | # Acct Docs_month               | Monthly document count                   |
| M-12 | customer_cnt / Customer count   | Distinct customer count                  |
| M-13 | Y Index                         | Row index for display ordering           |
| M-14 | Company Selected                | Returns selected company code            |
| M-15 | Show Message                    | Flag: show user message                  |
| M-16 | Show table                      | Flag: show data table visual             |
| M-17 | Message Text                    | Dynamic message text                     |
| M-18 | Has Data                        | Boolean: filtered data exists            |
| M-19 | show_month                      | Flag: show monthly breakdown             |
| M-20 | Tooltip Text / Tooltip info     | Dynamic tooltip strings                  |
| M-21 | Key_date_measure                | Returns key date value                   |
| M-22 | Refresh Date Measure            | Combines refresh_date_text + refresh date|
| M-23 | Selected Date Filter            | Returns current date filter selection    |
| M-24 | Show Visual Flag                | Controls visual visibility               |
| M-25 | minimum/maximum (Posting Date)  | MIN/MAX of Posting Date                  |
| M-26 | test (USERPRINCIPALNAME())      | Debug measure — returns current UPN      |

Conversion  : ✅ FULL (all)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU INITIAL TRANSFORMATIONS → POWER BI POWER QUERY (M)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | CVQ_FI_AR_GOTC_CUST_CLRD_CASH (Fact Table — SAP HANA)
─────────────────────────────────────────────────────────────
Tableau Source  : SAP HANA (federated, .hyper extract)
                  _SYS_BIC.QUERIES.CORPORATE_FUNCTIONS.GLOBAL_FINANCE.LEAPFROG::CVQ_FI_AR_GOTC_CUST_CLRD_CASH
PBI Connection  : SapHana.Database("pahpdb01.net.bms.com:30015", [Implementation = "2.0"])
PBI Method      : Value.NativeQuery (import mode)
PBI lineageTag  : (fact table)

Source Columns (28):
| SAP Column       | PBI Column                      | Type     |
|------------------|---------------------------------|----------|
| CUST_ACCT_GRP    | Customer Account Group          | string   |
| CUST_GRP         | Customer Group                  | string   |
| CUST_GRP_TXT     | Customer Group Description      | string   |
| CUST_NM1         | Customer Name1                  | string   |
| CMPY_CD_TXT      | Company Code Description        | string   |
| GL_AC            | GL Account                      | string   |
| PSTNG_KY         | Posting Key                     | string   |
| POST_DT          | Posting Date (raw yyyyMMdd)     | string   |
| ACCT_DOC_LN      | Accounting Document Line Item   | string   |
| REGN             | Region                          | string   |
| CMPY_CD          | Company Code                    | string   |
| ACCT_DOC_NO      | Accounting Document Number      | string   |
| YR               | Fiscal Year                     | string   |
| KEY_DT           | Key Date (raw yyyyMMdd)         | string   |
| CUNCY_KY         | Currency Key                    | string   |
| DBT_CR_IND_TXT   | Debit/Credit Indicator          | string   |
| GL_AC_TXT        | G/L Account Description         | string   |
| GL_AC_GRP        | G/L Account Group               | string   |
| GL_AC_GRP_TXT    | G/L Account Group Description   | string   |
| CUST             | Customer                        | string   |
| CUST_ACCT_GRP_TXT| Customer Account Group Name     | string   |
| DOC_TYPE         | Document Type                   | string   |
| CLNT             | Client                          | string   |
| DOC_TYPE_TXT     | Document Type Description       | string   |
| REV_DOC_NUM      | Reversal Document Number        | string   |
| REV_IND          | Reversal Indicator              | string   |
| PSTNG_KY_TXT     | Name of the Posting Key         | string   |
| CC_DATE_UPPER    | Cash Clearing Upper Date        | string   |
| PST_IND          | Past Month Indicator            | string   |
| AMT_USD          | Amount in USD                   | real     |
| AMT_LCL          | Amount in LCL                   | real     |

Approximate cardinalities: ~74 company codes, ~2,274 customers,
~6,733 accounting documents, ~945 posting dates, ~42 currencies,
~76 GL accounts, ~14 GL account groups, ~6 regions.


### PQ-02 | V_LEAP_SECURITY (Security Table — Oracle)
─────────────────────────────────────────────────────────────
Oracle Server   : uslvlbmsasp013.net.bms.com:1521/ihubp
Oracle Schema   : LEAPFROG
Columns         : BMSUID, LAST_NAME, FIRST_NAME, EMAIL_ADDRESS, APPROVED_BY,
                  APPROVED_DATE, START_DATE, END_DATE, CURRENT_STATUS,
                  APPROVER_IND, LAST_UPDATE_BY, LAST_UPDATE_DATE, APP_NAME,
                  COMPANY_CD, ROLE_NAME, COUNTRY
NOTE            : 4 date columns have LocalDateTable variations.


### PQ-03 | Dim_Company (Calculated Dimension)
─────────────────────────────────────────────────────────────
Type    : DAX Calculated table
Logic   : Union of unique Company Codes from both fact and security tables,
          filtered to exclude blanks/empty strings.
Column  : Company_code
Purpose : Bridge table for bidirectional security relationship.


### PQ-04 | Refresh Table (Utility)
─────────────────────────────────────────────────────────────
M Query : DateTimeZone.UtcNow() → MM/dd/yyyy hh:mm:ss tt
Measures: refresh_date_text = "Refresh Date:", refresh date = MAX([Refresh Table])


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — CONVERSION COMPLETENESS SCORECARD
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Category                         | Items | PBI Full | PBI Partial | Not Converted | % Complete |
|----------------------------------|-------|----------|-------------|---------------|------------|
| Parameters                       | 3     | 3        | 0           | 0             | 100%       |
| Calculated Columns (Fact)        | 13    | 13       | 0           | 0             | 100%       |
| Measures                         | 26+   | 26+      | 0           | 0             | 100%       |
| Power Query / Data Sources       | 4     | 4        | 0           | 0             | 100%       |
| Relationships                    | 7     | 7        | 0           | 0             | 100%       |
| RLS Security                     | 1     | 1        | 0           | 0             | 100%       |
| LocalDateTables                  | 6     | 6        | 0           | 0             | 100%       |
| TMDL Infrastructure              | 3     | 3        | 0           | 0             | 100%       |
| TOTAL                            | 63+   | 63+      | 0           | 0             | 🟢 100%    |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — CONVERSION PATTERN CHEAT SHEET
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PATTERN-01 | Currency Switching
─────────────────────────────────────────────────────────────
Tableau: CASE [Parameter 1] WHEN 'USD' THEN [AMT_USD] WHEN 'Local' THEN [AMT_LCL]
PBI: SELECTEDVALUE('Currency'[Currency]) → SWITCH → SUM(USD or LCL column)

PATTERN-02 | Debit/Credit Sign Convention
─────────────────────────────────────────────────────────────
Tableau: CASE [DBT_CR_IND_TXT] WHEN 'Debit' THEN [Amount] WHEN 'Credit' THEN -1*[Amount]
PBI: Same sign flip in DAX measure.

PATTERN-03 | GL Account Group Filtering
─────────────────────────────────────────────────────────────
Tableau: CASE [GL_AC_GRP] WHEN 'RECO' THEN [Amount] ELSE 0 END
PBI: Pre-calculated columns or CALCULATE with filter on GL_account_group.

PATTERN-04 | Dynamic ViewBy Dimension
─────────────────────────────────────────────────────────────
Tableau: CASE [Parameter 2] WHEN "Company" THEN [Company] ...
PBI: SWITCH(SELECTEDVALUE('ViewBy Parameter'[Parameter Fields]), ...)
     with ISINSCOPE pattern for tooltips.

PATTERN-05 | Date String Parsing
─────────────────────────────────────────────────────────────
Tableau: DATE(DATEPARSE("yyyyMMdd", [POST_DT]))
PBI: Calculated column DATE(VALUE(LEFT(4)), VALUE(MID(5,2)), VALUE(RIGHT(2)))

PATTERN-06 | LOD INCLUDE COUNTD → DISTINCTCOUNT
─────────────────────────────────────────────────────────────
Tableau: {INCLUDE [Dim1],[Dim2]: COUNTD([Key])}
PBI: DISTINCTCOUNT([Key]) — PBI visual context handles the INCLUDE dimensions.

PATTERN-07 | RLS Security via Bidirectional Relationship
─────────────────────────────────────────────────────────────
Tableau: UPPER(USERNAME()) = UPPER([BMSUID]) filter on data source
PBI: Bidirectional relationship (bothDirections) between fact and
     security tables on Company Code + Security groups role using
     USERPRINCIPALNAME().


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAP-01 | Compatibility Level 1601 vs. Standard 1604
  Severity : Low (informational)
  Detail   : K15 uses compatibility level 1601, while most other
             knowledge bases use 1604.
  Action   : Verify if 1601 is intentional or can be upgraded to 1604.

GAP-02 | SAP HANA + Oracle Dual Gateway Required
  Severity : High (infrastructure)
  Detail   : Both SAP HANA (pahpdb01.net.bms.com:30015) and Oracle
             (uslvlbmsasp013.net.bms.com:1521/ihubp) require on-premise
             Data Gateways.
  Action   : Configure gateway for both connections.

GAP-03 | Bidirectional RLS Relationship
  Severity : Medium
  Detail   : The Company Code relationship between fact and security
             tables uses bothDirections (many-to-many cross-filter).
             This can cause performance issues on large datasets and
             requires careful filter context management.
  Action   : Test RLS performance with production data volumes.
             Consider Dim_Company bridge table approach for optimization.

GAP-04 | Duplicate Posting Date Columns
  Severity : Low
  Detail   : Two parsed date columns exist: [Posting Date] and
             [Posting_date], each with its own LocalDateTable variation.
  Action   : Confirm whether both are needed or if one can be removed.

GAP-05 | Default Filters — Report-Layer Configuration
  Severity : Low (report-layer)
  Detail   : Tableau defaults: Past Month Indicator = "Past 6 Months",
             Region = ECC/LACC/Other. These must be configured as
             report-layer slicer defaults in PBI.
  Action   : Set default slicer values in PBI report canvas.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — FILE INVENTORY & RELATIONSHIPS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### Model Tables
─────────────────────────────────────────────────────────────
| Table                           | Type                    | Source              |
|---------------------------------|-------------------------|---------------------|
| CVQ_FI_AR_GOTC_CUST_CLRD_CASH  | Fact table              | SAP HANA            |
| V_LEAP_SECURITY                 | Security table          | Oracle              |
| Dim_Company                     | Calculated dimension    | DAX (bridge table)  |
| Currency                        | Parameter table         | DATATABLE           |
| ViewBy Parameter                | Parameter table         | Calculated table    |
| ViewBy2 Parameter               | Parameter table         | Calculated table    |
| Refresh Table                   | Utility                 | M Query (UTC now)   |
| DateTableTemplate               | Hidden, auto-generated  | PBI engine          |
| LocalDateTable (×6)             | Hidden, auto-generated  | PBI engine          |

### Relationships (7)
─────────────────────────────────────────────────────────────
| From Table → Column                         | To Table → Column               | Cross-Filter | Notes                |
|---------------------------------------------|----------------------------------|--------------|----------------------|
| CVQ_FI_AR...[Posting Date]                  | LocalDateTable_56b40e09.[Date]  | Single       | datePartOnly         |
| CVQ_FI_AR...[Posting_date]                  | LocalDateTable_b33fc482.[Date]  | Single       | datePartOnly         |
| CVQ_FI_AR...[Company Code]                  | V_LEAP_SECURITY.[COMPANY_CD]    | Both         | RLS many-to-many     |
| V_LEAP_SECURITY.[APPROVED_DATE]             | LocalDateTable_fcd7a49a.[Date]  | Single       | datePartOnly         |
| V_LEAP_SECURITY.[START_DATE]                | LocalDateTable_e84d411a.[Date]  | Single       | datePartOnly         |
| V_LEAP_SECURITY.[END_DATE]                  | LocalDateTable_14cc8952.[Date]  | Single       | datePartOnly         |
| V_LEAP_SECURITY.[LAST_UPDATE_DATE]          | LocalDateTable_2f50768f.[Date]  | Single       | datePartOnly         |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — SECURITY / ROW-LEVEL SECURITY (RLS)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### RLS-01 | Security Groups
─────────────────────────────────────────────────────────────
Tableau Mechanism : UPPER(USERNAME()) = UPPER([BMSUID]) filter on data source
Power BI Role     : Security groups
Power BI Mechanism: Bidirectional relationship between fact table and
                    V_LEAP_SECURITY on Company Code (bothDirections).
                    Users see only rows matching their company code access
                    in the LEAPFROG security table.
Debug Measure     : test (USERPRINCIPALNAME()) — returns current UPN.

⚠️  NOTE: Bidirectional many-to-many relationship enables RLS filtering
    in both directions but may impact query performance on large datasets.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — CROSS-TOOL COMPARISON
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Feature                 | Tableau (K15 1A)                              | Power BI (K15 2B)                                   |
|-------------------------|-----------------------------------------------|-----------------------------------------------------|
| Fact Source             | SAP HANA (direct + extract)                   | SAP HANA (import via Power Query)                   |
| Security Source         | Oracle/Tableau proxy                          | Oracle (Power Query)                                |
| RLS Mechanism           | USERNAME() = BMSUID filter                    | Bidirectional relationship + Security groups role   |
| Currency Toggle         | Parameter 1 (USD/Local)                       | Currency table slicer                               |
| ViewBy                  | Parameter 2 (7 options)                       | ViewBy Parameter calculated table                   |
| ViewBy2 (Trend)         | Parameter 3 (Net AR/Cash Impact)              | ViewBy2 Parameter calculated table                  |
| AR Debit Amount         | CASE on GL_AC_GRP = 'RECO'                    | DAX measure on GL_account_group                     |
| Cash Cleared Amount     | CASE on GL_AC_GRP = 'FUND'                    | DAX measure on GL_account_group                     |
| # Acct Docs             | LOD COUNTD                                    | DAX DISTINCTCOUNT                                   |
| Posting Date            | DATEPARSE yyyyMMdd                            | Calculated column, parsed date                      |
| Default Time Filter     | Past 6 Months                                 | Selected Date Filter measure                        |
| Refresh Tracking        | Extract update timestamp                      | Refresh Table (UTC timestamp)                       |
| Tooltip Dimension       | Tooltip fields on mark                        | Tooltip Dimension measure (SWITCH/ISINSCOPE)        |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 11 — GLOSSARY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Term                              | Definition                                                                |
|-----------------------------------|---------------------------------------------------------------------------|
| AR                                | Accounts Receivable                                                       |
| Cash Clearing                     | Matching and settling open AR items against cash payments                 |
| Net AR Impact                     | Net debit amount on RECO GL account group entries                        |
| Net Cash Impact                   | Net amount on FUND GL account group entries                              |
| RECO                              | GL Account Group for AR reconciliation accounts                          |
| FUND                              | GL Account Group for cash/bank clearing accounts                         |
| LEAPFROG                          | BMS internal data schema/layer in SAP HANA and Oracle                    |
| CVQ_FI_AR_GOTC_CUST_CLRD_CASH    | SAP HANA view: Customer Cash Cleared – GOTC                             |
| V_LEAP_SECURITY                   | Security view: authorized users and company code access                  |
| BMSUID                            | BMS User ID for row-level security matching                              |
| OTC                               | Order to Cash — end-to-end business process                             |


END OF KNOWLEDGE FILE
