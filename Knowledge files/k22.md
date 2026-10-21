# 19863 – WIP: Capacity and Slot Allocations
## Tableau → Power BI (PBIP) Conversion Knowledge File
**Generated:** 2026-07-03
**Model Compatibility Level:** 1604
**Culture:** en-US
**Data Source:** PostgreSQL – bms-eim-cell-therapy-auroradb-read-prod.web.bms.com:5432/postgres
**Schema:** ct_bia (sdop_capacity_slot_allocation, raw_treatment_fc, allocations_planning_aph_slot_allocations, sdop_confirmed_allocation_week)
**Time Intelligence:** Disabled
**Sets Processed:** 5 (Files 1–40)

---

## TABLE OF CONTENTS
1. [Data Model Overview](#1-data-model-overview)
2. [Relationships](#2-relationships)
3. [Parameter Mapping: Tableau → Power BI](#3-parameter-mapping-tableau--power-bi)
4. [Calculated Column Mapping](#4-calculated-column-mapping)
5. [DAX Measure Mapping](#5-dax-measure-mapping)
6. [Power Query (M) Transformation Mapping](#6-power-query-m-transformation-mapping)
7. [Dimension / Slicer Tables](#7-dimension--slicer-tables)
8. [Calculated Tables (DAX)](#8-calculated-tables-dax)
9. [Refresh / Metadata Tables](#9-refresh--metadata-tables)
10. [Conversion Completeness Summary](#10-conversion-completeness-summary)
11. [Known Gaps & Recommendations](#11-known-gaps--recommendations)

---

## 1. DATA MODEL OVERVIEW

### 1.1 All Tables in Model (32 total)

| # | Table Name | Type | Source |
|---|---|---|---|
| 1 | SDOP Capacity Slot Allocation_Custom SQL Query | Fact (main) | PostgreSQL – ct_bia.sdop_capacity_slot_allocation |
| 2 | Details Actuals and Cancels_Custom SQL Query | Fact (secondary) | PostgreSQL – ct_bia.sdop_capacity_slot_allocation (booked/used/cancelled + JSON) |
| 3 | Measures Table | Measures container | Hardcoded single-row M table |
| 4 | Date | Calendar | DAX CALENDARAUTO() |
| 5 | End Month | Parameter | M – monthly list 2020-01 to 2025-12, default 2025-02-01 |
| 6 | Start Month | Parameter | M – monthly list 2020-01 to 2025-12, default 2025-01-01 |
| 7 | Month Range | Parameter | M – hardcoded {custom/Custom, default/Default} |
| 8 | Month Range (2) | Parameter toggle | M – Base64 compressed {Custom, Default} |
| 9 | Aph Allocation Snapshot | Parameter | M – hardcoded {SD&OP→Franchise Decision, Updated→Operational Adjusted} |
| 10 | Aph Allocation Sanpshot | Dimension (typo) | M – DISTINCT level values from fact table |
| 11 | Aph Include Re-Aphs | Parameter | M – {aph→No, re→Yes} |
| 12 | Mfg Include Re-Aphs | Parameter | M – {aph→No, re→Yes (Actuals only)} |
| 13 | Capacity Level | Parameter | M – {approved→Approved, operational→Operational} |
| 14 | Product | Parameter | M – hardcoded 7-product list, default = GUID |
| 15 | Products | Dimension | M – DISTINCT product from fact |
| 16 | Show Accuracy | Parameter | M – {false→No, true→Yes}, default = GUID |
| 17 | Show Analysis | Parameter | M – {0→None, 1→Slot Utilization, 2→Allocation Accuracy} |
| 18 | Site Group | Parameter | M – {cmo→CMO, manufacturing→Manufacturing} |
| 19 | Site Id | Dimension | M – DISTINCT site_id from fact |
| 20 | Channel | Dimension | M – DISTINCT channel_name from fact |
| 21 | Geography Code | Dimension | M – DISTINCT geography_code from fact |
| 22 | View Demand By | Parameter | M – {apheresis→Apheresis Date, make→Manufacturing Start Date} |
| 23 | Week Starts | Parameter | M – {Sunday, Monday}, default = GUID |
| 24 | Parameter | Calculated table | DAX – {Monthly→Month field, Weekly→Week Date field} |
| 25 | PP | Calculated table | DAX – ALL(product) |
| 26 | Protocol | Calculated table | DAX – ALL(Protocol 1) |
| 27 | DTG | Calculated table | DAX – DISTINCT(Metrics) + sort column |
| 28 | Slotbychannel | Calculated table | DAX – DISTINCT(testt) |
| 29 | WeekStart | Calculated table | DAX – ALL(week_start) |
| 30 | Detail | Calculated table | DAX – ALL(Details 1) |
| 31 | Data Source Refresh Timestamp_Custom SQL Query | Metadata | PostgreSQL – GPS/APEX/Site Capacity timestamps |
| 32 | refresh | Metadata (duplicate) | PostgreSQL – same query as above, different display column |

---

## 2. RELATIONSHIPS

| # | From Table | From Column | To Table | To Column | Cardinality | Direction |
|---|---|---|---|---|---|---|
| 1 | SDOP Capacity Slot Allocation_Custom SQL Query | Month | Date | Date | Many-to-One | Single |
| 2 | SDOP Capacity Slot Allocation_Custom SQL Query | Channel Name | Channel | channel_name | Many-to-One | Single |
| 3 | SDOP Capacity Slot Allocation_Custom SQL Query | Geography Code | Geography Code | geography_code | Many-to-One | Single |
| 4 | SDOP Capacity Slot Allocation_Custom SQL Query | Site Id | Site Id | site_id | Many-to-One | Single |
| 5 | SDOP Capacity Slot Allocation_Custom SQL Query | Protocol 1 | Protocol | Protocol 1 | Many-to-One | Single |
| 6 | SDOP Capacity Slot Allocation_Custom SQL Query | Details 1 | Detail | Details 1 | Many-to-One | Single |
| 7 | SDOP Capacity Slot Allocation_Custom SQL Query | testt | Slotbychannel | testt | Many-to-One | Single |
| 8 | SDOP Capacity Slot Allocation_Custom SQL Query | week_start | WeekStart | week_start | Many-to-One | Single |
| 9 | SDOP Capacity Slot Allocation_Custom SQL Query | product | PP | product | Many-to-One | Single |
| 10 | Details Actuals and Cancels_Custom SQL Query | Month | Date | Date | Many-to-One | Single |
| 11 | Aph Allocation Snapshot | value1 | Capacity Level | ParameterTableKeyColumn | Many-to-One | Bidirectional |
| 12 | Date | Date | End Month | ParameterTableKeyColumn | Many-to-One | Single |
| 13 | Date | Date | Start Month | ParameterTableKeyColumn | Many-to-One | Single |
| 14–23 | (remaining 10 from relationships.txt) | Various | DTG, Site Group, WeekStart, etc. | Various | Many-to-One | Single |

**Tableau Equivalent:** Tableau data source joins and LOD scoping handled relationships implicitly. Power BI makes all relationships explicit in the model schema.

---

## 3. PARAMETER MAPPING: TABLEAU → POWER BI

| # | Tableau Parameter | Tableau Type | PBI Table Name | PBI Type | Default Value | Conversion Status |
|---|---|---|---|---|---|---|
| 1 | View Demand By (Parameter 1) | String list | View Demand By | M hardcoded table | Apheresis Date | ✅ Complete |
| 2 | Site Group (Parameter 2) | String list | Site Group | M hardcoded table | CMO | ✅ Complete |
| 3 | Start Month (Parameter 3) | Date | Start Month | M date list 2020–2025 | 2025-01-01 | ✅ Complete |
| 4 | Month Range (Parameter 4) | String list | Month Range | M hardcoded table | Default | ✅ Complete |
| 5 | Product (Parameter 5) | String list | Product | M hardcoded list (7 products) | GUID-based | ✅ Complete |
| 6 | Show Accuracy (Parameter 6) | Boolean | Show Accuracy | M boolean table | No (false) | ✅ Complete |
| 7 | Show Analysis (Parameter 7) | String list | Show Analysis | M numeric-keyed table | None (0.0) | ✅ Complete |
| 8 | Aggregation View (Parameter 8) | String list | Parameter | DAX calculated table + NAMEOF() | Monthly | ⚠️ Partial – Monthly branch dropped in Aggregation Calculation column |
| 9 | Week Starts (Parameter 9) | String list | Week Starts | M hardcoded {Sunday, Monday} | GUID-based | ✅ Complete |
| 10 | Aph. Allocation Snapshot | String list | Aph Allocation Snapshot | M hardcoded table | Operational Adjusted | ✅ Complete |
| 11 | Aph: Include Re-Aphs | String list | Aph Include Re-Aphs | M hardcoded table | No | ✅ Complete |
| 12 | Mfg: Include Re-Aphs | String list | Mfg Include Re-Aphs | M hardcoded table | No | ✅ Complete |
| 13 | Capacity Level | String list | Capacity Level | M hardcoded table | Approved | ✅ Complete |
| 14 | End Month | Date | End Month | M date list 2020–2025 | 2025-02-01 | ✅ Complete |
| 15 | Month Range (Custom toggle) | String list | Month Range (2) | M Base64 compressed | Default | ✅ Complete |

**Parameter Conversion Rate: 14/15 = 93%**
**Gap:** Parameter 8 (Aggregation View) – DAX `Aggregation Calculation` column hardcodes `Week Date`; Monthly branch not dynamically switched in the column itself (handled separately via `Parameter` table + `NAMEOF()`).

---

## 4. CALCULATED COLUMN MAPPING

### 4.1 SDOP Capacity Slot Allocation_Custom SQL Query (Main Fact Table)

| # | Column Name | Tableau Original Expression | Power BI DAX Expression | Status |
|---|---|---|---|---|
| 1 | Slots Redefined | `IF LOWER(slots)='site capacity' THEN 'capacity' ELSEIF LOWER(slots)='used' THEN 'collected' ELSEIF LOWER(slots)='booked' THEN 'confirmed' ELSE slots END` | `IF(LOWER(Slots)="site capacity","capacity", IF(LOWER(Slots)="used","collected", IF(LOWER(Slots)="booked","confirmed", Slots)))` | ✅ Complete |
| 2 | Channel Name Redefined | `IF LOWER(channel_name)='clinical' AND LOWER(Slots Redefined)='confirmed' THEN 'Clinical Sch.' ELSEIF lower(channel_name)='requested' AND LOWER(Slots Redefined)='confirmed' THEN 'Clinical Rsvn.' ELSE channel_name END` | `IF(LOWER(Channel Name)="clinical" && LOWER(Slots Redefined)="confirmed","Clinical Sch.", IF(LOWER(Channel Name)="requested" && LOWER(Slots Redefined)="confirmed","Clinical Rsvn.", Channel Name))` | ✅ Complete |
| 3 | Protocol 1 | `IFNULL([protocol], 'Capacity')` | `COALESCE(Protocol1, "Capacity")` | ✅ Complete |
| 4 | Details 1 | `IFNULL([details], 'Capacity')` | `COALESCE(Details1, "Capacity")` | ✅ Complete |
| 5 | Details (group) | `IF Details 1 IN {Actuals:*} THEN 'Actuals' ELSEIF {Allocation:*} THEN 'Allocations' ELSEIF {Cancels:*} THEN 'Cancels' ELSEIF {Capacity/Site Capacity} THEN 'Capacity' END` | `SWITCH(TRUE(), Details 1 IN {...} → "Actuals/Allocations/Cancels/Capacity", BLANK())` | ✅ Complete |
| 6 | Channel Name Redefined (group) | `IF Channel Name Redefined IN {APS} THEN 'APS' ELSEIF {Clinical*} THEN 'Clinical' ... END` | `SWITCH(TRUE(), Channel Name Redefined IN {APS}→"APS", {Clinical*}→"Clinical", {Commercial}→"Commercial", {NHU*}→"NHU", {Converted/SITE:All}→"Others (exclude)")` | ✅ Complete |
| 7 | Channel | `[Details (group)] + ': ' + [Channel Name Redefined]` | `Details (group) & ": " & Channel Name Redefined` | ✅ Complete |
| 8 | Booked | `INT(IF [Slots Redefined]='confirmed' THEN [value] ELSE NULL END)` | `INT(IF(Slots Redefined="confirmed", Value, BLANK()))` | ✅ Complete |
| 9 | Allocated | `IF [Slots Redefined]='allocated' THEN [value] END` | `IF(Slots Redefined="allocated", Value, BLANK())` | ✅ Complete |
| 10 | Used | `INT(IF [Slots Redefined]='collected' THEN [value] ELSE NULL END)` | `INT(IF(Slots Redefined="collected", Value, BLANK()))` | ✅ Complete |
| 11 | Cancelled | `INT(IF [Slots Redefined]='cancelled' THEN [value] ELSE NULL END)` | `INT(IF(Slots Redefined="cancelled", Value, BLANK()))` | ✅ Complete |
| 12 | Site Capacity | `INT(IF [Slots Redefined]='capacity' THEN [value] ELSE NULL END)` | `INT(IF(Slots Redefined="capacity", Value, BLANK()))` | ✅ Complete |
| 13 | Aggregation Calculation | `IF [Parameter 8]="Weekly" THEN DATE([week_date]) ELSE DATE([scope_calendar_month]) END` | `DATEVALUE(Week Date)` | ⚠️ Partial – always returns Week Date; Monthly branch not implemented in column |
| 14 | Confirmed Alloc. Week | `DATE(IF [Parameter 9]='Monday' THEN DATETRUNC('week', confirmed_allocation_week_date, 'Monday') ELSE DATETRUNC('week', confirmed_allocation_week_date, 'Sunday') END)` | `IF(MIN(Week Starts[ParameterTableKeyColumn])="Monday", date - WEEKDAY(date,2)+1, IF(MIN(...)="Sunday", date - WEEKDAY(date,1)+1, BLANK()))` | ✅ Complete |
| 15 | Consumption - Channel - Geo | `IF [Slots Redefined] IN {collected,confirmed,cancelled} THEN UPPER(Slots Redefined)+': '+Channel Name Redefined+'-'+IFNULL(region_code, geography_code) ELSE ' UNCONSUMED' END` | `IF(Slots Redefined IN {"collected","confirmed","cancelled"}, UPPER(Slots Redefined) & ": " & Channel Name Redefined & "-" & COALESCE(Region Code, Geography Code), " UNCONSUMED")` | ✅ Complete |
| 16 | Metrics | `IF Slots IN {booked,used} THEN 'Actuals' ELSEIF IN {allocated} THEN 'Allocations' ... END` | `SWITCH(TRUE(), Slots IN {booked,used}→"Actuals", {allocated}→"Allocations", {cancelled}→"Cancels", {site capacity}→"Capacity", BLANK())` | ✅ Complete |
| 17 | testt | (Slot Redefined + Channel Name Redefined grouping key) | `IF(Slots Redefined IN {collected,confirmed,cancelled,allocated}, UPPER(Slots Redefined) & ": " & Channel Name Redefined, UPPER(Slots Redefined) & ": " & Product Group)` | ✅ Complete |

### 4.2 Details Actuals and Cancels_Custom SQL Query (Secondary Fact Table)

| # | Column Name | Logic | Status |
|---|---|---|---|
| 1 | Metrics | `SWITCH(Slots IN {booked,used}→"Actuals", {cancelled}→"Cancels")` | ✅ Complete |
| 2 | Slots Redefined | Same as main fact table | ✅ Complete |
| 3 | testt1 | `IF(Slots Redefined IN {collected,confirmed,cancelled,allocated}, UPPER(Slots Redefined) & ": " & Channel Name, "NA")` | ✅ Complete |
| 4 | Date columns (5) | `Text.Middle(json_field, 0, 10)` → typed as date | ✅ Complete (M step) |

### 4.3 Data Source Refresh Timestamp_Custom SQL Query

| # | Column Name | Tableau Original | Power BI DAX | Status |
|---|---|---|---|---|
| 1 | Refersh Timestamp with TZ | `STR([refresh_timestamp]) + " " + "UTC"` | `FORMAT([refresh_timestamp], "0") & " UTC"` | ✅ Complete |

### 4.4 refresh Table

| # | Column Name | Logic | Status |
|---|---|---|---|
| 1 | tt | `SWITCH(source, "APEX", "", refresh_timestamp & " UTC")` | ✅ Complete |

**Calculated Column Conversion Rate: 22/23 = 96%**
**Gap:** `Aggregation Calculation` — Monthly branch of Parameter 8 not implemented in the DAX column.

---

## 5. DAX MEASURE MAPPING

### 5.1 Core Metric Chain (Measures Table)

| # | Measure Name | Tableau Equivalent | DAX Expression Summary | Layer | Status |
|---|---|---|---|---|---|
| 1 | Allocations | SUM(Allocated) | `CALCULATE(SUM(Value), Metrics="Allocations")` | Base | ✅ |
| 2 | Used | SUM(Used) | `CALCULATE(SUM(Value), Slots IN {"used","booked"})` | Base | ✅ |
| 3 | Capacities | SUM(Capacity) | `CALCULATE(SUM(Value), Metrics="Capacity")` | Base | ✅ |
| 4 | Allocatedv1 | Allocated (date ranged) | IF Default → apply startmonth/Endmonth window on Allocations, ELSE Allocations | v1 – Date range | ✅ |
| 5 | Consumedv1 | Consumed (date ranged) | IF Default → apply startmonth/Endmonth window on Actuals SUM, ELSE Actuals SUM | v1 – Date range | ✅ |
| 6 | Cancelsv1 | Cancels (date ranged) | IF Default → apply startmonth/Endmonth window on Cancels SUM, ELSE Cancels SUM | v1 – Date range | ✅ |
| 7 | Capacityv1 | Capacity (date ranged) | IF Default → apply startmonth/Endmonth window on Capacities, ELSE Capacities | v1 – Date range | ✅ |
| 8 | Allocatedv2 | Allocated (re-aph filtered) | IF Aph Include Re-Aphs="No" → Allocatedv1 minus re-aph rows, ELSE Allocatedv1 | v2 – Re-aph | ✅ |
| 9 | Consumedv2 | Consumed (re-aph filtered) | IF Aph Include Re-Aphs="No" → Consumedv1 minus re-aph rows, ELSE Consumedv1 | v2 – Re-aph | ✅ |
| 10 | Cancelsv2 | Cancels (re-aph filtered) | IF Aph Include Re-Aphs="No" → Cancelsv1 minus re-aph rows, ELSE Cancelsv1 | v2 – Re-aph | ✅ |
| 11 | Capacityv2 | Capacity (re-aph filtered) | IF Aph Include Re-Aphs="No" → Capacityv1 minus re-aph rows, ELSE Capacityv1 | v2 – Re-aph | ✅ |
| 12 | Allocatedv3 | Allocated (snapshot filtered) | IF Aph Allocation Snapshot="SD&OP" → exclude Details1 IN {"Allocation: Updated"}, ELSE exclude {"Allocation: SD&OP"} | v3 – Snapshot | ✅ |
| 13 | Consumedv3 | Consumed (snapshot filtered) | Same snapshot filter on Consumedv2 | v3 – Snapshot | ✅ |
| 14 | Cancelsv3 | Cancels (snapshot filtered) | Same snapshot filter on Cancelsv2 | v3 – Snapshot | ✅ |
| 15 | Capacityv3 | Capacity (level filtered) | IF Aph Allocation Snapshot value1="Approved" → filter Level="approved", ELSE Level="operational" | v3 – Level | ✅ |
| 16 | Unconsumedv1 | Unconsumed (base) | IF Allocatedv1 ≠ 0 → Allocatedv1 - Consumedv1 | Derived | ✅ |
| 17 | Unconsumedv2 | Unconsumed (re-aph) | INT(IF Allocatedv2 ≠ 0 → Allocatedv2 - Consumedv2) | Derived | ✅ |
| 18 | Unconsumedv3 | Unconsumed (snapshot) | IF Allocatedv3 ≠ 0 → Allocatedv3 - Consumedv3 | Derived | ✅ |
| 19 | Unconsumedv4 / consum1 | Unconsumed (floored) | IF Unconsumedv3 < 0 → 0, ELSE Unconsumedv3 | Floored | ✅ |
| 20 | Slot Utilization | Slot Utilization | `DIVIDE(Consumedv3, Allocatedv3)` | KPI | ✅ |
| 21 | Allocation Accuracyv1 | Allocation Accuracy | `1 - ABS(Consumed - Allocated)/Consumed`, capped at 1 | KPI | ✅ |
| 22 | Utilization (on fact table) | SUM(Used)/SUM(Allocated) | `DIVIDE(SUM(Used), SUM(Allocated))` | Simple | ✅ |

### 5.2 Presentation / Display Measures

| # | Measure | Logic | Status |
|---|---|---|---|
| 1 | Showanalysis | IF Show Analysis="Slot Utilization" → [Slot Utilization], IF="Allocation Accuracy" → [Allocation Accuracyv1] | ✅ |
| 2 | measureselection | SWITCH(Metrics, Allocations→Allocatedv3, Actuals→Consumedv3, Cancels→Cancelsv3, Capacity→Capacityv3) | ✅ |
| 3 | Val | IF Consumption-Channel-Geo=" UNCONSUMED" → Unconsumedv4, ELSE Consumedv3 | ✅ |
| 4 | Val1_1 | IF Consumption-Channel-Geo=" UNCONSUMED" → Allocatedv3-Consumedv2, ELSE Consumedv3 | ✅ |
| 5 | Valv1 | Val with Default/Custom date window | ✅ |
| 6 | Valv1_2 | Val1_1 with Default/Custom date window | ✅ |
| 7 | val2 | IF NOT ISINSCOPE(Details group) AND NOT ISINSCOPE(testt) → Allocatedv3, ELSE Valv1 | ✅ |
| 8 | val2_1 | Same as val2 using DTG[Metrics] and Slotbychannel[testt] | ✅ |
| 9 | val3 | IF Details(group)="Allocations" → Unconsumedv4, ELSE Consumedv3 | ✅ |
| 10 | val3_1 | Same as val3 using DTG[Metrics] | ✅ |
| 11 | yyy | SUM(Value) with Default/Custom date window (raw, no metric filter) | ✅ |
| 12 | yyy1 | yyy filtered by Capacity Level (Approved/Operational) via DTG selection | ✅ |
| 13 | yyy_2 | Scope-aware: IF NOT ISINSCOPE → Allocatedv3, ELSE yyy1 | ✅ |
| 14 | Title1 | "Capacity & Slot Utilization - by Apheresis date (" & product & ")" | ✅ |
| 15 | Title2 | "Capacity & Slot Utilization - by Manufacturing date (" & product & ")" | ✅ |
| 16 | Title3 | "Capacity and Slot Utilization -Detailed - by Apheresis date (" & product & ")" | ✅ |
| 17 | Title4 | "Capacity and Slot Utilization -Detailed - by Manufacturing date (" & product & ")" | ✅ |
| 18 | Definitions | Static text – Allocated/Consumed/Unconsumed definitions with source notes | ✅ |
| 19 | Row Number | RANKX(ALL(Measures Table), 1) | ✅ |
| 20 | Refresh Timestamp | = 1 (hidden placeholder) | ✅ |

### 5.3 WIP / Debug Measures (not in Tableau)

| # | Measure | Purpose | Status |
|---|---|---|---|
| 1 | test | WIP version of Allocatedv1 | ⚠️ WIP – not mapped |
| 2 | ff | IF Consumedv3<20 AND ISBLANK(Unconsumedv4) → 1 | ⚠️ WIP – visual flag |
| 3 | sel | IF val2>200 → 1 | ⚠️ WIP – threshold flag |
| 4 | sel1 | IF DTG Metrics="Allocations" → 1, ELSE 2 | ⚠️ WIP |
| 5 | slicerfil | COUNTROWS(SDOP fact table) | ⚠️ WIP – debug helper |
| 6 | filcheckk | Cross-date-context row count match | ⚠️ WIP – debug helper |
| 7 | b | BLANK() | ⚠️ WIP – placeholder |
| 8 | fffff (on Details table) | COUNTROWS with Default/Custom date range | ⚠️ WIP |

**DAX Measure Conversion Rate: 42/50 = 84%**
**Note:** 8 WIP/debug measures exist in Power BI with no Tableau counterpart — these are new additions during migration, not gaps.
**Confirmed Tableau equivalents converted: 42/42 = 100%**

---

## 6. POWER QUERY (M) TRANSFORMATION MAPPING

### 6.1 Main Fact Table – SDOP Capacity Slot Allocation_Custom SQL Query

| Step | Tableau Equivalent | Power Query (M) | Status |
|---|---|---|---|
| Connect to PostgreSQL | Tableau Live/Extract connection to ct_bia | `PostgreSQL.Database("bms-eim-cell-therapy-auroradb-read-prod...", "postgres")` | ✅ |
| Filter latest extract_date | Tableau data source filter | `WHERE extract_date = (SELECT MAX(extract_date)...) OR extract_date IS NULL` (in SQL) | ✅ |
| Cross join confirmed_allocation_week | Tableau join | `CROSS JOIN ct_bia.sdop_confirmed_allocation_week` (in SQL) | ✅ |
| Derive product_group_redef | Tableau calculated field | `CASE WHEN lower(product_group) LIKE '%early phase%'...` (in SQL) | ✅ |
| Native query folding | N/A (Tableau handles server-side) | `Value.NativeQuery(..., [EnableFolding=true])` | ✅ |

### 6.2 Secondary Fact Table – Details Actuals and Cancels_Custom SQL Query

| Step | Tableau Equivalent | Power Query (M) | Status |
|---|---|---|---|
| Filter booked/used/cancelled | Tableau data source filter | `WHERE slots IN ('booked','used','cancelled') AND scope_calendar_month IS NOT NULL` | ✅ |
| JSON array expansion | Tableau's json_array_elements | `json_array_elements(detailed_rows) ->> 'field'` (in SQL) | ✅ |
| Extract date substrings | Tableau date calc | `Table.AddColumn(..., each Text.Middle([field], 0, 10))` ×5 | ✅ |
| Type cast date columns | Tableau type cast | `Table.TransformColumnTypes(..., {date} ×5)` | ✅ |
| Remove raw JSON string columns | Tableau hide fields | `Table.RemoveColumns(...)` | ✅ |

### 6.3 Parameter Tables – M Pattern (Repeated)

| Pattern | M Code Pattern | Tableau Equivalent |
|---|---|---|
| Hardcoded key-value pair list | `Table.FromRows({{key, value},...}, {"ParameterTableKeyColumn","Value"})` | Tableau Parameter with fixed list |
| Default value filter (GUID-based) | `Table.SelectRows(..., each [Value/Key] = #"guid-string")` | Tableau Parameter default selection |
| Date range list | `List.Generate(() => StartDate, each _ <= EndDate, each Date.AddMonths(_, 1))` | Tableau date parameter with range |
| Distinct dimension from fact | `Table.Distinct(Table.SelectColumns(Source, {"col"}))` + remove blanks | Tableau dimension member list |
| Base64 compressed table | `Binary.Decompress(Binary.FromText("...", BinaryEncoding.Base64), Compression.Deflate)` | Tableau hardcoded parameter |

### 6.4 Refresh / Metadata Tables

| Step | Tableau Equivalent | Power Query (M) / SQL | Status |
|---|---|---|---|
| GPS timestamp | Tableau extract refresh timestamp | `SELECT DISTINCT 'GPS', to_char(_refresh_timestamp,...)` | ✅ |
| APEX timestamp | Tableau extract refresh timestamp | `SELECT 'APEX', to_char(max(pg_xact_commit_timestamp(xmin)),...)` | ✅ |
| Site Capacity timestamp | Tableau extract refresh timestamp | `SELECT 'Site Capacity', to_char(max(extract_date),...)` | ✅ |

**Power Query Conversion Rate: 18/18 = 100%**

---

## 7. DIMENSION / SLICER TABLES

| Table | Column | Source | Tableau Equivalent | Pattern |
|---|---|---|---|---|
| Channel | channel_name | DISTINCT from fact | channel_name dimension | DISTINCT + blank removal |
| Geography Code | geography_code | DISTINCT from fact | geography_code dimension | DISTINCT + blank removal |
| Site Id | site_id | DISTINCT from fact | site_id dimension | DISTINCT + blank removal |
| Products | product | DISTINCT from fact | product dimension | DISTINCT (no blank removal) |
| Aph Allocation Sanpshot | level | DISTINCT from fact | Level dimension (typo table) | DISTINCT (via M) |
| Detail | Details 1 | ALL() from fact | Details 1 dimension | DAX ALL() |
| Slotbychannel | testt | DISTINCT() from fact | Slot+Channel grouping key | DAX DISTINCT() |
| DTG | Metrics | DISTINCT() from fact | Metrics grouping | DAX DISTINCT() + sort col |
| PP | product | ALL() from fact | Product LOD override | DAX ALL() |
| Protocol | Protocol 1 | ALL() from fact | Protocol filter bypass | DAX ALL() |
| WeekStart | week_start | ALL() from fact | Week start filter bypass | DAX ALL() |

**⚠️ Naming issue:** `Aph Allocation Sanpshot` (typo) and `Aph Allocation Snapshot` (correct) both exist.
**Recommendation:** Consolidate into single table and fix typo.

---

## 8. CALCULATED TABLES (DAX)

| Table | DAX Source Expression | Purpose | Tableau Equivalent |
|---|---|---|---|
| Parameter | `{("Monthly", NAMEOF(Month), 0), ("Weekly", NAMEOF(Week Date), 1)}` | Time granularity axis switcher | Parameter 8 (Monthly/Weekly) |
| DTG | `DISTINCT('SDOP...'[Metrics])` + `srtclm` sort column | Metrics slicer with display order | Metrics dimension + sort |
| PP | `ALL('SDOP...'[product])` | Unfiltered product list | Fixed LOD product bypass |
| Protocol | `ALL('SDOP...'[Protocol 1])` | Unfiltered protocol list | Fixed LOD protocol bypass |
| Slotbychannel | `DISTINCT('SDOP...'[testt])` | Slot-by-channel grouping values | testt dimension |
| WeekStart | `ALL('SDOP...'[week_start])` | Unfiltered week_start list | week_start filter bypass |
| Detail | `ALL('SDOP...'[Details 1])` | Unfiltered details list | Details 1 filter bypass |
| Date | `CALENDARAUTO()` | Calendar spine | Tableau date dimension |

### DTG Sort Order (srtclm column)
| Metrics Value | Sort Order |
|---|---|
| Capacity | 1 |
| Allocations | 2 |
| Actuals | 3 |
| Cancels | 4 |

---

## 9. REFRESH / METADATA TABLES

### 9.1 Data Sources Covered
| Source Label | PostgreSQL Table | Timestamp Logic |
|---|---|---|
| GPS | ct_bia.raw_treatment_fc | `MAX(_refresh_timestamp)` formatted as `YYYY-MM-DD HH24:MI:SS` |
| APEX | ct_bia.allocations_planning_aph_slot_allocations | `MAX(pg_xact_commit_timestamp(xmin))` formatted |
| Site Capacity | ct_bia.sdop_capacity_slot_allocation | `MAX(extract_date)` formatted as `YYYY-MM-DD 00:00:00` |

### 9.2 Display Column Comparison

| Table | Display Column | Formula | Suppresses APEX? |
|---|---|---|---|
| Data Source Refresh Timestamp_Custom SQL Query | Refersh Timestamp with TZ | `FORMAT(refresh_timestamp,"0") & " UTC"` | No – shows all |
| refresh | tt | `SWITCH(source, "APEX", "", refresh_timestamp & " UTC")` | Yes – APEX blank |

**⚠️ Duplicate table issue:** Both tables run the same SQL query. Recommend consolidating to one table with both display columns.

---

## 10. CONVERSION COMPLETENESS SUMMARY

### 10.1 Conversion by Category

| Category | Tableau Items | Converted | Partial | Not Converted | Conversion % |
|---|---|---|---|---|---|
| Parameters | 15 | 14 | 1 | 0 | **93%** |
| Calculated Columns | 23 | 22 | 1 | 0 | **96%** |
| DAX Measures (Tableau-sourced) | 42 | 42 | 0 | 0 | **100%** |
| Power Query Transformations | 18 | 18 | 0 | 0 | **100%** |
| Relationships | 23 | 23 | 0 | 0 | **100%** |
| Dimension/Slicer Tables | 11 | 11 | 0 | 0 | **100%** |
| Calculated Tables | 8 | 8 | 0 | 0 | **100%** |
| Metadata/Refresh Tables | 3 | 3 | 0 | 0 | **100%** |

### 10.2 Overall Conversion Rate

| Metric | Value |
|---|---|
| **Total Tableau artefacts** | **143** |
| **Fully converted** | **141** |
| **Partially converted** | **2** |
| **Not converted** | **0** |
| **Overall Conversion Rate** | **🟢 98.6%** |

### 10.3 Partial Conversions Detail

| # | Item | Gap Description | Recommended Fix |
|---|---|---|---|
| 1 | Parameter 8 – Aggregation Calculation (column) | DAX column always returns `DATEVALUE(Week Date)`; Monthly branch (`scope_calendar_month`) dropped. The `Parameter` table with `NAMEOF()` partially compensates for axis switching in visuals, but the column itself is not dynamic. | Replace with: `IF(SELECTEDVALUE('Parameter'[Parameter Fields]) = NAMEOF(SDOP...[Month]), SDOP...[Month], SDOP...[Week Date])` |
| 2 | Month Range (Parameter 4) – Filtered Rows GUID | Default selection uses a GUID reference (`#"2077fd8c-..."`) that may not resolve correctly outside the original dataset. | Replace GUID with explicit value: `each [Value] = "Default"` |

---

## 11. KNOWN GAPS & RECOMMENDATIONS

### 11.1 Bugs / Issues to Fix

| Priority | Issue | Location | Recommendation |
|---|---|---|---|
| 🔴 High | Typo: `Aph Allocation Sanpshot` | Table name | Rename to `Aph Allocation Snapshot` and update all references |
| 🔴 High | Duplicate refresh tables | `Data Source Refresh Timestamp_Custom SQL Query` + `refresh` | Consolidate to one table; keep both display columns |
| 🟡 Medium | `Aggregation Calculation` column hardcodes Weekly | SDOP fact table | Add dynamic branch for Monthly (see Section 10.3) |
| 🟡 Medium | GUID-based default filters | `Product`, `Show Accuracy`, `Week Starts`, `Month Range` | Replace GUIDs with explicit value references for maintainability |
| 🟡 Medium | WIP measures not cleaned up | Measures Table | Review and remove: `test`, `ff`, `sel`, `sel1`, `slicerfil`, `filcheckk`, `b`, `fffff` |
| 🟢 Low | Column `tt` suppresses APEX timestamp | `refresh` table | Document or align behavior with `Refersh Timestamp with TZ` column |
| 🟢 Low | Typo in column name: `Refersh Timestamp with TZ` | `Data Source Refresh Timestamp_Custom SQL Query` | Rename to `Refresh Timestamp with TZ` |

### 11.2 Tableau Patterns Successfully Migrated

| Tableau Pattern | Power BI Equivalent | Notes |
|---|---|---|
| IFNULL(x, y) | COALESCE(x, y) | Fully converted in all 2 instances |
| STR(field) + " UTC" | FORMAT(field, "0") & " UTC" | Equivalent output |
| DATETRUNC('week', date, day) | date - WEEKDAY(date, mode) + 1 | Fully equivalent for Monday/Sunday modes |
| Parameter list (fixed) | M hardcoded Table.FromRows | All 15 parameters converted |
| Parameter default | GUID-filtered Table.SelectRows | Works but fragile – see recommendations |
| LOD FIXED expressions | DAX ALL() calculated tables | PP, Protocol, WeekStart, Detail |
| SUM(x) / SUM(y) | DIVIDE(SUM(x), SUM(y)) | Division-safe |
| Slot grouping logic | SWITCH(TRUE(), ...) | All 4 Metrics categories preserved |
| Dynamic title calc | DAX string measure | All 4 titles converted |
| json_array_elements | SQL-level JSON expansion + Text.Middle() in M | Full fidelity |
| Extract date filter | SQL MAX(extract_date) subquery | Preserves latest snapshot logic |
| product_group_redef logic | SQL CASE WHEN + product_group_redef column | Early phase product handling preserved |

### 11.3 New Power BI Capabilities Added (Not in Tableau)

| Feature | Description |
|---|---|
| ISINSCOPE() | Used in val2, val2_1, yyy_2 to detect visual drill level – not possible natively in Tableau |
| NAMEOF() for axis switching | Parameter table uses NAMEOF() to reference field objects – enables dynamic axis without hardcoding |
| Measure versioning chain (v1→v2→v3) | Layered DAX composition makes each filter layer explicit and testable |
| Bidirectional relationship | Aph Allocation Snapshot ↔ Capacity Level enables cross-filter in both directions |
| CALENDARAUTO() | Auto-detects model date range for calendar spine – more maintainable than Tableau date spine |

---

## 12. MODEL METADATA

| Property | Value |
|---|---|
| PBIP Model Name | 19863-_WIP_ Capacity and Slot Allocations |
| Compatibility Level | 1604 |
| Culture | en-US |
| Time Intelligence | Disabled |
| Total Tables | 32 |
| Total Relationships | 23 |
| Total DAX Measures | 50 (42 production + 8 WIP) |
| Total Calculated Columns | 23 |
| Total Calculated Tables (DAX) | 8 |
| Total Parameter Tables (M) | 15 |
| Total Dimension Tables (M) | 5 |
| Total Fact Tables | 2 |
| Total Metadata Tables | 2 |
| Primary Data Source | PostgreSQL – bms-eim-cell-therapy-auroradb-read-prod.web.bms.com:5432 |
| Primary Schema | ct_bia |
| Primary Tables Used | sdop_capacity_slot_allocation, raw_treatment_fc, allocations_planning_aph_slot_allocations, sdop_confirmed_allocation_week |
| Connections File | Dataset ID + Report ID + Workspace Object ID |

---

*End of Knowledge File – 19863 WIP Capacity and Slot Allocations – Tableau → PBIP Conversion*
*Generated by BMS Gen AI Storefront | 2026-07-03*
