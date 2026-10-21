\# K2 — Canada Order Trend Analysis

\## Tableau → Power BI Conversion Knowledge Base

\*\*Version:\*\* k2 | \*\*Last Updated:\*\* 2026-06-08

\*\*Source Workbook:\*\* Canada Order Trend Analysis (SAP HANA, Import mode, Workbook ID 18504)

\*\*Scope:\*\* SAP HANA connector, multi-join SQL, 8 Tableau parameters, complex DAX calc columns, multi-date variations, sign-flip, brand extraction, binning, exchange rate



\---



\## 1. SOURCE: SAP HANA CONNECTION



\*\*Tableau connection (from workbook XML):\*\*

```xml

<connection class='saphana'

&#x20; server='pahpdb01.net.bms.com' port='30015'

&#x20; schema='\_SYS\_BIC' username='GFSGLOBAL03'

&#x20; authentication='username-password' />

```



\*\*Power BI M partition (CORRECT pattern — Rule M14):\*\*

```m

let

&#x20;   Source = SapHana.Database(

&#x20;       "pahpdb01.net.bms.com:30015",

&#x20;       \[Implementation = "2.0", CommandTimeout = #duration(0, 4, 0, 0)]

&#x20;   ),

&#x20;   Query = "Select to\_date(Sales\_H.ERDAT) Order\_Created\_on\_Date,

Sales\_H.VTWEG Distribution\_Channel, Sales\_H.ERNAM Order\_Created\_By,

Sales\_H.AUGRU Order\_Reason, Sales\_H.BSARK PO\_Type, Sales\_H.VBELN Sales\_Doc,

Sales\_H.AUART Sales\_Doc\_Type, Sales\_H.VKORG Sales\_Org,

Sales\_H.WAERK SD\_Doc\_Currency, Sales\_H.KUNNR Sold\_To\_Party,

Sales\_I.Sales\_Document\_Sales\_I, Sales\_I.Sales\_Document\_Item\_Sales\_I,

Sales\_I.Sales\_Doc\_Item\_Cat, Sales\_I.Net\_Value\_Order\_Doc\_Currency,

PO\_Info.PO\_Number, to\_date(PO\_Info.Customer\_Reference\_Date) Customer\_Reference\_Date,

PO\_Info.Source, cust.Country\_Key\_Cust, cust.Customer\_Number, cust.Industry\_Key,

cust.Sold\_To\_Name, Sales\_Part.Country\_Key, Sales\_Part.Name Ship\_To\_Name,

Sales\_Part.Ship\_To\_City, Sales\_Part.Ship\_to\_ST,

to\_date(bill.Billing\_Date) Billing\_Date, bill.Billing\_Doc,

bill.Billing\_Doc\_Canceled, to\_date(bill.Billing\_Doc\_Created) Billing\_Doc\_Created,

bill.Actual\_Invoice\_Quantity, bill.Billing\_Item, bill.Material\_Description,

bill.Material\_Entered, bill.Company\_Code, bill.Net\_Value\_of\_the\_Billing\_Doc,

bill.Region\_of\_Sales\_Order, bill.Billing\_Type, bill.SD\_Document\_Currency,

bill.Subtotal\_5

FROM ""SAP\_ERP.TABLE\_CV\_S4::CV\_VBAK"" Sales\_H

join (Select cust.land1 Country\_Key\_Cust, cust.kunnr Customer\_Number,

&#x20;   cust.brsch Industry\_Key, cust.name1 Sold\_To\_Name

&#x20;   from ""\_SYS\_BIC"".""SAP\_ERP.TABLE\_CV\_S4/CV\_KNA1"" Cust

&#x20;   group by cust.land1, cust.kunnr, cust.brsch, cust.name1) cust

on cust.Customer\_Number = Sales\_H.KUNNR

join (Select Sales\_I.VBELN Sales\_Document\_Sales\_I,

&#x20;   Sales\_I.POSNR Sales\_Document\_Item\_Sales\_I,

&#x20;   Sales\_I.PSTYV Sales\_Doc\_Item\_Cat, Sales\_I.NETWR Net\_Value\_Order\_Doc\_Currency

&#x20;   FROM ""SAP\_ERP.TABLE\_CV\_S4::CV\_VBAP"" Sales\_I

&#x20;   group by Sales\_I.VBELN, Sales\_I.POSNR, Sales\_I.PSTYV, Sales\_I.NETWR) Sales\_I

on Sales\_I.Sales\_Document\_Sales\_I = Sales\_H.VBELN

left join (...ship-to subquery...) sales\_part on ...

left join (...billing subquery...) bill on ...

left join (...PO info subquery...) PO\_Info on ...

where Sales\_H.VKORG in ('S035')

and to\_integer(year(to\_date(Sales\_H.ERDAT,'YYYYMMDD'))) >= to\_integer((year(CURRENT\_DATE)-3))

group by ...",

&#x20;   Result = Value.NativeQuery(Source, Query, null, \[EnableFolding=true])

in

&#x20;   Result

```



\*\*Key HANA rules:\*\*

\- Pass SOURCE ROOT to `Value.NativeQuery` — NOT `Source{\[Name="..."]}\[Data]`

\- Use `null` (not `\[]`) as third parameter

\- HANA schema/table names use double-quote escaping: `""\_SYS\_BIC"".""TABLE""`

\- Requires on-premise Data Gateway

\- Credentials: Basic (username/password)



\---



\## 2. TABLEAU PARAMETERS (8 PARAMETERS FROM WORKBOOK)



| # | Caption | Type | Default | Values |

|---|---|---|---|---|

| 1 | Currency Selection | integer | 0 | 0=Doc Currency, 1=USD |

| 2 | Top Set | integer | 10 | Range 5–25 |

| 3 | Date Selector | integer | 0 | 0=Order Creation, 1=Billing Date |

| 4 | Reporting Period | integer | 1 | 1=Month, 2=Quarter, 3=Week |

| 5 | Measure Selection | integer | 2 | 1=Invoice Amount, 2=Invoice Count |

| 6 | Minimum Order Threshold | integer | 25000 | Range min=1 |

| 7 | Threshold Select | string | "Selected Threshold" | "Selected Threshold", "Avg Yearly by Sold To Group" |

| 8 | Bottom | real | 0.0 | Any |



\*\*Power BI pattern — Parameters.tmdl:\*\*

```tmdl

table Parameters

&#x20;   lineageTag: param-table-0000-000000000001



&#x20;   column 'Currency Selector'

&#x20;       dataType: int64

&#x20;       lineageTag: param-1-key

&#x20;       summarizeBy: none

&#x20;       sourceColumn: CurrSel

&#x20;   column 'Top Set'

&#x20;       dataType: int64

&#x20;       lineageTag: param-2-key

&#x20;       summarizeBy: none

&#x20;       sourceColumn: TopSet

&#x20;   column 'Date Selector'

&#x20;       dataType: int64

&#x20;       lineageTag: param-3-key

&#x20;       summarizeBy: none

&#x20;       sourceColumn: DateSel

&#x20;   column 'Reporting Period'

&#x20;       dataType: int64

&#x20;       lineageTag: param-4-key

&#x20;       summarizeBy: none

&#x20;       sourceColumn: PeriodSel

&#x20;   column 'Measure Selection'

&#x20;       dataType: int64

&#x20;       lineageTag: param-5-key

&#x20;       summarizeBy: none

&#x20;       sourceColumn: MeasureSel

&#x20;   column 'Min Order Threshold'

&#x20;       dataType: int64

&#x20;       lineageTag: param-6-key

&#x20;       summarizeBy: none

&#x20;       sourceColumn: MinThreshold

&#x20;   column 'Threshold Select'

&#x20;       dataType: string

&#x20;       lineageTag: param-7-key

&#x20;       summarizeBy: none

&#x20;       sourceColumn: ThresholdSel

&#x20;   column 'Bottom Value'

&#x20;       dataType: double

&#x20;       lineageTag: param-8-key

&#x20;       summarizeBy: none

&#x20;       sourceColumn: BottomVal



&#x20;   partition Parameters = m

&#x20;       mode: import

&#x20;       source = ```

&#x20;               let Source = #table(

&#x20;                   type table \[CurrSel=Int64.Type, TopSet=Int64.Type, DateSel=Int64.Type,

&#x20;                       PeriodSel=Int64.Type, MeasureSel=Int64.Type, MinThreshold=Int64.Type,

&#x20;                       ThresholdSel=text, BottomVal=number],

&#x20;                   {{0, 10, 0, 1, 2, 25000, "Selected Threshold", 0.0}}

&#x20;               ) in Source

&#x20;               ```

&#x20;       annotation PBI\_NavigationStepName = Navigation

&#x20;       annotation PBI\_ResultType = Table

```



\*\*SELECTEDVALUE measures (in Measures Table.tmdl):\*\*

```tmdl

measure 'Parameter 1 Value' = SELECTEDVALUE('Parameters'\[Currency Selector], 0)

&#x20;   lineageTag: mt-meas-param-1-value

measure 'Parameter 2 Value' = SELECTEDVALUE('Parameters'\[Top Set], 10)

&#x20;   lineageTag: mt-meas-param-2-value

measure 'Parameter 3 Value' = SELECTEDVALUE('Parameters'\[Date Selector], 0)

&#x20;   lineageTag: mt-meas-param-3-value

measure 'Parameter 4 Value' = SELECTEDVALUE('Parameters'\[Reporting Period], 1)

&#x20;   lineageTag: mt-meas-param-4-value

measure 'Parameter 5 Value' = SELECTEDVALUE('Parameters'\[Measure Selection], 2)

&#x20;   lineageTag: mt-meas-param-5-value

measure 'Parameter 6 Value' = SELECTEDVALUE('Parameters'\[Min Order Threshold], 25000)

&#x20;   lineageTag: mt-meas-param-6-value

measure 'Parameter 7 Value' = SELECTEDVALUE('Parameters'\[Threshold Select], "Selected Threshold")

&#x20;   lineageTag: mt-meas-param-7-value

measure 'Parameter 8 Value' = SELECTEDVALUE('Parameters'\[Bottom Value], 0.0)

&#x20;   lineageTag: mt-meas-param-8-value

```



\---



\## 3. CUSTOM COLOR PALETTE (FROM WORKBOOK XML)



\*\*theme.json:\*\*

```json

{

&#x20; "name": "BMS",

&#x20; "dataColors": \[

&#x20;   "#595454","#a69f9f","#c0f2fb","#33d6f1","#009fba","#097789",

&#x20;   "#c5ffe6","#59ffb9","#1dce9b","#138967","#be2bbb","#dac5c5",

&#x20;   "#cb7c78","#fedcca","#fda97d","#ffeccd","#ffd186","#eee7e7",

&#x20;   "#df603a","#ba4422","#ffac25","#c37900","#772a28","#501514"

&#x20; ],

&#x20; "background": "#FFFFFF",

&#x20; "foreground": "#252423"

}

```



\---



\## 4. DATE COLUMNS AND VARIATIONS



5 date columns → 5 LocalDateTable files:



| # | Column Name | Source Column | GUID Suffix | LDT Prefix | Relationship Name |

|---|---|---|---|---|---|

| 1 | Order Created on Date | ORDER\_CREATED\_ON\_DATE | 1111-1111-1111-111111111111 | ldt1 | rel-date-order-created |

| 2 | Customer Reference Date | CUSTOMER\_REFERENCE\_DATE | 2222-2222-2222-222222222222 | ldt2 | rel-date-customer-ref |

| 3 | Billing Date | BILLING\_DATE | 3333-3333-3333-333333333333 | ldt3 | rel-date-billing |

| 4 | Billing Doc Created | BILLING\_DOC\_CREATED | 4444-4444-4444-444444444444 | ldt4 | rel-date-billing-created |

| 5 | !Date Selection (calc) | (calculated) | 5555-5555-5555-555555555555 | ldt5 | rel-date-selection |



\*\*relationships.tmdl:\*\*

```tmdl

// Data join

relationship rel-currency-fcurr

&#x20;   fromColumn: 'Custom\_SQL\_Query'.SD\_DOC\_CURRENCY

&#x20;   toColumn: 'TCURR'.'From currency'



// Date variations

relationship rel-date-order-created

&#x20;   joinOnDateBehavior: datePartOnly

&#x20;   fromColumn: 'Custom\_SQL\_Query'.'Order Created on Date'

&#x20;   toColumn: LocalDateTable\_a1b2c3d4-1111-1111-1111-111111111111.Date



relationship rel-date-customer-ref

&#x20;   joinOnDateBehavior: datePartOnly

&#x20;   fromColumn: 'Custom\_SQL\_Query'.'Customer Reference Date'

&#x20;   toColumn: LocalDateTable\_a1b2c3d4-2222-2222-2222-222222222222.Date



relationship rel-date-billing

&#x20;   joinOnDateBehavior: datePartOnly

&#x20;   fromColumn: 'Custom\_SQL\_Query'.'Billing Date'

&#x20;   toColumn: LocalDateTable\_a1b2c3d4-3333-3333-3333-333333333333.Date



relationship rel-date-billing-created

&#x20;   joinOnDateBehavior: datePartOnly

&#x20;   fromColumn: 'Custom\_SQL\_Query'.'Billing Doc Created'

&#x20;   toColumn: LocalDateTable\_a1b2c3d4-4444-4444-4444-444444444444.Date



relationship rel-date-selection

&#x20;   joinOnDateBehavior: datePartOnly

&#x20;   fromColumn: 'Custom\_SQL\_Query'.'!Date Selection'

&#x20;   toColumn: LocalDateTable\_a1b2c3d4-5555-5555-5555-555555555555.Date

```



\---



\## 5. ALL CALCULATED COLUMNS (CORRECTED, PRODUCTION-READY)



```tmdl

// Parameter-driven date selector

column '!Date Selection' = IF(\[Parameter 3 Value] = 0,

&#x20;   'Custom\_SQL\_Query'\[Order Created on Date],

&#x20;   'Custom\_SQL\_Query'\[Billing Date])

&#x20;   dataType: dateTime

&#x20;   formatString: dd-mmm-yy

&#x20;   lineageTag: csq-calc-date-selection

&#x20;   summarizeBy: none

&#x20;   variation Variation

&#x20;       isDefault

&#x20;       relationship: rel-date-selection

&#x20;       defaultHierarchy: LocalDateTable\_a1b2c3d4-5555-5555-5555-555555555555.'Date Hierarchy'



// ISO weekday (1=Mon … 7=Sun)

column '!Date Selection (ISO Weekdays)' = WEEKDAY('Custom\_SQL\_Query'\[!Date Selection], 2)

&#x20;   dataType: int64

&#x20;   formatString: 0

&#x20;   lineageTag: csq-calc-iso-weekday

&#x20;   summarizeBy: none



// Monday-start reporting week

column '!Reporting Week' = 'Custom\_SQL\_Query'\[!Date Selection] - WEEKDAY('Custom\_SQL\_Query'\[!Date Selection], 2)

&#x20;   dataType: dateTime

&#x20;   formatString: dd-mmm-yy

&#x20;   lineageTag: csq-calc-reporting-week

&#x20;   summarizeBy: none



// Reporting Month label (Current / Previous / YYYYMM)

column '!Reporting Month' = ```

&#x20;       VAR \_ds = 'Custom\_SQL\_Query'\[!Date Selection]

&#x20;       VAR \_today = TODAY()

&#x20;       VAR \_curYM = YEAR(\_today) \* 100 + MONTH(\_today)

&#x20;       VAR \_prevYM = YEAR(EDATE(\_today, -1)) \* 100 + MONTH(EDATE(\_today, -1))

&#x20;       VAR \_dsYM = YEAR(\_ds) \* 100 + MONTH(\_ds)

&#x20;       RETURN

&#x20;           SWITCH(TRUE(),

&#x20;               \_dsYM = \_prevYM, "Previous",

&#x20;               \_dsYM = \_curYM, "Current",

&#x20;               FORMAT(\_dsYM, "0")

&#x20;           )

&#x20;       ```

&#x20;   dataType: string

&#x20;   lineageTag: csq-calc-reporting-month

&#x20;   summarizeBy: none



// Reporting Quarter label (Current / Previous / YYYYQ)

column '!Reporting Quarter' = ```

&#x20;       VAR \_ds = 'Custom\_SQL\_Query'\[!Date Selection]

&#x20;       VAR \_today = TODAY()

&#x20;       VAR \_curYQ = YEAR(\_today) \* 100 + QUARTER(\_today)

&#x20;       VAR \_prevDate = EDATE(\_today, -3)

&#x20;       VAR \_prevYQ = YEAR(\_prevDate) \* 100 + QUARTER(\_prevDate)

&#x20;       VAR \_dsYQ = YEAR(\_ds) \* 100 + QUARTER(\_ds)

&#x20;       RETURN

&#x20;           SWITCH(TRUE(),

&#x20;               \_dsYQ = \_curYQ, "Current",

&#x20;               \_dsYQ = \_prevYQ, "Previous",

&#x20;               FORMAT(\_dsYQ, "0")

&#x20;           )

&#x20;       ```

&#x20;   dataType: string

&#x20;   lineageTag: csq-calc-reporting-quarter

&#x20;   summarizeBy: none



// Parameter-driven period (Month or Quarter)

column '!Date Parameter Control' = IF(\[Parameter 4 Value] = 1,

&#x20;   'Custom\_SQL\_Query'\[!Reporting Month],

&#x20;   'Custom\_SQL\_Query'\[!Reporting Quarter])

&#x20;   dataType: string

&#x20;   lineageTag: csq-calc-date-parameter-control

&#x20;   summarizeBy: none



// Brand from MATERIAL\_DESCRIPTION (ORENCIA special-case)

column '!Brand' = ```

&#x20;       VAR \_md = 'Custom\_SQL\_Query'\[MATERIAL\_DESCRIPTION]

&#x20;       VAR \_firstSpace = SEARCH(" ", \_md, 1, 0)

&#x20;       VAR \_firstWord = IF(\_firstSpace > 0, LEFT(\_md, \_firstSpace - 1), \_md)

&#x20;       VAR \_firstTwoWords = IF(\_firstSpace > 0, LEFT(\_md, \_firstSpace + 2), \_md)

&#x20;       RETURN

&#x20;           SWITCH(TRUE(),

&#x20;               \_firstTwoWords = "ORENCIA CL", "ORENCIA SC",

&#x20;               \_firstTwoWords = "ORENCIA SC", "ORENCIA SC",

&#x20;               \_firstTwoWords = "ORENCIA LI", "ORENCIA IV",

&#x20;               \_firstWord

&#x20;           )

&#x20;       ```

&#x20;   dataType: string

&#x20;   lineageTag: csq-calc-brand

&#x20;   summarizeBy: none



// Material Brand (first word; ORENCIA gets 10 chars)

column '!Material Brand' = ```

&#x20;       VAR \_md = 'Custom\_SQL\_Query'\[MATERIAL\_DESCRIPTION]

&#x20;       VAR \_firstSpace = SEARCH(" ", \_md, 1, 0)

&#x20;       VAR \_firstWord = IF(\_firstSpace > 0, LEFT(\_md, \_firstSpace - 1), \_md)

&#x20;       RETURN IF(\_firstWord = "ORENCIA", LEFT(\_md, 10), \_firstWord)

&#x20;       ```

&#x20;   dataType: string

&#x20;   lineageTag: csq-calc-material-brand

&#x20;   summarizeBy: none



// Material Brand group (ELIQUIS normalisation)

column '!Material Brand (group)' = IF('Custom\_SQL\_Query'\[!Material Brand] IN {"ELIQUIS","Eliquis"},

&#x20;   "ELIQUIS", 'Custom\_SQL\_Query'\[!Material Brand])

&#x20;   dataType: string

&#x20;   lineageTag: csq-calc-material-brand-group

&#x20;   summarizeBy: none



// Sales Doc + Item concatenation key

column 'Sales Doc + Item' = 'Custom\_SQL\_Query'\[SALES\_DOC] \& 'Custom\_SQL\_Query'\[SALES\_DOCUMENT\_ITEM\_SALES\_I]

&#x20;   dataType: string

&#x20;   lineageTag: csq-calc-sales-doc-item

&#x20;   summarizeBy: none



// Cold chain vs standard shipping

column 'Shipping Type' = IF('Custom\_SQL\_Query'\[!Brand] IN {

&#x20;   "ORENCIA","NULOJIX","EROPOPHOS","YERVOY","OPDIVO","EMPLICITI","REBLOZYL"},

&#x20;   "Cold Chain", "Standard")

&#x20;   dataType: string

&#x20;   lineageTag: csq-calc-shipping-type

&#x20;   summarizeBy: none



// Order Net Value (sign-flip by doc type)

column 'Order Net Value' = ```

&#x20;       VAR \_net = 'Custom\_SQL\_Query'\[NET\_VALUE\_OF\_THE\_BILLING\_DOC]

&#x20;       VAR \_doctype = 'Custom\_SQL\_Query'\[SALES\_DOC\_TYPE]

&#x20;       VAR \_itemcat = 'Custom\_SQL\_Query'\[SALES\_DOC\_ITEM\_CAT]

&#x20;       RETURN

&#x20;           SWITCH(TRUE(),

&#x20;               \_doctype IN {"ZDOM","ZDMX","ZDMR"}, \_net,

&#x20;               \_doctype IN {"ZCMX","ZCMR","ZRET","ZREV"}, \_net \* -1,

&#x20;               \_doctype = "ZCCB" \&\& \_itemcat = "ZDMR", ABS(\_net),

&#x20;               \_doctype = "ZCCB" \&\& \_itemcat = "ZCMR", ABS(\_net) \* -1,

&#x20;               \_net

&#x20;           )

&#x20;       ```

&#x20;   dataType: double

&#x20;   formatString: #,##0.00

&#x20;   lineageTag: csq-calc-order-net-value

&#x20;   summarizeBy: sum



// Billing Invoice Line (sign-flip)

column 'Billing\_Invoice\_Line' = ```

&#x20;       VAR \_net = 'Custom\_SQL\_Query'\[NET\_VALUE\_OF\_THE\_BILLING\_DOC]

&#x20;       VAR \_doctype = 'Custom\_SQL\_Query'\[SALES\_DOC\_TYPE]

&#x20;       VAR \_itemcat = 'Custom\_SQL\_Query'\[SALES\_DOC\_ITEM\_CAT]

&#x20;       RETURN

&#x20;           SWITCH(TRUE(),

&#x20;               \_doctype IN {"ZDOM","ZDMX","ZDMR"}, \_net,

&#x20;               \_doctype IN {"ZCMX","ZCMR","ZRET","ZREV"}, \_net \* -1,

&#x20;               \_doctype = "ZCCB" \&\& \_itemcat = "ZDMR", ABS(\_net),

&#x20;               \_doctype = "ZCCB" \&\& \_itemcat = "ZCMR", \_net \* -1,

&#x20;               \_net

&#x20;           )

&#x20;       ```

&#x20;   dataType: double

&#x20;   formatString: #,##0.00

&#x20;   lineageTag: csq-calc-billing-invoice-line

&#x20;   summarizeBy: sum



// Exchange rate via RELATED

column 'Exchange Rate Statement' = IF(RELATED('TCURR'\[FFACT]) = 0,

&#x20;   RELATED('TCURR'\[Exchange Rate]),

&#x20;   RELATED('TCURR'\[Exchange Rate]) / RELATED('TCURR'\[FFACT]))

&#x20;   dataType: double

&#x20;   formatString: 0.00000

&#x20;   lineageTag: csq-calc-exchange-rate-statement

&#x20;   summarizeBy: none



// Sold To Name grouping (LEFT-prefix matching)

column 'Sold To Name (group)' = ```

&#x20;       SWITCH(TRUE(),

&#x20;           'Custom\_SQL\_Query'\[SOLD\_TO\_NAME] IN {

&#x20;               "BC CANCER AGENCY","BC CANCER AGENCY (PHSA)",

&#x20;               "BC CANCER AGENCY ABBOTSFORD CENTRE",

&#x20;               "BC CANCER AGENCY CNTR SOUTHERN INTE",

&#x20;               "BC CHILDREN'S HOSPITAL"}, "BC",

&#x20;           LEFT('Custom\_SQL\_Query'\[SOLD\_TO\_NAME], 8) = "BAYSHORE", "Bayshore Pharmacy",

&#x20;           LEFT('Custom\_SQL\_Query'\[SOLD\_TO\_NAME], 7) = "INNOMAR", "INNOMAR",

&#x20;           LEFT('Custom\_SQL\_Query'\[SOLD\_TO\_NAME], 8) = "MCKESSON", "MCKESSON",

&#x20;           LEFT('Custom\_SQL\_Query'\[SOLD\_TO\_NAME], 8) = "SHOPPERS", "Shoppers Drug Mart",

&#x20;           LEFT('Custom\_SQL\_Query'\[SOLD\_TO\_NAME], 4) = "QEII", "QEII",

&#x20;           LEFT('Custom\_SQL\_Query'\[SOLD\_TO\_NAME], 5) = "ROYAL", "ROYAL HOSPITALS",

&#x20;           'Custom\_SQL\_Query'\[SOLD\_TO\_NAME]

&#x20;       )

&#x20;       ```

&#x20;   dataType: string

&#x20;   lineageTag: csq-calc-sold-to-name-group

&#x20;   summarizeBy: none



// Invoice Groups binning

column 'Invoice Groups' = ```

&#x20;       SWITCH('Custom\_SQL\_Query'\[BILLING\_TYPE],

&#x20;           "ZB3","Credit Memo","ZOG2","Credit Memo",

&#x20;           "ZOL2","Debit Memo","ZS1","Debit Memo",

&#x20;           "ZFDB","Free Goods","ZS3","Free Goods",

&#x20;           "ZS2","Return and Rejection and Credit Memo",

&#x20;           "ZREB","Returns and Rejections","ZROB","Returns and Rejections",

&#x20;           "ZF2B","Sales Order","ZOF2","Sales Order",

&#x20;           "ZOF8","Samples","Other"

&#x20;       )

&#x20;       ```

&#x20;   dataType: string

&#x20;   lineageTag: csq-calc-invoice-groups

&#x20;   summarizeBy: none



// Sales Document Group binning

column 'Sales Document Group' = ```

&#x20;       SWITCH(TRUE(),

&#x20;           'Custom\_SQL\_Query'\[SALES\_DOC\_TYPE] IN {"ZCCB","ZCMR","ZCMX","ZDMR","ZDMX","ZRET"},

&#x20;               "Rebates and Adjustments",

&#x20;           'Custom\_SQL\_Query'\[SALES\_DOC\_TYPE] IN {"ZREV","ZRFE"}, "Returns",

&#x20;           'Custom\_SQL\_Query'\[SALES\_DOC\_TYPE] IN {"ZDOM","ZFD","ZSAM"}, "Sales Orders",

&#x20;           "Other"

&#x20;       )

&#x20;       ```

&#x20;   dataType: string

&#x20;   lineageTag: csq-calc-sales-doc-group

&#x20;   summarizeBy: none



// Action column (multi-field Tableau action emulation)

column 'Action (!Reporting Week,YEAR(!Date Selection),MONTH(!Date Selection),SOLD\_TO\_NAME (group))' = FORMAT('Custom\_SQL\_Query'\[!Reporting Week], "yyyy-mm-dd") \& " , " \& YEAR('Custom\_SQL\_Query'\[!Date Selection]) \& " , " \& MONTH('Custom\_SQL\_Query'\[!Date Selection]) \& " , " \& 'Custom\_SQL\_Query'\[Sold To Name (group)]

&#x20;   dataType: string

&#x20;   isHidden

&#x20;   lineageTag: csq-action-week-year-month-soldto

&#x20;   summarizeBy: none

```



\---



\## 6. SOLD TO GROUP HIERARCHY



```tmdl

hierarchy 'Sold To Group - Ship To'

&#x20;   lineageTag: csq-hier-sold-to-ship-to

&#x20;   level 'Sold To Name (group)'

&#x20;       lineageTag: csq-hier-lvl-1

&#x20;       column: 'Sold To Name (group)'

&#x20;   level SOLD\_TO\_NAME

&#x20;       lineageTag: csq-hier-lvl-2

&#x20;       column: SOLD\_TO\_NAME

&#x20;   level 'Ship-To Name'

&#x20;       lineageTag: csq-hier-lvl-3

&#x20;       column: SHIP\_TO\_NAME

```



\---



\## 7. MANUAL REVIEW FLAGS



| Item | Why Manual | Action |

|---|---|---|

| Sold To Name grouping (>80 values) | LEFT-prefix matching is approximation | Verify all groupings against source data |

| ORENCIA brand logic | Complex LEFT+SEARCH approximation | Validate against actual MATERIAL\_DESCRIPTION values |

| Exchange rate (RELATED) | Depends on TCURR relationship | Verify relationship + TCURR table structure |

| Action column emulation | Concatenated string ≠ true Tableau action | Wire cross-filter interactions in report layer |

| SAP HANA gateway | Requires on-prem gateway | Configure gateway + Basic credentials |

| `!Date Selection` variation | Calc column variation non-standard | Test date hierarchy drill-down in report layer |

| Parameter slicers | Slicers must be placed on report page | Add slicer visuals for each parameter in report layer |



