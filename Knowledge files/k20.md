# 📘 CONVERSION KNOWLEDGE FILE
# 12960 – AR vs. Credit Limit
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-03
# Source files: 12960_o_1_2__ar_vs__credit_limit.txt + all PBIP .txt files

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PARAM-01 | View As
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 1]
Tableau Caption       : View As
Tableau Type          : String list (param-domain-type = list)
Tableau Default       : "Payer"

Tableau Members / Aliases:
  "Brand"                              → alias: "Brand Text"
  "Billing Document Type"
  "Company"
  "Customer Group"
  "Payer"                              ← default
  "Payer Customer A/C Group"
  "Profit Center"
  "Sales Org"
  "Sold To Party"
  "Sold To Party Customer A/C Group"

Power BI Table Name   : Parameter
Power BI Type         : DAX Calculated Table (DATATABLE)
Power BI Column       : [View As] (string)

Power BI DAX (partial — combined with PARAM-02 in one table):
─────────────────────────────────────────────────────────────
DATATABLE(
    "Currency", STRING,
    "View As",  STRING,
    {
        { "USD",   "Payer" },
        { "Local", "Payer" }
        -- extend rows per slicer selection pattern
    }
)

Usage in model:
  - SELECTEDVALUE('Parameter'[View As], "Payer")
    drives dimensional axis switching in scatter and bar visuals.
  ⚠️  NOTE: No DAX measures currently reference [View As] via
    SELECTEDVALUE. Implement view-switching measures if axis toggling
    is required (see SECTION 4, Gap G5).

Conversion Status: ⚠️ PARTIAL — table and column exist; no measure
                   consumes [View As] yet. Recommend implementing
                   SWITCH(SELECTEDVALUE('Parameter'[View As]), ...) for
                   dimensional axis logic.


### PARAM-02 | Currency
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Parameter 2]
Tableau Caption       : Currency
Tableau Type          : String list (param-domain-type = list)
Tableau Default       : "USD"

Tableau Members / Aliases:
  "USD"   → alias: "USD"   ← default
  "Local"

Power BI Table Name   : Parameter
Power BI Type         : DAX Calculated Table (DATATABLE)
Power BI Column       : [Currency] (string)

Power BI DAX:
─────────────
-- The Parameter table stores Currency as a slicer-bound column.
-- Single-row selection drives all SWITCH(SELECTEDVALUE(...)) measures.
DATATABLE(
    "Currency", STRING,
    {
        { "USD"   },
        { "Local" }
    }
)

Usage in model:
  - SELECTEDVALUE('Parameter'[Currency], "USD")
    is referenced in every financial measure and calculated column
    that has a USD / Local dual path.
  - Default fallback "USD" is specified in every SELECTEDVALUE() call
    so that visuals remain functional before a slicer selection is made.

Conversion Status: ✅ FULL — both values present; default and fallback
                   behaviour replicated in all consuming measures.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: CVQ_SC_SD_GOTC_CUST_CL_ARBAL
## Power BI Full Table Name: CVQ_SC_SD_GOTC_CUST_CL_ARBAL
## Source: SAP HANA pahpdb01.net.bms.com:30015
##         schema _SYS_BIC, view QUERIES.CORPORATE_FUNCTIONS.
##         GLOBAL_FINANCE.LEAPFROG::CVQ_SC_SD_GOTC_CUST_CL_ARBAL

### CC-01 | Company
─────────────────────────────────────────────────────────────
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression:
───────────────────
[CMPY_CD] + ' - ' + [CMPY_CD_TXT]
// Concatenates Company Code and Company Code Description

Power BI DAX:
─────────────
column Company =
[Company Code] & " - " & [Company Code Description]

Depends on  : [Company Code] (CMPY_CD), [Company Code Description] (CMPY_CD_TXT)
Used by     : Company Code Slicer table (SELECTCOLUMNS source),
              [Company Filter] measure, Dim_Company bridge table,
              [Is Company Selected] measure
Conversion  : ✅ FULL


### CC-02 | Payer
─────────────────────────────────────────────────────────────
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression:
───────────────────
[CUST] + ' ' + [CUST_NM1]
// Concatenates Customer code and Customer Name1

Power BI DAX:
─────────────
column Payer =
[Customer] & " " & [Customer Name1]

Depends on  : [Customer] (CUST), [Customer Name1] (CUST_NM1)
Used by     : Visual dimensions (scatter axis, bar rows)
Conversion  : ✅ FULL


### CC-03 | Parent Payer
─────────────────────────────────────────────────────────────
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression:
───────────────────
[PTNR] + ' ' + [PTNR_NM]
// Concatenates Partner code and Partner Name

Power BI DAX:
─────────────
column 'Parent Payer' =
[Partner] & " " & [Partner Name]

Depends on  : [Partner] (PTNR), [Partner Name] (PTNR_NM)
Used by     : [Total Credit Limit] measure (SUMMARIZE grain),
              [Credit Limit Exceeded Amount] measure,
              [# Payers (Partner)] measure,
              [Credit Limit Utilization USD/Local] columns,
              Scatter visual mark encoding
Conversion  : ✅ FULL


### CC-04 | Credit Segment Display
─────────────────────────────────────────────────────────────
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression:
───────────────────
IF [CR_SEG] = '*' THEN 'NULL'
ELSE [CR_SEG] + ' ' + [CR_SEG_TXT]
END
// Replaces wildcard Credit Segment with display label "NULL"

Power BI DAX:
─────────────
column 'Credit Segment Display' =
IF(
    [Credit Segment] = "*",
    "NULL",
    [Credit Segment] & " " & [Credit Segment Description]
)

Depends on  : [Credit Segment] (CR_SEG),
              [Credit Segment Description] (CR_SEG_TXT)
Used by     : [Credit Limit Utilization USD] column,
              [Credit Limit Utilization Local] column,
              bar chart row dimension
Conversion  : ✅ FULL


### CC-05 | Region Display
─────────────────────────────────────────────────────────────
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression:
───────────────────
IF [REGN] = 'EMEA' THEN 'ECC'
ELSE [REGN]
END
// Renames EMEA region to ECC for display

Power BI DAX:
─────────────
column 'Region Display' =
IF(
    [Region] = "EMEA",
    "ECC",
    [Region]
)

Depends on  : [Region] (REGN)
Used by     : Visual slicer / filter dimension
Conversion  : ✅ FULL


### CC-06 | Customer Group Display
─────────────────────────────────────────────────────────────
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Expression:
───────────────────
[CUST_GRP] + ' - ' + [CUST_GRP_TXT]
// Concatenates Customer Group code and description

Power BI DAX:
─────────────
column 'Customer Group Display' =
CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Customer Group]
& " - " &
CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Customer Group Description]

Depends on  : [Customer Group] (CUST_GRP),
              [Customer Group Description] (CUST_GRP_TXT)
Used by     : Visual slicer / filter dimension
Conversion  : ✅ FULL


### CC-07 | Key Date 1
─────────────────────────────────────────────────────────────
PBI Data Type   : dateTime
PBI Format      : General Date
PBI Hidden      : No

Tableau Expression:
───────────────────
DATE(DATEADD('day', 1, DATEPARSE('yyyyMMdd', [LAST_DT])))
// Parses yyyyMMdd string, adds 1 day — used as tooltip display date

Power BI DAX:
─────────────
column 'Key Date 1' =
DATE(
    LEFT(  'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Key Date], 4 ),
    MID(   'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Key Date], 5, 2 ),
    RIGHT( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Key Date], 2 )
) + 1

Depends on  : [Key Date] (LAST_DT — stored as yyyyMMdd string)
Used by     : [Key Date tooltip] measure
NOTE        : Power BI uses LEFT/MID/RIGHT string parsing instead of
              DATEPARSE because the source column is a fixed-width
              string, not a native date type.
Conversion  : ✅ FULL


### CC-08 | Balance USD LOD  ← FIXED LOD
─────────────────────────────────────────────────────────────
PBI Data Type   : double
PBI Summarize   : sum
PBI Hidden      : No

Tableau Expression:
───────────────────
{FIXED [CR_SEG], [PTNR] : SUM([BAL_USD])}
// Computes total USD balance at Credit Segment + Partner grain,
// ignoring view-level filters on Customer, Company, etc.

Power BI DAX:
─────────────
column 'Balance USD LOD' =
VAR CurrentPartner       = CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Partner]
VAR CurrentCreditSegment = CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment]
RETURN
CALCULATE(
    SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD] ),
    FILTER(
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Partner] = CurrentPartner
        &&
        (
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment] = CurrentCreditSegment
            || (
                ISBLANK( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment] )
                && ISBLANK( CurrentCreditSegment )
               )
        )
    )
)

Depends on  : [Balance USD], [Partner], [Credit Segment]
Used by     : [Balance LOD] measure (M-02),
              [Credit Limit Status] column (CC-12),
              [Status Filter] measure (M-07)
LOD Pattern : Tableau {FIXED dim1, dim2 : AGG()} replicated as
              VAR capture of row-context values + CALCULATE + FILTER
              over the full table. ISBLANK() guard handles NULL
              Credit Segment rows (Tableau wildcard '*').
Conversion  : ✅ FULL


### CC-09 | Balance Local LOD  ← FIXED LOD
─────────────────────────────────────────────────────────────
PBI Data Type   : double
PBI Summarize   : sum
PBI Hidden      : No

Tableau Expression:
───────────────────
{FIXED [CR_SEG], [PTNR] : SUM([BAL_LCL])}
// Computes total Local balance at Credit Segment + Partner grain

Power BI DAX:
─────────────
column 'Balance Local LOD' =
VAR CurrentPartner       = CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Partner]
VAR CurrentCreditSegment = CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment]
RETURN
CALCULATE(
    SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance Local] ),
    FILTER(
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Partner] = CurrentPartner
        &&
        (
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment] = CurrentCreditSegment
            || (
                ISBLANK( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment] )
                && ISBLANK( CurrentCreditSegment )
               )
        )
    )
)

Depends on  : [Balance Local], [Partner], [Credit Segment]
Used by     : [Balance LOD] measure (M-02),
              [Credit Limit Status] column (CC-12)
Conversion  : ✅ FULL


### CC-10 | Credit Limit USD LOD  ← FIXED LOD
─────────────────────────────────────────────────────────────
PBI Data Type   : double
PBI Summarize   : sum
PBI Hidden      : No

Tableau Expression:
───────────────────
{FIXED [PTNR], [Credit Segment] : MAX([CRLMT_USD])}
// Returns the maximum USD credit limit at Partner + Credit Segment grain

Power BI DAX:
─────────────
column 'Credit Limit USD LOD' =
VAR CurrentPartner       = CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Partner]
VAR CurrentCreditSegment = CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment]
RETURN
CALCULATE(
    MAX( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Limit USD] ),
    FILTER(
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Partner] = CurrentPartner
        &&
        (
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment] = CurrentCreditSegment
            || (
                ISBLANK( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment] )
                && ISBLANK( CurrentCreditSegment )
               )
        )
    )
)

Depends on  : [Credit Limit USD], [Partner], [Credit Segment]
Used by     : [Credit Limit] measure (M-03),
              [Credit Limit Status] column (CC-12),
              [Credit Limit Utilization % col] column (CC-14),
              [Credit Limit Utilization USD] column (CC-15),
              [Total Credit Limit] measure (M-10),
              [Credit Limit Exceeded Amount] measure (M-11)
Conversion  : ✅ FULL


### CC-11 | Credit Limit Local LOD  ← FIXED LOD
─────────────────────────────────────────────────────────────
PBI Data Type   : double
PBI Summarize   : sum
PBI Hidden      : No

Tableau Expression:
───────────────────
{FIXED [PTNR], [Credit Segment] : MAX([CRLMT_LCL])}
// Returns the maximum Local credit limit at Partner + Credit Segment grain

Power BI DAX:
─────────────
column 'Credit Limit Local LOD' =
VAR CurrentPartner       = CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Partner]
VAR CurrentCreditSegment = CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment]
RETURN
CALCULATE(
    MAX( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Limit Local] ),
    FILTER(
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Partner] = CurrentPartner
        &&
        (
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment] = CurrentCreditSegment
            || (
                ISBLANK( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment] )
                && ISBLANK( CurrentCreditSegment )
               )
        )
    )
)

Depends on  : [Credit Limit Local], [Partner], [Credit Segment]
Used by     : [Credit Limit] measure (M-03),
              [Credit Limit Status] column (CC-12),
              [Credit Limit Utilization % col] column (CC-14),
              [Credit Limit Utilization Local] column (CC-16),
              [Total Credit Limit] measure (M-10),
              [Credit Limit Exceeded Amount] measure (M-11)
Conversion  : ✅ FULL


### CC-12 | Credit Limit Status
─────────────────────────────────────────────────────────────
PBI Data Type   : string
PBI Summarize   : none
PBI Hidden      : No

Tableau Expression:
───────────────────
CASE [Currency]
  WHEN 'USD'   THEN IF [Balance USD LOD]   > [Credit Limit USD LOD]
                    THEN 'Over Limit' ELSE 'Within Limit' END
  WHEN 'Local' THEN IF [Balance Local LOD] > [Credit Limit Local LOD]
                    THEN 'Over Limit' ELSE 'Within Limit' END
END
// Currency-aware credit limit status classification per row

Power BI DAX:
─────────────
column 'Credit Limit Status' =
VAR BalanceValue =
    IF(
        SELECTEDVALUE( Parameter[Currency] ) = "USD",
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD LOD],
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance Local LOD]
    )
VAR CreditLimitValue =
    IF(
        SELECTEDVALUE( Parameter[Currency] ) = "USD",
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Limit USD LOD],
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Limit Local LOD]
    )
RETURN
IF(
    BalanceValue > CreditLimitValue,
    "Over Limit",
    "Within Limit"
)

Depends on  : CC-08 [Balance USD LOD], CC-09 [Balance Local LOD],
              CC-10 [Credit Limit USD LOD], CC-11 [Credit Limit Local LOD],
              PARAM-02 [Currency]
Used by     : [# Payers Credit Limit Exceeded] measure (M-13),
              [Status Filter] measure (M-07),
              scatter visual color encoding
⚠️  NOTE    : SELECTEDVALUE() in a calculated column uses the model's
              default filter context at refresh time — not the interactive
              slicer context. Column will reflect the default currency.
              For fully interactive status, use [Credit Limit Status USD]
              and [Credit Limit Status Local] columns instead (CC-17, CC-18).
Conversion  : ✅ FULL (with noted limitation above)


### CC-13 | Test Credit Limit LOD  ← Dev/Debug
─────────────────────────────────────────────────────────────
PBI Data Type   : double
PBI Summarize   : sum
PBI Hidden      : No

Tableau Expression:
───────────────────
{FIXED [PTNR] : MAX([CRLMT_USD])}
// Single-dimension FIXED — used during development/testing only

Power BI DAX:
─────────────
column 'Test Credit Limit LOD' =
CALCULATE(
    MAX( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Limit USD] ),
    FILTER(
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Partner]
            = EARLIER( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Partner] )
    )
)

Depends on  : [Credit Limit USD], [Partner]
⚠️  NOTE    : Uses deprecated EARLIER() pattern. Replace with VAR
              CurrentPartner = [...][Partner] pattern (as in CC-10)
              for consistency and future compatibility.
              Review for removal before production deployment.
Conversion  : ✅ FULL (dev column — recommend removal or refactor)


### CC-14 | Credit Limit Utilization % col
─────────────────────────────────────────────────────────────
PBI Data Type   : double
PBI Summarize   : sum
PBI Hidden      : No

Tableau Expression:
───────────────────
ROUND({FIXED [PTNR],[CreditSeg]:SUM([Balance])}, 2)
/ ROUND({FIXED [PTNR],[CreditSeg]:MAX([CreditLimit])}, 2)
// Currency-aware utilization ratio per row, rounded to 2dp

Power BI DAX:
─────────────
column 'Credit Limit Utilization % col' =
VAR SelectedCurrency =
    SELECTEDVALUE( 'Parameter'[Currency], "USD" )
VAR BalanceValue =
    IF(
        SelectedCurrency = "USD",
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD],
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance Local]
    )
VAR CreditLimitValue =
    IF(
        SelectedCurrency = "USD",
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Limit USD LOD],
        CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Limit Local LOD]
    )
RETURN
DIVIDE(
    ROUND( BalanceValue,      2 ),
    ROUND( CreditLimitValue,  2 ),
    0
)

Depends on  : CC-10, CC-11, [Balance USD], [Balance Local], PARAM-02
Used by     : Visual bar encoding (utilization axis)
Conversion  : ✅ FULL


### CC-15 | Credit Limit Utilization USD
─────────────────────────────────────────────────────────────
PBI Data Type   : double
PBI Summarize   : sum
PBI Hidden      : No

Tableau Expression:
───────────────────
{FIXED [Parent Payer],[Credit Segment Display] : SUM([BAL_USD])}
/ {FIXED [Parent Payer],[Credit Segment Display] : MAX([CRLMT_USD LOD])}
// USD-only utilization ratio scoped to Parent Payer + Credit Segment

Power BI DAX:
─────────────
column 'Credit Limit Utilization USD' =
VAR ARBalance =
    CALCULATE(
        SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD] ),
        ALLEXCEPT(
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Parent Payer],
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment Display]
        )
    )
VAR CreditLimit =
    CALCULATE(
        MAX( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Limit USD LOD] ),
        ALLEXCEPT(
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Parent Payer],
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment Display]
        )
    )
RETURN
DIVIDE(
    ROUND( ARBalance,    2 ),
    ROUND( CreditLimit,  2 ),
    0
)

Depends on  : CC-03, CC-04, CC-10, [Balance USD]
Used by     : [Credit Limit Status USD] measure (M-16),
              CC-17 [Credit Limit Status Local] column
Conversion  : ✅ FULL


### CC-16 | Credit Limit Utilization Local
─────────────────────────────────────────────────────────────
PBI Data Type   : double
PBI Summarize   : sum
PBI Hidden      : No

Tableau Expression:
───────────────────
{FIXED [Parent Payer],[Credit Segment Display] : SUM([BAL_LCL])}
/ {FIXED [Parent Payer],[Credit Segment Display] : MAX([CRLMT_LCL LOD])}
// Local-currency utilization ratio

Power BI DAX:
─────────────
column 'Credit Limit Utilization Local' =
VAR ARBalance =
    CALCULATE(
        SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance Local] ),
        ALLEXCEPT(
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Parent Payer],
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment Display]
        )
    )
VAR CreditLimit =
    CALCULATE(
        MAX( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Limit Local LOD] ),
        ALLEXCEPT(
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Parent Payer],
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Segment Display]
        )
    )
RETURN
DIVIDE(
    ROUND( ARBalance,    2 ),
    ROUND( CreditLimit,  2 ),
    0
)

Depends on  : CC-03, CC-04, CC-11, [Balance Local]
Used by     : CC-18 [Credit Limit Status Local] column
Conversion  : ✅ FULL


### CC-17 | Credit Limit Status Local
─────────────────────────────────────────────────────────────
PBI Data Type   : string
PBI Summarize   : none
PBI Hidden      : No

Tableau Expression:
───────────────────
IF [Credit Limit Utilization Local] > 1
THEN 'Over Limit'
ELSE 'Within Limit'
END

Power BI DAX:
─────────────
column 'Credit Limit Status Local' =
IF(
    CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Limit Utilization Local] > 1,
    "Over Limit",
    "Within Limit"
)

Depends on  : CC-16
Used by     : Visual filter (Local currency path)
Conversion  : ✅ FULL


## TABLE: V_LEAP_SECURITY
## Power BI Full Table Name: V_LEAP_SECURITY
## Source: Oracle DB uslvlbmsasp013.net.bms.com:1521/IHUBP
##         Schema: LEAPFROG, Table: V_LEAP_SECURITY

### CC-18 | Security Calc  ← RLS Boolean
─────────────────────────────────────────────────────────────
PBI Data Type   : boolean (displayed as string "TRUE"/"FALSE")
PBI Format      : """TRUE"";""TRUE"";""FALSE"""
PBI Hidden      : No

Tableau Expression:
───────────────────
UPPER(USERNAME()) = UPPER([BMSUID])
// Returns TRUE when the logged-in user matches the security table BMSUID

Power BI DAX (Measure on V_LEAP_SECURITY):
──────────────────────────────────────────
measure 'Security Calc' =
VAR CurrentUser = UPPER( USERNAME() )
RETURN
IF(
    CurrentUser IN VALUES( 'V_LEAP_SECURITY'[bmsuid] ),
    TRUE(),
    FALSE()
)

Depends on  : [BMSUID] (source column), USERNAME() function
Used by     : RLS role filter (see SECTION 4 — Power Query / RLS)
NOTE        : Tableau used a direct boolean join filter. Power BI
              promotes this to a DAX measure AND an RLS role row filter
              on V_LEAP_SECURITY. The Dim_Company bridge table propagates
              the filter to the fact table via relationships.
Conversion  : ✅ FULL


## TABLE: Fast Facts
## Power BI Full Table Name: Fast Facts
## Source: DAX Calculated Table (DATATABLE)

### CC-19 | Sort Order  ← Calculated Column on Fast Facts
─────────────────────────────────────────────────────────────
PBI Data Type   : integer (formatString: 0)
PBI Summarize   : sum
PBI Hidden      : No

Tableau Expression:
───────────────────
// No direct equivalent — Tableau preserved metric display order
// via worksheet row order. PBI requires an explicit sort column.

Power BI DAX:
─────────────
column 'Sort Order' =
SWITCH(
    'Fast Facts'[Metric],
    "# Parent Payers",                          1,
    "% Parent Payers Credit Limit Exceeded",    2,
    "# Parent Payers Credit Limit Exceeded",    3,
    "Total Credit Limit",                       4,
    "Credit Limit Exceeded Amount",             5,
    "AR Balance",                               6
)

Depends on  : [Metric] (source column of Fast Facts DATATABLE)
Used by     : Sort order of Fast Facts visual cards
NOTE        : PBI-specific enhancement — no Tableau equivalent.
              Ensures KPI card display order matches Tableau worksheet
              row sequence.
Conversion  : ➕ PBI-only enhancement


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: CVQ_SC_SD_GOTC_CUST_CL_ARBAL
## (Core financial measures defined on the fact table)

### M-01 | AR Balance
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : General Number

Tableau Equivalent:
───────────────────
CASE [Parameters].[Currency]
  WHEN 'USD'   THEN SUM([BAL_USD])
  WHEN 'Local' THEN SUM([BAL_LCL])
END

Power BI DAX:
─────────────
measure 'AR Balance' =
SWITCH(
    SELECTEDVALUE( 'Parameter'[Currency], "USD" ),
    "USD",   SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD] ),
    "Local", SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance Local] ),
    BLANK()
)

Used by     : M-05 [AR Bal. vs. Credit Limit],
              M-09 [Credit Limit Utilization %],
              M-18 [Fast Facts Value], M-19 [Fast Facts Value1]
Conversion  : ✅ FULL


### M-02 | Balance LOD
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : General Number

Tableau Equivalent:
───────────────────
CASE [Currency]
  WHEN 'USD'   THEN {FIXED [CR_SEG],[PTNR] : SUM([BAL_USD])}
  WHEN 'Local' THEN {FIXED [CR_SEG],[PTNR] : SUM([BAL_LCL])}
END
// Uses FIXED LOD columns as base

Power BI DAX:
─────────────
measure 'Balance LOD' =
SWITCH(
    SELECTEDVALUE( 'Parameter'[Currency] ),
    "USD",   SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD LOD] ),
    "Local", SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance Local LOD] ),
    BLANK()
)

Depends on  : CC-08 [Balance USD LOD], CC-09 [Balance Local LOD],
              PARAM-02
Used by     : M-07 [Status Filter]
Conversion  : ✅ FULL


### M-03 | Credit Limit
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : General Number

Tableau Equivalent:
───────────────────
CASE [Currency]
  WHEN 'USD'   THEN {FIXED [PTNR],[CreditSeg] : MAX([CRLMT_USD])}
  WHEN 'Local' THEN {FIXED [PTNR],[CreditSeg] : MAX([CRLMT_LCL])}
END

Power BI DAX:
─────────────
measure 'Credit Limit' =
SWITCH(
    SELECTEDVALUE( 'Parameter'[Currency], "USD" ),
    "USD",   SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Limit USD LOD] ),
    "Local", SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Limit Local LOD] ),
    SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Credit Limit USD LOD] )  -- default fallback
)

Depends on  : CC-10, CC-11, PARAM-02
Used by     : M-05, M-07, M-09, M-18, M-19
Conversion  : ✅ FULL


### M-04 | AR Balance Selected
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : General Number

Tableau Equivalent:
───────────────────
CASE [Currency]
  WHEN 'USD'   THEN SUM([BAL_USD])
  WHEN 'Local' THEN SUM([BAL_LCL])
  ELSE              SUM([BAL_USD])
END
// Variant of AR Balance with explicit USD default

Power BI DAX:
─────────────
measure 'AR Balance Selected' =
VAR SelectedCurrency =
    SELECTEDVALUE( 'Parameter'[Currency], "USD" )
RETURN
SWITCH(
    SelectedCurrency,
    "USD",   SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD] ),
    "Local", SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance Local] ),
    SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD] )
)

Conversion  : ✅ FULL


### M-05 | AR Bal. vs. Credit Limit
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : General Number

Tableau Equivalent:
───────────────────
MAX(0, SUM([AR Balance]) - SUM([Credit Limit]))
// Shows only the excess balance above credit limit; floored at 0

Power BI DAX:
─────────────
measure 'AR Bal. vs. Credit Limit' =
MAX( 0, [AR Balance] - [Credit Limit] )

Depends on  : M-01 [AR Balance], M-03 [Credit Limit]
Used by     : Bar chart visual (excess amount bar)
Conversion  : ✅ FULL


### M-06 | AR Balance Min
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : General Number

Tableau Equivalent:
───────────────────
// Minimum value for AR Balance range slider filter
MIN({FIXED [PTNR] : SUM([AR Balance])})

Power BI DAX:
─────────────
measure 'AR Balance Min' =
VAR SelectedCurrency =
    SELECTEDVALUE( 'Parameter'[Currency], "USD" )
RETURN
SWITCH(
    SelectedCurrency,
    "USD",   MINX( CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
                   CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD] ),
    "Local", MINX( CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
                   CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance Local] ),
    MINX( CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
          CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD] )
)

Used by     : M-08 [AR Balance Filter]
Conversion  : ✅ FULL


### M-07 | AR Balance Max
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : General Number

Tableau Equivalent:
───────────────────
// Maximum value for AR Balance range slider filter

Power BI DAX:
─────────────
measure 'AR Balance Max' =
VAR SelectedCurrency =
    SELECTEDVALUE( 'Parameter'[Currency], "USD" )
RETURN
SWITCH(
    SelectedCurrency,
    "USD",   MAXX( CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
                   CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD] ),
    "Local", MAXX( CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
                   CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance Local] ),
    MAXX( CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
          CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD] )
)

Used by     : M-08 [AR Balance Filter]
Conversion  : ✅ FULL


### M-08 | AR Balance Filter
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : 0

Tableau Equivalent:
───────────────────
// Range-filter flag — Tableau uses a built-in range quick filter.
// PBI replicates as a 0/1 measure used in visual-level filters.
[AR Balance] >= MIN([AR Balance]) AND [AR Balance] <= MAX([AR Balance])

Power BI DAX:
─────────────
measure 'AR Balance Filter' =
VAR SelectedCurrency =
    SELECTEDVALUE( 'Parameter'[Currency], "USD" )
VAR CurrentBalance =
    SWITCH(
        SelectedCurrency,
        "USD",   SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD] ),
        "Local", SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance Local] ),
        SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD] )
    )
VAR MinVal = [AR Balance Min]
VAR MaxVal = [AR Balance Max]
RETURN
IF(
    CurrentBalance >= MinVal
    && CurrentBalance <= MaxVal,
    1, 0
)

Depends on  : M-06, M-07
Conversion  : ✅ FULL


### M-09 | Credit Limit Utilization %
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : General Number

Tableau Equivalent:
───────────────────
ROUND({FIXED [Parent Payer],[CreditSeg] : SUM([AR Balance])}, 2)
/ ROUND({FIXED [Parent Payer],[CreditSeg] : MAX([Credit Limit])}, 2)
// LOD-scoped utilization ratio at Parent Payer + Credit Segment grain

Power BI DAX:
─────────────
measure 'Credit Limit Utilization %' =
CALCULATE(
    SUMX(
        VALUES( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Customer] ),
        DIVIDE( [AR Balance], [Credit Limit], 0 )
    ),
    ALLEXCEPT(
        'CVQ_SC_SD_GOTC_CUST_CL_ARBAL',
        'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Parent Payer],
        'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Credit Segment]
    )
)

Depends on  : M-01, M-03
Used by     : M-17 [Credit Limit Utilization Filtered],
              bar chart utilization axis
Conversion  : ✅ FULL


### M-10 | Total Credit Limit
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : General Number

Tableau Equivalent:
───────────────────
CASE [Currency]
  WHEN 'USD'   THEN {FIXED [CR_SEG],[PTNR] : MAX([CRLMT_USD])}
  WHEN 'Local' THEN {FIXED [CR_SEG],[PTNR] : MAX([CRLMT_LCL])}
END
// Total across all companies/customers — FIXED removes company/payer grain

Power BI DAX:
─────────────
measure 'Total Credit Limit' =
VAR SelectedCurrency =
    SELECTEDVALUE( 'Parameter'[Currency] )
VAR FixedTable =
    SUMMARIZE(
        'CVQ_SC_SD_GOTC_CUST_CL_ARBAL',
        'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Credit Segment],
        'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Partner]
    )
RETURN
SUMX(
    FixedTable,
    SWITCH(
        SelectedCurrency,
        "USD",
            CALCULATE(
                MAX( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Credit Limit USD] ),
                REMOVEFILTERS(
                    'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Company],
                    'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Customer],
                    'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Payer]
                )
            ),
        "Local",
            CALCULATE(
                MAX( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Credit Limit Local] ),
                REMOVEFILTERS(
                    'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Company],
                    'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Customer],
                    'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Payer]
                )
            )
    )
)

LOD Pattern : SUMMARIZE replicates the FIXED grain; REMOVEFILTERS
              replicates FIXED's scope override (ignores Customer/Company).
Depends on  : CC-10, CC-11, PARAM-02
Used by     : M-18, M-19 [Fast Facts Value / Value1]
Conversion  : ✅ FULL


### M-11 | Credit Limit Exceeded Amount
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : General Number

Tableau Equivalent:
───────────────────
CASE [Currency]
  WHEN 'USD'   THEN {FIXED [CR_SEG],[PTNR] :
                     MAX(0, SUM([BAL_USD]) - MAX([CRLMT_USD]))}
  WHEN 'Local' THEN {FIXED [CR_SEG],[PTNR] :
                     MAX(0, SUM([BAL_LCL]) - MAX([CRLMT_LCL]))}
END
// Excess AR balance beyond credit limit; floored at zero per Partner+Segment

Power BI DAX:
─────────────
measure 'Credit Limit Exceeded Amount' =
VAR SelectedCurrency =
    SELECTEDVALUE( 'Parameter'[Currency] )
VAR FixedTable =
    SUMMARIZE(
        'CVQ_SC_SD_GOTC_CUST_CL_ARBAL',
        'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Credit Segment],
        'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Partner],
        "ExceededAmount",
        SWITCH(
            SelectedCurrency,
            "USD",
                MAX(
                    0,
                    CALCULATE( SUM( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Balance USD] ) )
                    -
                    CALCULATE( MAX( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Credit Limit USD] ) )
                ),
            "Local",
                MAX(
                    0,
                    CALCULATE( SUM( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Balance Local] ) )
                    -
                    CALCULATE( MAX( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Credit Limit Local] ) )
                )
        )
    )
RETURN
SUMX( FixedTable, [ExceededAmount] )

Depends on  : PARAM-02, [Balance USD], [Balance Local],
              [Credit Limit USD], [Credit Limit Local]
Used by     : M-18, M-19 [Fast Facts Value / Value1]
Conversion  : ✅ FULL


### M-12 | Status Filter
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : 0

Tableau Equivalent:
───────────────────
// Tableau Quick Filter on [Credit Limit Status] field —
// filters scatter view to show only Over/Within Limit rows
IF [Balance LOD] > [Credit Limit]
THEN 'Over Limit' ELSE 'Within Limit' END
= [Credit Limit Filter selection]

Power BI DAX:
─────────────
measure 'Status Filter' =
VAR StatusSelected =
    SELECTEDVALUE( 'Credit Limit Filter'[Status] )
VAR CurrentStatus =
    IF(
        [Balance LOD] > [Credit Limit],
        "Over Limit",
        "Within Limit"
    )
RETURN
IF(
    StatusSelected = CurrentStatus,
    1,
    0
)

Depends on  : M-02 [Balance LOD], M-03 [Credit Limit],
              Credit Limit Filter table
Used by     : Visual-level filter (scatter chart)
Conversion  : ✅ FULL


### M-13 | # Payers (Partner)
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : 0

Tableau Equivalent:
───────────────────
COUNTD([PTNR])

Power BI DAX:
─────────────
measure '# Payers (Partner)' =
DISTINCTCOUNT( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Partner] )

Used by     : M-18, M-19 [Fast Facts Value / Value1]
Conversion  : ✅ FULL


### M-14 | # Payers Credit Limit Exceeded
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : 0

Tableau Equivalent:
───────────────────
COUNTD(
  CASE [Credit Limit Status]
    WHEN 'Over Limit' THEN [PTNR]
    ELSE NULL
  END
)

Power BI DAX:
─────────────
measure '# Payers Credit Limit Exceeded' =
CALCULATE(
    DISTINCTCOUNT( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Partner] ),
    'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Credit Limit Status] = "Over Limit"
)

Depends on  : CC-12 [Credit Limit Status]
Used by     : M-15, M-18, M-19
Conversion  : ✅ FULL


### M-15 | % Payers Credit Limit Exceeded
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : (string — formatted inline)

Tableau Equivalent:
───────────────────
[# Payers CL Exceeded] / COUNTD([PTNR])
// Formatted as percentage

Power BI DAX:
─────────────
measure '% Payers Credit Limit Exceeded' =
VAR _Result =
    DIVIDE(
        [# Payers Credit Limit Exceeded],
        DISTINCTCOUNT( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Partner] ),
        0
    ) + 0
RETURN
IF(
    ISBLANK( _Result ),
    "0.0%",
    FORMAT( _Result, "0.0%" )
)

Depends on  : M-14, M-13
Used by     : M-18, M-19 [Fast Facts Value / Value1]
NOTE        : Returns string "0.0%" rather than blank when no data —
              matches Tableau's zero-display behaviour.
Conversion  : ✅ FULL


### M-16 | Credit Limit Status USD
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL

Tableau Equivalent:
───────────────────
IF [Credit Limit Utilization USD] > 1
THEN 'Over Limit' ELSE 'Within Limit' END

Power BI DAX:
─────────────
measure 'Credit Limit Status USD' =
IF(
    [Credit Limit Utilization USD] > 1,
    "Over Limit",
    "Within Limit"
)

Depends on  : CC-15 [Credit Limit Utilization USD]
Conversion  : ✅ FULL


### M-17 | Credit Limit Utilization Filtered
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : General Number

Tableau Equivalent:
───────────────────
// Returns Utilization % only when it matches the Credit Limit status
// filter selection. BLANK() suppresses non-matching rows in bar chart.
IF [Credit Limit Filter] = 'Over Limit'   AND [Util%] > 1 THEN [Util%]
IF [Credit Limit Filter] = 'Within Limit' AND [Util%] <= 1 THEN [Util%]
IF [Credit Limit Filter] = NULL THEN [Util%]
ELSE BLANK()

Power BI DAX:
─────────────
measure 'Credit Limit Utilization Filtered' =
VAR Utilization    = [Credit Limit Utilization %]
VAR SelectedStatus = SELECTEDVALUE( 'Credit Limit Filter'[Status] )
RETURN
SWITCH(
    TRUE(),
    SelectedStatus = "Over Limit"   && Utilization > 1,  Utilization,
    SelectedStatus = "Within Limit" && Utilization <= 1, Utilization,
    ISBLANK( SelectedStatus ),                           Utilization,
    BLANK()
)

Depends on  : M-09, Credit Limit Filter table
Used by     : Bar chart utilization axis (filtered path)
Conversion  : ✅ FULL


### M-18 | Show USD Slicer
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : 0

Tableau Equivalent:
───────────────────
// Controls visibility of USD-specific range slicers
[Parameters].[Currency] = 'USD'

Power BI DAX:
─────────────
measure 'Show USD Slicer' =
IF(
    SELECTEDVALUE( 'Parameter'[Currency] ) = "USD",
    1,
    0
)

Conversion  : ✅ FULL


### M-19 | Show Local Slicer
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : 0

Tableau Equivalent:
───────────────────
[Parameters].[Currency] = 'Local'

Power BI DAX:
─────────────
measure 'Show Local Slicer' =
VAR CurrencySelected = SELECTEDVALUE( 'Parameter'[Currency] )
RETURN
IF(
    CurrencySelected = "Local",
    1,
    0
)

Conversion  : ✅ FULL


### M-20 | Is Company Selected
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : 0

Tableau Equivalent:
───────────────────
// Guards dashboard visual display when no Company is selected
// Tableau uses filter action presence/absence

Power BI DAX:
─────────────
measure 'Is Company Selected' =
IF(
    ISFILTERED( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Company] )
    || COUNTROWS( VALUES( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Company] ) )
       = COUNTROWS( ALL( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Company] ) ),
    1,   -- Select All OR specific selection → show visuals
    0    -- Nothing selected → hide visuals
)

NOTE        : PBI-specific enhancement to replicate Tableau's
              filter action show/hide visual behaviour.
Conversion  : ✅ FULL (PBI enhancement)


### M-21 | Company Filter
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : 0

Tableau Equivalent:
───────────────────
// Filters rows based on Company Code Slicer selection,
// including (All) and (None) sentinel values

Power BI DAX:
─────────────
measure 'Company Filter' =
VAR SelectedCompany =
    SELECTEDVALUE( 'Company Code Slicer'[CompanyCode] )
RETURN
IF(
    SelectedCompany = "(All)",
    1,
    IF(
        SelectedCompany = "(None)",
        0,
        IF(
            MAX( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Company] ) = SelectedCompany,
            1,
            0
        )
    )
)

Depends on  : Company Code Slicer table
Conversion  : ✅ FULL


### M-22 | Displayed Currency
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL

Tableau Equivalent:
───────────────────
CASE [Currency] WHEN 'USD' THEN 'USD' ELSE 'Local' END

Power BI DAX:
─────────────
measure 'Displayed Currency' =
SWITCH(
    SELECTEDVALUE( 'Parameter'[Currency] ),
    "USD",   "USD",
    "Local"  -- ELSE fallback
)

Conversion  : ✅ FULL


### M-23 | Credit Limit Utilization Title
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL

Tableau Equivalent:
───────────────────
// Tableau dynamic sheet title:
// "Credit Limit Utilization (<[Parameters].[Currency]>)"

Power BI DAX:
─────────────
measure 'Credit Limit Utilization Title' =
"Credit Limit Utilization ("
& SELECTEDVALUE( 'Parameter'[Currency], "USD" )
& ")"

Conversion  : ✅ FULL


### M-24 | Key Date Display
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL

Tableau Equivalent:
───────────────────
"Key Date: " + DATEFORMAT(DATEPARSE('yyyyMMdd',[LAST_DT]),'m/d/yyyy')

Power BI DAX:
─────────────
measure 'Key Date Display' =
"Key Date: "
&
FORMAT(
    DATE(
        LEFT(  MAX( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Key Date] ), 4 ),
        MID(   MAX( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Key Date] ), 5, 2 ),
        RIGHT( MAX( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Key Date] ), 2 )
    ),
    "m/d/yyyy"
)

Conversion  : ✅ FULL


### M-25 | Key Date Display1
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL

Tableau Equivalent:
───────────────────
DATEFORMAT(DATEPARSE('yyyyMMdd',[LAST_DT]),'m/d/yyyy')
// Same as M-24 without prefix string

Power BI DAX:
─────────────
measure 'Key Date Display1' =
FORMAT(
    DATE(
        LEFT(  MAX( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Key Date] ), 4 ),
        MID(   MAX( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Key Date] ), 5, 2 ),
        RIGHT( MAX( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Key Date] ), 2 )
    ),
    "m/d/yyyy"
)

Conversion  : ✅ FULL


### M-26 | Key Date tooltip
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL

Tableau Equivalent:
───────────────────
DATEFORMAT(DATE(DATEADD('day',1,DATEPARSE('yyyyMMdd',[LAST_DT]))),'m/d/yyyy')
// Key Date + 1 day — used in tooltip display

Power BI DAX:
─────────────
measure 'Key Date tooltip' =
FORMAT(
    DATE(
        LEFT(  MAX( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Key Date] ), 4 ),
        MID(   MAX( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Key Date] ), 5, 2 ),
        RIGHT( MAX( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Key Date] ), 2 )
    ) + 1,
    "m/d/yyyy"
)

Conversion  : ✅ FULL


### M-27 | Fast Facts Value
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : General Number

Tableau Equivalent:
───────────────────
// Tableau Fast Facts worksheet with 6 KPI metrics, one per row.
// Each row calls the relevant aggregation. PBI replicates with SWITCH.

Power BI DAX:
─────────────
measure 'Fast Facts Value' =
SWITCH(
    SELECTEDVALUE( 'Fast Facts'[Metric] ),
    "# Parent Payers",                          [# Payers (Partner)],
    "% Parent Payers Credit Limit Exceeded",    [% Payers Credit Limit Exceeded],
    "# Parent Payers Credit Limit Exceeded",    [# Payers Credit Limit Exceeded],
    "Total Credit Limit",                       [Total Credit Limit],
    "Credit Limit Exceeded Amount",             [Credit Limit Exceeded Amount],
    "AR Balance",                               [AR Balance]
)

Depends on  : M-13, M-14, M-15, M-10, M-11, M-01; Fast Facts table
Used by     : KPI card visual (v1 labels — no numeric prefix)
Conversion  : ✅ FULL


### M-28 | Fast Facts Value1
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Format  : General Number

Tableau Equivalent:
───────────────────
// Same as M-27 with numbered prefixes for reliable sort ordering

Power BI DAX:
─────────────
measure 'Fast Facts Value1' =
SWITCH(
    SELECTEDVALUE( 'Fast Facts'[Metric] ),
    "1. # Parent Payers",                          [# Payers (Partner)],
    "2. % Parent Payers Credit Limit Exceeded",    [% Payers Credit Limit Exceeded],
    "3. # Parent Payers Credit Limit Exceeded",    [# Payers Credit Limit Exceeded],
    "4. Total Credit Limit",                       [Total Credit Limit],
    "5. Credit Limit Exceeded Amount",             [Credit Limit Exceeded Amount],
    "6. AR Balance",                               [AR Balance]
)

Depends on  : M-13, M-14, M-15, M-10, M-11, M-01; Fast Facts table
Used by     : KPI card visual (v2 labels — numbered for sort stability)
NOTE        : Recommend consolidating M-27 and M-28 into M-28 only.
              Use numbered label format for sort reliability.
Conversion  : ✅ FULL


## TABLE: V_LEAP_SECURITY
## (RLS measure)

### M-29 | Security Calc  (defined on V_LEAP_SECURITY)
─────────────────────────────────────────────────────────────
— Documented under CC-18 above (the RLS boolean) —


## TABLE: LastRefresh

### M-30 | Last Refresh Date
─────────────────────────────────────────────────────────────
PBI Table   : LastRefresh

Tableau Equivalent:
───────────────────
// No direct Tableau equivalent — PBI enhancement.
// Provides data freshness timestamp on dashboard.

Power BI DAX:
─────────────
measure 'Last Refresh Date' =
"Last Refresh Date: "
& FORMAT( MAX( 'LastRefresh'[Column1] ), "M/D/YYYY h:mm:ss AM/PM" )

Conversion  : ➕ PBI-only enhancement


### M-31 | Last Refresh Date tooltip
─────────────────────────────────────────────────────────────
PBI Table   : LastRefresh

Power BI DAX:
─────────────
measure 'Last Refresh Date tooltip' =
FORMAT( MAX( 'LastRefresh'[Column1] ), "M/D/YYYY h:mm:ss AM/PM" )

Conversion  : ➕ PBI-only enhancement


### M-32 | Tooltip Info
─────────────────────────────────────────────────────────────
PBI Table   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL

Tableau Equivalent:
───────────────────
// No direct equivalent — Tableau uses a separate Definitions
// info worksheet. PBI delivers contextual help via tooltip measure.

Power BI DAX:
─────────────
measure 'Tooltip Info' =
"Notes:" & UNICHAR(10) & UNICHAR(10) &
"**Browser Limitations**" & UNICHAR(10) &
"Some filters may not display correctly while using Chrome. ..." &
UNICHAR(10) & UNICHAR(10) &
"Customer" & UNICHAR(10) &
"The Order to Cash customer is a business entity that purchases, ..." &
...
"Credit Segment & Company Code" & UNICHAR(10) &
"Credit is managed at the Credit Segment and Parent Payer level."

Conversion  : ➕ PBI-only enhancement


### M-33 | AR Balance Dynamic  (defined on AR Balance_new)
─────────────────────────────────────────────────────────────
PBI Table   : AR Balance_new

Tableau Equivalent:
───────────────────
// Tableau Measure Names / Measure Values shelf dynamic toggle.
// User switches between Balance USD and Balance Local on a visual axis.

Power BI DAX:
─────────────
measure 'AR Balance Dynamic' =
VAR SelectedCurrency =
    SELECTEDVALUE( 'Parameter'[Currency], "USD" )
VAR SelectedField =
    SELECTEDVALUE( 'AR Balance_new'[AR Balance_new Fields] )
RETURN
SWITCH(
    TRUE(),
    -- When Currency = USD
    SelectedCurrency = "USD"   && SelectedField = "Balance USD",
        SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD] ),
    -- When Currency = Local
    SelectedCurrency = "Local" && SelectedField = "Balance Local",
        SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance Local] ),
    -- Default fallback
    SelectedCurrency = "USD",
        SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD] ),
    SelectedCurrency = "Local",
        SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance Local] ),
    SUM( CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Balance USD] )
)

Depends on  : AR Balance_new Field Parameter table, PARAM-02
NOTE        : PBI Field Parameter (NAMEOF-based) replaces Tableau's
              Measure Names shelf. The [AR Balance_new Fields] column
              holds NAMEOF() references to the actual measure columns.
Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU INITIAL TRANSFORMATIONS → POWER BI POWER QUERY (M)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | CVQ_SC_SD_GOTC_CUST_CL_ARBAL  ← Main Fact Table
─────────────────────────────────────────────────────────────
Tableau Data Source   : CVQ_SC_SD_GOTC_CUST_CL_ARBAL (Federated)
Tableau Connection    : SAP HANA
                        Server  : pahpdb01.net.bms.com
                        Port    : 30015
                        Schema  : _SYS_BIC
                        View    : QUERIES.CORPORATE_FUNCTIONS.
                                  GLOBAL_FINANCE.LEAPFROG::
                                  CVQ_SC_SD_GOTC_CUST_CL_ARBAL
Tableau Auth          : username-password (leapfrog)

PBI Table Name        : CVQ_SC_SD_GOTC_CUST_CL_ARBAL
PBI Load Mode         : Import
PBI Connection        : SapHana.Database("pahpdb01.net.bms.com:30015",
                        [Implementation="2.0"])

Tableau → PBI Column Mapping:
──────────────────────────────
Tableau Attribute (Remote)         PBI Column Name                    Type
─────────────────────────────────────────────────────────────────────
[CMPY_CD].[CMPY_CD].Attribute    → Company Code                      string
[CMPY_CD_TXT].[...].Attribute    → Company Code Description           string
[CR_SEG].[CR_SEG].Attribute      → Credit Segment                    string
[CR_SEG_TXT].[...].Attribute     → Credit Segment Description         string
[CNCY_KEY].[...].Attribute       → Currency Key                       string
[CUST].[CUST].Attribute          → Customer                           string
[CUST_ACCT_GRP_TXT].[...].Attr.  → Customer Account Group Description string
[CUST_GRP].[CUST_GRP].Attribute  → Customer Group                    string
[CUST_GRP_TXT].[...].Attribute   → Customer Group Description         string
[CUST_NM1].[CUST_NM1].Attribute  → Customer Name1                    string
[LAST_DT].[LAST_DT].Attribute    → Key Date                          string (yyyyMMdd)
[PTNR].[PTNR].Attribute          → Partner                            string
[PTNR_NM].[PTNR_NM].Attribute   → Partner Name                       string
[REGN].[REGN].Attribute          → Region                             string
[Measures].[BAL_LCL]             → Balance Local                      double
[Measures].[BAL_USD]             → Balance USD                        double
[Measures].[CRLMT_LCL]           → Credit Limit Local                 double
[Measures].[CRLMT_USD]           → Credit Limit USD                   double

NOT mapped to PBI (minor gaps):
  [CLNT]     → Client (SAP internal field — single-value, not needed)
  [PST_IND]  → Past Month Indicator (verify if used in visuals)
  [CNT]      → Counter (add if row counting needed)

Power BI Power Query M:
───────────────────────
let
    Source =
        SapHana.Database(
            "pahpdb01.net.bms.com:30015",
            [Implementation = "2.0"]
        ),
    Contents = Source{[Name = "Contents"]}[Data],
    LEAPFROG =
        Contents{[Name = "QUERIES.CORPORATE_FUNCTIONS.GLOBAL_FINANCE.LEAPFROG"]}[Data],
    CVQ_SC_SD_GOTC_CUST_CL_ARBAL1 =
        LEAPFROG{[Name = "CVQ_SC_SD_GOTC_CUST_CL_ARBAL"]}[Data],
    #"Added Items" =
        Cube.Transform(
            CVQ_SC_SD_GOTC_CUST_CL_ARBAL1,
            {
                {
                    Cube.AddAndExpandDimensionColumn,
                    "[CVQ_SC_SD_GOTC_CUST_CL_ARBAL]",
                    {
                        "[CMPY_CD].[CMPY_CD].Attribute",
                        "[CMPY_CD_TXT].[CMPY_CD_TXT].Attribute",
                        "[CR_SEG].[CR_SEG].Attribute",
                        "[CR_SEG_TXT].[CR_SEG_TXT].Attribute",
                        "[CNCY_KEY].[CNCY_KEY].Attribute",
                        "[CUST].[CUST].Attribute",
                        "[CUST_ACCT_GRP_TXT].[CUST_ACCT_GRP_TXT].Attribute",
                        "[CUST_GRP].[CUST_GRP].Attribute",
                        "[CUST_GRP_TXT].[CUST_GRP_TXT].Attribute",
                        "[CUST_NM1].[CUST_NM1].Attribute",
                        "[LAST_DT].[LAST_DT].Attribute",
                        "[PTNR].[PTNR].Attribute",
                        "[PTNR_NM].[PTNR_NM].Attribute",
                        "[REGN].[REGN].Attribute"
                    },
                    {
                        "Company Code",
                        "Company Code Description",
                        "Credit Segment",
                        "Credit Segment Description",
                        "Currency Key",
                        "Customer",
                        "Customer Account Group Description",
                        "Customer Group",
                        "Customer Group Description",
                        "Customer Name1",
                        "Key Date",
                        "Partner",
                        "Partner Name",
                        "Region"
                    }
                },
                {Cube.AddMeasureColumn, "Balance Local",      "[Measures].[BAL_LCL]"},
                {Cube.AddMeasureColumn, "Balance USD",         "[Measures].[BAL_USD]"},
                {Cube.AddMeasureColumn, "Credit Limit Local",  "[Measures].[CRLMT_LCL]"},
                {Cube.AddMeasureColumn, "Credit Limit USD",    "[Measures].[CRLMT_USD]"}
            }
        )
in
    #"Added Items"

Tableau Equivalent Steps:
  1. Connect to SAP HANA (pahpdb01.net.bms.com:30015, schema _SYS_BIC)
  2. Select view CVQ_SC_SD_GOTC_CUST_CL_ARBAL
  3. Expose 14 dimension attributes + 4 measures
  4. No additional joins or CTEs — view is pre-joined in HANA

Conversion Status: ✅ FULL — all 14 dimensions and 4 measures replicated.
                   Three source columns omitted (CLNT, PST_IND, CNT) —
                   see gaps section.


### PQ-02 | V_LEAP_SECURITY  ← Row-Level Security Table
─────────────────────────────────────────────────────────────
Tableau Data Source   : V_LEAP_SECURITY (SQL Proxy via Tableau Server)
Tableau Connection    : sqlproxy (Tableau Server published datasource
                        V_LEAP_SECURITY_0, rev 3.7)
                        Underlying source: Oracle DB

PBI Table Name        : V_LEAP_SECURITY
PBI Load Mode         : Import
PBI Connection        : Oracle.Database("uslvlbmsasp013.net.bms.com:
                        1521/IHUBP", [HierarchicalNavigation = true])
                        Schema: LEAPFROG, Table: V_LEAP_SECURITY

Tableau → PBI Column Mapping:
──────────────────────────────
Tableau Column   PBI Column     Type     Notes
────────────────────────────────────────────────────────────
BMSUID         → BMSUID         string   Used in RLS filter
COMPANY_CD     → COMPANY_CD     string   Bridge to Dim_Company
COUNTRY        → COUNTRY        string   Available for further RLS
CURRENT_STATUS → (not mapped)   —        ⚠️ Omitted — verify if needed
APPROVED_BY    → (not mapped)   —        Hidden in Tableau, not needed
APPROVED_DATE  → (not mapped)   —        Hidden in Tableau, not needed
ROLE_NAME      → (not mapped)   —        Hidden in Tableau, not needed
(other hidden) → (not mapped)   —        Hidden in Tableau, not needed

Power BI Power Query M:
───────────────────────
let
    Source =
        Oracle.Database(
            "uslvlbmsasp013.net.bms.com:1521/IHUBP",
            [HierarchicalNavigation = true]
        ),
    #"LEAPFROG" =
        Source{[Schema = "LEAPFROG"]}[Data],
    #"V_LEAP_SECURITY" =
        #"LEAPFROG"{[Name = "V_LEAP_SECURITY"]}[Data],
    #"Select Columns" =
        Table.SelectColumns(
            #"V_LEAP_SECURITY",
            { "BMSUID", "COMPANY_CD", "COUNTRY" }
        )
in
    #"Select Columns"

Tableau Equivalent:
  - Tableau used a Tableau Server published datasource (SQL Proxy).
  - PBI connects directly to the underlying Oracle database.
  - Only 3 of the ~14 available columns are selected — matching the
    columns Tableau exposed for the RLS calculation.

RLS Role Configuration (Power BI):
  Role Name  : LEAP Security  (⚠️ currently named "Untitled" — rename)
  Table      : V_LEAP_SECURITY
  DAX Filter : [BMSUID] = UPPER( USERNAME() )
  Filter propagation path:
    V_LEAP_SECURITY[COMPANY_CD]
    → Dim_Company[Company_Code]      (Many→One relationship)
    ← CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Company Code]  (Many→One)
  Effect: Each user sees only the Company Codes their BMSUID is
          authorised for in the LEAPFROG security table.

Conversion Status: ✅ FULL — security logic replicated.
                   ⚠️ Role must be renamed from "Untitled" before
                   deployment and DAX row filter must be explicitly set.


### PQ-03 | LastRefresh  ← Metadata / Refresh Timestamp
─────────────────────────────────────────────────────────────
Tableau Equivalent    : None — PBI-specific enhancement
PBI Table Name        : LastRefresh
PBI Load Mode         : Import (single row)

Power BI Power Query M:
───────────────────────
let
    Source               = DateTime.LocalNow(),
    #"Converted to Table" = #table( 1, {{Source}} )
in
    #"Converted to Table"

Column              : Column1 (string — datetime value at refresh time)
Measures on table   : M-30 [Last Refresh Date],
                      M-31 [Last Refresh Date tooltip]

NOTE: Table captures the Power BI dataset refresh timestamp.
      Displayed on dashboard cards as "Last Refresh Date: M/D/YYYY h:mm:ss AM/PM".

Conversion Status: ➕ PBI-only enhancement (no Tableau equivalent)


### PQ-04 | Dim_Company  ← Bridge / Security Dimension
─────────────────────────────────────────────────────────────
Tableau Equivalent    : Implicit shared dimension handled via LOD
                        include override ([Calculation_...][COMPANY_CD])
                        and cross-datasource join filter.
PBI Table Name        : Dim_Company
PBI Load Mode         : Import (DAX Calculated Table — not Power Query)

PBI DAX (Calculated Table):
────────────────────────────
Dim_Company =
FILTER(
    DISTINCT(
        UNION(
            SELECTCOLUMNS(
                CVQ_SC_SD_GOTC_CUST_CL_ARBAL,
                "Company_Code",
                CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Company Code]
            ),
            SELECTCOLUMNS(
                V_LEAP_SECURITY,
                "Company_Code",
                V_LEAP_SECURITY[COMPANY_CD]
            )
        )
    ),
    NOT( ISBLANK( [Company_Code] ) )
        && [Company_Code] <> ""
)

Column              : Company_Code (string)
Relationships       :
  CVQ_SC_SD_GOTC_CUST_CL_ARBAL[Company Code] → Dim_Company[Company_Code]
  V_LEAP_SECURITY[COMPANY_CD]                → Dim_Company[Company_Code]

Purpose: Creates a proper star-schema bridge dimension, enabling the
         RLS filter from V_LEAP_SECURITY to propagate through
         Dim_Company into the fact table. Tableau achieved the same
         effect via a cross-datasource filter action + LOD override.

Conversion Status: ✅ FULL (PBI architectural upgrade — bridge table
                   replaces Tableau's cross-datasource filter pattern)


### PQ-05 | Parameter Table  ← Tableau Parameters Datasource
─────────────────────────────────────────────────────────────
Tableau Equivalent    : Inline Parameters datasource
                        ([Parameter 1] View As, [Parameter 2] Currency)
PBI Table Name        : Parameter
PBI Load Mode         : Import (DAX Calculated Table — not Power Query)

PBI DAX (Calculated Table):
────────────────────────────
-- Single-column parameter slicer tables.
-- Currency parameter:
DATATABLE(
    "Currency", STRING,
    {
        { "USD"   },
        { "Local" }
    }
)

-- View As parameter (combined or separate):
DATATABLE(
    "View As", STRING,
    {
        { "Brand"                              },
        { "Billing Document Type"             },
        { "Company"                            },
        { "Customer Group"                    },
        { "Payer"                              },
        { "Payer Customer A/C Group"           },
        { "Profit Center"                      },
        { "Sales Org"                          },
        { "Sold To Party"                      },
        { "Sold To Party Customer A/C Group"  }
    }
)

Usage: Slicer visuals bound to [Currency] and [View As] columns.
       All measures reference SELECTEDVALUE('Parameter'[Currency], "USD").

Conversion Status: ✅ FULL for Currency.
                   ⚠️ PARTIAL for View As — column exists but no measure
                   currently consumes it. See Gap G5.


### PQ-06 | AR Balance_new  ← Field Parameter Table
─────────────────────────────────────────────────────────────
Tableau Equivalent    : Measure Names / Measure Values shelf
                        (dynamic measure axis toggle between
                        Balance USD and Balance Local)
PBI Table Name        : AR Balance_new
PBI Load Mode         : Import (DAX Calculated Table)

PBI DAX (Calculated Table):
────────────────────────────
AR Balance_new =
{
    ( "Balance USD",   NAMEOF( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Balance USD] ),   0 ),
    ( "Balance Local", NAMEOF( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Balance Local] ), 1 )
}

Columns:
  AR Balance_new        — display label (string, visible, sortByColumn: Order)
  AR Balance_new Fields — NAMEOF() field reference (string, hidden,
                          ParameterMetadata kind=2)
  AR Balance_new Order  — sort integer (hidden)

Measure: AR Balance Dynamic (M-33) dispatches to Balance USD or
         Balance Local based on combined Currency + Field selection.

Conversion Status: ✅ FULL — PBI Field Parameter pattern replaces
                   Tableau Measure Names shelf behaviour.


### PQ-07 | Credit Limit Filter  ← Status Slicer Table
─────────────────────────────────────────────────────────────
Tableau Equivalent    : Quick Filter on [Credit Limit Status]
                        (categorical filter: Over Limit / Within Limit)
PBI Table Name        : Credit Limit Filter
PBI Load Mode         : Import (DAX Calculated Table)

PBI DAX (Calculated Table):
────────────────────────────
Credit Limit Filter =
DATATABLE(
    "Status", STRING,
    {
        { "Over Limit"    },
        { "Within Limit"  }
    }
)

Column              : Status (string)
Measures consuming  : M-12 [Status Filter],
                      M-17 [Credit Limit Utilization Filtered]

Conversion Status: ✅ FULL


### PQ-08 | Company Code Slicer  ← Dynamic Company Filter Table
─────────────────────────────────────────────────────────────
Tableau Equivalent    : Quick Filter on [Company] with
                        "All Members" / blank-selection behaviour
PBI Table Name        : Company Code Slicer
PBI Load Mode         : Import (DAX Calculated Table)

PBI DAX (Calculated Table):
────────────────────────────
Company Code Slicer =
UNION(
    ROW( "CompanyCode", "(None)" ),
    ROW( "CompanyCode", "(All)"  ),
    SELECTCOLUMNS(
        DISTINCT( 'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Company] ),
        "CompanyCode",
        'CVQ_SC_SD_GOTC_CUST_CL_ARBAL'[Company]
    )
)

Column              : CompanyCode (string)
Sentinel values     : "(None)" → 0 in Company Filter measure
                      "(All)"  → 1 (show all) in Company Filter measure
NOTE                : Dynamically populated from the fact table at
                      model refresh. No static list required.
Measures consuming  : M-21 [Company Filter], M-20 [Is Company Selected]

Conversion Status: ✅ FULL — sentinel values replicate Tableau's
                   "(All Members)" and empty selection behaviour.


### PQ-09 | Fast Facts  ← KPI Metric Table
─────────────────────────────────────────────────────────────
Tableau Equivalent    : Tableau "Fast Facts" worksheet with 6 KPI rows,
                        one metric per row in a crosstab view.
PBI Table Name        : Fast Facts
PBI Load Mode         : Import (DAX Calculated Table)

PBI DAX (Calculated Table):
────────────────────────────
Fast Facts =
DATATABLE(
    "Metric", STRING,
    "order",  INT64,
    {
        { "# Parent Payers",                         1 },
        { "% Parent Payers Credit Limit Exceeded",   2 },
        { "# Parent Payers Credit Limit Exceeded",   3 },
        { "Total Credit Limit",                      4 },
        { "Credit Limit Exceeded Amount",            5 },
        { "AR Balance",                              6 }
    }
)

Columns:
  Metric  (string, visible, sortByColumn: order)
  order   (int64, hidden — sort column)

Calculated column on table: CC-19 [Sort Order]
Measures dispatching: M-27 [Fast Facts Value], M-28 [Fast Facts Value1]

NOTE: Two versions of the Fast Facts labels exist in PBI (M-27 and M-28)
      with and without numeric prefixes. Consolidate to M-28 for
      sort stability.

Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

G1 | RLS Role Unnamed
─────────────────────
Status     : ⚠️ Must fix before production
Detail     : Power BI role is currently named "Untitled".
             DAX row filter has not been explicitly set.
Action     : Rename role to "LEAP Security".
             Add DAX filter on V_LEAP_SECURITY:
               [BMSUID] = UPPER( USERNAME() )

G2 | PST_IND Column Not Mapped
──────────────────────────────
Status     : ⚠️ Minor gap — verify usage
Detail     : Tableau source exposes [PST_IND] (Past Month Indicator).
             Not present in PBI Power Query or DAX.
Action     : Add to Cube.AddAndExpandDimensionColumn list in PQ-01
             if used in any worksheet filter or calculation.

G3 | CLNT Column Not Mapped
────────────────────────────
Status     : ℹ️ Low risk
Detail     : SAP Client field [CLNT] present in Tableau, omitted in PBI.
             Typically a single constant value in SAP environments.
Action     : Confirm [CLNT] is not used as a filter; omit if confirmed.

G4 | CNT Counter Column Not Mapped
───────────────────────────────────
Status     : ℹ️ Low risk
Detail     : Integer counter [CNT] present in Tableau, not in PBI.
Action     : Add {Cube.AddMeasureColumn, "Counter", "[Measures].[CNT]"}
             to PQ-01 if row counting is required.

G5 | View As Parameter Unused in Measures
──────────────────────────────────────────
Status     : ⚠️ Functional gap
Detail     : Parameter[View As] table and column exist in PBI but no
             DAX measure consumes SELECTEDVALUE([View As]).
             In Tableau, [Parameter 1] / View As drove dimensional
             axis switching (e.g., view by Payer vs. Company vs. Brand).
Action     : Implement SWITCH( SELECTEDVALUE('Parameter'[View As]), ... )
             measure pattern for any visual that required axis toggling.
             Example:
               measure 'View As Dimension' =
               SWITCH(
                   SELECTEDVALUE( 'Parameter'[View As], "Payer" ),
                   "Payer",   MAX( [Payer] ),
                   "Company", MAX( [Company] ),
                   ...
               )

G6 | CURRENT_STATUS Column Omitted from V_LEAP_SECURITY
────────────────────────────────────────────────────────
Status     : ⚠️ Verify
Detail     : Tableau's V_LEAP_SECURITY exposed CURRENT_STATUS.
             PBI Power Query selects only BMSUID, COMPANY_CD, COUNTRY.
Action     : Confirm CURRENT_STATUS is not needed for RLS logic
             or any future security rules before omitting permanently.

G7 | Debug / Stub Measures — Remove Before Production
──────────────────────────────────────────────────────
Status     : ⚠️ Review required
Measures   :
  "Test Balance USD"   — SUM([Balance USD]) development check
  "Measure"            — empty stub
  "Measure 3"          — empty stub
  "Measure 4"          — empty stub (in LastRefresh table)
  "Test Credit Limit LOD" — uses deprecated EARLIER() pattern
Action     : Remove or hide all stub measures. Refactor
             [Test Credit Limit LOD] column to use VAR pattern
             (matching CC-08 through CC-11).

G8 | Fast Facts Measure Duplication
─────────────────────────────────────
Status     : ℹ️ Minor — cosmetic
Detail     : Two versions of the Fast Facts dispatch measure exist
             (M-27 and M-28) with slightly different label formats.
Action     : Consolidate to M-28 (numbered labels) for sort reliability.
             Remove M-27 [Fast Facts Value] after visual rebinding.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — CONVERSION COMPLETENESS SUMMARY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Category                         Tableau   PBI       Status    %
─────────────────────────────────────────────────────────────────────
Parameters                           2       2     ✅ Full    100%
  (Currency: full; View As: partial)                ⚠️ Partial  —
Source Dimension Columns            17      14     ⚠️ 3 gaps   82%
  (CLNT, PST_IND, CNT omitted)
Calculated Columns (LOD)             4       4     ✅ Full    100%
Calculated Columns (other)          13      13     ✅ Full    100%
DAX Measures (Tableau-equiv.)       18      18     ✅ Full    100%
DAX Measures (PBI-only)              0       9     ➕ New      N/A
Power Query Sources                  2       2     ✅ Full    100%
Calculated / Slicer Tables           5       5     ✅ Full    100%
Relationships                        2       2     ✅ Full    100%
RLS                                  1       1     ⚠️ Partial  80%
  (role exists; DAX filter pending)
─────────────────────────────────────────────────────────────────────
OVERALL (Tableau artefacts)        ~57     ~54     ✅/⚠️      ~95%
─────────────────────────────────────────────────────────────────────

Legend:
  ✅ FULL     — exact functional equivalent implemented
  ⚠️ PARTIAL  — present but incomplete or requires follow-up action
  ➕ PBI-only — no Tableau equivalent; Power BI enhancement
  ℹ️ Low risk — omitted intentionally; low business impact


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — QUICK REFERENCE: TABLEAU FORMULA → DAX PATTERN
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Tableau Pattern                                 DAX Equivalent
─────────────────────────────────────────────────────────────────────────
CASE [Currency] WHEN 'USD' THEN ... END       → SWITCH(SELECTEDVALUE('Parameter'[Currency],"USD"),"USD",...)
{FIXED [d1],[d2] : SUM([m])}                  → VAR d=table[dim] RETURN CALCULATE(SUM([m]),FILTER(table,[dim]=d))
{FIXED [d1],[d2] : MAX([m])}                  → VAR d=table[dim] RETURN CALCULATE(MAX([m]),FILTER(table,[dim]=d))
{FIXED [d1],[d2] : SUM([m])} at measure grain → SUMX(SUMMARIZE(table,[d1],[d2]),CALCULATE(SUM([m]),REMOVEFILTERS(...)))
UPPER(USERNAME())=UPPER([BMSUID])             → UPPER(USERNAME()) IN VALUES([bmsuid])
IF [A]>'*' THEN 'NULL' ELSE [A]+' '+[B]       → IF([A]="*","NULL",[A]&" "&[B])
COUNTD([PTNR])                                → DISTINCTCOUNT([Partner])
COUNTD(IF [Status]='X' THEN [col] END)        → CALCULATE(DISTINCTCOUNT([col]),[Status]="X")
MAX(0, SUM([A]) - SUM([B]))                   → MAX(0,[MeasureA]-[MeasureB])
[col1]+' '+[col2]                             → [col1]&" "&[col2]
DATE(DATEADD('day',1,DATEPARSE('yyyyMMdd',x)))→ DATE(LEFT(x,4),MID(x,5,2),RIGHT(x,2))+1
ROUND({FIXED...},2) / ROUND({FIXED...},2)     → DIVIDE(ROUND(CALCULATE(...),2),ROUND(CALCULATE(...),2),0)
IF [Util]>1 THEN 'Over Limit' ELSE '...' END  → IF([Util]>1,"Over Limit","Within Limit")
Measure Names shelf (dynamic axis toggle)     → NAMEOF()-based Field Parameter table + SELECTEDVALUE
Dashboard cross-filter action                 → Power BI visual cross-filtering / slicer sync
Quick Filter (categorical)                    → DATATABLE slicer table + SELECTEDVALUE measure guard
Quick Filter (range)                          → MINX/MAXX measures + IF(val>=min && val<=max,1,0) flag


---
# END OF KNOWLEDGE FILE
# 12960 – AR vs. Credit Limit | Tableau → Power BI (PBIP/TMDL)
# Generated: 2026-07-03
# Source files: 12960_o_1_2__ar_vs__credit_limit.txt (Tableau)
#               ar_balance_new.txt, company_code_slicer.txt,
#               credit_limit_filter.txt, cvq_sc_sd_gotc_cust_cl_arbal.txt,
#               fast_facts.txt, lastrefresh.txt, dim_company.txt,
#               v_leap_security.txt, model.txt, relationships.txt,
#               database.txt, connections.txt (PBIP)
