# Prompt: BI Migration Accelerator deck

Copy everything below the line into a new conversation. Replace anything in `[square brackets]` with your own facts, or delete the bracketed line if you want the model to flag it as an assumption.

---

Create a PowerPoint presentation about an internal web application that migrates BI content from Tableau, Qlik Sense, Cognos and SAP BusinessObjects into Power BI. The audience is my delivery team and our leadership team in the same room, so it needs to work for both: enough technical substance that engineers trust it, framed so leadership can make a decision.

## The single message the deck must land

"Migrate to Power BI" sounds like one problem. It is four. The difficulty varies enormously by source platform, and the hard part sits in a different place each time — so **each source platform needs its own conversion knowledge base**. Scope, estimate and staff the work per platform, not as one generic migration.

## Source material — use these mappings verbatim, one slide per platform

**Tableau → Power BI** (closest conceptual match; the difficulty is semantic)
- LOD expressions → DAX
- Parameters → What-if parameters / disconnected tables
- Calculated fields → Measures / calculated columns
- Dashboards → Power BI report pages

**Qlik Sense → Power BI** (different again; Qlik carries its own ETL layer *and* an associative interaction model Power BI does not have)
- Load script (QVDs, resident loads) → Power Query (M) / dataflows
- Associative data model → Star schema semantic model
- Set analysis expressions → DAX filter context (CALCULATE)
- Master items & variables → Measures / what-if parameters
- Sheets, selections & bookmarks → Report pages, slicers & bookmarks
- Section Access → Row-level security (RLS)

**Cognos → Power BI** (a very different problem; the difficulty is structural — the centre of gravity is the enterprise modelling layer, not the report)
- Framework Manager model → Power BI semantic model
- Report expressions → DAX
- Cognos prompts → Power BI filters / parameters
- Cognos reports → Power BI visuals

**SAP BusinessObjects → Power BI** (another distinct problem; the difficulty is architectural)
- Universe → Power BI semantic model
- WebI variables → DAX
- WebI prompts → Power BI filters
- WebI reports → Power BI report pages

On each of those four slides, add a "where the effort actually goes" note calling out the genuinely hard construct — LOD/filter-context semantics for Tableau; the associative model and selection state for Qlik (green/white/grey selections and alternate states have no Power BI equivalent, and the whole load script has to move to Power Query); the Framework Manager model for Cognos; the universe for SAP BO.

## Slide plan (14 slides)

1. **Title** — dark. Headline on the "one platform, four different migration problems" idea.
2. **Why we are building this** — the cost of manual migration: rework, knowledge silos, inconsistent output, no traceability. Close with the opportunity: encode the conversion knowledge once per platform and apply it consistently.
3. **What the application does** — five-step pipeline: Ingest → Parse → Map → Generate → Validate. Note that human review sits between Map and Generate; the app proposes, the developer approves.
4. **The key insight** — dark pivot slide. The difficulty varies enormously by source platform, with a one-line characterisation of each of the four: Tableau semantic, Qlik behavioural, Cognos structural, SAP BO architectural.
5. **Tableau → Power BI** — mapping table.
6. **Qlik Sense → Power BI** — mapping table.
7. **Cognos → Power BI** — mapping table.
8. **SAP BusinessObjects → Power BI** — mapping table.
9. **Comparison** — a native clustered column chart of relative effort (1–5) across five workstreams: semantic model rebuild, data prep & load logic, expression conversion, prompts/selections & parameters, visual layout rebuild. Label it explicitly as a team judgement to be recalibrated after a pilot — do not present it as measured data.
10. **Architecture** — one shared engine, four pluggable source knowledge bases, Power BI artefacts out. Make the point that adding a fifth source means writing a new knowledge base, not rebuilding the engine.
11. **What sits inside a conversion knowledge base** — object mappings, function dictionary, conversion patterns, exception register, validation rules, naming standards. Frame it as the durable IP; the app is just the delivery mechanism.
12. **What automates and what will not** — three columns: fully automated / assisted, review required / stays manual. Be honest that modelling judgement, business validation and change management stay human. Set analysis, load-script translation and Section Access all belong in the assisted column.
13. **Delivery approach** — five phases with exit criteria: [engine + Tableau first] → [pilot and calibrate] → [Qlik Sense KB] → [Cognos KB] → [SAP BO KB].
14. **Asks** — close on the decisions being requested: [confirm source priority], [name a pilot estate], [allocate platform SMEs], [agree the success measure].

## Design requirements

- 16:9 widescreen, 13.3" × 7.5".
- Dark navy title and closing slides, light content slides in between.
- Palette: deep navy primary, ice blue secondary, one sharp amber accent. Give each source platform its own consistent colour used across every slide it appears on — four colours that stay distinguishable side by side in a chart legend (e.g. blue, crimson, purple, teal).
- Serif headers paired with a sans body. Slide titles 32–36pt bold, body 12–15pt.
- Every slide needs a visual element — cards, icons in coloured circles, a process flow, a real chart. No plain title-and-bullets slides.
- No accent stripes, no coloured bars under titles, no decorative edge borders.
- For the mapping slides use a source-concept pill → arrow → Power BI-equivalent pill layout, repeated identically on all four so the comparison is obvious. Qlik has more rows than the others — shrink the row height on that slide rather than letting it overflow.
- Use arrow *shapes* rather than typed arrow characters, so glyphs don't substitute badly.
- Add speaker notes to every slide with the point to make out loud.

## Constraints

- Do not invent metrics, percentages, timelines, headcounts or cost figures. Where a number would normally go, either leave it out or mark it clearly as an assumption to be validated.
- Anything you assume rather than take from this brief, list back to me at the end so I can correct it.
- Render the deck and visually check every slide for text overflow, overlap and alignment before you hand it over.
