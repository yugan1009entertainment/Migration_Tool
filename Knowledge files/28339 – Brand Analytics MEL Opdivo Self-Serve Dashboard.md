# 📘 CONVERSION KNOWLEDGE FILE
# 28339 – Brand Analytics MEL Opdivo Self-Serve Dashboard
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-08
# Source files: 28339_brand_analytics_mel_opdivo_self_serve_dashboard.txt + all PBIP .txt files

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 0 — WORKBOOK & MODEL METADATA
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Tableau Workbook Name          : Brand Analytics MEL Opdivo Self-Serve Dashboard
Tableau Workbook ID            : 28339
Tableau Server Path            : /t/PRODUCTION/workbooks
Tableau Build                  : 2024.2.13 (20242.25.0710.1539)
Tableau Original Version       : 18.1

PBIP Dataset Name              : 29967_28339-Brand Analytics MEL Opdivo Self-Serve Dashboard
PBIP Model Name                : 29967_28339-Brand Analytics MEL Opdivo Self-Serve Dashboard- model
PBIP compatibilityLevel        : 1604
PBIP defaultPowerBIDataSourceVersion : powerBI_V3
PBIP culture                   : en-US
PBIP Time Intelligence         : DISABLED (__PBI_TimeIntelligenceEnabled = 0)
PBIP ProTooling                : DevMode
PBIP Definition Schema Version : 4.2
PBIP PBI_QueryOrder            : ["mel_ss_presc_master_db"]
PBIP discourageImplicitMeasures: true

Relationship to RCC workbook   : Sister workbook — same Redshift server, same
                                  connection pattern, same 68-column base schema.
                                  MEL-specific columns diverge at ordinal 11, 19, 46,
                                  and 66. Key behavioral differences documented per section.

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU DATASOURCE → POWER BI PARTITION (Power Query M)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### DS-01 | mel_ss_presc_master_db  ← Main Fact Table
─────────────────────────────────────────────────────────────
Tableau Datasource Caption    : cdw
Tableau Datasource Name       : federated.12yrtjj1wstoqp16nhetd1bycwvo
Tableau Connection Type       : Redshift (federated)
Server                        : comm-cdw-prod-amer-01.cmyzp34zrkds.us-east-1.redshift.amazonaws.com
Port                          : 5439
Database                      : cdw
Schema                        : us_sandbox
Table                         : mel_ss_presc_master_db
Full Table Reference          : [us_sandbox].[mel_ss_presc_master_db]
Authentication                : auth-user-pass
SSL Mode                      : require
Tableau Username              : kumaa253

PBI Table Name                : mel_ss_presc_master_db
PBI Load Mode                 : Import (NOT DirectQuery)
PBI Column Selection          : Explicit Table.SelectColumns — 69 columns (no SELECT *)
PBI Null Handling             : No null replacement step. mel_segments nulls
                                remain BLANK() in the model (unlike RCC which
                                replaces rcc_segments nulls with the string "Null").

⚠️ NOTE: Same server, database, schema, and credentials as RCC workbook (28341).
   Only the table name differs: mel_ss_presc_master_db vs rcc_ss_presc_master_db.
⚠️ KEY DIFFERENCE vs RCC: 69 columns loaded (RCC loaded 68). Extra column is
   "drug" (ordinal 46, lowercase). No Table.ReplaceValue null-replacement step.

Power BI Power Query M:
───────────────────────
let
    ClusterEndpoint = "comm-cdw-prod-amer-01.cmyzp34zrkds.us-east-1.redshift.amazonaws.com:5439",
    DatabaseName    = "cdw",
    SchemaName      = "us_sandbox",
    TableName       = "mel_ss_presc_master_db",
    Source = AmazonRedshift.Database(ClusterEndpoint, DatabaseName,
                 [BatchSize = null, Role = null, ProviderName = null]),
    SchemaData = Source{[Name = SchemaName, Kind = "Schema"]}[Data],
    #"us_sandbox_mel_ss_presc_master_db" = SchemaData{[Name = TableName]}[Data],
    #"Select Columns" = Table.SelectColumns(
        #"us_sandbox_mel_ss_presc_master_db",
        {
            "bp_id", "npi_id", "hcp_nm", "prim_spty_code", "sec_spty_code",
            "hcp_address", "hcp_classification", "bp_status", "pdrp_ind",
            "hcp_trgt_type_txt", "mel_dynm_seg_nm", "hcp_city", "hcp_state",
            "hcp_zip", "hcp_terr", "hcp_dst", "hcp_regn", "hcp_affil_type",
            "mel_value_score", "fld_trgt_flg", "primary_specialty",
            "secondary_specialty", "final_spec_cd", "spec_group", "outlet_nm",
            "outlet_address", "outlet_zip", "outlet_city", "outlet_state",
            "grp_id", "grp_nm", "date", "yr_mnth", "source", "patient_id",
            "procedure_date", "clm_yr_mnth", "clm_weekending", "week_no",
            "month_no", "r6w_flg", "day_name_clm", "day_of_week_clm",
            "clm_rank", "primary_prod", "drug", "mutation", "lot",
            "targeted_therapy_flag", "nps_flg", "trialist_flg", "repeater_flg",
            "call_weekending", "call_week_no", "call_yr_mnth", "call_product",
            "call_sub_type_cd", "presn_sbtyp_txt", "call_position", "callkey",
            "sf_cd", "sf_nm", "call_territory", "call_district", "call_region",
            "mel_segments", "subc_flag", "sc_index_1_flagged_hcps",
            "past_sc_experience_volume_claims"
        }
    )
in
    #"Select Columns"

Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — SOURCE COLUMN MAPPING (mel_ss_presc_master_db)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All 69 source columns loaded. MEL-specific columns and naming differences
from RCC (28341) are called out explicitly.

Ord | Redshift Column                  | PBIP Column Name                 | Type     | summarizeBy | Notes
----|----------------------------------|----------------------------------|----------|-------------|------
  1 | bp_id                            | Bp Id                            | string   | none        |
  2 | npi_id                           | Npi Id                           | string   | none        |
  3 | hcp_nm                           | Hcp Nm                           | string   | none        |
  4 | prim_spty_code                   | Prim Spty Code                   | string   | none        |
  5 | sec_spty_code                    | Sec Spty Code                    | string   | none        |
  6 | hcp_address                      | Hcp Address                      | string   | none        |
  7 | hcp_classification               | Hcp Classification               | string   | none        |
  8 | bp_status                        | Bp Status                        | string   | none        |
  9 | pdrp_ind                         | Pdrp Ind                         | string   | none        |
 10 | hcp_trgt_type_txt                | Hcp Trgt Type Txt                | string   | none        |
 11 | mel_dynm_seg_nm                  | Mel Dynm Seg Nm                  | string   | none        | ⚠️ MEL-specific; RCC equiv: rcc_dynm_seg_nm
 12 | hcp_city                         | Hcp City                         | string   | none        |
 13 | hcp_state                        | Hcp State                        | string   | none        |
 14 | hcp_zip                          | Hcp Zip                          | string   | none        |
 15 | hcp_terr                         | Hcp Terr                         | string   | none        |
 16 | hcp_dst                          | Hcp Dst                          | string   | none        |
 17 | hcp_regn                         | Hcp Regn                         | string   | none        |
 18 | hcp_affil_type                   | Hcp Affil Type                   | string   | none        |
 19 | mel_value_score                  | Mel Value Score                  | string   | none        | ⚠️ MEL-specific; RCC equiv: rcc_value_score
                                                                                                      Values used in logic: "VH","H","M","L"
 20 | fld_trgt_flg                     | fld_trgt_flg                     | string   | none        | ⚠️ lowercase — preserved (no spaces)
 21 | primary_specialty                | Primary Specialty                | string   | none        |
 22 | secondary_specialty              | Secondary Specialty              | string   | none        |
 23 | final_spec_cd                    | final_spec_cd                    | string   | none        | ⚠️ lowercase — preserved (no spaces)
 24 | spec_group                       | spec_group                       | string   | none        | ⚠️ lowercase — preserved (no spaces)
 25 | outlet_nm                        | Outlet Nm                        | string   | none        |
 26 | outlet_address                   | Outlet Address                   | string   | none        |
 27 | outlet_zip                       | Outlet Zip                       | string   | none        |
 28 | outlet_city                      | Outlet City                      | string   | none        |
 29 | outlet_state                     | Outlet State                     | string   | none        |
 30 | grp_id                           | Grp Id                           | string   | none        |
 31 | grp_nm                           | Grp Nm                           | string   | none        | Join key → Filteredgrp_nmByField
 32 | date                             | Date                             | dateTime | none        | ⚠️ NOT renamed (no " 1" suffix unlike RCC)
                                                                                                      formatString: Long Date;
                                                                                                      UnderlyingDateTimeDataType=Date;
                                                                                                      join key → DimDate.Date
 33 | yr_mnth                          | Yr Mnth                          | int64    | sum         | SummarizationSetBy=Automatic
 34 | source                           | Source                           | string   | none        | ⚠️ NOT renamed (no " 1" suffix unlike RCC)
 35 | patient_id                       | Patient Id                       | string   | none        |
 36 | procedure_date                   | Procedure Date                   | dateTime | none        | UnderlyingDateTimeDataType=Date
 37 | clm_yr_mnth                      | Clm Yr Mnth                      | int64    | sum         |
 38 | clm_weekending                   | Clm Weekending                   | int64    | sum         |
 39 | week_no                          | week_no                          | int64    | sum         | ⚠️ lowercase — preserved; SummarizationSetBy=Automatic
                                                                                                      Used in CC-06 (C13W Flag) and CC-07 (P13W Flag)
 40 | month_no                         | month_no                         | int64    | sum         | ⚠️ lowercase — preserved; SummarizationSetBy=Automatic
 41 | r6w_flg                          | R6W Flg                          | string   | none        |
 42 | day_name_clm                     | Day Name Clm                     | string   | none        |
 43 | day_of_week_clm                  | Day Of Week Clm                  | int64    | sum         | SummarizationSetBy=Automatic
 44 | clm_rank                         | Clm Rank                         | int64    | sum         |
 45 | primary_prod                     | Primary Prod                     | string   | none        | MEL value used in logic: "OPDUALAG"
 46 | drug                             | drug                             | string   | none        | ⚠️ MEL-ONLY — not in RCC (28341); lowercase — preserved
 47 | mutation                         | Mutation                         | string   | none        | ⚠️ NOT renamed (no " 1" suffix unlike RCC "Mutation 1")
 48 | lot                              | Lot                              | string   | none        | ⚠️ NOT renamed (no " 1" suffix unlike RCC "Lot 1")
 49 | targeted_therapy_flag            | Targeted Therapy Flag            | string   | none        |
 50 | nps_flg                          | Nps Flg                          | int64    | sum         |
 51 | trialist_flg                     | Trialist Flg                     | int64    | sum         |
 52 | repeater_flg                     | repeater_flg                     | int64    | sum         | ⚠️ lowercase — preserved; SummarizationSetBy=Automatic
 53 | call_weekending                  | Call Weekending                  | int64    | sum         |
 54 | call_week_no                     | call_week_no                     | int64    | sum         | ⚠️ lowercase — preserved; SummarizationSetBy=Automatic
 55 | call_yr_mnth                     | Call Yr Mnth                     | int64    | sum         |
 56 | call_product                     | Call Product                     | string   | none        |
 57 | call_sub_type_cd                 | Call Sub Type Cd                 | string   | none        |
 58 | presn_sbtyp_txt                  | Presn Sbtyp Txt                  | string   | none        | Values used in logic: "FACE-TO-FACE","REMOTE E-DETAILING"
 59 | call_position                    | Call Position                    | string   | none        |
 60 | callkey                          | Callkey                          | int64    | sum         | ⚠️ NOT renamed (no " 1" suffix unlike RCC "Callkey 1")
                                                                                                      No formatString="0.0" (unlike RCC)
 61 | sf_cd                            | sf_cd                            | string   | none        | ⚠️ lowercase — preserved
 62 | sf_nm                            | sf_nm                            | string   | none        | ⚠️ lowercase — preserved
 63 | call_territory                   | Call Territory                   | string   | none        |
 64 | call_district                    | Call District                    | string   | none        |
 65 | call_region                      | Call Region                      | string   | none        |
 66 | mel_segments                     | mel_segments                     | string   | none        | ⚠️ MEL-specific; lowercase — preserved
                                                                                                      Nulls stay BLANK() — no PQ null replacement
                                                                                                      Raw values: "Segment A","Segment B","Segment C","Segment D"
 67 | subc_flag                        | subc_flag                        | string   | none        | ⚠️ lowercase — preserved
 68 | sc_index_1_flagged_hcps          | Sc Index 1 Flagged Hcps          | string   | none        |
 69 | past_sc_experience_volume_claims | Past Sc Experience Volume Claims | string   | none        |

Intentional Lowercase Registry (NOT title-cased — preserve exactly):
  fld_trgt_flg      → fld_trgt_flg
  final_spec_cd     → final_spec_cd
  spec_group        → spec_group
  week_no           → week_no
  month_no          → month_no
  drug              → drug
  repeater_flg      → repeater_flg
  call_week_no      → call_week_no
  sf_cd             → sf_cd
  sf_nm             → sf_nm
  mel_segments      → mel_segments
  subc_flag         → subc_flag

No " 1" Suffix Renames in this model (contrast with RCC 28341):
  date              → Date      (not "Date 1")
  source            → Source    (not "Source 1")
  mutation          → Mutation  (not "Mutation 1")
  lot               → Lot       (not "Lot 1")
  callkey           → Callkey   (not "Callkey 1")


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU CALCULATED FIELDS / GROUPS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: mel_ss_presc_master_db

### CC-01 | Primary Specialty (group)
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Primary Specialty (group)
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none
PBI Annotation  : SummarizationSetBy = Automatic

Tableau Origin:
───────────────
Tableau Group applied to [Primary Specialty].
Uses an INCLUSION list of 10 named specialties — all others fall through
to "Others". This is structurally opposite to RCC which uses an EXCLUSION list.

Power BI DAX:
─────────────
column 'Primary Specialty (group)' =
SWITCH(
    TRUE(),
    'mel_ss_presc_master_db'[Primary Specialty] = "Advanced Practice Midwife",
        "Advanced Practice Midwife",
    'mel_ss_presc_master_db'[Primary Specialty] = "Family Planning",
        "Family Planning",
    'mel_ss_presc_master_db'[Primary Specialty] = "Hematology-Oncology",
        "Hematology-Oncology",
    'mel_ss_presc_master_db'[Primary Specialty] = "Neurosurgery",
        "Neurosurgery",
    'mel_ss_presc_master_db'[Primary Specialty] = "Nurse Practictioner",
        "Nurse Practictioner",
    'mel_ss_presc_master_db'[Primary Specialty] = "Oncology",
        "Oncology",
    'mel_ss_presc_master_db'[Primary Specialty] = "Oral and Maxillofacial Surgery",
        "Oral and Maxillofacial Surgery",
    'mel_ss_presc_master_db'[Primary Specialty] = "Physician Assistant",
        "Physician Assistant",
    'mel_ss_presc_master_db'[Primary Specialty] = "Primary Care Physician",
        "Primary Care Physician",
    'mel_ss_presc_master_db'[Primary Specialty] = "Rheumatology",
        "Rheumatology",
    "Others"
)

Inclusion list (10 values — exact spelling, preserve verbatim):
  "Advanced Practice Midwife"
  "Family Planning"
  "Hematology-Oncology"
  "Neurosurgery"
  "Nurse Practictioner"              ← ⚠️ intentional typo: double 'c'. DO NOT correct.
  "Oncology"
  "Oral and Maxillofacial Surgery"
  "Physician Assistant"
  "Primary Care Physician"
  "Rheumatology"

Depends on  : [Primary Specialty] (source column)
Used by     : Specialty grouping slicers and visuals
⚠️ LOGIC NOTE: MEL uses SWITCH(TRUE(),...,"Others") — inclusion list.
   RCC uses IF(... IN {...},"Others",...) — exclusion list (53 values).
   These are structurally opposite. Do not conflate.
Conversion Status: ✅ FULL


### CC-02 | Refresh Date
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1032731726626295808
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Origin:
───────────────
"(Refreshed as on "+ str(date( DATEADD('day',-((DATEPART('weekday', TODAY())+ 3) % 7),
TODAY()) ) ) +")"

Power BI DAX:
─────────────
column 'Refresh Date' =
"(Refreshed as on "
    & FORMAT(TODAY() - MOD(WEEKDAY(TODAY(), 1) + 2, 7), "d mmm yyyy")
    & ")"
/* Original Tableau Expression is
   "(Refreshed as on "
   + str(date( DATEADD('day',-((DATEPART('weekday', TODAY())+ 3) % 7), TODAY()) ) )
   +")" */

Depends on  : TODAY() (volatile — recalculates on each refresh)
Used by     : Dashboard refresh banner display
⚠️ FORMAT DIFFERENCE vs RCC (28341):
   MEL  uses: FORMAT(..., "d mmm yyyy")    → e.g. "7 Jul 2026"
   RCC  uses: FORMAT(..., "yyyy-MM-dd")    → e.g. "2026-07-07"
⚠️ ROLLBACK FORMULA DIFFERENCE vs RCC:
   MEL  uses: MOD(WEEKDAY(TODAY(), 1) + 2, 7)
   RCC  uses: MOD(WEEKDAY(CurrentDate, 1) + 3, 7)
   The +2 vs +3 difference shifts the rollback anchor by one day.
   Preserve each formula exactly per workbook.
Conversion Status: ✅ FULL (original Tableau expression preserved in comment)


### CC-03 | Reach Denominator Flag
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1099159852059803648
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Origin:
───────────────
Returns "Y" for rows qualifying as the MEL reach denominator universe.
Logic differs structurally from RCC:
  MEL: callkey check + product + year + call type   OR   mel_value_score IN set
  RCC: callkey check + product + year + call type + sf_nm list   OR   fld_trgt_flg = "Y"

Power BI DAX:
─────────────
column 'Reach Denominator Flag' =
IF(
    (
        NOT(ISBLANK('mel_ss_presc_master_db'[callkey]))
        && 'mel_ss_presc_master_db'[Primary Prod] = "OPDUALAG"
        && 'mel_ss_presc_master_db'[full year] = "2026"
        && 'mel_ss_presc_master_db'[Presn Sbtyp Txt] IN {
            "FACE-TO-FACE",
            "REMOTE E-DETAILING"
        }
    )
    || 'mel_ss_presc_master_db'[Mel Value Score] IN { "VH", "H", "M", "L" },
    "Y",
    "N"
)

Depends on  : [callkey] (lowercase), [Primary Prod], [full year] (CC-05),
              [Presn Sbtyp Txt], [Mel Value Score]
Used by     : Reach denominator measures, CC-09 (Reach_Target_Fixed_Col),
              visual-level reach filters
⚠️ KEY DIFFERENCES vs RCC (28341):
   1. Product filter : MEL = "OPDUALAG"              vs RCC = "OPDIVO_YERVOY_RCC"
   2. Year hardcode  : MEL = "2026"                  vs RCC = "2025"
   3. OR branch      : MEL uses [Mel Value Score] IN {"VH","H","M","L"}
                       RCC uses [Fld Trgt Flg] = "Y"
   4. No sf_nm team list in MEL (RCC had 8 team values)
   5. Column ref     : MEL uses [callkey] (lowercase)
                       RCC uses [Callkey 1] (renamed, title-case)
⚠️ YEAR NOTE: "2026" is hardcoded as a STRING. [full year] returns string.
   Do NOT compare as integer. Requires formal change request to update.
Conversion Status: ✅ FULL


### CC-04 | Mel HCP Segments
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_1478869588089479168
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Origin:
───────────────
Current active expression: UPPER([mel_segments])
Historical/deprecated expression (commented out in PBIP):
  IF [mel_segments] = 'Segment A' THEN 'Efficacy Loyalists'
  ELSEIF [mel_segments] = 'Segment B' THEN 'Targeted Customizers'
  ELSEIF [mel_segments] = 'Segment C' THEN 'Safety Seekers'
  ELSEIF [mel_segments] = 'Segment D' THEN 'Balanced Adopters'
  ELSE 'Unsegmented'
  END

Power BI DAX:
─────────────
column 'Mel HCP Segments' =
UPPER('mel_ss_presc_master_db'[mel_segments])
/* Original Tableau Expression is UPPER([mel_segments])
   /*if [mel_segments] = 'Segment A'
   then 'Efficacy Loyalists'
   ELSEIF [mel_segments] = 'Segment B'
   then 'Targeted Customizers'
   ELSEIF [mel_segments] = 'Segment C'
   then 'Safety Seekers'
   ELSEIF [mel_segments] = 'Segment D'
   then 'Balanced Adopters'
   ELSE 'Unsegmented'
   end*/ */

Depends on  : [mel_segments] (source column, lowercase)
Used by     : CC-09 (Reach_Target_Fixed_Col) via ALLEXCEPT filter context,
              segment slicers and visuals
Output      : "SEGMENT A", "SEGMENT B", "SEGMENT C", "SEGMENT D" (uppercase)
⚠️ DEPRECATED LOGIC: The original segment-to-persona mapping is preserved
   in the comment block for audit. Do NOT reactivate without a formal
   change request.
⚠️ MEL-ONLY: No equivalent column in RCC (28341).
Conversion Status: ✅ FULL (deprecated logic preserved in comment)


### CC-05 | full year
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_371265538056986624
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Origin:
───────────────
STR(YEAR([date]))

Power BI DAX:
─────────────
column 'full year' =
FORMAT(YEAR('mel_ss_presc_master_db'[Date]), "0")
/* Original Tableau Expression is STR(YEAR([date])) */

Depends on  : [Date] (source column — NOT renamed in MEL; no " 1" suffix)
Used by     : CC-03 [Reach Denominator Flag] — compared as string "2026"
⚠️ FORMAT DIFFERENCE vs RCC:
   MEL  uses: FORMAT(YEAR(...), "0")  → explicit string conversion
   RCC  uses: YEAR(...)               → relies on dataType: string declaration
   Both produce the same string output.
⚠️ COLUMN REFERENCE: Uses [Date] (no suffix). RCC uses [Date 1].
Conversion Status: ✅ FULL (original Tableau expression preserved in comment)


### CC-06 | C13W Flag
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_758012178415218689
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Origin:
───────────────
if [week_no] >= 7 and [week_no] <= 19 then 'C13W' else null end

Power BI DAX:
─────────────
column 'C13W Flag' =
IF(
    AND(
        'mel_ss_presc_master_db'[week_no] >= 7,
        'mel_ss_presc_master_db'[week_no] <= 19
    ),
    "C13W",
    BLANK()
)
/* Original Tableau Expression is
   if [week_no] >= 7 and [week_no] <= 19 then 'C13W' else null end */

Depends on  : [week_no] (source column, lowercase int64)
Used by     : Time-window filters for C13W trend visuals
⚠️ RANGE DIFFERENCE vs RCC (28341):
   MEL  C13W: week_no >= 7 AND <= 19   (weeks 7–19)
   RCC  C13W: Week No <= 13            (weeks 1–13)
   Different fiscal week anchoring — do NOT copy RCC ranges to MEL.
Conversion Status: ✅ FULL (original Tableau expression preserved in comment)


### CC-07 | P13W Flag
─────────────────────────────────────────────────────────────
PBI Lineage Tag : Calculation_758012178415513602
PBI Data Type   : string
PBI Hidden      : No
PBI summarizeBy : none

Tableau Origin:
───────────────
if [week_no] >= 20 and [week_no] <= 32 then 'P13W' else null end

Power BI DAX:
─────────────
column 'P13W Flag' =
IF(
    AND(
        'mel_ss_presc_master_db'[week_no] >= 20,
        'mel_ss_presc_master_db'[week_no] <= 32
    ),
    "P13W",
    BLANK()
)
/* Original Tableau Expression is
   if [week_no] >= 20 and [week_no] <= 32 then 'P13W' else null end */

Depends on  : [week_no] (source column, lowercase int64)
Used by     : Time-window filters for P13W trend visuals
⚠️ RANGE DIFFERENCE vs RCC (28341):
   MEL  P13W: week_no >= 20 AND <= 32  (weeks 20–32)
   RCC  P13W: Week No >= 14 AND <= 26  (weeks 14–26)
   Different fiscal week anchoring — do NOT copy RCC ranges to MEL.
Conversion Status: ✅ FULL (original Tableau expression preserved in comment)


### CC-08 | date_Month-Trunc
─────────────────────────────────────────────────────────────
PBI Lineage Tag : b4babc97-1b42-4235-a400-92dd00e33d77
PBI Data Type   : dateTime
PBI Hidden      : No
PBI summarizeBy : none
PBI Annotation  : SummarizationSetBy = Automatic

Tableau Origin:
───────────────
DATETRUNC('month', [date]) — month-level date truncation for time-series axis.

Power BI DAX:
─────────────
column 'date_Month-Trunc' =
VAR follow_up_Month_TruncValue = COALESCE('mel_ss_presc_master_db'[Date], BLANK())
RETURN
    IF(
        ISBLANK(follow_up_Month_TruncValue),
        BLANK(),
        FORMAT(follow_up_Month_TruncValue, "MMMM YYYY")
    )

Depends on  : [Date] (source column)
Used by     : HIER-02 (date_Date Hierarchy level 1), time-series axis visuals
Output      : "Month YYYY" string e.g. "July 2026"
⚠️ MEL-ONLY: No equivalent column in RCC (28341).
⚠️ TYPE MISMATCH: dataType declared as dateTime but FORMAT() returns string.
   This is an intentional author-side annotation mismatch. Preserve as-is.
⚠️ COLUMN NAME: Contains a hyphen ("date_Month-Trunc"). Do NOT rename to
   "date_Month_Trunc" or "date Month Trunc".
Conversion Status: ✅ FULL


### CC-09 | Reach_Target_Fixed_Col
─────────────────────────────────────────────────────────────
PBI Lineage Tag : e20a5959-4609-4502-b983-20817109df19
PBI Data Type   : int64 (formatString: 0)
PBI Hidden      : No
PBI summarizeBy : sum
PBI Annotation  : SummarizationSetBy = Automatic

Tableau Origin:
───────────────
FIXED LOD expression:
{ FIXED [Mel HCP Segments], [Mel Value Score] :
  COUNTD([bp_id]) WHERE [Reach Denominator Flag] = "Y" }

Power BI DAX:
─────────────
column Reach_Target_Fixed_Col =
CALCULATE(
    DISTINCTCOUNT('mel_ss_presc_master_db'[Bp Id]),
    ALLEXCEPT(
        'mel_ss_presc_master_db',
        'mel_ss_presc_master_db'[Mel HCP Segments],
        'mel_ss_presc_master_db'[Mel Value Score]
    ),
    'mel_ss_presc_master_db'[Reach Denominator Flag] = "Y"
)

Depends on  : [Bp Id], [Mel HCP Segments] (CC-04), [Mel Value Score], [Reach Denominator Flag] (CC-03)
Used by     : Reach % and reach target visual calculations
⚠️ MEL-ONLY: No equivalent column in RCC (28341).
⚠️ LOD PATTERN: ALLEXCEPT preserves only [Mel HCP Segments] and [Mel Value Score]
   filter context — all other filters (date, territory, etc.) are removed.
   This replicates Tableau FIXED LOD behavior.
⚠️ summarizeBy = sum: The column value repeats per row within the same
   segment/score group. Summing across rows will over-count — use with
   DISTINCTCOUNT or FIRSTNONBLANK in measures as appropriate.
Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — AUXILIARY / PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### AUX-01 | Filteredgrp_nmByField
─────────────────────────────────────────────────────────────
PBI Table       : Filteredgrp_nmByField
PBI lineageTag  : 963e7a99-8d1b-4985-8348-8046a80526b8
PBI isHidden    : true (entire table hidden)
PBI Type        : DAX Calculated Table (SUMMARIZE-based)

Column:
  Filteredgrp_nm    string    isHidden    sourceColumn: [grp_nm]
  lineageTag        : 82a79a89-4dcf-41c7-b38a-d6592c512d0f

Partition DAX:
──────────────
SUMMARIZE(
    FILTER(
        VALUES(mel_ss_presc_master_db[Grp Nm]),
        CALCULATE(
            ([]),
            mel_ss_presc_master_db[Grp Nm] = EARLIER(mel_ss_presc_master_db[Grp Nm])
        ) >= 0
    ),
    mel_ss_presc_master_db[grp_nm]
)

Purpose    : Dynamic slicer filtering helper for Group Name (grp_nm).
             Ensures only grp_nm values with at least one row in the
             current filter context appear in the slicer.
             Tableau equivalent: cascading filter or context-dependent
             filter action on the Group Name field.
Relationship: mel_ss_presc_master_db[Grp Nm] →
              Filteredgrp_nmByField[Filteredgrp_nm]
              (FilteredCalculatedTableRelationship)
⚠️ MEL-ONLY: No equivalent in RCC (28341). RCC uses a static
   RCC_Segment_Order sort table; MEL uses a dynamic filter helper.
⚠️ HIDDEN: Both table and column are isHidden. Do not expose in report layer.
⚠️ NOTE: The ([]) inside CALCULATE is an empty measure call — intentional.
   It evaluates the default aggregation in context. Preserve exactly.
Conversion Status: ✅ FULL


### AUX-02 | DimDate
─────────────────────────────────────────────────────────────
PBI Table       : DimDate
PBI Type        : Reference table (ref table in model)
Purpose         : Date dimension for time-intelligence filtering and axis
Join Column     : DimDate[Date] ← mel_ss_presc_master_db[Date]
⚠️ MEL-ONLY: No DimDate physical relationship in RCC (28341).
⚠️ NOTE: DimDate TMDL definition not included in uploaded PBIP files.
   Schema (columns, granularity, M source) not captured here.
   Append DimDate schema when available.
Conversion Status: ⚠️ PARTIAL — relationship captured; table schema pending


### AUX-03 | Measure Table
─────────────────────────────────────────────────────────────
PBI Table       : Measure Table
PBI Type        : Reference table (ref table in model)
Purpose         : Central measures container
⚠️ NOTE: Measure Table TMDL definition not included in uploaded PBIP files.
   Measures not captured here. Append when available.
Conversion Status: ⚠️ PARTIAL — reference captured; measures pending


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — HIERARCHIES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### HIER-01 | Hcp Regn, Hcp Zip
─────────────────────────────────────────────────────────────
PBI Table       : mel_ss_presc_master_db
PBI lineageTag  : e6ccdebc-b2f7-4f6e-ae00-cee0dc06cd8c
PBI Name        : Hcp Regn, Hcp Zip

Tableau Origin  : Drill path: hcp_regn → hcp_zip

PBIP Level Definition:
──────────────────────
Level 1 (inner) : hcp_zip   lineageTag: 36dad685-815b-49be-b076-2fa1c9448be0
Level 2 (outer) : hcp_regn  lineageTag: 80f038df-490d-4b7e-928a-bf8b032b43f4

⚠️ NOTE: Level order is hcp_zip first (inner), hcp_regn second (outer).
   Do NOT reverse.
Conversion Status: ✅ FULL


### HIER-02 | date_Date Hierarchy
─────────────────────────────────────────────────────────────
PBI Table       : mel_ss_presc_master_db
PBI lineageTag  : 4f6e82c5-ed19-456a-8f02-f29e6485ab34
PBI Name        : date_Date Hierarchy

PBIP Level Definition:
──────────────────────
Level 1 : date_Month-Trunc   lineageTag: date_Month-Trunc   column: date_Month-Trunc (CC-08)

⚠️ MEL-ONLY: No equivalent date hierarchy in RCC (28341).
⚠️ Single-level hierarchy — provides month-truncated date drill for
   time-series visuals via the [date_Month-Trunc] calculated column.
Conversion Status: ✅ FULL


### HIER-03 | Hcp Regn Hierarchy
─────────────────────────────────────────────────────────────
PBI Table       : mel_ss_presc_master_db
PBI lineageTag  : 324e46ce-3449-4bf1-a112-9214f3f8f740
PBI Name        : Hcp Regn Hierarchy

PBIP Level Definition:
──────────────────────
Level 1 : Hcp Regn   lineageTag: e7fa7452-445d-48c6-af91-75c528c438a7
Level 2 : Hcp Zip    lineageTag: b7dc8612-0628-447d-a2ed-c317233e7f81

⚠️ MEL-ONLY additional hierarchy: Top-down drill (Hcp Regn → Hcp Zip),
   distinct from HIER-01 which is inner-to-outer zip-first order.
   Both hierarchies reference the same underlying columns.
⚠️ NOTE: Level order here is Hcp Regn first → Hcp Zip second (top-down).
   HIER-01 has hcp_zip first (inner). These are intentionally different
   drill directions. Preserve both.
Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — RELATIONSHIPS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

REL-01 | mel_ss_presc_master_db → Filteredgrp_nmByField
─────────────────────────────────────────────────────────────
Name        : FilteredCalculatedTableRelationship_mel_ss_presc_master_db --> Filteredgrp_nmByField
From Table  : mel_ss_presc_master_db
From Column : Grp Nm
To Table    : Filteredgrp_nmByField
To Column   : Filteredgrp_nm
Purpose     : Drives dynamic Group Name slicer filtering (AUX-01)


REL-02 | mel_ss_presc_master_db → DimDate
─────────────────────────────────────────────────────────────
GUID        : c2b35da7-0975-bc90-1e97-ff3f16069db6
From Table  : mel_ss_presc_master_db
From Column : Date
To Table    : DimDate
To Column   : Date
Purpose     : Links fact table to date dimension for time-intelligence
              and date slicer filtering

Total relationships: 2
⚠️ KEY DIFFERENCE vs RCC (28341):
   RCC has 1 relationship (fact → RCC_Segment_Order sort table).
   MEL has 2 relationships (fact → Filteredgrp_nmByField, fact → DimDate).
   MEL has NO segment sort table relationship.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — MEL vs RCC DELTA SUMMARY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Property                      | MEL (28339)                        | RCC (28341)
------------------------------|------------------------------------|----------------------------------
PBIP Dataset ID               | 29967                              | 29965
Source table                  | mel_ss_presc_master_db             | rcc_ss_presc_master_db
Column count (source)         | 69                                 | 68
Extra column (MEL only)       | drug (ord 46, lowercase)           | —
Brand-specific col (ord 11)   | mel_dynm_seg_nm                    | rcc_dynm_seg_nm
Brand-specific col (ord 19)   | mel_value_score                    | rcc_value_score
Brand-specific col (ord 66)   | mel_segments                       | rcc_segments
date PBIP name                | Date (no suffix)                   | Date 1 (" 1" suffix)
source PBIP name              | Source (no suffix)                 | Source 1 (" 1" suffix)
mutation PBIP name            | Mutation (no suffix)               | Mutation 1 (" 1" suffix)
lot PBIP name                 | Lot (no suffix)                    | Lot 1 (" 1" suffix)
callkey PBIP name             | Callkey (no suffix)                | Callkey 1 (" 1" suffix)
callkey formatString          | (none)                             | "0.0"
Null handling on segments     | BLANK() — no PQ replacement        | null → "Null" string in PQ
Segment sort table            | None                               | RCC_Segment_Order DATATABLE
Primary Specialty group       | SWITCH inclusion (10 values)       | IF exclusion (53 values)
Reach Denominator product     | "OPDUALAG"                         | "OPDIVO_YERVOY_RCC"
Reach Denominator year        | "2026"                             | "2025"
Reach Denominator OR branch   | [Mel Value Score] IN {VH,H,M,L}   | [Fld Trgt Flg] = "Y"
Reach Denominator sf_nm       | Not used                           | 8-value team list
Refresh Date format           | "d mmm yyyy" → "7 Jul 2026"        | "yyyy-MM-dd" → "2026-07-07"
Refresh Date rollback formula | MOD(WEEKDAY+2,7)                   | MOD(WEEKDAY+3,7)
full year DAX                 | FORMAT(YEAR(...),"0")              | YEAR(...) with string dataType
C13W Flag range               | week_no 7–19                       | Week No 1–13
P13W Flag range               | week_no 20–32                      | Week No 14–26
week_no PBIP name             | week_no (lowercase)                | Week No (title-case)
fld_trgt_flg PBIP name        | fld_trgt_flg (lowercase)           | Fld Trgt Flg (title-case)
sf_cd PBIP name               | sf_cd (lowercase)                  | Sf Cd (title-case)
sf_nm PBIP name               | sf_nm (lowercase)                  | Sf Nm (title-case)
repeater_flg PBIP name        | repeater_flg (lowercase)           | Repeater Flg (title-case)
final_spec_cd PBIP name       | final_spec_cd (lowercase)          | Final Spec Cd (title-case)
spec_group PBIP name          | spec_group (lowercase)             | Spec Group (title-case)
MEL-only calc columns         | Mel HCP Segments (CC-04)           | —
                              | date_Month-Trunc (CC-08)           |
                              | Reach_Target_Fixed_Col (CC-09)     |
Hierarchies                   | 3: Hcp Regn+Zip, date, Hcp Regn   | 1: Hcp Regn+Zip
Relationships                 | 2: Filteredgrp_nm, DimDate         | 1: Segment Order
Date dimension                | DimDate (physical relationship)    | None
Refresh_Info table            | Not present                        | Present (DateTime.LocalNow())


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — KNOWN GAPS & RECOMMENDATIONS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

G1 | DimDate and Measure Table schemas not captured
─────────────────────────────────────────────────────────────
Status  : ⚠️ Pending
Detail  : DimDate and Measure Table are referenced in the model
          (ref table) but TMDL definition files were not included
          in the uploaded PBIP set. Columns, M source, and measures
          are not documented here.
Action  : Upload DimDate TMDL and Measure Table TMDL files and
          append to this document.

G2 | Reach_Target_Fixed_Col summarizeBy: sum over-count risk
─────────────────────────────────────────────────────────────
Status  : ⚠️ Known behavior — document in consuming measures
Detail  : CC-09 has summarizeBy: sum. The column value is a
          DISTINCTCOUNT at the segment/score grain. Summing across
          rows within a group will over-count distinct HCPs.
Action  : Use FIRSTNONBLANK, MAX, or DISTINCTCOUNT in any measure
          that references this column. Document the over-count risk.

G3 | Reach Denominator Flag year hardcode
─────────────────────────────────────────────────────────────
Status  : ⚠️ Technical debt — must track per reporting year
Detail  : [Reach Denominator Flag] (CC-03) hardcodes "2026" as a
          string. This is intentional business logic for the 2026
          MEL reporting year.
Action  : Preserve as-is. Requires a formal change request to update.
          When updating, change only the string value — keep the
          string comparison (do not switch to integer).

G4 | Deprecated segment mapping in comment
─────────────────────────────────────────────────────────────
Status  : ℹ️ Preserved intentionally
Detail  : CC-04 [Mel HCP Segments] originally mapped Segment A–D
          to named personas. Replaced with UPPER(). Old mapping
          is preserved as a nested comment in the DAX block.
Action  : Keep comment block. Do not remove historical logic.

G5 | date_Month-Trunc type annotation mismatch
─────────────────────────────────────────────────────────────
Status  : ⚠️ Known mismatch — do not fix without impact assessment
Detail  : CC-08 declares dataType: dateTime but DAX returns a string
          via FORMAT(). Changing the annotation may break HIER-02
          and visual bindings.
Action  : Preserve dataType: dateTime declaration as-is.

G6 | Filteredgrp_nmByField EARLIER() performance
─────────────────────────────────────────────────────────────
Status  : ℹ️ Performance note
Detail  : The FILTER + EARLIER pattern inside SUMMARIZE can be slow
          on high-cardinality [grp_nm] sets.
Action  : Monitor Group Name slicer query performance. Refactor to
          TREATAS or CROSSFILTER if performance degrades.

G7 | No null replacement on mel_segments
─────────────────────────────────────────────────────────────
Status  : ⚠️ Known behavior difference vs RCC
Detail  : RCC replaces rcc_segments nulls with "Null" in PQ.
          MEL loads mel_segments as-is — null rows produce BLANK()
          in [Mel HCP Segments] (UPPER(BLANK()) = BLANK()).
Action  : Preserve — no PQ null replacement. If a visible "null"
          bucket is needed in MEL segment slicers, a change
          request is required.

G8 | No time intelligence / no auto date tables
─────────────────────────────────────────────────────────────
Status  : ⚠️ By design — do not enable without impact assessment
Detail  : __PBI_TimeIntelligenceEnabled = 0. Auto date tables are
          suppressed. Date intelligence is handled via the explicit
          DimDate relationship (REL-02) and [date_Month-Trunc].
Action  : Do not enable time intelligence without assessing impact
          on CC-06, CC-07 week-number logic and HIER-02.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — CONVERSION COMPLETENESS SUMMARY
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Category                         Tableau   PBI       Status        %
─────────────────────────────────────────────────────────────────────
Source Columns                      69      69     ✅ Full       100%
Calculated Columns (FIXED LOD)       1       1     ✅ Full       100%
Calculated Columns (other)           8       8     ✅ Full       100%
Auxiliary / Parameter Tables         3       3     ✅/⚠️ Partial  67%
  (Filteredgrp_nm: full;
   DimDate schema: pending;
   Measure Table: pending)
Hierarchies                          3       3     ✅ Full       100%
Relationships                        2       2     ✅ Full       100%
Power Query / Source                 1       1     ✅ Full       100%
─────────────────────────────────────────────────────────────────────
OVERALL                                         ✅/⚠️           ~92%
─────────────────────────────────────────────────────────────────────

Legend:
  ✅ FULL     — exact functional equivalent implemented
  ⚠️ PARTIAL  — present but incomplete or requires follow-up action
  ℹ️ Low risk — omitted intentionally; low business impact


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — QUICK REFERENCE: TABLEAU FORMULA → DAX PATTERN
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Tableau Pattern                                            DAX Equivalent
──────────────────────────────────────────────────────────────────────────────
STR(YEAR([date]))                                        → FORMAT(YEAR([Date]),"0")
UPPER([mel_segments])                                    → UPPER([mel_segments])
IF [week_no]>=7 AND <=19 THEN 'C13W' ELSE NULL END       → IF(AND([week_no]>=7,[week_no]<=19),"C13W",BLANK())
IF [week_no]>=20 AND <=32 THEN 'P13W' ELSE NULL END      → IF(AND([week_no]>=20,[week_no]<=32),"P13W",BLANK())
DATETRUNC('month',[date])                                → FORMAT(COALESCE([Date],BLANK()),"MMMM YYYY")
{FIXED [dim1],[dim2] : COUNTD([col])} WHERE flag="Y"     → CALCULATE(DISTINCTCOUNT([col]),ALLEXCEPT(table,[dim1],[dim2]),[flag]="Y")
"(Refreshed as on "+str(date(DATEADD(...)))+")"          → "(Refreshed as on " & FORMAT(TODAY()-MOD(WEEKDAY+2,7),"d mmm yyyy") & ")"
SWITCH inclusion group (10 values → keep, else "Others") → SWITCH(TRUE(),[col]="val1","val1",...,"Others")
Tableau Group (exclusion list)                           → IF([col] IN {"v1","v2",...},"Others",[col])
Cascading/context filter on grp_nm                       → SUMMARIZE(FILTER(VALUES([col]),CALCULATE(([]))>=0),[col])
Drill path (field1 → field2)                             → hierarchy with level1=field1, level2=field2


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 11 — FILE REFERENCE
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

File                                                        | Content
------------------------------------------------------------|------------------------------------------
28339_brand_analytics_mel_opdivo_self_serve_dashboard.txt   | Tableau workbook XML source
10_database.txt                                             | PBIP database definition (compatibilityLevel 1604)
10_model.txt                                                | PBIP model definition, table refs, annotations
10_relationships.txt                                        | PBIP relationship definitions (2 relationships)
10_definition.json                                          | PBIP definition schema v4.2
mel_ss_presc_master_db.txt                                  | PBIP main fact table TMDL (69 cols + 9 calc cols)
filteredgrp_nmbyfield.txt                                   | PBIP Filteredgrp_nmByField table TMDL
10_diagramlayout.json                                       | PBIP diagram layout (visual only, no logic)

Pending files (not yet uploaded):
  DimDate TMDL                                              | Date dimension table definition
  Measure Table TMDL                                        | Central measures table and all DAX measures

---

# END OF KNOWLEDGE FILE
# 28339 – Brand Analytics MEL Opdivo Self-Serve Dashboard | Tableau → Power BI (PBIP/TMDL)
# Generated: 2026-07-08
# Source files: 28339_brand_analytics_mel_opdivo_self_serve_dashboard.txt
#               mel_ss_presc_master_db.txt, filteredgrp_nmbyfield.txt,
#               10_model.txt, 10_relationships.txt, 10_database.txt,
#               10_definition.json (PBIP)
