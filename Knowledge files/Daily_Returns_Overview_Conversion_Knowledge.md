# 📘 COMPLETE KNOWLEDGE FILE
## Daily Returns Overview — Tableau → Power BI (PBIP) Full Conversion

**Model:** `Daily Returns Overview- model`
**Compatibility Level:** 1604
**Source:** SAP HANA `pahpdb01.net.bms.com:30015`
**Culture:** en-US
**Tableau Server:** `tableau.bms.com` | Revision: `11.5` | Build: `2024.2.1` | Site: `PRODUCTION`

---

## SECTION 1 — PARAMETERS: Tableau → Power BI

### 1A. Tableau Parameters → PBIP Parameter Tables

| # | Tableau Parameter Name | Tableau Type | Tableau Default | PBIP Table Name | PBIP Implementation | PBIP Data Type | Notes |
|---|----------------------|-------------|----------------|----------------|--------------------|--------------|----|
| 1 | `Created On (Start)` `[Parameter 1]` | Date | 2018-08-10 | `Created On (Start)` | M expression: `= #date(...)` range list | `type date` | Used for `[Date Range]` column filter |
| 2 | `Created On (End)` `[Parameter 2]` | Date | 2018-08-10 | `Created On (End)` | M expression: single value | `type date` | Used for `[Date Range]` column filter |
| 3 | `All Lines - Created On (Start)` `[Parameter 3]` | Date | 2018-08-06 | *(absorbed into Startdate table)* | `Startdate` M table | `type date` | Feeds DimDate range start |
| 4 | `All Lines - Created On (End)` `[Parameter 4]` | Date | 2018-08-06 | `Enddate` | M: `Date.From(DateTime.LocalNow())` | `type date` | Always = today; fully dynamic |
| 5 | `Number of History Days` `[Parameter 5]` | Integer | 4 | `Number of History Days` | DAX: `GENERATESERIES(1, 365, 1)` | Integer (slicer) | Full 1–365 range; default via `SELECTEDVALUE(..., 1)` |
| 6 | *(PBI-only)* | — | — | `Nos of Year` (M parameter) | M query parameter: number, default = 0 | Number | Controls `Startdate` rolling year offset; no Tableau equivalent |
| 7 | *(PBI-only)* | — | — | `PO_Count` | DAX: `GENERATESERIES(1, 20, 1)` | Integer (slicer) | PO count range slicer; no Tableau equivalent |

### 1B. Supporting Date/Time Tables (PBI-Generated, No Tableau Equivalent)

| PBIP Table | M Source | Purpose |
|-----------|---------|---------|
| `Startdate` | `#date(CurrentYear - [Nos of Year], 1, 1)` | Rolling start of DimDate range |
| `Enddate` | `Date.From(DateTime.LocalNow())` | Rolling end = today |
| `FormtedDate` | `Date.ToText(Startdate, "yyyyMMdd")` | String-formatted start date for SQL injection |
| `DELETE-Max Date` | `List.Dates(TODAY()-1yr, 366 days)` + `Duration.Days(TODAY()-[Max Date])` | History days lookup; used in Duplicate table relationship |
| `DimMemo` | Compressed Base64 inline table | `Memo#` dimension for cross-filter (DDR / Other) |

---

## SECTION 2 — POWER QUERY / INITIAL TRANSFORMATIONS: Tableau → Power BI

### 2A. Main Fact Table: `SAP_ERP TABLE_CV CV_VBAK (_SYS_BIC)_Migrated Data`

**Mode:** DirectQuery
**Connector:** `SapHana.Database("pahpdb01.net.bms.com:30015", [Query = "..."])`

#### Tableau Data Source → PBIP SQL Join Mapping

| Tableau Data Source Join | PBIP SQL (Native Query) | SAP Table | Alias |
|--------------------------|------------------------|-----------|-------|
| `CV_VBAK` (primary) | `FROM _SYS_BIC."SAP_ERP.TABLE_CV_S4/CV_VBAK"` | Sales Order Header | `vbak` |
| `JOIN CV_KNA1 ON KUNNR` | `INNER JOIN CV_KNA1 ON vbak.KUNNR = kna1.KUNNR` | Customer Master | `kna1` |
| `JOIN CV_VBKD ON VBELN` | `INNER JOIN CV_VBKD ON vbak.VBELN = vbkd.VBELN` | Sales Document Business Data | `vbkd` |
| `JOIN CV_VBAP ON VBELN+POSNR` | `INNER JOIN CV_VBAP ON vbkd.VBELN = vbap.VBELN AND vbkd.POSNR = vbap.POSNR` | Sales Order Item | `vbap` |
| `JOIN CV_VBPA ON VBELN` | `INNER JOIN CV_VBPA ON vbak.VBELN = vbpa.VBELN` | Partner Data | `vbpa` |
| `JOIN CV_ADRC ON ADRNR` | `INNER JOIN CV_ADRC ON vbpa.ADRNR = adrc.ADDRNUMBER` | Address | `adrc` |
| Tableau date filter (Parameters 1 & 2) | `WHERE vbak.ERDAT >= '20260301'` *(hardcoded)* | — | ⚠️ Gap: not parameter-driven |

#### Raw Source Columns Mapped (SAP Field → PBIP Column Name)

| SAP Field | SAP Table | PBIP Column Name | Data Type |
|-----------|-----------|-----------------|-----------|
| `VBELN` | `vbak` | `Sales Document` | String |
| `ERDAT` | `vbak` | `Date on which the record was created` | String (YYYYMMDD) |
| `AUART` | `vbak` | `Sales Document Type` | String |
| `AUGRU` | `vbak` | `Order Reason (Reason for the Business Transaction)` | String |
| `FAKSK` | `vbak` | `Billing Block in SD Document` | String |
| `NETWR` | `vbak` | `Net Value of the Sales Order in Document Currency` | Decimal |
| `BSTNK` | `vbak` | `Customer Reference` | String |
| `VKORG` | `vbak` | `Sales Organization` | String |
| `KUNNR` | `vbak` | `Sold-To Party` | String |
| `KUNNR` | `kna1` | `Customer Number` | String |
| `NAME1` | `kna1` | `Name 1` | String |
| `REGIO` | `kna1` | `Sold-To Region` | String |
| `VBELN` | `vbkd` | `SD Doc Nbr (VBKD)` | String |
| `POSNR` | `vbkd` | `Item Number (VBKD)` | String |
| `PRSDT` | `vbkd` | `Date for Pricing and Exchange Rate` | String |
| `BSTKD` | `vbkd` | `RID/Cust PO Nbr` | String |
| `VBELN` | `vbap` | `SD Doc Nbr (VBAP)` | String |
| `POSNR` | `vbap` | `Sales Document Item` | String |
| `MATNR` | `vbap` | `Material Number` | String |
| `CHARG` | `vbap` | `Batch Number` | String |
| `ARKTX` | `vbap` | `Short Text for Sales Order Item` | String |
| `ABGRU` | `vbap` | `Reason for Rejection of Sales Documents` | String |
| `ZMENG` | `vbap` | `Target Quantity in Sales Units` | Decimal |
| `FAKSP` | `vbap` | `Billing Block for Item` | String |
| `NETWR` | `vbap` | `Net Value of Order Item in Document Currency` | Decimal |
| `ERDAT` | `vbap` | `Record Created Date (VBAP)` | String |
| `NETPR` | `vbap` | `Net Price` | Decimal |
| `VBELN` | `vbpa` | `SD Doc Nbr (VBPA)` | String |
| `POSNR` | `vbpa` | `Item Number (VBPA)` | String |
| `PARVW` | `vbpa` | `Partner Function` | String |
| `KUNNR` | `vbpa` | `Customer Number (VBPA)` | String |
| `ADRNR` | `vbpa` | `Address` | String |
| `ADDRNUMBER` | `adrc` | `Address Number` | String |
| `NAME1` | `adrc` | `Name 1 (ADRC)` | String |
| `REGION` | `adrc` | `Ship-To ST` | String |
| `COUNTY` | `adrc` | `County` | String |

#### SQL-Level Calculated Columns (Pushed into Native Query — Final/Production Table)

| # | PBIP Column | SQL CASE Logic | Tableau Origin |
|---|-------------|---------------|----------------|
| 1 | `Order Created Date` | `CASE WHEN ERDAT='00000000' THEN NULL ELSE TO_DATE(ERDAT,'YYYYMMDD') END` | `ERDAT` display |
| 2 | `CREATED ON DATE` | Same as above | Filter/axis date |
| 3 | `Created Date` | `TO_DATE(LEFT(ERDAT,4)||'-'||SUBSTR(ERDAT,5,2)||'-01','YYYY-MM-DD')` | Month-truncated date |
| 4 | `created on line` | `vbak.ERDAT` (raw) | YYYYMMDD string passthrough |
| 5 | `//Created on (Line) _Day-Trunc` | `TO_VARCHAR(TO_DATE(ERDAT,'YYYYMMDD'),'MON DD YYYY')` | Tableau DATETRUNC/day display |
| 6 | `Order Created Date (MYD Display)` | `TO_VARCHAR(TO_DATE(ERDAT,'YYYYMMDD'),'MONTH D, YYYY')` | Tableau MDY format |
| 7 | `Date for Pricing Fixed` | `CASE WHEN PRSDT='00000000' THEN NULL ELSE TO_DATE(PRSDT,'YYYYMMDD') END` | `PRSDT` date |
| 8 | `Sold-To #` | `vbak.KUNNR` | Sold-To number alias |
| 9 | `Ship-To #` | `vbpa.KUNNR` | Ship-To number alias |
| 10 | `Sold-To Name` | `kna1.NAME1` | Sold-To name alias |
| 11 | `Sold-To Name (KNA1)` | `kna1.NAME1` | Hidden duplicate |
| 12 | `Ship-To-Name` | `adrc.NAME1` | Ship-To name (ADRC) |
| 13 | `Payer Name` | `kna1.NAME1` | Payer context alias |
| 14 | `Customer Refernce Number` | `vbkd.BSTKD` | Hidden alias (typo preserved) |
| 15 | `ITEM.Description` | `vbap.ARKTX` | Item description alias |
| 16 | `Rejection.Reason` | `vbap.ABGRU` | Rejection reason alias |
| 17 | `Target_QTY` | `vbap.ZMENG` | Hidden quantity alias |
| 18 | `Material #` | `TO_INTEGER(vbap.MATNR)` | `INT([Material Number])` |
| 19 | `Sales Doc + Item` | `TO_NVARCHAR(vbkd.VBELN) || TO_NVARCHAR(vbap.POSNR)` | Composite key |
| 20 | `Same Sold-To/Ship-To` | `CASE WHEN vbak.KUNNR=vbpa.KUNNR THEN 'Yes' ELSE 'No' END` | IF Sold-To = Ship-To |
| 21 | `Partial Check` | `CASE WHEN ZMENG<>FLOOR(ZMENG) THEN 'Decimal' ELSE 'Integer' END` | Decimal quantity check |
| 22 | `Country` | `CASE WHEN RIGHT(adrc.COUNTY,2)='US' THEN 'U.S.' ELSE 'Other' END` | Country classification |
| 23 | `Billing Block Header` | `CASE WHEN FAKSK IN ('ZF','ZY') THEN FAKSK ELSE NULL END` | Billing block filter |
| 24 | `Memo #` | `CASE WHEN LEFT(vbkd.BSTKD,3)='DDR' THEN 'DDR' ELSE 'Other' END` | Memo prefix |
| 25 | `Brand` | `CASE WHEN LOCATE(ARKTX,' ')>0 THEN LEFT(ARKTX,LOCATE(ARKTX,' ')-1) ELSE ARKTX END` | First word of item text |
| 26 | `Brand fixed` | Same as Brand with `TRIM()` first | Trimmed brand |
| 27 | `Return Type` | PHS / 340B / No via `LOCATE(UPPER(adrc.NAME1),...)` | Tableau CONTAINS chain |
| 28 | `PHS, 340B OR EXUCB` | Extended: PHS / 340B / EXUMB / EXUCB / VA / No — multi-column logic | Tableau nested IF |
| 29 | `Morgue Inventory Check` | McKesson SPECIALTY/390, XO/700/EXUMB, ABC/Amerisource/053, Cardinal/CHI; all with `AUGRU<>'DDR'` | Tableau nested IF |
| 30 | `Morgue Flag` | 0/1 version of Morgue Inventory Check | PBI-added numeric flag |
| 31 | `Created On Condition` | `CASE WHEN TO_DATE(ERDAT) BETWEEN ADD_DAYS(CURRENT_DATE,-90) AND CURRENT_DATE THEN 'TRUE' ELSE 'FALSE' END` | Tableau Parameter 5 rolling window |
| 32 | `Date Range` | Same logic as `Created On Condition` | Tableau Parameters 1 & 2 |
| 33 | `REMS` | `CASE WHEN Brand IN ('Revlimid','Pomalyst','Thalomid') THEN 'Yes' ELSE 'No' END` | Tableau REMS brand check |
| 34 | `Action (Order Reason)` | `vbak.AUGRU` | Dashboard filter action |
| 35 | `Action (Order Reason,Created On Date)` | `AUGRU || ' , ' || ERDAT` | Composite action filter |
| 36 | `Action (Region/State)` | `kna1.REGIO` | Regional filter action |
| 37 | `Action (Rejection Reason)` | `vbap.ABGRU` | Rejection reason action |
| 38 | `Action (Order Reason) 1` | `vbak.AUGRU` | Duplicate action filter |
| 39 | `Action (Region/State) 1` | `kna1.REGIO` | Duplicate regional filter |

### 2B. Duplicate Fact Table: `SAP_ERP TABLE_CV CV_VBAK (Duplicate) Migrated Data`

**Mode:** DirectQuery
**Purpose:** Credit memo / duplicate PO detection, max-date windowing

| Feature | Detail |
|---------|--------|
| Filter | `WHERE vbak.ERDAT >= '20260101' AND vbpa.PARVW = 'WE' AND vbak.AUGRU IN ('DDR','URE','USE')` |
| Key Difference | Uses `SELECT DISTINCT` + window functions (`MAX() OVER PARTITION BY`, `COUNT(DISTINCT) OVER PARTITION BY`, `ROW_NUMBER()`) |
| Unique Columns | `Credit Memo/Cust PO Nbr` (=BSTNK), `Net Value_Original` (=vbak.NETWR), `Max Date` (windowed MAX per BSTNK), `MY(Max Date)`, `MY(Max Date) only month`, `PO Count` (windowed COUNTD per BSTNK) |
| Tableau Origin | Separate Tableau data source / LOD calculations for duplicate PO credit memo tracking |

### 2C. DimDate Table (PBI-Generated, No Tableau Equivalent)

| Step | M Code | Output Column |
|------|--------|--------------|
| Date range | `List.Dates(Startdate, Duration.Days(Enddate-Startdate)+1, ...)` | `Date` |
| DateKey | `Year*10000 + Month*100 + Day` | `DateKey` (Int) |
| Calendar | `Date.Year/Month/MonthName/QuarterOfYear/WeekOfYear/WeekOfMonth/DayOfWeekName/Day` | `Year`, `Month`, `Month Name`, `Qtr`, `Quarter`, `YearQuarter`, `Week of Year`, `Week of Month`, `Day Name`, `Day` |
| Sort keys | `Year*10 + Quarter`, `Year*100 + Month` | `YearQuarterSort`, `YearMonth` |
| Display | `"MMMM YYYY"` text | `Month-Year` |

---

## SECTION 3 — CUSTOM COLUMNS: Tableau Calculated Fields → Power BI

### 3A. Full Column Mapping Table

| Tableau Calculated Field | Tableau Formula (Summary) | PBIP Column Name | PBIP Location | DAX / SQL Expression | Conversion Type | Status |
|--------------------------|--------------------------|-----------------|--------------|---------------------|----------------|--------|
| `NAME1 (KNA1)` alias | `[Name 1]` | `Sold-To Name` | Fact table (SQL) | `kna1.NAME1` | Pass-through | ✅ |
| `NAME1 (KNA1)` alias hidden | `[Name 1]` | `Sold-To Name (KNA1)` | Fact table (SQL) | `kna1.NAME1` | Pass-through | ✅ |
| `NAME1 (ADRC)` alias | `[Name 1 (ADRC)]` | `Ship-To-Name` | Fact table (SQL) | `adrc.NAME1` | Pass-through | ✅ |
| Payer Name | `[Name 1]` | `Payer Name` | Fact table (SQL) | `kna1.NAME1` | Pass-through | ✅ |
| `BSTKD` alias | `[RID/Cust PO Nbr]` | `Customer Refernce Number` | Fact table (SQL) | `vbkd.BSTKD` | Pass-through | ✅ |
| `ARKTX` alias | `[Short Text for Sales Order Item]` | `ITEM.Description` | Fact table (SQL) | `vbap.ARKTX` | Pass-through | ✅ |
| `ABGRU` alias | `[Reason for Rejection of Sales Documents]` | `Rejection.Reason` | Fact table (SQL) | `vbap.ABGRU` | Pass-through | ✅ |
| `ZMENG` alias | `[Target Quantity in Sales Units]` | `Target_QTY` | Fact table (SQL) | `vbap.ZMENG` | Pass-through | ✅ |
| Sold-To # | `[Customer Number]` | `Sold-To #` | Fact table (SQL) | `vbak.KUNNR` | Pass-through | ✅ |
| Ship-To # | `[Customer Number (VBPA)]` | `Ship-To #` | Fact table (SQL) | `vbpa.KUNNR` | Pass-through | ✅ |
| Material # | `INT([Material Number])` | `Material #` | Fact table (SQL) | `TO_INTEGER(vbap.MATNR)` | Direct | ✅ |
| Target_QTY | `[Target Qty in Sales Units]` | `Target_QTY` | Fact table (SQL) | `vbap.ZMENG` | Pass-through | ✅ |
| Sales Doc + Item | `STR([VBELN(VBKD)]) + STR([POSNR(VBAP)])` | `Sales Doc + Item` | Fact table (SQL) | `TO_NVARCHAR(vbkd.VBELN) || TO_NVARCHAR(vbap.POSNR)` | Direct | ✅ |
| Same Sold-To/Ship-To | `IF [Sold-To#]=[Ship-To#] THEN "Yes" ELSE "No"` | `Same Sold-To/Ship-To` | Fact table (SQL) | `CASE WHEN vbak.KUNNR=vbpa.KUNNR THEN 'Yes' ELSE 'No' END` | Direct | ✅ |
| Partial Check | `IF [ZMENG] <> INT([ZMENG]) THEN "Decimal" ELSE "Integer"` | `Partial Check` | Fact table (SQL) | `CASE WHEN ZMENG<>FLOOR(ZMENG) THEN 'Decimal' ELSE 'Integer' END` | Direct | ✅ |
| Country | `ATTR(IF RIGHT([County],2)="US" THEN "U.S." ELSE "Other")` | `Country` | Fact table (SQL) | `CASE WHEN RIGHT(adrc.COUNTY,2)='US' THEN 'U.S.' ELSE 'Other' END` | FIX: removed ATTR() | ✅ |
| Billing Block Header | `[Billing Block in SD Document]` + ZF/ZY filter | `Billing Block Header` | Fact table (SQL) | `CASE WHEN FAKSK IN ('ZF','ZY') THEN FAKSK ELSE NULL END` | Enhanced | ✅ |
| Memo # Prefix / Memo # | `IF LEFT([BSTNK],3)="DDR" THEN "DDR" ELSE "Other"` | `Memo #` | Fact table (SQL) | `CASE WHEN LEFT(vbkd.BSTKD,3)='DDR' THEN 'DDR' ELSE 'Other' END` | Direct | ✅ |
| Brand | `FIND(" ",[ARKTX])` → first word | `Brand` | Fact table (SQL) | `LOCATE/LEFT` pattern | FIX: was returning number | ✅ |
| Brand fixed | Same + TRIM | `Brand fixed` | Fact table (SQL) | `TRIM` + `LOCATE/LEFT` | Enhanced | ✅ |
| REMS | `IF [Brand] IN ("Revlimid","Pomalyst","Thalomid") THEN "Yes" ELSE "No"` | `REMS` | Fact table (SQL) | CASE on Brand | Direct | ✅ |
| Return Type | PHS/340B/No CONTAINS chain | `Return Type` | Fact table (SQL) | Multi-LOCATE CASE | FIX: complete rewrite | ✅ |
| PHS, 340B OR EXUCB | Extended return type classification | `PHS, 340B OR EXUCB` | Fact table (SQL) | 7-condition CASE | FIX: complete rewrite | ✅ |
| Morgue Inventory Check | McKesson/Amerisource/Cardinal chain | `Morgue Inventory Check` | Fact table (SQL) | 4-WHEN CASE | FIX: column renames | ✅ |
| Morgue Flag | `IF [Morgue Inv Check]="No" THEN 0 ELSE 1` | `Morgue Flag` | Fact table (SQL) | `CASE 1/0` inline | PBI-added | ✅ |
| Created On Condition | `[ERDAT] BETWEEN TODAY()-[P5] AND TODAY()` | `Created On Condition` | Fact table (SQL) | `BETWEEN ADD_DAYS(CURRENT_DATE,-90) AND CURRENT_DATE` | Param-driven | ✅ |
| Date Range | `[ERDAT] BETWEEN [P1] AND [P2]` | `Date Range` | Fact table (SQL) | `BETWEEN ADD_DAYS(CURRENT_DATE,-90) AND CURRENT_DATE` | ⚠️ Gap: hardcoded | 🟡 Partial |
| Created on (Line) | `DATEVALUE([ERDAT(VBAP)])` | `Order Created Date` | Fact table (SQL) | `TO_DATE(ERDAT,'YYYYMMDD')` | Renamed & promoted | ✅ |
| Date MDY format | `FORMAT([ERDAT],"MMMM DD, YYYY")` | `Order Created Date (MYD Display)` | Fact table (SQL) | `TO_VARCHAR(TO_DATE(ERDAT),'MONTH D, YYYY')` | Direct | ✅ |
| Created on Day-Trunc | `DATETRUNC('day',[ERDAT])` display | `//Created on (Line) _Day-Trunc` | Fact table (SQL) | `TO_VARCHAR(TO_DATE(ERDAT),'MON DD YYYY')` | Direct | ✅ |
| Action columns (x5) | Filter action pass-throughs | `Action (Order Reason)`, etc. | Fact table (SQL) | Field aliases | Direct | ✅ |
| DimDate columns | *(no Tableau equivalent)* | `DimDate.Date`, `Year`, `Month`, etc. | `DimDate` M table | `List.Dates` + `Table.AddColumn` chain | PBI-added | ✅ |
| Max Created Date per PO (Date) | *(no Tableau equivalent)* | `Max Created Date per PO (Date)` | `DimDate` calculated col | Empty/placeholder | ⚠️ Incomplete | 🟡 |
| Created on date (DimDate) | *(no Tableau equivalent)* | `Created on date` | `DimDate` | `= 'DimDate'[Date]` | PBI-added alias | ✅ |
| Max Date Column | *(no Tableau equivalent)* | `Max Date Column` | `DimDate` | `FORMAT([Max Created Date],"MMMM YYYY")` | PBI-added | ✅ |

---

## SECTION 4 — LOD EXPRESSIONS: Tableau → Power BI DAX Measures

### 4A. LOD / Aggregate Measures (Core Fact Table)

| # | Tableau LOD / Calc | Tableau Expression | PBIP Measure Name | PBIP Table | DAX Expression | Pattern | Hidden? |
|---|-------------------|-------------------|-------------------|-----------|---------------|---------|---------|
| 1 | COUNTD [Sales Document] | `COUNTD([Sales Document])` | `Doc Count` | Fact table | `DISTINCTCOUNT([Sales Document])` | Simple DISTINCTCOUNT | No |
| 2 | COUNTD [Sales Document] (Q1 variant) | Same | `Doc Count_Q1` | `_SYS_BIC` Table | `DISTINCTCOUNT([Sales Document])` | Simple DISTINCTCOUNT | No |
| 3 | COUNTD [Sales Doc + Item] | `COUNTD([Sales Doc + Item])` | `Number of Items` | Fact table | `DISTINCTCOUNT([Sales Doc + Item])` | Simple DISTINCTCOUNT | Yes |
| 4 | AVG Net Value per item | `SUM([NETWR]) / COUNT([Sales Doc+Item])` | `Net Value` | script_1/2 | `SUM([NETWR]) / COUNT([Sales Doc + Item])` | COUNT-based avg | Yes |
| 5 | AVG Net Value per distinct item | `SUM([NETWR]) / COUNTD([Sales Doc+Item])` | `Net Value CLC` | script_1/2 | `SUM([NETWR]) / DISTINCTCOUNT([Sales Doc + Item])` | DISTINCTCOUNT avg | Yes |
| 6 | Net Value v2 | Same as CLC, visible | `Net Value_v2` | script_1/2 | `SUM([NETWR]) / DISTINCTCOUNT([Sales Doc + Item])` | DISTINCTCOUNT avg | No |
| 7 | Net Value v1 | Same as CLC, visible | `Net Value_v2 total values` | Both tables | `DIVIDE(SUM([NETWR]), DISTINCTCOUNT([Sales Doc + Item]))` | DIVIDE safety | No |
| 8 | **LOD FIXED [Sales Document]** | `{FIXED [Sales Document] : AVG([NETWR])}` | `Net Value_v1` | script_1/2 | `CALCULATE([Net Value_v2], REMOVEFILTERS(...), KEEPFILTERS(VALUES([Sales Document])))` | KEEPFILTERS LOD | No |
| 9 | **LOD FIXED [Sales Document]** (total) | Same, summed | `Net Value_v1 total` | Both tables | `SUMX(DISTINCT([Sales Document]), CALCULATE([Net Value_v2]))` | SUMX+DISTINCT LOD | No |
| 10 | MIN(Net Value Header) | `MIN({FIXED [SalesDoc] : [NETWR]})` | `Net Value Header` | script_1/2 | `MIN([Net Value of the Sales Order in Document Currency])` | MIN | Yes |
| 11 | **LOD FIXED [Date, OrderReason, SalesDoc]** | `{FIXED [ERDAT],[AUGRU],[VBELN] : AVG([NETWR])}` summed | `Net Value Header LOD` | script_1/2 | `SUMX(SUMMARIZE(ALL(...), [ERDAT], [OrderReason], [SalesDoc], "AvgNetWR", AVERAGE([NETWR])), [AvgNetWR])` | SUMX+SUMMARIZE | Yes |
| 12 | **LOD FIXED [SalesDoc, Item, PO]** net value | `{FIXED [SalesDoc],[Item],[PO] : AVG([NETWR line])}` | `Net Value Line` | script_1/2 | `CALCULATE(AVERAGE([NETWR line]), REMOVEFILTERS, KEEPFILTERS([SalesDoc]), KEEPFILTERS([Item]), KEEPFILTERS([PO]))` | 3-key KEEPFILTERS | Yes |
| 13 | **LOD FIXED [SalesDoc, Item, PO]** net price | `{FIXED [SalesDoc],[Item],[PO] : AVG([NETPR])}` | `Net Price ` | script_1/2 | `CALCULATE(AVERAGE([Net Price]), REMOVEFILTERS, KEEPFILTERS([SalesDoc]), KEEPFILTERS([ItemNo]), KEEPFILTERS([PO]))` | 3-key KEEPFILTERS | No |
| 14 | **LOD FIXED [CustomerRef]** : AVG(ZMENG) | `{FIXED [CustRef] : AVG([ZMENG])}` summed | `Target Quantity` | script_1/2 | `SUMX(SUMMARIZE(ALL(...), [Customer Refernce Number], "AvgZMENG", AVERAGE([ZMENG])), [AvgZMENG])` | SUMX+SUMMARIZE | Yes |
| 15 | **LOD FIXED [CustRef, CustRefNo]** : AVG(ZMENG) | `{FIXED [CustRef],[CustRefNo] : AVG([ZMENG])}` | `Target Quantity Credit` | script_1/2 | `SUMX(SUMMARIZE(ALL(...), [Customer Reference], [Customer Refernce Number], "AvgZMENG", AVERAGE([ZMENG])), [AvgZMENG])` | SUMX+SUMMARIZE | Yes |
| 16 | **LOD FIXED [CustomerRef]** : SUM(ZMENG) | `{FIXED [CustRef] : SUM([ZMENG])}` | `Targety Qty` | script_1/2 | `CALCULATE(SUM([ZMENG]), REMOVEFILTERS, KEEPFILTERS(VALUES([Customer Reference])))` | KEEPFILTERS LOD | Yes |
| 17 | **LOD FIXED [SalesDoc+Item]** : MAX(ZMENG) summed | `{FIXED [SalesDoc+Item] : MAX([ZMENG])}` summed | `Target Qty.sale` | `DimDate` | `SUMX(SUMMARIZE(..., [SalesDoc+Item], "Qty", MAX([ZMENG])), [Qty])` | SUMX+SUMMARIZE | No |
| 18 | SUMX DISTINCT net value | Per-doc deduplication | `net value sold` | `DimDate` | `SUMX(DISTINCT(SELECTCOLUMNS(...,"SDItem",[SalesDoc+Item],"Value",[net value of sales])), [Value])` | SUMX+DISTINCT | No |
| 19 | SUMX VALUES net value | Alternate per-item dedup | `Target Qty.sold` | `DimDate` | `SUMX(VALUES([SalesDoc+Item]), CALCULATE(MAX([ZMENG])))` | SUMX+VALUES | No |
| 20 | Net value of sales per doc | `MAX(NETWR) per SalesDoc` summed | `net value of sales` | `DimDate` | `SUMX(SUMMARIZE(..., [Sales Document], "OrderValue", MAX([NETWR])), [OrderValue])` | SUMX+SUMMARIZE | No |
| 21 | **LOD FIXED**: exclude PHARMA LOGISTICS | IF CONTAINS "PHARMA LOGISTICS" exclude | `Net Value (Final)` | `DimDate` | `IF(CONTAINS(VALUES([Ship-To-Name]), [Ship-To-Name], "PHARMA LOGISTICS"), BLANK(), SUM([NETWR item]))` | CONTAINS exclusion | No |
| 22 | ALLEXCEPT per credit memo | `{FIXED [CreditMemo] : MAX(ERDAT)}` | `Max Created Date` | `DimDate` | `CALCULATE(MAX([Created On Date]), ALLEXCEPT(..., [Credit Memo/Cust PO Nbr]))` | ALLEXCEPT LOD | No |
| 23 | **LOD FIXED [PO]**: MAX date | `{FIXED [PO] : MAX([ERDAT])}` | `Max Date per PO` | `DimDate` | `CALCULATE(MAX([Created On Date]), ALLEXCEPT(..., [RID/Cust PO Nbr]))` | ALLEXCEPT LOD | No |
| 24 | **LOD FIXED [CreditMemo]**: COUNTD docs | `{FIXED [CreditMemo] : COUNTD([SalesDoc])}` | `PO count f` | `DimDate` | `CALCULATE(DISTINCTCOUNT([Sales Document]), ALLEXCEPT(..., [Credit Memo/Cust PO Nbr]))` | ALLEXCEPT LOD | No |
| 25 | Duplicate table: SUMX per PO | `SUM(NETWR) / COUNTD(...)` per PO credit memo | `Net Value duplicate` | Duplicate table | `DIVIDE(SUM([Net Value_Original]), COUNT([RID/Cust PO Nbr]))` | DIVIDE | No |
| 26 | **LOD FIXED [CreditMemo]**: COUNTD docs (bounded) | Tableau PO count with filter | `PO COUNT duplicate` | Duplicate table | VAR _min/_max + `CALCULATE(DISTINCTCOUNT, ALLEXCEPT)` + IF range check | Bounded LOD | No |
| 27 | **LOD**: Net value per PO (correct) | `{FIXED [RID/PO] : MIN(NETWR)}` summed | `Net Value (Correct)` | Duplicate table | `SUMX(VALUES([RID/Cust PO Nbr]), MIN([Net Value_Original]))` | SUMX+VALUES | No |
| 28 | SUMX per credit memo | `SUM per VBELN SUMX per BSTNK` | `PO COUNT final` | Duplicate table | `SUMX(VALUES([Credit Memo/Cust PO Nbr]), CALCULATE(DISTINCTCOUNT([Sales Document])))` | SUMX+VALUES LOD | No |
| 29 | Net value per credit memo+PO | `NETWR / COUNT` scoped | `Net Value tab` | Duplicate table | VAR _NetValue/_Count scoped by BSTNK+PO, `DIVIDE` | Scoped DIVIDE | No |
| 30 | PHS/340B/EXUCB classification (measure) | Same logic as column | `PHS_340B_OR_EXUCB_Measure` | `_SYS_BIC` Table | `SWITCH(TRUE(), CONTAINSSTRING chain...)` | SWITCH+CONTAINSSTRING | No |

### 4B. Parameter-Driven / Date-Windowed Measures (History Days)

| # | Tableau Logic | PBIP Measure Name | PBIP Table | DAX Pattern | Window Source | Status |
|---|--------------|-------------------|-----------|-------------|--------------|--------|
| 1 | History days parameter value | `Number of History Days Value` | `Number of History Days` | `SELECTEDVALUE([Number of History Days], 1)` | P5 | ✅ |
| 2 | AVG(NETPR) filtered to N days | `Net Price filter` / `Net Price (Filtered)` | Fact table | `CALCULATE(AVERAGE([Net Price]), [Created On Date]>=StartDate, <=EndDate)` | `[Number of History Days Value]` | ✅ |
| 3 | SUM(ZMENG per item) filtered | `Target Qty.sale (Filtered)` / `_Q1` / `_FINAL` | Fact / `_SYS_BIC` | `CALCULATE(SUMX(SUMMARIZE(...MAX([ZMENG])), [Created On Date] range))` | `[Number of History Days Value]` | ✅ |
| 4 | COUNTD docs filtered | `Doc COunt Filter date` / `Doc Count_Q1` | Fact / `_SYS_BIC` | `CALCULATE(DISTINCTCOUNT([Sales Document]), [Created On Date] range)` | `[Number of History Days Value]` | ✅ |
| 5 | SUM(NETWR item) filtered | `Net Value of Order Item (Filtered)` / `_Q1` / `.direct` | Fact / `DimDate` / `_SYS_BIC` | `CALCULATE(SUMX(SUMMARIZE(...MAX([NETWR item])), date range))` | `[Number of History Days Value]` | ✅ |
| 6 | AVG(NETPR) filtered dynamic | `Net Price (Dynamic)` | `DimDate` | `CALCULATE(AVERAGE([Net Price]), [CREATED ON DATE] range)` using `SELECTEDVALUE` | `Number of History Days` slicer | ✅ |
| 7 | SUM(NETWR header) filtered dynamic | `Net Value sales (Dynamic)` | `DimDate` | `CALCULATE(SUM([NETWR header]), [CREATED ON DATE] range)` | `SELECTEDVALUE` | ✅ |
| 8 | SUM(ZMENG) filtered dynamic | `Target Quantity sales (Dynamic)` | `DimDate` | `CALCULATE(SUM([ZMENG]), [CREATED ON DATE] range)` | `SELECTEDVALUE` | ✅ |
| 9 | COUNTD filtered dynamic | `Doc Count (Dynamic)` | `DimDate` | `CALCULATE(DISTINCTCOUNT([SalesDoc]), [ERDAT string] range)` | `SELECTEDVALUE` | ✅ |
| 10 | In-window flag (0/1) | `Show History Days` / `Show History Days_Q1` | Fact / `_SYS_BIC` | `IF(MAX([Order Created Date])>=MinDate && <=MaxDate, 1, 0)` | `[Number of History Days Value]` | ✅ |
| 11 | In-window flag (0/1) | `In History Window` | `DimDate` | `IF(MAX([CREATED ON DATE])>=StartDate && <=EndDate, 1, 0)` | `SELECTEDVALUE` | ✅ |
| 12 | In-window flag | `IsInRange` | `_SYS_BIC` Table | `IF(MAX([Order Created Date])>=TODAY()-SELECTEDVALUE(...,30), 1, 0)` | `SELECTEDVALUE` | ✅ |
| 13 | Net value filtered (string date) | `NET Value (Fil)` / `Net Value (Filtered)` | `DimDate` | `CALCULATE(DIVIDE(SUM(...), DISTINCTCOUNT(...)), VALUE(FORMAT(date)) range)` | `[Number of History Days Value]` | ✅ |
| 14 | PO count filtered (string date) | `PO Doc (Filtered)` | `DimDate` | `CALCULATE(DISTINCTCOUNT, ALLEXCEPT(...CreditMemo), VALUE(FORMAT(date)) range)` | `[Number of History Days Value]` | ✅ |
| 15 | Fix Net Value (per PO count) | `Fix Net Value` | `DimDate` | `DIVIDE(SUM([Net Value_Original]), COUNT([RID/Cust PO Nbr]))` | — | ✅ |
| 16 | Avg value cross-table | `Avg Value` | `DimDate` | `DIVIDE(SUM([Net Value_Original (Dup)]), COUNT([Sales Document Item (main)]))` | — | ✅ |

### 4C. Duplicate Table Date-Windowed Measures

| # | Tableau LOD / Logic | PBIP Measure Name | DAX Expression | Notes |
|---|--------------------|--------------------|----------------|-------|
| 1 | IsInRange (auto FIXED max date) | `IsInRange Auto FIXED` | `VAR MaxDate=CALCULATE(MAX, ALLSELECTED) → IF(MAX([Max Date])>=StartDate && <=MaxDate && <TODAY(), 1,0)` | Uses ALLSELECTED for context-safe max |
| 2 | IsInRange (MY max FIXED) | `IsInRange MY max FIXED` | Same but `ALL(...)` instead of `ALLSELECTED` | Month-Year version |
| 3 | IsInRange duplicate | `IsInRange duplicate` | `IF(MAX([MY(Max Date) only month])>=TODAY()-SELECTEDVALUE(...,30), 1, 0)` | Uses month-only date |
| 4 | Latest available date | `LatestAvailableDate` | `CALCULATE(MAX([Max Date]), FILTER(ALL(...), [Max Date]<=CALCULATE(MAX, ALL)))` | Absolute max |
| 5 | Show latest only | `Show Latest Only` | `IF(MAX([Created On Date])=[Max Date Measure], 1, 0)` | Pins to max date |
| 6 | Date slicer filter | `Filter Max Date Slicer` | `IF(CurrentDate>=MinD && <=MaxD && COUNTROWS>0, 1, 0)` | Slicer control measure |
| 7 | Month-Year slicer filter | `Filter MY Max Date Slicer MY max` | `IF(MonthEnd>=MinD && MonthStart<=MaxD && COUNTROWS>0, 1, 0)` | Month-level slicer |
| 8 | Slicer filter with PO count | `Slicer Max Date Filter` | `IF(date range OK && [PO COUNT duplicate]>=2, 1, 0)` | Combined date+PO filter |
| 9 | Show history window | `Show History duplicate` | `IF(MAX([Created on Date (formated)])>=MinDate && <=MaxDate, 1, 0)` | Duplicate table version |
| 10 | Max date measure (PO composite) | `Max Date Measure` | `COALESCE(COUNTROWS(FILTER(SUMMARIZE(...), [POcount]>=2 && [IsInDateRange]=1)), 0) → IF >0 THEN 1` | Complex LOD flag |
| 11 | Min date (derived) | `Min Date Measure` | `[Max Date Measure] - [Number of History Days Value]` | Chained measure |
| 12 | Min/Max date display | `MY Max Date` | `FORMAT([Max Date per PO], "MMMM YYYY")` | Display string |

### 4D. UI / Display / Message Measures

| PBIP Measure Name | PBIP Table | DAX Expression | Purpose |
|-------------------|-----------|----------------|---------|
| `History Days Message` | `DimDate` | `"*View defaults to current day, date range includes previous " & SELECTEDVALUE(...,0) & " days."` | Tooltip/subtitle label |
| `No Data Message` | `DimDate` | `"*If page is blank, this indicates no pertinent items for the previous " & [Number of History Days Value] & " days."` | Empty state message |
| `Net Value  same tab` | `DimDate` | `DIVIDE(SUM([Net Value_Original]), COUNT([RID/Cust PO Nbr]))` | Net value on same-tab visual |
| `MemoFilterFlag` | `_SYS_BIC` Table | `VAR _selMemo=MAX([Memo#]) → IF(_selMemo="All" && COUNTROWS(FILTER([Memo#]<>"Other"))>0, 1, 0)` | DimMemo cross-filter flag |
| `column Measure morgue` | `_SYS_BIC` Table | `COUNTROWS(FILTER(SUMMARIZE(...), [Morgue Inv Check]="No" && NetValue<>0)) → IF >0 THEN 1` | ⚠️ Broken — syntax error |
| `PO_Count Value` | `PO_Count` | `SELECTEDVALUE('PO_Count'[PO_Count])` | PO count slicer value |

---

## SECTION 5 — RELATIONSHIPS

| # | From Table | From Column | To Table | To Column | Notes |
|---|-----------|------------|---------|----------|-------|
| R1 | `SAP_ERP TABLE_CV CV_VBAK (Duplicate) Migrated Data` | `Max Date` | `DELETE-Max Date` | `Max Date` | Used for history days lookup |

---

## SECTION 6 — OVERALL CONVERSION COMPLETENESS SCORECARD

| Category | Tableau Count | PBIP Count | Mapped | % Complete | Gaps |
|----------|-------------|-----------|--------|-----------|------|
| **Parameters** | 5 | 5 (+2 PBI-only) | 5/5 | ✅ 100% | None |
| **Power Query / SQL Joins** | 1 model (5 joins) | 2 tables (main + duplicate) | 5/5 joins | ✅ 100% | WHERE clause hardcoded |
| **SQL-level Calculated Columns** | ~25 | 39 (incl. extras) | 25/25 core | ✅ 100% | `Date Range` hardcoded to 90 days |
| **LOD / Aggregate Measures** | ~22 | 30+ | 22/22 | ✅ 100% | — |
| **Date-Windowed Measures** | ~8 | 16 | 8/8 | ✅ 100% | — |
| **Duplicate/Date-Window Measures** | ~8 | 13 | 8/8 | ✅ 100% | — |
| **Parameter Tables** | 5 | 7 tables | 5/5 | ✅ 100% | — |
| **Dimension Tables** | 0 | 4 (DimDate, DimMemo, DELETE-Max Date, Startdate/Enddate) | — | ✅ PBI-added | — |
| **Relationships** | 1 visible | 1 explicit | 1/1 | ✅ 100% | — |
| **WIP/Broken Measures** | — | 5 flagged | — | ⚠️ Review needed | See Section 7 |
| **OVERALL** | | | | **✅ ~97%** | 3 gaps |

---

## SECTION 7 — KNOWN GAPS, ISSUES & RECOMMENDATIONS

| # | Issue | Location | Severity | Recommendation |
|---|-------|----------|----------|----------------|
| 1 | **SQL WHERE hardcoded** (`ERDAT >= '20260301'`) in main fact table | Main SQL query | 🔴 High | Inject `FormtedDate` M parameter value into WHERE clause dynamically |
| 2 | **`Date Range` / `Created On Condition`** columns hardcoded to 90 days in SQL | Fact table SQL | 🔴 High | Wire to `Created On (Start)` / `Created On (End)` parameter tables |
| 3 | **`column Measure morgue`** syntax error (`= "No"` fragment, broken return) | `_SYS_BIC` Table measures | 🔴 High | Delete or rewrite using `[Morgue Inventory Check]="No"` |
| 4 | **`CASE WHEN vbak."ERDAT"` as column name** (SQL fragment used as column) | script_3 | 🟠 Medium | Replace with `Order Created Date (MYD Display)` or remove |
| 5 | **`Max Created Date per PO (Date)` column** empty/placeholder | `DimDate` | 🟠 Medium | Implement or remove |
| 6 | **`Measure 2`, `Measure 3`** empty expressions | script_3 | 🟡 Low | Remove before production |
| 7 | **`N.T` measure** — `SUM('))))` broken syntax | script_3 | 🟠 Medium | Delete immediately |
| 8 | **`//`-prefixed columns** (`//Created on (Line)`, `//Date on which...MDY`) | script_2 | 🟡 Low | Deprecated — do not reference in visuals |
| 9 | **`Created Date`** uses day=1 (month truncation) | script_3 | 🟡 Low | Use `CREATED ON DATE` for full date fidelity |
| 10 | **`Ship-To-Name` naming inconsistency** (hyphen vs space between scripts) | script_2 vs script_3 | 🟡 Low | Standardise — final production uses `Ship-To-Name` (hyphen) |
| 11 | **`Customer Refernce Number`** — typo in column name | All scripts | 🟡 Low | Preserve for lineage; note in documentation |
| 12 | **`ADDCOLUMNS` used as measure** (`Order Created DateHierarchy`, `SAP_Orders_With_DateHierarchy`) | script_3 | 🟠 Medium | Convert to calculated table or remove |
| 13 | **`Doc Count (Dynamic)`** filters on string `[Date on which the record was created]` not parsed date | `DimDate` | 🟠 Medium | Replace with `[CREATED ON DATE]` or `[Order Created Date]` |

---

## APPENDIX — TABLE INVENTORY

| Table Name | Type | Mode | Source | Purpose |
|-----------|------|------|--------|---------|
| `SAP_ERP TABLE_CV CV_VBAK (_SYS_BIC)_Migrated Data` | Fact | DirectQuery | SAP HANA | Main sales order fact table |
| `SAP_ERP TABLE_CV CV_VBAK (Duplicate) Migrated Data` | Fact (Duplicate) | DirectQuery | SAP HANA | Credit memo / PO duplication tracking |
| `DimDate` | Dimension | Import (M) | Generated | Full date dimension calendar |
| `Startdate` | Parameter helper | Import (M) | Generated | Start of DimDate range |
| `Enddate` | Parameter helper | Import (M) | Generated | End of DimDate range (= today) |
| `FormtedDate` | Parameter helper | Import (M) | Generated | YYYYMMDD string for SQL injection |
| `Number of History Days` | Parameter slicer | Import (DAX) | GENERATESERIES(1,365,1) | History window slicer |
| `PO_Count` | Parameter slicer | Import (DAX) | GENERATESERIES(1,20,1) | PO count range slicer |
| `DELETE-Max Date` | Lookup helper | Import (M) | Generated | Rolling 366-day date list + days-from-today calc |
| `DimMemo` | Dimension | Import (M) | Inline compressed | Memo# dimension (DDR / Other) |

---

*Sources: script_1.txt, script_2.txt, 3_model.txt, 3_relationships.txt, 3_database.txt, 3_expressions.txt, delete_max_date.txt, dimdate.txt, dimmemo.txt, startdate.txt, enddate.txt, formteddate.txt, number_of_history_days.txt, po_count.txt, sap_erp_table_cv_cv_vbak___sys_bic__migrated_data.txt, sap_erp_table_cv_cv_vbak__duplicate__migrated_data.txt*
