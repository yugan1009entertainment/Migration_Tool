# 📘 CONVERSION KNOWLEDGE FILE
# 24451 – SUF Skills Matrix
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-07
# Source files: suf_skills_matrix.txt + all PBIP .txt files

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

NO PARAMETERS. This workbook has no Tableau parameters and no
parameter tables in the PBIP model.

Conversion Status: ✅ N/A


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: dvn_suf_skills_matrix

### CC-01 | Status Icon Column
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 8d4b963e-a922-4269-be73-e816eb206714
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Equivalent:
───────────────────
// Tableau used Shape encoding on [status] with custom shape files:
//   "qualified trainer" → :filled/asterisk  (★)
//   "completed"         → :filled/circle    (●)
//   "in progress"       → Proportions/2.png (◐)
// PBI cannot use Tableau custom shape files — converted to Unicode
// character column for display in visuals/tooltips.

Power BI DAX:
─────────────
column 'Status Icon Column' =
SWITCH(
    dvn_suf_skills_matrix[Status],
    "in progress",       "◐",
    "completed",         "●",
    "qualified trainer", "★"
)

Depends on  : [Status] (source column)
Used by     : Visual mark display (replaces Tableau shape encoding)
⚠️  NOTE    : Tableau used Proportions/2.png (a half-filled circle image)
              for "in progress". PBI uses Unicode ◐ as the closest
              text-based equivalent. If pixel-perfect shape parity is
              required, use a custom visual or icon set conditional
              formatting with a half-circle SVG/PNG image.
Conversion  : ✅ FULL (with noted shape approximation)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: dvn_suf_skills_matrix

### M-01 | Measure  ← Placeholder / Scaffold
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 913aa014-2c24-40a5-8020-4fae1c826c60
PBI Format      : (none)

Tableau Equivalent:
───────────────────
// No equivalent — PBI scaffold measure only

Power BI DAX:
─────────────
measure Measure =
// (empty — no expression defined)

Annotation: 43dbc3e8-3a1c-4b6f-9923-b49ff7d6691c = True
NOTE      : Empty placeholder measure created automatically by
            Power BI Desktop when the table was first added.
            Safe to keep or remove — has no impact on report logic.
Conversion  : ➕ PBI-only (scaffold)


### M-02 | Color Measure
─────────────────────────────────────────────────────────────
PBI Lineage Tag : fc3b7db5-7ac0-4a2c-b3a4-02a882969dac
PBI Format      : (string — hex color)

Tableau Equivalent:
───────────────────
// Tableau used a palette color encoding on [designation]:
//   Each designation value → fixed hex color defined in workbook XML
// PBI cannot use Tableau palette encoding directly —
// converted to a DAX measure returning the hex string for use
// in conditional formatting (background/font color rules).

Power BI DAX:
─────────────
measure 'Color Measure' =
SWITCH(
    SELECTEDVALUE(dvn_suf_skills_matrix[Designation]),
    "Instructor Led PQC", "#87D180",
    "OJT Tier 1",         "#4E79A7",
    "OJT Tier 2",         "#B07AA1",
    "OJT Tier 3",         "#9C755F",
    "Wave 1",             "#E15759",
    "WAVE-1",             "#F28E2B",
    "Wave 2",             "#76B7B2",
    "Wave 3",             "#59A14F",
    "Wave 4",             "#EDC948",
    "Wave 5",             "#FF9DA7",
    "Wave 6",             "#B07AA1",
    "Wave 7",             "#9C755F"
)

Depends on  : [Designation] (source column, post-PQ rename)
Used by     : Conditional formatting rules on visuals (color field)
⚠️  NOTE    : "WAVE-1" in DAX corresponds to "WAVE 1" in the Tableau
              source. The Power Query rename (WAVE 1 → WAVE-1) is what
              makes this value differ between Tableau and PBI — see PQ
              transform note in Section 5.
              OJT Tier 2 and Wave 6 intentionally share color #B07AA1.
              OJT Tier 3 and Wave 7 intentionally share color #9C755F.
Conversion  : ✅ FULL


### M-03 | UserNameMeasure
─────────────────────────────────────────────────────────────
PBI Lineage Tag : ead1da7c-abde-462e-a2f5-e44e58ca4ce3
PBI Format      : (string)

Tableau Equivalent:
───────────────────
USERNAME()
// Returns the current user's BMS ID for RLS or tooltip context.

Power BI DAX:
─────────────
measure UserNameMeasure =
VAR u = USERNAME()
RETURN
    UPPER(
        IF(
            CONTAINSSTRING(u, "\"),
            RIGHT(u, LEN(u) - FIND("\", u)),
            LEFT(u, FIND("@", u) - 1)
        )
    )

Logic:
  - If the username contains a backslash (domain\user format):
      → extract everything AFTER the backslash
  - Else (UPN format user@domain):
      → extract everything BEFORE the @ symbol
  - Result is uppercased in both cases.

Examples:
  "BMS\jsmith"     → "JSMITH"
  "jsmith@bms.com" → "JSMITH"

Depends on  : USERNAME() (DAX context function)
Used by     : RLS or display context — returns normalised BMS user ID
⚠️  NOTE    : Tableau USERNAME() returns the raw Tableau username.
              PBI USERNAME() returns either domain\user or UPN depending
              on environment. This measure normalises both formats.
              Verify which format is returned in the BMS Fabric/PBI
              Service environment and confirm the correct branch fires.
Conversion  : ✅ FULL + enhanced (dual-format handling)


### M-04 | Color box
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 6bbd9e34-44da-4113-b0dc-506bb605624c
PBI Format      : (string)

Tableau Equivalent:
───────────────────
// No direct equivalent — PBI-only legend/label helper

Power BI DAX:
─────────────
measure 'Color box' = "■"

NOTE      : Returns a filled square Unicode character.
            Used as a visual legend label alongside [Color Measure]
            to display a colored square next to designation labels
            in tooltips or custom legends. Pair with [Color Measure]
            for conditional font color to produce a colored ■ symbol.
Conversion  : ➕ PBI-only enhancement


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — POWER QUERY / DATA SOURCE
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PQ-01 | dvn_suf_skills_matrix  ← Main Fact Table
─────────────────────────────────────────────────────────────
Tableau Data Source   : Redshift (federated), live + .hyper extract
                        Server   : bms-gpsdl-non-validated-redshift-prod.web.bms.com
                        Port     : 5439
                        Database : gpsdb
                        Schema   : gps_bia_mfg
                        Table    : [gps_bia_mfg].[dvn_suf_skills_matrix]
                        Auth     : auth-user-pass (username: raynore)
                        SSL      : require

PBI Table Name        : dvn_suf_skills_matrix
PBI lineageTag        : 11eabec6-9829-4404-afd0-3d192c288a9b
PBI Load Mode         : Import
PBI Partition ID      : 24b50404-b016-4c20-8ea4-ff5a7c96a04d_Partition
Annotations           : TableName = dvn_suf_skills_matrix
                        TableType = DataTable
                        PBI_NavigationStepName = Navigation
                        PBI_ResultType = Table

Power BI Power Query M:
───────────────────────
let
    ClusterEndpoint  = "bms-gpsdl-non-validated-redshift-prod.web.bms.com:5439",
    DatabaseName     = "gpsdb",
    SchemaName       = "gps_bia_mfg",
    TableName        = "dvn_suf_skills_matrix",
    Source           = AmazonRedshift.Database(
                           ClusterEndpoint,
                           DatabaseName,
                           [BatchSize = null, Role = null, ProviderName = null]
                       ),
    SchemaData       = Source{[Name = SchemaName, Kind = "Schema"]}[Data],
    #"gps_bia_mfg_dvn_suf_skills_matrix" =
                       SchemaData{[Name = TableName]}[Data],
    #"Select Columns" = Table.SelectColumns(
                           #"gps_bia_mfg_dvn_suf_skills_matrix",
                           {"name", "user_id", "shift", "pqc_title",
                            "pqc_id", "status", "designation", "us/ds"}
                        ),
    #"Filtered Rows"  = Table.SelectRows(#"Select Columns", each true),
    #"Replaced Value" = Table.ReplaceValue(
                            #"Filtered Rows",
                            "WAVE 1", "1 WAVE",
                            Replacer.ReplaceText, {"designation"}
                        ),
    #"Filtered Rows1" = Table.SelectRows(#"Replaced Value", each true),
    #"Replaced Value1" = Table.ReplaceValue(
                             #"Filtered Rows1",
                             "1 WAVE", "WAVE-1",
                             Replacer.ReplaceText, {"designation"}
                         )
in
    #"Replaced Value1"

⚠️  CRITICAL DATA TRANSFORM — DESIGNATION RENAME:
    Tableau source stores "WAVE 1" (space). Power Query renames it
    via a two-step intermediate transform:
      Step 1: "WAVE 1"  → "1 WAVE"   (intermediate — never visible)
      Step 2: "1 WAVE"  → "WAVE-1"   (final stored value in PBI)
    Result: PBI stores "WAVE-1" (hyphen); Tableau stores "WAVE 1" (space).
    ALL DAX measures, color maps, and filters referencing this value
    MUST use "WAVE-1" in PBI and "WAVE 1" in Tableau.
    The Tableau color map had two variants: "Wave 1" (#E15759) and
    "WAVE 1" (#F28E2B). In PBI these become "Wave 1" and "WAVE-1".
    If the upstream source ever renames this value, the PQ steps
    must be updated.

Source Column Mapping:

Ordinal  PBI Column Name    lineageTag     Data Type  Source Column  Summarize
────────────────────────────────────────────────────────────────────────────────
1        Name               [name]         string     name           none
2        User Id            [user_id]      string     user_id        none
3        Shift              [shift]        string     shift          none
4        Pqc Title          [pqc_title]    string     pqc_title      none
5        Pqc Id             [pqc_id]       string     pqc_id         none
6        Status             [status]       string     status         none
7        Designation        [designation]  string     designation    none  ← rename applied
8        Us/Ds              [us/ds]        string     us/ds          none

Conversion Status: ✅ FULL — all 8 source columns loaded;
                   designation rename transform applied.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — MEASURES TABLE
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### Measures Table
─────────────────────────────────────────────────────────────
PBI Table Name  : Measures Table
PBI lineageTag  : 55462a6b-b4fe-4974-8c87-758bb87d95af
PBI Type        : Empty scaffold table (M query, Base64 compressed)
Annotations     : TableType = MeasureTable
                  PBI_ResultType = Table
                  PBI_NavigationStepName = Navigation

Columns:
  Column1 — string, hidden (changedProperty = IsHidden), summarizeBy: none

Power BI Power Query M:
───────────────────────
let
    Source = Table.FromRows(
        Json.Document(
            Binary.Decompress(
                Binary.FromText("i44FAA==", BinaryEncoding.Base64),
                Compression.Deflate
            )
        ),
        let _t = ((type nullable text) meta [Serialized.Text = true])
        in type table [Column1 = _t]
    ),
    #"Changed Type" = Table.TransformColumnTypes(Source, {{"Column1", type text}})
in
    #"Changed Type"

NOTE: "i44FAA==" decodes to a single empty/null row — standard PBI
      scaffold pattern for a measures-only table. No measures are
      currently hosted on this table; all measures live on
      dvn_suf_skills_matrix. This is valid but non-standard —
      consider migrating measures to Measures Table in a future
      refactor for cleaner model organisation.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — WORKBOOK METADATA
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PBIP Report Name          : 24451_SUF Skills Matrix
PBIP Model Name           : 24451_SUF Skills Matrix- model
Compatibility Level       : 1604
Schema Version            : 4.2  (definition.json)
Culture                   : en-US
PBI Data Source Version   : powerBI_V3
Time Intelligence         : Enabled  (__PBI_TimeIntelligenceEnabled = 1)
Dev Mode                  : Enabled  (PBI_ProTooling = ["DevMode"])
Query Order               : Measures Table → dvn_suf_skills_matrix
Created By                : TOM

Tableau Workbook          : SUF Skills Matrix
Tableau Workbook ID       : SUFSkillsMatrix
Tableau URL               : https://tableau.bms.com/t/PRODUCTION/workbooks/SUFSkillsMatrix
Tableau Revision          : 2.8
Tableau Build             : 2024.2.13 (20242.25.0710.1539)
Last Extract Refresh      : 07/06/2026 11:31:11 AM


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — DASHBOARD FILTERS & REPORT STRUCTURE
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### Locked / Hidden Filters

Filter               Value           Scope               PBI Implementation
──────────────────────────────────────────────────────────────────────────────
Us/Ds = "downstream" Locked filter   Downstream page     Hidden page-level filter
                                                         on [Us/Ds] = "downstream"
Us/Ds = "upstream"   Locked filter   Upstream page       Hidden page-level filter
                                                         on [Us/Ds] = "upstream"

### Slicers

Slicer Caption  Field            Scope          PBI Column
──────────────────────────────────────────────────────────
OJT Mapping     [Designation]    Both pages     dvn_suf_skills_matrix[Designation]
PQC Title       [Pqc Title]      Both pages     dvn_suf_skills_matrix[Pqc Title]
Shift           [Shift]          Upstream only  dvn_suf_skills_matrix[Shift]

### Dashboard Actions

ACTION-01 | Highlight Row/Columns
  Tableau Type : Brush/Highlight action
  Trigger      : On Select (auto-clear)
  Fields       : Name, Pqc Id
  Target       : upstream sheet
  PBI Equiv.   : Cross-highlight via Edit Interactions between
                 Downstream and Upstream page matrix visuals
                 using [Name] and [Pqc Id] as the linking fields.

### Report Pages

PAGE-01 | Downstream Dashboard  (Tableau: "downstream" sheet)
  Visual Type  : Matrix / cross-tab
  Rows         : Shift → Name
  Columns      : Designation → Pqc Id  (rotated -90° header labels)
  Values       : Status Icon Column (CC-01)
  Color        : Designation (M-02 Color Measure — conditional format)
  Tooltip      : Pqc Title, Pqc Id (PQC Code), Name (Employee Name),
                 Status / Status Icon Column (Training Status),
                 Shift, Designation (OJT Mapping)
  Dashboard    : 1366 × 768 fixed
  Locked Filter: Us/Ds = "downstream"
  Slicers      : OJT Mapping (Designation), PQC Title
  Column widths: Default cell = 150px; Pqc Id = 39px
  Name header  : 176px wide; Pqc Id header = 228px tall (rotated)
  Display      : Column field labels hidden
                 (display-field-labels cols = false)

PAGE-02 | Upstream Dashboard  (Tableau: "upstream" sheet)
  Visual Type  : Matrix / cross-tab
  Rows         : Shift → Name
  Columns      : Designation → Pqc Id  (rotated -90° header labels)
  Values       : Status Icon Column (CC-01)
  Color        : Designation (M-02 Color Measure — conditional format)
  Tooltip      : Pqc Title, Designation (OJT Mapping), Pqc Id (PQC Code),
                 Name (Employee Name), Status (Training Status), Shift
  Dashboard    : 1366 × 768 fixed
  Locked Filter: Us/Ds = "upstream"
  Slicers      : OJT Mapping (Designation), Shift, PQC Title
  Cell widths  : Default cell = 5px; Pqc Id = 45px
  Name row     : 26px height; Name header = 116px wide
  Pqc Id header: 208px tall (rotated); Shift header = 76px wide
  Display      : Column field labels hidden
                 (display-field-labels cols = false)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — COLOR & SHAPE MAP
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### Color Map — Designation

Designation Value       Tableau Source Value  PBI Stored Value  Hex Color
──────────────────────────────────────────────────────────────────────────
Instructor Led PQC      Instructor Led PQC    (same)            #87D180
OJT Tier 1              OJT Tier 1            (same)            #4E79A7
OJT Tier 2              OJT Tier 2            (same)            #B07AA1
OJT Tier 3              OJT Tier 3            (same)            #9C755F
Wave 1                  Wave 1                (same)            #E15759
WAVE 1 ← Tableau only   WAVE 1             → WAVE-1            #F28E2B  ⚠️ PQ renamed
Wave 2                  Wave 2                (same)            #76B7B2
Wave 3                  Wave 3                (same)            #59A14F
Wave 4                  Wave 4                (same)            #EDC948
Wave 5                  Wave 5                (same)            #FF9DA7
Wave 6                  Wave 6                (same)            #B07AA1  (= OJT Tier 2)
Wave 7                  Wave 7                (same)            #9C755F  (= OJT Tier 3)

### Shape Map — Status

Status Value          Tableau Shape            PBI Unicode  PBI Char
──────────────────────────────────────────────────────────────────────
completed             :filled/circle           U+25CF       ●
in progress           Proportions/2.png        U+25D0       ◐  (approx)
qualified trainer     :filled/asterisk         U+2605       ★


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — KNOWN ISSUES / NOTES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAP-01 | "WAVE 1" → "WAVE-1" Power Query rename  ⚠️ CRITICAL
  Severity : High
  Detail   : Tableau source stores "WAVE 1" (with space). Power Query
             renames it to "WAVE-1" (with hyphen) via a two-step
             intermediate transform. ALL DAX references, filters,
             and slicers must use "WAVE-1" in PBI.
             If the upstream Redshift source ever renames this value,
             the PQ replace steps must be updated.
  Action   : Document the rename in any data dictionary or data contract.
             Verify PQ steps are retained on every PBIP refresh.

GAP-02 | Shape encoding approximation (in progress → ◐)
  Severity : Low
  Detail   : Tableau uses Proportions/2.png (half-filled circle image)
             for "in progress". PBI uses Unicode ◐ as the closest
             text-based equivalent. Rendering may differ across fonts.
  Action   : If pixel-perfect shape parity is required, use a custom
             visual or PBI icon set conditional formatting with a
             half-circle SVG/PNG image.

GAP-03 | Tooltip references internal Tableau calculation IDs
  Severity : Info
  Detail   : Tableau tooltips reference [Calculation_826973483539910656]
             (downstream) and [Calculation_189714200731176960] (upstream)
             labelled as "PQC Code". These are internal Tableau IDs
             resolving to [pqc_id] at runtime.
  Action   : Use [Pqc Id] directly in PBI tooltip configuration.

GAP-04 | Highlight action (cross-page)
  Severity : Medium
  Detail   : Tableau's "Highlight Row/columns" action fires on select
             across both sheets using Name + Pqc Id as keys.
             PBI cross-page interaction requires bookmarks or
             cross-filter/cross-highlight configuration.
  Action   : Implement via Edit Interactions → cross-highlight between
             Downstream and Upstream page matrix visuals on [Name]
             and [Pqc Id].

GAP-05 | Measures Table is empty
  Severity : Info
  Detail   : The Measures Table scaffold exists but holds no measures.
             All measures (M-01 to M-04) live on dvn_suf_skills_matrix.
             This is valid but non-standard.
  Action   : Consider migrating measures to Measures Table in a future
             refactor for cleaner model organisation.

GAP-06 | UserNameMeasure environment dependency
  Severity : Medium
  Detail   : USERNAME() returns different formats in different PBI
             environments (domain\user vs. UPN). The measure handles
             both, but the active branch depends on the BMS deployment
             environment (Fabric, PBI Service, PBI Desktop).
  Action   : Test USERNAME() output in the target BMS Fabric/PBI Service
             environment and confirm which branch of the IF is exercised.

GAP-07 | No date columns — no LocalDateTables
  Severity : Info
  Detail   : This model has no date-type columns. Therefore no
             LocalDateTables, DateTableTemplate, or date hierarchies
             are present. This is expected and correct for this workbook.
  Action   : None.

GAP-08 | Single-table model — no relationships
  Severity : Info
  Detail   : The Tableau workbook had a single data source and no joins.
             The PBI model has no cross-table relationships. This is
             expected and correct.
  Action   : None.
