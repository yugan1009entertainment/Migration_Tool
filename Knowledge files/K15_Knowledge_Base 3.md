# K15 Knowledge Base Documentation

> **Generated:** 2026-06-12
> **Scope:** BMS Finance – Accounts Receivable / Cash Clearing
> **Tools:** K15 1A (Tableau) · K15 2B (Power BI)
> **Domain:** Global Finance – Order to Cash (OTC)

---

## Table of Contents

1. [Overview](#1-overview)
2. [K15 1A – Tableau Workbook](#2-k15-1a--tableau-workbook)
   - 2.1 [Workbook Metadata](#21-workbook-metadata)
   - 2.2 [Data Sources](#22-data-sources)
   - 2.3 [Source Columns – CVQ_FI_AR_GOTC_CUST_CLRD_CASH](#23-source-columns--cvq_fi_ar_gotc_cust_clrd_cash)
   - 2.4 [Calculated Fields](#24-calculated-fields)
   - 2.5 [Parameters](#25-parameters)
   - 2.6 [Dashboards & Worksheets](#26-dashboards--worksheets)
   - 2.7 [Dashboard Actions](#27-dashboard-actions)
   - 2.8 [Security Model](#28-security-model)
   - 2.9 [Default Filters](#29-default-filters)
3. [K15 2B – Power BI Semantic Model](#3-k15-2b--power-bi-semantic-model)
   - 3.1 [Model Metadata](#31-model-metadata)
   - 3.2 [Tables](#32-tables)
   - 3.3 [Measures (DAX)](#33-measures-dax)
   - 3.4 [Calculated Columns](#34-calculated-columns)
   - 3.5 [Parameters](#35-parameters)
   - 3.6 [Relationships](#36-relationships)
   - 3.7 [Security](#37-security)
   - 3.8 [Local Date Tables](#38-local-date-tables)
4. [Cross-Tool Comparison](#4-cross-tool-comparison)
5. [Data Sources & Connections](#5-data-sources--connections)
6. [Glossary](#6-glossary)

---

## 1. Overview

**K15** is the BMS enterprise reporting knowledge base for the **CA-2: Cash Clearing** report. It tracks Accounts Receivable (AR) cash clearing activity across global BMS company codes. The report enables finance teams to monitor:

- **Net AR Debit Amount** – AR impact from RECO GL account group entries
- **Net Cash Cleared Amount** – Cash clearing activity from FUND GL account group entries
- **Monthly trends**, **document counts**, and **customer-level breakdowns**

The same business logic is implemented in two BI tools:
- **K15 1A** – Tableau (Production workbook on `tableau.bms.com`)
- **K15 2B** – Power BI (Semantic model, compatibility level 1601)

Both tools connect to the same underlying SAP HANA / Oracle data sources via the BMS LEAPFROG data layer.

---

## 2. K15 1A – Tableau Workbook

### 2.1 Workbook Metadata

| Property | Value |
|---|---|
| **Workbook Name** | CA-2: Cash Clearing |
| **Tableau Server** | `tableau.bms.com` |
| **Site** | PRODUCTION |
| **Repository Path** | `/t/PRODUCTION/workbooks/CA-2CashClearing_0` |
| **Revision** | 2.9 |
| **Tableau Build** | 2024.2.13 (20242.25.0710.1539) |
| **Version** | 18.1 |
| **Last Extract Update** | 06/05/2026 09:01:51 AM |
| **Font** | Arial, 8pt (worksheet), 10pt (title), 9pt (tooltip) |

---

### 2.2 Data Sources

#### Primary Fact Source – CVQ_FI_AR_GOTC_CUST_CLRD_CASH
- **Connection type:** SAP HANA (federated)
- **Server:** `pahpdb01.net.bms.com:30015`
- **Schema:** `_SYS_BIC`
- **Table:** `QUERIES.CORPORATE_FUNCTIONS.GLOBAL_FINANCE.LEAPFROG::CVQ_FI_AR_GOTC_CUST_CLRD_CASH`
- **Authentication:** Username/password (`LEAPFROG`)
- **Extract:** `.hyper` file (`federated_1409oqs1jlb0so1blb14c0184lrb`)
- **Extract storage path:** `Data/Extracts/federated_1409oqs1jlb0so1blb14c0184lrb.hyper`

#### Security Source – V_LEAP_SECURITY
- **Connection type:** Tableau Server data source proxy (`sqlproxy`)
- **Server:** `tableau.bms.com`
- **Published DS ID:** `V_LEAP_SECURITY_0` (revision 1.7)
- **Exposed columns (subset):** `BMSUID`, `COMPANY_CD`, `COUNTRY`, `CURRENT_STATUS`

---

### 2.3 Source Columns – CVQ_FI_AR_GOTC_CUST_CLRD_CASH

| Column (Remote) | Caption | Data Type | Role |
|---|---|---|---|
| `CUST_ACCT_GRP` | Customer Account Group | string | Dimension |
| `CUST_GRP` | Customer Group | string | Dimension |
| `CUST_GRP_TXT` | Customer Group Description | string | Dimension |
| `CUST_NM1` | Customer Name1 | string | Dimension |
| `CMPY_CD_TXT` | Company Code Description | string | Dimension |
| `GL_AC` | GL Account | string | Dimension |
| `PSTNG_KY` | Posting Key | string | Dimension |
| `POST_DT` | Posting Date (raw yyyyMMdd) | string | Dimension |
| `ACCT_DOC_LN` | Accounting Document Line Item | string | Dimension |
| `REGN` | Region | string | Dimension |
| `CMPY_CD` | Company Code | string | Dimension |
| `ACCT_DOC_NO` | Accounting Document Number | string | Dimension |
| `YR` | Fiscal Year | string | Dimension |
| `KEY_DT` | Key Date (raw yyyyMMdd) | string | Dimension |
| `CUNCY_KY` | Currency Key | string | Dimension |
| `DBT_CR_IND_TXT` | Debit/Credit Indicator | string | Dimension |
| `GL_AC_TXT` | G/L Account Description | string | Dimension |
| `GL_AC_GRP` | G/L Account Group | string | Dimension |
| `GL_AC_GRP_TXT` | G/L Account Group Description | string | Dimension |
| `CUST` | Customer | string | Dimension |
| `CUST_ACCT_GRP_TXT` | Customer Account Group Name | string | Dimension |
| `DOC_TYPE` | Document Type | string | Dimension |
| `CLNT` | Client | string | Dimension |
| `DOC_TYPE_TXT` | Document Type Description | string | Dimension |
| `REV_DOC_NUM` | Reversal Document Number | string | Dimension |
| `REV_IND` | Reversal Indicator | string | Dimension |
| `PSTNG_KY_TXT` | Name of the Posting Key | string | Dimension |
| `CC_DATE_UPPER` | Cash Clearing Upper Date | string | Dimension |
| `PST_IND` | Past Month Indicator | string | Dimension |
| `AMT_USD` | Amount in USD | real | Measure |
| `AMT_LCL` | Amount in LCL | real | Measure |

**Approximate cardinalities (from extract):**
- Company Codes: ~74
- Customers: ~2,274
- Accounting Documents: ~6,733
- Posting Dates: ~945
- Currency Keys: ~42
- GL Accounts: ~76
- GL Account Groups: ~14
- Regions: ~6

---

### 2.4 Calculated Fields

| Caption | Formula Summary |
|---|---|
| **Posting Date** | `DATE(DATETRUNC('day', DATEPARSE("yyyyMMdd", [POST_DT])))` – converts raw string to date |
| **Key Date** | `DATE(DATEPARSE("yyyyMMdd", [KEY_DT]))` – converts raw string to date |
| **Amount in USD/LCL** | `CASE [Parameter 1] WHEN 'Local' THEN [AMT_LCL] WHEN 'USD' THEN [AMT_USD] END` |
| **Amount** | `CASE [DBT_CR_IND_TXT] WHEN 'Debit' THEN [USD/LCL amount] WHEN 'Credit' THEN -1*[USD/LCL amount] END` – applies sign |
| **Cash Cleared Amount** | `CASE [GL_AC_GRP] WHEN 'FUND' THEN [Amount] ELSE 0 END` – FUND group only |
| **AR Debit Amount** | `CASE [GL_AC_GRP] WHEN 'RECO' THEN [Amount] ELSE 0 END` – RECO group only |
| **Net AR/Cash Impact** | `CASE [Parameter 3] WHEN 'Cash Cleared Amount' THEN [Cash Cleared Amount] WHEN 'AR Debit Amount' THEN [AR Debit Amount] END` |
| **# Acct Docs** | `{INCLUDE [Company],[ACCT_DOC_NO],YEAR([Posting Date]):COUNTD([ACCT_DOC_NO])}` – LOD count |
| **#Acct Doc Line Item** | `{INCLUDE [Company],[ACCT_DOC_LN],[ACCT_DOC_NO],YEAR([Posting Date]):COUNTD([ACCT_DOC_LN])}` |
| **Company** | `[CMPY_CD] + ' ' + [CMPY_CD_TXT]` |
| **Customer** | `[CUST] + ' ' + [CUST_NM1]` |
| **Doc. Type** | `[DOC_TYPE] + ' ' + [DOC_TYPE_TXT]` |
| **GL Account** | `[GL_AC] + ' ' + [GL_AC_TXT]` |
| **GL Account Group** | `[GL_AC_GRP] + ' ' + [GL_AC_GRP_TXT]` |
| **Posting Key** | `[PSTNG_KY] + ' ' + [PSTNG_KY_TXT]` |
| **Cust. Acct. Group** | `[CUST_ACCT_GRP] + ' - ' + [CUST_ACCT_GRP_TXT]` |
| **Customer Group** | `[CUST_GRP] + ' - ' + [CUST_GRP_TXT]` |
| **View** | CASE on Parameter 2 – returns the selected dimension field |
| **ViewBy2 (TT)** | Alias for Net AR/Cash Impact – used in tooltips |
| **Displayed Currency** | `CASE [Parameter 1] WHEN 'USD' THEN 'USD' ELSE 'Local' END` |
| **Security Calc** | `UPPER(USERNAME()) = UPPER([BMSUID])` – row-level security filter |
| **Index** | `INDEX()` – table calculation for ranking rows |

---

### 2.5 Parameters

| Parameter | Caption | Default | Values |
|---|---|---|---|
| `[Parameter 1]` | Currency | USD | USD, Local |
| `[Parameter 2]` | ViewBy | Company | Company, Customer, Debit/Credit Indicator, Doc. Type, GL Account, GL Account Group, Posting Key |
| `[Parameter 3]` | ViewBy2 | AR Debit Amount (alias: Net AR Impact) | AR Debit Amount (Net AR Impact), Cash Cleared Amount (Net Cash Impact) |

---

### 2.6 Dashboards & Worksheets

| Sheet Name | Type | Description |
|---|---|---|
| `CA-2: Cash Clearing` | Dashboard | Main dashboard |
| `CA-2.1A` | Worksheet | Bar chart – ViewBy dimension vs. Amount & # Acct Docs (dual axis), sorted DESC by amount |
| `CA-2.1B` | Worksheet | Secondary chart (referenced in filter actions) |
| `Fast Facts` | Worksheet | KPI/summary tile (referenced in filter actions) |
| `HelpIcon` | Worksheet | Navigation buttons for Support and Company Code Access links |

**CA-2.1A visual encoding:**
- **Rows:** View (dynamic dimension based on Parameter 2)
- **Columns:** Net AR/Cash Impact amount + # Acct Docs (dual axis)
- **Mark type:** Bar
- **Color:** Measure Names
- **Sort:** DESC by Amount in USD/LCL
- **Tooltip:** ViewBy label, Amount, # Acct Docs
- **Axis title (amount):** "Amount"
- **Axis title (docs):** "# Acct. Doc"

---

### 2.7 Dashboard Actions

| Action | Trigger | Source | Target | Type |
|---|---|---|---|---|
| Highlight 1 | On Select | CA-2.1A | CA-2.1A | Brush (Measure Names) |
| Filter 1 | On Select | CA-2.1A (Dashboard) | CA-2: Cash Clearing | Filter (all fields) |
| Filter 2 | On Select | CA-2.1B (Dashboard) | CA-2: Cash Clearing | Filter (all fields) |
| Filter 3 | On Select | Fast Facts (Dashboard) | CA-2: Cash Clearing | Filter (all fields) |
| Support | Click | HelpIcon | URL | `https://sites.bms.com/teams/GIADataAnalyticsResources/SitePages/Support.aspx` |
| Company Code Access | Click | HelpIcon | URL | `https://tableau.bms.com/#/site/PRODUCTION/views/SecurityAuthorization/SecurityAuthorization` |

---

### 2.8 Security Model

Row-level security is enforced via the `V_LEAP_SECURITY` data source:

- **Filter logic:** `UPPER(USERNAME()) = UPPER([BMSUID])` → only TRUE rows pass
- The security source is joined to the fact source on **Company Code** (`CMPY_CD` ↔ `COMPANY_CD`)
- Users only see data for company codes they are authorized for in the LEAPFROG security table
- `CURRENT_STATUS`, `COUNTRY`, `COMPANY_CD`, `BMSUID` are the key fields

---

### 2.9 Default Filters

The workbook applies these filters by default (shared view):

| Field | Default Filter |
|---|---|
| Past Month Indicator (`PST_IND`) | "Past 6 Months" |
| Region (`REGN`) | ECC, LACC, Other |
| Company (`Calculation_810647991399141378`) | Empty (user selection required) |
| Security Calc | TRUE (enforced) |
| All other dimensions | All members |

---

## 3. K15 2B – Power BI Semantic Model

### 3.1 Model Metadata

| Property | Value |
|---|---|
| **Compatibility Level** | 1601 |
| **Culture** | en-US |
| **Source Query Culture** | en-US |
| **Default Power BI Data Source Version** | powerBI_V3 |
| **Time Intelligence** | Enabled (`__PBI_TimeIntelligenceEnabled = 1`) |
| **Dev Mode** | Enabled |
| **Query Order** | CVQ_FI_AR_GOTC_CUST_CLRD_CASH → Refresh Table → V_LEAP_SECURITY |

---

### 3.2 Tables

#### CVQ_FI_AR_GOTC_CUST_CLRD_CASH (Fact Table)
- **Source:** SAP HANA via Power Query M
- **Server:** `pahpdb01.net.bms.com:30015` / SAP HANA schema `_SYS_BIC`
- **Query:** `QUERIES.CORPORATE_FUNCTIONS.GLOBAL_FINANCE.LEAPFROG::CVQ_FI_AR_GOTC_CUST_CLRD_CASH`
- **Load mode:** Import
- **Key columns:** Company Code, Customer, Posting Date, GL_account, GL_account_group, Doc Type, Debit/Credit Indicator, Posting Key, AR Debit Amount, Cash Cleared Amount, Amount in USD/LCL

#### V_LEAP_SECURITY (Security Table)
- **Source:** Oracle Database via Power Query M
- **Server:** `uslvlbmsasp013.net.bms.com:1521/ihubp`
- **Schema:** LEAPFROG
- **Object:** `V_LEAP_SECURITY`
- **Load mode:** Import
- **Columns:**

| Column | Data Type | Notes |
|---|---|---|
| BMSUID | string | BMS user ID |
| LAST_NAME | string | |
| FIRST_NAME | string | |
| EMAIL_ADDRESS | string | |
| APPROVED_BY | string | |
| APPROVED_DATE | dateTime | Has date variation (LocalDateTable) |
| START_DATE | dateTime | Has date variation (LocalDateTable) |
| END_DATE | dateTime | Has date variation (LocalDateTable) |
| CURRENT_STATUS | string | |
| APPROVER_IND | string | |
| LAST_UPDATE_BY | string | |
| LAST_UPDATE_DATE | dateTime | Has date variation (LocalDateTable) |
| APP_NAME | string | |
| COMPANY_CD | string | Used for RLS join |
| ROLE_NAME | string | |
| COUNTRY | string | |

#### Dim_Company (Calculated Dimension)
- **Type:** Calculated table (DAX)
- **Logic:** Union of unique Company Codes from both `CVQ_FI_AR_GOTC_CUST_CLRD_CASH[Company Code]` and `V_LEAP_SECURITY[COMPANY_CD]`, filtered to exclude blanks/empty strings
- **Column:** `Company_code`

#### Currency (Parameter Table)
- **Values:** USD, Local
- **Purpose:** User-selectable currency toggle
- **Key measures:** Amount local, Amount USD, Displayed Currency

#### ViewBy Parameter
- **Type:** Calculated table
- **Purpose:** Dynamic dimension slicer (ViewBy)
- **Values & mapped fields:**

| Label | Mapped Field |
|---|---|
| Company | `CVQ_FI_AR_GOTC_CUST_CLRD_CASH[company]` |
| Customer | `CVQ_FI_AR_GOTC_CUST_CLRD_CASH[Customer_cal]` |
| Debit/Credit Indicator | `CVQ_FI_AR_GOTC_CUST_CLRD_CASH[Debit/Credit Indicator]` |
| Doc. Type | `CVQ_FI_AR_GOTC_CUST_CLRD_CASH[Doc Type]` |
| GL Account | `CVQ_FI_AR_GOTC_CUST_CLRD_CASH[GL_account]` |
| GL Account Group | `CVQ_FI_AR_GOTC_CUST_CLRD_CASH[GL_account_group]` |
| Posting Key | `CVQ_FI_AR_GOTC_CUST_CLRD_CASH[posting_key]` |

- **Tooltip Dimension measure:** SWITCH/ISINSCOPE pattern returning `"[Label]: [Selected Value]"` for each dimension

#### ViewBy2 Parameter
- **Type:** Calculated table
- **Purpose:** Toggle between Net AR Impact and Net Cash Impact for trend chart
- **Values:**

| Label | Mapped Measure |
|---|---|
| Net AR Impact | `CVQ_FI_AR_GOTC_CUST_CLRD_CASH[AR Debit Amount]` |
| Net Cash Impact | `CVQ_FI_AR_GOTC_CUST_CLRD_CASH[Cash Cleared Amount]` |

- **Title measure:** `"Monthly Trend of " & IF(SELECTEDVALUE(order)=0, "Net AR Impact", "Net Cash Impact")`

#### Refresh Table
- **Source:** Power Query – `DateTimeZone.UtcNow()` formatted as `MM/dd/yyyy hh:mm:ss tt`
- **Measures:**
  - `refresh_date_text` = "Refresh Date:"
  - `refresh date` = `MAX('Refresh Table'[Refresh Table])`

#### DateTableTemplate (Hidden)
- Standard Power BI date template, hidden, used as base for LocalDateTables
- Hierarchy: Year → Quarter → Month → Day

---

### 3.3 Measures (DAX)

Key measures on the `CVQ_FI_AR_GOTC_CUST_CLRD_CASH` table:

| Measure | Description |
|---|---|
| **Amount in USD** | SUM of USD amount |
| **Amount in LCL** | SUM of local currency amount |
| **Amount in USD/LCL** | Returns USD or LCL based on currency slicer selection |
| **AR Debit Amount** | Net AR impact (RECO GL group, debit positive / credit negative) |
| **Cash Cleared Amount** | Net cash cleared (FUND GL group, debit positive / credit negative) |
| **AR Debit Amount matrix** | Matrix visual variant of AR Debit Amount |
| **Cash Cleared Amount Matrix** | Matrix visual variant of Cash Cleared Amount |
| **# Acct Docs** | DISTINCTCOUNT of accounting document numbers |
| **# Acct Doc Line Item** | DISTINCTCOUNT of accounting document line items |
| **# Acct Docs_month** / **# Acct_Docs_month** | Monthly accounting document count |
| **customer_cnt** / **Customer count** | Distinct customer count |
| **Amount** | Base net amount (signed: debit+, credit−) |
| **Y Index** | Row index for display ordering |
| **Company Selected** | Returns selected company code |
| **Show Message** | Flag: show user message |
| **Show table** | Flag: show data table visual |
| **Message Text** | Dynamic message text for user guidance |
| **Has Data** | Boolean: returns whether filtered data exists |
| **show_month** | Flag: show monthly breakdown |
| **Tooltip Text** | Dynamic tooltip string |
| **Tooltip info** | Extended tooltip information |
| **Key_date_measure** | Returns key date value |
| **Refresh Date Measure** | Combines refresh_date_text + refresh date |
| **Date Refresh Text** | Formatted text for refresh date display |
| **Selected Date Filter** | Returns currently selected date filter |
| **Show Visual Flag** | Controls visibility of specific visuals |
| **minimum (Posting Date)** | MIN of Posting Date |
| **maximum (Posting Date)** | MAX of Posting Date |
| **test (USERPRINCIPALNAME())** | Debug measure – returns current user's UPN |

---

### 3.4 Calculated Columns

Key calculated columns on `CVQ_FI_AR_GOTC_CUST_CLRD_CASH`:

| Column | Purpose |
|---|---|
| `company` | Display label: Company Code + Description |
| `Customer_cal` | Display label: Customer ID + Name |
| `Customer` | Formatted customer display field |
| `GL_account` | Display label: GL Account + Description |
| `GL_account_group` | Display label: GL Account Group + Description |
| `Doc Type` | Display label: Doc Type + Description |
| `posting_key` | Display label: Posting Key + Description |
| `Debit/Credit Indicator` | Debit/Credit indicator text |
| `Posting Date` | Parsed date from raw string field |
| `Posting_date` | Alternate parsed date column |
| `AR Debit Amount` | Pre-calculated AR debit column |
| `Cash Cleared Amount` | Pre-calculated cash clearing column |

---

### 3.5 Parameters

Power BI parameters mirror the Tableau parameters exactly:

| Power BI Table | Equivalent Tableau Parameter | Options |
|---|---|---|
| Currency | Parameter 1 | USD, Local |
| ViewBy Parameter | Parameter 2 | Company, Customer, Debit/Credit Indicator, Doc. Type, GL Account, GL Account Group, Posting Key |
| ViewBy2 Parameter | Parameter 3 | Net AR Impact, Net Cash Impact |

---

### 3.6 Relationships

| Relationship ID | From Table → Column | To Table → Column | Notes |
|---|---|---|---|
| `dfab0faf-...` | CVQ_FI_AR_GOTC_CUST_CLRD_CASH.[Posting Date] | LocalDateTable_56b40e09.[Date] | Date part only |
| `ada65254-...` | CVQ_FI_AR_GOTC_CUST_CLRD_CASH.[Posting_date] | LocalDateTable_b33fc482.[Date] | Date part only |
| `fd6606a3-...` | CVQ_FI_AR_GOTC_CUST_CLRD_CASH.[Company Code] | V_LEAP_SECURITY.[COMPANY_CD] | **Both directions** (many-to-many cross-filter) |
| `7ac7ccf7-...` | V_LEAP_SECURITY.[APPROVED_DATE] | LocalDateTable_fcd7a49a.[Date] | Date part only |
| `6a51e614-...` | V_LEAP_SECURITY.[START_DATE] | LocalDateTable_e84d411a.[Date] | Date part only |
| `46d25954-...` | V_LEAP_SECURITY.[END_DATE] | LocalDateTable_14cc8952.[Date] | Date part only |
| `2fbc3634-...` | V_LEAP_SECURITY.[LAST_UPDATE_DATE] | LocalDateTable_2f50768f.[Date] | Date part only |

> **Note:** The bidirectional relationship between `CVQ_FI_AR_GOTC_CUST_CLRD_CASH` and `V_LEAP_SECURITY` on Company Code enables row-level security filtering in both directions.

---

### 3.7 Security

- **Role:** `Security groups`
- **Mechanism:** Bidirectional relationship between fact table and `V_LEAP_SECURITY` on Company Code
- **RLS logic:** Users can only see rows where their company code appears in `V_LEAP_SECURITY[COMPANY_CD]`
- **Debug measure:** `test (USERPRINCIPALNAME())` – returns current user's UPN for troubleshooting

---

### 3.8 Local Date Tables

Power BI auto-generates hidden LocalDateTables for each date column. All follow the same structure:

**Columns:** Date, Year, MonthNo, Month (name), QuarterNo, Quarter ("Qtr N"), Day
**Hierarchy:** Year → Quarter → Month → Day

| LocalDateTable GUID | Bound To | Source Calendar Range |
|---|---|---|
| `56b40e09-...` | CVQ_FI_AR_GOTC_CUST_CLRD_CASH.[Posting Date] | MIN–MAX of Posting Date |
| `b33fc482-...` | CVQ_FI_AR_GOTC_CUST_CLRD_CASH.[Posting_date] | MIN–MAX of Posting_date |
| `fcd7a49a-...` | V_LEAP_SECURITY.[APPROVED_DATE] | MIN–MAX of APPROVED_DATE |
| `e84d411a-...` | V_LEAP_SECURITY.[START_DATE] | MIN–MAX of START_DATE |
| `14cc8952-...` | V_LEAP_SECURITY.[END_DATE] | MIN–MAX of END_DATE |
| `2f50768f-...` | V_LEAP_SECURITY.[LAST_UPDATE_DATE] | MIN–MAX of LAST_UPDATE_DATE |

---

## 4. Cross-Tool Comparison

| Feature | K15 1A (Tableau) | K15 2B (Power BI) |
|---|---|---|
| **Fact Source** | SAP HANA (direct + extract) | SAP HANA (import via Power Query) |
| **Security Source** | Oracle/Tableau proxy | Oracle (Power Query) |
| **RLS Mechanism** | `USERNAME() = BMSUID` filter on data source | Bidirectional relationship + Security groups role |
| **Currency Toggle** | Parameter 1 (USD/Local) | Currency table slicer |
| **ViewBy** | Parameter 2 (7 options) | ViewBy Parameter calculated table (7 options) |
| **ViewBy2 (Trend)** | Parameter 3 (Net AR/Cash Impact) | ViewBy2 Parameter calculated table |
| **AR Debit Amount** | CASE on GL_AC_GRP = 'RECO' | DAX measure on GL_account_group |
| **Cash Cleared Amount** | CASE on GL_AC_GRP = 'FUND' | DAX measure on GL_account_group |
| **# Acct Docs** | LOD COUNTD | DAX DISTINCTCOUNT |
| **Posting Date** | Parsed from raw string yyyyMMdd | Calculated column, parsed date |
| **Default Time Filter** | Past 6 Months | Selected Date Filter measure |
| **Default Region Filter** | ECC, LACC, Other | (Applied via visuals) |
| **Refresh Tracking** | Extract update timestamp | Refresh Table (UTC timestamp) |
| **Tooltip Dimension** | Tooltip fields on mark | Tooltip Dimension measure (SWITCH/ISINSCOPE) |

---

## 5. Data Sources & Connections

### SAP HANA – Primary Fact Data
| Property | Value |
|---|---|
| Server | `pahpdb01.net.bms.com` |
| Port | 30015 |
| Schema | `_SYS_BIC` |
| Table/View | `QUERIES.CORPORATE_FUNCTIONS.GLOBAL_FINANCE.LEAPFROG::CVQ_FI_AR_GOTC_CUST_CLRD_CASH` |
| Auth | Username/password – account: `LEAPFROG` |

### Oracle – Security Data
| Property | Value |
|---|---|
| Server | `uslvlbmsasp013.net.bms.com` |
| Port | 1521 |
| Service | `ihubp` |
| Schema | `LEAPFROG` |
| Object | `V_LEAP_SECURITY` |

### Tableau Server Published Data Source
| Property | Value |
|---|---|
| Server | `tableau.bms.com` |
| Site | PRODUCTION |
| DS Name | `V_LEAP_SECURITY_0` |
| Revision | 1.7 |
| Port | 443 (HTTPS) |

---

## 6. Glossary

| Term | Definition |
|---|---|
| **AR** | Accounts Receivable |
| **Cash Clearing** | The process of matching and settling open AR items against cash payments received |
| **Net AR Impact** | Net debit amount on RECO GL account group entries – represents the AR balance effect |
| **Net Cash Impact** | Net amount on FUND GL account group entries – represents actual cash cleared |
| **RECO** | GL Account Group for AR reconciliation accounts |
| **FUND** | GL Account Group for cash/bank clearing accounts |
| **LEAPFROG** | BMS internal data schema/layer in SAP HANA and Oracle used for finance reporting |
| **CVQ_FI_AR_GOTC_CUST_CLRD_CASH** | SAP HANA view: Customer Cash Cleared – GOTC (Global Order to Cash) |
| **V_LEAP_SECURITY** | Security view containing authorized users and their company code access |
| **BMSUID** | BMS User ID – used for row-level security matching |
| **Company Code** | 4-character SAP company code identifier |
| **Posting Date** | Date the accounting document was posted in SAP |
| **Key Date** | Reference date for the cash clearing run |
| **Doc Type** | SAP document type (e.g., payment, journal entry) |
| **Posting Key** | SAP 2-digit code indicating the nature of a line item (debit/credit, account type) |
| **GL Account Group** | Grouping of GL accounts (e.g., RECO, FUND) |
| **Past Month Indicator** | Calculated field bucketing records into time periods (e.g., "Past 6 Months") |
| **LOD** | Level of Detail expression (Tableau) |
| **DAX** | Data Analysis Expressions (Power BI measure language) |
| **OTC** | Order to Cash – the end-to-end business process from sales order to cash receipt |
| **GIA** | Global Internal Audit / Global Insights & Analytics (BMS) |
| **ECC / LACC** | BMS regional codes: Europe/Canada/China; Latin America/Canada/China |
