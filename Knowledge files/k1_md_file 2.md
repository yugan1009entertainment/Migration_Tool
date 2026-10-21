\# K1 — Enterprise Exchange Workbook

\## Tableau → Power BI Conversion Knowledge Base

\*\*Version:\*\* k1 | \*\*Last Updated:\*\* 2026-06-08

\*\*Source Workbook:\*\* Enterprise Exchange (PostgreSQL, Import mode)

\*\*Scope:\*\* Source connectivity, file manifest, TMDL structure, DAX patterns, real column/calc examples from pair\_1



\---



\## 1. OUTPUT FILE MANIFEST



Every conversion produces these files. Prefix with `out\_` during generation; user renames by dropping `out\_` and changing `.txt` → `.tmdl` / `.json`.



| # | Output Filename | Rename To | Folder |

|---|---|---|---|

| 1 | out\_database.txt | database.tmdl | /definition/ |

| 2 | out\_model.txt | model.tmdl | /definition/ |

| 3 | out\_\\<TableName\\>.txt | \\<TableName\\>.tmdl | /definition/tables/ |

| 4 | out\_measures\_table.txt | Measures Table.tmdl | /definition/tables/ |

| 5 | out\_relationships.txt | relationships.tmdl | /definition/ |

| 6 | out\_datetabletemplate.txt | DateTableTemplate\_\\<GUID\\>.tmdl | /definition/tables/ |

| 7 | out\_en\_us.txt | en-US.tmdl | /definition/cultures/ |

| 8 | out\_diagramlayout.json | diagramLayout.json | /definition/ |

| 9+ | out\_localdatetable\_\\<col\\>.txt | LocalDateTable\_\\<GUID\\>.tmdl | /definition/tables/ |



\*\*Conditional:\*\*

\- `out\_parameters.txt` → `Parameters.tmdl` if Tableau parameters exist

\- `out\_roles.txt` → `roles.tmdl` if Tableau RLS exists

\- `out\_theme.json` → `theme.json` if custom color palette exists



\*\*.pbip folder structure:\*\*

```

<ProjectName>.SemanticModel/definition/

├── database.tmdl

├── model.tmdl

├── relationships.tmdl

├── diagramLayout.json

├── tables/

│   ├── <TableName>.tmdl

│   ├── Measures Table.tmdl

│   ├── Parameters.tmdl              (if applicable)

│   └── LocalDateTable\_<GUID>.tmdl   (one per date column)

├── cultures/en-US.tmdl

└── roles/<RoleName>.tmdl            (if applicable)

```



\---



\## 2. SOURCE: POSTGRESQL (ENTERPRISE EXCHANGE)



\*\*Tableau connection:\*\*

```xml

<connection class='postgres'

&#x20; server='primary-passcdraprod-rds-db-instance-7ijwssk.cvegddsdpypp.us-east-1.rds.amazonaws.com:5432'

&#x20; dbname='passcdraprod' />

```



\*\*Power BI M partition (actual from pair\_1):\*\*

```m

let

&#x20;   Source = PostgreSQL.Database(

&#x20;       "primary-passcdraprod-rds-db-instance-7ijwssk.cvegddsdpypp.us-east-1.rds.amazonaws.com:5432",

&#x20;       "passcdraprod"

&#x20;   ),

&#x20;   Query =

&#x20;       "with r as (

&#x20;   select first\_name, last\_name, email, district, district\_name, territory\_code,

&#x20;          territory\_name, role\_code

&#x20;   from cdra\_stage.align1\_dal\_roster\_pass

&#x20;   where sales\_force\_id in ('009', '109')

&#x20;   order by sales\_force\_id desc, territory\_code, role\_code desc

),

d as (

&#x20;   select \* from r

&#x20;   where right(r.territory\_code, 4) = '0000'

&#x20;     and r.role\_code = 'PRIM'

)

SELECT

&#x20;   ees.status, submission\_date, submitter,

&#x20;   ees.territory\_name AS territory\_name\_xl,

&#x20;   therapeutic\_area, category, topic, description,

&#x20;   adro\_response, adro\_comment, ees.region, ees.director,

&#x20;   roster\_idx, idx, r.\*,

&#x20;   d.first\_name || ' ' || d.last\_name AS director\_roster

FROM ""cdra\_stg\_pass"".""enterprise\_exchange\_submissions"" ees

LEFT OUTER JOIN r ON r.email = ees.submitter AND r.role\_code = 'PRIM'

LEFT OUTER JOIN d ON d.territory\_code = left(r.territory\_code, 4) || '0000'

&#x20;   AND d.role\_code = 'PRIM'",

&#x20;   Result = Value.NativeQuery(Source, Query, \[], \[EnableFolding=true]),

&#x20;   #"Filtered Rows" = Table.SelectRows(Result, each true),

&#x20;   #"Replaced Value"  = Table.ReplaceValue(#"Filtered Rows","","Null",Replacer.ReplaceValue,{"region"}),

&#x20;   #"Replaced Value1" = Table.ReplaceValue(#"Replaced Value",null,"Null",Replacer.ReplaceValue,{"region"})

in

&#x20;   #"Replaced Value1"

```



\*\*TMDL partition block:\*\*

```tmdl

partition 402bc123-6bbc-426d-a942-ac7c92535218\_Partition = m

&#x20;   mode: import

&#x20;   source = ```

&#x20;           let

&#x20;               Source = PostgreSQL.Database("server:5432", "dbname"),

&#x20;               Query  = "<SQL>",

&#x20;               Result = Value.NativeQuery(Source, Query, \[], \[EnableFolding=true]),

&#x20;               #"Filtered Rows" = Table.SelectRows(Result, each true)

&#x20;           in

&#x20;               #"Filtered Rows"

&#x20;           ```

&#x20;   annotation PBI\_NavigationStepName = Navigation

&#x20;   annotation PBI\_ResultType = Table

```



\---



\## 3. TMDL FILE TEMPLATES



\### 3.1 database.tmdl

```tmdl

database

&#x20;   compatibilityLevel: 1604

```



\### 3.2 model.tmdl

```tmdl

model Model

&#x20;   culture: en-US

&#x20;   defaultPowerBIDataSourceVersion: powerBI\_V3

&#x20;   sourceQueryCulture: en-US

&#x20;   dataAccessOptions

&#x20;       legacyRedirects

&#x20;       returnErrorValuesAsNull



&#x20;   annotation \_\_TEdtr = 1

&#x20;   annotation \_\_PBI\_TimeIntelligenceEnabled = 1

&#x20;   annotation PBI\_QueryOrder = \["Measures Table","Custom SQL Query"]

&#x20;   annotation PBI\_ProTooling = \["DevMode"]



&#x20;   ref table 'Measures Table'

&#x20;   ref table 'Custom SQL Query'

&#x20;   ref table DateTableTemplate\_<GUID>

&#x20;   ref table LocalDateTable\_<GUID>

&#x20;   ref cultureInfo en-US

```



\### 3.3 LocalDateTable.tmdl (one per date column)

```tmdl

table LocalDateTable\_<GUID>

&#x20;   isHidden

&#x20;   showAsVariationsOnly

&#x20;   lineageTag: <GUID>



&#x20;   column Date

&#x20;       dataType: dateTime

&#x20;       isHidden

&#x20;       formatString: General Date

&#x20;       lineageTag: ldt<n>-date

&#x20;       dataCategory: PaddedDateTableDates

&#x20;       summarizeBy: none

&#x20;       isNameInferred

&#x20;       sourceColumn: \[Date]

&#x20;   column Year = YEAR(\[Date])

&#x20;       dataType: int64

&#x20;       isHidden

&#x20;       formatString: 0

&#x20;       lineageTag: ldt<n>-year

&#x20;       dataCategory: Years

&#x20;       summarizeBy: none

&#x20;   column MonthNo = MONTH(\[Date])

&#x20;       dataType: int64

&#x20;       isHidden

&#x20;       formatString: 0

&#x20;       lineageTag: ldt<n>-monthno

&#x20;       dataCategory: MonthOfYear

&#x20;       summarizeBy: none

&#x20;   column Month = FORMAT(\[Date], "MMMM")

&#x20;       dataType: string

&#x20;       isHidden

&#x20;       lineageTag: ldt<n>-month

&#x20;       dataCategory: Months

&#x20;       summarizeBy: none

&#x20;       sortByColumn: MonthNo

&#x20;   column QuarterNo = INT((\[MonthNo] + 2) / 3)

&#x20;       dataType: int64

&#x20;       isHidden

&#x20;       formatString: 0

&#x20;       lineageTag: ldt<n>-qno

&#x20;       dataCategory: QuarterOfYear

&#x20;       summarizeBy: none

&#x20;   column Quarter = "Qtr " \& \[QuarterNo]

&#x20;       dataType: string

&#x20;       isHidden

&#x20;       lineageTag: ldt<n>-q

&#x20;       dataCategory: Quarters

&#x20;       summarizeBy: none

&#x20;       sortByColumn: QuarterNo

&#x20;   column Day = DAY(\[Date])

&#x20;       dataType: int64

&#x20;       isHidden

&#x20;       formatString: 0

&#x20;       lineageTag: ldt<n>-day

&#x20;       dataCategory: DayOfMonth

&#x20;       summarizeBy: none



&#x20;   hierarchy 'Date Hierarchy'

&#x20;       lineageTag: ldt<n>-hier

&#x20;       level Year

&#x20;           lineageTag: ldt<n>-hier-y

&#x20;           column: Year

&#x20;       level Quarter

&#x20;           lineageTag: ldt<n>-hier-q

&#x20;           column: Quarter

&#x20;       level Month

&#x20;           lineageTag: ldt<n>-hier-m

&#x20;           column: Month

&#x20;       level Day

&#x20;           lineageTag: ldt<n>-hier-d

&#x20;           column: Day



&#x20;   partition LocalDateTable\_<GUID> = calculated

&#x20;       mode: import

&#x20;       source = Calendar(Date(Year(MIN('TableName'\[DateCol])),1,1), Date(Year(MAX('TableName'\[DateCol])),12,31))



&#x20;   annotation \_\_PBI\_LocalDateTable = true

```



\*\*GUID assignment:\*\* Date col 1 → suffix `1111-1111-1111-111111111111`, prefix `ldt1`; col 2 → `2222...`, `ldt2`; etc.



\### 3.4 Measures Table.tmdl

```tmdl

table 'Measures Table'

&#x20;   lineageTag: mt-0000-0000-0000-000000000001



&#x20;   measure 'Total Submissions' = COUNT('Custom SQL Query'\[Status])

&#x20;       formatString: #,##0

&#x20;       lineageTag: mt-meas-total-submissions



&#x20;   partition 'Measures Table' = m

&#x20;       mode: import

&#x20;       source = ```

&#x20;               let

&#x20;                   Source = Table.FromRows(

&#x20;                       Json.Document(Binary.Decompress(

&#x20;                           Binary.FromText("i44FAA==", BinaryEncoding.Base64),

&#x20;                           Compression.Deflate)),

&#x20;                       let \_t = ((type nullable text) meta \[Serialized.Text = true])

&#x20;                       in type table \[Column1 = \_t]),

&#x20;                   #"Removed Columns" = Table.RemoveColumns(Source, {"Column1"})

&#x20;               in

&#x20;                   #"Removed Columns"

&#x20;               ```

&#x20;       annotation PBI\_NavigationStepName = Navigation

&#x20;       annotation PBI\_ResultType = Table

```



\### 3.5 relationships.tmdl (pair\_1 actual)

```tmdl

// Week Start date variation

relationship 3fdd7d34-51e1-40ff-b66e-7a36643a4824

&#x20;   joinOnDateBehavior: datePartOnly

&#x20;   fromColumn: 'Custom SQL Query'.'Week Start'

&#x20;   toColumn: LocalDateTable\_37266a76-90b5-4078-942d-75a59988e0d3.Date



// Submission Date date variation

relationship bbe4a354-0f5e-4c9f-8381-bd4fafcd17df

&#x20;   joinOnDateBehavior: datePartOnly

&#x20;   fromColumn: 'Custom SQL Query'.'Submission Date'

&#x20;   toColumn: LocalDateTable\_572388b4-2112-40ad-9630-af36b8afc93e.Date



// Submission Week date variation

relationship 141f5ec5-fd3c-4a31-96a5-7768a11081d5

&#x20;   joinOnDateBehavior: datePartOnly

&#x20;   fromColumn: 'Custom SQL Query'.'Submission Week'

&#x20;   toColumn: LocalDateTable\_2e8b88fa-14b2-4993-9f15-905d8ee79573.Date

```



\### 3.6 en-US.tmdl

```tmdl

cultureInfo en-US

&#x20;   linguisticMetadata = ```

&#x20;           {

&#x20;             "Version": "1.0.0",

&#x20;             "Language": "en-US"

&#x20;           }

&#x20;           ```

&#x20;       annotation TopLevelSchema = True

```



\### 3.7 diagramLayout.json

```json

{

&#x20; "version": "1.1",

&#x20; "tables": \[

&#x20;   { "id": "Custom SQL Query", "name": "Custom SQL Query",

&#x20;     "x": 0, "y": 0, "width": 260, "height": 400 }

&#x20; ],

&#x20; "relationships": \[]

}

```



\---



\## 4. REAL COLUMN EXAMPLES FROM PAIR\_1 (Enterprise Exchange)



\### 4.1 Source columns (hidden, standard)

```tmdl

column Status

&#x20;   dataType: string

&#x20;   displayOrdinal: 1

&#x20;   lineageTag: csq-col-status

&#x20;   summarizeBy: none

&#x20;   sourceColumn: status

&#x20;   changedProperty = IsHidden



column 'Submission Date'

&#x20;   dataType: dateTime

&#x20;   formatString: dd-mmm-yy

&#x20;   displayOrdinal: 2

&#x20;   lineageTag: csq-col-submission-date

&#x20;   summarizeBy: none

&#x20;   sourceColumn: submission\_date

&#x20;   variation Variation

&#x20;       isDefault

&#x20;       relationship: bbe4a354-0f5e-4c9f-8381-bd4fafcd17df

&#x20;       defaultHierarchy: LocalDateTable\_572388b4-2112-40ad-9630-af36b8afc93e.'Date Hierarchy'

&#x20;   changedProperty = IsHidden

&#x20;   changedProperty = DataType

&#x20;   annotation SummarizationSetBy = Automatic

&#x20;   annotation UnderlyingDateTimeDataType = Date

&#x20;   annotation PBI\_FormatHint = {"isDateTimeCustom":true}

```



\### 4.2 Calculated columns (pair\_1 actual, lineageTags corrected per M11)

```tmdl

// Territory display name (concatenate name + form override)

column Territory = 'Custom SQL Query'\[territory\_name] \& IF(

&#x20;   TRIM(LOWER('Custom SQL Query'\[Territory (form)])) = TRIM(LOWER('Custom SQL Query'\[territory\_name])),

&#x20;   "",

&#x20;   "(" \& 'Custom SQL Query'\[Territory (form)] \& ")")

&#x20;   dataType: string

&#x20;   lineageTag: csq-calc-territory

&#x20;   summarizeBy: none

&#x20;   changedProperty = IsHidden



// ARM identifier string

column '#ARMs' = 'Custom SQL Query'\[Submitter] \& UNICHAR(13) \& 'Custom SQL Query'\[first\_name] \& " " \& 'Custom SQL Query'\[last\_name]

&#x20;   dataType: string

&#x20;   lineageTag: csq-calc-arms

&#x20;   summarizeBy: none

&#x20;   changedProperty = IsHidden



// Submission Week (Monday-start ISO week)

column 'Submission Week' = 'Custom SQL Query'\[Submission Date] - WEEKDAY('Custom SQL Query'\[Submission Date], 2) + 1

&#x20;   dataType: dateTime

&#x20;   formatString: Short Date

&#x20;   lineageTag: csq-calc-submission-week

&#x20;   summarizeBy: none

&#x20;   variation Variation

&#x20;       isDefault

&#x20;       relationship: 141f5ec5-fd3c-4a31-96a5-7768a11081d5

&#x20;       defaultHierarchy: LocalDateTable\_2e8b88fa-14b2-4993-9f15-905d8ee79573.'Date Hierarchy'

&#x20;   changedProperty = IsHidden

&#x20;   changedProperty = DataType

&#x20;   annotation SummarizationSetBy = Automatic

&#x20;   annotation UnderlyingDateTimeDataType = Date



// Action columns (Tableau action filter emulation — always isHidden, lineageTag kebab per M11)

column 'Action (Category,Submission Date)' = 'Custom SQL Query'\[Category] \& " , " \& 'Custom SQL Query'\[Submission Date]

&#x20;   dataType: string

&#x20;   isHidden

&#x20;   lineageTag: csq-action-category-submission-date

&#x20;   summarizeBy: none

&#x20;   annotation SummarizationSetBy = Automatic



column 'Action (Category,Topic)' = 'Custom SQL Query'\[Category] \& " , " \& 'Custom SQL Query'\[Topic]

&#x20;   dataType: string

&#x20;   isHidden

&#x20;   lineageTag: csq-action-category-topic

&#x20;   summarizeBy: none

&#x20;   annotation SummarizationSetBy = Automatic



column 'Action (Submission Week,Category)' = 'Custom SQL Query'\[Submission Week] \& " , " \& 'Custom SQL Query'\[Category]

&#x20;   dataType: string

&#x20;   isHidden

&#x20;   lineageTag: csq-action-submission-week-category

&#x20;   summarizeBy: none

&#x20;   annotation SummarizationSetBy = Automatic



// Week Start (same calc as Submission Week, different format)

column 'Week Start' = 'Custom SQL Query'\[Submission Date] - WEEKDAY('Custom SQL Query'\[Submission Date], 2) + 1

&#x20;   formatString: dd-mmm-yy

&#x20;   lineageTag: csq-calc-week-start

&#x20;   summarizeBy: none

&#x20;   variation Variation

&#x20;       isDefault

&#x20;       relationship: 3fdd7d34-51e1-40ff-b66e-7a36643a4824

&#x20;       defaultHierarchy: LocalDateTable\_37266a76-90b5-4078-942d-75a59988e0d3.'Date Hierarchy'

&#x20;   annotation SummarizationSetBy = Automatic

&#x20;   annotation PBI\_FormatHint = {"isDateTimeCustom":true}



// COALESCE null replacement

column Topic2 = COALESCE('Custom SQL Query'\[Topic], "Null")

&#x20;   lineageTag: csq-calc-topic2

&#x20;   summarizeBy: none

&#x20;   annotation SummarizationSetBy = Automatic



// Date display string (sorted by yyyymmdd column)

column 'Submission Week dmy' = FORMAT('Custom SQL Query'\[Submission Week], "dd-MMM-yyyy")

&#x20;   lineageTag: csq-calc-submission-week-dmy

&#x20;   summarizeBy: none

&#x20;   sortByColumn: 'Submission Week dmy sort'

&#x20;   changedProperty = SortByColumn

&#x20;   annotation SummarizationSetBy = Automatic



column 'Submission Week dmy sort' = FORMAT('Custom SQL Query'\[Submission Week], "yyyymmdd")

&#x20;   lineageTag: csq-calc-submission-week-dmy-sort

&#x20;   summarizeBy: none

&#x20;   annotation SummarizationSetBy = Automatic

```



\---



\## 5. DAX TRANSLATION PATTERNS



\### 5.1 Conditional Logic

| Tableau | DAX |

|---|---|

| `IF \[x]=1 THEN "A" ELSEIF \[x]=2 THEN "B" ELSE "C" END` | `SWITCH(\[x], 1, "A", 2, "B", "C")` |

| `IF \[x]>10 THEN "Hi" ELSEIF \[x]>5 THEN "Mid" ELSE "Lo" END` | `SWITCH(TRUE(), \[x]>10, "Hi", \[x]>5, "Mid", "Lo")` |

| `ISNULL(\[x])` | `ISBLANK(T\[x])` |

| `ZN(\[x])` | `IF(ISBLANK(T\[x]), 0, T\[x])` |

| `COALESCE(\[x], "Null")` | `COALESCE(T\[x], "Null")` |



\### 5.2 LOD Expressions

| Tableau LOD | DAX |

|---|---|

| `{FIXED \[A],\[B]: MAX(\[Val])}` | `CALCULATE(MAX(T\[Val]), ALLEXCEPT(T, T\[A], T\[B]))` |

| `{FIXED \[A]: MAX(\[Val])}` as measure | `SUMX(SUMMARIZE(T,T\[A]), CALCULATE(MAX(T\[Val])))` |

| `{EXCLUDE \[Cat]: SUM(\[Sales])}` | `CALCULATE(SUM(T\[Sales]), REMOVEFILTERS(T\[Cat]))` |

| `{INCLUDE \[Sub]: SUM(\[Sales])}` | `SUMX(VALUES(T\[Sub]), CALCULATE(SUM(T\[Sales])))` |



\### 5.3 Date Functions

| Tableau | DAX |

|---|---|

| `DATEPART('year', \[Date])` | `YEAR(T\[Date])` |

| `DATEPART('month', \[Date])` | `MONTH(T\[Date])` |

| `DATEPART('iso-weekday', \[Date])` | `WEEKDAY(T\[Date], 2)` |

| `DATETRUNC('week', \[Date], 'monday')` | `T\[Date] - WEEKDAY(T\[Date], 2) + 1` |

| `DATETRUNC('month', \[Date])` | `DATE(YEAR(T\[Date]), MONTH(T\[Date]), 1)` |

| `DATEDIFF('day', \[S], \[E])` | `DATEDIFF(T\[S], T\[E], DAY)` |

| `DATEADD('month', -1, TODAY())` | `EDATE(TODAY(), -1)` |



\### 5.4 String Functions

| Tableau | DAX |

|---|---|

| `FIND(\[S], "sub")` | `SEARCH("sub", T\[S], 1, 0)` |

| `LEFT(\[S], n)` | `LEFT(T\[S], n)` |

| `CONTAINS(\[F], "v")` | `SEARCH("v", T\[F], 1, 0) > 0` |

| `STR(\[N])` | `FORMAT(T\[N], "0")` |

| `TRIM(\[S])` | `TRIM(T\[S])` |

| `LOWER(\[S])` | `LOWER(T\[S])` |



\### 5.5 Aggregations (Measures)

| Tableau | DAX Measure |

|---|---|

| `SUM(\[Sales])` | `SUM(T\[Sales])` |

| `COUNT(\[ID])` | `COUNT(T\[ID])` |

| `COUNTD(\[ID])` | `DISTINCTCOUNT(T\[ID])` |

| `AVG(\[Sales])` | `AVERAGE(T\[Sales])` |



\### 5.6 Tableau Action Filter → Power BI Action Column Pattern

Tableau action filters that pass multiple fields are emulated as a concatenated string column:

```tmdl

column 'Action (Field1,Field2)' = T\[Field1] \& " , " \& T\[Field2]

&#x20;   dataType: string

&#x20;   isHidden

&#x20;   lineageTag: tbl-action-field1-field2   // kebab-case only — Rule M11

&#x20;   summarizeBy: none

&#x20;   annotation SummarizationSetBy = Automatic

```



\---



\## 6. TABLEAU FEATURES → POWER BI MAPPING



| Tableau Feature | Power BI Equivalent | Handled In |

|---|---|---|

| Dimension Alias | `SWITCH` calculated column | TMDL |

| Categorical Bin | `SWITCH(TRUE(), col IN {...}, ...)` | TMDL |

| LOD {FIXED} | `CALCULATE` + `ALLEXCEPT` | DAX |

| Parameters | Disconnected table + SELECTEDVALUE measure | Parameters.tmdl |

| Action Filter | Concatenated string column (isHidden) | TMDL |

| Custom Color Palette | `theme.json` `dataColors` array | theme.json |

| RLS | `role` + `tablePermission` TMDL | roles.tmdl |

| Drill Path | `hierarchy` block | TMDL |

| Tooltip Sheet | Tooltip report page | Report layer only |

| Dashboard | Report page | Report layer only |



\---



\## 7. LINEAGETAG NAMING CONVENTIONS



| Object | Pattern | Example |

|---|---|---|

| Source column | `<prefix>-col-<name>` | `csq-col-status` |

| Calculated column | `<prefix>-calc-<name>` | `csq-calc-territory` |

| Measure | `<prefix>-meas-<name>` | `mt-meas-total-submissions` |

| Action column | `<prefix>-action-<slug>` | `csq-action-category-topic` |

| Hierarchy | `<prefix>-hier-<name>` | `csq-hier-territory` |

| Hierarchy level n | `<prefix>-hier-lvl-<n>` | `csq-hier-lvl-1` |

| Table | `<prefix>-0001-0000-0000-<12digits>` | `csq-0001-0000-0000-000000000001` |



\*\*Table prefix codes:\*\* `csq` = Custom SQL Query, `mt` = Measures Table, `param` = Parameters, `ldt<n>` = LocalDateTable n



\---



\## 8. CONVERSION REPORT TEMPLATE



```

\# Pair\_N Conversion Report — \[Workbook Name]

\## Source: \[type + server + db + mode]

\## Tables Emitted: \[list with col counts]

\## Relationships: \[list]

\## Calculated Columns Translated: \[Tableau → DAX pattern]

\## Parameters: \[describe mapping]

\## Manual Review Flags: \[list]

\## Files Emitted (\[N] total): \[numbered list]

```



