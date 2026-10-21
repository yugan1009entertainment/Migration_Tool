# 📘 CONVERSION KNOWLEDGE FILE
# Customer Shipment Report
# Tableau → Power BI (PBIP/TMDL)
# Purpose: Use this file as the definitive reference to convert any
#          future Tableau workbook update into a matching PBIP dataset.
# Generated: 2026-07-08
# Source files: customer_shipment_report.txt + all PBIP .txt extraction files

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 1 — TABLEAU PARAMETERS → POWER BI PARAMETER TABLES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

NO TABLEAU PARAMETERS. This workbook has no Tableau parameters and
no standalone parameter tables in the PBIP model.

Conversion Status: ✅ N/A

Slicers and filter-driving tables (BB slicer, Drop-ship Slicer,
Back-OrderStatus, Base QTY, Invloice QTY, Back Order) are documented
in Section 5 as auxiliary calculated/static tables.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 2 — TABLEAU CUSTOM COLUMNS → POWER BI CALCULATED COLUMNS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: MARA_MAKT_MARM_Delivery_Left_Join

### CC-01 | !Brand
─────────────────────────────────────────────────────────────
PBI Lineage Tag : c143cd58-a8bd-4e83-961a-8c6dbc17887d
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Equivalent:
───────────────────
// Tableau computed brand from [Product Description] using LEFT/CONTAINS
// logic — NULOJIX, ELIQUIS as 7-char prefix; ORENCIA SC/IV as 10-char
// prefix; all others = first word (up to first space).

Power BI DAX:
─────────────
column '!Brand' =
VAR Prefix7   = LEFT('MARA_MAKT_MARM_Delivery_Left_Join'[Product Description], 7)
VAR SpacePos  = FIND(" ", 'MARA_MAKT_MARM_Delivery_Left_Join'[Product Description], 1,
                    LEN('MARA_MAKT_MARM_Delivery_Left_Join'[Product Description]))
VAR PrefixOre = LEFT('MARA_MAKT_MARM_Delivery_Left_Join'[Product Description], SpacePos + 2)
VAR FirstWord = LEFT('MARA_MAKT_MARM_Delivery_Left_Join'[Product Description], SpacePos - 1)
RETURN
    UPPER(
        SWITCH(TRUE(),
            Prefix7   = "NULOJIX",    "NULOJIX",
            Prefix7   = "ELIQUIS",    "ELIQUIS",
            PrefixOre = "ORENCIA SC", "ORENCIA SC",
            PrefixOre = "ORENCIA LI", "ORENCIA IV",
            PrefixOre = "ORENCIA CJ", "ORENCIA IV",
            FirstWord
        )
    )

Depends on  : [Product Description] (source column from MAKT.MAKTG)
Used by     : [REMS] calculated column (CC-02)
⚠️  NOTE    : DAX uses SpacePos+2 for PrefixOre (dynamic 10-char),
              SQL uses LEFT(...,10) directly. Functionally equivalent.
              DAX UPPER() wrapper uppercases all output; SQL CASE does not.
              Both layers coexist intentionally — SQL [Brand] used for
              push-down; DAX [!Brand] used for REMS column reference.
⚠️  NOTE    : The SQL REMS Products column uses LEFT(MAKTG,7) for POMALYST
              (8 chars) — this is a pre-existing bug inherited from Tableau.
              DO NOT FIX. The DAX [REMS] column inherits the same truncation.
Conversion  : ✅ FULL (dual-layer: SQL Brand + DAX !Brand)


### CC-02 | REMS
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 8fd5da02-7934-4aab-8be4-6d0800474dfc
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Equivalent:
───────────────────
// Tableau computed REMS flag using brand logic on [Product Description]:
//   POMALYST, REVLIMID, THALOMID → "REMS"; all others → "Other"

Power BI DAX:
─────────────
column REMS =
IF(
    'MARA_MAKT_MARM_Delivery_Left_Join'[!Brand] = "POMALYST"
 || 'MARA_MAKT_MARM_Delivery_Left_Join'[!Brand] = "REVLIMID"
 || 'MARA_MAKT_MARM_Delivery_Left_Join'[!Brand] = "THALOMID",
    "REMS",
    "Other"
)

Depends on  : [!Brand] (CC-01)
⚠️  NOTE    : Due to the 7-char truncation bug in CC-01, "POMALYST"
              (8 chars) will never match via [!Brand]. The SQL
              [REMS Products] column has the same bug. Preserved as-is
              for Tableau fidelity — do not correct.
Conversion  : ✅ FULL (with noted pre-existing bug preserved)


---

## TABLE: MARA_MAKT_MARM_Delivery_Left_Join (Back Order)

### CC-03 | !Brand(back order)
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 35dfc044-4a7c-4762-b47d-ebafe0ea7f70
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Equivalent:
───────────────────
// Same brand logic as CC-01 — Back Order stream mirror.

Power BI DAX:
─────────────
column '!Brand(back order)' =
VAR Prefix7   = LEFT('MARA_MAKT_MARM_Delivery_Left_Join (Back Order)'[Product Description], 7)
VAR SpacePos  = FIND(" ", 'MARA_MAKT_MARM_Delivery_Left_Join (Back Order)'[Product Description], 1,
                    LEN('MARA_MAKT_MARM_Delivery_Left_Join (Back Order)'[Product Description]))
VAR PrefixOre = LEFT('MARA_MAKT_MARM_Delivery_Left_Join (Back Order)'[Product Description], SpacePos + 2)
VAR FirstWord = LEFT('MARA_MAKT_MARM_Delivery_Left_Join (Back Order)'[Product Description], SpacePos - 1)
RETURN
    UPPER(
        SWITCH(TRUE(),
            Prefix7   = "NULOJIX",    "NULOJIX",
            Prefix7   = "ELIQUIS",    "ELIQUIS",
            PrefixOre = "ORENCIA SC", "ORENCIA SC",
            PrefixOre = "ORENCIA LI", "ORENCIA IV",
            PrefixOre = "ORENCIA CJ", "ORENCIA IV",
            FirstWord
        )
    )

⚠️  NOTE    : Column name is "!Brand(back order)" — note no space before
              "(back order)". Preserve exactly.
              Back Order stream has NO [REMS] DAX column — only the SQL
              [REMS Products] source column exists for this stream.
Conversion  : ✅ FULL


---

## TABLE: VBAK_VBAP_left_join

### CC-04 | Lips3 / Lips_KEY2  (Composite Key — Main Stream)
─────────────────────────────────────────────────────────────
PBI Lineage Tags: Lips3    = 1b07dd01-66c7-48c8-acf6-dd1b8ab2cc3d
                  Lips_KEY2 = c872178f-76c6-4d4d-b53d-bd7283a41684
PBI Data Type   : string
PBI Summarize   : none

Power BI DAX:
─────────────
column Lips3 =
    CONCATENATE(VBAK_VBAP_left_join[Sales Document (VBAP)], " - ")

column Lips_KEY2 =
    CONCATENATE(VBAK_VBAP_left_join[Lips3], VBAK_VBAP_left_join[Sales Document Item (VBAP)])

Result format : "<Sales Document (VBAP)> - <Sales Document Item (VBAP)>"
Joins to      : LIPS-Delivery[VBAP_KEY]
Conversion    : ✅ FULL


---

## TABLE: VBAK_VBAP_left_join (Back Order)

### CC-05 | BB_or_SB  (Block Classification)
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 911d81b7-032c-4c43-8b3e-670652014d2f
PBI Data Type   : string
PBI Hidden      : No
PBI Summarize   : none

Tableau Equivalent:
───────────────────
// Tableau computed block status using IF/ISNULL checks across
// billing block, delivery block, schedule line delivery block (LIFSP),
// and rejection reason. Combined with OR logic.

Power BI DAX:
─────────────
column BB_or_SB =
IF(
    OR(
        OR(
            NOT(ISBLANK('VBAK_VBAP_left_join (Back Order)'[Billing Block in SD Document])),
            NOT(ISBLANK('VBAK_VBAP_left_join (Back Order)'[Delivery Block (Document Header)]))
        ),
        OR(
            NOT(ISBLANK(RELATED('VBEP(Back Order)'[LIFSP]))),
            NOT(ISBLANK(RELATED('TVAGT (Back Order)'[Reason for Rejection of Sales Document])))
        )
    ),
    "Block",
    "No Block"
)

Logic   : "Block" if ANY of the following is non-blank:
          1. Billing Block in SD Document (from VBAK)
          2. Delivery Block (Document Header) (from VBAK)
          3. LIFSP — schedule line delivery block (from VBEP via RELATED)
          4. Reason for Rejection of Sales Document (from TVAGT via RELATED)
Depends on : RELATED('VBEP(Back Order)'[LIFSP])
             RELATED('TVAGT (Back Order)'[Reason for Rejection of Sales Document])
Used by    : Back Order analysis visuals (primary segmentation field)
Conversion : ✅ FULL


### CC-06 | vbep2 / vbep_KEY  (Composite Key — Back Order)
─────────────────────────────────────────────────────────────
PBI Lineage Tags: vbep2    = d6fc7acd-5dcf-498c-9a7a-ce3a6869e877
                  vbep_KEY = a1f24507-b75c-4801-9588-b7b0075fbfbd

Power BI DAX:
─────────────
column vbep2 =
    CONCATENATE('VBAK_VBAP_left_join (Back Order)'[Sales Document (VBAP)], " - ")

column vbep_KEY =
    CONCATENATE('VBAK_VBAP_left_join (Back Order)'[vbep2],
                'VBAK_VBAP_left_join (Back Order)'[Sales Document Item (VBAP)])

Result format : "<Sales Document (VBAP)> - <Sales Document Item (VBAP)>"
Joins to      : VBEP(Back Order)[VBAP_KEY]
Conversion    : ✅ FULL


---

## TABLE: VBAP  (Main Stream)

### CC-07 | Lips2 / Lips_KEY  (Composite Key — Main VBAP)
─────────────────────────────────────────────────────────────
PBI Lineage Tags: Lips2    = 6d18e951-dc16-42f6-9008-d1ef3522efd8
                  Lips_KEY = 06958e20-7c01-4e2b-9f3e-0ab6ff124531

Power BI DAX:
─────────────
column Lips2 =
    CONCATENATE(VBAP[Sales Document (VBAP)], " - ")

column Lips_KEY =
    CONCATENATE(VBAP[Lips2], VBAP[Sales Document Item (VBAP)])

Result format : "<Sales Document (VBAP)> - <Sales Document Item (VBAP)>"
Joins to      : LIPS-Delivery[VBAP_KEY]
⚠️  NOTE      : This is the main-stream equivalent of vbep_KEY.
                Key column names differ between streams intentionally.
Conversion    : ✅ FULL


---

## TABLE: VBAP (Back Order)

### CC-08 | vbep1 / VBEP_KEY  (Composite Key — Back Order VBAP)
─────────────────────────────────────────────────────────────
PBI Lineage Tags: vbep1    = 90196bb2-a6a8-4207-a518-73af90d659cb
                  VBEP_KEY = 133b716d-5edd-4741-aedb-aa45803fe81b

Power BI DAX:
─────────────
column vbep1 =
    CONCATENATE('VBAP (Back Order)'[Sales Document (VBAP)], " - ")

column VBEP_KEY =
    CONCATENATE('VBAP (Back Order)'[vbep1], 'VBAP (Back Order)'[Sales Document Item (VBAP)])

Result format : "<Sales Document (VBAP)> - <Sales Document Item (VBAP)>"
Joins to      : VBEP(Back Order)[VBAP_KEY]  (via VBAK_VBAP Back Order stream)
Conversion    : ✅ FULL


---

## TABLE: VBEP(Back Order)

### CC-09 | vbap2 / VBAP_KEY  (Composite Key — Schedule Lines)
─────────────────────────────────────────────────────────────
PBI Lineage Tags: vbap2    = 69fee7d9-1a85-4727-866b-711dfbcfb2b3
                  VBAP_KEY = 3bd84ced-fef0-4f21-b9e0-363a0c0cd177

Power BI DAX:
─────────────
column vbap2 =
    CONCATENATE('VBEP(Back Order)'[Sales Document(VBEP)], " - ")

column VBAP_KEY =
    CONCATENATE('VBEP(Back Order)'[vbap2], 'VBEP(Back Order)'[Sales Document Item(VBEP)])

⚠️  NOTE    : Column names "Sales Document(VBEP)" and
              "Sales Document Item(VBEP)" have NO SPACE before "(VBEP)".
              This is intentional — preserve exactly.
Joins to    : VBAK_VBAP_left_join (Back Order)[vbep_KEY]
Conversion  : ✅ FULL


### CC-10 | BackOrder Flag Label
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 19a2bc62-df11-4a0a-84e7-505705105065
PBI Data Type   : int64 (format: 0)
PBI Summarize   : sum

Tableau Equivalent:
───────────────────
// Tableau: ISNULL([Confirmed Qty]) OR [Confirmed Qty] = 0 → back order

Power BI DAX:
─────────────
column 'BackOrder Flag Label' =
    IF(COALESCE('VBEP(Back Order)'[BMENG], 0) = 0,
        1,   -- Back-Order (confirmed qty = zero or null)
        0    -- Ignore
    )

Logic    : BMENG = confirmed quantity. When 0 or null → schedule line
           is a back order. Returns integer 1 (Back-Order) or 0 (Ignore).
Used by  : [Back-Order Filter] (CC-11)
Conversion: ✅ FULL


### CC-11 | Back-Order Filter
─────────────────────────────────────────────────────────────
PBI Lineage Tag : bd298d3e-1d7e-48c2-b2c0-bd85f4d5f42e
PBI Data Type   : string
PBI Summarize   : none

Power BI DAX:
─────────────
column 'Back-Order Filter' =
    IF('VBEP(Back Order)'[BackOrder Flag Label] = 1, "Back-Order", "Ignore")

Depends on  : [BackOrder Flag Label] (CC-10)
Used by     : Slicer / visual filter on Back Order pages
Conversion  : ✅ FULL


---

## TABLE: VEPO_VEKP(Tracking)_Inner_Join

### CC-12 | LIPS1 / Lips-delivery_KEY  (Composite Key — Tracking)
─────────────────────────────────────────────────────────────
PBI Lineage Tags: LIPS1              = 6858c44d-7071-4a74-84f1-dedd29cd930d
                  Lips-delivery_KEY  = c5feb853-59c7-4dbf-b460-4d9cbc170b5d

Power BI DAX:
─────────────
column LIPS1 =
    CONCATENATE('VEPO_VEKP(Tracking)_Inner_Join'[Delivery (VEPO)], " - ")

column 'Lips-delivery_KEY' =
    CONCATENATE('VEPO_VEKP(Tracking)_Inner_Join'[LIPS1],
                'VEPO_VEKP(Tracking)_Inner_Join'[Delivery Item (VEPO)])

⚠️  NOTE    : Column name is "Lips-delivery_KEY" — contains a hyphen.
              Preserve exactly including the hyphen.
Joins to    : LIPS-Delivery[VEPO_KEY]  (delivery-side HU key)
Conversion  : ✅ FULL


---

## TABLE: VTTP_VTTK_TVROT_Left_Join

### CC-13 | Month Year of Ship Date
─────────────────────────────────────────────────────────────
PBI Lineage Tag : ecd24276-4fdc-475c-b194-d207f96bfa49
PBI Data Type   : string
PBI Summarize   : none

Tableau Equivalent:
───────────────────
// Tableau: DATENAME("month", [Ship Date]) + " " + STR(YEAR([Ship Date]))

Power BI DAX:
─────────────
column 'Month Year of Ship Date' =
SWITCH(MONTH(VTTP_VTTK_TVROT_Left_Join[Ship Date]),
    1,  "January",   2,  "February",  3,  "March",
    4,  "April",     5,  "May",       6,  "June",
    7,  "July",      8,  "August",    9,  "September",
    10, "October",   11, "November",  12, "December"
) & " " & YEAR(VTTP_VTTK_TVROT_Left_Join[Ship Date])

Example output : "March 2024"
Depends on     : [Ship Date] (SQL-computed DATE column)
Conversion     : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 3 — TABLEAU LODs / AGGREGATIONS → POWER BI DAX MEASURES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## TABLE: VBAK_VBAP_left_join

### M-01 | VBAK_count
─────────────────────────────────────────────────────────────
PBI Lineage Tag : 35580cd7-84b6-428b-ae7b-345ccc5b1efd
PBI Format      : 0

Tableau Equivalent:
───────────────────
// COUNT([Sales Document]) — row count of VBAK header records

Power BI DAX:
─────────────
measure VBAK_count = COUNTROWS(VBAK_VBAP_left_join)

Conversion  : ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 4 — LOCKED / HIDDEN FILTERS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All locked filters are implemented as hardcoded SQL WHERE clauses
in the DirectQuery partition. There are no separate PBIP background
filter objects for these — the filter is pushed to SAP HANA at query
time. The Language = "E" filter is additionally enforced via TREATAS
in DAX queries.

Table                                   Filter Field      Value / Logic             Mechanism
────────────────────────────────────────────────────────────────────────────────────────────
VBAK_VBAP_left_join (both streams)     AUART             IN ('ZDOM', 'ZFD')         SQL WHERE
VBAK_VBAP_left_join (both streams)     VKORG             IN ('R020')                SQL WHERE
VBAK_VBAP_left_join (both streams)     ERDAT             >= '20220301'              SQL WHERE
VBAP (both streams)                    ERDAT             >= '20220301'              SQL WHERE
VBRK                                   ERDAT             >= '20220301'              SQL WHERE
VBRP                                   FKART_ANA         NOT IN ('ZOF8')            SQL WHERE
VBRP                                   ERDAT             >= '20220301'              SQL WHERE
VEPO_VEKP(Tracking)_Inner_Join         VEKP.ERDAT        >= '20220301'              SQL WHERE
VTTP_VTTK_TVROT_Left_Join              VTTP.ERDAT        >= '20220301'              SQL WHERE
VBPA_ADRC_Left_Join (both streams)     PARVW             IN ('WE')                  SQL WHERE
MARA_MAKT_MARM (both streams)         Language           IFNULL(SPRAS,'E')='E'     SQL default + TREATAS
VBEP(Back Order)                       (none)            Full unfiltered pull       No WHERE clause
TVAGT (both streams)                   (none)            Full unfiltered pull       No WHERE clause
STXL                                   (none)            Full unfiltered pull       No WHERE clause
Payer_KNA1                             (none)            Full unfiltered pull       No WHERE clause

⚠️  CRITICAL — Language = 'E' filter:
    The MARA_MAKT_MARM tables default SPRAS to 'E' via SQL IFNULL.
    TREATAS is used in DAX queries to enforce Language = 'E' as the
    context filter. This mirrors the Tableau locked context filter
    on the MARA/MAKT datasource. If the SAP source ever returns
    multi-language rows, the TREATAS filter must remain in place.

⚠️  CRITICAL — ZOF8 exclusion in VBRP:
    Tableau excluded billing type ZOF8 as a datasource filter.
    PBI implements this in SQL WHERE. If new billing types require
    exclusion, update the SQL WHERE clause in the VBRP partition.

⚠️  CRITICAL — Date cutoff 20220301:
    All transactional tables use ERDAT >= '20220301' (March 1, 2022).
    This is a hardcoded business rule. Update the SQL partition source
    if the history window changes.


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 5 — POWER QUERY / DATA SOURCES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All tables connect to a single SAP HANA endpoint via DirectQuery:
  Connection  : SapHana.Database("pahpdb01.net.bms.com:30015")
  Mode        : DirectQuery (all tables)
  Auth        : Windows / Kerberos (BMS standard)

### PQ-01 | MARA_MAKT_MARM_Delivery_Left_Join
─────────────────────────────────────────────────────────────
PBI lineageTag  : 35b9cef1-081c-4b39-864c-51d57cc3925f
SAP Tables      : CV_MARA ← CV_MAKT (LEFT JOIN) ← CV_MARM (LEFT JOIN)
Join keys       : MARA.MATNR = MAKT.MATNR; MAKT.MATNR = MARM.MATNR
Schema prefix   : SAP_ERP.TABLE_CV_S4::
WHERE           : None (full material master)

Source Column Mapping (selected columns):
Ordinal  PBIP Column Name              SAP Field          Type           Notes
─────────────────────────────────────────────────────────────────────────────────────
1        Material Number               MARA.MATNR         NVARCHAR(40)   Join key
2        Transportation Group          MARA.TRAGR         NVARCHAR(4)
3        Valid-From Date               MARA.DATAB         NVARCHAR(8)    Raw SAP date
4        Cold Chain Flag               CASE TRAGR='2TO8'  VARCHAR(3)     'Yes'/'No'; NOT NULL
5        Product Description           MAKT.MAKTG         NVARCHAR(40)   Base for brand
6        MAKTX                         MAKT.MAKTX         NVARCHAR(40)
7        Language Key                  MAKT.SPRAS         NVARCHAR(1)    Raw language code
8        Language                      IFNULL(SPRAS,'E')  NVARCHAR(1)    NOT NULL; lock filter anchor
9        Brand                         CASE MAKTG...      NVARCHAR(40)   SQL-computed (see CC-01)
10       REMS Products                 CASE brand...      VARCHAR(5)     'REMS'/'Other'; NOT NULL
11       National Code                 MARM._STTPEC_NCODE NVARCHAR(40)   NDC duplicate col 1
12       NDC 11                        MARM._STTPEC_NCODE NVARCHAR(40)   NDC duplicate col 2
13       National Code Type            MARM._STTPEC_NCODE_TY NVARCHAR(10)

DAX Columns: !Brand (CC-01), REMS (CC-02)
Conversion Status: ✅ FULL


### PQ-02 | MARA_MAKT_MARM_Delivery_Left_Join (Back Order)
─────────────────────────────────────────────────────────────
PBI lineageTag  : 454728aa-0254-4e0c-8dee-2cf7fe7e4f79
SAP SQL         : Identical to PQ-01
Differences     : DAX column is "!Brand(back order)" not "!Brand";
                  NO [REMS] DAX column in this stream.
Conversion Status: ✅ FULL


### PQ-03 | Payer_KNA1
─────────────────────────────────────────────────────────────
PBI lineageTag  : 80878f1c-5a8e-47c9-a4cd-003e5da7d151
SAP Table       : CV_KNA1
Schema prefix   : SAP_ERP.TABLE_CV_S4::
WHERE           : None (full customer master)

Source Column Mapping:
Ordinal  PBIP Column Name                              SAP Field    Type          Notes
──────────────────────────────────────────────────────────────────────────────────────────
1        Customer Number                               KUNNR        NVARCHAR(10)  Join key → VBRP.Payer
2        Payer Name                                    NAME1        NVARCHAR(35)
3        International location number (part 1)        BBBNR        NVARCHAR(7)   ⚠️ lowercase 'p'
4        International location number (Part 2)        BBSNR        NVARCHAR(5)   ⚠️ uppercase 'P'
5        Check digit                                   BUBKZ        NVARCHAR(1)
6        Created Date                                  ERDAT        NVARCHAR(8)
7        Country                                       LAND1        NVARCHAR(3)
8        Region                                        REGIO        NVARCHAR(3)
9        City                                          ORT01        NVARCHAR(35)
10       Ship-To GLN                                   BBBNR||BBSNR||BUBKZ  NVARCHAR(13)  SQL concat

⚠️  NOTE: "(part 1)" vs "(Part 2)" case mismatch is intentional. Preserve.
Conversion Status: ✅ FULL


### PQ-04 | STXL
─────────────────────────────────────────────────────────────
PBI lineageTag  : 911128f0-8313-414f-a054-9964380a5e09
SAP Table       : CV_STXL
Schema prefix   : SAP_ERP.TABLE_CV_S4::
WHERE           : None (full long text table)

Source Column Mapping:
Ordinal  PBIP Column Name                              SAP Field  Type          Notes
──────────────────────────────────────────────────────────────────────────────────
1        CLIENT__SAP_ERP_TABLE_CV_S4__CV_STXL_         CLIENT     NVARCHAR(3)   Auto-prefixed collision
2        ITEMNO                                         ITEMNO     INTEGER       sumBy: sum
3        MANDT__SAP_ERP_TABLE_CV_S4__CV_STXL_          MANDT      NVARCHAR(3)   Auto-prefixed collision
4        RELID                                          RELID      NVARCHAR(2)
5        TDFORMAT                                       TDFORMAT   NVARCHAR(2)
6        TDID                                           TDID       NVARCHAR(4)
7        TDLINE                                         TDLINE     NVARCHAR(132) Long text line content
8        TDNAME                                         TDNAME     NVARCHAR(70)  Join key → VBAK_VBAP(BO)
9        TDOBJECT                                       TDOBJECT   NVARCHAR(10)
10       TDSPRAS                                        TDSPRAS    NVARCHAR(1)

Conversion Status: ✅ FULL


### PQ-05 | TVAGT  /  TVAGT (Back Order)
─────────────────────────────────────────────────────────────
PBI lineageTags : TVAGT = 0d9e7f77-d7b6-46d3-b966-a761d6bb34cd
                  TVAGT (Back Order) = f33dcbac-90e6-400d-9ab1-74a1e47bdb21
SAP View        : _SYS_BIC."SAP_ERP.TABLE_CV_S4/CV_TVAGT"
                  ⚠️ Slash-separated path — different schema prefix from
                  TABLE_CV_S4:: tables. Both streams use identical SQL.
WHERE           : None (full rejection reason table)

Source Column Mapping:
Ordinal  PBIP Column Name                                SAP Field      Type
────────────────────────────────────────────────────────────────────────────────
1        Reason for Rejection of Sales Document           ABGRU         NVARCHAR(2)  Join key
2        Description                                      BEZEI         NVARCHAR(40)
3        Language Key                                     SPRAS         NVARCHAR(1)
4        Client                                           MANDT         NVARCHAR(3)
5        Timestamp                                        ZETLTIMSTMP   NVARCHAR(14)

⚠️  NOTE: Used in BB_or_SB (CC-05) via RELATED() for back order stream.
Conversion Status: ✅ FULL (both streams)


### PQ-06 | VBAK_VBAP_left_join
─────────────────────────────────────────────────────────────
PBI lineageTag  : 81a9408a-a59e-45e9-bb97-6fa4a3571c27
SAP Tables      : CV_VBAK ← CV_VBAP (LEFT JOIN)  via _SYS_BIC.
Join key        : VBAK.VBELN = VBAP.VBELN
WHERE (locked)  : AUART IN ('ZDOM','ZFD')  AND  VKORG IN ('R020')
                  AND  ERDAT >= '20220301'

Source Column Mapping (key columns):
Ordinal  PBIP Column Name                                     SAP Field       Type
──────────────────────────────────────────────────────────────────────────────────────────
1        Sales Document                                        VBAK.VBELN      NVARCHAR(10)
2        Order Creation Date STR                               VBAK.ERDAT      NVARCHAR(8)
3        Sales Document Type                                   VBAK.AUART      NVARCHAR(4)
4        Order Reason (Reason for the Business Transaction)    VBAK.AUGRU      NVARCHAR(3)
5        Delivery Block (Document Header)                      VBAK.LIFSK      NVARCHAR(2)
6        Billing Block in SD Document                          VBAK.FAKSK      NVARCHAR(2)
7        Sales Organization                                    VBAK.VKORG      NVARCHAR(4)   Always 'R020'
8        Customer Reference                                    VBAK.BSTNK      NVARCHAR(20)
9        PO Number                                             VBAK.BSTNK      NVARCHAR(20)  Dup of above
10       Sold-To Party                                         VBAK.KUNNR      NVARCHAR(10)
11       Customer Purchase Order Type                          VBAK.BSARK      NVARCHAR(4)
12       Order Created Date                                    TO_DATE(ERDAT)  DATE          NULL-safe
13       !Order Creation Date                                  TO_DATE(ERDAT)  DATE          Dup of above; ! prefix
14       Reason for Rejection of Sales Documents               VBAP.ABGRU      NVARCHAR(2)
15       Material Number                                       VBAP.MATNR      NVARCHAR(40)
16       ANZSN                                                 VBAP.ANZSN      INTEGER       sumBy: sum
17       Sales Document Type (VBAP)                            VBAP.AUART_ANA  NVARCHAR(4)
18       Document Date (Date Received/Sent) (VBAP)             VBAP.AUDAT_ANA  NVARCHAR(8)
19       Payer                                                 VBAP.KUNRG_ANA  NVARCHAR(10)
20       Ship-to Party                                         VBAP.KUNWE_ANA  NVARCHAR(10)
21       Customer Reference (VBAP)                             VBAP.BSTKD_ANA  NVARCHAR(35)
22       Sales Document (VBAP)                                 VBAP.VBELN      NVARCHAR(10)
23       SD Document Category (VBAP)                           VBAP.VBTYP_ANA  NVARCHAR(4)
24       Sales Document Item (VBAP)                            VBAP.POSNR      NVARCHAR(6)
25       Sales Organization (VBAP)                             VBAP.VKORG_ANA  NVARCHAR(4)

DAX Measure : VBAK_count (M-01)
DAX Columns : Lips3, Lips_KEY2 (CC-04)
Conversion Status: ✅ FULL


### PQ-07 | VBAK_VBAP_left_join (Back Order)
─────────────────────────────────────────────────────────────
PBI lineageTag  : b96e80f9-d46e-48b3-a196-cee6978b68a1
SAP SQL         : Identical to PQ-06 (same WHERE, same joins)
Key differences vs. main stream:
  - BB_or_SB calculated column present (CC-05)
  - Composite key columns: vbep2, vbep_KEY (CC-06) instead of Lips3/Lips_KEY2
  - No VBAK_count measure
Conversion Status: ✅ FULL


### PQ-08 | VBAP  (Main Stream — Wide Pull)
─────────────────────────────────────────────────────────────
PBI lineageTag  : d77ad8fa-46f5-42d5-a532-153a65d15068
SAP Table       : CV_VBAP  via SAP_ERP.TABLE_CV_S4::
WHERE           : ERDAT >= '20220301'  (date cutoff only — no type/org filter)
DAX Columns     : Lips2, Lips_KEY (CC-07)
Note            : 36 columns pulled — full item detail for main delivery stream.
                  Structurally identical to VBAP (Back Order) except for
                  key column names (Lips2/Lips_KEY vs vbep1/VBEP_KEY).
Conversion Status: ✅ FULL


### PQ-09 | VBAP (Back Order)
─────────────────────────────────────────────────────────────
PBI lineageTag  : 323a9108-e2a4-45a6-a1ac-03f53b639afb
SAP SQL         : Identical to PQ-08
DAX Columns     : vbep1, VBEP_KEY (CC-08)
⚠️  NOTE        : MATNR column is named "Material Number (VBAP)" in this
                  table (vs plain "Material Number" in main VBAP).
Conversion Status: ✅ FULL


### PQ-10 | VBEP(Back Order)
─────────────────────────────────────────────────────────────
PBI lineageTag  : 4de9382e-9277-47aa-bd25-a4c2b660c06d
SAP Table       : CV_VBEP  via SAP_ERP.TABLE_CV_S4::
WHERE           : NONE — full unfiltered pull of CV_VBEP
Note            : Only table in model with no date filter.
                  70+ columns pulled — full schedule line detail.

Key columns for back order logic:
Ordinal  PBIP Column Name                    SAP Field    Type             Notes
────────────────────────────────────────────────────────────────────────────────────
1        Sales Document(VBEP)                VBELN        NVARCHAR(10)     ⚠️ No space before (VBEP)
2        Sales Document Item(VBEP)           POSNR        NVARCHAR(6)      ⚠️ No space before (VBEP)
3        LIFSP                               LIFSP        NVARCHAR(2)      Schedule line delivery block
4        BMENG                               BMENG        DECIMAL(13,3)    Confirmed qty — back order key
5        WMENG                               WMENG        DECIMAL(13,3)    Open quantity
6        EDATU                               EDATU        NVARCHAR(8)      Scheduled delivery date
7        ZZEDATU                             ZZEDATU      NVARCHAR(8)      Custom BMS extended date
8        ORDQTY_BU / ORDQTY_SU               ...          DECIMAL(13,3)    Ordered qty
9        DLVQTY_BU / DLVQTY_SU               ...          DECIMAL(13,3)    Delivered qty
10       MANDT__SAP_ERP_TABLE_CV_S4__CV_VBEP_ MANDT       NVARCHAR(3)      Auto-prefixed collision
11       AUFNR__SAP_ERP_TABLE_CV_S4__CV_VBEP_ AUFNR       NVARCHAR(12)     Auto-prefixed collision
12       XDATAAGING__SAP_ERP_TABLE_CV_S4__CV_VBEP_ _DATAAGING NVARCHAR(8) _ → X rename pattern

DAX Columns: vbap2, VBAP_KEY (CC-09), BackOrder Flag Label (CC-10),
             Back-Order Filter (CC-11)
Conversion Status: ✅ FULL


### PQ-11 | VBPA_ADRC_Left_Join  /  VBPA_ADRC_Left_Join (Back Order)
─────────────────────────────────────────────────────────────
PBI lineageTags : Main = 3dd93551-89f0-4990-8d38-679ab7d741db
                  Back Order = 3aeedd92-44f6-4a99-a896-0ed56301e52e
SAP Tables      : CV_VBPA ← CV_ADRC (LEFT JOIN)
Join key        : VBPA.ADRNR = ADRC.ADDRNUMBER
WHERE (locked)  : PARVW IN ('WE')  — ship-to partners only
SQL             : Identical in both streams

Key source column mapping:
PBIP Column Name                       SAP Field            Notes
─────────────────────────────────────────────────────────────────────────────
Sales and Distribution Document Number VBPA.VBELN           Join key → VBAK_VBAP
Partner Function                       VBPA.PARVW           Always 'WE'
KUNNR                                  VBPA.KUNNR
MANDT_VBPA                             VBPA.MANDT           Renamed collision
Address                                VBPA.ADRNR           NVARCHAR(10)
Address Number                         ADRC.ADDRNUMBER      NVARCHAR(10)
Ship-To Name                           ADRC.MC_NAME1        NVARCHAR(25)
City                                   ADRC.CITY1           NVARCHAR(40)
Ship-To City                           ADRC.CITY1           NVARCHAR(40)   Dup alias of City
Valid-to date in current Release       ADRC.DATE_TO         NVARCHAR(8)
Valid-from date in current Release     ADRC.DATE_FROM       NVARCHAR(8)

⚠️  NOTE: City and Ship-To City are both CITY1 — intentional SQL alias dup.
Conversion Status: ✅ FULL (both streams)


### PQ-12 | VBRK
─────────────────────────────────────────────────────────────
PBI lineageTag  : c2e80fcb-b42a-4ee9-b57b-1b787f7a5227
SAP Table       : CV_VBRK  via SAP_ERP.TABLE_CV_S4::
WHERE           : ERDAT >= '20220301'

Key source column mapping:
PBIP Column Name                SAP Field    Type             Notes
─────────────────────────────────────────────────────────────────────────────
Billing Document (VBRK)         VBELN        NVARCHAR(10)     Join key → VBRP.VBRP Billing Document
VBELN                           VBELN        NVARCHAR(10)     Raw duplicate of above
FKART                           FKART        NVARCHAR(4)      Billing type
FKDAT                           FKDAT        NVARCHAR(8)      Billing date
KUNAG                           KUNAG        NVARCHAR(10)     Sold-To
KUNRG                           KUNRG        NVARCHAR(10)     Payer
KUNWE                           KUNWE        NVARCHAR(10)     Ship-To
NETWR                           NETWR        DECIMAL(15,2)    sumBy: none ⚠️
WAERK                           WAERK        NVARCHAR(5)      Currency
VKORG                           VKORG        NVARCHAR(4)      Sales org
ERDAT                           ERDAT        NVARCHAR(8)      Created date

⚠️  NOTE: NETWR has sumBy:none despite being numeric — intentional.
          Use VBRP billing item values for aggregation, not header NETWR.
Conversion Status: ✅ FULL


### PQ-13 | VBRP
─────────────────────────────────────────────────────────────
PBI lineageTag  : 9aa5849f-34a4-4fbc-bf64-ed3634375d21
SAP Table       : CV_VBRP  via SAP_ERP.TABLE_CV_S4::
WHERE (locked)  : FKART_ANA NOT IN ('ZOF8')  AND  ERDAT >= '20220301'

Key source column mapping:
PBIP Column Name                                              SAP Field        Type             Notes
─────────────────────────────────────────────────────────────────────────────────────────────────────
VBRP Billing Document                                         VBELN            NVARCHAR(10)     → VBRK
Sales Document (VBRP)                                         AUBEL            NVARCHAR(10)     → VBAK_VBAP
Billing Item                                                  POSNR            NVARCHAR(6)
Material                                                      MATNR            NVARCHAR(40)
Material entered (VBRP)                                       MATWA            NVARCHAR(40)
Material # (VBRP)                                             RIGHT(MATNR,7)   NVARCHAR(7)      SQL computed
Billed Quantity                                               FKIMG            DECIMAL(13,3)    sumBy: sum
Actual billed quantity                                        FKIMG            DECIMAL(13,3)    Dup of above
Net Value                                                     NETWR            DECIMAL(15,2)    sumBy: sum
Net value of the billing item in document currency            NETWR            DECIMAL(15,2)    Dup of above
Invoice Amount                                                NETWR            DECIMAL(15,2)    Dup of above
Currency                                                      WAERK            NVARCHAR(5)
Payer                                                         KUNRG_ANA        NVARCHAR(10)     → Payer_KNA1
Payer #                                                       KUNRG_ANA        NVARCHAR(10)     Dup of Payer
Ship-to party                                                 KUNWE_ANA        NVARCHAR(10)     ⚠️ lowercase 'p'
Ship-To #                                                     KUNWE_ANA        NVARCHAR(10)     Dup; uppercase 'T'
Billing Date                                                  FKDAT_ANA        NVARCHAR(8)      Raw date string
Invoice Date                                                  TO_DATE(FKDAT_ANA) DATE           NULL-safe; Short Date
Created Date                                                  ERDAT            NVARCHAR(8)
VBRP Billing Type                                             FKART_ANA        NVARCHAR(4)      Excludes ZOF8
Order Reason (Reason for the Business Transaction) (VBRP)     AUGRU_AUFT       NVARCHAR(3)
Numerator (factor) for conversion of sales quantity into SKU  UMVKZ            DECIMAL(5,0)

⚠️  NOTE: NETWR mapped to 3 aliases: "Net Value", "Net value of the billing
          item in document currency", "Invoice Amount". All intentional.
⚠️  NOTE: "Ship-to party" (lowercase 'p') vs "Ship-To #" (uppercase 'T') —
          intentional case difference, preserve.
Conversion Status: ✅ FULL


### PQ-14 | VEPO_VEKP(Tracking)_Inner_Join
─────────────────────────────────────────────────────────────
PBI lineageTag  : b83bc815-ab11-49f1-bedb-99f3358ce485
SAP Tables      : CV_VEPO INNER JOIN CV_VEKP
Join key        : VEPO.VENUM = VEKP.VENUM  (INNER JOIN — only HU records)
WHERE           : VEKP.ERDAT >= '20220301'
⚠️  NOTE        : ONLY INNER JOIN in the model. Deliveries without HU
                  packing records are excluded.

Key source column mapping:
PBIP Column Name                                   SAP Field         Notes
─────────────────────────────────────────────────────────────────────────────────
Batch Number (SAP_ERP.TABLE_CV_S4::CV_VEPO)        VEPO.CHARG        Full-path collision name
Batch                                              VEPO.CHARG        Clean alias of same
Delivery (VEPO)                                    VEPO.VBELN        Join anchor for key
Delivery Item (VEPO)                               VEPO.POSNR        Second part of key
Internal Handling Unit Number                      VEPO.VENUM        Join to VEKP.VENUM
Base Quantity Packed in the Handling Unit Item     VEPO.VEMNG        format #,0
Batch Number (SAP_ERP.TABLE_CV_S4::CV_VEKP)        VEKP.CHARG        Full-path collision name
Handling Unit's 2nd External Identification        VEKP.EXIDV2       Source for tracking
Tracking Number_V1                                 CASE EXIDV2...    'No Tracking' if blank
Total Weight of Handling Unit                      VEKP.BRGEW        format #,0
Handling Unit Identification Type                  VEKP.SPE_IDART_01

DAX Columns: LIPS1, Lips-delivery_KEY (CC-12)
Tracking Logic: CASE WHEN IFNULL(EXIDV2,'')='' THEN 'No Tracking' ELSE EXIDV2 END
Conversion Status: ✅ FULL


### PQ-15 | VTTP_VTTK_TVROT_Left_Join
─────────────────────────────────────────────────────────────
PBI lineageTag  : 5c6cdb2e-e7c1-4a41-870b-d1974ead0f87
SAP Tables      : CV_VTTP ← CV_VTTK (LEFT JOIN) ← CV_TVROT (LEFT JOIN)
Join keys       : VTTP.TKNUM = VTTK.TKNUM
                  COALESCE(VTTK.EXTI1, VTTK.ROUTE) = TVROT.ROUTE
WHERE           : VTTP.ERDAT >= '20220301'
⚠️  NOTE        : TVROT route join uses COALESCE(EXTI1, ROUTE) — prefers
                  external carrier ID over SAP route code. This drives
                  the [Carrier] column and [Route Description] lookup.

Key source column mapping:
PBIP Column Name                        SAP Field              Notes
────────────────────────────────────────────────────────────────────────────────
Delivery(VTTP)                          VTTP.VBELN             ⚠️ No space before (VTTP)
Shipment Number                         VTTP.TKNUM             NVARCHAR(10)
Shipment Number (Null Removed)          IFNULL(TKNUM,'No Shipment') NOT NULL
MANDT_VTTP                              VTTP.MANDT             Renamed collision
Date Created (VTTP)                     VTTP.ERDAT
Ship Date                               TO_DATE(VTTK.DALBG)    NULL-safe DATE; Short Date
External identification 1               VTTK.EXTI1
External identification 2               VTTK.EXTI2
Bill of Landing                         VTTK.EXTI2             ⚠️ Typo "Landing" not "Lading"
Carrier                                 COALESCE(EXTI1,ROUTE)
Plant                                   VTTK.TPLST             Dup alias
Transportation planning point           VTTK.TPLST             Dup alias
Forwarding Agent Tracking ID            VTTK.TNDR_TRKID        NVARCHAR(35)
ROUTE                                   TVROT.ROUTE
Route Description                       TVROT.BEZEI

SQL-computed filter columns:
  Weekly Ship Creation Filter    : 'Current' if Ship Date >= TODAY()-7, else 'Historic'
  Daily Ship Creation Filter     : Monday-aware: if Mon → TODAY()-3; else → TODAY()-1
  Monthly Shipment Filter        : 'Current' / 'Previous' / ISO month-start date string
  ABC Spec Weekly Ship Creation Filter : Identical logic to Weekly filter (separate column)

DAX Column: Month Year of Ship Date (CC-13)
Conversion Status: ✅ FULL


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 6 — MODEL METADATA
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PBIP Report Name          : Customer Shipment Report
PBIP Model Name           : Customer Shipment Report - model
Compatibility Level       : 1600
Schema Version            : 4.2  (definition.json)
Culture                   : en-IN
PBI Data Source Version   : powerBI_V3
Time Intelligence         : Enabled  (__PBI_TimeIntelligenceEnabled = 1)
Connection Mode           : DirectQuery (all tables)
SAP HANA Endpoint         : pahpdb01.net.bms.com:30015

Total Tables              : 28
Delivery Stream Tables    : LIPS-Delivery, LIKP, VBAK_VBAP_left_join,
                            VBAP, VBPA_ADRC_Left_Join,
                            MARA_MAKT_MARM_Delivery_Left_Join,
                            TVAGT, STXL, Payer_KNA1,
                            VTTP_VTTK_TVROT_Left_Join,
                            VEPO_VEKP(Tracking)_Inner_Join,
                            VBRK, VBRP
Back Order Stream Tables  : VBAK_VBAP_left_join (Back Order),
                            VBAP (Back Order),
                            VBPA_ADRC_Left_Join (Back Order),
                            MARA_MAKT_MARM_Delivery_Left_Join (Back Order),
                            TVAGT (Back Order),
                            VBEP(Back Order),
                            KNA1_Sold_to (Back Order)
Auxiliary Tables          : Measure_Table, BB slicer, Drop-ship Slicer,
                            Back-OrderStatus, Base QTY, Invloice QTY,
                            Back Order

Tableau Workbook Source   : customer_shipment_report.twbx (XML export)
Tableau Data Source       : SAP HANA — pahpdb01.net.bms.com:30015
                            All transactional data from S4 table CVs


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 7 — RELATIONSHIPS
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All relationships are single-column joins. Composite keys are
pre-built as DAX calculated columns (see Section 2) and then joined
as single string columns.

Key Relationship Groups:

GROUP A — Delivery Stream
From Table                              From Column             To Table                        To Column
───────────────────────────────────────────────────────────────────────────────────────────────────────────
LIPS-Delivery                           VBAP_KEY                VBAK_VBAP_left_join             Lips_KEY2
LIPS-Delivery                           VBAP_KEY                VBAP                            Lips_KEY
LIPS-Delivery                           Delivery                LIKP                            Delivery
LIPS-Delivery                           VEPO_KEY                VEPO_VEKP(Tracking)_Inner_Join  Lips-delivery_KEY
LIPS-Delivery                           Delivery                VTTP_VTTK_TVROT_Left_Join       Delivery(VTTP)
VBAK_VBAP_left_join                     Sales Document (VBAP)   VBPA_ADRC_Left_Join             Sales and Distribution Document Number
VBAK_VBAP_left_join                     Reason for Rejection    TVAGT                           Reason for Rejection of Sales Document
VBAP                                    Material Number (VBAP)  MARA_MAKT_MARM_Delivery_Left_Join Material Number
VBRP                                    VBRP Billing Document   VBRK                            Billing Document (VBRK)
VBRP                                    Payer                   Payer_KNA1                      Customer Number
VBRP                                    Sales Document (VBRP)   VBAK_VBAP_left_join             Sales Document

GROUP B — Back Order Stream
From Table                              From Column             To Table                                    To Column
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
VBAK_VBAP_left_join (Back Order)        vbep_KEY                VBEP(Back Order)                            VBAP_KEY
VBAP (Back Order)                       VBEP_KEY                VBEP(Back Order)                            VBAP_KEY
VBAK_VBAP_left_join (Back Order)        Sales Document (VBAP)   VBPA_ADRC_Left_Join (Back Order)            Sales and Distribution Document Number
VBAK_VBAP_left_join (Back Order)        Reason for Rejection    TVAGT (Back Order)                          Reason for Rejection of Sales Document
VBAP (Back Order)                       Material Number (VBAP)  MARA_MAKT_MARM_Delivery_Left_Join (BO)      Material Number
VBAK_VBAP_left_join (Back Order)        Sold-To Party           KNA1_Sold_to (Back Order)                   Customer Number
VBEP(Back Order)                        VBAP_KEY                VBAK_VBAP_left_join (Back Order)            vbep_KEY
STXL                                    TDNAME                  VBAK_VBAP_left_join (Back Order)            Sales Document

GROUP C — Slicer Tables
From Table          From Column  To Table                            Notes
───────────────────────────────────────────────────────────────────────────────
BB slicer           BB_or_SB     VBAK_VBAP_left_join (Back Order)   BB/SB filter slicer
Drop-ship Slicer    (key)        VBAK_VBAP_left_join                Drop-ship classification
Invloice QTY        (key)        VBRP                               ⚠️ Typo "Invloice" preserved
Base QTY            Sales Doc    VBRP                               Sales Document(VBRP) — no space


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 8 — INTENTIONAL NAMING ARTIFACTS REGISTER
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

The following naming artifacts are intentional, sourced from Tableau,
and MUST be preserved exactly in all TMDL files. Do NOT autocorrect.

Type             Table / Context                          Value                               Note
──────────────────────────────────────────────────────────────────────────────────────────────────────────
Intentional typo Invloice QTY  (table name)               "Invloice"                          Not "Invoice"
No space         Base QTY      (column in slicer table)   Sales Document(VBRP)                No space before (VBRP)
No space         VBAP / VBAP (Back Order)                 Date of creation(VBAP)              No space before (VBAP)
No space         VBEP(Back Order)                         Sales Document(VBEP)                No space before (VBEP)
No space         VBEP(Back Order)                         Sales Document Item(VBEP)           No space before (VBEP)
No space         VTTP_VTTK_TVROT_Left_Join                Delivery(VTTP)                      No space before (VTTP)
No space         VBEP(Back Order)  (table name)           VBEP(Back Order)                    No space in table name
Case mismatch    Payer_KNA1                               (part 1) vs (Part 2)                Lowercase vs uppercase P
_ → X rename     VBAP / VBAP (Back Order)                 _KJEPS_WS_SUB → XKJEPS_WS_SUB      SAP leading _ replaced
_ → X rename     VBEP(Back Order)                         _DATAAGING → XDATAAGING__...        Same pattern
! prefix         MARA_MAKT_MARM_Delivery_Left_Join        !Brand                              Exclamation prefix
! prefix         MARA_MAKT_MARM_Delivery_Left_Join (BO)   !Brand(back order)                  Exclamation + no space
! prefix         VBAK_VBAP_left_join (both)               !Order Creation Date                Exclamation prefix
Intentional typo VTTP_VTTK_TVROT_Left_Join                Bill of Landing                     Should be "Lading" — preserve
Duplicate alias  VBRP                                     Net Value / Net value of the billing item... / Invoice Amount  All = NETWR
Duplicate alias  VBRP                                     Payer / Payer #                     Both = KUNRG_ANA
Duplicate alias  VBRP                                     Ship-to party / Ship-To #           Both = KUNWE_ANA; case differs
Duplicate alias  VBRK                                     Billing Document (VBRK) / VBELN     Both = VBELN
Duplicate alias  VTTP_VTTK_TVROT_Left_Join               Plant / Transportation planning point  Both = TPLST
Duplicate alias  VBPA_ADRC_Left_Join (both)              City / Ship-To City                 Both = CITY1
Full-path name   VEPO_VEKP(Tracking)_Inner_Join          Batch Number (SAP_ERP.TABLE_CV_S4::CV_VEPO)  CHARG from VEPO
Full-path name   VEPO_VEKP(Tracking)_Inner_Join          Batch Number (SAP_ERP.TABLE_CV_S4::CV_VEKP)  CHARG from VEKP
Hyphen in name   VEPO_VEKP(Tracking)_Inner_Join          Lips-delivery_KEY                   Contains hyphen in DAX col


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 9 — DUAL-STREAM ARCHITECTURE REFERENCE
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

This model replicates the Tableau multi-datasource pattern as two
fully mirrored table streams in a single PBI model.

Concept:
  Tableau had separate datasource connections for Delivery and Back Order.
  PBI merges them into one model with "(Back Order)" suffix tables for
  the back order stream. Both streams query the same SAP HANA views.

Stream Mapping:
Tableau Datasource          PBI Main Stream Table                     PBI Back Order Table
────────────────────────────────────────────────────────────────────────────────────────────────────
Delivery (VBAK/VBAP join)   VBAK_VBAP_left_join                       VBAK_VBAP_left_join (Back Order)
Item detail (VBAP wide)     VBAP                                      VBAP (Back Order)
Material master             MARA_MAKT_MARM_Delivery_Left_Join         MARA_MAKT_MARM_Delivery_Left_Join (Back Order)
Ship-To address             VBPA_ADRC_Left_Join                       VBPA_ADRC_Left_Join (Back Order)
Rejection reason lookup     TVAGT                                     TVAGT (Back Order)
Schedule lines              (not in main stream)                      VBEP(Back Order)

Key structural differences between streams:
Feature                   Main Stream                Back Order Stream
─────────────────────────────────────────────────────────────────────────────────────
Block classification      Not present                BB_or_SB (CC-05)
Composite key columns     Lips_KEY / Lips_KEY2       vbep_KEY / vbep2 / VBEP_KEY
REMS DAX column           Present (CC-02)            Not present
Schedule lines table      Not used                   VBEP(Back Order) (PQ-10)
Sold-To dimension         (in VBAK_VBAP source)      KNA1_Sold_to (Back Order)
Long text (notes)         Not used                   STXL joined via TDNAME
Block/slicer tables       Not applicable             BB slicer, Back-OrderStatus


---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## SECTION 10 — KNOWN ISSUES / NOTES
## ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAP-01 | POMALYST 7-char truncation bug in REMS logic  ⚠️ PRE-EXISTING
  Severity : Medium
  Detail   : The SQL REMS Products column uses LEFT(MAKTG, 7) to check
             for POMALYST (8 chars). LEFT(...,7) returns "POMALY", which
             never matches "POMALYST". The DAX [REMS] column inherits
             the same bug via [!Brand] which also uses a 7-char prefix.
             Result: POMALYST products are classified as "Other" instead
             of "REMS" in both SQL and DAX layers.
             This bug exists in Tableau and is intentionally preserved
             for fidelity. DO NOT FIX without a formal change request.
  Action   : Document for business review. If correction is approved,
             update the SQL CASE to LEFT(MAKTG,8) and the DAX SWITCH
             to use Prefix8 = "POMALYST".

GAP-02 | Language filter TREATAS dependency  ⚠️ CRITICAL
  Severity : High
  Detail   : MARA_MAKT_MARM tables default Language to 'E' via SQL
             IFNULL. TREATAS is additionally used in DAX queries to
             enforce Language = 'E' context. If the TREATAS filter is
             ever removed or the SQL IFNULL default changes, multi-
             language rows may appear in product visuals.
  Action   : Do not remove the TREATAS filter from any DAX query
             referencing these tables. Verify after any model refresh
             that Language context is applied correctly.

GAP-03 | INNER JOIN on VEPO_VEKP excludes unpackaged deliveries
  Severity : Medium
  Detail   : VEPO_VEKP(Tracking)_Inner_Join uses an INNER JOIN on
             VEPO.VENUM = VEKP.VENUM. Deliveries without a Handling
             Unit (HU) record in VEKP are excluded from tracking data.
             This is intentional (mirrors Tableau behaviour) but means
             unpackaged shipments will show no tracking information.
  Action   : If coverage of unpackaged deliveries is required, change
             INNER JOIN to LEFT JOIN and handle NULL tracking columns.

GAP-04 | Carrier / Route Description COALESCE join
  Severity : Info
  Detail   : VTTP_VTTK_TVROT joins to TVROT using
             COALESCE(EXTI1, ROUTE) = TVROT.ROUTE. When EXTI1 is
             populated (external carrier code), it is used as the join
             key; when blank, ROUTE is used. The Route Description
             returned may therefore be for either an external carrier
             ID or the SAP route — depending on the row.
  Action   : When filtering or displaying Route Description, verify
             whether the Carrier column (EXTI1 or ROUTE) aligns with
             the expected Route Description value.

GAP-05 | VBRK.NETWR sumBy:none
  Severity : Info
  Detail   : Billing header net value (VBRK.NETWR) has summarizeBy:none.
             Aggregation should be done at item level using VBRP.NETWR
             (mapped to Net Value / Invoice Amount). Do not use VBRK.NETWR
             in measure calculations.
  Action   : Ensure all invoice amount measures reference VBRP columns,
             not VBRK.NETWR.

GAP-06 | Monthly Shipment Filter returns 3 value types  ⚠️ IMPORTANT
  Severity : Medium
  Detail   : The SQL Monthly Shipment Filter column returns:
               "Current"        — for current month rows
               "Previous"       — for previous month rows
               "YYYY-MM-DD"     — ISO month-start date string for older
             This mixed-type string column (type VARCHAR(5000)) requires
             special handling in PBI slicers. Filter logic must account
             for all three value patterns. Sorting by this column will
             sort alphabetically ("Current" before "Previous" before dates).
  Action   : Build a sort column or ordinal rank measure if chronological
             ordering of the Monthly Shipment Filter slicer is required.

GAP-07 | Daily Ship Creation Filter Monday special logic
  Severity : Info
  Detail   : On Mondays (DAYOFWEEK = 2 in SAP HANA), the Daily filter
             looks back 3 days to cover Friday/Saturday/Sunday shipments.
             On all other days it matches only yesterday. This is
             computed at query time in DirectQuery and refreshes live.
  Action   : Verify DAYOFWEEK SAP HANA convention (1=Sunday vs 1=Monday)
             matches the implemented logic. SAP HANA DAYOFWEEK: 1=Sunday,
             2=Monday, confirmed by the SQL condition DAYOFWEEK=2 for Monday.

GAP-08 | VBEP(Back Order) has no date filter — full table pull
  Severity : Medium
  Detail   : VBEP(Back Order) pulls the entire CV_VBEP table with no
             WHERE clause. All other transactional tables filter on
             ERDAT >= '20220301'. VBEP performance depends entirely on
             relationship-pushed filters at query time.
  Action   : Monitor DirectQuery performance for back order pages.
             If performance degrades, consider adding ERDAT filter
             matching other tables, with confirmation from business.

GAP-09 | "Bill of Landing" typo in VTTP_VTTK_TVROT
  Severity : Info
  Detail   : Column is named "Bill of Landing" (not the correct
             "Bill of Lading"). This is preserved from the Tableau
             source. The column maps to VTTK.EXTI2 (same as External
             identification 2 — intentional duplicate alias).
  Action   : Do not correct the spelling. Document in any data dictionary
             that this column name is a preserved Tableau artifact.

GAP-10 | STXL joins on TDNAME to Sales Document — text concatenation risk
  Severity : Info
  Detail   : STXL.TDNAME (NVARCHAR 70) joins to VBAK_VBAP_left_join
             (Back Order)[Sales Document] (NVARCHAR 10). TDNAME stores
             the SAP text object key which may include concatenated
             identifiers beyond the 10-char sales document. Verify
             the join produces the expected 1:1 or N:1 cardinality
             for back order notes retrieval.
  Action   : Review TDNAME values in sample data. Confirm join key
             format matches Sales Document format exactly.
