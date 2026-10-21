# Tableau Datasource Reference v1.0

---

## 1. Purpose

A Tableau Datasource is the definition of how a workbook connects to, retrieves, and models data. It describes the physical connection to the source system, the logical structure of tables and relationships, all field definitions, calculated fields, filters, and metadata.

Every worksheet, dashboard, and story in a Tableau workbook draws its data from one or more datasources. The datasource is the single most critical structure in a Tableau workbook — if it is misread, every downstream element (measures, relationships, visuals, filters) will be incorrect.

### Datasource Variants

**Embedded Datasource** — Defined entirely inside the `.twb` file. The connection details, schema, fields, and data are all self-contained within the workbook. Most workbooks use embedded datasources.

**Published Datasource** — Defined externally on Tableau Server or Tableau Cloud. The workbook references it by name. The full connection definition is not present in the `.twb` — only a reference. Published datasources appear as `<datasource>` elements with a `repository-location` child and no inline connection block.

**Parameters Datasource** — A special reserved datasource named exactly `Parameters`. It is not a data connection. It holds all workbook-level parameters as `<column>` elements. It must be identified and excluded from all connection, relationship, and table extraction logic.

**Federated Datasource** — A datasource that combines multiple physical connections into one logical datasource. Common when joining tables across databases or blending cloud and on-premise sources.

**File-Based Datasource** — Connects to flat files: Excel, CSV, Google Sheets, JSON, spatial files. The connection `class` attribute identifies these (e.g., `excel-direct`, `textscan`).

**Database Datasource** — Connects to a relational or cloud database. The most common type in enterprise workbooks.

---

## 2. Datasource XML

Each datasource is defined as a `<datasource>` element inside the root `<datasources>` block.

```xml
<datasource
    name="federated.1abc2345def6"
    caption="Sales Performance"
    inline="true">

  <named-connections>...</named-connections>
  <relation .../>
  <metadata-records>...</metadata-records>
  <column .../>
  <filter .../>

</datasource>
```

### Attribute Reference

| Attribute | Description |
|---|---|
| `name` | Internal identifier. System-generated for embedded datasources (e.g., `federated.xxxxxxxx`). For the Parameters datasource, this is always the literal string `Parameters`. |
| `caption` | Display name shown in the Tableau UI. This is the human-readable datasource label. |
| `inline` | `"true"` when the datasource is fully defined inside the workbook. Absent or `"false"` for published datasources. |
| `source-platform` | Platform of the original datasource file when migrated (e.g., `win`, `mac`). |
| `hasconnection` | `"false"` indicates a datasource with no live or extract connection — typically the Parameters datasource or a scaffold datasource. |

### Published Datasource Indicator

A published datasource includes a `<repository-location>` child element:

```xml
<datasource name="Sales" caption="Sales">
  <repository-location derived="Sales" path="/..." site="Default" />
</datasource>
```

When `<repository-location>` is present, the full connection definition is not available in the workbook. Record the reference name and flag as published.

---

## 3. Datasource Types

| Type | Identifier in XML | Description |
|---|---|---|
| Embedded | `inline="true"`, full `<named-connections>` block present | Complete definition inside the workbook |
| Published | `<repository-location>` child present | External reference to Tableau Server/Cloud |
| Parameters | `name="Parameters"` | Special reserved datasource; not a data connection |
| Federated | Multiple `<named-connection>` entries | Multiple physical connections in one datasource |
| File-Based | `class` is `excel-direct`, `textscan`, `csv`, `google-sheets` | Connects to flat files |
| Database | `class` is `sqlserver`, `oracle`, `mysql`, `postgres`, etc. | Connects to a relational or cloud database |
| Extract | `<extract>` child element present | Backed by a `.hyper` or `.tde` extract file |

---

## 4. Named Connections

Named connections map a logical connection name to a physical connection definition.

```xml
<named-connections>
  <named-connection
      caption="Sales DB"
      name="sqlserver.prod.sales">
    <connection
        class="sqlserver"
        dbname="SalesDB"
        server="prod-sql-server.corp.com"
        username="svc_tableau"
        authentication="sspi"
        port="1433" />
  </named-connection>
</named-connections>
```

### Attribute Reference

#### On `<named-connection>`

| Attribute | Description |
|---|---|
| `caption` | Display name of the connection |
| `name` | Internal connection identifier — typically `class.server.database` pattern |

#### On `<connection>`

| Attribute | Description |
|---|---|
| `class` | Connection driver type — identifies the source system (see Section 5) |
| `server` | Hostname or IP address of the server |
| `dbname` | Database or catalog name |
| `schema` | Schema name |
| `username` | Authentication username |
| `authentication` | Auth method: `sspi`, `username-password`, `oauth`, `no-authentication` |
| `port` | Port number |
| `sslmode` | SSL mode: `require`, `verify-ca`, `disable` |
| `warehouse` | Cloud data warehouse name (Snowflake, BigQuery) |
| `project` | Project identifier (BigQuery) |
| `dataset` | Dataset name (BigQuery) |

---

## 5. Connection Types

The `class` attribute on `<connection>` identifies the source system. The parser must recognize all of the following.

### Relational Databases

| `class` value | Source System |
|---|---|
| `sqlserver` | Microsoft SQL Server |
| `oracle` | Oracle Database |
| `mysql` | MySQL |
| `postgres` | PostgreSQL |
| `db2` | IBM Db2 |
| `sybase` | SAP Sybase |
| `teradata` | Teradata |
| `netezza` | IBM Netezza |
| `hive` | Apache Hive |
| `impala` | Apache Impala |
| `spark` | Apache Spark SQL |
| `presto` | Presto / Trino |
| `greenplum` | Greenplum |
| `exasol` | Exasol |

### Cloud Data Warehouses

| `class` value | Source System |
|---|---|
| `snowflake` | Snowflake |
| `redshift` | Amazon Redshift |
| `bigquery` | Google BigQuery |
| `athena` | Amazon Athena |
| `synapse` | Azure Synapse Analytics |
| `databricks` | Databricks |

### File-Based Sources

| `class` value | Source System |
|---|---|
| `excel-direct` | Microsoft Excel |
| `textscan` | CSV / Text file |
| `google-sheets` | Google Sheets |
| `json` | JSON file |
| `spatial-file` | Spatial / GIS file |
| `hyper` | Tableau Extract (`.hyper`) |
| `tde` | Tableau Data Extract (legacy `.tde`) |

### Other Sources

| `class` value | Source System |
|---|---|
| `salesforce` | Salesforce |
| `serviceNow` | ServiceNow |
| `marketo` | Marketo |
| `odata` | OData API |
| `webdata-connector` | Web Data Connector |
| `genericodbc` | Generic ODBC |
| `genericjdbc` | Generic JDBC |

---

## 6. Live vs Extract

### Live Connection

The datasource queries the source system in real time. No extract file is present.

**XML indicator:** No `<extract>` child element on the `<datasource>`.

```xml
<datasource name="federated.123" caption="Sales" inline="true">
  <named-connections>
    <named-connection name="sqlserver.prod.sales">
      <connection class="sqlserver" server="prod-server" dbname="SalesDB" ... />
    </named-connection>
  </named-connections>
  <relation type="table" table="[dbo].[FactSales]" name="FactSales" />
</datasource>
```

### Extract Connection

The datasource reads from a pre-built Tableau extract file (`.hyper` or `.tde`) rather than querying the source live.

**XML indicator:** An `<extract>` child element is present on the `<datasource>`.

```xml
<datasource name="federated.123" caption="Sales" inline="true">
  <named-connections>
    <named-connection name="hyper.extract">
      <connection class="hyper" dbname="Data/SalesExtract.hyper" ... />
    </named-connection>
  </named-connections>
  <extract>
    <connection class="hyper" dbname="Data/SalesExtract.hyper" />
  </extract>
  <relation type="table" table="Extract" name="Extract" connection="hyper.extract" />
</datasource>
```

When `class="hyper"` or `class="tde"` is present on the connection, the datasource is always extract-backed. Record the extract file path from the `dbname` attribute.

### Identification Summary

| Condition | Connection Mode |
|---|---|
| No `<extract>` element, non-hyper connection class | Live |
| `<extract>` element present | Extract |
| `class="hyper"` or `class="tde"` | Extract |
| `<repository-location>` present | Published (mode unknown without server access) |

---

## 7. Physical Layer

The Physical Layer describes the raw tables, SQL, joins, and unions that form the base data before any logical modeling.

In the XML, the physical layer is represented by `<relation>` elements directly inside `<datasource>` or nested within each other for joins and unions.

### Physical Table

```xml
<relation type="table"
          table="[dbo].[FactSales]"
          name="FactSales"
          connection="sqlserver.prod.sales" />
```

### Join

```xml
<relation type="join" join="inner" name="FactSales+DimProduct">
  <clause type="join">
    <expression op="=">
      <expression op="[FactSales].[ProductKey]" />
      <expression op="[DimProduct].[ProductKey]" />
    </expression>
  </clause>
  <relation type="table" table="[dbo].[FactSales]" name="FactSales" connection="..." />
  <relation type="table" table="[dbo].[DimProduct]" name="DimProduct" connection="..." />
</relation>
```

### Union

```xml
<relation type="union" name="SalesAll">
  <relation type="table" table="[dbo].[Sales2022]" name="Sales2022" connection="..." />
  <relation type="table" table="[dbo].[Sales2023]" name="Sales2023" connection="..." />
</relation>
```

### Custom SQL

```xml
<relation type="text" name="CustomQuery" connection="sqlserver.prod.sales">
  SELECT
    o.OrderID,
    o.OrderDate,
    c.CustomerName,
    SUM(od.Quantity * od.UnitPrice) AS Revenue
  FROM Orders o
  JOIN Customers c ON o.CustomerID = c.CustomerID
  JOIN OrderDetails od ON o.OrderID = od.OrderID
  GROUP BY o.OrderID, o.OrderDate, c.CustomerName
</relation>
```

For `type="text"`, the text content of the element is the SQL query. Preserve it exactly.

---

## 8. Logical Layer

Tableau's modern data model (introduced in version 2020.2) introduces a Logical Layer above the Physical Layer. The Logical Layer uses relationships instead of joins and supports multi-table analysis without row duplication.

In the XML, the Logical Layer is represented by `<datasource-dependencies>` and `<_.fcp.ObjectModelTableType.true...>` elements, or by `<logical-table>` elements in newer workbook versions.

### Logical Table

```xml
<logical-tables>
  <logical-table id="Orders" caption="Orders">
    <drill-paths>...</drill-paths>
  </logical-table>
  <logical-table id="Products" caption="Products">
    <drill-paths>...</drill-paths>
  </logical-table>
</logical-tables>
```

### Relationship

```xml
<relationship-mapping>
  <relationship-mapping-info>
    <logical-table-ref id="Orders" />
    <logical-table-ref id="Products" />
    <clause type="join">
      <expression op="=">
        <expression op="[Orders].[ProductID]" />
        <expression op="[Products].[ProductID]" />
      </expression>
    </clause>
  </relationship-mapping-info>
</relationship-mapping>
```

### Logical Layer Properties

| Property | Description |
|---|---|
| Cardinality | `many-to-one`, `one-to-one`, `many-to-many` — defines how tables relate |
| Referential Integrity | `some-records-match` or `all-records-match` — affects query generation |
| Performance Options | Stored on the relationship; affects join strategy at query time |

The Logical Layer may not be present in all workbooks. If `<logical-tables>` or `<relationship-mapping>` elements are absent, the datasource uses only the Physical Layer.

---

## 9. Relations

Relations are the core structural elements of the Physical Layer. Every relation has a `type` attribute.

| `type` | Description |
|---|---|
| `table` | A physical table in the source database |
| `join` | A join between two or more relations |
| `union` | A union of two or more relations (stacked rows) |
| `text` | A custom SQL query |

### `type="table"`

```xml
<relation type="table"
          table="[dbo].[DimCustomer]"
          name="DimCustomer"
          connection="sqlserver.prod" />
```

| Attribute | Description |
|---|---|
| `table` | Fully qualified table name in source format, including schema |
| `name` | Internal reference name used elsewhere in the datasource |
| `connection` | References the `name` of a `<named-connection>` |

### `type="join"`

```xml
<relation type="join" join="inner" name="Result">
  <clause type="join">...</clause>
  <relation type="table" ... />
  <relation type="table" ... />
</relation>
```

| Attribute | Description |
|---|---|
| `join` | Join type: `inner`, `left`, `right`, `full` |
| `name` | Internal name for the joined result |

### `type="union"`

```xml
<relation type="union" name="AllSales">
  <relation type="table" ... />
  <relation type="table" ... />
</relation>
```

### `type="text"` (Custom SQL)

```xml
<relation type="text" name="RevenueQuery" connection="sqlserver.prod">
  SELECT ...
</relation>
```

The SQL content is the text node of the element. Preserve exactly.

---

## 10. Joins

Joins appear as `<relation type="join">` elements. The join type is set by the `join` attribute and the join condition is defined inside a `<clause type="join">` child element.

### Join Types

| `join` value | SQL Equivalent |
|---|---|
| `inner` | INNER JOIN |
| `left` | LEFT OUTER JOIN |
| `right` | RIGHT OUTER JOIN |
| `full` | FULL OUTER JOIN |

### Join Clause Structure

```xml
<relation type="join" join="left" name="Orders+Customers">
  <clause type="join">
    <expression op="=">
      <expression op="[Orders].[CustomerID]" />
      <expression op="[Customers].[CustomerID]" />
    </expression>
  </clause>
  <relation type="table" table="[dbo].[Orders]" name="Orders" connection="..." />
  <relation type="table" table="[dbo].[Customers]" name="Customers" connection="..." />
</relation>
```

### Multi-Condition Join

```xml
<clause type="join">
  <expression op="AND">
    <expression op="=">
      <expression op="[Orders].[CustomerID]" />
      <expression op="[Customers].[CustomerID]" />
    </expression>
    <expression op="=">
      <expression op="[Orders].[RegionID]" />
      <expression op="[Customers].[RegionID]" />
    </expression>
  </expression>
</clause>
```

Extract: join type, left table name, right table name, join key pairs from the `<clause>` expression tree.

---

## 11. Custom SQL

Custom SQL relations are defined as `<relation type="text">` elements. The SQL query is stored as the text content of the element — not as an attribute.

```xml
<relation type="text"
          name="MonthlyRevenueSummary"
          connection="sqlserver.prod.sales">
  SELECT
    YEAR(o.OrderDate) AS OrderYear,
    MONTH(o.OrderDate) AS OrderMonth,
    p.Category,
    SUM(od.Quantity * od.UnitPrice * (1 - od.Discount)) AS NetRevenue,
    COUNT(DISTINCT o.CustomerID) AS UniqueCustomers
  FROM Orders o
  INNER JOIN OrderDetails od ON o.OrderID = od.OrderID
  INNER JOIN Products p ON od.ProductID = p.ProductID
  WHERE o.OrderDate >= '2020-01-01'
  GROUP BY
    YEAR(o.OrderDate),
    MONTH(o.OrderDate),
    p.Category
</relation>
```

### Extraction Rules for Custom SQL

- Extract the full SQL string exactly as written. Do not reformat, simplify, or modify it.
- Record the `name` attribute as the query identifier.
- Record the `connection` attribute to identify which database connection this SQL targets.
- Custom SQL may reference temp tables, CTEs, stored procedures, or database-specific syntax. Do not attempt to parse or validate the SQL.
- Flag every custom SQL relation — it is a high-complexity migration signal.

---

## 12. Metadata Records

Metadata records are system-generated descriptions of the fields available from the physical connection. They document each field as it exists in the source system.

```xml
<metadata-records>

  <metadata-record class="column">
    <remote-name>order_date</remote-name>
    <remote-type>135</remote-type>
    <local-name>[Order Date]</local-name>
    <parent-name>[Orders]</parent-name>
    <remote-alias>order_date</remote-alias>
    <local-type>date</local-type>
    <aggregation>None</aggregation>
    <precision>0</precision>
    <scale>0</scale>
    <type>ordinal</type>
    <contains-null>true</contains-null>
    <attributes>
      <attribute datatype="string" name="DebugRemoteType">"WDT_DATE"</attribute>
    </attributes>
  </metadata-record>

</metadata-records>
```

### Field Reference

| Element | Description |
|---|---|
| `remote-name` | Field name as it exists in the source system |
| `remote-type` | Numeric type code from the source driver |
| `local-name` | Field name as used inside Tableau (always in `[brackets]`) |
| `parent-name` | Table or relation this field belongs to (in `[brackets]`) |
| `remote-alias` | Source alias for the field if different from `remote-name` |
| `local-type` | Tableau data type: `string`, `integer`, `real`, `boolean`, `date`, `datetime` |
| `aggregation` | Default aggregation: `Sum`, `Avg`, `Count`, `None`, etc. |
| `precision` | Numeric precision |
| `scale` | Numeric scale |
| `contains-null` | `true` if nulls are possible |
| `class` | Always `"column"` for field records |

Metadata records define the available schema from the connection. They should not be modified or inferred.

---

## 13. Columns

Columns are the field definitions inside a datasource. They appear as `<column>` elements and fall into three types.

### Native Column (Passthrough)

Directly references a source field. No `<calculation>` child is present.

```xml
<column name="[Sales]"
        caption="Sales"
        datatype="real"
        role="measure"
        type="quantitative"
        aggregation="Sum" />
```

### Calculated Column

Contains a `<calculation>` child element with a Tableau formula.

```xml
<column name="[Profit Ratio]"
        caption="Profit Ratio"
        datatype="real"
        role="measure"
        type="quantitative">
  <calculation class="tableau" formula="SUM([Profit]) / SUM([Sales])" />
</column>
```

### Hidden Column

A field that exists in the datasource but is hidden from the view. Identified by `hidden="true"`.

```xml
<column name="[Internal Key]"
        caption="Internal Key"
        datatype="integer"
        role="dimension"
        type="ordinal"
        hidden="true" />
```

### Column Attributes

| Attribute | Description |
|---|---|
| `name` | Internal identifier in `[brackets]` |
| `caption` | Display name in the Tableau UI |
| `datatype` | `string`, `integer`, `real`, `boolean`, `date`, `datetime` |
| `role` | `dimension` or `measure` |
| `type` | `nominal`, `ordinal`, `quantitative`, `temporal` |
| `hidden` | `true` if hidden from views |
| `aggregation` | Default aggregation |
| `semantic-role` | Geographic role if applicable |

### Aliases on Columns

Value-level aliases rename specific data values within a field:

```xml
<column name="[Ship Mode]">
  <aliases>
    <alias key="'First Class'" value="Priority Shipping" />
    <alias key="'Standard Class'" value="Standard Shipping" />
  </aliases>
</column>
```

Extract `key` (source value) and `value` (display value) for every alias.

---

## 14. Datasource Filters

Datasource filters restrict data before it reaches any worksheet. They are defined as `<filter>` elements directly inside `<datasource>`.

### Filter Scope and Execution Order

Filters execute in this order in Tableau:

1. **Extract filters** — Applied when the extract is created. Permanently restrict data in the extract.
2. **Datasource filters** — Applied at query time for all worksheets using this datasource.
3. **Context filters** — Worksheet-level; execute before other worksheet filters.
4. **Dimension filters** — Standard worksheet-level categorical filters.
5. **Measure filters** — Applied after aggregation.

Datasource-level filters (scope 2) appear directly inside the `<datasource>` element.

### Categorical Filter

```xml
<filter class="categorical" column="[Region]">
  <groupfilter function="union" user:ui-marker="enumerate">
    <groupfilter function="member" member="'East'" />
    <groupfilter function="member" member="'West'" />
    <groupfilter function="member" member="'Central'" />
  </groupfilter>
</filter>
```

### Range Filter

```xml
<filter class="quantitative" column="[Sales]" include-values="in-range">
  <min>1000</min>
  <max>500000</max>
</filter>
```

### Date Filter

```xml
<filter class="relative-date" column="[Order Date]">
  <period-type>years</period-type>
  <date-period>3</date-period>
</filter>
```

### Extract Filter

An extract filter appears inside an `<extract>` block:

```xml
<extract>
  <connection class="hyper" ... />
  <refresh>
    <run-at-startup value="no" />
  </refresh>
  <filter class="categorical" column="[Region]">
    <groupfilter function="member" member="'East'" />
  </filter>
</extract>
```

### Filter Classes

| `class` | Description |
|---|---|
| `categorical` | Include or exclude discrete dimension members |
| `quantitative` | Numeric range |
| `relative-date` | Rolling date window |
| `date-range` | Fixed date bounds |
| `null` | Null / non-null membership |

---

## 15. Aliases

Aliases map raw source data values to display labels. They operate at the field level, not the column-definition level.

### Aliases Block on a Column

```xml
<column name="[Category]" caption="Category" datatype="string" role="dimension" type="nominal">
  <aliases>
    <alias key="'FRN'" value="Furniture" />
    <alias key="'OFF'" value="Office Supplies" />
    <alias key="'TEC'" value="Technology" />
  </aliases>
</column>
```

### Datasource-Level Aliases

Less common. Appear directly inside `<datasource>` as an `<aliases>` block applying to a specific column:

```xml
<aliases enabled="yes">
  <alias key="1" value="Active" />
  <alias key="0" value="Inactive" />
</aliases>
```

### Extraction Rules

- Extract every `<alias>` element inside an `<aliases>` block.
- Record `key` as the raw source value and `value` as the display label.
- Preserve the `key` format exactly — string values appear with surrounding single quotes (e.g., `'East'`), numeric values without.

---

## 16. Connection Properties

Connection properties define how Tableau authenticates and communicates with the source system. They are stored as attributes on the `<connection>` element inside `<named-connection>`.

### Full Property Reference

| Property | Attribute | Description |
|---|---|---|
| Server | `server` | Hostname, IP address, or FQDN of the data source |
| Database | `dbname` | Database or catalog name |
| Schema | `schema` | Schema name within the database |
| Port | `port` | TCP port number |
| Username | `username` | Authentication username |
| Authentication | `authentication` | Auth type: `sspi`, `username-password`, `oauth`, `no-authentication` |
| SSL | `sslmode` | SSL enforcement: `require`, `verify-ca`, `verify-full`, `disable` |
| Warehouse | `warehouse` | Compute warehouse (Snowflake) |
| Role | `role` | Session role (Snowflake) |
| Project | `project` | GCP project ID (BigQuery) |
| Dataset | `bq-dataset` | Dataset name (BigQuery) |
| Timeout | `query-timeout` | Query timeout in seconds |
| Initial SQL | `initial-sql` | SQL statement executed at connection open time |

### Initial SQL

Some connections define SQL that runs when the connection is opened:

```xml
<connection class="snowflake" server="..." ...>
  <initial-sql>ALTER SESSION SET TIMEZONE = 'UTC';</initial-sql>
</connection>
```

Extract initial SQL exactly. It is executed before any queries and may set session-level database state.

---

## 17. Parsing Rules

The following rules govern how a parser must handle Tableau datasource XML.

- **Never merge datasources.** Each `<datasource>` element is independent. Fields, filters, and relations from one datasource must not be mixed with those of another.
- **The Parameters datasource is not a data connection.** It has `name="Parameters"` and no `<named-connections>` or `<relation>` children. Do not attempt to extract connection details, tables, or relationships from it.
- **Preserve connection names exactly.** The `name` attribute on `<named-connection>` is referenced by `<relation>` elements via their `connection` attribute. Altering these names breaks the linkage.
- **Preserve server and database values verbatim.** Do not normalize, trim, or interpret connection string values.
- **Never infer relationships.** Only extract relationships that are explicitly defined in `<clause>` or `<relationship-mapping>` elements. Do not assume joins from field names or naming patterns.
- **Preserve custom SQL exactly.** The text content of `<relation type="text">` must be captured as-is. Do not reformat, rewrite, or validate the SQL.
- **Preserve formula strings exactly.** The `formula` attribute on `<calculation>` elements must not be modified.
- **Record absent values as null.** If an attribute is not present on an element, record it as null. Do not substitute defaults.
- **Published datasources are incomplete by definition.** When `<repository-location>` is present, the connection details are not in the workbook. Record the reference and flag as published — do not attempt to infer connection details.
- **Named connection references must be matched.** A `<relation>` element's `connection` attribute must be matched to a `<named-connection>` with the same `name`. If no match exists, flag as a broken reference.

---

## 18. Extraction Schema

Every extracted datasource must produce one record conforming to this schema:

```json
{
  "DatasourceName": "",
  "Caption": "",
  "IsInline": true,
  "IsPublished": false,
  "IsParametersDatasource": false,
  "ConnectionMode": "",
  "Connections": [
    {
      "ConnectionName": "",
      "ConnectionClass": "",
      "Server": "",
      "Database": "",
      "Schema": "",
      "Warehouse": "",
      "Port": "",
      "Username": "",
      "Authentication": "",
      "SSL": "",
      "InitialSQL": ""
    }
  ],
  "PhysicalLayer": {
    "Tables": [
      {
        "Name": "",
        "SourceTable": "",
        "Connection": "",
        "Type": ""
      }
    ],
    "Joins": [
      {
        "Name": "",
        "JoinType": "",
        "LeftTable": "",
        "RightTable": "",
        "JoinKeys": []
      }
    ],
    "Unions": [
      {
        "Name": "",
        "Tables": []
      }
    ],
    "CustomSQL": [
      {
        "Name": "",
        "Connection": "",
        "SQL": ""
      }
    ]
  },
  "LogicalLayer": {
    "Present": false,
    "LogicalTables": [],
    "Relationships": [
      {
        "LeftTable": "",
        "RightTable": "",
        "Cardinality": "",
        "ReferentialIntegrity": "",
        "JoinKeys": []
      }
    ]
  },
  "MetadataRecords": [
    {
      "LocalName": "",
      "RemoteName": "",
      "ParentName": "",
      "LocalType": "",
      "Aggregation": "",
      "ContainsNull": true
    }
  ],
  "Columns": [
    {
      "Name": "",
      "Caption": "",
      "Datatype": "",
      "Role": "",
      "Type": "",
      "Hidden": false,
      "Aggregation": "",
      "IsCalculated": false,
      "Formula": "",
      "Aliases": []
    }
  ],
  "Filters": [
    {
      "Scope": "",
      "Class": "",
      "Column": "",
      "Values": []
    }
  ],
  "ExtractFilePath": "",
  "HasInitialSQL": false,
  "PublishedReference": ""
}
```

---

## 19. Parser Notes

These notes govern specific edge cases the parser must handle correctly.

**The Parameters datasource must be excluded from all connection, table, and relationship extraction.** It has `name="Parameters"` and exists solely to hold `<column>` parameter definitions. Setting `IsParametersDatasource: true` and skipping all physical extraction logic is the correct behavior.

**Federated datasources have multiple `<named-connection>` entries.** Each connection must be extracted independently. Do not collapse them into a single connection record.

**A datasource may have a `<connection>` at the datasource level AND inside `<named-connections>`.** The top-level `<connection>` is a legacy pattern. Prefer the `<named-connections>` block when both are present.

**Extract datasources may carry both the original live connection and the extract connection.** The live connection represents the original source; the extract connection (`class="hyper"`) represents the cached extract. Both should be recorded. Set `ConnectionMode` to `"Extract"` and record the extract file path from the `hyper` connection's `dbname` attribute.

**`<relation>` elements can be deeply nested.** A join of a join is valid. Parse recursively until all leaf nodes (`type="table"` or `type="text"`) are reached.

**Named connections are referenced by name, not by position.** Always match a relation's `connection` attribute to a named connection by the `name` attribute value — not by order of appearance.

**A datasource with no `<named-connections>` and no `<relation>` is either the Parameters datasource or a scaffold datasource.** Set `HasConnection: false` and extract only column/parameter definitions.

---

## 20. Validation Notes

These flags must be raised during extraction when specific conditions are detected.

| Condition | Severity | Flag |
|---|---|---|
| Datasource present but no `<named-connections>` block | Warning | No connection defined |
| `<named-connection>` present but `server` attribute is empty or absent | Warning | Missing server |
| `<relation>` references a `connection` name not found in `<named-connections>` | Error | Broken connection reference |
| `<relation type="join">` has no `<clause>` child | Error | Join with no condition |
| `<relation type="text">` has empty text content | Error | Empty custom SQL |
| `<repository-location>` present — published datasource | Info | Published datasource — full definition unavailable |
| `<extract>` present — extract-backed datasource | Info | Extract connection detected |
| Multiple `<named-connection>` entries — federated datasource | Info | Federated datasource detected |
| `<connection initial-sql>` present | Warning | Initial SQL present — session-level state may be set |
| Column `formula` attribute is empty string | Warning | Calculated field with empty formula |
| `<aliases>` present on a column | Info | Value aliases detected — display values differ from source values |
| Logical Layer present (`<logical-tables>` detected) | Info | Modern data model detected |

---

## 21. Migration Impact

This section classifies the complexity contribution of each datasource feature. It does not describe how to migrate — it provides signals for the Orchestrator to assess workbook complexity before agent assignment.

| Feature | Migration Impact | Reason |
|---|---|---|
| Live Connection | High | Requires matching live connectivity in the target environment |
| Extract Connection | Medium | Extract must be rebuilt; source connection still required |
| Custom SQL | High | SQL may use source-specific syntax, temp tables, or CTEs not portable to Power BI |
| Multiple Datasources | High | Cross-datasource relationships must be re-modeled |
| Federated Datasource | High | Multi-source connections require complex semantic model design |
| Published Datasource | High | Full definition unavailable in workbook; requires server access |
| Logical Layer (Relationships) | Medium | Must be re-expressed as semantic model relationships |
| Physical Layer Joins | Medium | Join logic must be replicated in the semantic model |
| Union Relations | Medium | Unions must be replicated in Power Query |
| Datasource Filters | Medium | Must be re-applied at semantic model or report level |
| Extract Filters | High | Filter restricts the extract — may hide data scope from migration agent |
| Initial SQL | High | Session-level state not portable; requires manual review |
| Aliases | Low | Value-level display mapping; manageable in Power BI |
| Hidden Columns | Low | Hidden fields carry metadata but no visual impact |
| Parameters Datasource | Low | Parameters are extracted separately; no physical connection involved |
| File-Based Connection (CSV/Excel) | Low | File sources are straightforward to replicate |
| Geospatial Fields | Medium | Geographic roles require specific handling in Power BI |

---

*Tableau Datasource Reference v1.0 — Structure and connection reference for metadata extraction agents. Tableau datasources only. No migration output, DAX, or Power BI content is described in this document.*
