# ES-004 · Tableau Function Engineering Specification
**Version:** 1.0  
**Purpose:** Agent knowledge dictionary for Tableau → Power BI migration  
**Scope:** All Tableau calculation functions — syntax, semantics, XML shape, parser rules, and migration impact

---

## How to Use This Document

This specification is consumed by the **Calculation Agent** during Tableau → Power BI migration. When the agent encounters any Tableau formula, it looks up each function here to understand:

1. **What the function does** (semantics, return type)
2. **How it nests** (dependency chain, complexity)
3. **How it appears in XML** (parser-level shape)
4. **How hard it is to migrate** (impact, Power BI equivalent)

### Example: Why This Matters

Given the formula:
```
WINDOW_SUM(LOOKUP(SUM([Sales]), -1))
```

**Without this document** → the agent sees plain text.

**With this document** → the agent resolves the full dependency chain:

```
WINDOW_SUM
  └─ Category: Table Calculation
  └─ Depends on: Partition / Addressing
     └─ LOOKUP
        └─ Category: Table Calculation (Offset)
        └─ Requires: Inner aggregate to resolve first
           └─ SUM
              └─ Category: Aggregate
              └─ Creates: Aggregate context
```

This lets the agent correctly translate the formula into DAX/Power Query with the right evaluation order.

---

## Function Template

Every function in this document follows this schema:

| Field | Description |
|---|---|
| **Syntax** | Exact call signature |
| **Arguments** | Parameter names and types |
| **Return Type** | Output data type |
| **Description** | Semantic meaning |
| **XML Shape** | How it appears inside `<calculation>` |
| **Parser Notes** | Rules the parser must follow |
| **Dependency** | What execution context it creates or requires |
| **Complexity** | Simple / Medium / Complex |
| **Migration Impact** | Low / Medium / High + Power BI equivalent |
| **Example** | Concrete usage |

---

## Categories Index

| # | Category | Functions |
|---|---|---|
| 1 | [Aggregate Functions](#1-aggregate-functions) | SUM, AVG, MIN, MAX, COUNT, COUNTD, MEDIAN, ATTR |
| 2 | [String Functions](#2-string-functions) | LEFT, RIGHT, MID, LEN, UPPER, LOWER, REPLACE, CONTAINS, STARTSWITH, ENDSWITH, SPLIT |
| 3 | [Date Functions](#3-date-functions) | DATEADD, DATEDIFF, DATEPART, DATENAME, MAKEDATE, TODAY, NOW |
| 4 | [Logical Functions](#4-logical-functions) | IF, IIF, CASE, AND, OR, NOT |
| 5 | [Number Functions](#5-number-functions) | ROUND, ABS, SQRT, POWER, INT, FLOAT, ZN |
| 6 | [Type Conversion](#6-type-conversion) | STR, INT, FLOAT, DATE, DATETIME |
| 7 | [LOD Expressions](#7-lod-expressions) | FIXED, INCLUDE, EXCLUDE |
| 8 | [Table Calculations](#8-table-calculations) | WINDOW_SUM, WINDOW_AVG, WINDOW_MIN, WINDOW_MAX, LOOKUP, INDEX, FIRST, LAST, SIZE, RANK, RANK_DENSE, RUNNING_SUM, RUNNING_AVG, PREVIOUS_VALUE |
| 9 | [Spatial Functions](#9-spatial-functions) | MAKEPOINT, MAKELINE, DISTANCE, AREA, BUFFER |
| 10 | [Pass-Through SQL](#10-pass-through-sql) | RAWSQL_INT, RAWSQL_REAL, RAWSQL_STR, RAWSQL_BOOL |

---

## 1. Aggregate Functions

Aggregate functions collapse multiple row values into a single value. They require a GROUP BY context (implicit in Tableau's view-level aggregation).

---

### SUM

**Syntax**
```
SUM(expression)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| expression | Number | The field or numeric expression to sum |

**Return Type:** Number

**Description**  
Returns the total sum of all non-null values in the expression across the aggregated group.

**XML Shape**
```xml
<calculation class="tableau" formula="SUM([Sales])">
  <arguments>
    <argument output="qNumeric" output-column="[Sales]"/>
  </arguments>
</calculation>
```

**Parser Notes**
- Preserve function name exactly (`SUM`, not `sum`)
- Preserve nesting — do not flatten `SUM(WINDOW_SUM(...))` chains
- Do not evaluate or pre-compute
- NULL values are excluded automatically

**Dependency:** Creates → `Aggregate` context  
**Complexity:** Simple  
**Migration Impact:** Low — maps directly to `SUM()` in DAX  
**Power BI Equivalent:** `SUM(Table[Column])` or `SUMX(Table, expression)`

**Example**
```
SUM([Sales])
SUM([Profit] / [Sales])
```

---

### AVG

**Syntax**
```
AVG(expression)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| expression | Number | Field to average |

**Return Type:** Number

**Description**  
Returns the arithmetic mean of all non-null values.

**XML Shape**
```xml
<calculation class="tableau" formula="AVG([Discount])"/>
```

**Parser Notes**
- NULLs are excluded from both numerator and denominator
- Do not substitute with `SUM / COUNT` — behavior differs on NULLs

**Dependency:** Creates → `Aggregate` context  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `AVERAGE(Table[Column])` or `AVERAGEX(Table, expression)`

**Example**
```
AVG([Discount])
```

---

### MIN

**Syntax**
```
MIN(expression)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| expression | Number \| Date \| String | Field to evaluate |

**Return Type:** Same as input type

**Description**  
Returns the minimum value. Works on numbers, dates, and strings (lexicographic for strings).

**Parser Notes**
- Can appear in both aggregate and non-aggregate contexts (row-level MIN of two fields: `MIN([A], [B])`)
- Disambiguate by arity: single-argument = aggregate; two-argument = row-level scalar

**Dependency:** Creates → `Aggregate` context (single-arg form)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `MIN(Table[Column])` or `MINX(Table, expression)`

**Example**
```
MIN([Order Date])
MIN([Price], [Cost])   -- row-level, returns smaller of two fields
```

---

### MAX

**Syntax**
```
MAX(expression)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| expression | Number \| Date \| String | Field to evaluate |

**Return Type:** Same as input type

**Description**  
Returns the maximum value. Mirrors `MIN` in all behaviors.

**Dependency:** Creates → `Aggregate` context (single-arg form)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `MAX(Table[Column])` or `MAXX(Table, expression)`

**Example**
```
MAX([Ship Date])
```

---

### COUNT

**Syntax**
```
COUNT(expression)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| expression | Any | Field to count |

**Return Type:** Integer

**Description**  
Returns the count of non-null values. Includes duplicates.

**Dependency:** Creates → `Aggregate` context  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `COUNT(Table[Column])` or `COUNTX(Table, expression)`

**Example**
```
COUNT([Order ID])
```

---

### COUNTD

**Syntax**
```
COUNTD(expression)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| expression | Any | Field to count distinctly |

**Return Type:** Integer

**Description**  
Returns the count of **distinct** non-null values. Equivalent to SQL `COUNT(DISTINCT ...)`.

**Parser Notes**
- Not available in all Tableau data source connections (e.g., some live connections)
- Flag if source is non-extract for compatibility check

**Dependency:** Creates → `Aggregate` context  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `DISTINCTCOUNT(Table[Column])`

**Example**
```
COUNTD([Customer ID])
```

---

### MEDIAN

**Syntax**
```
MEDIAN(expression)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| expression | Number | Numeric field |

**Return Type:** Number

**Description**  
Returns the median (50th percentile) value. Only available on extract or certain live connections.

**Parser Notes**
- Not supported in all databases — check connectivity before migration
- No direct SQL equivalent in many engines

**Dependency:** Creates → `Aggregate` context  
**Complexity:** Medium  
**Migration Impact:** Medium — DAX has no native `MEDIAN` over a measure; requires `PERCENTILE` workaround  
**Power BI Equivalent:** `MEDIAN(Table[Column])` (available in DAX since 2016)

**Example**
```
MEDIAN([Profit])
```

---

### ATTR

**Syntax**
```
ATTR(expression)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| expression | Any | Field expected to have a single value per group |

**Return Type:** Same as input

**Description**  
Returns the value of the expression if it is the **same for all rows** in the group, otherwise returns `*` (asterisk). Used to "promote" a dimension to a pseudo-aggregate.

**Parser Notes**
- `ATTR` is a special sentinel aggregate — not a mathematical function
- Output `*` signals ambiguity; preserve this behavior in migration
- Commonly used to pass dimension values into table calculations

**Dependency:** Creates → `Aggregate` context (sentinel)  
**Complexity:** Medium  
**Migration Impact:** High — no DAX equivalent; requires restructuring with `SELECTEDVALUE()` or `HASONEVALUE()` guard  
**Power BI Equivalent:** `IF(HASONEVALUE(Table[Column]), VALUES(Table[Column]), "*")`

**Example**
```
ATTR([Region])
```

---

## 2. String Functions

String functions operate on character data. They are row-level (non-aggregate) unless wrapped in an aggregate.

---

### LEFT

**Syntax**
```
LEFT(string, num_chars)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| string | String | Source string |
| num_chars | Integer | Number of characters from the left |

**Return Type:** String

**Description**  
Returns the leftmost `num_chars` characters from `string`.

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `LEFT(text, num_chars)`

**Example**
```
LEFT([Product Code], 3)
```

---

### RIGHT

**Syntax**
```
RIGHT(string, num_chars)
```

**Return Type:** String

**Description**  
Returns the rightmost `num_chars` characters from `string`.

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `RIGHT(text, num_chars)`

**Example**
```
RIGHT([Order ID], 4)
```

---

### MID

**Syntax**
```
MID(string, start, [length])
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| string | String | Source string |
| start | Integer | 1-based start position |
| length | Integer (optional) | Number of characters to return |

**Return Type:** String

**Description**  
Returns a substring beginning at `start`. If `length` is omitted, returns to end of string.

**Parser Notes**
- Tableau uses **1-based indexing** — flag if migrating to systems with 0-based indexing

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `MID(text, start, num_chars)` — note DAX requires `num_chars`

**Example**
```
MID([Product SKU], 4, 6)
```

---

### LEN

**Syntax**
```
LEN(string)
```

**Return Type:** Integer

**Description**  
Returns the number of characters in `string`. NULL returns NULL.

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `LEN(text)`

**Example**
```
LEN([Customer Name])
```

---

### UPPER

**Syntax**
```
UPPER(string)
```

**Return Type:** String

**Description**  
Converts all characters to uppercase.

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `UPPER(text)`

**Example**
```
UPPER([Country])
```

---

### LOWER

**Syntax**
```
LOWER(string)
```

**Return Type:** String

**Description**  
Converts all characters to lowercase.

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `LOWER(text)`

**Example**
```
LOWER([Email])
```

---

### REPLACE

**Syntax**
```
REPLACE(string, substring, replacement)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| string | String | Source string |
| substring | String | Pattern to find (literal, not regex) |
| replacement | String | Replacement text |

**Return Type:** String

**Description**  
Replaces all occurrences of `substring` in `string` with `replacement`. Case-sensitive.

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `SUBSTITUTE(text, old_text, new_text)`

**Example**
```
REPLACE([Product Name], "Old", "New")
```

---

### CONTAINS

**Syntax**
```
CONTAINS(string, substring)
```

**Return Type:** Boolean

**Description**  
Returns `TRUE` if `string` contains `substring`. Case-sensitive.

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `SEARCH(substring, string, 1, 0) > 0` (case-insensitive in DAX)

**Example**
```
CONTAINS([Notes], "urgent")
```

---

### STARTSWITH

**Syntax**
```
STARTSWITH(string, substring)
```

**Return Type:** Boolean

**Description**  
Returns `TRUE` if `string` begins with `substring`.

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `LEFT(string, LEN(substring)) = substring`

**Example**
```
STARTSWITH([Order ID], "CA-")
```

---

### ENDSWITH

**Syntax**
```
ENDSWITH(string, substring)
```

**Return Type:** Boolean

**Description**  
Returns `TRUE` if `string` ends with `substring`.

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `RIGHT(string, LEN(substring)) = substring`

**Example**
```
ENDSWITH([File Name], ".csv")
```

---

### SPLIT

**Syntax**
```
SPLIT(string, delimiter, token_number)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| string | String | Source string |
| delimiter | String | Character(s) to split on |
| token_number | Integer | 1-based position of token to return; negative counts from end |

**Return Type:** String

**Description**  
Splits `string` by `delimiter` and returns the token at `token_number`.

**Parser Notes**
- Positive integers count from left; negative from right
- If `token_number` exceeds available tokens, returns NULL

**Dependency:** None (row-level)  
**Complexity:** Medium  
**Migration Impact:** Medium — no single DAX function; requires `SUBSTITUTE` + `MID` pattern  
**Power BI Equivalent:** Custom DAX or Power Query `Text.Split`

**Example**
```
SPLIT([Full Name], " ", 1)   -- returns first name
SPLIT([Path], "/", -1)       -- returns last segment
```

---

## 3. Date Functions

Date functions manipulate or extract components of date/datetime values.

---

### DATEADD

**Syntax**
```
DATEADD(date_part, interval, date)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| date_part | String literal | Unit of time: `'day'`, `'month'`, `'year'`, `'hour'`, `'minute'`, `'second'`, `'week'`, `'quarter'` |
| interval | Integer | Number of units to add (negative subtracts) |
| date | Date \| Datetime | Starting date |

**Return Type:** Date or Datetime (matches input)

**Description**  
Adds `interval` units of `date_part` to `date`.

**Parser Notes**
- `date_part` must be a string literal — do not treat as a field reference
- Preserve quotes around date_part in migration

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `DATEADD(dates, number_of_intervals, interval)`

**Example**
```
DATEADD('month', 3, [Order Date])
DATEADD('day', -7, TODAY())
```

---

### DATEDIFF

**Syntax**
```
DATEDIFF(date_part, start_date, end_date, [start_of_week])
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| date_part | String literal | Unit for difference |
| start_date | Date | Earlier date |
| end_date | Date | Later date |
| start_of_week | String literal (optional) | `'monday'`, `'sunday'`, etc. |

**Return Type:** Integer

**Description**  
Returns the number of `date_part` boundaries crossed between `start_date` and `end_date`.

**Parser Notes**
- Result is signed — negative if `start_date` > `end_date`
- `start_of_week` only affects `'week'` date_part

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `DATEDIFF(start_date, end_date, interval)`

**Example**
```
DATEDIFF('day', [Order Date], [Ship Date])
```

---

### DATEPART

**Syntax**
```
DATEPART(date_part, date, [start_of_week])
```

**Return Type:** Integer

**Description**  
Extracts a numeric component from `date`. Returns an integer (e.g., month 3 for March).

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `MONTH()`, `YEAR()`, `DAY()`, `WEEKNUM()` etc. depending on date_part

**Example**
```
DATEPART('year', [Order Date])   -- returns 2024
DATEPART('month', [Order Date])  -- returns 1-12
```

---

### DATENAME

**Syntax**
```
DATENAME(date_part, date, [start_of_week])
```

**Return Type:** String

**Description**  
Returns a named string component from `date` (e.g., `"March"` instead of `3`).

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Medium — DAX uses `FORMAT()` function  
**Power BI Equivalent:** `FORMAT(date, "MMMM")` for month name, etc.

**Example**
```
DATENAME('month', [Order Date])   -- returns "March"
DATENAME('weekday', [Order Date]) -- returns "Tuesday"
```

---

### MAKEDATE

**Syntax**
```
MAKEDATE(year, month, day)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| year | Integer | Four-digit year |
| month | Integer | Month (1–12) |
| day | Integer | Day (1–31) |

**Return Type:** Date

**Description**  
Constructs a date value from year, month, and day components.

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `DATE(year, month, day)`

**Example**
```
MAKEDATE(2024, 1, 1)
```

---

### TODAY

**Syntax**
```
TODAY()
```

**Return Type:** Date

**Description**  
Returns the current date (no time component). Evaluated at render/query time.

**Parser Notes**
- No arguments
- Volatile function — result changes between renders

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `TODAY()`

**Example**
```
DATEDIFF('day', [Order Date], TODAY())
```

---

### NOW

**Syntax**
```
NOW()
```

**Return Type:** Datetime

**Description**  
Returns the current date and time. More precise than `TODAY()`.

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `NOW()`

**Example**
```
NOW()
```

---

## 4. Logical Functions

Logical functions control flow and conditional evaluation.

---

### IF

**Syntax**
```
IF condition THEN value
[ELSEIF condition THEN value]
...
[ELSE default_value]
END
```

**Return Type:** Any (must be consistent across branches)

**Description**  
Evaluates conditions in order and returns the value of the first `TRUE` branch. If no condition matches and no `ELSE` is present, returns NULL.

**Parser Notes**
- Multi-branch `IF` uses `ELSEIF`, not `ELSE IF`
- All return values should be the same type; mixed types cause implicit casting

**Dependency:** None (row-level)  
**Complexity:** Simple–Medium (depends on nesting depth)  
**Migration Impact:** Low  
**Power BI Equivalent:** `IF(condition, true_value, false_value)` or `SWITCH(TRUE(), ...)`

**Example**
```
IF [Sales] > 1000 THEN "High"
ELSEIF [Sales] > 500 THEN "Medium"
ELSE "Low"
END
```

---

### IIF

**Syntax**
```
IIF(condition, true_value, false_value, [unknown_value])
```

**Return Type:** Any

**Description**  
Inline ternary conditional. Returns `true_value` if condition is TRUE, `false_value` if FALSE, and `unknown_value` (or NULL) if condition is NULL/unknown.

**Parser Notes**
- 3-argument form: unknown maps to NULL
- 4-argument form: unknown returns `unknown_value`

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `IF(condition, true_value, false_value)`

**Example**
```
IIF([Profit] > 0, "Profitable", "Loss")
```

---

### CASE

**Syntax**
```
CASE expression
WHEN value1 THEN result1
WHEN value2 THEN result2
...
[ELSE default_result]
END
```

**Return Type:** Any (consistent across branches)

**Description**  
Compares `expression` against each `WHEN` value and returns the matching `result`. Useful for discrete value mapping.

**Parser Notes**
- `expression` is evaluated once; each `WHEN` is a literal match
- For range/conditional logic, use `IF` instead

**Dependency:** None (row-level)  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `SWITCH(expression, value1, result1, value2, result2, ..., default)`

**Example**
```
CASE [Region]
WHEN "East" THEN 1
WHEN "West" THEN 2
ELSE 0
END
```

---

### AND

**Syntax**
```
condition1 AND condition2
```

**Return Type:** Boolean

**Description**  
Returns TRUE only if both conditions are TRUE. Short-circuits on first FALSE.

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `condition1 && condition2` or `AND(condition1, condition2)`

**Example**
```
[Region] = "East" AND [Sales] > 1000
```

---

### OR

**Syntax**
```
condition1 OR condition2
```

**Return Type:** Boolean

**Description**  
Returns TRUE if either condition is TRUE.

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `condition1 || condition2` or `OR(condition1, condition2)`

**Example**
```
[Status] = "Pending" OR [Status] = "Open"
```

---

### NOT

**Syntax**
```
NOT condition
```

**Return Type:** Boolean

**Description**  
Negates the boolean condition.

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `NOT(condition)`

**Example**
```
NOT ISNULL([Customer ID])
```

---

## 5. Number Functions

Row-level numeric utility functions.

---

### ROUND

**Syntax**
```
ROUND(number, [decimal_places])
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| number | Number | Value to round |
| decimal_places | Integer (optional) | Digits after decimal; negative rounds left of decimal |

**Return Type:** Number

**Description**  
Rounds `number` to `decimal_places`. Uses "round half away from zero."

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `ROUND(number, num_digits)`

**Example**
```
ROUND([Sales], 2)
ROUND([Revenue], -3)   -- rounds to nearest thousand
```

---

### ABS

**Syntax**
```
ABS(number)
```

**Return Type:** Number

**Description**  
Returns the absolute (non-negative) value of `number`.

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `ABS(number)`

**Example**
```
ABS([Variance])
```

---

### SQRT

**Syntax**
```
SQRT(number)
```

**Return Type:** Number

**Description**  
Returns the square root of `number`. Returns NULL for negative inputs.

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `SQRT(number)`

**Example**
```
SQRT([Area])
```

---

### POWER

**Syntax**
```
POWER(base, exponent)
```

**Return Type:** Number

**Description**  
Returns `base` raised to the `exponent` power.

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `POWER(base, exponent)`

**Example**
```
POWER([Radius], 2)
```

---

### INT

**Syntax**
```
INT(expression)
```

**Return Type:** Integer

**Description**  
Truncates `expression` toward zero (not rounds). Also coerces booleans: TRUE → 1, FALSE → 0.

**Parser Notes**
- Different from `ROUND(x, 0)` — truncates, does not round
- Also used as type conversion (see Type Conversion section)

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `TRUNC(number)` or `INT(number)`

**Example**
```
INT(3.9)   -- returns 3
INT(-3.9)  -- returns -3
```

---

### FLOAT

**Syntax**
```
FLOAT(expression)
```

**Return Type:** Float (decimal number)

**Description**  
Converts `expression` to a floating-point number. Also used as type conversion.

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `VALUE(text)` or implicit coercion in DAX

**Example**
```
FLOAT("3.14")
```

---

### ZN

**Syntax**
```
ZN(expression)
```

**Return Type:** Number

**Description**  
Returns `expression` if it is not NULL; returns `0` if it is NULL. Short for "Zero if Null."

**Parser Notes**
- Commonly used before aggregations to avoid NULL propagation
- Does NOT affect string or date NULLs — numeric only

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `IF(ISBLANK(expression), 0, expression)` or `expression + 0`

**Example**
```
ZN([Discount])
SUM(ZN([Sales]))
```

---

## 6. Type Conversion

These functions coerce values from one data type to another.

---

### STR

**Syntax**
```
STR(expression)
```

**Return Type:** String

**Description**  
Converts `expression` to a string. Numbers are formatted with default decimal representation.

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `FORMAT(value, "General")` or `TEXT(value, format)`

**Example**
```
STR([Year])
```

---

### INT *(Type Conversion)*

**Syntax**
```
INT(expression)
```

**Return Type:** Integer

**Description**  
When used with a string or boolean, coerces to integer. See also: [Number Functions → INT](#int).

**Example**
```
INT("42")
INT(TRUE)   -- returns 1
```

---

### FLOAT *(Type Conversion)*

**Syntax**
```
FLOAT(expression)
```

**Return Type:** Float

**Description**  
Coerces string or integer to floating-point. See also: [Number Functions → FLOAT](#float).

**Example**
```
FLOAT("3.14")
```

---

### DATE

**Syntax**
```
DATE(expression)
```

**Return Type:** Date

**Description**  
Converts a string, number, or datetime to a date (strips time component).

**Parser Notes**
- String format must be recognizable (ISO 8601 preferred)
- Datetime → Date truncates time

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `DATEVALUE(text)` or `DATE(year, month, day)`

**Example**
```
DATE("2024-01-15")
DATE([Order Datetime])
```

---

### DATETIME

**Syntax**
```
DATETIME(expression)
```

**Return Type:** Datetime

**Description**  
Converts a string or date to a datetime. If input is a date, time is set to midnight.

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** Low  
**Power BI Equivalent:** `DATEVALUE()` + time component or Power Query `DateTime.FromText`

**Example**
```
DATETIME("2024-01-15 08:30:00")
```

---

## 7. LOD Expressions

Level of Detail (LOD) expressions fix the granularity at which an aggregate is computed, independently of the view's dimensions.

> ⚠️ LOD expressions are among the most complex structures to migrate. They frequently require complete restructuring in DAX using `CALCULATE`, `ALLEXCEPT`, or `SUMMARIZE`.

---

### FIXED

**Syntax**
```
{ FIXED [dim1], [dim2], ... : aggregate_expression }
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| dimensions | Field list | Dimensions that define the grouping |
| aggregate_expression | Aggregate | Calculation to perform at that grain |

**Return Type:** Same as aggregate_expression

**Description**  
Computes the aggregate at the specified dimension level, **ignoring the view's dimensions entirely**. The result is then joined back to the view.

**XML Shape**
```xml
<calculation class="tableau" formula="{ FIXED [Customer ID] : SUM([Sales]) }">
  <lod type="fixed">
    <groupby>[Customer ID]</groupby>
    <aggregate>SUM([Sales])</aggregate>
  </lod>
</calculation>
```

**Parser Notes**
- `FIXED` with empty dimension list (`{ FIXED : ... }`) computes at the grand total level
- Executes before dimension filters (but after context filters)
- Creates a sub-query join in the generated SQL

**Dependency:** Creates → `LOD/FIXED` context; bypasses view filters  
**Complexity:** Complex  
**Migration Impact:** High  
**Power BI Equivalent:** `CALCULATE(SUM(Table[Sales]), ALLEXCEPT(Table, Table[CustomerID]))`

**Example**
```
{ FIXED [Customer ID] : SUM([Sales]) }
{ FIXED [Region], [Year] : AVG([Profit]) }
{ FIXED : MAX([Order Date]) }   -- grand total max
```

---

### INCLUDE

**Syntax**
```
{ INCLUDE [dim1], [dim2], ... : aggregate_expression }
```

**Return Type:** Same as aggregate_expression

**Description**  
Computes the aggregate using **the view's dimensions PLUS** the specified additional dimensions. Produces finer granularity than the view.

**Parser Notes**
- Executes **at** or **below** the view level of detail
- Useful for calculating "per customer" metrics on a regional view

**Dependency:** Creates → `LOD/INCLUDE` context  
**Complexity:** Complex  
**Migration Impact:** High  
**Power BI Equivalent:** Requires `SUMMARIZE` + `CALCULATE` pattern

**Example**
```
{ INCLUDE [Order ID] : SUM([Sales]) }
```

---

### EXCLUDE

**Syntax**
```
{ EXCLUDE [dim1], [dim2], ... : aggregate_expression }
```

**Return Type:** Same as aggregate_expression

**Description**  
Computes the aggregate using **the view's dimensions MINUS** the specified dimensions. Produces coarser granularity — useful for "% of total" patterns.

**Parser Notes**
- Cannot exclude dimensions not in the view
- Common use: `SUM([Sales]) / { EXCLUDE [Sub-Category] : SUM([Sales]) }` for % of category total

**Dependency:** Creates → `LOD/EXCLUDE` context  
**Complexity:** Complex  
**Migration Impact:** High  
**Power BI Equivalent:** `CALCULATE(SUM(Table[Sales]), REMOVEFILTERS(Table[SubCategory]))`

**Example**
```
{ EXCLUDE [Sub-Category] : SUM([Sales]) }
```

---

## 8. Table Calculations

Table calculations are applied **after** the aggregate query completes. They operate on the result set partition and are evaluated by the Tableau engine, not pushed to the database.

> ⚠️ Table calculations are stateful — they depend on partition, addressing, and sort order configured in the view. Always capture partition/addressing metadata alongside the formula.

---

### WINDOW_SUM

**Syntax**
```
WINDOW_SUM(expression, [start, end])
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| expression | Aggregate expression | Value to sum |
| start | Integer or keyword (optional) | Start offset relative to current row; `FIRST` = start of partition |
| end | Integer or keyword (optional) | End offset; `LAST` = end of partition |

**Return Type:** Number

**Description**  
Returns the sum of `expression` over a window of rows within the current partition. Default window is the entire partition.

**Parser Notes**
- Operates on post-aggregate data
- Partition and addressing must be preserved as metadata
- `FIRST`, `LAST`, `0` are valid as start/end

**Dependency:** Requires → `Aggregate` inner expression; creates → `Table Calculation/Partition` context  
**Complexity:** Complex  
**Migration Impact:** High  
**Power BI Equivalent:** `CALCULATE(SUM(...), ALLSELECTED(...))` or running total pattern

**Example**
```
WINDOW_SUM(SUM([Sales]))
WINDOW_SUM(SUM([Sales]), FIRST(), 0)   -- running total
```

---

### WINDOW_AVG

**Syntax**
```
WINDOW_AVG(expression, [start, end])
```

**Return Type:** Number

**Description**  
Returns the average of `expression` over the window.

**Dependency:** Requires → `Aggregate`; creates → `Table Calculation/Partition`  
**Complexity:** Complex  
**Migration Impact:** High  
**Power BI Equivalent:** `AVERAGEX` over filtered table

**Example**
```
WINDOW_AVG(SUM([Sales]))
```

---

### WINDOW_MIN

**Syntax**
```
WINDOW_MIN(expression, [start, end])
```

**Return Type:** Number or Date

**Description**  
Returns the minimum value of `expression` over the window.

**Dependency:** Requires → `Aggregate`; creates → `Table Calculation/Partition`  
**Complexity:** Complex  
**Migration Impact:** High

**Example**
```
WINDOW_MIN(SUM([Profit]))
```

---

### WINDOW_MAX

**Syntax**
```
WINDOW_MAX(expression, [start, end])
```

**Return Type:** Number or Date

**Description**  
Returns the maximum value of `expression` over the window.

**Dependency:** Requires → `Aggregate`; creates → `Table Calculation/Partition`  
**Complexity:** Complex  
**Migration Impact:** High

**Example**
```
WINDOW_MAX(SUM([Sales]))
```

---

### LOOKUP

**Syntax**
```
LOOKUP(expression, offset)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| expression | Aggregate | Value to retrieve |
| offset | Integer | Relative row offset within partition (negative = previous rows) |

**Return Type:** Same as expression

**Description**  
Returns the value of `expression` at a row `offset` positions from the current row. Returns NULL if out of bounds.

**Parser Notes**
- `LOOKUP(expr, 0)` = current row (identity)
- `LOOKUP(expr, -1)` = previous row
- `LOOKUP(expr, FIRST())` = first row of partition
- Commonly nested inside `WINDOW_SUM`

**Dependency:** Requires → `Aggregate` inner expression; creates → `Table Calculation/Offset` context  
**Complexity:** Complex  
**Migration Impact:** High  
**Power BI Equivalent:** `CALCULATE(..., DATEADD(...))` for time-based; no generic equivalent

**Example**
```
LOOKUP(SUM([Sales]), -1)         -- prior period
LOOKUP(SUM([Sales]), FIRST())    -- first in partition
```

---

### INDEX

**Syntax**
```
INDEX()
```

**Return Type:** Integer

**Description**  
Returns the 1-based row index of the current row within the partition.

**Dependency:** Creates → `Table Calculation/Index` context  
**Complexity:** Simple  
**Migration Impact:** Medium  
**Power BI Equivalent:** `RANKX` or row context index in DAX

**Example**
```
INDEX()
IF INDEX() = 1 THEN "First" ELSE "Other" END
```

---

### FIRST

**Syntax**
```
FIRST()
```

**Return Type:** Integer

**Description**  
Returns the offset from the current row to the first row in the partition. Always ≤ 0.

**Dependency:** Creates → `Table Calculation/Offset` context  
**Complexity:** Simple  
**Migration Impact:** Medium

**Example**
```
FIRST()
LOOKUP(SUM([Sales]), FIRST())
```

---

### LAST

**Syntax**
```
LAST()
```

**Return Type:** Integer

**Description**  
Returns the offset from the current row to the last row in the partition. Always ≥ 0.

**Dependency:** Creates → `Table Calculation/Offset` context  
**Complexity:** Simple  
**Migration Impact:** Medium

**Example**
```
LAST() = 0   -- TRUE if on last row
```

---

### SIZE

**Syntax**
```
SIZE()
```

**Return Type:** Integer

**Description**  
Returns the total number of rows in the current partition.

**Dependency:** Creates → `Table Calculation/Partition` context  
**Complexity:** Simple  
**Migration Impact:** Medium

**Example**
```
SIZE()
SUM([Sales]) / WINDOW_SUM(SUM([Sales])) * 100   -- % of total (alternative pattern)
```

---

### RANK

**Syntax**
```
RANK(expression, [order])
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| expression | Aggregate | Value to rank |
| order | String literal (optional) | `'asc'` or `'desc'` (default: `'desc'`) |

**Return Type:** Integer

**Description**  
Returns the competition rank (gaps for ties). Tied values receive the same rank; next rank skips accordingly.

**Parser Notes**
- Default order is descending (highest = rank 1)
- Compare with `RANK_DENSE` which does not skip ranks

**Dependency:** Requires → `Aggregate`; creates → `Table Calculation/Rank`  
**Complexity:** Medium  
**Migration Impact:** Medium  
**Power BI Equivalent:** `RANKX(Table, expression, , DESC, Skip)`

**Example**
```
RANK(SUM([Sales]))
RANK(SUM([Sales]), 'asc')
```

---

### RANK_DENSE

**Syntax**
```
RANK_DENSE(expression, [order])
```

**Return Type:** Integer

**Description**  
Returns the dense rank (no gaps for ties). Tied values receive the same rank; the next distinct value gets the next consecutive rank.

**Dependency:** Requires → `Aggregate`; creates → `Table Calculation/Rank`  
**Complexity:** Medium  
**Migration Impact:** Medium  
**Power BI Equivalent:** `RANKX(Table, expression, , DESC, Dense)`

**Example**
```
RANK_DENSE(SUM([Profit]))
```

---

### RUNNING_SUM

**Syntax**
```
RUNNING_SUM(expression)
```

**Return Type:** Number

**Description**  
Returns the running (cumulative) sum of `expression` from the first row of the partition to the current row. Equivalent to `WINDOW_SUM(expr, FIRST(), 0)`.

**Dependency:** Requires → `Aggregate`; creates → `Table Calculation/Running`  
**Complexity:** Medium  
**Migration Impact:** Medium  
**Power BI Equivalent:** `CALCULATE(SUM(...), FILTER(ALLSELECTED(...), [Date] <= MAX([Date])))`

**Example**
```
RUNNING_SUM(SUM([Sales]))
```

---

### RUNNING_AVG

**Syntax**
```
RUNNING_AVG(expression)
```

**Return Type:** Number

**Description**  
Returns the running (cumulative) average from the first row to the current row.

**Dependency:** Requires → `Aggregate`; creates → `Table Calculation/Running`  
**Complexity:** Medium  
**Migration Impact:** Medium

**Example**
```
RUNNING_AVG(SUM([Profit]))
```

---

### PREVIOUS_VALUE

**Syntax**
```
PREVIOUS_VALUE(default)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| default | Any | Value returned on the first row (where there is no previous row) |

**Return Type:** Same as default

**Description**  
Returns the value of the current table calculation from the previous row. Enables recursive carry-forward patterns.

**Parser Notes**
- Self-referential — refers to the result of the same calc on the prior row
- Requires careful handling; can produce circular dependency errors if misused

**Dependency:** Creates → `Table Calculation/Self-Reference` context  
**Complexity:** Complex  
**Migration Impact:** High — no direct DAX equivalent; requires iterative measure

**Example**
```
PREVIOUS_VALUE(0)
[Running Total] = SUM([Sales]) + PREVIOUS_VALUE(0)
```

---

## 9. Spatial Functions

Spatial functions create or manipulate geographic geometry objects.

> ⚠️ Spatial functions have no equivalent in Power BI's DAX. Migration requires a GIS layer or custom visual.

---

### MAKEPOINT

**Syntax**
```
MAKEPOINT(latitude, longitude)
```

**Return Type:** Geometry (Point)

**Description**  
Creates a point geometry from latitude and longitude decimal values.

**Dependency:** None  
**Complexity:** Simple  
**Migration Impact:** High — requires custom visual or Azure Maps integration  
**Power BI Equivalent:** No native equivalent; use Azure Maps visual with lat/long columns

**Example**
```
MAKEPOINT([Latitude], [Longitude])
```

---

### MAKELINE

**Syntax**
```
MAKELINE(point1, point2)
```

**Return Type:** Geometry (Line)

**Description**  
Creates a line geometry connecting two point geometries.

**Migration Impact:** High  
**Power BI Equivalent:** No native equivalent

**Example**
```
MAKELINE(MAKEPOINT([Origin Lat], [Origin Long]), MAKEPOINT([Dest Lat], [Dest Long]))
```

---

### DISTANCE

**Syntax**
```
DISTANCE(point1, point2, unit)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| point1 | Geometry | Start point |
| point2 | Geometry | End point |
| unit | String literal | `'km'` or `'mi'` |

**Return Type:** Number

**Description**  
Returns the geodesic distance between two points in the specified unit.

**Migration Impact:** High  
**Power BI Equivalent:** Custom DAX Haversine formula

**Example**
```
DISTANCE(MAKEPOINT([Lat1],[Lon1]), MAKEPOINT([Lat2],[Lon2]), 'km')
```

---

### AREA

**Syntax**
```
AREA(geometry, unit)
```

**Return Type:** Number

**Description**  
Returns the area of a polygon geometry in the specified unit (`'km'` squared, `'mi'` squared, etc.).

**Migration Impact:** High  
**Power BI Equivalent:** No native equivalent

---

### BUFFER

**Syntax**
```
BUFFER(point, distance, unit)
```

**Return Type:** Geometry (Polygon)

**Description**  
Creates a circular polygon around `point` with the given `distance` radius.

**Migration Impact:** High  
**Power BI Equivalent:** No native equivalent

**Example**
```
BUFFER(MAKEPOINT([Lat],[Long]), 10, 'km')
```

---

## 10. Pass-Through SQL

Pass-through SQL functions inject raw SQL expressions into the query, bypassing Tableau's formula engine. They are database-specific and non-portable.

> ⚠️ These functions are the hardest to migrate — they contain raw SQL that must be re-evaluated for the target database and then re-expressed in DAX or Power Query.

---

### RAWSQL_INT

**Syntax**
```
RAWSQL_INT("sql_expression", [arg1], [arg2], ...)
```

**Arguments**

| Argument | Type | Description |
|---|---|---|
| sql_expression | String literal | Raw SQL with `%1`, `%2`, ... placeholders for args |
| arg1, arg2 | Fields | Substituted into placeholders |

**Return Type:** Integer

**Description**  
Executes `sql_expression` directly in the database and returns an integer result.

**Parser Notes**
- `%1`, `%2` etc. are positional placeholders for the subsequent arguments
- Database-specific SQL — must flag the source database during migration
- Cannot be parsed as Tableau formula — treat as opaque SQL blob

**Dependency:** Creates → `Pass-Through/Database` context  
**Complexity:** Complex  
**Migration Impact:** High  
**Migration Action:** Extract the SQL string → rewrite for Power BI's source database → express in Power Query M or DirectQuery custom SQL

**Example**
```
RAWSQL_INT("DATEDIFF(day, %1, %2)", [Start Date], [End Date])
```

---

### RAWSQL_REAL

**Syntax**
```
RAWSQL_REAL("sql_expression", [arg1], [arg2], ...)
```

**Return Type:** Float

**Description**  
Same as `RAWSQL_INT` but returns a floating-point number.

**Migration Impact:** High

**Example**
```
RAWSQL_REAL("LOG(%1)", [Value])
```

---

### RAWSQL_STR

**Syntax**
```
RAWSQL_STR("sql_expression", [arg1], [arg2], ...)
```

**Return Type:** String

**Description**  
Same pattern; returns a string.

**Migration Impact:** High

**Example**
```
RAWSQL_STR("FORMAT(%1, 'YYYY-MM')", [Order Date])
```

---

### RAWSQL_BOOL

**Syntax**
```
RAWSQL_BOOL("sql_expression", [arg1], [arg2], ...)
```

**Return Type:** Boolean

**Description**  
Same pattern; returns a boolean (TRUE/FALSE).

**Migration Impact:** High

**Example**
```
RAWSQL_BOOL("%1 LIKE '%urgent%'", [Notes])
```

---

## Appendix A: Migration Impact Summary

| Impact Level | Meaning | Typical Functions |
|---|---|---|
| **Low** | Direct 1-to-1 DAX equivalent | SUM, AVG, LEFT, IF, ROUND, TODAY |
| **Medium** | Equivalent exists but syntax differs | SPLIT, MEDIAN, RANK, RUNNING_SUM |
| **High** | Requires restructuring or has no direct equivalent | ATTR, LOD expressions, Table Calculations, Spatial, RAWSQL_* |

---

## Appendix B: Execution Order Reference

Understanding when each category evaluates helps the agent determine migration complexity:

```
1. Extract Filters
2. Data Source Filters
3. Context Filters
4. ─────────────────────────────────
5. FIXED LOD expressions         ← before dimension filters
6. ─────────────────────────────────
7. Dimension Filters
8. ─────────────────────────────────
9. INCLUDE / EXCLUDE LOD         ← after dimension filters
10. Aggregate Query (SUM, AVG, etc.)
11. ─────────────────────────────────
12. Measure Filters
13. ─────────────────────────────────
14. Table Calculations            ← last, post-query
    (WINDOW_*, LOOKUP, RANK, etc.)
```

---

## Appendix C: Dependency Chain Notation

The agent should resolve formulas into a dependency chain before translating:

```
WINDOW_SUM(LOOKUP(SUM([Sales]), -1))

→ WINDOW_SUM
    type: Table Calculation
    category: Window Aggregate
    requires: inner aggregate resolved first
    depends-on: Partition + Addressing

  → LOOKUP
      type: Table Calculation
      category: Offset
      requires: inner aggregate resolved first

    → SUM
        type: Aggregate
        category: Standard Aggregate
        creates: Aggregate context
        maps-to: SUM([Sales]) in DAX
```

Resolution order (innermost first):
1. `SUM([Sales])` → aggregate
2. `LOOKUP(..., -1)` → offset into prior row
3. `WINDOW_SUM(...)` → sum over entire partition

---

*End of ES-004 · Tableau Function Engineering Specification v1.0*
