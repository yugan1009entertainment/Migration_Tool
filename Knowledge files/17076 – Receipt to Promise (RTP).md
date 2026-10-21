# 📘 CONVERSION KNOWLEDGE FILE
# 17076 – Receipt to Promise (RTP)
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-09
# Source files: 17076_receipt_to_promise__rtp_.txt + all PBIP .txt files
#               (12_database.txt, 12_model.txt, 12_definition.json,
#                12_.json, 12_relationships.txt, 12_expressions.txt,
#                t_rtp_final_yi__gpssc__t_rtp_final_yi_temp.txt,
#                t_rtp_final_yi__gpssc__t_rtp_rr_attr.txt,
#                sss_parameter.txt, end_date__copy_2_.txt,
#                localdatetable_7699c87b...txt × 4,
#                datetabletemplate_797db397...txt)

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 0 — WORKBOOK & MODEL METADATA
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Tableau Workbook ID       : 17076
Tableau Title             : Receipt to Promise (RTP)
Tableau Revision          : 3.9
Tableau Build             : 2024.2 (20242.25.0710.1539)

PBIP Dataset Name         : 29913_17076-Receipt to Promise _RTP_ dataset
PBIP Model Name           : 29913_17076-Receipt to Promise _RTP_- model
compatibilityLevel        : 1604
  ⚠️  ARTIFACT: Model comment reads "1550 compatibility level" but
      file declares 1604. Both values preserved as-is.
Culture                   : en-US
sourceQueryCulture        : en-US
defaultPowerBIDataSourceVersion : powerBI_V3
dataAccessOptions         : legacyRedirects, returnErrorValuesAsNull
Time Intelligence         : Enabled (__PBI_TimeIntelligenceEnabled = 1)
DevMode                   : Yes (PBI_ProTooling = ["DevMode"])
Model Comment             : /// A Demo Tabular data model with 1550
                            compatibility level.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — DATA SOURCES & POWER QUERY (M)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### DS-01 | t_RTP_Final_Yi_temp  (MySQL — Import)
─────────────────────────────────────────────────────────────
Tableau Datasource Name : t_RTP_Final_Yi (gpssc)
PBIP Table Name         : t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp
Source Type             : MySQL
Server                  : v490721-infocenter-rds-prod.c9pxjgrv7vhh
                          .us-east-1.rds.amazonaws.com:1521
Database                : gpssc
Table                   : t_RTP_Final_Yi_temp
Mode                    : Import (Tableau .hyper extract present)
lineageTag              : 71bbf6d2-6bf8-4a3f-803e-cc0c8072a2df
sourceLineageTag        : federated.15pxieq1i1e4291eq3j2i0b4jofs

Power BI M Query (Partition):
─────────────────────────────
let
    Source = MySQL.Database(
        "v490721-infocenter-rds-prod.c9pxjgrv7vhh.us-east-1.rds.amazonaws.com:1521",
        "gpssc"
    ),
    #"gpssc_t_RTP_Final_Yi_temp" =
        Source{[Schema = "gpssc", Item = "t_RTP_Final_Yi_temp"]}[Data],
    #"Removed Other Columns" = Table.SelectColumns(
        #"gpssc_t_RTP_Final_Yi_temp",
        {
            "Receiving.Plant.Name",
            "Supplying.Plant.and.Description",
            "Vendor_Number",
            "Material",
            "Material Desc",
            "PO.Line.item",
            "Stat Delivery Date",
            "Last.Received.Date",
            "Fully.Received.Date",
            "Brand",
            "Delivery_Complete",
            "Material.Type",
            "On.time.Qty",
            "line_qty",
            "total_received_qty",
            "Units Under (-)/Over (+)",
            "date_early_late",
            "include_exclude_flag",
            "DOSAGE_STRENGTH"
        }
    )
in
    #"Removed Other Columns"

/***********************
 * DATASOURCE FILTERS  (documented in source comment — not enforced in M)
 ***********************/
// Filter 1 ----------------------------
// TableName   : t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp
// ColumnName  : Stat Delivery Date_Month-Trunc
// FilterType  : Advanced  |  Operator: And
// Conditions  :
//   - LessThanOrEqual    : #2026-05-01 00:00:00#
//   - GreaterThanOrEqual : #2025-01-01 00:00:00#
//   - IsBlank            : (null rows pass through)

Columns Dropped from Source (hidden in Tableau, excluded from M):
  receiving_plant_desc, supplying_plant_desc, vendor_desc,
  Current_Date, day_diff, Rank, POCount


### DS-02 | t_rtp_rr_attr  (Amazon Redshift — Import)
─────────────────────────────────────────────────────────────
Tableau Datasource Name : t_rtp_rr_attr
PBIP Table Name         : t_RTP_Final_Yi (gpssc)_t_rtp_rr_attr
Source Type             : Amazon Redshift
Endpoint                : bms-gpsdl-non-validated-redshift-prod
                          .web.bms.com:5439
Database                : gpsdb
Schema                  : gps_bia_sc
Table                   : t_rtp_rr_attr
Mode                    : Import
lineageTag              : 3c8a8da4-ab3f-4d74-a828-e36b8b2eff28

Power BI M Query (Partition):
─────────────────────────────
let
    ClusterEndpoint = "bms-gpsdl-non-validated-redshift-prod.web.bms.com:5439",
    DatabaseName    = "gpsdb",
    SchemaName      = "gps_bia_sc",
    TableName       = "t_rtp_rr_attr",
    Source = AmazonRedshift.Database(
        ClusterEndpoint,
        DatabaseName,
        [BatchSize = null, Role = null, ProviderName = null]
    ),
    SchemaData = Source{[Name = SchemaName, Kind = "Schema"]}[Data],
    #"gps_bia_sc_t_rtp_rr_attr" = SchemaData{[Name = TableName]}[Data],
    #"Select Columns" = Table.SelectColumns(
        #"gps_bia_sc_t_rtp_rr_attr",
        {"cluster", "matl_sub_type", "dosage", "part"}
    )
in
    #"Select Columns"

Columns Dropped from Source (hidden in Tableau, excluded from M):
  part_descriptions, brand


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PARAM-01 | Start Date (date slicer)
─────────────────────────────────────────────────────────────
Tableau Internal Name : [End Date (copy)]
Tableau Caption       : Start Date
Tableau Type          : Date  |  Default: 2018-07-01
Tableau Range         : 2017-01-01 → 2021-06-30 (monthly)

⚠️  NAMING ARTIFACT: Internal name is "End Date (copy)" but caption
    is "Start Date". Names are inverted vs. semantics. Preserved as-is.

Power BI Table Name   : End Date (copy)      ← name preserved verbatim
Power BI Column       : ParameterTableKeyColumn  (date)
Power BI Usage        : Date slicer — lower bound of [Date range] column
                        (see CC-02)
Conversion Status     : ✅ FULL — artifact name preserved


### PARAM-02 | End Date (date slicer)
─────────────────────────────────────────────────────────────
Tableau Internal Name : [Line Qty Parameter]
Tableau Caption       : End Date
Tableau Type          : Date  |  Default: 2019-06-01
Tableau Range         : 2017-01-01 → 2021-06-30 (monthly)

⚠️  NAMING ARTIFACT: Internal name is "Line Qty Parameter" but caption
    is "End Date". Names are inverted vs. semantics. Preserved as-is.

Power BI Table Name   : Line Qty Parameter   ← name preserved verbatim
Power BI Column       : ParameterTableKeyColumn  (date)
Power BI Usage        : Date slicer — upper bound of [Date range] column
                        (see CC-02)
Conversion Status     : ✅ FULL — artifact name preserved


### PARAM-03 | SSS Numeric Selector
─────────────────────────────────────────────────────────────
Tableau Internal Name : [End Date (copy 2)]
Tableau Caption       : SSS (numeric confidence level)
Tableau Type          : Any (numeric)  |  Default: 32.8
Tableau Note          : Fixed-value numeric parameter — not a date.

⚠️  NAMING ARTIFACT: "End Date (copy 2)" is a date-like name used
    for a numeric SSS confidence value (32.8). Preserved as-is.

Expression GUID       : 0e2b167d-075e-4fa5-b7a4-a19fc988cc12
Expression DAX        : 32.8 meta [IsParameterQuery=true, List={32.8},
                        DefaultValue=32.8, Type=type number,
                        IsParameterQueryRequired=true]
lineageTag            : End Date  (copy 2)
  ⚠️  NOTE: Two spaces between "Date" and "(copy 2)" — preserved exactly.

Power BI Table Name   : End Date (copy 2)    ← name preserved verbatim
Power BI Column       : ParameterTableKeyColumn  (double)
Power BI M Source     : Table.FromRows({{32.8}}, {"ParameterTableKeyColumn"})
                        filtered to expression GUID above.
Conversion Status     : ✅ FULL — artifact name and double-space preserved


### PARAM-04 | SSS Parameter (GENERATESERIES slicer)
─────────────────────────────────────────────────────────────
Tableau Equivalent    : None — PBI-native GENERATESERIES slicer replacing
                        Tableau's fixed-value SSS numeric parameter.
Purpose               : User-selectable SSS confidence level 0–100 (step 0.1)

Power BI Table Name   : SSS Parameter
Power BI Type         : Calculated partition — Import
Power BI DAX Source   : GENERATESERIES(0, 100, 0.1)
Power BI Column       : SSS Parameter  (double, format 0.0,
                          sourceColumn [Value])
                          extendedProperty ParameterMetadata version=0
Power BI Measure      : SSS Parameter Value =
                          SELECTEDVALUE('SSS Parameter'[SSS Parameter], 32.8)
Default Value         : 32.8  (matches End Date (copy 2) expression value)
PBI_Id                : 30cc200fca3043bc8bef3ca7dd817758

Usage in model:
  - [SSS Parameter Value] returns the user-selected confidence level
    (defaulting to 32.8) for use in any SSS-driven measure or visual.

Conversion Status     : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — SOURCE COLUMN MAPPING
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp

| Src Column (MySQL)               | Tableau Caption              | PBIP Column Name                  | Type     | Hidden | SumBy | Ord |
|----------------------------------|------------------------------|-----------------------------------|----------|--------|-------|-----|
| Receiving.Plant.Name             | Receiving.Plant.Name1        | Receiving.Plant.Name1             | string   | No     | none  | 1   |
| Supplying.Plant.and.Description  | Supplying.Plant.and.Desc…1   | Supplying.Plant.and.Description1  | string   | No     | none  | 3   |
| Vendor_Number                    | Vendor Number                | Vendor Number                     | string   | No     | none  | 5   |
| Material                         | Material                     | Material                          | string   | No     | none  | 7   |
| Material Desc                    | (none)                       | Material Desc                     | string   | No     | none  | 8   |
| PO.Line.item                     | PO.Line.item                 | PO.Line.item                      | string   | No     | none  | 9   |
| Stat Delivery Date               | Stat Delivery Date           | Stat Delivery Date                | dateTime | No     | none  | 10  |
| Last.Received.Date               | (none)                       | Last.Received.Date                | string   | No     | none  | 11  |
| Fully.Received.Date              | (none)                       | Fully.Received.Date               | string   | No     | none  | 12  |
| Brand                            | Brand                        | Brand                             | string   | No     | none  | 13  |
| Delivery_Complete                | Delivery_Complete_Indicator  | Delivery_Complete_Indicator       | string   | No     | none  | 14  |
| Material.Type                    | Material.Type1               | Material.Type1                    | string   | No     | none  | 15  |
| On.time.Qty                      | (none)                       | On.time.Qty                       | double   | No     | sum   | 16  |
| line_qty                         | Line Qty                     | Line Qty                          | double   | No     | sum   | 17  |
| total_received_qty               | Total Received Qty           | Total Received Qty                | double   | No     | sum   | 18  |
| Units Under (-)/Over (+)         | Units Under (-)/Over (+)1    | Units Under (-)/Over (+)1         | double   | No     | sum   | 21  |
| date_early_late                  | Date Early Late              | Date Early Late                   | double   | No     | sum   | 22  |
| include_exclude_flag             | Include Exclude Flag         | Include Exclude Flag              | string   | No     | none  | 24  |
| DOSAGE_STRENGTH                  | Dosage Strength              | Dosage Strength                   | string   | No     | none  | 26  |

⚠️  NAMING ARTIFACTS (dot-in-name columns — PBI appends "1" suffix):
    Receiving.Plant.Name → Receiving.Plant.Name1
    Supplying.Plant.and.Description → Supplying.Plant.and.Description1
    Material.Type → Material.Type1
    Units Under (-)/Over (+) → Units Under (-)/Over (+)1


## TABLE: t_RTP_Final_Yi (gpssc)_t_rtp_rr_attr

| Src Column (Redshift) | Tableau Caption | PBIP Column Name | Type   | Hidden | SumBy | Ord |
|-----------------------|-----------------|------------------|--------|--------|-------|-----|
| cluster               | Cluster         | Cluster          | string | No     | none  | 31  |
| matl_sub_type         | Matl Sub Type   | Matl Sub Type    | string | No     | none  | 32  |
| dosage                | Dosage          | Dosage           | string | No     | none  | 33  |
| part                  | Part (hidden)   | part             | string | No     | none  | —   |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — TABLEAU CALCULATED FIELDS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All calculated columns reside in table:
  t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp

### CC-01 | Material 1
─────────────────────────────────────────────────────────────
PBI Column Name  : Material 1
PBI Data Type    : string
PBI lineageTag   : Calculation_1476617733722247168
PBI summarizeBy  : none

Tableau Expression:
───────────────────
[Material]+':'+' '+[Material Desc]

Power BI DAX:
─────────────
/// Material@@ - Column
column 'Material 1' =
't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Material]
& ":" & " "
& 't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Material Desc]
/* Original Tableau Expression is [Material]+':'+' '+[Material Desc] */

Depends on   : [Material], [Material Desc] (source columns)
Conversion   : ✅ FULL


### CC-02 | Date range
─────────────────────────────────────────────────────────────
PBI Column Name  : Date range
PBI Data Type    : boolean
PBI lineageTag   : Calculation_1756966843683663872
PBI summarizeBy  : none

Tableau Expression:
───────────────────
[Stat Delivery Date] <= [Parameters].[Line Qty Parameter]
AND [Stat Delivery Date] >= [Parameters].[End Date (copy)]

Power BI DAX:
─────────────
/// Date range@@ - Column
column 'Date range' =
IF(
    AND(
        't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Stat Delivery Date]
            <= SELECTEDVALUE('Line Qty Parameter'[ParameterTableKeyColumn]),
        't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Stat Delivery Date]
            >= SELECTEDVALUE('End Date  (copy)'[ParameterTableKeyColumn])
    ),
    TRUE(),
    FALSE()
)
/* Original Tableau Expression is [Stat Delivery Date] <=
   [Parameters].[Line Qty Parameter] and [Stat Delivery Date] >=
   [Parameters].[End Date  (copy)] */

⚠️  NOTE: References 'End Date  (copy)' with TWO spaces — must match
    exactly the table name artifact from PARAM-01.
Depends on   : [Stat Delivery Date], PARAM-01, PARAM-02
Used by      : All Action Filter proxy columns (see CC-17 – CC-30)
Conversion   : ✅ FULL


### CC-03 | 99% SSS  (Date Early Late base column)
─────────────────────────────────────────────────────────────
PBI Column Name  : 99% SSS
PBI Data Type    : double
PBI lineageTag   : Calculation_1882786160788426752
PBI summarizeBy  : sum

Tableau Expression:
───────────────────
2.33 * SQRT(SQUARE(STDEVP([date_early_late]))) + AVG([date_early_late])
  → mathematically = 2.33 * ABS(STDEVP) + AVG

Power BI DAX:
─────────────
/// 99% SSS - Column
column '99% SSS' =
VAR StdDevVal =
    STDEV.P('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Date Early Late])
VAR MeanVal =
    AVERAGE('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Date Early Late])
RETURN
    2.33 * ABS(StdDevVal) + MeanVal
/* Original Tableau Expression is
   2.33*SQRT(SQUARE(STDEVP([date_early_late]))) + AVG([date_early_late]) */

✅ CONVERSION NOTE: SQRT(SQUARE(x)) = ABS(x). Converted to STDEV.P + ABS()
   — mathematically equivalent.
Depends on   : [Date Early Late] (source column)
Conversion   : ✅ FULL


### CC-04 | 98% SSS
─────────────────────────────────────────────────────────────
PBI Column Name  : 98% SSS  |  lineageTag: Calculation_1882786160789549058
DAX: 2.05 * ABS(STDEV.P([Date Early Late])) + AVERAGE([Date Early Late])
/* Original Tableau Expression is
   2.05*SQRT(SQUARE(STDEVP([date_early_late]))) + AVG([date_early_late]) */
Conversion   : ✅ FULL


### CC-05 | 97% SSS
─────────────────────────────────────────────────────────────
PBI Column Name  : 97% SSS  |  lineageTag: Calculation_1882786160789774340
DAX: 1.88 * ABS(STDEV.P([Date Early Late])) + AVERAGE([Date Early Late])
Conversion   : ✅ FULL


### CC-06 | 96%SSS
─────────────────────────────────────────────────────────────
PBI Column Name  : 96%SSS  |  lineageTag: Calculation_1882786160792588296
  ⚠️  No space before % — naming artifact preserved.
DAX: 1.75 * ABS(STDEV.P([Date Early Late])) + AVERAGE([Date Early Late])
Conversion   : ✅ FULL


### CC-07 | 95%SSS
─────────────────────────────────────────────────────────────
PBI Column Name  : 95%SSS  |  lineageTag: Calculation_1882786160792489991
  ⚠️  No space before % — naming artifact preserved.
DAX: 1.65 * ABS(STDEV.P([Date Early Late])) + AVERAGE([Date Early Late])
Conversion   : ✅ FULL


### CC-08 | 99% SSS_LT  (floor at 0)
─────────────────────────────────────────────────────────────
PBI Column Name  : 99% SSS_LT
PBI lineageTag   : Calculation_1882786161118318604

Tableau Expression:
───────────────────
IF [Calculation_1882786160788426752] <0 THEN 0
ELSE [Calculation_1882786160788426752] END

Power BI DAX:
─────────────
/// 99% SSS_LT@@ - Column
column '99% SSS_LT' =
IF(
    't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[99% SSS] < 0,
    0,
    't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[99% SSS]
)
/* Original Tableau Expression is if [Calculation_1882786160788426752]
   <0 THEN 0 ELSE [Calculation_1882786160788426752] end */

Depends on   : CC-03 [99% SSS]
Conversion   : ✅ FULL


### CC-09 | 98% SSS_LT
─────────────────────────────────────────────────────────────
PBI Column Name  : 98% SSS_LT  |  lineageTag: Calculation_1882786161118818317
DAX: IF([98% SSS] < 0, 0, [98% SSS])
Depends on   : CC-04  |  Conversion: ✅ FULL


### CC-10 | 97% SSS_LT
─────────────────────────────────────────────────────────────
PBI Column Name  : 97% SSS_LT  |  lineageTag: Calculation_1882786161119502352
DAX: IF([97% SSS] < 0, 0, [97% SSS])
Depends on   : CC-05  |  Conversion: ✅ FULL


### CC-11 | 96%SSS_LT
─────────────────────────────────────────────────────────────
PBI Column Name  : 96%SSS_LT  |  lineageTag: Calculation_1882786161119129614
  ⚠️  No space before % — naming artifact preserved.
DAX: IF([96%SSS] < 0, 0, [96%SSS])
Depends on   : CC-06  |  Conversion: ✅ FULL


### CC-12 | 95%SSS_LT
─────────────────────────────────────────────────────────────
PBI Column Name  : 95%SSS_LT  |  lineageTag: Calculation_1882786161120190481
  ⚠️  No space before % — naming artifact preserved.
DAX: IF([95%SSS] < 0, 0, [95%SSS])
Depends on   : CC-07  |  Conversion: ✅ FULL


### CC-13 | 99% unit  (Units Under/Over base column)
─────────────────────────────────────────────────────────────
PBI Column Name  : 99% unit   ⚠️  lowercase 'u' — preserved exactly
PBI lineageTag   : Calculation_1882786161123381266

Tableau Expression:
───────────────────
2.33 * SQRT(SQUARE(STDEVP([Units Under (-)/Over (+)])))
+ AVG([Units Under (-)/Over (+)])

Power BI DAX:
─────────────
/// 99% Unit - Column
column '99% unit' =
VAR StdDevVal =
    STDEV.P('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'
        [Units Under (-)/Over (+)1])
VAR MeanVal =
    AVERAGE('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'
        [Units Under (-)/Over (+)1])
RETURN
    2.33 * ABS(StdDevVal) + MeanVal

Depends on   : [Units Under (-)/Over (+)1] (source column)
Conversion   : ✅ FULL


### CC-14 | 98% Unit
─────────────────────────────────────────────────────────────
PBI Column Name  : 98% Unit  |  lineageTag: Calculation_1882786161123840020
DAX: 2.05 * ABS(STDEV.P([Units Under (-)/Over (+)1]))
     + AVERAGE([Units Under (-)/Over (+)1])
Conversion   : ✅ FULL


### CC-15 | 97% Unit
─────────────────────────────────────────────────────────────
PBI Column Name  : 97% Unit  |  lineageTag: Calculation_1882786161125273627
DAX: 1.88 * ABS(STDEV.P([Units Under (-)/Over (+)1]))
     + AVERAGE([Units Under (-)/Over (+)1])
Conversion   : ✅ FULL


### CC-16 | 96% Unit
─────────────────────────────────────────────────────────────
PBI Column Name  : 96% Unit  |  lineageTag: Calculation_1882786161124421654
DAX: 1.75 * ABS(STDEV.P([Units Under (-)/Over (+)1]))
     + AVERAGE([Units Under (-)/Over (+)1])
Conversion   : ✅ FULL


### CC-17 | 95% Unit
─────────────────────────────────────────────────────────────
PBI Column Name  : 95% Unit  |  lineageTag: Calculation_1882786161123938325
DAX: 1.65 * ABS(STDEV.P([Units Under (-)/Over (+)1]))
     + AVERAGE([Units Under (-)/Over (+)1])
Conversion   : ✅ FULL


### CC-18 | 99%SSS_IF  (Units floor at 0)
─────────────────────────────────────────────────────────────
PBI Column Name  : 99%SSS_IF  |  lineageTag: Calculation_1882786161127665699
  ⚠️  No space before % — naming artifact preserved.

Tableau Expression:
───────────────────
IF [Calculation_1882786161123381266] <0 THEN 0
ELSE [Calculation_1882786161123381266] END

Power BI DAX:
─────────────
column 99%SSS_IF =
IF(
    't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[99% Unit] < 0,
    0,
    't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[99% unit]
)
/* Original Tableau Expression is if [Calculation_1882786161123381266]
   <0 THEN 0 ELSE [Calculation_1882786161123381266] end */

⚠️  CASE ARTIFACT: DAX references [99% unit] (lowercase 'u') in the ELSE
    branch while the IF branch uses [99% Unit] (uppercase 'U').
    The column is defined as '99% unit' (lowercase). Preserved as-is.
Depends on   : CC-13 [99% unit]
Conversion   : ✅ FULL (artifact preserved)


### CC-19 | 98%SSS_IF
─────────────────────────────────────────────────────────────
PBI Column Name  : 98%SSS_IF  |  lineageTag: Calculation_1882786161127497762
DAX: IF([98% Unit] < 0, 0, [98% Unit])
Depends on   : CC-14  |  Conversion: ✅ FULL


### CC-20 | 97% SSS_IF
─────────────────────────────────────────────────────────────
PBI Column Name  : 97% SSS_IF  |  lineageTag: Calculation_1882786161125601309
  ⚠️  Space before % on this one vs. no space on 96/95/98/99 — preserved.
DAX: IF([97% Unit] < 0, 0, [97% Unit])
Depends on   : CC-15  |  Conversion: ✅ FULL


### CC-21 | 96%SSS_IF
─────────────────────────────────────────────────────────────
PBI Column Name  : 96%SSS_IF  |  lineageTag: Calculation_1882786161126957088
DAX: IF([96% Unit] < 0, 0, [96% Unit])
Depends on   : CC-16  |  Conversion: ✅ FULL


### CC-22 | 95%SSS_IF
─────────────────────────────────────────────────────────────
PBI Column Name  : 95%SSS_IF  |  lineageTag: Calculation_1882786161127247905
DAX: IF([95% Unit] < 0, 0, [95% Unit])
Depends on   : CC-17  |  Conversion: ✅ FULL


### CC-23 | Stat Delivery Date_Month-Trunc
─────────────────────────────────────────────────────────────
PBI Column Name  : Stat Delivery Date_Month-Trunc
PBI Data Type    : dateTime
PBI lineageTag   : 6c45be3e-5a13-40d7-ae5b-df6ffff130fd
PBI summarizeBy  : none
Variation        : → LocalDateTable_b06e5be8 'Date Hierarchy'

Power BI DAX:
─────────────
column 'Stat Delivery Date_Month-Trunc' =
VAR follow_up_Month_TruncValue =
    COALESCE(
        't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Stat Delivery Date],
        BLANK()
    )
RETURN
    IF(
        ISBLANK(follow_up_Month_TruncValue),
        BLANK(),
        FORMAT(follow_up_Month_TruncValue, "MMMM YYYY")
    )

Purpose      : Monthly label for axis display; feeds LocalDateTable
               relationship and Stat Delivery Date_Date Hierarchy.
Conversion   : ✅ FULL


### CC-24 | Day of Stat Delivery Date
─────────────────────────────────────────────────────────────
PBI Column Name  : Day of Stat Delivery Date
PBI Data Type    : dateTime
PBI lineageTag   : c16015d4-5a4a-4edd-8143-5deed3d1ef97
PBI formatString : mmmm d\, yyyy
Variation        : → LocalDateTable_41fbab0d 'Date Hierarchy'

Power BI DAX:
─────────────
column 'Day of Stat Delivery Date' =
DATE(
    YEAR('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Stat Delivery Date]),
    MONTH('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Stat Delivery Date]),
    DAY('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Stat Delivery Date])
)

Conversion   : ✅ FULL


### CC-25 | Sort Order
─────────────────────────────────────────────────────────────
PBI Column Name  : Sort Order
PBI Data Type    : integer (format 0)
PBI lineageTag   : a793a831-645c-42a4-9e74-0a1e997a2d18

Power BI DAX:
─────────────
column 'Sort Order' =
RANKX(
    ALL('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Units Under (-)/Over (+)1]),
    't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Units Under (-)/Over (+)1],
    ,
    DESC
)

Purpose      : Sort helper column for [dup unit].
Conversion   : ✅ FULL


### CC-26 | dup unit
─────────────────────────────────────────────────────────────
PBI Column Name  : dup unit
PBI Data Type    : double
PBI lineageTag   : f999ab9b-4391-49b7-91b1-863cf7034108
PBI sortByColumn : Sort Order  (changedProperty: SortByColumn)

Power BI DAX:
─────────────
column 'dup unit' =
't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Units Under (-)/Over (+)1]

Purpose      : Duplicate of Units column sorted by Sort Order DESC.
Depends on   : CC-25 [Sort Order]
Conversion   : ✅ FULL


### CC-27 | Column  (stub)
─────────────────────────────────────────────────────────────
PBI Column Name  : Column
PBI Type         : calculated  (no formula present)
PBI lineageTag   : 5b0e3341-1f8d-44fc-963a-dbb6be47eed2
annotation       : 43dbc3e8-3a1c-4b6f-9923-b49ff7d6691c = True

⚠️  STRUCTURAL DEBT: Empty calculated column stub — no DAX expression.
    Preserved as-is. Validate with developer whether this is intentional.
Conversion   : ⚠️  PRESERVED — stub only


### CC-28 | Column 2  (stub)
─────────────────────────────────────────────────────────────
PBI Column Name  : Column 2
PBI Type         : calculated  (no formula present)
PBI lineageTag   : 02573a3f-7da9-4ee6-95e8-a6c3b360595e
annotation       : 43dbc3e8-3a1c-4b6f-9923-b49ff7d6691c = True

⚠️  STRUCTURAL DEBT: Empty calculated column stub. Preserved as-is.
Conversion   : ⚠️  PRESERVED — stub only


### CC-29 — CC-45 | Action Filter Proxy Columns  (all hidden)
─────────────────────────────────────────────────────────────
Purpose      : Replicate Tableau dashboard cross-filter action behavior.
               Each column is a proxy alias or concatenation — hidden,
               used as cross-visual filter keys.

Pattern A — [Date range] alias:
  All columns below return [Date range] directly (boolean alias).

  Column Name                                                  | lineageTag
  ─────────────────────────────────────────────────────────────────────────
  Action (Date range )                                         | Action (Date range )
  Action (Date range ,Date Early Late)                         | Action (Date range ,Date Early Late)
  Action (Date range ,Date Early/Late)                         | Action (Date range ,Date Early/Late)
  Action (Date range ,Days Early/Late)                         | Action (Date range ,Days Early/Late)
  Action (Date range ,Is Outlier 1,Date Early/Late)            | —
  Action (Date range ,Is Outlier 1,Stat Delivery Date)         | —
  Action (Date range ,Is Outlier 2,Date Early/Late)            | —
  Action (Date range ,Is Outlier 2,Stat Delivery Date)         | —
  Action (Date range ,Stat Delivery Date) 1                    | Action (Date range ,Stat Delivery Date) 1
  Action (Date range ,Stat.Delivery.Dt)                        | —
  Action (Date range ,Units Under (-)/Over (+))                | —
  Action (Date range ,Units Under (-)/Over (+)1)               | —

  DAX (all above):
  = 't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Date range]

Pattern B — [Date range] + [Stat Delivery Date] concat:
  Column Name                             | DAX
  ─────────────────────────────────────────────────────────────
  Action (Date range ,Stat Delivery Date) | [Date range] & " , " & [Stat Delivery Date]
  Action (Date range ,Stat Delivery Date1)| [Date range] & " , " & [Stat Delivery Date]

Pattern C — [Vendor Number] alias:
  Column Name                                                           | lineageTag
  ─────────────────────────────────────────────────────────────────────────────────
  Action (Brand,Material.Type,…,Vendor Number)                          | (long tag)
  Action (Brand,cf_Apply_Filter,…,Vendor Number)                        | (long tag)
  Action (Brand,filter,…,Vendor Number)                                 | (long tag)

  DAX (all above):
  = 't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Vendor Number]

Pattern D — Multi-column concat:
  Column Name                                       | DAX
  ─────────────────────────────────────────────────────────────────────────────
  Action (Material.Type,Material,…,Vendor Number)   | [Material.Type1] & " , " & [Material]
                                                    | & " , " & [Receiving.Plant.Name1]
                                                    | & " , " & [Supplying.Plant.and.Description1]
                                                    | & " , " & [Vendor Number]

  Brand, Fully.Received.Date, Last.Received.Date    | [Brand] & " , " & [Fully.Received.Date]
    and 7 more (Combined)                           | & " , " & [Last.Received.Date]
                                                    | & " , " & [Material Desc]
                                                    | & " , " & [Material]
                                                    | & " , " & [PO.Line.item]
                                                    | & " , " & [Receiving.Plant.Name1]
                                                    | & " , " & [Supplying.Plant.and.Description1]
                                                    | & " , " & [Vendor Number]
                                                    | & " , " & [Include Exclude Flag]

Pattern E — [Stat Delivery Date] alias (hidden, with date variation):
  Action (Stat Delivery Date1)
  = 't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Stat Delivery Date]
  isHidden; variation → LocalDateTable_8b7827f7 'Date Hierarchy'


### CC-46 | Stat Delivery Date_Date Hierarchy
─────────────────────────────────────────────────────────────
PBI Type     : Hierarchy
lineageTag   : 759376ce-600a-4654-a5fb-a2a09146793c
Level 1      : Stat Delivery Date_Month-Trunc  (→ CC-23)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — DAX MEASURES — FULL CATALOG
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All measures in table 't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'
unless noted otherwise.


## 5A | Attribute Single-Value Guards

### M-01 | Attribute_…_Fully.Received.Date
─────────────────────────────────────────────────────────────
PBI lineageTag : [Fully.Received.Date]

Power BI DAX:
─────────────
/// If has one value then return else return *
measure 'Attribute_t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp_Fully.Received.Date' =
IF(
    HASONEVALUE(
        't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Fully.Received.Date]
    ),
    VALUES('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Fully.Received.Date]),
    "*"
)

Conversion   : ✅ FULL


### M-02 | Attribute_…_Last.Received.Date
─────────────────────────────────────────────────────────────
PBI lineageTag : [Last.Received.Date]
DAX: Same HASONEVALUE pattern on [Last.Received.Date]
Conversion   : ✅ FULL


### M-03 | Attribute_…_Stat Delivery Date
─────────────────────────────────────────────────────────────
PBI lineageTag : [Stat Delivery Date]
DAX: Same HASONEVALUE pattern on [Stat Delivery Date]
Conversion   : ✅ FULL


## 5B | PO Count Measures

### M-04 | PO Count
─────────────────────────────────────────────────────────────
PBI lineageTag  : 7d65765a-e689-467b-9292-23b861da67f6
PBI formatString: 0.00

Power BI DAX:
─────────────
measure 'PO Count' =
COUNT('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[PO.Line.Item])

Conversion   : ✅ FULL


### M-05 | PO Distinct Count
─────────────────────────────────────────────────────────────
PBI lineageTag  : e549f1d7-2fdd-4ae1-a90a-d5415a15ced7
PBI formatString: 0

Power BI DAX:
─────────────
measure 'PO Distinct Count' =
DISTINCTCOUNT('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[PO.Line.Item])

Conversion   : ✅ FULL


### M-06 | Total PO
─────────────────────────────────────────────────────────────
PBI lineageTag  : 0676fced-d1ae-436a-b5d4-a5fb30d2d6ed
PBI formatString: 0

Power BI DAX:
─────────────
measure 'Total PO' =
CALCULATE(
    [PO Count],
    ALLSELECTED('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp')
)

Conversion   : ✅ FULL


### M-07 | Total Count
─────────────────────────────────────────────────────────────
PBI lineageTag  : 33f908c1-66d8-4ec4-b89d-2cb4213f5184
PBI formatString: 0

Power BI DAX:
─────────────
measure 'Total Count' =
CALCULATE(
    [PO Count],
    ALLSELECTED(
        't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Units Under (-)/Over (+)1]
    )
)

Conversion   : ✅ FULL


## 5C | Cumulative / Running Measures

### M-08 | Cumulative Count
─────────────────────────────────────────────────────────────
PBI lineageTag  : 5d825b66-120c-418b-80b4-ced8d7ad7b03
PBI formatString: 0

Power BI DAX:
─────────────
measure 'Cumulative Count' =
CALCULATE(
    [PO Count],
    FILTER(
        ALLSELECTED(
            't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Date Early Late]
        ),
        't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Date Early Late]
            <= MAX('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Date Early Late])
    )
)

Conversion   : ✅ FULL


### M-09 | Cumulative %
─────────────────────────────────────────────────────────────
PBI lineageTag  : 7b9cc614-6139-4af6-bfed-dbb9d1a37d4c
PBI formatString: 0.00%;-0.00%;0.00%

Power BI DAX:
─────────────
measure 'Cumulative %' =
DIVIDE([Cumulative Count], [Total PO])

Depends on   : M-08, M-06
Conversion   : ✅ FULL


### M-10 | Running Count
─────────────────────────────────────────────────────────────
PBI lineageTag  : 44d5068c-b7ea-4bda-83bf-eff768e4d5c24
PBI formatString: 0

Power BI DAX:
─────────────
measure 'Running Count' =
VAR CurrentValue =
    MAX('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Units Under (-)/Over (+)1])
RETURN
CALCULATE(
    [PO Distinct Count],
    FILTER(
        ALLSELECTED(
            't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Units Under (-)/Over (+)1]
        ),
        't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Units Under (-)/Over (+)1]
            >= CurrentValue
    )
)

Depends on   : M-05
Conversion   : ✅ FULL


### M-11 | Cumulative % 2
─────────────────────────────────────────────────────────────
PBI lineageTag  : f9cba855-cddd-49ac-b136-f25cb6715acd
PBI formatString: 0.00%;-0.00%;0.00%

Power BI DAX:
─────────────
measure 'Cumulative % 2' =
DIVIDE([Running Count], [Total Count])

Depends on   : M-10, M-07
Conversion   : ✅ FULL


### M-12 | % Running Distinct PO Line
─────────────────────────────────────────────────────────────
PBI lineageTag  : 445962b0-6696-4eed-9346-5f5c0dda4ffe
PBI formatString: 0.00%;-0.00%;0.00%

Power BI DAX:
─────────────
measure '% Running Distinct PO Line' =
VAR CurrentValue =
    MAX('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Date Early Late])
VAR RunningValue =
    CALCULATE(
        DISTINCTCOUNT(
            't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[PO.Line.Item]
        ),
        FILTER(
            ALLSELECTED('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'),
            't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Date Early Late]
                <= CurrentValue
        )
    )
VAR TotalValue =
    CALCULATE(
        DISTINCTCOUNT(
            't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[PO.Line.Item]
        ),
        ALLSELECTED('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp')
    )
RETURN DIVIDE(RunningValue, TotalValue)

Conversion   : ✅ FULL


### M-13 | Running % pg1
─────────────────────────────────────────────────────────────
PBI lineageTag  : 6e92a946-f32e-40b4-9f7e-8fce04fed7b6
PBI formatString: 0.00%;-0.00%;0.00%

Power BI DAX:
─────────────
measure 'Running % pg1' =
VAR CurrentValue =
    MAX('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Units Under (-)/Over (+)1])
VAR DistinctValues =
    CALCULATETABLE(
        VALUES(
            't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Units Under (-)/Over (+)1]
        ),
        ALLSELECTED('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp')
    )
VAR TotalRows   = COUNTROWS(DistinctValues)
VAR CurrentRank =
    RANKX(
        DistinctValues,
        't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Units Under (-)/Over (+)1],
        CurrentValue,
        ASC,
        DENSE
    )
RETURN 1 - DIVIDE(CurrentRank - 1, TotalRows - 1)

Conversion   : ✅ FULL


### M-14 | Running %
─────────────────────────────────────────────────────────────
PBI lineageTag  : 94d4ca15-6bff-41eb-b0c0-7c4b2cd699f1
PBI formatString: 0.00%;-0.00%;0.00%

Power BI DAX:
─────────────
measure 'Running %' =
VAR CurrentValue =
    MAX('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Date Early Late])
VAR RunningTotal =
    CALCULATE(
        COUNT('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[PO.Line.item]),
        FILTER(
            ALLSELECTED('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'),
            /* ✅ Key Fix */
            ISBLANK(
                't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Date Early Late]
            )
            ||
            't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Date Early Late]
                <= CurrentValue
        )
    )
VAR TotalCount =
    CALCULATE(
        COUNT('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[PO.Line.item]),
        ALLSELECTED('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp')
    )
RETURN DIVIDE(RunningTotal, TotalCount)

NOTE: ISBLANK guard in FILTER ensures null [Date Early Late] rows are
      included in running total — comment "✅ Key Fix" preserved from source.
Conversion   : ✅ FULL


## 5D | Statistical Summary Measures  (formatted strings)

### M-15 | Avg of Units Under (-)/Over (+)1
─────────────────────────────────────────────────────────────
PBI lineageTag: bddef3cb-adde-495d-b6d1-babc6a019650
DAX: FORMAT(AVERAGE([Units Under (-)/Over (+)1]), "#,##0.00")
Conversion   : ✅ FULL

### M-16 | Min of Units Under (-)/Over (+)1
─────────────────────────────────────────────────────────────
PBI lineageTag: 4781c6e3-568b-4457-99de-07c2828da397
DAX: FORMAT(MIN([Units Under (-)/Over (+)1]), "#,##0.##")
Conversion   : ✅ FULL

### M-17 | Max of Units Under (-)/Over (+)1
─────────────────────────────────────────────────────────────
PBI lineageTag: fd8eb7a0-e17c-4a74-b8d6-a43f60ac20ad
DAX: FORMAT(MAX([Units Under (-)/Over (+)1]), "#,##0.00")
Conversion   : ✅ FULL

### M-18 | stddev of Units Under (-)/Over (+)1
─────────────────────────────────────────────────────────────
PBI lineageTag: cb01f788-84c0-4190-9e7c-836c776e9244
DAX: FORMAT(STDEV.P([Units Under (-)/Over (+)1]), "#,##0.00")
Conversion   : ✅ FULL

### M-19 | Distinct Count ofPO.Line.Items
─────────────────────────────────────────────────────────────
PBI lineageTag: 507a08c3-6d4b-40d1-ab2b-9602f48d4844
  ⚠️  TYPO ARTIFACT: "of" and "PO" run together — "ofPO" — preserved exactly.
DAX: FORMAT(DISTINCTCOUNT([PO.Line.item]), "#,##0.00")
Conversion   : ✅ FULL (typo preserved)

### M-20 | Avg of Date Early Late
─────────────────────────────────────────────────────────────
PBI lineageTag: 53fb4a08-c524-4961-921d-625648c4b347
DAX: FORMAT(AVERAGE([Date Early Late]), "#,##0.##")
Conversion   : ✅ FULL

### M-21 | Min of Date Early Late
─────────────────────────────────────────────────────────────
PBI lineageTag: f40d6a02-8c79-4ac2-9d7e-b4f70da3b991
DAX: FORMAT(MIN([Date Early Late]), "#,##0.##")
  ⚠️  Uses 'Min' (capital M, lowercase 'in') — preserved exactly.
Conversion   : ✅ FULL

### M-22 | Max of Date Early Late
─────────────────────────────────────────────────────────────
PBI lineageTag: c9d53e08-0e5e-42db-aef5-ef50c97954a9
DAX: FORMAT(MAX([Date Early Late]), "#,##0.##")
Conversion   : ✅ FULL

### M-23 | Std.dev of Date Early Late
─────────────────────────────────────────────────────────────
PBI lineageTag: e43356bd-ca95-49b0-9f01-414a835e17f5
DAX: FORMAT(STDEV.P([Date Early Late]), "#,##0.##")
Conversion   : ✅ FULL


## 5E | SSS Test Measures — Date Early Late Series

### M-24 | 99% SSS test
─────────────────────────────────────────────────────────────
PBI lineageTag: 2376cee6-d5c4-4584-95e6-b8cd3b3f5b95

Power BI DAX:
─────────────
measure '99% SSS test' =
2.33 * STDEV.P('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Date Early Late])
    + AVERAGE('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Date Early Late])

NOTE: Measure-level parallel to CC-03 column — dev/test artifact.
Conversion   : ✅ FULL

### M-25 | 99% SSS_LT test
─────────────────────────────────────────────────────────────
PBI lineageTag: c8c7876b-2334-4137-b56a-0275c4c1f4a2
DAX: IF([99% SSS test] < 0, 0, [99% SSS test])
Conversion   : ✅ FULL

### M-26 | 98% SSS test
─────────────────────────────────────────────────────────────
PBI lineageTag: f3b0e0e2-7f76-4cd8-a861-c5738170f1f5
DAX: 2.05 * STDEV.P([Date Early Late]) + AVERAGE([Date Early Late])
Conversion   : ✅ FULL

### M-27 | 98% SSS_LT test
─────────────────────────────────────────────────────────────
PBI lineageTag: fae8f8ca-fdad-43dd-8d93-1310157bcd20
DAX: IF([98% SSS test] < 0, 0, [98% SSS test])
Conversion   : ✅ FULL

### M-28 | 97% SSS test
─────────────────────────────────────────────────────────────
PBI lineageTag: 20b9dc38-07f6-4f2d-a7fa-083d1c9a5902
DAX: 1.88 * STDEV.P([Date Early Late]) + AVERAGE([Date Early Late])
Conversion   : ✅ FULL

### M-29 | 97% SSS_LT test
─────────────────────────────────────────────────────────────
PBI lineageTag: 4e5eae5f-5617-4b60-bc44-a16565066975
DAX: IF([97% SSS test] < 0, 0, [97% SSS test])
Conversion   : ✅ FULL

### M-30 | 96% SSS test
─────────────────────────────────────────────────────────────
PBI lineageTag: fb92390a-5d36-4e91-9cd6-daa46dee996f
DAX: 1.75 * STDEV.P([Date Early Late]) + AVERAGE([Date Early Late])
Conversion   : ✅ FULL

### M-31 | 96% SSS_LT test
─────────────────────────────────────────────────────────────
PBI lineageTag: b5e9851e-fafe-49ea-a52e-59620472547a
DAX: IF([96% SSS test] < 0, 0, [96% SSS test])
Conversion   : ✅ FULL

### M-32 | 95% SSS test
─────────────────────────────────────────────────────────────
PBI lineageTag: 7df88bd7-87bc-40e5-bb2d-28afdd39370d
DAX: 1.65 * STDEV.P([Date Early Late]) + AVERAGE([Date Early Late])
Conversion   : ✅ FULL

### M-33 | 95% SSS_LT test
─────────────────────────────────────────────────────────────
PBI lineageTag: 8b1bb198-5921-40af-a266-a1df06d60821
DAX: IF([95% SSS test] < 0, 0, [95% SSS test])
Conversion   : ✅ FULL


## 5F | SSS Test Measures — Units Under/Over Series

### M-34 | 99% unit m
─────────────────────────────────────────────────────────────
PBI lineageTag: 683e286c-6f07-4834-9f92-a5e821970c24
  ⚠️  Name uses lowercase 'unit m' — preserved exactly.

Power BI DAX:
─────────────
measure '99% unit m' =
2.33 * STDEV.P('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'
    [Units Under (-)/Over (+)1])
+ AVERAGE('t_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'
    [Units Under (-)/Over (+)1])

Conversion   : ✅ FULL

### M-35 | 99% Unit test
─────────────────────────────────────────────────────────────
PBI lineageTag: 24a50e94-76a8-4750-9ac4-a635bd980c31
DAX: 2.33 * STDEV.P([Units Under (-)/Over (+)1])
         + AVERAGE([Units Under (-)/Over (+)1])
Conversion   : ✅ FULL

### M-36 | 99%SSS_IF test
─────────────────────────────────────────────────────────────
PBI lineageTag: 32030a3c-c432-4362-8802-0d85ec64abfa
PBI formatString: 0.00
DAX: IF([99% Unit test] < 0, 0, [99% Unit test])
Conversion   : ✅ FULL

### M-37 | 98 % Unit testt
─────────────────────────────────────────────────────────────
PBI lineageTag: c9985341-4183-4abe-bddd-87f2dad0a9d4
  ⚠️  TYPO ARTIFACT: "testt" — double-t — preserved exactly.
DAX: 2.05 * STDEV.P([Units Under (-)/Over (+)1])
         + AVERAGE([Units Under (-)/Over (+)1])
Conversion   : ✅ FULL (typo preserved)

### M-38 | 98%SSS_IF test
─────────────────────────────────────────────────────────────
PBI lineageTag: c8dcee3e-3ff2-4c6f-98cc-7672c33254d4
DAX: IF([98 % Unit testt] < 0, 0, [98 % Unit testt])
  ⚠️  References M-37 with double-t typo — preserved exactly.
Conversion   : ✅ FULL

### M-39 | 97 % Unit test
─────────────────────────────────────────────────────────────
PBI lineageTag: fe2d0414-13c4-4299-94c3-374e8bca654c
DAX: 1.88 * STDEV.P([Units Under (-)/Over (+)1])
         + AVERAGE([Units Under (-)/Over (+)1])
Conversion   : ✅ FULL

### M-40 | 97%SSS_IF test
─────────────────────────────────────────────────────────────
PBI lineageTag: 5aebd043-afcb-44f6-894b-5d870b32028a
DAX: IF([97 % Unit test] < 0, 0, [97 % Unit test])
Conversion   : ✅ FULL

### M-41 | 96% Unit test
─────────────────────────────────────────────────────────────
PBI lineageTag: 17dd1ec3-5eb6-4509-9a2c-694fa5cae117
DAX: 1.75 * STDEV.P([Units Under (-)/Over (+)1])
         + AVERAGE([Units Under (-)/Over (+)1])
Conversion   : ✅ FULL

### M-42 | 96%SSS_IF test
─────────────────────────────────────────────────────────────
PBI lineageTag: 51d4f018-c195-4631-a05e-54b10fc1edb0
DAX: IF([96% Unit test] < 0, 0, [96% Unit test])
Conversion   : ✅ FULL

### M-43 | 95% Unit test
─────────────────────────────────────────────────────────────
PBI lineageTag: 8977fa9b-65f2-4109-80c3-7e6866e7f72b
DAX: 1.65 * STDEV.P([Units Under (-)/Over (+)1])
         + AVERAGE([Units Under (-)/Over (+)1])
Conversion   : ✅ FULL

### M-44 | 95%SSS_IF test
─────────────────────────────────────────────────────────────
PBI lineageTag: 0f52c331-2151-4f02-a533-98b0434d3cf5
DAX: IF([95% Unit test] < 0, 0, [95% Unit test])
Conversion   : ✅ FULL


## 5G | Display (FORMAT) Measures

### M-45 | 99%SSS_IF display
─────────────────────────────────────────────────────────────
PBI lineageTag: 9e7414c5-f08b-488b-9ef4-c712bdf8676c
DAX: FORMAT([99%SSS_IF test], "#,##0.##")
Conversion   : ✅ FULL

### M-46 | 98%SSS_IF display
─────────────────────────────────────────────────────────────
PBI lineageTag: db8bb026-2b35-45dc-a034-cb08b48b4ffe
DAX: FORMAT([98%SSS_IF test], "#,##0.00")
Conversion   : ✅ FULL

### M-47 | 97%SSS_IF display
─────────────────────────────────────────────────────────────
PBI lineageTag: a509763c-9b20-4d2a-8b9d-8a2983abbf18
DAX: FORMAT([97%SSS_IF test], "#,##0.##")
Conversion   : ✅ FULL

### M-48 | 96%SSS_IF display
─────────────────────────────────────────────────────────────
PBI lineageTag: 26da8716-160e-432f-a803-bd0ebfc885b0
DAX: FORMAT([96%SSS_IF test], "#,##0.##")
Conversion   : ✅ FULL

### M-49 | 95%SSS_IF display
─────────────────────────────────────────────────────────────
PBI lineageTag: 0860063c-8e14-46d8-916f-f7a6cb340c3b
DAX: FORMAT([95%SSS_IF test], "#,##0.00")
Conversion   : ✅ FULL

### M-50 | 99%SSS_LT display
─────────────────────────────────────────────────────────────
PBI lineageTag: 41db67de-a564-4785-8c1e-e90232d44902
DAX: FORMAT([99% SSS_LT test], "#,##0.##")
Conversion   : ✅ FULL

### M-51 | 98%SSS_LT display
─────────────────────────────────────────────────────────────
PBI lineageTag: 1ae47523-90b9-48f3-b5f8-e4046719d5e2
DAX: FORMAT([98% SSS_LT test], "#,##0.##")
Conversion   : ✅ FULL

### M-52 | 97%SSS_LT display
─────────────────────────────────────────────────────────────
PBI lineageTag: 754b2bfd-ba73-470f-afdb-2b552bfe4595
DAX: FORMAT([97% SSS_LT test], "#,##0.##")
Conversion   : ✅ FULL

### M-53 | 96%SSS_LT display
─────────────────────────────────────────────────────────────
PBI lineageTag: 37cfe13a-cea9-4b33-b012-92f859bcf00b
DAX: FORMAT([96% SSS_LT test], "#,##0.##")
Conversion   : ✅ FULL

### M-54 | 95%SSS_LT display
─────────────────────────────────────────────────────────────
PBI lineageTag: f41aa235-b7e9-4cc4-9e6a-ef770447ba08
DAX: FORMAT([95% SSS_LT test], "#,##0.##")
Conversion   : ✅ FULL


## 5H | Card Label Measures  (static string literals)

| Measure      | Value          | lineageTag                             |
|--------------|----------------|----------------------------------------|
| card 99      | "99%SSS_IF"    | 5a783e3f-d5a2-4049-bd10-21927e59b0e9   |
| card 98      | "98%SSS_IF"    | 527411b4-d340-4ab9-8f8d-233a127963aa   |
| card 97      | "97%SSS_IF"    | 4b89bd5a-67bb-4295-9d74-f938481b0706   |
| card 96      | "96%SSS_IF"    | 56a77eb2-1fb6-4f2f-9e99-6027fa98552a   |
| card 95      | "95%SSS_IF"    | f5be254f-ac57-47eb-ae38-78b378eff9c7   |
| card 99lt    | "99% SSS_LT"   | d64bd27d-31e3-4002-8c46-421f5465263a   |
| card 98lt    | "98% SSS_LT"   | 7b6fa52e-297e-464c-acd0-d8b99e3a11bd   |
| card 97lt    | "97% SSS_LT"   | e305cefb-e8fd-4177-bcea-30c400947407   |
| card 96lt    | "96% SSS_LT"   | 5477413e-88ef-4b21-8c18-896c73f502c9   |
| card 95lt    | "95% SSS_LT"   | 1965b51c-4824-42cc-a20f-e8cf9ea5656e   |
| pg1 Card 1   | "Avg. Units Under(-)/Over(+)1"                     | 9bc9fd24… |
| pg1 Card 2   | "Population std.dev. Of Units Under(-)Over(+)1"    | 25e07c03… |
| pg1 Card 3   | "Min. Units Under(-)/Over(+)1"                     | df42e42f… |
| pg1 Card 4   | "Max. Units Under(-)/Over(+)1"                     | 236dbee4… |
| pg1 Card 5   | "Distict Count of PO.Line.item"  ⚠️ typo: "Distict" | ca9e4ed6… |
| pg2 Card1    | "Avg. Date Early Late"                             | 061119f1… |
| pg2 Card2    | "Population std. dev. of Date Early Late"          | a9381a30… |
| pg2 Card 3   | "Min. Date Early Late"                             | 496c4eb9… |
| pg2 Card 4   | "Max. Date Early Late"                             | c45c1c1c… |

⚠️  "Distict Count" — missing 'n' in "Distinct" — intentional artifact,
    preserved exactly as found in source.


## 5I | Box Plot Measures

### M-55 | Median
─────────────────────────────────────────────────────────────
PBI lineageTag: 7b0f9147-3abf-4076-9e07-facfcf27314a
DAX: MEDIAN([Units Under (-)/Over (+)1])
Conversion   : ✅ FULL

### M-56 | Lower Hinge
─────────────────────────────────────────────────────────────
PBI lineageTag: e171df9f-33d7-4460-ab01-2bacc5beaf12
DAX: PERCENTILE.EXC([Units Under (-)/Over (+)1], 0.25)
Conversion   : ✅ FULL

### M-57 | Upper Hinge
─────────────────────────────────────────────────────────────
PBI lineageTag: ce6126ce-c84d-4df5-9401-205ce484e70a
DAX: PERCENTILE.EXC([Units Under (-)/Over (+)1], 0.75)
Conversion   : ✅ FULL

### M-58 | Lower Whisker
─────────────────────────────────────────────────────────────
PBI lineageTag: f1876b58-c19d-4122-8fd3-bb8ee1921614
DAX: MIN([Units Under (-)/Over (+)1])
Conversion   : ✅ FULL

### M-59 | Upper Whisker
─────────────────────────────────────────────────────────────
PBI lineageTag: 9dc2d626-ef0f-4fce-adf9-33aa1637b2e5
DAX: MAX([Units Under (-)/Over (+)1])
Conversion   : ✅ FULL


## 5J | Misplaced Measure  (in 'End Date (copy 2)' table — structural debt)

### M-60 | 98 % Unit test  (⚠️ wrong table)
─────────────────────────────────────────────────────────────
PBI Table    : End Date (copy 2)   ← misplaced
PBI lineageTag: bb9a5262-ceca-40e6-bf09-4d90155056ce

Power BI DAX:
─────────────
measure '98 % Unit test' =
2.05 * STDEV.P(
    't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Units Under (-)/Over (+)1]
)
+ AVERAGE(
    't_RTP_Final_Yi (gpssc)_t_RTP_Final_Yi_temp'[Units Under (-)/Over (+)1]
)

⚠️  STRUCTURAL DEBT: This measure belongs logically in the fact table.
    It is orphaned in the SSS parameter table. Preserved as-is.
Conversion   : ✅ FULL (placement preserved)


## 5K | Placeholder Measure

### M-61 | dummy
─────────────────────────────────────────────────────────────
PBI lineageTag: 6d95d3d9-192f-4e75-9b0e-a7ddd84a0f2b
DAX: " "   (single space string)
⚠️  STRUCTURAL DEBT: Empty placeholder. Preserved as-is.
Conversion   : ✅ FULL (preserved)


## 5L | SSS Parameter Measure  (in 'SSS Parameter' table)

### M-62 | SSS Parameter Value
─────────────────────────────────────────────────────────────
PBI Table    : SSS Parameter
PBI lineageTag: 76fcd40e-919e-4df9-b2f2-11e368da911b

Power BI DAX:
─────────────
measure 'SSS Parameter Value' =
SELECTEDVALUE('SSS Parameter'[SSS Parameter], 32.8)

Purpose      : Returns user-selected SSS confidence level; defaults to 32.8.
Conversion   : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — RELATIONSHIPS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| ID (short)  | From Table / Column                            | To Table / Column                         | Cardinality  | CrossFilter    | Behavior     | Notes                         |
|-------------|------------------------------------------------|-------------------------------------------|--------------|----------------|--------------|-------------------------------|
| e9315db1    | t_RTP_Final_Yi_temp [Stat Delivery Date]       | LocalDateTable_7699c87b [Date]            | many-to-one  | single         | datePartOnly | Auto date — primary           |
| b62d7133    | t_RTP_Final_Yi_temp [Action (Stat Del. Date1)] | LocalDateTable_8b7827f7 [Date]            | many-to-one  | single         | datePartOnly | Auto date — action col        |
| 96e7a4d8    | t_RTP_Final_Yi_temp [Stat Delivery Date_Month-Trunc] | LocalDateTable_b06e5be8 [Date]      | many-to-one  | single         | datePartOnly | Auto date — month trunc       |
| fdc10baf    | t_RTP_Final_Yi_temp [Day of Stat Delivery Date]| LocalDateTable_41fbab0d [Date]            | many-to-one  | single         | datePartOnly | Auto date — day col           |
| 3a923d62    | t_RTP_Final_Yi_temp [Material]                 | t_rtp_rr_attr [part]                      | many-to-many | bothDirections | —            | Tableau join key; bidirectional ⚠️ |

⚠️  RISK NOTE: The Material ↔ part relationship is many-to-many with
    bothDirections cross-filtering. Validate for DAX fan-out/double-count
    risk in any measure that spans both tables.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — DATE TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### DT-01 | DateTableTemplate_797db397-d0b9-41a2-83b1-f5a6a6ec2823
─────────────────────────────────────────────────────────────
Type         : Template, isHidden, isPrivate
Partition    : Calendar(Date(2015,1,1), Date(2015,1,1))  — stub seed only
Annotation   : __PBI_TemplateDateTable = true
               DefaultItem = DateHierarchy
Hierarchy    : Date Hierarchy → Year → Quarter → Month → Day
Purpose      : PBI internal date intelligence template


### DT-02 | LocalDateTable_7699c87b-9f1c-4de6-8005-a6702283cf22
─────────────────────────────────────────────────────────────
Type         : Auto local date table, isHidden, showAsVariationsOnly
Partition    : Calendar(Date(Year(MIN([Stat Delivery Date])), 1, 1),
                        Date(Year(MAX([Stat Delivery Date])), 12, 31))
Annotation   : __PBI_LocalDateTable = true
Hierarchy    : Date Hierarchy → Year → Quarter → Month → Day
Serves       : [Stat Delivery Date] (relationship e9315db1)

### DT-03 | LocalDateTable_8b7827f7-8cb5-437a-81ab-df70762afc3f
─────────────────────────────────────────────────────────────
Same structure as DT-02.
Serves       : [Action (Stat Delivery Date1)] (relationship b62d7133)

### DT-04 | LocalDateTable_b06e5be8-766d-40a0-a63c-e42e4f95a9c0
─────────────────────────────────────────────────────────────
Same structure as DT-02.
Serves       : [Stat Delivery Date_Month-Trunc] (relationship 96e7a4d8)

### DT-05 | LocalDateTable_41fbab0d-d6a4-4d36-b97e-2a3849a6498d
─────────────────────────────────────────────────────────────
Same structure as DT-02.
Serves       : [Day of Stat Delivery Date] (relationship fdc10baf)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — CONVERSION COMPLETENESS SCORECARD
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Category                       | Tableau Items | PBI Full | PBI Partial | Not Converted | % Complete |
|-------------------------------|---------------|----------|-------------|---------------|------------|
| Workbook & Model Metadata      | 1             | 1        | 0           | 0             | 100%       |
| Data Source Connections        | 2             | 2        | 0           | 0             | 100%       |
| Power Query / M                | 2             | 2        | 0           | 0             | 100%       |
| Parameters / Slicer Tables     | 4             | 4        | 0           | 0             | 100%       |
| Calculated Columns             | 28            | 26       | 0           | 2             | 93%        |
| DAX Measures                   | 62            | 62       | 0           | 0             | 100%       |
| Relationships                  | 5             | 5        | 0           | 0             | 100%       |
| Date Tables                    | 5             | 5        | 0           | 0             | 100%       |
| Report Layer (visuals)         | n/a           | —        | —           | ❌ not in scope | —         |
| TOTAL (semantic model)         | 109           | 107      | 0           | 2             | 🟢 98%     |

NOT CONVERTED (structural debt — stubs, no formula):
  ⚠️  CC-27 | Column   — empty calculated column, no DAX
  ⚠️  CC-28 | Column 2 — empty calculated column, no DAX


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — STRUCTURAL DEBT & AUDIT FLAGS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

DEBT-01 | compatibilityLevel mismatch
  Severity : Low
  Detail   : File declares compatibilityLevel 1604; model comment reads
             "with 1550 compatibility level". Both preserved as-is.
  Action   : No change needed unless upgrading to newer PBID version.

DEBT-02 | Parameter name inversion (PARAM-01, PARAM-02)
  Severity : High
  Detail   : [End Date (copy)] = Start Date slicer.
             [Line Qty Parameter] = End Date slicer.
             Internal names are semantically inverted vs. captions.
  Action   : Do NOT rename — preserved verbatim. Document in report
             developer notes.

DEBT-03 | End Date (copy 2) holds numeric SSS value
  Severity : High
  Detail   : A date-named table holds a fixed numeric value (32.8)
             for SSS confidence parameter. Misleading table name.
  Action   : Preserved as-is. GENERATESERIES slicer (SSS Parameter)
             is the production replacement.

DEBT-04 | 98 % Unit test measure in wrong table
  Severity : Medium
  Detail   : M-60 is defined in 'End Date (copy 2)' but references
             the fact table. Should be in the fact table.
  Action   : Move to fact table in a future cleanup pass.

DEBT-05 | Column / Column 2 empty stubs
  Severity : Low
  Detail   : CC-27 and CC-28 are calculated columns with no formula.
             Annotation flag = True marks them as auto/tool-generated.
  Action   : Validate with developer — delete if unused.

DEBT-06 | dummy measure
  Severity : Low
  Detail   : M-61 returns a single space. Likely a placeholder for
             visual layout purposes.
  Action   : Preserved. Delete if no visual depends on it.

DEBT-07 | 99%SSS_IF case mismatch
  Severity : Medium
  Detail   : CC-18 (99%SSS_IF) references [99% unit] (lowercase 'u')
             in ELSE branch, [99% Unit] (uppercase) in IF branch.
             Column is defined as '99% unit' (lowercase).
  Action   : Preserved as-is. Functional risk is low (PBI column refs
             are case-insensitive at runtime) but audit trail must note.

DEBT-08 | 98 % Unit testt double-t typo
  Severity : Low
  Detail   : M-37 name contains "testt" (double-t). M-38 references it
             by exact name. Both preserved exactly.
  Action   : Preserved. Do not correct — downstream measures reference
             the typo'd name.

DEBT-09 | Distict Count typo in card label
  Severity : Low
  Detail   : M (pg1 Card 5) = "Distict Count of PO.Line.item"
             Missing 'n' in "Distinct". Preserved exactly.
  Action   : Cosmetic only — fix in report layer if desired.

DEBT-10 | Many-to-many bidirectional relationship (Material ↔ part)
  Severity : Medium
  Detail   : Relationship 3a923d62 is many-to-many with bothDirections
             cross-filtering. This replicates Tableau's object-model join
             but may cause fan-out in DAX aggregations.
  Action   : Validate all measures that reference both tables. Consider
             converting to a bridge table if fan-out is confirmed.

DEBT-11 | Datasource date filter as comment only
  Severity : Medium
  Detail   : The global date filter (2025-01-01 to 2026-05-01 on
             Stat Delivery Date_Month-Trunc) is documented as a comment
             block in the M partition but NOT enforced as an active
             Table.SelectRows step.
  Action   : If data volume requires pre-filtering, add:
             #"Filtered Rows" = Table.SelectRows(
                 #"Removed Other Columns",
                 each [Stat Delivery Date] >= #date(2025,1,1)
                   and [Stat Delivery Date] <= #date(2026,5,1)
             )
             Confirm date range with the report owner before applying.

DEBT-12 | Parallel column + measure logic (SSS series)
  Severity : Medium
  Detail   : All 10 SSS confidence levels exist as BOTH calculated columns
             (CC-03 – CC-22) AND "_test" measures (M-24 – M-44). The
             measure versions appear to be dev/test artifacts.
  Action   : Validate which version report visuals consume. Remove unused
             "_test" measures in a cleanup pass.

DEBT-13 | Distinct Count ofPO.Line.Items — missing space
  Severity : Low
  Detail   : Measure name M-19 has "ofPO" with no space. Preserved exactly.
  Action   : Cosmetic — correct in display name only if required.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — CONVERSION PATTERN CHEAT SHEET
## (Reference for future Tableau → PBIP conversions on this workbook)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PATTERN-01 | SQRT(SQUARE(STDEVP(x))) → STDEV.P + ABS()
─────────────────────────────────────────────────────────────
Tableau:
  SQRT(SQUARE(STDEVP([field])))

Power BI:
  STDEV.P([field])
  -- SQRT(SQUARE(x)) = ABS(x); ABS() on a std-dev is always redundant
  -- but applying ABS() preserves the equivalence proof in comments.

WHY: Tableau computed SQRT(SQUARE()) as a workaround for absolute value.
     DAX STDEV.P always returns a non-negative value, so ABS is optional.


PATTERN-02 | SSS confidence bands (z·σ + μ, floored at 0)
─────────────────────────────────────────────────────────────
Tableau (base calc):
  [z] * SQRT(SQUARE(STDEVP([metric]))) + AVG([metric])

Tableau (LT / floored version):
  IF [base_calc] < 0 THEN 0 ELSE [base_calc] END

Power BI (base calculated column):
  VAR StdDevVal = STDEV.P([metric_column])
  VAR MeanVal   = AVERAGE([metric_column])
  RETURN [z] * ABS(StdDevVal) + MeanVal

Power BI (floored column):
  IF([base_column] < 0, 0, [base_column])

Z-scores in use: 99%=2.33, 98%=2.05, 97%=1.88, 96%=1.75, 95%=1.65


PATTERN-03 | Date range boolean from two parameter slicers
─────────────────────────────────────────────────────────────
Tableau:
  [Stat Delivery Date] <= [Parameters].[End Date (param name)]
  AND [Stat Delivery Date] >= [Parameters].[Start Date (param name)]

Power BI:
  IF(
      AND(
          [Stat Delivery Date] <=
              SELECTEDVALUE('EndDateTable'[ParameterTableKeyColumn]),
          [Stat Delivery Date] >=
              SELECTEDVALUE('StartDateTable'[ParameterTableKeyColumn])
      ),
      TRUE(),
      FALSE()
  )

NOTE: Both parameter tables use [ParameterTableKeyColumn] as the value
      column. The slicer selects a single date which SELECTEDVALUE returns.


PATTERN-04 | Tableau action filter key column
─────────────────────────────────────────────────────────────
Tableau:
  Dashboard action: clicking [DimA] + [DimB] filters Sheet2

Power BI:
  Hidden calculated column:
  [Action Key] = [DimA] & " , " & [DimB]
  Use as cross-visual filter or bookmark target.
  Separator is always " , " (space-comma-space) — preserved exactly.


PATTERN-05 | Running cumulative over sorted numeric axis
─────────────────────────────────────────────────────────────
Tableau:
  RUNNING_SUM(COUNT([PO.Line.item])) / TOTAL(COUNT([PO.Line.item]))

Power BI:
  VAR CurrentValue = MAX([axis_column])
  VAR RunningTotal =
      CALCULATE(
          COUNT([measure_column]),
          FILTER(
              ALLSELECTED([Table]),
              ISBLANK([axis_column]) || [axis_column] <= CurrentValue
          )
      )
  VAR TotalCount =
      CALCULATE(COUNT([measure_column]), ALLSELECTED([Table]))
  RETURN DIVIDE(RunningTotal, TotalCount)

NOTE: ISBLANK guard ensures null rows in the axis column are included
      in running total (replicates Tableau's null handling).


PATTERN-06 | Tableau comment preservation in TMDL
─────────────────────────────────────────────────────────────
For every calculated column from Tableau, add at end of DAX:
  /* Original Tableau Expression is <tableau_formula_verbatim> */
For column-level display comments, use TMDL /// prefix:
  /// ColumnCaption@@ - Column
The @@ suffix marks calculated columns in the lineageTag convention
used in this workbook.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 11 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAP-01 | Column / Column 2 stubs have no DAX
  Severity : Low
  Detail   : CC-27 and CC-28 have type=calculated but no expression.
             Power BI will flag these as errors until a formula is added
             or the columns are deleted.
  Fix      : Open in Tabular Editor, either add DAX or delete the columns.
             Confirm with original developer.

GAP-02 | Datasource date filter not enforced in M
  Severity : Medium
  Detail   : Filter 2025-01-01 to 2026-05-01 is commented out.
             All data loads regardless of date unless the M query is updated.
  Fix      : Add Table.SelectRows step in DS-01 M query (see DEBT-11).

GAP-03 | Report layer not migrated
  Severity : Info
  Detail   : Tableau dashboards "On-Time Performance" and "In Full Analysis",
             all worksheets, SSS reference line sheets, and action filters
             are not part of this PBIP semantic model conversion.
  Fix      : Report layer (Power BI .pbir) must be built separately.

GAP-04 | SSS test measures vs. columns — unclear which visuals use
  Severity : Medium
  Detail   : Both CC-03–CC-22 (calculated columns) and M-24–M-44 (measures)
             implement the same SSS logic. It is unclear which the visuals
             consume.
  Fix      : Audit each visual in the report layer and remove unused
             duplicates.

GAP-05 | Many-to-many relationship fan-out risk
  Severity : Medium
  Detail   : Material ↔ part (many-to-many, bothDirections). Any measure
             that uses CALCULATE or implicit filter across both tables
             may double-count.
  Fix      : Test all measures with the relationship. If fan-out is
             confirmed, introduce a bridge/dimension table or switch to
             CROSSFILTER(…, None) on the relationship.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 12 — FILE INVENTORY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| File                                              | Content                                     | Status  |
|---------------------------------------------------|---------------------------------------------|---------|
| 17076_receipt_to_promise__rtp_.txt                | Tableau workbook XML                        | ✅      |
| 12_database.txt                                   | database.tmdl — compatibilityLevel 1604     | ✅      |
| 12_model.txt                                      | model.tmdl — table refs, annotations        | ✅      |
| 12_definition.json                                | definition.pbidataset — schema v4.2         | ✅      |
| 12_.json                                          | diagramLayout.json — actual 5-table layout  | ✅      |
| 12_diagramlayout.json                             | Old stale stub (Calendar/Fact placeholders) | ⚠️ stale |
| 12_relationships.txt                              | relationships.tmdl — 5 relationships        | ✅      |
| 12_expressions.txt                                | expressions.tmdl — SSS expression GUID      | ✅      |
| t_rtp_final_yi__gpssc__t_rtp_final_yi_temp.txt    | Fact table TMDL — all columns/measures/M    | ✅      |
| t_rtp_final_yi__gpssc__t_rtp_rr_attr.txt          | Attribute table TMDL — Redshift M + cols    | ✅      |
| sss_parameter.txt                                 | SSS Parameter table TMDL                    | ✅      |
| end_date__copy_2_.txt                             | End Date (copy 2) table TMDL                | ✅      |
| localdatetable_7699c87b…txt                       | LocalDateTable for Stat Delivery Date       | ✅      |
| localdatetable_8b7827f7…txt                       | LocalDateTable for Action (Stat Del. Date1) | ✅      |
| localdatetable_b06e5be8…txt                       | LocalDateTable for Month-Trunc              | ✅      |
| localdatetable_41fbab0d…txt                       | LocalDateTable for Day of Stat Del. Date    | ✅      |
| datetabletemplate_797db397…txt                    | DateTableTemplate stub                      | ✅      |

END OF KNOWLEDGE FILE
