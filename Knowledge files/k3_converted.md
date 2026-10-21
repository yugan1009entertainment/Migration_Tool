# 📘 CONVERSION KNOWLEDGE FILE
# K3 – TMDL Strict Compliance Rules (Bug-Fix Knowledge Base)
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference for TMDL syntax
#          compliance. Apply ALL rules automatically to every generated
#          TMDL file before delivery. These rules are non-negotiable.
# Generated: 2026-07-23
# Derived from: Live Power BI Desktop May 2026 (v2.154.1260.0)
#               validation errors during pair_3 conversion
# Scope: Rules M11–M17 — 7 rules discovered through iterative error
#        correction when loading generated TMDL files into PBI Desktop

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TMDL COMPLIANCE RULES OVERVIEW
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

7 rules discovered through iterative error correction when loading
generated TMDL files into Power BI Desktop May 2026. Each rule has:
the exact error message, root cause, and correct pattern.

**These rules are non-negotiable. Apply all 7 to every file. Never skip.**

| Rule   | One-line Summary                                                    | Error if Violated                           |
|--------|---------------------------------------------------------------------|---------------------------------------------|
| M11    | `lineageTag` = `[a-zA-Z0-9_-]+` kebab-case only                    | `InvalidLineType`                           |
| M12    | Multi-line DAX → wrap in ``` backtick blocks                       | `InvalidLineType: Unexpected line type: Empty!` |
| M13    | `///` only on table/column/measure/hierarchy                        | `Property 'description' is unknown`         |
| M14    | SAP HANA: `Value.NativeQuery(Source, sql, null, [...])`             | Data load failure                           |
| M15    | Omit `crossFilteringBehavior: oneDirection` (it's default)          | Parse warning                               |
| M16    | No blank lines before `annotation`/`changedProperty`                | `InvalidLineType`                           |
| M17    | `PBI_NavigationStepName`/`PBI_ResultType` inside partition          | Semantics mismatch                          |


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — RULE M11: lineageTag KEBAB-CASE ONLY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### RULE-M11 | lineageTag: Kebab-Case Only
─────────────────────────────────────────────────────────────
Error Message   : InvalidLineType (silent parse failure)
Severity        : FATAL — parser crash
Applies to      : ALL TMDL files

Root Cause:
───────────
TMDL `lineageTag` values are parsed as identifiers. Special characters
including `()`, `[]`, spaces, commas, `!`, `/`, dots are invalid and
crash the parser.

Rule:
─────
`lineageTag` values MUST match `[a-zA-Z0-9_-]+` only.

Naming Convention:
──────────────────
| Object             | Pattern                              | Example                                  |
|--------------------|--------------------------------------|------------------------------------------|
| Source column      | `<prefix>-col-<name>`                | `csq-col-order-created-on-date`          |
| Calculated column  | `<prefix>-calc-<name>`               | `csq-calc-date-selection`                |
| Measure            | `<prefix>-meas-<name>`               | `mt-meas-total-sales`                    |
| Action column      | `<prefix>-action-<slug>`             | `csq-action-week-year-month-soldto`      |
| Hierarchy          | `<prefix>-hier-<name>`               | `csq-hier-sold-to-ship-to`              |
| Hierarchy level n  | `<prefix>-hier-lvl-<n>`              | `csq-hier-lvl-1`                         |
| Table              | `<prefix>-0001-0000-0000-<12digits>` | `csq-0001-0000-0000-000000000001`        |
| Parameter column   | `param-<n>-<type>`                   | `param-3-key`                            |

Table prefix codes:
  csq  = Custom SQL Query
  mt   = Measures Table
  param = Parameters
  ldt<n> = LocalDateTable n
  hcp  = HCP table

Bad → Good Examples:
────────────────────
❌  lineageTag: [ORDER_CREATED_ON_DATE]
❌  lineageTag: Calculation_1242149131112218624
❌  lineageTag: Action (Category,Submission Date)
❌  lineageTag: Action (!Reporting Week,YEAR(!Date Selection),MONTH(!Date Selection),SOLD_TO_NAME (group))

✅  lineageTag: csq-col-order-created-on-date
✅  lineageTag: csq-calc-date-selection
✅  lineageTag: csq-action-category-submission-date
✅  lineageTag: csq-action-week-year-month-soldto

⚠️  RETROFIT: All k1/k2 outputs using [BRACKET], Calculation_NNNNN,
    or Action (X,Y,Z) lineageTags must be corrected before loading.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — RULE M12: MULTI-LINE DAX TRIPLE-BACKTICK BLOCKS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### RULE-M12 | Multi-line DAX Must Use Triple-Backtick Blocks
─────────────────────────────────────────────────────────────
Error Message   : TMDL Format Error:
                  Parsing error type - InvalidLineType
                  Detailed error - Unexpected line type: Empty!
                  Document - './tables/Custom_SQL_Query'
Severity        : FATAL — parser crash
Applies to      : ALL table .tmdl files with DAX expressions

Root Cause:
───────────
Without backtick wrapping, the TMDL parser uses indentation to
determine scope. Any blank line or VAR/RETURN block causes it to
misinterpret property lines (e.g., `dataType:`) as DAX content,
then crash on what it thinks is an empty line.

Rule:
─────
Any DAX expression spanning more than 1 line MUST be wrapped in
triple-backtick blocks.

Always use backtick block if expression contains any of:
  VAR, RETURN, SWITCH(TRUE(),, SUMX(, AVERAGEX(,
  CALCULATETABLE(, nested IF(, FILTER(

❌ WRONG — bare multi-line DAX (CRASHES):
```
column '!Reporting Month' =
    VAR _ds = 'Table'[Date]
    VAR _today = TODAY()
    RETURN FORMAT(YEAR(_ds), "0")
    dataType: string
    lineageTag: csq-calc-reporting-month
```

✅ CORRECT — triple-backtick wrapped:
```
column '!Reporting Month' = ```
        VAR _ds = 'Custom_SQL_Query'[!Date Selection]
        VAR _today = TODAY()
        VAR _curYM = YEAR(_today) * 100 + MONTH(_today)
        VAR _prevYM = YEAR(EDATE(_today, -1)) * 100 + MONTH(EDATE(_today, -1))
        VAR _dsYM = YEAR(_ds) * 100 + MONTH(_ds)
        RETURN
            SWITCH(TRUE(),
                _dsYM = _prevYM, "Previous",
                _dsYM = _curYM, "Current",
                FORMAT(_dsYM, "0")
            )
        ```
    dataType: string
    lineageTag: csq-calc-reporting-month
    summarizeBy: none
```

Indentation Rules:
──────────────────
  - Opening ``` immediately after = on declaration line
  - DAX body: 8 spaces indent inside block
  - Closing ```: own line, same indent as body
  - dataType:, lineageTag: etc. resume at 4-space indent after closing ```


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — RULE M13: TRIPLE-SLASH COMMENTS RESTRICTIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### RULE-M13 | Triple-Slash Comments Only on Description-Supporting Objects
─────────────────────────────────────────────────────────────
Error Message   : Property 'description' is unknown and is not expected
                  in the situation it appears.
                  Stack trace: SingleColumnRelationship.ReadMetadataProperties
Severity        : FATAL — parser error
Applies to      : ALL TMDL files

Root Cause:
───────────
`///` (triple-slash) lines are syntactic sugar for the `description`
property. `SingleColumnRelationship` and several other objects do NOT
have this property, so the parser throws "unknown property".

Rule:
─────
Use `///` ONLY on objects that support `description`.
Use `//` for everything else.

Objects Supporting `///`:
─────────────────────────
| Object                         | `///` Allowed? |
|--------------------------------|----------------|
| `table`                        | ✅ Yes         |
| `column` (source or calculated)| ✅ Yes         |
| `measure`                      | ✅ Yes         |
| `hierarchy`                    | ✅ Yes         |
| `role`                         | ✅ Yes         |
| **`relationship`**             | ❌ NO          |
| **`partition`**                | ❌ NO          |
| **hierarchy `level`**          | ❌ NO          |
| **`variation`**                | ❌ NO          |
| **`annotation`**               | ❌ NO          |
| **`changedProperty`**          | ❌ NO          |
| **`roleMember`**               | ❌ NO          |

Safe rule of thumb: `///` only on `table`, `column`, `measure`,
`hierarchy`. `//` for everything else.

❌ WRONG:
```
/// Currency join: Table.Col → TCURR.Col
relationship rel-currency-fcurr
    fromColumn: 'Custom_SQL_Query'.SD_DOC_CURRENCY
    toColumn: 'TCURR'.'From currency'
```

✅ CORRECT:
```
// Currency join: Table.Col → TCURR.Col
relationship rel-currency-fcurr
    fromColumn: 'Custom_SQL_Query'.SD_DOC_CURRENCY
    toColumn: 'TCURR'.'From currency'
```


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — RULE M14: SAP HANA ROOT-LEVEL VALUE.NATIVEQUERY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### RULE-M14 | SAP HANA: Root-Level Value.NativeQuery + null Params
─────────────────────────────────────────────────────────────
Error Message   : Data load / connector failure during refresh
                  [DataSource.Error] or gateway connection error
Severity        : FATAL — data load failure
Applies to      : SAP HANA partitions only

Root Cause:
───────────
Schema navigation (`Source{[Name="_SYS_BIC"]}[Data]`) before
`Value.NativeQuery` is unreliable in HANA connector v2.0. The SQL
already contains fully-qualified schema names, so schema navigation
in M is redundant and error-prone.

Rule:
─────
Pass SOURCE ROOT directly to `Value.NativeQuery`.
Use `null` (not `[]`) as the third parameter.

❌ WRONG:
```
Result = Value.NativeQuery(Source{[Name="_SYS_BIC"]}[Data], Query, [], [EnableFolding=true])
```

✅ CORRECT:
```
Result = Value.NativeQuery(Source, Query, null, [EnableFolding=true])
```

HANA SQL Identifier Quoting:
─────────────────────────────
Inside the M string, HANA schema/table names with special chars use
double-quote escaping:
  ""_SYS_BIC"".""SAP_ERP.TABLE_CV_S4/CV_VBAK""

Infrastructure Requirements:
─────────────────────────────
  - Requires Power BI On-Premises Data Gateway
  - Credentials: Basic (username/password)
  - Document gateway requirement in conversion report Prerequisites section


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — RULE M15: NO crossFilteringBehavior: oneDirection
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### RULE-M15 | Never Emit crossFilteringBehavior: oneDirection
─────────────────────────────────────────────────────────────
Error Message   : Parse warning (non-fatal but noisy; may become
                  fatal in future versions)
Severity        : WARNING — non-fatal but noisy
Applies to      : relationships.tmdl

Root Cause:
───────────
`crossFilteringBehavior: oneDirection` is the default value. Emitting
it explicitly is redundant.

Rule:
─────
Omit `crossFilteringBehavior: oneDirection` entirely.
Only emit `crossFilteringBehavior: bothDirections` when explicitly
required (RLS passthrough, many-to-many bridge).

❌ WRONG:
```
relationship rel-currency-fcurr
    fromColumn: 'Custom_SQL_Query'.SD_DOC_CURRENCY
    toColumn: 'TCURR'.'From currency'
    crossFilteringBehavior: oneDirection
```

✅ CORRECT:
```
relationship rel-currency-fcurr
    fromColumn: 'Custom_SQL_Query'.SD_DOC_CURRENCY
    toColumn: 'TCURR'.'From currency'
```


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — RULE M16: NO BLANK LINES BEFORE ANNOTATION/CHANGEDPROPERTY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### RULE-M16 | No Blank Lines Before annotation or changedProperty
─────────────────────────────────────────────────────────────
Error Message   : InvalidLineType or Unknown property
Severity        : FATAL — parser crash
Applies to      : ALL table .tmdl files

Root Cause:
───────────
TMDL uses indentation-based scope. A blank line between the last
property of an object and an `annotation` or `changedProperty` line
causes the parser to close the object's scope at the blank line,
then interpret the annotation as a new root-level statement — which
is invalid at that position.

Rule:
─────
`annotation` and `changedProperty` MUST be immediately contiguous
with the parent object's property block. No blank line before them.

❌ WRONG:
```
column 'Order Created on Date'
    dataType: dateTime
    lineageTag: csq-col-order-created-on-date
    summarizeBy: none
    sourceColumn: ORDER_CREATED_ON_DATE
    changedProperty = IsHidden

    annotation SummarizationSetBy = Automatic
    annotation UnderlyingDateTimeDataType = Date
```

✅ CORRECT:
```
column 'Order Created on Date'
    dataType: dateTime
    lineageTag: csq-col-order-created-on-date
    summarizeBy: none
    sourceColumn: ORDER_CREATED_ON_DATE
    changedProperty = IsHidden
    annotation SummarizationSetBy = Automatic
    annotation UnderlyingDateTimeDataType = Date
```

Correct Full Column Block Order:
─────────────────────────────────
```
column 'Column Name'
    dataType: <type>
    formatString: <format>
    displayOrdinal: <n>
    lineageTag: <tag>
    dataCategory: <cat>
    summarizeBy: <none/sum/etc>
    sourceColumn: SOURCE_COL
    isNameInferred
    isHidden
    variation Variation
        isDefault
        relationship: rel-name
        defaultHierarchy: LocalDateTable_<GUID>.'Date Hierarchy'
    changedProperty = IsHidden
    changedProperty = DataType
    sortByColumn: ColName
    annotation SummarizationSetBy = Automatic
    annotation UnderlyingDateTimeDataType = Date
    annotation PBI_FormatHint = {"isDateTimeCustom":true}
```

⚠️  KEY POINT: No blank lines anywhere inside the column block.
    Properties, changedProperty, and annotation lines are all
    contiguous with 4-space indentation.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — RULE M17: PARTITION ANNOTATIONS INSIDE PARTITION
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### RULE-M17 | Partition Annotations Inside Partition Block
─────────────────────────────────────────────────────────────
Error Message   : Semantics mismatch — annotations attached to wrong
                  parent (non-fatal but functionally incorrect)
Severity        : WARNING — non-fatal but functionally incorrect
Applies to      : ALL table .tmdl files with partitions

Root Cause:
───────────
`PBI_NavigationStepName` and `PBI_ResultType` are partition properties.
Placed at table root (outside partition), they become table-level
annotations, changing their semantics and potentially causing Power BI
Service to misidentify the table's query type.

Rule:
─────
`PBI_NavigationStepName` and `PBI_ResultType` MUST be indented inside
the partition block.

❌ WRONG — annotations at table root:
```
table 'My Table'
    lineageTag: tbl-my-table

    partition 'My Table' = m
        mode: import
        source = ```
                let ... in Result
                ```

    annotation PBI_NavigationStepName = Navigation
    annotation PBI_ResultType = Table
```

✅ CORRECT — annotations inside partition:
```
table 'My Table'
    lineageTag: tbl-my-table

    partition 'My Table' = m
        mode: import
        source = ```
                let ... in Result
                ```
        annotation PBI_NavigationStepName = Navigation
        annotation PBI_ResultType = Table
```

Indentation Reference:
──────────────────────
  - Table keyword: 0 spaces
  - Table properties: 4 spaces
  - Partition declaration: 4 spaces
  - Partition properties (mode, source): 8 spaces
  - Content inside source = ``` ```: 12 spaces
  - **Partition annotations: 8 spaces** (same level as mode and source)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — RETROFIT CHECKLIST (k1/k2 OUTPUTS)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Apply when re-generating or patching any k1/k2 output:

### relationships.tmdl (any workbook)
─────────────────────────────────────────────────────────────
  [ ] Replace all `///` with `//` (M13)
  [ ] Remove all `crossFilteringBehavior: oneDirection` (M15)


### Any table .tmdl
─────────────────────────────────────────────────────────────
  [ ] Wrap all multi-line DAX in ``` blocks (M12)
  [ ] Convert all [BRACKET] lineageTags → prefix-col-name (M11)
  [ ] Convert all Calculation_NNNNN → prefix-calc-name (M11)
  [ ] Convert all Action (X,Y,Z) → prefix-action-slug (M11)
  [ ] Remove blank lines before annotation/changedProperty (M16)
  [ ] Move partition annotations inside partition block (M17)


### SAP HANA partitions only
─────────────────────────────────────────────────────────────
  [ ] Change Source{[Name="..."]}[Data] → Source (M14)
  [ ] Change [], [EnableFolding...] → null, [EnableFolding...] (M14)
  [ ] Document gateway requirement in conversion report (M14)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — PRE-DELIVERY VALIDATION CHECKLIST
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Before delivering any TMDL file, verify every item:

### VALIDATE-01 | lineageTags
─────────────────────────────────────────────────────────────
  [ ] Every lineageTag uses only [a-zA-Z0-9_-]
  [ ] No [BRACKET] format
  [ ] No Calculation_NNNNN format
  [ ] No Action (X,Y,Z) format
  [ ] All lineageTags unique within file


### VALIDATE-02 | DAX Expressions
─────────────────────────────────────────────────────────────
  [ ] All multi-line DAX uses backtick blocks
  [ ] No bare VAR/RETURN outside backtick blocks
  [ ] Backtick body indented 8 spaces
  [ ] dataType/lineageTag appear after closing backtick


### VALIDATE-03 | Comments
─────────────────────────────────────────────────────────────
  [ ] No /// before relationship
  [ ] No /// before partition
  [ ] No /// before hierarchy level
  [ ] No /// before variation


### VALIDATE-04 | Relationships
─────────────────────────────────────────────────────────────
  [ ] No crossFilteringBehavior: oneDirection
  [ ] Date variations have joinOnDateBehavior: datePartOnly
  [ ] Relationship names match column variation declarations


### VALIDATE-05 | Annotations
─────────────────────────────────────────────────────────────
  [ ] No blank lines before annotation lines
  [ ] No blank lines before changedProperty lines
  [ ] PBI_NavigationStepName inside partition (not table root)
  [ ] PBI_ResultType inside partition (not table root)


### VALIDATE-06 | SAP HANA (if applicable)
─────────────────────────────────────────────────────────────
  [ ] Value.NativeQuery uses Source root
  [ ] Third parameter is null not []
  [ ] Gateway requirement documented


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 11 — CONVERSION COMPLETENESS SCORECARD
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Category                         | Items | Documented | % Complete |
|----------------------------------|-------|------------|------------|
| TMDL Compliance Rules (M11–M17)  | 7     | 7          | 100%       |
| Retrofit Checklist Items         | 10    | 10         | 100%       |
| Pre-Delivery Validation Checks   | 18    | 18         | 100%       |
| TOTAL                            | 35    | 35         | 🟢 100%    |

NOTE: This knowledge file is a rules/compliance reference — not a
      workbook conversion. Sections 1–8 correspond to the 7 compliance
      rules. Sections 9–10 provide operational checklists.
      There are no parameters, calculated columns, measures, or
      Power Query sources in this file.


END OF KNOWLEDGE FILE
