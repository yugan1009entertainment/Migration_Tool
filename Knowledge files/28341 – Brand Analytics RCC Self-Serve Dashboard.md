# 📘 CONVERSION KNOWLEDGE FILE
# 28341 – Brand Analytics RCC Self-Serve Dashboard
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-08
# Source files: 28341_brand_analytics_rcc_self_serve_dashboard.txt + all PBIP .txt files

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 0 — WORKBOOK & MODEL METADATA
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Tableau Workbook Name          : Brand Analytics RCC Self-Serve Dashboard
Tableau Workbook ID            : 28341
Tableau Server Path            : /t/PRODUCTION/workbooks
Tableau Server URL             : https://tableau.bms.com/t/PRODUCTION/workbooks/BrandAnalyticsRCCSelf-ServeDashboard
Tableau Revision               : 1.0 (derived-from rev 2.0)
Tableau Build                  : 2024.2.13 (20242.25.0710.1539)
Tableau Original Version       : 18.1

PBIP Dataset Name              : 29965_28341-Brand Analytics RCC Self-Serve Dashboard
PBIP Model Name                : 29965_28341-Brand Analytics RCC Self-Serve Dashboard- model
PBIP compatibilityLevel        : 1604
PBIP defaultPowerBIDataSourceVersion : powerBI_V3
PBIP culture                   : en-US
PBIP sourceQueryCulture        : en-US
PBIP Time Intelligence         : DISABLED (__PBI_TimeIntelligenceEnabled = 0)
PBIP ProTooling                : DevMode
PBIP Definition Schema Version : 4.2
PBIP PBI_QueryOrder            : ["Measures Table", "rcc_ss_presc_master_db", "Refresh_Info"]

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — DATA SOURCE
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### DS-01 | Tableau Datasource
─────────────────────────────────────────────────────────────
Datasource Caption    : cdw
Datasource Name       : federated.12yrtjj1wstoqp16nhetd1bycwvo
Connection Type       : Redshift (federated)
Server                : comm-cdw-prod-amer-01.cmyzp34zrkds.us-east-1.redshift.amazonaws.com
Port                  : 5439
Database              : cdw
Schema                : us_sandbox
Table                 : rcc_ss_presc_master_db
Full Table Reference  : [us_sandbox].[rcc_ss_presc_master_db]
Authentication        : auth-user-pass
SSL Mode              : require
Tableau Username      : kumaa253
Extract Format        : .hyper (last updated 01/07/2026)


### DS-02 | PBIP Partition (Power Query M)
─────────────────────────────────────────────────────────────
Mode                  : Import (NOT DirectQuery)
Column Selection      : Explicit Table.SelectColumns — 68 columns (no SELECT *)
Null Handling         : rcc_segments nulls → replaced with string "Null"
                        via Table.ReplaceValue step (required for sort table join)

Power BI M Source:
──────────────────
let
    ClusterEndpoint = "comm-cdw-prod-amer-01.cmyzp34zrkds.us-east-1.redshift.amazonaws.com:5439",
    DatabaseName    = "cdw",
    SchemaName      = "us_sandbox",
    TableName       = "rcc_ss_presc_master_db",
    Source = AmazonRedshift.Database(ClusterEndpoint, DatabaseName,
                 [BatchSize = null, Role = null, ProviderName = null]),
    SchemaData = Source{[Name = SchemaName, Kind = "Schema"]}[Data],
    #"us_sandbox_rcc_ss_presc_master_db" = SchemaData{[Name = TableName]}[Data],
    #"Select Columns" = Table.SelectColumns(
        #"us_sandbox_rcc_ss_presc_master_db",
        {
            "bp_id", "npi_id", "hcp_nm", "prim_spty_code", "sec_spty_code",
            "hcp_address", "hcp_classification", "bp_status", "pdrp_ind",
            "hcp_trgt_type_txt", "rcc_dynm_seg_nm", "hcp_city", "hcp_state",
            "hcp_zip", "hcp_terr", "hcp_dst", "hcp_regn", "hcp_affil_type",
            "rcc_value_score", "fld_trgt_flg", "primary_specialty",
            "secondary_specialty", "final_spec_cd", "spec_group", "outlet_nm",
            "outlet_address", "outlet_zip", "outlet_city", "outlet_state",
            "grp_id", "grp_nm", "date", "yr_mnth", "source", "patient_id",
            "procedure_date", "clm_yr_mnth", "clm_weekending", "week_no",
            "month_no", "r6w_flg", "day_name_clm", "day_of_week_clm",
            "clm_rank", "primary_prod", "mutation", "lot",
            "targeted_therapy_flag", "nps_flg", "trialist_flg", "repeater_flg",
            "call_weekending", "call_week_no", "call_yr_mnth", "call_product",
            "call_sub_type_cd", "presn_sbtyp_txt", "call_position", "callkey",
            "sf_cd", "sf_nm", "call_territory", "call_district", "call_region",
            "rcc_segments", "subc_flag", "sc_index_1_flagged_hcps",
            "past_sc_experience_volume_claims"
        }
    ),
    #"Replaced Value" = Table.ReplaceValue(
        #"Select Columns", null, "Null", Replacer.ReplaceValue, {"rcc_segments"}
    )
in
    #"Replaced Value"

⚠️ KEY NOTE: rcc_segments nulls are replaced with the string "Null" (not BLANK).
   This is required for the RCC_Segment_Order join — that table uses "Null"
   as a valid key value at SortOrder = 1. Do NOT remove this step.

Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU PARAMETERS → POWER BI PARAMETER / SORT TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PARAM-01 | RCC Segment Sort Order
─────────────────────────────────────────────────────────────
Tableau Origin        : Custom sort applied to [rcc_segments] / [rcc_dynm_seg_nm]
                        field; no explicit Tableau parameter — sort was
                        defined in worksheet properties.
Tableau Default Sort  : Null → Dual IO Believers → IO-TKI Leaners →
                        Splitters → TKI Mono Holdouts → Unsegmented

Power BI Table Name   : RCC_Segment_Order
Power BI Type         : DAX Calculated Table (DATATABLE, import mode)
Power BI lineageTag   : 97956f9a-797c-4427-8d47-e245a7120fdd
Power BI Columns      : [Rcc Segments] (string, isNameInferred, sortByColumn: SortOrder)
                        [SortOrder]    (integer, format: 0, isNameInferred)

Power BI DAX:
─────────────
DATATABLE(
    "Rcc Segments", STRING,
    "SortOrder",    INTEGER,
    {
        { "Null",              1 },
        { "Dual IO Believers", 2 },
        { "IO-TKI Leaners",    3 },
        { "Splitters",         4 },
        { "TKI Mono Holdouts", 5 },
        { "Unsegmented",       6 }
    }
)

Usage in model:
  - [Rcc Segments] joins to rcc_ss_presc_master_db[Rcc Segments]
    (relationship 6bad4888-04de-68b4-df98-1eb636c3e7ad).
  - sortByColumn: SortOrder on [Rcc Segments] drives visual sort order
    on all segment slicers and charts.
  - "Null" (string) at position 1 pairs with the PQ null-replacement step.
    The string "Null" is a deliberate choice — do not change to BLANK().

⚠️ CRITICAL: All 6 values and their SortOrder integers are exact.
   Do not reorder, rename, or add values without a formal change request.

Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU GROUPS / CALCULATED FIELDS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: rcc_ss_presc_master_db

### CC-01 | Primary Specialty Group
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Primary Specialty (group)
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Origin:
───────────────
Tableau Group applied to [Primary Specialty].
All specialties not in the named list are grouped into "Others".
BLANK() / null is explicitly included in the "Others" bucket.

Power BI DAX:
─────────────
column 'Primary Specialty Group' =
IF (
    'rcc_ss_presc_master_db'[Primary Specialty] IN {
        BLANK(),
        "Addiction Medicine",
        "Associate Registered Nurse",
        "Cardiovascular Disease",
        "Certified Nursing Assistant",
        "Clinical Nurse Specialist",
        "Dentistry",
        "Diagnostic Radiology",
        "Emergency Medicine",
        "Endocrinology",
        "Family Medicine",
        "Gastroenterology",
        "Gastrointestinal Oncology",
        "General Practitioner",
        "General Surgery",
        "Hematology",
        "Hepatology",
        "Hospice and Palliative Medicine",
        "Hospital Medicine Physician",
        "Hospital Pharmacy Foundation",
        "Hospitalist",
        "Immunology and Genetics",
        "Infectious Disease",
        "Internal Medicine",
        "Internal Medicine-Geriatrics",
        "Laboratory Medicine",
        "Maternal-Fetal Medicine",
        "Medical Physics",
        "Nephrology",
        "Neurology and Neurosurgery",
        "Not Available (context dependent)",
        "Nursing",
        "Obstetrics and Gynecology",
        "Occupational Safety and Medicine",
        "Ophthalmology",
        "Orthopedic Surgery",
        "Otolaryngology",
        "Palliative Care Consultant",
        "Palliative Medicine",
        "Pediatric Dentistry",
        "Pediatric Hematology-Oncology",
        "Pharmacy",
        "Preventive Medicine Doctor",
        "Psychology",
        "Pulmonary Disease",
        "Pulmonary Therapy",
        "Pulmonologist",
        "Radiation Oncology",
        "Registered Nurse - Rehabilitation",
        "Rheumatology and Immunology",
        "Surgical Oncology",
        "Thoracic Surgery",
        "Urology"
    },
    "Others",
    'rcc_ss_presc_master_db'[Primary Specialty]
)

Depends on  : [Primary Specialty] (source column)
Used by     : Visual filters, specialty grouping slicers
⚠️ NOTE: The exclusion list contains 53 values (including BLANK()).
   Any specialty NOT in this list is shown under its own name (not "Others").
   The Tableau group logic inverts this — listed values → "Others",
   unlisted values (e.g. Hematology-Oncology, Medical Oncology) → kept as-is.
   Do NOT add or remove values without confirming with the business owner.
Conversion Status: ✅ FULL


### CC-02 | Reach Denominator Flag
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1099159852059803648
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Origin:
───────────────
Calculated field — Reach Denominator Flag.
Returns "Y" for rows qualifying as the reach denominator universe:
  - Row has a valid call (Callkey not blank)
  - Product = OPDIVO_YERVOY_RCC
  - Year = 2025 (hardcoded)
  - Call type is face-to-face or remote e-detailing
  - Sales force team is in the approved list
  OR the HCP is a field target (fld_trgt_flg = "Y")

Power BI DAX:
─────────────
column 'Reach Denominator Flag' =
IF(
    (
        NOT(ISBLANK(rcc_ss_presc_master_db[Callkey 1]))
        && rcc_ss_presc_master_db[Primary Prod] = "OPDIVO_YERVOY_RCC"
        && rcc_ss_presc_master_db[full year] = "2025"
        && rcc_ss_presc_master_db[Presn Sbtyp Txt] IN {
            "FACE-TO-FACE",
            "REMOTE E-DETAILING"
        }
        && rcc_ss_presc_master_db[Sf Nm] IN {
            "ONC TEAM 1",
            "ONC TEAM 2",
            "ONC TEAM 3",
            "ONC TEAM 4",
            "CPS 5",
            "GI/GU ITS",
            "LUNG/MELANOMA ITS",
            "IO FLEX"
        }
    )
    || rcc_ss_presc_master_db[Fld Trgt Flg] = "Y",
    "Y",
    "N"
)

Depends on  : [Callkey 1], [Primary Prod], [full year] (CC-05),
              [Presn Sbtyp Txt], [Sf Nm], [Fld Trgt Flg]
Used by     : Reach denominator measures, visual-level filters
⚠️ NOTE: The year "2025" is hardcoded as a STRING (not integer).
   [full year] returns YEAR() as string — comparison must remain string.
   This hardcode is intentional business logic and requires a formal
   change request to update to a new year.
Conversion Status: ✅ FULL


### CC-03 | Refresh Date (calculated column)
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1890948930571083776
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Origin:
───────────────
Calculated field for the "(Refreshed as on ...)" display banner.
Finds the most recent Monday relative to TODAY() using a
MOD(WEEKDAY + 3, 7) rollback formula, then formats as "yyyy-MM-dd".

Power BI DAX:
─────────────
column 'Refresh Date' =
VAR CurrentDate = TODAY()
VAR weekday     = CurrentDate - MOD(WEEKDAY(CurrentDate, 1) + 3, 7)
RETURN
    "(Refreshed as on "
        & FORMAT(weekday, "yyyy-MM-dd")
        & ")"

Depends on  : TODAY() (volatile — recalculates on each refresh)
Used by     : Dashboard refresh banner display
⚠️ NOTE: This is a row-level calculated COLUMN on the main fact table,
   distinct from the Refresh_Info table measure (Section 5, M-02).
   Both coexist — different visuals may use either.
   This column uses Monday-rollback logic; the measure uses LocalNow().
Conversion Status: ✅ FULL


### CC-04 | full year
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_371265538056986624
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Origin:
───────────────
YEAR([date])
Simple year extraction from the primary date field.

Power BI DAX:
─────────────
column 'full year' =
YEAR(rcc_ss_presc_master_db[Date 1])

Depends on  : [Date 1] (source column — renamed from "date")
Used by     : CC-02 [Reach Denominator Flag] — compared as string "2025"
⚠️ NOTE: Despite YEAR() returning an integer, the dataType is declared
   as string in the PBIP TMDL. All downstream references use string
   comparison (e.g. = "2025"). Do NOT change to integer.
Conversion Status: ✅ FULL


### CC-05 | C13W Flag
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_758012178415218689
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Origin:
───────────────
Flag for the current 13-week window (weeks 1–13 of the year).

Power BI DAX:
─────────────
column 'C13W Flag' =
IF(
    'rcc_ss_presc_master_db'[Week No] <= 13,
    "C13W",
    BLANK()
)

Depends on  : [Week No] (source column, int64)
Used by     : Time-window filters for C13W trend visuals
Conversion Status: ✅ FULL


### CC-06 | P13W Flag
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_758012178415513602
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Origin:
───────────────
Flag for the prior 13-week window (weeks 14–26 of the year).

Power BI DAX:
─────────────
column 'P13W Flag' =
IF(
    'rcc_ss_presc_master_db'[Week No] >= 14
        && 'rcc_ss_presc_master_db'[Week No] <= 26,
    "P13W",
    BLANK()
)

Depends on  : [Week No] (source column, int64)
Used by     : Time-window filters for P13W trend visuals
Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — SOURCE COLUMN MAPPING (rcc_ss_presc_master_db)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All 68 source columns loaded. Naming conflicts resolved with suffix " 1".
Two columns intentionally NOT title-cased (preserved as-is from source).

Ord | Redshift Column                  | PBIP Column Name                    | Type      | summarizeBy | Notes
----|----------------------------------|-------------------------------------|-----------|-------------|------
 1  | bp_id                            | Bp Id                               | string    | none        |
 2  | npi_id                           | Npi Id                              | string    | none        |
 3  | hcp_nm                           | Hcp Nm                              | string    | none        | cast-to-local-type=true (SQL_WLONGVARCHAR)
 4  | prim_spty_code                   | Prim Spty Code                      | string    | none        |
 5  | sec_spty_code                    | Sec Spty Code                       | string    | none        |
 6  | hcp_address                      | Hcp Address                         | string    | none        |
 7  | hcp_classification               | Hcp Classification                  | string    | none        |
 8  | bp_status                        | Bp Status                           | string    | none        |
 9  | pdrp_ind                         | Pdrp Ind                            | string    | none        | width=4
10  | hcp_trgt_type_txt                | Hcp Trgt Type Txt                   | string    | none        |
11  | rcc_dynm_seg_nm                  | Rcc Dynm Seg Nm                     | string    | none        |
12  | hcp_city                         | Hcp City                            | string    | none        |
13  | hcp_state                        | Hcp State                           | string    | none        |
14  | hcp_zip                          | Hcp Zip                             | string    | none        |
15  | hcp_terr                         | Hcp Terr                            | string    | none        |
16  | hcp_dst                          | Hcp Dst                             | string    | none        |
17  | hcp_regn                         | Hcp Regn                            | string    | none        |
18  | hcp_affil_type                   | Hcp Affil Type                      | string    | none        | width=14
19  | rcc_value_score                  | Rcc Value Score                     | string    | none        | width=12
20  | fld_trgt_flg                     | Fld Trgt Flg                        | string    | none        | width=1; values: "Y"/"N"
21  | primary_specialty                | Primary Specialty                   | string    | none        |
22  | secondary_specialty              | Secondary Specialty                 | string    | none        |
23  | final_spec_cd                    | Final Spec Cd                       | string    | none        | width=10
24  | spec_group                       | Spec Group                          | string    | none        | width=6
25  | outlet_nm                        | Outlet Nm                           | string    | none        |
26  | outlet_address                   | Outlet Address                      | string    | none        |
27  | outlet_zip                       | Outlet Zip                          | string    | none        | width=10
28  | outlet_city                      | Outlet City                         | string    | none        |
29  | outlet_state                     | Outlet State                        | string    | none        |
30  | grp_id                           | Grp Id                              | string    | none        | width=100
31  | grp_nm                           | Grp Nm                              | string    | none        | width=150
32  | date                             | Date 1                              | dateTime  | none        | ⚠️ RENAMED — reserved word; Long Date; UnderlyingDateTimeDataType=Date
33  | yr_mnth                          | Yr Mnth                             | string    | none        | width=21
34  | source                           | Source 1                            | string    | none        | ⚠️ RENAMED — reserved word; width=3
35  | patient_id                       | Patient Id                          | string    | none        | width=20
36  | procedure_date                   | Procedure Date                      | dateTime  | none        | UnderlyingDateTimeDataType=Date
37  | clm_yr_mnth                      | Clm Yr Mnth                         | int64     | sum         |
38  | clm_weekending                   | Clm Weekending                      | int64     | sum         |
39  | week_no                          | Week No                             | int64     | sum         | Used in CC-05, CC-06
40  | month_no                         | Month No                            | int64     | sum         |
41  | r6w_flg                          | R6W Flg                             | string    | none        |
42  | day_name_clm                     | Day Name Clm                        | string    | none        |
43  | day_of_week_clm                  | Day Of Week Clm                     | int64     | sum         |
44  | clm_rank                         | Clm Rank                            | int64     | sum         |
45  | primary_prod                     | Primary Prod                        | string    | none        | Values include "OPDIVO_YERVOY_RCC"
46  | mutation                         | Mutation 1                          | string    | none        | ⚠️ RENAMED — conflict
47  | lot                              | Lot 1                               | string    | none        | ⚠️ RENAMED — conflict
48  | targeted_therapy_flag            | Targeted Therapy Flag               | string    | none        |
49  | nps_flg                          | Nps Flg                             | int64     | sum         |
50  | trialist_flg                     | Trialist Flg                        | int64     | sum         |
51  | repeater_flg                     | Repeater Flg                        | int64     | sum         |
52  | call_weekending                  | Call Weekending                     | int64     | sum         |
53  | call_week_no                     | call_week_no                        | int64     | sum         | ⚠️ NOT title-cased — preserved lowercase intentionally
54  | call_yr_mnth                     | Call Yr Mnth                        | int64     | sum         |
55  | call_product                     | Call Product                        | string    | none        |
56  | call_sub_type_cd                 | Call Sub Type Cd                    | string    | none        |
57  | presn_sbtyp_txt                  | Presn Sbtyp Txt                     | string    | none        | Values: "FACE-TO-FACE", "REMOTE E-DETAILING"
58  | call_position                    | Call Position                       | string    | none        |
59  | callkey                          | Callkey 1                           | int64     | sum         | ⚠️ RENAMED — conflict; formatString: "0.0"
60  | sf_cd                            | Sf Cd                               | string    | none        |
61  | sf_nm                            | Sf Nm                               | string    | none        | Values: "ONC TEAM 1"–"4","CPS 5","GI/GU ITS","LUNG/MELANOMA ITS","IO FLEX"
62  | call_territory                   | Call Territory                      | string    | none        |
63  | call_district                    | Call District                       | string    | none        |
64  | call_region                      | Call Region                         | string    | none        |
65  | rcc_segments                     | Rcc Segments                        | string    | none        | ⚠️ Null → "Null" in PQ; join key to RCC_Segment_Order
66  | subc_flag                        | subc_flag                           | string    | none        | ⚠️ NOT title-cased — preserved lowercase intentionally
67  | sc_index_1_flagged_hcps          | Sc Index 1 Flagged Hcps             | string    | none        |
68  | past_sc_experience_volume_claims | Past Sc Experience Volume Claims    | string    | none        |

Naming Conflict Registry (columns renamed with " 1" suffix):
  date           → Date 1
  source         → Source 1
  mutation       → Mutation 1
  lot            → Lot 1
  callkey        → Callkey 1

Intentional Lowercase (NOT renamed):
  call_week_no   → call_week_no   (remains all-lowercase, no spaces)
  subc_flag      → subc_flag      (remains all-lowercase, no spaces)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: Refresh_Info

### M-01 | Refresh Date Text
─────────────────────────────────────────────────────────────
PBI Table       : Refresh_Info
PBI Lineage Tag : 81ad16ee-339c-4cec-b135-cbb74cbf2856
PBI Format      : (text string)

Tableau Origin:
───────────────
Display measure for the refresh date banner in the dashboard header.
Equivalent to the [Refresh Date] calculated column (CC-03) but sourced
from the Refresh_Info single-row table (DateTime.LocalNow()).

Power BI DAX:
─────────────
measure 'Refresh Date Text' =
"(Refreshed as on "
    & FORMAT(MAX(Refresh_Info[RefreshDate]), "yyyy-mm-dd")
    & ")"

⚠️ NOTE: Format string is "yyyy-mm-dd" (lowercase mm = minutes, NOT months).
   This matches the original author's formula verbatim.
   Do NOT "fix" to "yyyy-MM-dd" without confirming with the business owner —
   the banner text may display minute values instead of months.
Conversion Status: ✅ FULL (preserved as-is including format anomaly)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — HIERARCHY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### HIER-01 | Hcp Regn, Hcp Zip
─────────────────────────────────────────────────────────────
PBI Table       : rcc_ss_presc_master_db
PBI lineageTag  : b5cf7b95-0e7f-4f56-a090-cff575f27858
PBI Name        : Hcp Regn, Hcp Zip

Tableau Origin:
───────────────
Drill path defined in Tableau: hcp_regn → hcp_zip
Used in geographic drill-down on map/table visuals.

PBIP Level Definition:
──────────────────────
Level 1 (inner) : hcp_zip  — lineageTag: af55be3f-9b7d-4a6b-8541-add3fd4070c8
Level 2 (outer) : hcp_regn — lineageTag: 431238ca-bb75-4f0e-a54b-b9c4e6299a79

⚠️ NOTE: Level order is hcp_zip first, hcp_regn second — this matches
   Tableau's inner-to-outer drill path direction. Do NOT reverse.
Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — RELATIONSHIPS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

REL-01
─────────────────────────────────────────────────────────────
GUID          : 6bad4888-04de-68b4-df98-1eb636c3e7ad
From Table    : rcc_ss_presc_master_db
From Column   : Rcc Segments
To Table      : RCC_Segment_Order
To Column     : Rcc Segments
Cardinality   : Many-to-One (default)
Cross-filter  : Single
Purpose       : Drives sort order on segment slicers and visuals
                via sortByColumn: SortOrder on RCC_Segment_Order[Rcc Segments]

Total relationships: 1


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — AUXILIARY TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### AUX-01 | Refresh_Info
─────────────────────────────────────────────────────────────
PBI Table       : Refresh_Info
PBI lineageTag  : 138c6c81-3ab4-4162-9902-3c12aad86964
PBI Type        : M table, single-row, import mode
Purpose         : Provides current refresh timestamp for display

Column:
  RefreshDate   dateTime (General Date, summarizeBy: none)

Power BI M:
───────────
let
    Source = #table(
        type table [RefreshDate = datetime],
        {{DateTime.LocalNow()}}
    )
in
    Source

Conversion Status: ✅ FULL


### AUX-02 | Measures Table
─────────────────────────────────────────────────────────────
PBI Table       : Measures Table
PBI lineageTag  : 6af4a663-5a54-42f5-a209-bd64ef19638d
PBI Type        : M table (empty row container), import mode
Column          : Column1 (string, isHidden)

⚠️ NOTE: This Measures Table contains ~35 carry-over measures from
   a DIFFERENT workbook (SDOP Capacity & Slot Allocations / Patient Ops).
   These measures reference tables not present in this model
   (e.g. 'SDOP Capacity Slot Allocation_Custom SQL Query', 'Date',
   'Month Range (2)', 'Aph Include Re-Aphs', 'Aph Allocation Snapshot').
   They are structural debt from a shared PBIP template.
   PRESERVE all carry-over measures as-is for structural fidelity.
   Do NOT surface any carry-over measure in the RCC dashboard report layer.

Carry-over measures (do not expose in RCC report layer):
  Refresh Timestamp, Definitions, Row Number,
  Allocated / Allocatedv1 / Allocatedv2 / Allocatedv3,
  Used, Consumed / Consumedv1 / Consumedv2 / Consumedv3,
  Unconsumed / Unconsumedv1 / Unconsumedv2 / Unconsumedv3 / Unconsumedv4,
  Cancels / Cancelsv1 / Cancelsv2 / Cancelsv3,
  Capacities / Capacityv1 / Capacityv2 / Capacityv3,
  Slot Utilization, Allocation Accuracyv1,
  Val / Valv1 / val2 / val3 / Val1_1 / Valv1_2 / val2_1 / val3_1,
  yyy / yyy1 / yyy_2, measureselection, Showanalysis,
  Title1 / Title2 / Title3 / Title4,
  ff, sel, sel1, slicerfil, filcheckk, b, consum1

Conversion Status: ✅ PRESERVED (carry-over measures retained structurally)


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — KNOWN BEHAVIORS, AUDIT NOTES & FLAGS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FLAG-01 | rcc_segments null handling
─────────────────────────────────────────────────────────────
Description : Redshift nulls in rcc_segments are replaced with the
              string "Null" in the Power Query M step before load.
              This pairs with the RCC_Segment_Order DATATABLE which
              uses "Null" (string) as a valid key at SortOrder = 1.
Action      : PRESERVE — do not remove the ReplaceValue step.
              Do not change "Null" to BLANK(), empty string, or "null".


FLAG-02 | Reach Denominator Flag year hardcode
─────────────────────────────────────────────────────────────
Description : The [Reach Denominator Flag] column (CC-02) contains
              [full year] = "2025" hardcoded as a string literal.
              This is intentional business logic for the 2025 reporting year.
Action      : PRESERVE as-is. Requires a formal change request to update
              to a future year. Note that [full year] is a string type —
              any update must use string comparison (e.g. = "2026").


FLAG-03 | Refresh Date Text format anomaly
─────────────────────────────────────────────────────────────
Description : The [Refresh Date Text] measure (M-01) uses format string
              "yyyy-mm-dd" where mm = minutes (not months). This is
              preserved verbatim from the original author's formula.
              The banner may display minute values in the "month" position.
Action      : PRESERVE as-is. Flag for business owner review if the banner
              text appears incorrect in production.


FLAG-04 | Dual refresh display mechanisms
─────────────────────────────────────────────────────────────
Description : Two separate refresh display mechanisms coexist:
              (a) CC-03 [Refresh Date] column on the main table —
                  Monday-rollback logic via MOD(WEEKDAY+3,7), format "yyyy-MM-dd"
              (b) M-01 [Refresh Date Text] measure on Refresh_Info —
                  MAX(RefreshDate) from DateTime.LocalNow(), format "yyyy-mm-dd"
              Different visuals in the report may use either.
Action      : PRESERVE both. Do not consolidate without confirming which
              visual uses which mechanism.


FLAG-05 | full year data type (string vs integer)
─────────────────────────────────────────────────────────────
Description : [full year] (CC-04) uses YEAR() which returns an integer,
              but the PBIP column dataType is declared as string.
              All downstream DAX references compare it as a string (= "2025").
Action      : PRESERVE string type. Do not change to int64.


FLAG-06 | Intentional lowercase column names
─────────────────────────────────────────────────────────────
Description : Two columns are NOT title-cased, matching their source names:
              - call_week_no  (all lowercase, no spaces)
              - subc_flag     (all lowercase, no spaces)
              All other source columns follow Title Case in PBIP.
Action      : PRESERVE exact casing. Any DAX or report reference must use
              the lowercase names.


FLAG-07 | Carry-over Measures Table measures
─────────────────────────────────────────────────────────────
Description : ~35 measures from the SDOP / Capacity workbook are present
              in the Measures Table. They reference tables not in this model.
              They will produce errors if evaluated in an RCC context.
Action      : PRESERVE structurally. Do not expose in the RCC report layer.
              Mark as hidden or move to a separate dev-only display folder
              if the report layer is rebuilt.


FLAG-08 | No time intelligence / no auto date tables
─────────────────────────────────────────────────────────────
Description : __PBI_TimeIntelligenceEnabled = 0.
              Power BI does not auto-generate date tables for any date columns.
              All date-related logic (week flags, month labels) is handled
              via source columns (week_no, month_no, yr_mnth, clm_yr_mnth)
              and calculated columns (CC-04, CC-05, CC-06).
Action      : Do not enable time intelligence without a full impact assessment
              on CC-05 and CC-06 week-number logic.


FLAG-09 | Import mode (not DirectQuery)
─────────────────────────────────────────────────────────────
Description : Unlike most BMS SAP HANA workbooks, this model uses
              Redshift Import mode. No TREATAS locked-filter pattern
              is required. All filters are report/visual level or
              implemented as calculated columns.
Action      : No special handling needed. Confirm Redshift credentials
              and gateway configuration on deployment.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — FILE REFERENCE
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

File                                              | Content
--------------------------------------------------|--------------------------------------------
28341_brand_analytics_rcc_self_serve_dashboard.txt | Tableau workbook XML source
9_database.txt                                    | PBIP database definition (compatibilityLevel 1604)
9_model.txt                                       | PBIP model definition, table refs, annotations
9_relationships.txt                               | PBIP relationship definitions
9_definition.json                                 | PBIP definition schema v4.2
rcc_ss_presc_master_db.txt                        | PBIP main fact table TMDL (68 cols + 6 calc cols)
rcc_segment_order.txt                             | PBIP segment sort DATATABLE TMDL
refresh_info.txt                                  | PBIP refresh timestamp table TMDL
1_measures_table.txt                              | PBIP measures table TMDL (carry-over measures)

---

# End of conversion knowledge document
# 28341 – Brand Analytics RCC Self-Serve Dashboard
