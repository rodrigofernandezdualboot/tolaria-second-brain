---
type: Note
belongs_to: "[[pacific-seafood]]"
related_to: "[[technical-intake-brief-pacific-seafood-as-400-invoice-flow-discovery-poc]]"
---

# Pacific Seafood - Implementation plan

Execution plan for the AS/400 invoice-flow discovery POC. Four stages (formerly "anchors"): three produce evidence, the fourth converts evidence into documented business logic. Staffing constraint baked in: **no in-house RPG expert** — de-risked through intensive AI use, a named client-side validator, and an on-call RPG contractor as safety valve.

## Team

| Role | Seniority | Allocation | Covers |
|---|---|---|---|
| AI/agent engineer | Senior | Full-time | Stages 2–4: AI guidance, skill, orchestration, reconciliation |
| .NET engineer | Mid/Senior | Part-time | Stage 1, correlation ID in Stage 3 |
| Tooling/QA engineer | Mid | Full-time | Stage 2 scripting, Stage 3 test design & execution |
| Business analyst | Mid/Senior | Part-time → full in Stage 4 | ERP-relevant artifacts, gap view, workshop prep |
| Engagement lead | Senior | Part-time | Governance, validation cadence, client escalation |
| Client AS/400 SME (Ryan's team) | — | **Named, committed h/week — gating input** | Validation + unblocking (compiles, library lists, member routing) |
| RPG contractor | Senior | On-call, ~10–20h total | Tie-breaks when AI and client SME disagree |

## Stage 1 — .NET entry points

**Goal:** Convert "analyze the AS/400 estate" into "start from these named entry programs with these parameters." Zero RPG knowledge required — that is the point of this stage.

**Tasks:**

- Inventory the .NET integration code for the three invoice flows.
- Identify the invocation mechanism into the AS/400 (program call, stored procedure, data queue, file drop).
- Extract named entry programs and parameter payloads per flow; document payload schemas.
- Map SQL artifacts participating in the invoice path.

**Risks:**

- Invocation mechanism turns out indirect (e.g., staged through SQL replication) → entry points harder to name. Mitigation: integration/BI perimeter mapping before concluding.
- Some invoice logic lives in the .NET/SQL layer itself, not the AS/400. Mitigation: document it here; don't assume the box does everything.

## Stage 2 — Structure extraction

**Goal:** A derived, defensible scope boundary: the transitive closure of programs and files reachable from the entry points. Deterministic — no model interpretation involved.

**Tasks:**

- Run native extraction: `DSPPGMREF`, `DSPFD`/`DSPDBR`, DDS layouts, DB cross-reference files.
- Script the call graph and compute the reachable set from Stage 1 entry points.
- Produce the branch inventory (feeds Stage 3 test design).
- Analyze CL for library-list and member-routing context.
- Snapshot the analyzed library set (codebase is live; drift control).
- **Client SME review:** one bounded session to sanity-check the reachable set, especially member routing.

**Risks:**

- Member routing / library-list context misread → wrong pathway documented. Mitigation: this is the client SME's named review, not an AI judgment call.
- Dynamic calls (`CALL` with variable program name) invisible to static cross-reference. Mitigation: flag as gaps; Stage 3 traces confirm actual call paths.

## Stage 3 — Runtime evidence

**Goal:** Ground truth. Journaling captures what data changed; traces capture which path executed and why. This converts validation from "re-derive the logic" into "check the claim against the trace" — the asymmetry that makes the no-RPG-expert model viable.

**Tasks:**

- Design the trace physical file (correlation ID, timestamp, job, program, tag, field, value) and a single logging subprocedure in a service program.
- Pick injection points from the branch inventory: branch/decision points, file writes, program entry/exit, reused-field snapshots, .NET→RPG handoff.
- AI writes the instrumentation (one-line calls only); **client SME code-reviews the diff before any compile**; Bob-assisted compile in non-prod.
- Native-trace fallback (`STRDBG`/`TRCJOB`) for programs that won't compile or that the client won't let us touch.
- Start journaling (`STRJRNPF`) on A/P, A/R, and relevant master files.
- Design test invoices per flow with PGT finance SMEs, targeting known branches including error paths.
- Execute runs with correlation IDs; harvest journal receivers + trace rows into one evidence bundle per invoice.

**Risks:**

- Non-prod environment or compile access not granted → stage collapses, approach degrades to static-only. **Gating; resolve before SOW pricing.**
- Recompile failures (missing `/COPY`, target-release mismatch). Mitigation: native-trace fallback per program.
- Coverage illusion: traces only prove executed paths. Mitigation: test design against branch inventory; unexercised logic ships labeled `inferred`.
- Client discomfort with source modification even in non-prod. Mitigation: native-trace-first posture, injection only where field values are essential.

## Stage 4 — Extraction & synthesis

**Goal:** Turn evidence into validated, ERP-relevant business-logic documentation. The model never guesses; it explains what the evidence shows.

**Tasks:**

- Build the Dualboot reverse-engineering skill encoding the eight hard traits (member routing, field reuse, RPG-cycle semantics, integration perimeter, etc.) as mandatory checks.
- Stand up engine + Flow orchestration; ingest the evidence store (source, structure, traces, journals, Pacific's prior docs).
- Run extraction per flow; fixed-form→free-form transform as reading aid (analysis-only).
- Reconcile every claim against trace/journal; auto-flag contradictions.
- Cross-model divergence filter: claims where two models agree *and* match runtime evidence go to the client SME as "confirm"; everything else arrives pre-flagged as "resolve."
- Confidence labels on every rule: confirmed / inferred / conflicting / deprecated.
- SME validation workshops in small batches on fixed cadence (no big-bang review).
- RPG contractor tie-break on AI-vs-SME disagreements.
- BA converts validated logic into deliverables: narratives, ERDs, Mermaid diagrams, business-rule catalog, gap view (retain / review / retire).

**Risks:**

- No internal RPG pre-validation gate → client SME sees rawer drafts. Mitigation: divergence filter does the triage.
- Client SME overload → rubber-stamp validation, destroying the trust claim. Mitigation: committed hours in SOW, batched cadence, evidence-first artifacts.
- Validator circularity: Ryan's team's beliefs are both input and acceptance gate. Mitigation: runtime evidence as independent arbiter + RPG contractor tie-break.
- RPG comprehension gap (cycle semantics, fixed-format idioms). Mitigation: Stages 1–3 removed most inference burden; skill forces explicit checks.

## Technical tasks — step-by-step

### Phase 0 — Access & setup (Engagement lead + AI engineer)

1. Confirm non-prod IBM i environment with run, journaling, and compile authority.
2. Confirm read-only source access incl. all `/COPY` copybooks and target release.
3. Confirm named client SME and weekly hour commitment; agree validation cadence.
4. Resolve engine data-governance question (does source leave Pacific's environment).
5. Set up evidence repository and correlation-ID convention.

### Stage 1 (.NET engineer, ~1 week)

1. Clone/read the .NET integration solution; locate handlers for the three invoice flows.
2. Trace each flow to the AS/400 boundary call; record mechanism and target object names.
3. Dump parameter schemas per flow (field names, types, sample payloads).
4. Identify SQL objects (stored procs, staging tables) on each path.
5. Deliver: entry-point map, one page per flow.

### Stage 2 (Tooling engineer + AI engineer, ~1–2 weeks)

1. Run `DSPPGMREF *ALL` to outfiles; extract DB cross-reference (`QSYS` XREF files) and `DSPFD`/`DSPDBR` for candidate libraries.
2. Load outputs into SQL/Python; build directed graph program→program and program→file.
3. Seed with Stage 1 entry programs; compute transitive closure → reachable set.
4. Extract DDS for every file in the set; build field/record-layout catalog.
5. Parse CL sources for library-list manipulation and member overrides (`OVRDBF`); annotate the graph with routing context.
6. Generate branch inventory per program (AI-assisted parse of RPG source, flagged not trusted).
7. Client SME session: review reachable set + routing annotations; log corrections.
8. Snapshot library sources (date-stamped) for drift control.

### Stage 3 (AI engineer + tooling engineer + .NET engineer + client SME, ~2–3 weeks)

1. Create trace physical file and `TraceLog` service program; compile.
2. AI generates injection diffs per program (one-line calls at points from the branch inventory); batch per program.
3. Client SME reviews each diff batch; approved batches compiled via Bob into non-prod.
4. Programs that fail compile or review → register for native tracing (`STRDBG`/`TRCJOB`).
5. .NET engineer threads correlation ID from integration call into entry parameters (or trace-side correlation if payload can't change).
6. Start `STRJRNPF` on A/P, A/R, and master files in the reachable set.
7. With PGT finance SME, define test invoice matrix per flow: happy path + known variants + error cases mapped to branch inventory.
8. Execute runs; after each, extract journal receiver entries and trace rows by correlation ID into an evidence bundle.
9. QA pass: verify each bundle is complete (entry→exit trace continuity, journal entries present); rerun gaps.

### Stage 4 (AI engineer + BA + client SME + engagement lead, ~3–4 weeks, overlaps Stage 3)

1. Author the RE skill: eight-trait checklist as mandatory analysis steps; test on a sample program against known behavior.
2. Ingest evidence store into Flow orchestration; fixed→free transform of reachable RPG as reading copies.
3. Per flow: run extraction → draft rule set with evidence links.
4. Run reconciliation: every claim matched to trace/journal lines; contradictions auto-flagged `conflicting`.
5. Run second model over hard sections; agreement + evidence match → "confirm" queue; divergence → "resolve" queue.
6. Weekly validation workshop: client SME processes confirm/resolve queues; decisions logged (confirmed / corrected / deprecated).
7. Escalate unresolved AI-vs-SME disagreements to RPG contractor (batched, to respect the ~10–20h budget).
8. BA builds per-flow deliverables from confirmed rules: current-state narrative, ERD segments, Mermaid workflows, rule catalog with confidence labels.
9. BA builds gap view: confirmed behavior vs Pacific's requirements/swim lanes → hidden-logic list, sorted retain / review / retire.
10. Final review with Gene + Ryan: sign-off per flow against success criteria; open items logged as known gaps, never silently dropped.

## Dependencies that gate the plan

1. Non-prod environment + compile authority (Stage 3 dies without it).
2. Named client SME with committed hours (validation model dies without it).
3. Source + copybooks + correct target release.
4. Pacific's prior requirements/swim lanes (gap view input).
5. Representative invoice cases per flow.
