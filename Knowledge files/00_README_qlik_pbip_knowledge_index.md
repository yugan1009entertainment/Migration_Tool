# 📘 QLIK SENSE → POWER BI (PBIP/TMDL) MIGRATION KNOWLEDGE BASE
# Index & Usage Guide
# Generated: 2026-08-27
# Companion to: k61.md (Tableau → PBIP conversion knowledge file)

---

## WHY THIS SET IS STRUCTURED DIFFERENTLY FROM THE TABLEAU FILE

The Tableau knowledge file (k61.md) is a **single per-workbook artifact**: it records
what one workbook became in PBIP. That works for Tableau because Tableau calc fields
map to DAX fairly directly and the .twb already carries a resolved data model.

Qlik Sense needs **two layers**:

  LAYER 1 — STANDING RULE REFERENCES (files 02–06)
    Qlik has three constructs with no Power BI equivalent at all:
      • Load script (an ETL language, not a query) → must become Power Query / SQL
      • Associative model (bidirectional by nature, synthetic keys) → must become a star schema
      • Set analysis + Aggr() (context syntax) → must become CALCULATE/iterator patterns
    These need reusable rule tables, not per-app notes. The tool looks up rules here.

  LAYER 2 — PER-APP CONVERSION FILE (file 01, the template)
    One filled copy per migrated Qlik app, in the same shape as k61.md.
    This is what accumulates as migration history.

---

## FILE MANIFEST

| # | File                                          | Layer | Purpose                                                        |
|---|-----------------------------------------------|-------|----------------------------------------------------------------|
|00 | 00_README_qlik_pbip_knowledge_index.md         | —     | This file. Index + tool consumption order.                     |
|01 | 01_qlik_pbip_conversion_knowledge_TEMPLATE.md  | 2     | Per-app conversion record. Copy → fill → keep.                 |
|02 | 02_qlik_script_to_powerquery_reference.md      | 1     | Load script → M / native SQL. SCR-xx rules.                    |
|03 | 03_qlik_datamodel_to_tabular_reference.md      | 1     | Associative model → star schema. DM-xx rules.                  |
|04 | 04_qlik_set_analysis_to_dax_reference.md       | 1     | Set analysis + Aggr → CALCULATE. SET-xx / AGG-xx rules.        |
|05 | 05_qlik_expression_to_dax_reference.md         | 1     | Function-by-function mapping. FN-xx rules.                     |
|06 | 06_qlik_ui_to_pbi_report_reference.md          | 1     | Sheets/objects → pages/visuals. UI-xx rules.                   |
|07 | 07_qlik_gap_catalog_and_validation.md          | 1     | Known unmappable features + validation protocol. QGAP-xx.      |

---

## RECOMMENDED TOOL PROCESSING ORDER

The order matters — later stages depend on decisions made in earlier ones.

```
STAGE 0  Extract app
         Unzip .qvf → app metadata, load script, data model (tables/fields),
         master items, variables, sheets/objects, bookmarks, section access.
         (Qlik Engine API: GetScript, GetAllInfos, GetObject, GetTablesAndKeys,
          GetVariableList, or qsr/QRS REST for published apps.)

STAGE 1  Data model FIRST  → apply 03_datamodel rules
         Resolve synthetic keys, link tables, circular refs BEFORE touching
         expressions. The star schema you land on changes every ALLEXCEPT /
         relationship decision downstream. Do not skip.

STAGE 2  Load script       → apply 02_script rules
         Decide per table: fold to native SQL | Power Query M | keep as
         calculated table. Emits DS-xx entries in the per-app file.

STAGE 3  Field mappings    → emit column table (Section 3 of per-app file)
         Qlik field name → PBI column name + dataType + summarizeBy.

STAGE 4  Variables         → apply 04 §VAR rules
         Classify each: constant | expression macro | user-input parameter.
         Emits PARAM-xx / VAR-xx entries.

STAGE 5  Master measures   → apply 04 + 05 rules
         Set analysis first (04), then residual function mapping (05).
         Emits M-xx entries.

STAGE 6  Master dimensions → calculated columns or model columns
         Emits CC-xx entries.

STAGE 7  Sheets & objects  → apply 06 rules
         Emits report-layer notes. Lowest fidelity stage — expect ⚠️ PARTIAL.

STAGE 8  Gaps + validation → apply 07
         Emits QGAP-xx entries and the reconciliation checklist.
```

---

## STATUS VOCABULARY (keep identical to the Tableau file)

| Symbol | Meaning                                                                 |
|--------|-------------------------------------------------------------------------|
| ✅ FULL | Logic converted, results expected to match.                             |
| ✅ FULL (different approach) | Different implementation, same result. Note the approach. |
| ⚠️ PARTIAL | Converted but needs report-layer config or manual completion.        |
| ⚠️ VERIFY | Converted but numeric equivalence not proven. Owner must reconcile.   |
| ❌ GAP  | No equivalent. Requires redesign or is dropped. Must have an owner.    |
| ➕ PBI-only | Exists in PBI with no Qlik source. Enhancement or plumbing.         |

Every ⚠️ and ❌ **must** carry an `Owner:` line. Unowned gaps are how migrations
silently ship wrong numbers.

---

## HARD RULES FOR THE TOOL (apply to every app)

**R-01 — Never auto-accept a bidirectional relationship.**
Qlik's associative engine filters in all directions natively. The naive port is
bidirectional PBI relationships everywhere. This produces ambiguity errors,
wrong totals, and bad performance. Default to single-direction; escalate to
bidirectional only where a documented Qlik behaviour requires it (see DM-07).

**R-02 — Preserve names exactly, including quirks.**
Qlik field names are case-insensitive on reference but case-preserving on
display; Qlik permits spaces, `%` prefixes (`%KeyCustomer`), and duplicated
names across tables. TMDL is case-sensitive. Record every rename in the
per-app file. Typos and double spaces in the target must be preserved as-is
once shipped (same discipline as CC-04 / PARAM-06 in the Tableau file).

**R-03 — `Sum({1<...>})` is not `CALCULATE(SUM(...), ...)`.**
The `1` set identifier discards *all* selections. Forgetting `REMOVEFILTERS()`
is the single most common Qlik→DAX defect. See SET-03.

**R-04 — Aggr() is never a calculated column by default.**
Qlik Aggr() is evaluated at chart time in the current selection state. Porting
it to a calculated column freezes it at refresh time. Default to an iterator
measure (SUMX/AVERAGEX over VALUES/SUMMARIZE). Only demote to a column when
the grain is genuinely static. See AGG-01.

**R-05 — Null semantics differ.**
Qlik NULL propagates and is excluded from associations; `Sum()` of an empty
set returns 0, not null. DAX `SUM()` over an empty set returns BLANK. Test
every KPI that can hit an empty selection. See FN-40.

**R-06 — Dual() values have a display face and a numeric face.**
`Dual('High', 3)` sorts by 3 and shows 'High'. In PBI this becomes a column
plus a separate sort column with `sortByColumn`. Never drop the numeric face.
See FN-22.

**R-07 — Section Access is a data-reduction mechanism, not just a filter.**
It reduces the loaded model per user. RLS filters at query time and does not
reduce the dataset. Rows the user cannot see are still in the PBIX file.
This is a security-posture change and must be signed off, not silently ported.
See QGAP-02.

---

## PER-APP FILE NAMING CONVENTION

```
qlik_pbip_<AppName>_<YYYY-MM-DD>.md
e.g. qlik_pbip_SalesPerformance_2026-08-27.md
```

Keep every revision. When the Qlik app is updated, diff the new extraction
against the previous per-app file and only convert the delta — that is the
whole point of keeping these.

---
# END OF INDEX
