<ch>
# Customer Tender Lead Time - Tableau to Power BI Migration Knowledge Document

This document is the complete migration knowledge base for converting the Tableau workbook "Customer Tender Lead Time 1" to the Power BI semantic model "Customer Tender Lead Time Optimized 1". Every mapping was extracted from and validated against the actual input and output files listed below.

| Item | Detail |
|---|---|
| Input (Tableau) | Customer Tender Lead Time 1.json (converted from Customer Tender Lead Time 1.twbx) |
| Tableau version | 2026.1.0 (document format 18.1), extract-based federated data source |
| Output (Power BI) | Customer Tender Lead Time Optimized 1.SemanticModel (TMDL / PBIP developer mode, import mode) |
| Power BI model name | Customer Tender Lead Time- model (compatibility level 1550) |
| Data platforms | Snowflake (uscoldprod.east-us-2.azure.snowflakecomputing.com, CDW database, warehouse BI) and SharePoint Excel (Finalized Sailing Schedule Formatted for Snowflake with Audit Changes.xlsx) |
| Delivery scope note | The delivery contains only the SemanticModel folder (no .Report folder and no .pbip descriptor). All mappings in this document are model-level; Tableau dashboards and worksheets are mapped to the model objects that support them, and report-page composition cannot be validated from files |
</ch>

<ch>
## High-Level Inventory

| Object Type | Tableau (Input) | Power BI (Output) |
|---|---|---|
| Data connections | 2 named connections in 1 federated extract source "Custom SQL Query (TRANSFORM)": Snowflake custom SQL + SharePoint Excel | 2 import queries (Custom SQL Query, Sailings Schedules) + 21 utility/parameter tables |
| Parameters | 8 (Lead Time Threshold, Order Entry Hour Cutoff, % or Number, Customer Name - Showing, Date Selection, Date From, Date To, Consignee or State) | 8 Power Query parameter expressions (expressions.tmdl) plus interactive equivalents: 2 what-if numeric tables, 2 field parameters, 1 DATATABLE selector, 2 CALENDAR date tables + Datepicker, 1 customer list table |
| Calculated fields | 62 in the data source (61 formulas + 0 bins; includes 4 label-constant fields) + 8 worksheet-level parameter surrogates | 73 calculated columns on Custom SQL Query + 102 measures across 6 tables (88 in Custom Measures) |
| LOD expressions | 0 (no FIXED/INCLUDE/EXCLUDE in the workbook) | Not applicable; parameter-dependent row logic was instead moved from columns into measures (see the Calculated Lead Time chunk) |
| Table calculations | 1 (Index = INDEX()) | 'Rank Measure', 'dense Rank Measure', 'Top 20 Filter' measures |
| Groups (bins) | 0 (9 auto-generated Action groups only) | 9 hidden Action columns |
| Data source relationship | Logical model: Custom SQL Query + Sailings Schedules (Excel) | Relationship on synthetic CustomID key (many-to-many, both directions) |
| Worksheets | 29 | Model-level support objects (measures, legend/sort tables); no report file to map onto |
| Dashboards | 11 (7 at 1600 x 900, 3 at 1300 x 800, 1 at 1600 x 800) | No report file in delivery |
| Actions | 7 filter actions | 9 hidden Action columns preserve the field scopes; interactivity is native cross-filtering once a report is built |
| Model relationships | Not applicable | 3 (CustomID, Hours Late Bucket sort, ACTUAL_SHIP_DATE to DimDate) |
</ch>

<ch>
## Initial Transformation: Custom SQL Query (Snowflake)

### Tableau
The federated source's first logical table is a 8,933-character Snowflake custom SQL. Structure (validated verbatim in both tools):
1. Inner shipment subquery over cdw.transform.mbl_shipment (Rank() OVER partition by internalid11 keeping latest row) joined to mbl_shipments, mbl_referencenumbers and nine mbl_referencenumber aliases (Shipment Number primary, Order Number, PO Number, Delivery Ticket Number, Warehouse Number, Legacy Customer Number, Shipping Order, Temp, Zone), EWM_TMS_PARENT_CUSTOMER, and the MBL package/quantity/dimension/weight chain (LISTAGG rollups; MAX cases, cube, pallets, weight).
2. REGIONS derived per warehouse: Midwest (110,120,130), Southeast (140,145,370,240), Covington (200), South (400-445), CA (500-640), Northeast (800-825). SNAPSHOT TEMP derived as CF for 140/145/370/200, F for temperature starting with I, else first letter of TEMPERATURE.
3. LEFT JOIN FACT_ORDER_HISTORY on legacy customer + delivery ticket; LEFT JOIN deduplicated FACT_WINS_940_ORDERS (first transaction by MIN(LAST_MAINTENANCE_DATETIME)); INNER JOIN DIM_CONSIGNEE_MASTER; filter SHIPMENT_CREATE_DATE > '05/01/2021'.
4. 940 REQUESTED SHIP DATE reconstructed from SHIP_DATE_NUMERIC with a 05:00:00 timestamp; FIRST_DAY_OF_WEEK derived with DAYOFWEEK.
5. Two LEFT JOINs to DIM_CSC_SAILING_SCHEDULE_HISTORY (alias SSD on TEMP + DELIVERY_LOCATION_CODE + region, alias SS3 on TEMP + region + 3DZ; both restricted to year(end_date) = 2099) producing coalesced SHIP_DAY_MIN/MAX, MIN/MAX_DELIVERY, SAILING_SCHEDULE_SHIP_DATE and HOURS_DIFFERENCE.

### Power BI (Power Query)
Table 'Custom SQL Query' partition: Value.NativeQuery(Snowflake.Databases("uscoldprod.east-us-2.azure.snowflakecomputing.com","BI",[Implementation="2.0"]){[Name="CDW"]}[Data], "<identical SQL>", null, [EnableFolding=true]), followed by one added column:

```
CustomID = [REGIONS]
  & (if List.Contains({200, 240, 140, 145, 370}, [WAREHOUSE NUMBER]) then "CF"
     else if Text.Contains(Text.Lower([SNAPSHOT TEMP]), "frozen") or Text.Contains(Text.Lower([SNAPSHOT TEMP]), "ice") then "F"
     else if Text.Contains(Text.Lower([SNAPSHOT TEMP]), "chill") then "C"
     else "")
  & Text.From([CONSIGNEE 3DZ])
```

### Conversion Notes
The SQL is preserved character-for-character. The only Power Query addition is the synthetic CustomID join key (region + temperature class letter + consignee 3-digit ZIP) used to relate the fact table to the Sailings Schedules Excel table, replacing Tableau's logical-model pairing of the two tables.
</ch>

<ch>
## Initial Transformation: Sailings Schedules (SharePoint Excel)

### Tableau
Second logical table "Sailings Schedules" from the OneDrive/SharePoint Excel connection "Finalized Sailing Schedule Formatted for Snowflake with Audit Changes.xlsx". Fields surface in the workbook with the "(Sailings Schedules)" suffix: SHIP_DAY_MIN/MAX, MIN/MAX_DELIVERY, ORDER_DAY_MIN/MAX, REGION, TEMP, 3DZ, DELIVERY_LOCATION_CODE, STATE.

### Power BI (Power Query)
Table 'Sailings Schedules' partition: Excel.Workbook(Web.Contents("https://uscoldstorage.sharepoint.com/sites/BIDataRepository/Logistics%20Data%20Repository/Finalized%20Sailing%20Schedule%20Formatted%20for%20Snowflake%20with%20Audit%20Changes.xlsx")), sheet "Sailings Schedules", promoted headers, Table.SelectColumns of the 11 schedule columns, then the same synthetic key:

```
CustomID = Text.From([REGION]) & Text.From([TEMP]) & Text.From([3DZ])
```

### Conversion Notes
Column captions with the "(Sailings Schedules)" suffix are preserved as Power BI column names (SHIP DAY MIN (Sailings Schedules), MIN DELIVERY (Sailings Schedules), etc.) so Tableau field references remain traceable. The CustomID on this side concatenates REGION + TEMP + 3DZ, matching the fact-side key construction.
</ch>

<ch>
## Model Relationships

| # | Relationship | Properties | Origin |
|---|---|---|---|
| 1 | 'Custom SQL Query'[CustomID] to 'Sailings Schedules'[CustomID] | many-to-many, both directions | Replaces the Tableau logical-model pairing of the custom SQL with the Excel sailing schedule |
| 2 | 'Custom SQL Query'[Hours Late Bucket] to HoursBucketSortTable[Hours Late Bucket] | one-to-one context, both directions | Power BI addition supplying a custom sort order for the Hours Late buckets |
| 3 | 'Custom SQL Query'[ACTUAL_SHIP_DATE] to DimDate[Date] | many-to-one, single direction | Power BI addition; DimDate + DATESBETWEEN([Start Date],[End Date]) implements the Tableau Date Filter parameter logic |

Auto date/time is disabled (__PBI_TimeIntelligenceEnabled = 0); explicit DimDate/Datepicker tables and format-string date helper columns replace Tableau date hierarchies.
</ch>

<ch>
## Parameter: Lead Time Threshold

### Tableau
Integer list parameter [% or Number (copy)], members {6, 7, 8}, default 7. Consumed by [# Out of Adherence].

### Power BI
1. M parameter expression: 7 meta [IsParameterQuery=true, List={6, 7, 8}, DefaultValue=7].
2. Interactive what-if table 'Lead Time Threshold' = GENERATESERIES(6, 8, 1) with measure 'Lead Time Threshold Value' = SELECTEDVALUE('Lead Time Threshold'[Lead Time Threshold], 7).

### Conversion Notes
Dual implementation: the M parameter preserves the authoring-time default; the GENERATESERIES table + SELECTEDVALUE measure gives report users the interactive slicer that Tableau parameter controls provided. Measures reference [Lead Time Threshold Value].
</ch>

<ch>
## Parameter: Order Entry Hour Cutoff

### Tableau
Integer range parameter [Customer Name - Showing (copy)], range 0-23 step 1, default 5. Orders entered after this hour lose a day of lead time.

### Power BI
1. M parameter expression: 5 meta [List={0..23}, DefaultValue=5].
2. What-if table 'Order Entry Hour Cutoff 2' = GENERATESERIES(0, 23, 1) with measure 'Order Entry Hour Cutoff Value' = SELECTEDVALUE('Order Entry Hour Cutoff 2'[Order Entry Hour Cutoff], 5).

### Conversion Notes
Same dual pattern as Lead Time Threshold. All lead-time measures take VAR _Cutoff = [Order Entry Hour Cutoff Value].
</ch>

<ch>
## Parameter: % or Number (Metric Selector)

### Tableau
String list parameter [Parameter 1], members {% In Adherence, Orders Out of Adherence}, default "Orders Out of Adherence". Consumed by [% Orders in Adherence / # of Orders out of Adherence] (CASE switch) and [Metric Selected Suffix].

### Power BI
1. M parameter expression with the same 2-member list and default.
2. Field parameter table '% or Number final' = {("% In Adherence", NAMEOF('Custom Measures'[% In Adherence Opt]), 0), ("Orders Out of Adherence", NAMEOF('Custom Measures'[# Out of Adherence Opt]), 1)} with ParameterMetadata version 3 kind 2.
3. Supporting measures on table '% or Number': 'Metric Selected Suffix' = SWITCH(SELECTEDVALUE('% or Number'[% or Number]), "% In Adherence", "%", "Orders Out of Adherence", "", BLANK()).

### Conversion Notes
The Tableau CASE-based measure swap became a native Power BI field parameter that swaps the optimized measures directly in visuals. The suffix helper reproduces the Tableau '%' label suffix behavior.
</ch>

<ch>
## Parameter: Customer Name - Showing

### Tableau
String list parameter [Parameter 2] with 170+ customer members plus "All", default "ARYZTA". Consumed by [Customer Name] masking logic.

### Power BI
1. M parameter expression with the identical member list and default "ARYZTA".
2. Table 'Customer Name - Showing' (M Table.FromRows of the same list, column ParameterTableKeyColumn).
3. Table 'Customer Slicer' = DISTINCT('Custom SQL Query'[PARENT CUSTOMER NAME]) for a live slicer.
4. Measure 'Masked Customer' in Custom Measures implements the show/hide behavior.

### Conversion Notes
The static member list is preserved twice (M parameter + key table) and complemented by a dynamic distinct-customer table, which stays current as data refreshes - an improvement over Tableau's frozen member list.
</ch>

<ch>
## Parameter: Date Selection, Date From, Date To

### Tableau
[Parameter 3] "Date Selection" list {Year to Date, Month to Date, Previous Month, Exact Dates}, default "Previous Month"; [Parameter 4] "Date From" date, default 2022-10-01; [Parameter 5] "Date To" date, default 2023-04-30. Consumed by the [Date Filter] calculated field (1/0 flag on ACTUAL_SHIP_DATE).

### Power BI
1. Three M parameter expressions with identical defaults.
2. 'Date Selection2' = DATATABLE("Date Selection", STRING, "SortOrder", INTEGER, {{"Year to Date",1},{"Month to Date",2},{"Exact Dates",4},{"Previous Month",3}}).
3. 'Date From Parameter2' and 'Date To Parameter2' = CALENDAR(DATE(2020,1,1), DATE(2030,12,31)); Datepicker = copy of DimDate for the Exact Dates range slicer.
4. Measures 'Start Date' and 'End Date' (Custom Measures): SWITCH on SELECTEDVALUE('Date Selection2'[Date Selection], "Month to Date") - Year to Date gives DATE(YEAR(TODAY()),1,1) to TODAY(); Month to Date gives first of month to TODAY(); Previous Month gives EOMONTH(TODAY(),-2)+1 to EOMONTH(TODAY(),-1); Exact Dates gives MIN/MAX(Datepicker[Date]).
5. Every date-scoped measure applies CALCULATE(..., DATESBETWEEN(DimDate[Date], [Start Date], [End Date])).

### Conversion Notes
The Tableau row-level [Date Filter] flag became a measure-level window: Start/End Date measures resolve the selection, and DATESBETWEEN over the related DimDate applies it. The columns 'Date Filter' and 'Date Filter3' also exist on Custom SQL Query as row-level ports, plus a 'DateFilterFlag' measure, giving both mechanisms.
</ch>

<ch>
## Parameter: Consignee or State

### Tableau
String list parameter [Parameter 6], members {Consignee, State}, default "State". Consumed by [Consignee or State] = if Parameter = "Consignee" then [Consignee ID] else [CONSIGNEE STATE].

### Power BI
1. M parameter expression with the same list and default.
2. Field parameter table 'Consignee or State' = {("Consignee ", NAMEOF('Custom SQL Query'[Consignee ID]), 0), ("State", NAMEOF('Custom SQL Query'[CONSIGNEE STATE]), 1)}.

### Conversion Notes
A column-swapping Tableau calculation becomes a native Power BI field parameter that swaps the axis column directly in visuals.
</ch>

<ch>
## Core Conversion: Calculated Lead Time Chain

### Tableau
Four chained fields, all depending on the Order Entry Hour Cutoff parameter:

```
Calculated Lead Time - base:
if [Order Entered Hour] > [Order Entry Hour Cutoff] then DATEDIFF('day', [LOAD ENTERED DATE], [Expected Sailing Day - Base]) - 1
ELSE DATEDIFF('day', [LOAD ENTERED DATE], [Expected Sailing Day - Base]) END

Expected Sailing Day - Base: DATEADD('day', [SHIP_DAY_MIN]-1, [FIRST_DAY_OF_WEEK])

Expected Sailing Day:
IF [Calculated Lead Time - base] < 1 then dateadd('day', [SHIP_DAY_MIN]-1, dateadd('week',1,[FIRST_DAY_OF_WEEK]))
ELSE [Expected Sailing Day - Base] END

Calculated Lead Time:
if [Calculated Lead Time - base] < 0 then DATEDIFF('day', [LOAD ENTERED DATE], [Expected Sailing Day]) - 1
ELSE [Calculated Lead Time - base] END
```

### Power BI DAX
Two implementations exist:
1. Calculated columns 'Calculated Lead Time - base', 'Expected Sailing Day - Base', 'Expected Sailing Day', 'Calculated Lead Time' on Custom SQL Query (row-level ports, evaluated with the parameter defaults).
2. Optimized measures ('Calculated Lead Time', '# Out of Adherence', 'Average Lead Time', the 'Opt' family) that re-derive the chain inline: SUMX/AVERAGEX over SUMMARIZE('Custom SQL Query', [Order Entered Hour], [Order Submitted Date], [Expected Sailing Day - Base], [FIRST_DAY_OF_WEEK], [SHIP_DAY_MIN]) with VAR _CLTB (cutoff-adjusted DATEDIFF), VAR _ESD (roll to next week when _CLTB < 1: _FirstDayOfWeek + 7 + (_ShipDayMin - 1)), VAR _CLT (re-diff minus 1 when _CLTB < 0), wrapped in CALCULATE(..., DATESBETWEEN(DimDate[Date], [Start Date], [End Date])).

### Conversion Notes
Because the chain depends on two interactive parameters, static calculated columns cannot respond to slicer changes. The "Optimized" model therefore recomputes the chain inside measures using the SELECTEDVALUE-based [Order Entry Hour Cutoff Value] and [Lead Time Threshold Value]. The columns remain for detail-row display at default parameter values.

### Assumptions
- SHIP_DAY_MIN/FIRST_DAY_OF_WEEK arrive from the coalesced sailing-schedule join in the SQL.
- Order Submitted Date is the renamed LOAD ENTERED DATE column.
</ch>

<ch>
## Core Conversion: # Out of Adherence and the Adherence Family

### Tableau
```
# Out of Adherence: if [Calculated Lead Time] >= [Lead Time Threshold] THEN 0 else 1 END
# in Adherence: COUNT([SHIPPING ORDER]) - SUM([# Out of Adherence])
% In Adherence: 1 - SUM([# Out of Adherence]) / COUNT([SHIPMENT ID])
% out of Adherence: SUM([# Out of Adherence]) / COUNT([SHIPMENT ID])
% of shipments out of adherence: sum([# Out of Adherence]) / [Total Orders test]
Total Orders test: sum([# Out of Adherence]) + [# in Adherence]
% Orders in Adherence / # of Orders out of Adherence: CASE [% or Number] WHEN '% In Adherence' THEN [% In Adherence]*100 WHEN 'Orders Out of Adherence' THEN sum([# Out of Adherence]) END
```

### Power BI DAX
- Column '# Out of Adherence' (row-level port with defaults) plus measure '# Out of Adherence' that re-derives the lead-time chain per SUMMARIZE row and returns IF(ISBLANK(_CLT) || _CLT < _Threshold, 1, 0) summed within the Start/End date window.
- '# in Adherence' = CALCULATE(COUNT('Custom SQL Query'[SHIPPING ORDER]), DATESBETWEEN(...)) - [# Out of Adherence].
- '% In Adherence' = 1 - DIVIDE([# Out of Adherence], CALCULATE(COUNT([SHIPMENT ID]), DATESBETWEEN(...))).
- '% out of Adherence' = CALCULATE(DIVIDE([# Out of Adherence], COUNT([SHIPMENT ID])), DATESBETWEEN(...)).
- '% of shipments out of adherence' = DIVIDE(COALESCE(SUM([# Out of Adherence]),0), [Total Orders test]).
- Optimized and card variants: '# Out of Adherence Opt', '# In Adherence Opt', '% In Adherence Opt', '% Out of adherence Opt', '# Out of Adherence Cards', '# In Adherence Cards', '% In Adherence Cards', '% Out of Adherence Cards' - the Opt pair feeds the '% or Number final' field parameter.

### Conversion Notes
The Tableau parameter-driven CASE measure became the field parameter (see the % or Number chunk). Note the Tableau inversion is preserved: an order is out of adherence when Calculated Lead Time is below the threshold.
</ch>

<ch>
## Custom Column Family: Lead-Day Buckets

### Tableau
```
6 Day Lead: IF [Calculated Lead Time] >= 6 and < 7 THEN 1 ELSE 0 END
5 Day Lead: IF [Calculated Lead Time] >= 5 and < 6 THEN 1 ELSE 0 END
4 Day Lead: IF [Calculated Lead Time] >= 4 and < 5 THEN 1 ELSE 0 END
< 3 Day: IF [Calculated Lead Time] <= 3 THEN 1 ELSE 0 END
6th Day Lead Time: sum([6 Day Lead]) / COUNTD([SHIPPING ORDER])
6th Day?: if [Calculated Lead Time] = 6 then "Yes" Else "No" END
Lead Time Compliant?: if [Calculated Lead Time] >= 7 then "Yes" else "No" END
Out of Compliance?: if [Calculated Lead Time] >= 7 THEN "In Compliance" else "Out of Compliance" END
```

### Power BI DAX
Calculated columns '6 Day Lead', '5 Day Lead', '4 Day Lead', '< 3 Day Lead', '6th Day?', 'Lead Time Compliant?', 'Out of Compliance?' with identical logic, plus measures '6 Day Lead', '5 Day Lead', '4 Day Lead', '< 3 Day Lead' (parameter-aware re-derivations of the same buckets), '6th Day Lead Time' = CALCULATE(DIVIDE([6 Day Lead], DISTINCTCOUNT([SHIPPING ORDER])), DATESBETWEEN(...)), and Opt variants ('6 Day Lead Opt' etc.) plus '6 Day Lead Time Cards' and '6th Day Lead Time % Cards'.

### Conversion Notes
Column versions use the static Calculated Lead Time column; the measure/Opt versions recompute lead time with live parameter values, so slicer changes flow through.
</ch>

<ch>
## Custom Column: Sailing Schedule Compliance Family

### Tableau
```
Sailing Schedule Compliance: IF [Requeste Ship Date #] = [SHIP_DAY_MIN] or = [SHIP_DAY_MAX] then "Yes" else "No" END
Requeste Ship Date #: ISOWEEKDAY([940 REQUESTED SHIP DATE]) + 1
# Out of Sailing Schedule: IF [Sailing Schedule Compliance] = "No" THEN 1 ELSE 0 END
Sailing Schedule Compliance Count: if [Sailing Schedule Compliance] = "No" THEN 1 ELSE 0 END
Sailing Schedule Compliance %: 1 - sum([Sailing Schedule Compliance Count]) / COUNT([SHIPPING ORDER])
Sailing Schedule Total In Adherence %: 1 - (sum([Sailing Schedule Compliance Count]) / COUNT([SHIPPING ORDER]))
Shipping Day Adherence: IF [Sailing Schedule Compliance] = "Yes" then 1 else 0 END
Ship Day Adherence %: sum([Shipping Day Adherence]) / COUNTD([SHIPMENT ID])
```

### Power BI DAX
- Column 'Requeste Ship Date #' = WEEKDAY('Custom SQL Query'[940 REQUESTED SHIP DATE], 2) + 1 (ISO weekday emulation).
- Columns 'Sailing Schedule Compliance', '# Out of Sailing Schedule', 'Sailing Schedule Compliance Count', 'Shipping Day Adherence' with identical IF logic.
- Measures: 'Sailing Schedule Total In Adherence %' (on Custom SQL Query), 'Sailing Schedule Compliance %', 'Ship Day Adherence %', 'Sailing Schedule Compliance Count Measure' = CALCULATE(SUM([Sailing Schedule Compliance Count]), DATESBETWEEN(...)), '# Out of Sailing Schedule', plus card variants ('Sailing Schedule Ship Adherence Cards', 'Sailing Schedule Compliance Count Cards', 'Total % in SSA Cards', 'Out of Sailing Schedule Cards', 'Total Shipping Orders Cards') and SELECTEDVALUE-based test measures ('Sailing Schedule Compliance Flag', 'Sailing Schedule Compliance test2', 'Sailing Schedule Compliance est3') on the '% or Number' table.

### Conversion Notes
WEEKDAY(date, 2) returns Monday=1 in DAX, matching ISOWEEKDAY; the +1 offset is preserved from Tableau.
</ch>

<ch>
## Custom Column: Hours Late and Hours Late Bucket

### Tableau
```
Hours submitted before due date: ([Expected Sailing Day] - [LOAD ENTERED DATE]) * 24
Hours Late:
if [Hours submitted before due date] < 0 then [Hours submitted before due date]
ELSEIF [Hours submitted before due date] <= 168 then 168 - [Hours submitted before due date] else 0 end
Hours Late Bucket:
if [Out of Compliance?] = "In Compliance" then "Submitted On Time"
elseif [Hours Late] < 0 then "Order Submitted Off Sailing Schedule"
elseif [Hours Late] <= 4 and >= 0 then "< 4 Hours"
elseif > 4 and <= 12 then "4-12 Hours"
elseif > 12 and <= 24 then "12-24 Hours"
elseif > 24 and <= 48 then "24-48 Hours"
else "> 48 Hours" end
```

### Power BI DAX
Columns 'Hours submitted before due date' = ('Custom SQL Query'[Expected Sailing Day] - [Order Submitted Date]) * 24, 'Hours Late', and 'Hours Late Bucket' with identical thresholds; measures 'Hours Submitted before due date' and 'Hours Late' = CALCULATE(SUM([Hours Late]), DATESBETWEEN(DimDate[Date], [Start Date], [End Date])). Supporting tables: HoursBucketSortTable (related, SortOrder for the 7 buckets), HoursLateDisplaySort (display order with Submitted On Time first), HoursLateLegend (bucket-to-hex-color legend rows using UNICHAR(9632) square swatches: Submitted On Time #b07aa1, Order Submitted Off Sailing Schedule #ffa500, < 4 Hours #00aeef, 4-12 Hours #363d41, 12-24 Hours #e0e0e1, 24-48 Hours #6c7478, > 48 Hours #d11f37).

### Conversion Notes
Tableau color assignments and manual sort of the bucket dimension are externalized into the three legend/sort tables, with the sort table physically related to the fact bucket column.
</ch>

<ch>
## Custom Column: Warehouse, Region, and Temp Attributes

### Tableau
```
Warehouse Region: IF [WAREHOUSE NUMBER] = 110/120/130 THEN "Midwest" ... 140/145 "McDonough", 370 "Lake City", 400-440 "South", 520-550 "NCal", 600/770/620 "SCal", 800-825 "Northeast" END
Warehouse Name: 22-branch warehouse-number-to-name ladder (110 Minooka, 120 Wilmington, 130 Lebanon, 140/145 McDonough, 370 Lake City, 400 Arlington, 410 Dallas, 420 Fort Worth, 430 Dallas South, 440 Denton, 520 Sacramento, 525 McClellan Park, 530 Tracy, 550 Turlock, 600 Bakersfield, 770 Syracuse, 620 Fresno, 800 Bethlehem, 810 Hazleton, 820 Quakertown East, 825 Quakertown West)
Shipment Temp: warehouse-number plus SNAPSHOT TEMP ladder returning F / C / CF
Exclude - Region: if ISNULL([REGIONS]) then "EXCLUDE" ELSE "INCLUDE" END
```

### Power BI DAX
Calculated columns 'Warehouse Region', 'Warehouse Name', 'Shipment Temp', 'Exclude - Region' on Custom SQL Query, each preserving the original Tableau expression as a TMDL comment with identical branch values. Supporting legend tables: RegionsLegend (CA #9c755f, Midwest #e15759, Northeast #59a14f, South #b07aa1, Southeast #bab0ac with UNICHAR(9632) swatch column) and 'Region Group' (NULL #4e79a7, Southeast #f28e2b) reproduce the Tableau color legends; a 'Region Group' calculated column also exists on the fact table.

### Conversion Notes
The SQL-level REGIONS column (Midwest/Southeast/Covington/South/CA/Northeast) is distinct from the workbook-level Warehouse Region groupings; both are preserved.
</ch>

<ch>
## Custom Column: Customer and Consignee Attributes

### Tableau
```
PARENT CUSTOMER NAME: IF NOT ISNULL([PARENT CUSTOMER]) THEN [PARENT CUSTOMER] ELSE [CUSTOMER] END
CUSTOMER NAME: IF [PARENT CUSTOMER] = "RICH PRODUCTS" THEN "RICH PRODUCTS" ELSE "CUSTOMER NAME" END
Customer Name (masking): IF [Customer Name - Showing] = [PARENT CUSTOMER NAME] THEN [PARENT CUSTOMER NAME] ELSEIF [Customer Name - Showing] = "All" THEN [PARENT CUSTOMER NAME] ELSE "Customer" END
Rich Products: CONTAINS([CUSTOMER NAME], 'RICH')
Consignee ID: trim([CONSIGNEE NAME]) + ' - ' + trim([CONSIGNEE CITY]) + ' - ' + trim([CONSIGNEE STATE])
Consignee Parent Name: CONTAINS ladder mapping consignee names to PUBLIX / WALMART / TARGET / KROGER / WALGREENS / C&S / ASSOCIATED / AMAZON / COMMERCIAL WAREHOUSING / OTHER
Consignee or State: parameter-driven column swap (see the Consignee or State parameter chunk)
```

### Power BI DAX
Columns 'PARENT CUSTOMER NAME', 'CUSTOMER NAME' (lineage Calculation_2412803550991962148), 'Rich Products' = IF(CONTAINSSTRING([CUSTOMER NAME], "RICH"), TRUE(), FALSE()), 'Consignee ID', 'Consignee Parent Name' with identical CONTAINSSTRING ladders. The masking logic became the measure 'Masked Customer' (Custom Measures) driven by the Customer Name - Showing parameter/slicer; the column swap became the 'Consignee or State' field parameter.

### Conversion Notes
Customer masking moved from a row-level column to a measure so it responds to slicer selection; 'Customer Slicer' (DISTINCT parent customers) and 'Customer Order Summary' (SUMMARIZE by parent customer with COUNTA of SHIPMENT ID) support the customer views and Top 20 logic.
</ch>

<ch>
## Custom Column: Mode, TL Flag, and Order Timing Attributes

### Tableau
```
Mode: IF [WEIGHT] >= 30000 THEN "TL" ELSEIF [CUBE] > 1500 THEN "TL" ELSEIF [PALLETS] > 20 THEN "TL" ELSE "LTL" END
TL Flag: if [WEIGHT] > 30000 or [PALLETS] > 20 or [CUBE] > 1500 then 'TL' else 'LTL' end
Order Entered Hour: DATEPART('hour', [LOAD ENTERED DATE])
Requested vs. Entered: DATEDIFF('day', [940 REQUESTED SHIP DATE], [LOAD ENTERED DATE])
Requested Ship Week: DATEPART('week', [940 REQUESTED SHIP DATE])
Requested Ship Year: DATEPART('year', [940 REQUESTED SHIP DATE])
Requested Ship Week Day: INT(weekday-name ladder over [SHIP_DAY_MIN])
Original Difference: IF [Order Entered Hour] > 5 THEN DATEDIFF('day', [LOAD ENTERED DATE], [940 REQUESTED SHIP DATE]) - 1 ELSE DATEDIFF(...) END
# of Days from Order to Ship: cutoff-adjusted DATEDIFF minus 3 * DATEDIFF('week', ...) weekend correction
Calculated Ship Date 2: if [Expected Sailing Day] < [LOAD ENTERED DATE] then DATEADD('day', [SHIP_DAY_MAX]-1, [FIRST_DAY_OF_WEEK]) else [Expected Sailing Day] end
```

### Power BI DAX
All ported as calculated columns on Custom SQL Query with the original Tableau expression preserved as a comment: Mode, 'TL Flag', 'Order Entered Hour' (HOUR()), 'Requested vs. Entered' (DATEDIFF), 'Requested Ship Week', 'Requested Ship Year' (YEAR()), 'Requested Ship Week Day', 'Original Difference', '# of Days from Order to Ship' (VAR-based port), 'Calculated Ship Date 2'.

### Conversion Notes
'Original Difference' hard-codes the cutoff hour 5 in both tools (it does not read the parameter). The Requested Ship Week Day INT-of-string ladder is a Tableau oddity ported as-is.
</ch>

<ch>
## Custom Columns: Label Constants and Hidden Action Columns

### Tableau
Four constant label fields used as column headers: [Complaint to Sailing Schedule?] = "Complaint to Sailing Schedule?", [Late Bucket] = "Late Bucket", [Lead Time] = "Lead Time", [Orders out of Lead Time] = "Orders out of Lead Time". Nine auto-generated Action groups: Action (Consignee ID), Action (Consignee or State), Action (Hours Late Bucket,WEEKDAY(Expected Sailing Day)), Action (PARENT CUSTOMER NAME), Action (PARENT CUSTOMER NAME,Metric Selected Suffix), Action (PARENT CUSTOMER NAME,Orders out of Lead Time,REGIONS), Action (PARENT CUSTOMER NAME,REGIONS), Action (PARENT CUSTOMER), Action (Rich Products,PARENT CUSTOMER).

### Power BI DAX
The four label constants became measures on Custom Measures: 'Complaint to Sailing Schedule?' = "COMPLAINT TO SAILING SCHEDULE?", 'Late Bucket' = "LATE BUCKET", 'Lead Time' = "LEAD TIME", 'Orders out of Lead Time' = "ORDERS OUT OF LEAD TIME". The nine Action groups became hidden concatenation columns on Custom SQL Query (each preserving the original Tableau expression as a comment).

### Conversion Notes
Label casing changed to uppercase in the Power BI measures. The Action columns document the cross-filter field scopes of the seven Tableau filter actions for when a report is assembled.
</ch>

<ch>
## Table Calculation: Index / Top 20

### Tableau
[Index] = INDEX() - a table calculation used to rank and cap the Top 20 Out of Adherence views.

### Power BI DAX
Measures in Custom Measures: 'Rank Measure' and 'dense Rank Measure' (RANKX-based ranks over customers/consignees) and 'Top 20 Filter' (returns 1 for rows whose rank is within 20; applied as a visual-level filter).

### Conversion Notes
Tableau INDEX() depends on the table layout; the DAX replacement ranks explicitly by the adherence measure so it is layout-independent.
</ch>

<ch>
## Measures: BAN / Cards / Opt Families and Utilities

The Custom Measures table (88 measures) is organized in three tiers plus utilities, all validated in TMDL:

| Tier | Members (validated names) | Purpose |
|---|---|---|
| Base | # Out of Adherence, # in Adherence, % In Adherence, % out of Adherence, % of shipments out of adherence, 6th Day Lead Time, Average Lead Time, Sailing Schedule Compliance %, Ship Day Adherence %, Total Orders, Weighted Lead Time, 6/5/4/< 3 Day Lead, Avg_Calculated Lead Time, # Out of Sailing Schedule, Sum. Calculated Lead Time, Hours Submitted before due date, Hours Late, Calculated Lead Time, Calculated Lead Time - base, Calculated Lead Time - base final, Expected Sailing Day Final, Count Shipping ID, Count Shipping Order, Total Shipping Order | Direct ports of the Tableau aggregate fields, each wrapped in DATESBETWEEN(DimDate[Date], [Start Date], [End Date]) |
| Cards | Avg. Calculated Lead Time card, Total Orders Cards, # In Adherence Cards, # Out of Adherence Cards, % In Adherence Cards, % Out of Adherence Cards, 6th Day Lead Time % Cards, 6 Day Lead Time Cards, Ship Day Adherence Cards, Average Lead time Cards, Sailing Schedule Ship Adherence Cards, Sailing Schedule Compliance Count Cards, Total % in SSA Cards, Total Shipping Orders Cards, Out of Sailing Schedule Cards, Total Non - Distinct Orders Cards, Total Non Distinct Orders Cards | The Tableau BAN worksheets (Total Orders BAN, % In Adherence BAN, Average Lead Time - BAN, 6th Day Lead Time BAN, Ship Day Adherence BAN, Sailing Schedule # Out BAN, Total % in Sailing Schedule Adherence) map to this family |
| Opt | Average Lead Time Opt, % Out of adherence Opt, # In Adherence Opt, % In Adherence Opt, # Out of Adherence Opt, < 3 / 4 / 5 / 6 Day Lead Opt, Avg. Calculated Lead Time Opt dummy, Out of Sailing Schedule Opt, Total Orders Cards Opt | Performance-optimized re-derivations; the colon-suffixed display measures (Distinct count of SHIPMENT ID:, % out of Adherence:, # in Adherence:, # Out of Adherence:, % in Adherence:, Avg. Calculated Lead Time:) alias into this tier |
| Utility / debug | Start Date, End Date, Selected Date Range, DateFilterFlag, Debug Date, Debug Date Count, CLT Filter Measure, Show CLT Row, Show Visual, CLT Outside Range Count, CLT Outside Range Sum, Masked Customer, Total SQL Rows, Total SS Rows, Sailing Schedule Compliance TEST, Copy. Calculated Lead Time, Average Lead Time Display, Top 20 Filter, Rank Measure, dense Rank Measure, Total Orders test, Total Non - Distinct Orders | Date-window resolution, data-quality checks, ranking, masking, and development leftovers |

Refresh Info adds 'Last Refresh' and 'Last Refresh Display' ("Last Refreshed: " & FORMAT(...)) over an M table containing DateTime.LocalNow() - the model's refresh-timestamp mechanism.
</ch>

<ch>
## Date Infrastructure: DimDate, Datepicker, and Date Helper Columns

| Object | Definition (validated) | Purpose |
|---|---|---|
| DimDate | CALENDARAUTO() with Year, Quarter, Month (MMM), Weeknum, Month Start, Week Start columns and 'Month Start Hierarchy' (Month Start > Week Start > Date) | Related to Custom SQL Query[ACTUAL_SHIP_DATE]; the DATESBETWEEN window target |
| Datepicker | Calculated copy of DimDate | Disconnected range-slicer table for the Exact Dates selection |
| Date helper columns on Custom SQL Query | Expected Sailing Day_Day-Trunc, 940 REQUESTED SHIP DATE_Day-Trunc, ACTUAL_SHIP_DATE_Month/Week-Trunc/Week/Year, Expected Sailing Day_WeekDay, Expected Sailing WeekDay_Name, Month/Week/Quarter/Year/Day of ACTUAL_SHIP_DATE, Actual Ship Date Week Name, Order Submitted Date Format, Expected Sailing Day Format, DATEONLY variants | Replace Tableau date-part shelf derivations; three hierarchies built on them (Expected Sailing Day_Date Hierarchy, ACTUAL_SHIP_DATE_Date Hierarchy, Year - Day Start Hierarchy) |
</ch>

<ch>
## Tableau Dashboards and Worksheets Coverage

The workbook has 29 worksheets composing 11 dashboards. Because the delivery contains no .Report folder, the mapping targets are the model objects that support each dashboard:

| Tableau Dashboard (size) | Worksheets (validated zones) | Supporting Power BI Model Objects |
|---|---|---|
| Customer Tender Lead Time Dashboard (1600x900) | Customer In Adherence Graph, Out of Adherence Data, 7 BANs (% In Adherence, 6th Day Lead Time, Average Lead Time, In Adherence, Ship Day Adherence, Total Orders) | Cards measure family, Adherence family, Hours Late Bucket + sort/legend tables |
| Company Dashboard (1600x900) | Company Graph, % Adherence, Out of Adherence Data + BANs | Same families sliced by PARENT CUSTOMER NAME; Customer Slicer table |
| Regional Dashboard (1600x900) | Region Graph, % Adherence, Out of Adherence Data + BANs | Warehouse Region / REGIONS columns, RegionsLegend |
| Customer Dashboard Data (1600x900) | Customer Data + BANs | Detail columns incl. Consignee ID, Masked Customer |
| Customer Consignee Data (1600x900) | % Adherence (Consignee) + BANs | Consignee Parent Name, Consignee or State field parameter |
| Customer Dashboard Graph - Sharing (1600x900) | % Adherence - Sharing + BANs | Masked Customer measure |
| Sailing Schedule Compliance - Customer Overview (1600x900) | Customer Data (2), Sailing Schedule # Out BAN, Sailing Schedule % Graph, Total % in Sailing Schedule Adherence, Total Orders BAN | Sailing Schedule Compliance family, Cards |
| USCS Customer Order Submission Overview (1300x800) | Order Submission Data (4), Top 20 Out of Adherence | Top 20 Filter / Rank measures, Customer Order Summary |
| USCS Customer Order Submission Overview (2) (1300x800) | Order Submission Data Consignee, Top 20 Out of Adherence Consignees | Same, consignee grain |
| USCS Customer Order Submission Trend (1300x800) | Top 20 Out of Adherence Consignees (2) | Same plus week columns |
| USCS Order Submission Analysis (1600x800) | Order Submission Data, Out of Adherence Graph | Hours Late Bucket family, % or Number field parameter |

The remaining worksheets (Sheet 24, % Adherence (Company)) are development sheets. The 7 Tableau filter actions map to native cross-filtering; the 9 hidden Action columns preserve their field scopes.
</ch>

<ch>
## Known Gaps and Intentional Differences

| # | Item | Status / Detail |
|---|---|---|
| 1 | Report layer | No .Report folder or .pbip in the delivery; only the semantic model exists. Dashboard/worksheet-to-page mapping cannot be validated from files |
| 2 | Parameter duality | Every Tableau parameter exists twice: as an M parameter expression (authoring default) and as an interactive table/field parameter. The M parameters do not drive the measures; the SELECTEDVALUE tables do |
| 3 | Row-level vs measure-level lead time | Calculated columns evaluate the lead-time chain with default parameter values only; the Opt/measure family is the authoritative parameter-aware implementation. Mixing the two in one visual can show different numbers |
| 4 | Development artifacts | TEST/Debug/dummy measures (Sailing Schedule Compliance TEST, Copy. Calculated Lead Time, Avg. Calculated Lead Time Opt dummy, Debug Date, Expected Sailing Day - Base TEST, Expected Sailing Day TEST columns, Date Filter3), 7 saved DAX queries, and 1 TMDL script are retained |
| 5 | Label constant casing | The four header-label measures return uppercase text while Tableau returned mixed case |
| 6 | Hidden UNICHAR swatches | HoursLateLegend, RegionsLegend, and Region Group use UNICHAR(9632) filled-square characters as color swatch columns - a Power BI pattern replacing Tableau color legends |
| 7 | Refresh Info uses DateTime.LocalNow() | Evaluates in the refresh service's local time zone |
| 8 | Requested Ship Week Day | The Tableau INT(weekday-name) expression returns null in practice; ported as-is for fidelity |
</ch>

<ch>
## Validation Summary

| Check | Input (Tableau JSON) | Output (SemanticModel TMDL) | Result |
|---|---|---|---|
| Snowflake custom SQL | 8,933-character query | Identical query embedded in Value.NativeQuery with EnableFolding=true | Match |
| Excel sailing schedule | OneDrive/SharePoint xlsx, Sailings Schedules sheet | Web.Contents SharePoint URL, same sheet, 11 columns + CustomID | Match |
| Join model | Logical pairing of the two tables | CustomID (REGION + temp letter + 3DZ) many-to-many bidirectional relationship, keys built symmetrically in M | Match (mechanism documented) |
| Parameters | 8 with defaults 7, 5, "Orders Out of Adherence", "ARYZTA", "Previous Month", 2022-10-01, 2023-04-30, "State" | 8 M parameter expressions with identical lists/defaults + interactive equivalents | Match |
| LOD expressions | 0 | Not applicable | Match |
| Table calculations | 1 (INDEX()) | Rank / dense Rank / Top 20 Filter measures | Match |
| Calculated fields | 62 (61 formulas + 4 label constants within) | 73 calculated columns + 102 measures; each ported object carries the original Tableau expression as a TMDL comment | Accounted (families documented per chunk) |
| Lead time chain | 4-field chain with cutoff and threshold parameters | Columns (defaults) + parameter-aware measure re-derivation verified line-by-line (# Out of Adherence, Average Lead Time) | Match |
| Adherence thresholds | Out when Calculated Lead Time < threshold (default 7) | IF(ISBLANK(_CLT) || _CLT < _Threshold, 1, 0) | Match |
| Hours Late buckets | 7 buckets with 4/12/24/48/168-hour boundaries | Identical column logic + 3 sort/legend tables | Match |
| Warehouse mappings | 22-branch name ladder, region ladders, temp ladder | Identical DAX ladders with original expressions in comments | Match |
| Date Filter parameter logic | Year to Date / Month to Date / Previous Month / Exact Dates flag | Start Date / End Date measures + DATESBETWEEN over DimDate; identical window boundaries incl. Previous Month December edge handled by EOMONTH arithmetic | Match |
| Worksheets / dashboards | 29 / 11 | Model support objects documented; no report file | Documented limitation |
| Relationships | Not applicable | 3 verified in relationships.tmdl | Verified |
| Measures | Not applicable | 102 verified (88 Custom Measures + 9 % or Number + 2 Refresh Info + 1 Custom SQL Query + 1 Lead Time Threshold + 1 Order Entry Hour Cutoff 2) | Verified |

End of document.
</ch>
