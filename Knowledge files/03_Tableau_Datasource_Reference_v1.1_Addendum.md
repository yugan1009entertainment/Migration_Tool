# Tableau Datasource Reference v1.1 - Enterprise Addendum

## Revision History

  -----------------------------------------------------------------------
  Version                             Changes
  ----------------------------------- -----------------------------------
  1.1                                 Added datasource dependency graph,
                                      capability matrix, complexity
                                      classification, security, health,
                                      parser metrics, feature inventory,
                                      readiness score, unsupported
                                      features.

  1.0                                 Initial release
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# Datasource Dependency Graph

    Datasource
        │
        ├── Named Connections
        │       │
        │       ├── Physical Relations
        │       │       ├── Tables
        │       │       ├── Joins
        │       │       ├── Unions
        │       │       └── Custom SQL
        │       │
        │       └── Metadata Records
        │               ├── Columns
        │               ├── Calculated Fields
        │               └── Aliases
        │
        └── Worksheets
                └── Dashboards

Parser should preserve these dependencies for downstream agents.

------------------------------------------------------------------------

# Connection Capability Matrix

  Source             Live   Extract
  ----------------- ------ ---------
  SQL Server          ✓        ✓
  Oracle              ✓        ✓
  PostgreSQL          ✓        ✓
  Snowflake           ✓        ✓
  Amazon Redshift     ✓        ✓
  Excel               ✓        ✗
  CSV/Text            ✓        ✗
  Hyper               ✗        ✓
  TDE                 ✗        ✓

------------------------------------------------------------------------

# Datasource Complexity Classification

  -----------------------------------------------------------------------
  Level                               Criteria
  ----------------------------------- -----------------------------------
  Simple                              Single table, single connection

  Medium                              Multiple tables, joins

  Complex                             Custom SQL, Extracts, Logical Layer

  Enterprise                          Federated, Published Datasources,
                                      Initial SQL, Multiple Connections
  -----------------------------------------------------------------------

Expose a `DatasourceComplexity` value.

------------------------------------------------------------------------

# Connection Security

Recognize and preserve:

-   Windows Authentication (SSPI)
-   Username / Password
-   OAuth
-   Kerberos
-   No Authentication
-   SSL Mode
-   Certificates (if defined)

Never expose secrets. Record only authentication metadata.

------------------------------------------------------------------------

# Datasource Health

Classify datasource health.

  Status    Meaning
  --------- ------------------------------------------------
  Healthy   All required metadata available
  Warning   Missing optional properties
  Error     Broken references or missing mandatory objects

------------------------------------------------------------------------

# Parser Metrics

Produce a summary.

``` json
{
  "DatasourceCount":0,
  "ConnectionCount":0,
  "TableCount":0,
  "RelationshipCount":0,
  "CalculatedFieldCount":0,
  "PublishedDatasourceCount":0
}
```

------------------------------------------------------------------------

# Datasource Feature Inventory

``` json
{
  "UsesExtract":false,
  "UsesPublishedDatasource":false,
  "UsesFederatedDatasource":false,
  "UsesCustomSQL":false,
  "UsesLogicalLayer":false,
  "UsesInitialSQL":false
}
```

------------------------------------------------------------------------

# Migration Readiness Score

Estimate readiness only.

    Start Score = 100

    Subtract:
    Published Datasource
    Federated Datasource
    Custom SQL
    Initial SQL
    Unsupported Driver
    Missing Connection
    Broken Join

Do not block migration; provide the score as an advisory metric.

------------------------------------------------------------------------

# Unsupported Datasource Features

Recognize and flag:

-   SAP BW
-   SAP HANA OLAP
-   Legacy Cube Connections
-   Web Data Connector
-   Analytics Extensions
-   Legacy TDE
-   Deprecated Drivers

------------------------------------------------------------------------

# Engineering Notes

-   Preserve datasource hierarchy.
-   Preserve connection identifiers.
-   Preserve relationship graph.
-   Never infer missing connection properties.
-   Treat this specification as authoritative for datasource parsing.

End of v1.1 Addendum.
