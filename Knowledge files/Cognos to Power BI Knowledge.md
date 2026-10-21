# Cognos to Power BI Knowledge

## Purpose

This knowledge file is designed for an **IBM Cognos Analytics →
Microsoft Power BI / PBIP migration accelerator**.

The core principle is:

> **Do not migrate Cognos syntax blindly. Extract Cognos semantics,
> normalize them into an intermediate representation, then generate the
> appropriate Power BI semantic model and report artifacts.**

It covers Cognos architecture, Framework Manager, packages, data
modules, report specifications/XML, queries, expressions, calculations,
filters, prompts, parameters, crosstabs, lists, charts, drill-through,
bursting, scheduling, security, OLAP concepts, DAX/Power Query mapping,
Power BI semantic modeling, PBIP/PBIR/TMDL generation, paginated
reports, validation, performance, automation and public migration
experience.

------------------------------------------------------------------------

# 1. Cognos architecture

IBM Cognos commonly separates:

1.  Physical data sources
2.  Metadata/business modeling
3.  Packages or data modules
4.  Report specifications
5.  Queries and expressions
6.  Report presentation
7.  Prompts and runtime behavior
8.  Security
9.  Scheduling and distribution

Power BI generally separates:

1.  Data acquisition/transformation through Power Query
2.  Tabular semantic model
3.  Measures/calculated columns
4.  Report pages and visuals
5.  Filters/slicers/interactions
6.  Security through semantic-model roles/RLS
7.  Service deployment and distribution

A Cognos migration should therefore be treated as at least three related
workstreams:

``` text
Cognos semantic/model
        ↓
Power BI semantic model

Cognos report
        ↓
Power BI interactive report or paginated report

Cognos operational/distribution behavior
        ↓
Power BI Service / subscriptions / workflows / paginated reports
```

------------------------------------------------------------------------

# 2. Important Cognos artifacts

## 2.1 Report specification

A Cognos report is represented by a **report specification**.

IBM describes the report specification as an XML document defining the
structure and layout of a report. It contains queries and layouts and
can be viewed and manipulated programmatically.

High-value XML elements/concepts include:

-   report
-   queries
-   query
-   selection
-   dataItem
-   filter
-   detailFilter
-   summaryFilter
-   sorting
-   layouts
-   pages
-   blocks
-   lists
-   crosstabs
-   charts
-   text items
-   prompt controls
-   variables
-   drill-through definitions
-   conditional styles

Official references:

https://www.ibm.com/docs/en/cognos-analytics/12.1.x?topic=developer-using-report-specifications

https://www.ibm.com/docs/en/cognos-analytics/12.1.0?topic=specifications-basic-structure-report-specification

https://www.ibm.com/docs/en/cognos-analytics/12.0.x?topic=reporting-creating-report

------------------------------------------------------------------------

## 2.2 Framework Manager

Framework Manager is Cognos's metadata modeling tool. IBM describes a
model as metadata containing physical and business information from one
or more data sources.

Important objects:

-   project
-   namespace
-   query subject
-   query item
-   relationship
-   cardinality
-   calculation
-   filter
-   parameter
-   determinant
-   shortcut
-   alias
-   package
-   dimension
-   hierarchy
-   level
-   security

Framework Manager can be understood as:

``` text
Physical database
       ↓
Database layer
       ↓
Logical/business layer
       ↓
Dimensional layer
       ↓
Package
       ↓
Cognos reports
```

Official references:

https://www.ibm.com/docs/en/cognos-analytics/12.1.x?topic=manager-getting-started-framework

https://www.ibm.com/docs/en/pm-and-q/2.6.3?topic=product-cognos-framework-manager-model-description

------------------------------------------------------------------------

## 2.3 Packages

A package exposes a subset of a Cognos model to reporting.

A package can define:

-   query subjects
-   query items
-   relationships
-   calculations
-   filters
-   dimensions
-   hierarchies
-   security
-   drill-through behavior

For migration lineage, capture:

``` text
Report
  ↓
Package
  ↓
Model object
  ↓
Physical source
```

Do not assume that a package should become one Power BI semantic model.
First identify common reusable model logic.

------------------------------------------------------------------------

## 2.4 Data modules

Cognos Analytics also supports data modules.

A migration extractor should identify whether each report is based on:

-   Framework Manager package
-   Data module
-   relational source
-   dimensional/OLAP source
-   uploaded data
-   other Cognos-supported source

The conversion path can differ significantly.

------------------------------------------------------------------------

# 3. Cognos → Power BI conceptual mapping

  -------------------------------------------------------------------------------------
  Cognos                  Power BI target                       Notes
  ----------------------- ------------------------------------- -----------------------
  Physical data source    Power Query/source                    Connection and lineage

  Framework Manager model Semantic model                        Strong conceptual
                                                                mapping

  Query subject           Table/query/model object              Depends on role

  Query item              Column or measure                     Determine grain

  Relationship            Model relationship                    Validate cardinality

  Package                 Shared semantic model/report exposure Not 1:1

  Report specification    PBIR/report definition                Parse and regenerate

  List                    Table visual or paginated tablix      Depends on usage

  Crosstab                Matrix or paginated tablix            Often needs redesign

  Chart                   Power BI visual                       Map fields and
                                                                properties

  Prompt                  Slicer/filter/parameter               Context dependent

  Parameter               Slicer/field parameter/Power Query    Context dependent
                          parameter                             

  Calculation             DAX/M/source SQL                      Depends on grain

  Filter                  Visual/page/report/model filter       Scope is critical

  Drill-through           Drillthrough page                     Strong mapping

  Master/detail           Drillthrough/navigation               Often redesigned

  Conditional formatting  Power BI conditional formatting       Usually possible

  Variables               DAX/bookmarks/parameters/formatting   Context dependent

  Section/data security   RLS/OLS/Entra groups                  Explicit mapping
                                                                required

  Burst                   Subscription/workflow/paginated       No universal 1:1
                          report                                mapping

  Schedule                Refresh/subscription/workflow         Separate data/report
                                                                scheduling

  OLAP cube               Tabular semantic model                May require redesign
  -------------------------------------------------------------------------------------

------------------------------------------------------------------------

# 4. Migration principle: semantics over syntax

A Cognos expression such as:

``` text
sum([Revenue])
```

is not simply a string to replace.

The migration engine should determine:

-   source object
-   aggregation
-   grain
-   filter context
-   scope
-   lineage
-   whether it is reusable
-   whether it is a measure or row calculation

Then produce:

``` dax
Total Revenue =
SUM ( Sales[Revenue] )
```

For more complex expressions, use an AST/intermediate representation
rather than string replacement.

------------------------------------------------------------------------

# 5. Recommended migration phases

## Phase 0 --- Discovery

Inventory:

-   Cognos version
-   Framework Manager projects
-   packages
-   data modules
-   reports
-   dashboards
-   schedules
-   jobs
-   bursting
-   security
-   namespaces
-   data sources
-   cubes/OLAP
-   report owners
-   consumers

## Phase 1 --- Report inventory

Capture for every report:

-   name
-   path
-   package
-   owner
-   last execution
-   execution frequency
-   page count
-   query count
-   data-item count
-   calculation count
-   filter count
-   prompt count
-   visual count
-   crosstab count
-   drill-through count
-   nested report/subreport count
-   bursting
-   scheduling
-   security
-   data sources
-   business criticality

## Phase 2 --- Rationalization

Classify:

``` text
Migrate
Consolidate
Redesign
Retire
Archive
```

Do not migrate obsolete/duplicate reports simply because they exist.

## Phase 3 --- Semantic model

Build:

-   tables
-   columns
-   measures
-   relationships
-   hierarchies
-   security
-   source lineage

## Phase 4 --- Report

Build:

-   pages
-   visuals
-   slicers
-   filters
-   drillthrough
-   navigation
-   formatting

## Phase 5 --- Validation

Validate:

-   metadata
-   numbers
-   filters
-   prompts
-   security
-   visuals
-   behavior

## Phase 6 --- Deployment

Deploy to test, perform UAT, then production.

Microsoft's migration guidance emphasizes requirements gathering,
planning, proof of concept, content creation/validation, and
deployment/support.

References:

https://learn.microsoft.com/en-us/power-bi/guidance/powerbi-migration-pre-migration-steps

https://learn.microsoft.com/en-us/power-bi/guidance/powerbi-migration-requirements

https://learn.microsoft.com/en-us/power-bi/guidance/powerbi-migration-planning

https://learn.microsoft.com/en-us/power-bi/guidance/powerbi-migration-proof-of-concept

https://learn.microsoft.com/en-us/power-bi/guidance/powerbi-migration-create-validate-content

https://learn.microsoft.com/en-us/power-bi/guidance/powerbi-migration-deploy-support-monitor

------------------------------------------------------------------------

# 6. Extraction strategy

Use four extraction layers.

## Layer A --- Report XML

Extract:

-   queries
-   data items
-   expressions
-   filters
-   prompts
-   layouts
-   visual types
-   formatting
-   drill-through
-   variables
-   parameters

## Layer B --- Framework Manager

Extract:

-   query subjects
-   query items
-   relationships
-   cardinality
-   calculations
-   filters
-   packages
-   dimensions
-   hierarchies
-   determinants
-   security

## Layer C --- Cognos usage/content metadata

Extract:

-   folders
-   owners
-   execution history
-   schedules
-   dependencies
-   package usage
-   user/group information

## Layer D --- Physical source metadata

Extract:

-   schemas
-   tables
-   columns
-   data types
-   keys
-   views
-   relationships
-   source SQL where available

The strongest lineage graph is:

``` text
Report
 ↓
Query
 ↓
Package
 ↓
Framework Manager object
 ↓
Physical source
```

------------------------------------------------------------------------

# 7. Framework Manager → semantic model

## Query subject

Do not automatically convert every query subject to a Power BI table.

Determine whether it represents:

-   physical table
-   logical projection
-   business object
-   calculated object
-   bridge
-   helper
-   dimensional structure

## Query item

Possible targets:

-   Power BI column
-   DAX measure
-   calculated column
-   Power Query transformation
-   source SQL

Decision rule:

``` text
Row-level value
    → column

Reusable aggregation
    → measure

Data cleansing/transformation
    → Power Query/source

Complex business aggregation
    → DAX measure
```

------------------------------------------------------------------------

# 8. Relationships and cardinality

Extract:

-   left object
-   right object
-   join condition
-   cardinality
-   optionality
-   determinants
-   role
-   ambiguity

Example:

``` text
Customer.Customer_ID = Sales.Customer_ID
```

Possible target:

``` text
Customer[Customer_ID] 1 ─── * Sales[Customer_ID]
```

Do not infer filter direction blindly.

Power BI is optimized for well-designed semantic models, particularly
star schemas.

Reference:

https://learn.microsoft.com/en-us/power-bi/guidance/star-schema

------------------------------------------------------------------------

# 9. Determinants

Cognos determinants describe granularity/grouping behavior.

They may influence:

-   aggregation
-   duplicate suppression
-   relationship behavior
-   dimensional grain

There is no universal Power BI "determinant" object.

Translate the semantic effect into:

-   correct table grain
-   relationships
-   bridge tables
-   measure logic
-   source transformations

This should be classified as medium/high complexity when the effect is
not obvious.

------------------------------------------------------------------------

# 10. Cognos calculations → DAX

## SUM

Cognos:

``` text
sum([Revenue])
```

DAX:

``` dax
Total Revenue =
SUM ( Sales[Revenue] )
```

## COUNT

``` text
count([Customer ID])
```

``` dax
Customer Count =
COUNT ( Customer[Customer ID] )
```

## COUNT DISTINCT

``` text
count(distinct [Customer ID])
```

``` dax
Distinct Customers =
DISTINCTCOUNT ( Sales[Customer ID] )
```

## AVERAGE

``` text
average([Revenue])
```

``` dax
Average Revenue =
AVERAGE ( Sales[Revenue] )
```

## MIN/MAX

``` text
minimum([Revenue])
maximum([Revenue])
```

``` dax
Min Revenue = MIN ( Sales[Revenue] )

Max Revenue = MAX ( Sales[Revenue] )
```

## IF

Cognos:

``` text
if ([Revenue] > 10000) then ('High') else ('Low')
```

DAX:

``` dax
Revenue Band =
IF (
    Sales[Revenue] > 10000,
    "High",
    "Low"
)
```

## CASE

Cognos:

``` text
case
  when [Sales] > 100000 then 'High'
  when [Sales] > 50000 then 'Medium'
  else 'Low'
end
```

DAX:

``` dax
Sales Band =
SWITCH (
    TRUE(),
    Sales[Sales] > 100000, "High",
    Sales[Sales] > 50000, "Medium",
    "Low"
)
```

## COALESCE

Cognos:

``` text
coalesce([Amount], 0)
```

DAX:

``` dax
COALESCE ( [Amount], 0 )
```

## ROUND

``` text
round([Revenue], 2)
```

``` dax
ROUND ( [Revenue], 2 )
```

## String concatenation

Cognos:

``` text
[First Name] || ' ' || [Last Name]
```

DAX:

``` dax
Full Name =
Customer[First Name] & " " & Customer[Last Name]
```

## Upper/lower

``` text
upper([Name])
lower([Name])
```

``` dax
UPPER ( Customer[Name] )
LOWER ( Customer[Name] )
```

## Length

``` text
length([Name])
```

``` dax
LEN ( Customer[Name] )
```

## Substring

Use:

``` dax
LEFT()
MID()
RIGHT()
```

depending on the Cognos substring semantics.

## Division

Prefer:

``` dax
DIVIDE ( [Profit], [Revenue] )
```

over:

``` dax
[Profit] / [Revenue]
```

when zero/blank denominator behavior matters.

------------------------------------------------------------------------

# 11. Grain-aware conversion

A critical distinction:

Cognos:

``` text
[Quantity] * [Unit Price]
```

is row-level.

Potential calculated column:

``` dax
Line Revenue =
Sales[Quantity] * Sales[Unit Price]
```

But a report-level aggregate:

``` text
sum([Quantity] * [Unit Price])
```

should generally become:

``` dax
Revenue =
SUMX (
    Sales,
    Sales[Quantity] * Sales[Unit Price]
)
```

The migration engine must not turn every Cognos calculation into a
simple `SUM(column)`.

------------------------------------------------------------------------

# 12. Filter semantics

Cognos filters can occur at:

-   package/model
-   query
-   detail
-   summary
-   report
-   page
-   object
-   prompt-driven scope

Power BI filters can occur at:

-   semantic model
-   report
-   page
-   visual
-   slicer
-   drillthrough

A migration defect often occurs when a report-level Cognos filter
becomes only a visual-level Power BI filter.

The IR should store:

``` json
{
  "filter": {
    "expression": "...",
    "scope": "report"
  }
}
```

------------------------------------------------------------------------

# 13. Detail vs summary filters

Detail filter:

``` text
Revenue > 1000
```

may filter rows before aggregation.

Summary filter:

``` text
sum(Revenue) > 100000
```

filters after aggregation.

These must not be converted into the same Power BI behavior.

------------------------------------------------------------------------

# 14. Prompts

Cognos prompts may implement:

-   single selection
-   multi-selection
-   date selection
-   optional selection
-   cascading selection
-   search selection
-   query-driven selection
-   prompt pages
-   default values
-   macros

Power BI equivalents include:

-   slicers
-   filter pane
-   field parameters
-   what-if parameters
-   Power Query parameters
-   drillthrough

There is no universal 1:1 prompt mapping.

------------------------------------------------------------------------

# 15. Prompt mappings

  Cognos prompt               Typical Power BI target
  --------------------------- ----------------------------------------
  Single-value                Single-select slicer
  Multi-value                 Multi-select slicer
  Date prompt                 Date slicer
  Optional prompt             Slicer/filter with all-values behavior
  Cascading prompt            Related dimension slicers
  Dynamic metric prompt       Field parameter
  Numeric user input          What-if parameter
  Source query parameter      Power Query parameter
  Security-dependent prompt   RLS/security design

Capture:

-   default
-   required/optional
-   multi-select
-   dependencies
-   source query
-   parameter name
-   filter target

------------------------------------------------------------------------

# 16. Cascading prompts

Example:

``` text
Country → State → City
```

Power BI can reproduce this if the model correctly propagates filters:

``` text
Country
   ↓
State
   ↓
City
```

Incorrect relationships can cause cascading slicers to behave
differently from Cognos.

------------------------------------------------------------------------

# 17. Macros

Cognos macros may control:

-   prompt substitution
-   user/session identity
-   locale
-   dates
-   dynamic SQL
-   runtime behavior
-   query construction

Do not convert macros by string replacement.

First classify:

``` text
Security
Filter
Dynamic label
Query generation
Connection behavior
Runtime configuration
```

Then choose:

-   DAX
-   Power Query
-   source SQL
-   RLS
-   report configuration
-   workflow

Complex macros should normally receive a manual-review flag.

------------------------------------------------------------------------

# 18. Crosstab → Matrix

Basic mapping:

``` text
Cognos rows
    → Matrix rows

Cognos columns
    → Matrix columns

Cognos measures
    → Matrix values

Cognos totals
    → Matrix subtotals/grand totals
```

Complex cases requiring review:

-   asymmetric sets
-   member calculations
-   nested dimensions
-   custom member sets
-   advanced pagination
-   complex conditional cells

------------------------------------------------------------------------

# 19. List → Table or Paginated

Use Power BI Table visual when the report is interactive.

Use Power BI Paginated Report when it requires:

-   fixed layout
-   printing
-   repeating headers
-   many pages
-   operational distribution
-   PDF/Excel output
-   pixel-precise formatting

Microsoft documents paginated reports as print-friendly, multipage and
pixel-precise reports.

Reference:

https://learn.microsoft.com/en-us/power-bi/paginated-reports/report-builder-power-bi

------------------------------------------------------------------------

# 20. Chart conversion

Typical mappings:

  Cognos        Power BI
  ------------- -------------------------------------------
  Bar           Bar/column
  Line          Line
  Pie           Pie/donut
  Combination   Line + column
  Scatter       Scatter
  Area          Area
  Gauge         Gauge
  Map           Map/Azure Maps/other supported map visual

Extract before mapping:

-   category
-   series
-   measures
-   aggregation
-   sorting
-   axis
-   legend
-   labels
-   title
-   formatting
-   reference lines
-   conditional formatting

------------------------------------------------------------------------

# 21. Conditional formatting

Cognos conditional formatting may use thresholds, expressions or
variables.

Power BI can use:

-   rules
-   color scales
-   field values
-   DAX measures

Example:

``` dax
Variance Color =
SWITCH (
    TRUE(),
    [Variance] < 0, "#FF0000",
    [Variance] > 0, "#008000",
    "#808080"
)
```

The conversion engine should preserve the condition and target property
rather than just the color.

------------------------------------------------------------------------

# 22. Variables

Cognos variables can control:

-   visibility
-   style
-   content
-   conditional sections

Possible Power BI targets:

-   bookmarks
-   page navigation
-   conditional formatting
-   DAX measures
-   field parameters
-   visual filters

There is no single Power BI replacement for every Cognos variable.

------------------------------------------------------------------------

# 23. Drill-through

Cognos drill-through commonly passes context from a source report to a
target report.

Power BI target:

``` text
Source visual
     ↓
Drillthrough page
     ↓
Filter context
```

Preserve:

-   source field
-   target field
-   target report/page
-   parameter mapping
-   default behavior

------------------------------------------------------------------------

# 24. Master/detail

Cognos master/detail may become:

-   drillthrough
-   page navigation
-   bookmarks
-   synchronized slicers
-   detail pages
-   matrix expansion

Select based on the desired user experience.

------------------------------------------------------------------------

# 25. Bursting and scheduling

Cognos bursting is an operational distribution capability.

It should not be treated as an ordinary visual conversion.

Possible targets:

-   Power BI subscriptions
-   paginated report subscriptions
-   Power Automate
-   application-level workflows
-   separate workspaces/audiences

Cognos scheduling and Power BI data refresh are also separate concepts.

Migration should distinguish:

``` text
Report execution schedule
Data refresh schedule
Distribution schedule
Bursting
```

------------------------------------------------------------------------

# 26. Security

Inventory security at:

-   namespace
-   folder
-   package
-   report
-   object
-   data level

Power BI equivalents can include:

-   workspace permissions
-   app permissions
-   semantic-model permissions
-   RLS
-   OLS
-   Microsoft Entra groups

Do not convert every Cognos permission into RLS.

Separate:

``` text
Content authorization
Data authorization
Object authorization
```

------------------------------------------------------------------------

# 27. RLS conversion

If Cognos has a rule such as:

``` text
Region = user's allowed region
```

a Power BI implementation might use a user-security table:

``` text
User
  |
UserRegion
  |
Region
  |
Sales
```

and RLS based on user identity.

Example:

``` dax
[UserEmail] = USERPRINCIPALNAME()
```

The exact implementation depends on the Cognos security architecture.

------------------------------------------------------------------------

# 28. OLAP/dimensional Cognos

Dimensional Cognos migrations can be significantly more difficult.

Important concepts:

-   dimensions
-   hierarchies
-   levels
-   members
-   attributes
-   measures
-   calculated members
-   named sets
-   member functions
-   dimensional filters

Power BI's tabular model is not a literal copy of every Cognos cube
behavior.

Potential target:

``` text
Dimensions
   ↓
Fact tables
   ↓
DAX measures
   ↓
Hierarchies
   ↓
Calculation groups where appropriate
```

Named sets and member expressions may require redesign.

------------------------------------------------------------------------

# 29. Date and time intelligence

Cognos reports commonly implement:

-   year
-   quarter
-   month
-   week
-   YTD
-   MTD
-   prior period
-   rolling periods

Use a dedicated Date dimension in Power BI.

Example:

``` dax
Sales YTD =
TOTALYTD (
    [Sales],
    'Date'[Date]
)
```

Prior year:

``` dax
Sales PY =
CALCULATE (
    [Sales],
    SAMEPERIODLASTYEAR ( 'Date'[Date] )
)
```

Do not recreate identical date logic in every report.

------------------------------------------------------------------------

# 30. Percentage of total

Cognos:

``` text
[Revenue] / total([Revenue])
```

Potential Power BI:

``` dax
Revenue % =
DIVIDE (
    [Revenue],
    CALCULATE (
        [Revenue],
        ALLSELECTED ( Sales )
    )
)
```

But this is **not a universal Cognos TOTAL replacement**.

The engine must determine:

-   which grouping is removed
-   which filters remain
-   whether prompt filters remain
-   whether page filters remain
-   whether visual filters remain

Sometimes `REMOVEFILTERS` or `ALL` on a specific dimension is more
correct.

------------------------------------------------------------------------

# 31. Running total

Possible DAX:

``` dax
Running Revenue =
CALCULATE (
    [Revenue],
    FILTER (
        ALLSELECTED ( 'Date'[Date] ),
        'Date'[Date] <= MAX ( 'Date'[Date] )
    )
)
```

The date grain and intended filter scope must be inferred from the
Cognos report.

------------------------------------------------------------------------

# 32. Ranking

Possible DAX:

``` dax
Revenue Rank =
RANKX (
    ALLSELECTED ( Customer[Customer] ),
    [Revenue],
    ,
    DESC
)
```

The ranking domain must match the Cognos grouping context.

------------------------------------------------------------------------

# 33. Semantic-model design

Prefer:

``` text
             Date
              |
Customer ── Sales ── Product
              |
           Geography
```

Dimensions:

-   Date
-   Customer
-   Product
-   Geography
-   Employee

Facts:

-   Sales
-   Orders
-   Inventory
-   Budget

Maintain consistent fact grain.

Reference:

https://learn.microsoft.com/en-us/power-bi/guidance/star-schema

------------------------------------------------------------------------

# 34. Shared semantic models

If 30 Cognos reports use the same package/model:

``` text
Cognos package
      ↓
Shared Power BI semantic model
      ↓
Report A
Report B
Report C
...
```

Do not automatically generate 30 independent Power BI semantic models.

Look for:

-   common dimensions
-   common measures
-   repeated calculations
-   common source tables
-   common business definitions

------------------------------------------------------------------------

# 35. Power Query vs DAX vs source SQL

Use source SQL/warehouse for:

-   large joins
-   reusable enterprise transformations
-   heavy cleansing
-   shared business logic

Use Power Query for:

-   ingestion
-   shaping
-   cleansing
-   type conversion
-   moderate transformations

Use calculated columns for:

-   row-level derived attributes

Use measures for:

-   aggregations
-   KPIs
-   ratios
-   filter-aware calculations
-   time intelligence

This prevents overloading DAX with transformation work.

------------------------------------------------------------------------

# 36. PBIP

Power BI Project (PBIP) stores report and semantic-model definitions as
text files in a folder structure.

Conceptually:

``` text
Project/
├── Report/
│   ├── definition.pbir
│   └── definition/
├── SemanticModel/
│   ├── definition.pbism
│   └── definition/
└── Project.pbip
```

Power BI project capabilities and exact structures are
version-sensitive.

Reference:

https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-overview

------------------------------------------------------------------------

# 37. PBIR

PBIR is the enhanced Power BI report format.

It organizes report metadata into structured files and is designed for
source control and programmatic editing.

Microsoft notes that PBIR represents pages, visuals, bookmarks and other
report objects separately and has public JSON schemas.

Reference:

https://learn.microsoft.com/en-us/power-bi/developer/embedded/projects-enhanced-report-format

Use PBIR carefully:

-   validate against the target Desktop version
-   validate schemas
-   do not assume preview behavior is permanent
-   open generated projects in Power BI Desktop
-   validate all references

------------------------------------------------------------------------

# 38. TMDL

TMDL = Tabular Model Definition Language.

It can represent:

-   tables
-   columns
-   measures
-   relationships
-   hierarchies
-   roles
-   model metadata

TMDL is useful for programmatic semantic-model generation.

References:

https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-tmdl-view

https://learn.microsoft.com/en-us/analysis-services/tmdl/tmdl-how-to?view=sql-analysis-services-2025

------------------------------------------------------------------------

# 39. PBIP generation pipeline

Recommended:

``` text
Cognos Model Metadata
       ↓
Canonical Semantic IR
       ↓
Tables
Columns
Relationships
Measures
Hierarchies
Roles
       ↓
TMDL
       ↓
Power BI semantic model
```

Report:

``` text
Cognos Report XML
       ↓
Canonical Report IR
       ↓
Pages
Visuals
Fields
Filters
Interactions
       ↓
PBIR
       ↓
PBIP
```

------------------------------------------------------------------------

# 40. Intermediate Representation

Do not generate PBIP directly from Cognos XML.

Use:

``` json
{
  "sourcePlatform": "Cognos",
  "sourceVersion": "...",
  "report": {},
  "model": {},
  "queries": [],
  "tables": [],
  "columns": [],
  "measures": [],
  "relationships": [],
  "filters": [],
  "parameters": [],
  "prompts": [],
  "visuals": [],
  "pages": [],
  "security": [],
  "drillthrough": [],
  "schedules": [],
  "bursting": [],
  "lineage": [],
  "conversionWarnings": []
}
```

This also makes the accelerator extensible:

``` text
Tableau ─┐
Qlik ────┤
Cognos ──┼→ Canonical BI IR → Power BI
SSRS ────┘
```

------------------------------------------------------------------------

# 41. Expression AST

Avoid string replacement.

Cognos:

``` text
sum([Revenue]) / nullif(sum([Sales]),0)
```

Normalize:

``` text
DIVIDE(
  SUM(Revenue),
  SUM(Sales)
)
```

Generate:

``` dax
Revenue Ratio =
DIVIDE (
    [Revenue],
    [Sales]
)
```

Store both source and target expressions.

------------------------------------------------------------------------

# 42. Lineage

Every generated object should have lineage.

Example:

``` json
{
  "target": "[Total Revenue]",
  "source": [
    "Report: Sales by Region",
    "Query: Query1",
    "Data Item: Total Revenue",
    "Expression: sum([Sales].[Revenue])",
    "Package: Sales Package"
  ]
}
```

Lineage is critical for:

-   debugging
-   validation
-   explainability
-   human review
-   regression testing

------------------------------------------------------------------------

# 43. Conversion confidence

Every generated object should receive confidence.

Example:

``` text
SUM → SUM
99%

COUNT DISTINCT → DISTINCTCOUNT
99%

CASE → SWITCH(TRUE())
97%

Cognos macro → DAX
40%

Complex OLAP named set → DAX
30%

Bursting → workflow
50%
```

Confidence should represent semantic certainty, not parser success.

------------------------------------------------------------------------

# 44. Complexity scoring

Low:

-   simple lists
-   simple tables
-   basic charts
-   SUM/COUNT
-   basic filters
-   simple prompts

Medium:

-   crosstabs
-   nested calculations
-   multiple prompts
-   drillthrough
-   conditional formatting
-   complex joins

High:

-   OLAP
-   calculated members
-   named sets
-   macros
-   bursting
-   advanced security
-   nested reports
-   dynamic query generation
-   custom extensions

------------------------------------------------------------------------

# 45. Validation framework

## Structural

Check:

-   report exists
-   pages exist
-   visuals exist
-   model loads
-   relationships exist
-   measures compile

## Metadata

Compare:

-   columns
-   measures
-   data types
-   relationships
-   filters
-   parameters
-   security

## Numerical

Compare:

-   totals
-   subtotals
-   distinct counts
-   KPIs
-   ratios
-   time intelligence

## Visual

Compare:

-   page count
-   visual count
-   titles
-   fields
-   filters
-   sorting
-   formatting

## Behavioral

Test:

-   slicers
-   prompts
-   drillthrough
-   navigation
-   interactions
-   security
-   export
-   refresh

------------------------------------------------------------------------

# 46. Golden test cases

For every important report create:

``` text
Default state
Region = West
Year = 2025
Product = A
Customer = All
```

Compare Cognos and Power BI results.

A grand-total-only validation is insufficient.

------------------------------------------------------------------------

# 47. Common migration pitfalls

## 1. Ignoring Framework Manager

Report XML alone may not contain the complete semantic logic.

## 2. Wrong filter scope

A report-level Cognos filter can become incorrectly implemented at
visual level.

## 3. Wrong aggregation grain

Row expressions and aggregate expressions must be separated.

## 4. Duplicate joins

Poor relationship conversion can inflate totals.

## 5. Hidden model calculations

A package query item may hide a Framework Manager calculation.

## 6. Prompt defaults

Default/optional/multi-select behavior can be lost.

## 7. Macro behavior

Dynamic query construction is difficult to translate safely.

## 8. Security

Content permissions and data permissions are different.

## 9. Report count

Thousands of legacy reports do not necessarily mean thousands of target
reports.

## 10. Pixel-perfect requirements

Some reports should become paginated reports instead of interactive
Power BI reports.

------------------------------------------------------------------------

# 48. Paginated report strategy

Use Power BI paginated reports for:

-   invoices
-   statements
-   operational reports
-   fixed-layout reports
-   multi-page reports
-   repeating headers
-   PDF/Excel output
-   highly formatted tables

Power BI paginated reports use Report Builder/RDL and are separate from
standard interactive Power BI reports.

References:

https://learn.microsoft.com/en-us/power-bi/paginated-reports/

https://learn.microsoft.com/en-us/power-bi/paginated-reports/report-builder-power-bi

------------------------------------------------------------------------

# 49. Interactive report strategy

Use standard Power BI reports for:

-   executive dashboards
-   KPIs
-   trend analysis
-   interactive filtering
-   drill-down
-   cross-filtering
-   exploratory analytics

The accelerator should classify the source report before selecting the
target artifact.

------------------------------------------------------------------------

# 50. Public migration experience

Public Cognos→Power BI discussions repeatedly identify:

-   report inventory/rationalization
-   filters and prompts
-   source-system changes
-   semantic-model redesign
-   list vs crosstab complexity
-   security
-   numerical validation
-   automation of repetitive work

A recent community discussion also emphasized that organizations should
identify inactive/duplicate reports and treat cubes, security and
business logic as major scope drivers.

Community references:

https://www.reddit.com/r/PowerBI/comments/1bto46n/to_those_of_you_who_had_experience_migrating_from/

https://www.reddit.com/r/PowerBI/comments/1s1ipd2/cognos_to_power_bi_migration/

Treat community posts as practitioner evidence, not authoritative
product documentation.

------------------------------------------------------------------------

# 51. Public migration tooling

Senturus has publicly described a **Migration Assistant** for Cognos
migrations. Its public material focuses on programmatically
decoding/inventorying Cognos content, lineage, usage and migration
scoping.

References:

https://senturus.com/

https://www.youtube.com/watch?v=W7KlYC3BK0g

https://www.youtube.com/watch?v=p_W3NY35rd0

Vendor capability statements should be treated as vendor claims and
validated with a proof of concept.

------------------------------------------------------------------------

# 52. What to automate

Strong automation candidates:

-   inventory
-   XML parsing
-   package mapping
-   query extraction
-   expression extraction
-   expression normalization
-   lineage
-   simple DAX generation
-   table/column generation
-   relationship generation
-   measure generation
-   page generation
-   visual field mapping
-   filter mapping
-   PBIP scaffolding
-   TMDL generation
-   PBIR generation
-   validation metadata
-   migration logs

------------------------------------------------------------------------

# 53. What should remain human-reviewed

Usually:

-   complex OLAP calculations
-   macros
-   advanced security
-   bursting
-   nested reports
-   custom extensions
-   ambiguous business logic
-   complex crosstabs
-   source-system redesign
-   unsupported visuals
-   exact pixel-perfect requirements

The accelerator should surface uncertainty rather than silently generate
an incorrect target.

------------------------------------------------------------------------

# 54. Recommended architecture

``` text
                 COGNOS
                    |
        +-----------+-----------+
        |                       |
 Report Specifications      Model Metadata
        |                       |
        +-----------+-----------+
                    |
              Metadata Parser
                    |
               Metadata Graph
                    |
               Canonical IR
                    |
        +-----------+-----------+
        |                       |
 Semantic Converter       Report Converter
        |                       |
       TMDL                    PBIR
        |                       |
        +-----------+-----------+
                    |
                   PBIP
                    |
              Validation
                    |
             Human Review
                    |
                 Publish
```

------------------------------------------------------------------------

# 55. Recommended extractor modules

## Cognos Report Parser

-   XML parsing
-   queries
-   layouts
-   prompts
-   filters
-   visuals
-   drillthrough

## Framework Manager Parser

-   query subjects
-   query items
-   relationships
-   calculations
-   filters
-   packages
-   dimensions
-   hierarchies

## Usage Analyzer

-   execution frequency
-   last execution
-   owners
-   dependencies

## Expression Engine

-   Cognos expression parsing
-   AST
-   grain classification
-   function mapping
-   DAX/M/SQL generation

## Model Generator

-   tables
-   columns
-   measures
-   relationships
-   hierarchies
-   roles

## Report Generator

-   pages
-   visuals
-   filters
-   slicers
-   interactions
-   drillthrough

## Validation Engine

-   structural
-   metadata
-   numerical
-   visual
-   behavioral

------------------------------------------------------------------------

# 56. Knowledge repository design

Recommended chunks:

``` text
01_Cognos_Architecture
02_Framework_Manager
03_Data_Modules
04_Packages
05_Report_Specifications
06_Query_Expressions
07_Function_Mapping
08_Filters
09_Prompts
10_Parameters
11_Macros
12_Calculations
13_Crosstabs
14_Charts
15_Conditional_Formatting
16_Drillthrough
17_Bursting
18_Scheduling
19_Security
20_OLAP
21_PowerQuery
22_DAX
23_Semantic_Model
24_PBIP
25_PBIR
26_TMDL
27_Paginated
28_Validation
29_Performance
30_Migration_Best_Practices
31_Public_Migration_Experience
32_Troubleshooting
```

Recommended metadata:

``` json
{
  "source": "Cognos",
  "target": "PowerBI",
  "category": "calculation",
  "feature": "distinct_count",
  "complexity": "low",
  "confidence": 0.99
}
```

------------------------------------------------------------------------

# 57. Rule-library design

Example:

``` json
{
  "ruleId": "COGNOS_SUM_001",
  "source": "Cognos",
  "sourcePattern": "sum(expression)",
  "target": "PowerBI",
  "targetType": "DAX_MEASURE",
  "template": "SUM({column})",
  "confidence": 0.99,
  "requiresReview": false
}
```

Rules should include:

-   source pattern
-   normalized AST
-   target expression
-   target artifact type
-   supported contexts
-   edge cases
-   examples
-   confidence
-   validation tests

------------------------------------------------------------------------

# 58. AI migration context

For every object provide the AI engine:

``` text
Source platform
Source version
Target platform
Cognos report
Package
Query
Expression
Expression AST
Lineage
Grain
Scope
Security context
Retrieved mapping rules
Similar migration examples
Validation rules
```

Important instructions:

``` text
Do not invent source lineage.
Do not assume a query item is a measure.
Do not assume filter scope.
Do not translate macros without classification.
Do not create relationships without cardinality evidence.
Do not claim unsupported visual equivalence.
Return confidence and warnings.
```

------------------------------------------------------------------------

# 59. Generated migration manifest

Example:

``` json
{
  "sourceReport": "Sales by Region",
  "targetReport": "Sales by Region",
  "sourcePackage": "Sales Package",
  "targetSemanticModel": "Sales Model",
  "status": "PARTIALLY_AUTOMATED",
  "confidence": 0.91,
  "automatedObjects": 47,
  "manualReviewObjects": 5,
  "warnings": [
    "Complex prompt macro",
    "Crosstab formatting requires review"
  ],
  "validation": {
    "metadata": "PASS",
    "measures": "PASS",
    "totals": "PASS",
    "visuals": "REVIEW"
  }
}
```

------------------------------------------------------------------------

# 60. Migration KPI dashboard

Track:

``` text
Total Cognos reports
Active reports
Retired reports
Consolidated reports
Migrated reports
Auto-generated reports
Manual-review reports
Automation percentage
Expression conversion rate
Visual conversion rate
Validation pass rate
Security validation rate
Average remediation time
Average migration time
```

------------------------------------------------------------------------

# 61. Final migration checklist

## Discovery

-   [ ] Cognos version identified
-   [ ] Reports inventoried
-   [ ] Packages inventoried
-   [ ] Framework Manager projects inventoried
-   [ ] Data modules inventoried
-   [ ] Data sources inventoried
-   [ ] Security inventoried
-   [ ] Schedules inventoried
-   [ ] Bursting inventoried
-   [ ] OLAP dependencies inventoried

## Model

-   [ ] Tables identified
-   [ ] Columns identified
-   [ ] Measures identified
-   [ ] Calculated columns identified
-   [ ] Relationships identified
-   [ ] Cardinality validated
-   [ ] Grain validated
-   [ ] Hierarchies identified
-   [ ] Security mapped
-   [ ] Star schema reviewed

## Report

-   [ ] Pages identified
-   [ ] Visuals identified
-   [ ] Fields mapped
-   [ ] Filters mapped
-   [ ] Prompts mapped
-   [ ] Parameters mapped
-   [ ] Drillthrough mapped
-   [ ] Conditional formatting mapped
-   [ ] Navigation mapped
-   [ ] Export requirements identified

## PBIP

-   [ ] PBIP structure generated
-   [ ] Semantic model generated
-   [ ] TMDL validated
-   [ ] Report definition generated
-   [ ] PBIR validated
-   [ ] Power BI Desktop opens project
-   [ ] References valid
-   [ ] Measures compile

## Validation

-   [ ] Totals match
-   [ ] Subtotals match
-   [ ] Filtered totals match
-   [ ] Distinct counts match
-   [ ] KPIs match
-   [ ] Prompt behavior validated
-   [ ] Drillthrough validated
-   [ ] Security validated
-   [ ] Visuals reviewed
-   [ ] Performance tested
-   [ ] UAT completed

------------------------------------------------------------------------

# 62. Final principle

A Cognos → Power BI accelerator should behave like a **compiler**, not a
file converter:

``` text
Cognos
  ↓
Parse
  ↓
Understand
  ↓
Normalize
  ↓
Resolve lineage
  ↓
Determine semantic grain
  ↓
Map business logic
  ↓
Generate Power BI model
  ↓
Generate report
  ↓
Validate
  ↓
Human review where uncertainty remains
```

The highest-value capability is not merely producing a PBIP file. It is
producing a **correct, traceable and explainable Power BI implementation
from Cognos semantics**, while clearly identifying what was automated
and what still requires human validation.

------------------------------------------------------------------------

# 63. Primary references

## IBM

Cognos Analytics: https://www.ibm.com/docs/en/cognos-analytics

Framework Manager:
https://www.ibm.com/docs/en/cognos-analytics/12.1.x?topic=manager-getting-started-framework

Framework Manager model:
https://www.ibm.com/docs/en/pm-and-q/2.6.3?topic=product-cognos-framework-manager-model-description

Report creation:
https://www.ibm.com/docs/en/cognos-analytics/12.0.x?topic=reporting-creating-report

Report specification structure:
https://www.ibm.com/docs/en/cognos-analytics/12.1.0?topic=specifications-basic-structure-report-specification

Using report specifications:
https://www.ibm.com/docs/en/cognos-analytics/12.1.x?topic=developer-using-report-specifications

Cognos SDK:
https://www.ibm.com/docs/en/cognos-analytics/11.2.x?topic=report-cognos-software-development-kit

Cognos functionality mapping:
https://www.ibm.com/docs/en/cognos-analytics/12.0.x?topic=products-functionality-mapping-from-cognos-bi-cognos-analytics

Report specification schema reference:
https://www.ibm.com/support/pages/node/492723

## Microsoft

Power BI migration preparation:
https://learn.microsoft.com/en-us/power-bi/guidance/powerbi-migration-pre-migration-steps

Migration requirements:
https://learn.microsoft.com/en-us/power-bi/guidance/powerbi-migration-requirements

Migration planning:
https://learn.microsoft.com/en-us/power-bi/guidance/powerbi-migration-planning

Proof of concept:
https://learn.microsoft.com/en-us/power-bi/guidance/powerbi-migration-proof-of-concept

Create/validate:
https://learn.microsoft.com/en-us/power-bi/guidance/powerbi-migration-create-validate-content

Deploy/support/monitor:
https://learn.microsoft.com/en-us/power-bi/guidance/powerbi-migration-deploy-support-monitor

Star schema:
https://learn.microsoft.com/en-us/power-bi/guidance/star-schema

PBIP:
https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-overview

PBIR:
https://learn.microsoft.com/en-us/power-bi/developer/embedded/projects-enhanced-report-format

PBIR report folder:
https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-report

TMDL:
https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-tmdl-view

TMDL API:
https://learn.microsoft.com/en-us/analysis-services/tmdl/tmdl-how-to?view=sql-analysis-services-2025

Paginated reports:
https://learn.microsoft.com/en-us/power-bi/paginated-reports/

Power BI Report Builder:
https://learn.microsoft.com/en-us/power-bi/paginated-reports/report-builder-power-bi

## Public migration/practitioner material

Senturus: https://senturus.com/

Senturus Cognos migration material:
https://www.youtube.com/watch?v=W7KlYC3BK0g

Updated Cognos migration material:
https://www.youtube.com/watch?v=p_W3NY35rd0

Practitioner discussion:
https://www.reddit.com/r/PowerBI/comments/1bto46n/to_those_of_you_who_had_experience_migrating_from/

Recent Cognos migration discussion:
https://www.reddit.com/r/PowerBI/comments/1s1ipd2/cognos_to_power_bi_migration/

Community material should be treated as anecdotal evidence, while IBM
and Microsoft documentation should be treated as the primary technical
authority.

------------------------------------------------------------------------

## Version note

Cognos report specifications, Framework Manager behavior and available
APIs vary by Cognos version.

Power BI PBIP/PBIR/TMDL capabilities and schemas evolve with Power BI
Desktop and Fabric releases.

For production automation, store:

``` text
sourceVersion
targetVersion
ruleVersion
schemaVersion
```

with every migration artifact.
