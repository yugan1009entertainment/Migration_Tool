\# K3 — TMDL Strict Compliance Rules

\## Bug-Fix Knowledge Base

\*\*Version:\*\* k3 | \*\*Last Updated:\*\* 2026-06-08

\*\*Derived from:\*\* Live Power BI Desktop May 2026 (v2.154.1260.0) validation errors during pair\_3 conversion

\*\*Scope:\*\* Rules M11–M17 — apply ALL automatically to every generated TMDL file before delivery



\---



\## OVERVIEW



7 rules discovered through iterative error correction when loading generated TMDL files into Power BI Desktop May 2026. Each rule has: the exact error message, root cause, and correct pattern.



\*\*These rules are non-negotiable. Apply all 7 to every file. Never skip.\*\*



\---



\## RULE M11 — lineageTag: Kebab-Case Only



\*\*Error:\*\*

```

InvalidLineType (silent parse failure)

```



\*\*Root cause:\*\* TMDL `lineageTag` values are parsed as identifiers. Special characters including `()`, `\[]`, spaces, commas, `!`, `/`, dots are invalid and crash the parser.



\*\*Rule:\*\* `lineageTag` values MUST match `\[a-zA-Z0-9\_-]+` only.



\### Naming Convention



| Object | Pattern | Example |

|---|---|---|

| Source column | `<prefix>-col-<name>` | `csq-col-order-created-on-date` |

| Calculated column | `<prefix>-calc-<name>` | `csq-calc-date-selection` |

| Measure | `<prefix>-meas-<name>` | `mt-meas-total-sales` |

| Action column | `<prefix>-action-<slug>` | `csq-action-week-year-month-soldto` |

| Hierarchy | `<prefix>-hier-<name>` | `csq-hier-sold-to-ship-to` |

| Hierarchy level n | `<prefix>-hier-lvl-<n>` | `csq-hier-lvl-1` |

| Table | `<prefix>-0001-0000-0000-<12digits>` | `csq-0001-0000-0000-000000000001` |

| Parameter column | `param-<n>-<type>` | `param-3-key` |



\*\*Table prefix codes:\*\* `csq`=Custom SQL Query, `mt`=Measures Table, `param`=Parameters, `ldt<n>`=LocalDateTable n, `hcp`=HCP table



\### Bad → Good



```

❌  lineageTag: \[ORDER\_CREATED\_ON\_DATE]

❌  lineageTag: Calculation\_1242149131112218624

❌  lineageTag: Action (Category,Submission Date)

❌  lineageTag: Action (!Reporting Week,YEAR(!Date Selection),MONTH(!Date Selection),SOLD\_TO\_NAME (group))



✅  lineageTag: csq-col-order-created-on-date

✅  lineageTag: csq-calc-date-selection

✅  lineageTag: csq-action-category-submission-date

✅  lineageTag: csq-action-week-year-month-soldto

```



\*\*Retrofit:\*\* All k1/k2 outputs using `\[BRACKET]`, `Calculation\_NNNNN`, or `Action (X,Y,Z)` lineageTags must be corrected before loading.



\---



\## RULE M12 — Multi-line DAX Must Use Triple-Backtick Blocks



\*\*Error:\*\*

```

TMDL Format Error:

&#x20;   Parsing error type - InvalidLineType

&#x20;   Detailed error - Unexpected line type: Empty!

&#x20;   Document - './tables/Custom\_SQL\_Query'

&#x20;   Line Number - 408

```



\*\*Root cause:\*\* Without backtick wrapping, the TMDL parser uses indentation to determine scope. Any blank line or VAR/RETURN block causes it to misinterpret property lines (e.g., `dataType:`) as DAX content, then crash on what it thinks is an empty line.



\*\*Rule:\*\* Any DAX expression spanning more than 1 line MUST be wrapped in triple-backtick blocks.



\*\*Always use backtick block if expression contains any of:\*\*

`VAR`, `RETURN`, `SWITCH(TRUE(),`, `SUMX(`, `AVERAGEX(`, `CALCULATETABLE(`, nested `IF(`, `FILTER(`



\### Pattern



\*\*❌ WRONG — bare multi-line DAX (CRASHES):\*\*

```tmdl

column '!Reporting Month' =

&#x20;   VAR \_ds = 'Table'\[Date]

&#x20;   VAR \_today = TODAY()

&#x20;   RETURN FORMAT(YEAR(\_ds), "0")

&#x20;   dataType: string

&#x20;   lineageTag: csq-calc-reporting-month

```



\*\*✅ CORRECT — triple-backtick wrapped:\*\*

```tmdl

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

```



\### Indentation Rules

\- Opening ` ``` ` immediately after `= ` on declaration line

\- DAX body: 8 spaces indent inside block

\- Closing ` ``` `: own line, same indent as body

\- `dataType:`, `lineageTag:` etc. resume at 4-space indent after closing ` ``` `



\---



\## RULE M13 — Triple-Slash Comments Only on Description-Supporting Objects



\*\*Error:\*\*

```

Property 'description' is unknown and is not expected in the situation it appears.

Stack trace: SingleColumnRelationship.ReadMetadataProperties

```



\*\*Root cause:\*\* `///` (triple-slash) lines are syntactic sugar for the `description` property. `SingleColumnRelationship` and several other objects do NOT have this property, so the parser throws "unknown property".



\*\*Rule:\*\* Use `///` ONLY on objects that support `description`. Use `//` for everything else.



\### Which Objects Support `///`



| Object | `///` Allowed? |

|---|---|

| `table` | ✅ Yes |

| `column` (source or calculated) | ✅ Yes |

| `measure` | ✅ Yes |

| `hierarchy` | ✅ Yes |

| `role` | ✅ Yes |

| \*\*`relationship`\*\* | ❌ NO |

| \*\*`partition`\*\* | ❌ NO |

| \*\*hierarchy `level`\*\* | ❌ NO |

| \*\*`variation`\*\* | ❌ NO |

| \*\*`annotation`\*\* | ❌ NO |

| \*\*`changedProperty`\*\* | ❌ NO |

| \*\*`roleMember`\*\* | ❌ NO |



\*\*Safe rule of thumb:\*\* `///` only on `table`, `column`, `measure`, `hierarchy`. `//` for everything else.



\### Bad → Good



\*\*❌ WRONG:\*\*

```tmdl

/// Currency join: Table.Col → TCURR.Col

relationship rel-currency-fcurr

&#x20;   fromColumn: 'Custom\_SQL\_Query'.SD\_DOC\_CURRENCY

&#x20;   toColumn: 'TCURR'.'From currency'

```



\*\*✅ CORRECT:\*\*

```tmdl

// Currency join: Table.Col → TCURR.Col

relationship rel-currency-fcurr

&#x20;   fromColumn: 'Custom\_SQL\_Query'.SD\_DOC\_CURRENCY

&#x20;   toColumn: 'TCURR'.'From currency'

```



\---



\## RULE M14 — SAP HANA: Root-Level Value.NativeQuery + null Params



\*\*Error:\*\*

```

Data load / connector failure during refresh

\[DataSource.Error] or gateway connection error

```



\*\*Root cause:\*\* Schema navigation (`Source{\[Name="\_SYS\_BIC"]}\[Data]`) before `Value.NativeQuery` is unreliable in HANA connector v2.0. The SQL already contains fully-qualified schema names, so schema navigation in M is redundant and error-prone.



\*\*Rule:\*\* Pass SOURCE ROOT directly to `Value.NativeQuery`. Use `null` (not `\[]`) as the third parameter.



\### Pattern



\*\*❌ WRONG:\*\*

```m

Result = Value.NativeQuery(Source{\[Name="\_SYS\_BIC"]}\[Data], Query, \[], \[EnableFolding=true])

```



\*\*✅ CORRECT:\*\*

```m

Result = Value.NativeQuery(Source, Query, null, \[EnableFolding=true])

```



\### HANA SQL Identifier Quoting

Inside the M string, HANA schema/table names with special chars use double-quote escaping:

\- `""\_SYS\_BIC"".""SAP\_ERP.TABLE\_CV\_S4/CV\_VBAK""`



\### Infrastructure Requirements

\- Requires \*\*Power BI On-Premises Data Gateway\*\*

\- Credentials: \*\*Basic (username/password)\*\*

\- Document gateway requirement in conversion report Prerequisites section



\---



\## RULE M15 — Never Emit crossFilteringBehavior: oneDirection



\*\*Error:\*\* Parse warning (non-fatal but noisy; may become fatal in future versions)



\*\*Root cause:\*\* `crossFilteringBehavior: oneDirection` is the default value. Emitting it explicitly is redundant.



\*\*Rule:\*\* Omit `crossFilteringBehavior: oneDirection` entirely. Only emit `crossFilteringBehavior: bothDirections` when explicitly required (RLS passthrough, many-to-many bridge).



\*\*❌ WRONG:\*\*

```tmdl

relationship rel-currency-fcurr

&#x20;   fromColumn: 'Custom\_SQL\_Query'.SD\_DOC\_CURRENCY

&#x20;   toColumn: 'TCURR'.'From currency'

&#x20;   crossFilteringBehavior: oneDirection

```



\*\*✅ CORRECT:\*\*

```tmdl

relationship rel-currency-fcurr

&#x20;   fromColumn: 'Custom\_SQL\_Query'.SD\_DOC\_CURRENCY

&#x20;   toColumn: 'TCURR'.'From currency'

```



\---



\## RULE M16 — No Blank Lines Before annotation or changedProperty



\*\*Error:\*\*

```

InvalidLineType or Unknown property

```



\*\*Root cause:\*\* TMDL uses indentation-based scope. A blank line between the last property of an object and an `annotation` or `changedProperty` line causes the parser to close the object's scope at the blank line, then interpret the annotation as a new root-level statement — which is invalid at that position.



\*\*Rule:\*\* `annotation` and `changedProperty` MUST be immediately contiguous with the parent object's property block. No blank line before them.



\*\*❌ WRONG:\*\*

```tmdl

column 'Order Created on Date'

&#x20;   dataType: dateTime

&#x20;   lineageTag: csq-col-order-created-on-date

&#x20;   summarizeBy: none

&#x20;   sourceColumn: ORDER\_CREATED\_ON\_DATE

&#x20;   changedProperty = IsHidden



&#x20;   annotation SummarizationSetBy = Automatic

&#x20;   annotation UnderlyingDateTimeDataType = Date

```



\*\*✅ CORRECT:\*\*

```tmdl

column 'Order Created on Date'

&#x20;   dataType: dateTime

&#x20;   lineageTag: csq-col-order-created-on-date

&#x20;   summarizeBy: none

&#x20;   sourceColumn: ORDER\_CREATED\_ON\_DATE

&#x20;   changedProperty = IsHidden

&#x20;   annotation SummarizationSetBy = Automatic

&#x20;   annotation UnderlyingDateTimeDataType = Date

```



\### Correct Full Column Block Order

```tmdl

column 'Column Name'

&#x20;   dataType: <type>

&#x20;   formatString: <format>

&#x20;   displayOrdinal: <n>

&#x20;   lineageTag: <tag>

&#x20;   dataCategory: <cat>

&#x20;   summarizeBy: <none/sum/etc>

&#x20;   sourceColumn: SOURCE\_COL

&#x20;   isNameInferred

&#x20;   isHidden

&#x20;   variation Variation

&#x20;       isDefault

&#x20;       relationship: rel-name

&#x20;       defaultHierarchy: LocalDateTable\_<GUID>.'Date Hierarchy'

&#x20;   changedProperty = IsHidden

&#x20;   changedProperty = DataType

&#x20;   sortByColumn: ColName

&#x20;   annotation SummarizationSetBy = Automatic

&#x20;   annotation UnderlyingDateTimeDataType = Date

&#x20;   annotation PBI\_FormatHint = {"isDateTimeCustom":true}

```



\---



\## RULE M17 — Partition Annotations Inside Partition Block



\*\*Error:\*\* Semantics mismatch — annotations attached to wrong parent (non-fatal but functionally incorrect)



\*\*Root cause:\*\* `PBI\_NavigationStepName` and `PBI\_ResultType` are partition properties. Placed at table root (outside partition), they become table-level annotations, changing their semantics and potentially causing Power BI Service to misidentify the table's query type.



\*\*Rule:\*\* `PBI\_NavigationStepName` and `PBI\_ResultType` MUST be indented inside the partition block.



\*\*❌ WRONG — annotations at table root:\*\*

```tmdl

table 'My Table'

&#x20;   lineageTag: tbl-my-table



&#x20;   partition 'My Table' = m

&#x20;       mode: import

&#x20;       source = ```

&#x20;               let ... in Result

&#x20;               ```



&#x20;   annotation PBI\_NavigationStepName = Navigation

&#x20;   annotation PBI\_ResultType = Table

```



\*\*✅ CORRECT — annotations inside partition:\*\*

```tmdl

table 'My Table'

&#x20;   lineageTag: tbl-my-table



&#x20;   partition 'My Table' = m

&#x20;       mode: import

&#x20;       source = ```

&#x20;               let ... in Result

&#x20;               ```

&#x20;       annotation PBI\_NavigationStepName = Navigation

&#x20;       annotation PBI\_ResultType = Table

```



\### Indentation Reference

\- Table keyword: 0 spaces

\- Table properties: 4 spaces

\- Partition declaration: 4 spaces

\- Partition properties (`mode`, `source`): 8 spaces

\- Content inside `source = ``` ``` `: 12 spaces

\- \*\*Partition annotations: 8 spaces\*\* (same level as `mode` and `source`)



\---



\## QUICK REFERENCE TABLE



| Rule | One-line Summary | Error if Violated |

|---|---|---|

| \*\*M11\*\* | `lineageTag` = `\[a-zA-Z0-9\_-]+` kebab-case only | `InvalidLineType` |

| \*\*M12\*\* | Multi-line DAX → wrap in ` ``` ` backtick blocks | `InvalidLineType: Unexpected line type: Empty!` |

| \*\*M13\*\* | `///` only on table/column/measure/hierarchy | `Property 'description' is unknown` |

| \*\*M14\*\* | SAP HANA: `Value.NativeQuery(Source, sql, null, \[...])` | Data load failure |

| \*\*M15\*\* | Omit `crossFilteringBehavior: oneDirection` (it's default) | Parse warning |

| \*\*M16\*\* | No blank lines before `annotation`/`changedProperty` | `InvalidLineType` |

| \*\*M17\*\* | `PBI\_NavigationStepName`/`PBI\_ResultType` inside partition | Semantics mismatch |



\---



\## RETROFIT CHECKLIST



Apply when re-generating or patching any k1/k2 output:



\### relationships.tmdl (any workbook)

\- \[ ] Replace all `///` with `//` (M13)

\- \[ ] Remove all `crossFilteringBehavior: oneDirection` (M15)



\### Any table .tmdl

\- \[ ] Wrap all multi-line DAX in ` ``` ` blocks (M12)

\- \[ ] Convert all `\[BRACKET]` lineageTags → `prefix-col-name` (M11)

\- \[ ] Convert all `Calculation\_NNNNN` → `prefix-calc-name` (M11)

\- \[ ] Convert all `Action (X,Y,Z)` → `prefix-action-slug` (M11)

\- \[ ] Remove blank lines before `annotation`/`changedProperty` (M16)

\- \[ ] Move partition annotations inside partition block (M17)



\### SAP HANA partitions only

\- \[ ] Change `Source{\[Name="..."]}\[Data]` → `Source` (M14)

\- \[ ] Change `\[], \[EnableFolding...]` → `null, \[EnableFolding...]` (M14)

\- \[ ] Document gateway requirement in conversion report (M14)



\---



\## PRE-DELIVERY VALIDATION CHECKLIST



Before delivering any TMDL file, verify every item:



```

lineageTags

\[ ] Every lineageTag uses only \[a-zA-Z0-9\_-]

\[ ] No \[BRACKET] format

\[ ] No Calculation\_NNNNN format

\[ ] No Action (X,Y,Z) format

\[ ] All lineageTags unique within file



DAX Expressions

\[ ] All multi-line DAX uses backtick blocks

\[ ] No bare VAR/RETURN outside backtick blocks

\[ ] Backtick body indented 8 spaces

\[ ] dataType/lineageTag appear after closing backtick



Comments

\[ ] No /// before relationship

\[ ] No /// before partition

\[ ] No /// before hierarchy level

\[ ] No /// before variation



Relationships

\[ ] No crossFilteringBehavior: oneDirection

\[ ] Date variations have joinOnDateBehavior: datePartOnly

\[ ] Relationship names match column variation declarations



Annotations

\[ ] No blank lines before annotation lines

\[ ] No blank lines before changedProperty lines

\[ ] PBI\_NavigationStepName inside partition (not table root)

\[ ] PBI\_ResultType inside partition (not table root)



SAP HANA (if applicable)

\[ ] Value.NativeQuery uses Source root

\[ ] Third parameter is null not \[]

\[ ] Gateway requirement documented

```



