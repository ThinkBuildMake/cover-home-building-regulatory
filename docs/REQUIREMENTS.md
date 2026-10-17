# REQUIREMENTS.md — Home Building Regulatory Engine

**Status:** draft

**Purpose.** Turn the problem and users in `docs/USERS.md` (with `docs/USERS_handoff.md`) into testable requirements and constraints with numeric thresholds, so the key-metrics doc can measure against these numbers, and the stack and architecture docs can be chosen against them rather than justified afterwards. Every requirement traces to a use case, a given constraint, or a "must never" in the users doc.

**Scope.**

*This document decides:* what the system must do per use case in the minimum product (users doc §6 first iteration), the "must never" rules, numeric thresholds for correctness, performance, reliability, observability, cost and scalability, how each is verified, and what is out of scope by design. Later iterations get functional rows only when they enter a build; until then they appear only as change-readiness rows (CH-).

*This document defers:*
- Technology choices and tradeoffs → the stack document.
- Components, data flow, schemas, and the techniques used to meet a budget (caching, pre-computation, parallelism) → the high-level design and architecture documents.
- Why the problem exists and the cut line → `USERS.md` §2, §3, §7.
- Flows and personas → `USERS.md` (cited, not restated).
- Which requirements become tracked metrics and dashboards → the key-metrics document.

If a row below names a library, a component or a way of meeting a threshold, it is in the wrong document.

**How to read a requirement.**

| Field | Meaning |
|---|---|
| **Requirement** | *Functional:* from the user's side, no number. *Non-functional and given:* from the system's side; the requirement is the number itself, tagged. If the number isn't met, the requirement fails. |
| **Verified by** | Non-functional and given rows: the test, eval, log query or review that proves it. Functional rows are verified by a user seeing or using the feature. |
| **Trace** | `U§n` = users doc section n; `UC-n` = use case; `A-n` = users-doc assumption; `LI-n` = later-iteration entry n in `USERS.md` §6. |
| **Priority** | **Gate** = must hold before real data. **Must** = required for the product to be done. **Should** = required unless a documented tradeoff says otherwise. This document does not cut scope for the one-week deadline; if a Gate or Must is deferred, the builder decides that when stories are written. |

**Where the numbers come from.** **[src]** = from a cited source, a given constraint, or computed from one. **[prop]** = proposed target; the origin is shown in parentheses when it comes from a provisional figure upstream, e.g. `[prop (U§5 A-4)]`. **[tbd]** = cannot be set until a prototype or measurement exists; the row says what sets it. Section 5 lists every [prop] and [tbd].

**Assumptions.**
1. There is no separate problem statement. `USERS.md` §2, §3 and §7 (problem, why-now, out of scope) stand in for it.
2. All of `USERS.md` A-1 to A-11 carry forward. No meeting with Cover is possible, so several numbers here (notably the 10 s latency, A-4) are the builder's proposals, not Cover's.
3. The proof of concept is used by a small internal group with no login or per-user permissions. This removes an authorization group. If real client data or external users appear, one must be added.
4. The "range of provided parcels" in the brief has not been received. The eval set in EV-1 is built by the builder and should be replaced or extended when Cover's parcels arrive.
5. The Cover brief (`docs/Cover - Home Building Regulatory Engine.pdf`) and the PRD (`docs/Portal PRD.md`) are the source of truth. Where `USERS.md` or an earlier draft of this document differed from them, they govern; section 0a records each such change.

**Definitions.**

| Term | Meaning |
|---|---|
| Assessment | The full result for one parcel, one building type and one set of project inputs: the constraint list, the not-determined list, the map. |
| Constraint | One rule's outcome for the parcel and building type (e.g. "rear setback 15 ft"), with value, units, citations, derivation, label and confidence. The unit that is cited, labelled and graded. |
| Rule set | The rules the engine evaluates in v1: whether the building type is permitted on the parcel, zone, height, setbacks, lot coverage, floor area, ADU size, overlay flags (A-8). |
| Rule fragment | One rule from the regulatory text stored in structured form (topic, building types, parameters, jurisdiction) and tied to the source passage it was taken from. The unit that ingestion produces. |
| Buildability summary | The block at the top of an answer that states, for the chosen building type, whether it is permitted on the parcel and the headline limits. It repeats values from the constraints below it and adds none. |
| Deterministic | A constraint whose value is computed from parcel/zoning data and a numeric rule with no language model in the path. Tested by VR-2. |
| Interpreted | A constraint whose value depends on a language model's reading of prose. Always shown apart from deterministic ones. |
| Confidence signal | A displayed indicator, per constraint, of how far to trust it, with at least three levels (high, medium, low). |
| Citation | A pointer to a specific passage in a specific source document that the user can open. |
| Derivation | The visible steps from parcel data and the cited rule to the constraint's value. |
| Not-determined item | A rule in the rule set that the engine could not resolve for this parcel, listed with a reason. |
| Project inputs | Building type, attached/detached, bedroom count, bathroom count, building footprint/floor area (UC-2). |
| Eval case | One parcel + building type + project inputs with hand-checked expected constraints (EV-1). |
| Data snapshot | A dated, versioned copy of the parcel, zoning and regulatory data an answer was computed from. |

---

## 0. Requirements summary (one screen)

| Area | Headline threshold |
|---|---|
| What is allowed | Every answer opens with a buildability summary: building type permitted or not, and the headline limits, each traced to a constraint |
| Honesty | 100% of constraints cited, labelled, with confidence; 0 wrong-parcel answers; every gap named |
| Ingestion | 100% of rules in the rule set stored as structured fragments tied to a source passage and checked by hand against it |
| Usability | A first-time target user completes an assessment and opens a citation unaided in ≤ 2 min, in 3 of 3 trials |
| Correctness | 100% deterministic values match hand-checked truth on the eval set |
| Speed | p95 ≤ 10 s for first answer and for re-answer [prop (U§5 A-4)] |
| Failure | Any single dependency failing gives a named partial answer or explicit error, never a silent gap |
| Stale answers | 0 cases of an old answer shown as current after an input changes |
| Scale | 5 concurrent users; unseen parcels need 0 code changes |
| Eval | ≥ 30 hand-checked cases (10 parcels × 3 building types) |
| Given | 1 week, one builder, Python + JS/TS, LA City residential, public or labelled mock data |

---

## 0a. Key decisions

| # | Decision | Alternative rejected | Where |
|---|---|---|---|
| 1 | A small rule set done with full citations | Broad coverage with weak sourcing (brief: "small, well-reasoned subset") | FR-2, VR-1 |
| 2 | "Deterministic" is defined by an LLM-free path and tested by disabling the LLM | Labelling by the builder's judgement | VR-2 |
| 3 | An unresolved or ambiguous parcel yields no answer rather than a best guess | Always return the closest match (U§6: wrong parcel is worse than none) | FR-1.3, VR-4 |
| 4 | A partial answer with named gaps beats failing whole when a dependency is down | All-or-nothing answers | RL-1, DE-3 |
| 5 | One shared 10 s budget for first answer and re-answer | Separate looser budget for UC-2 (U§5 gives both the same) | PF-1, PF-2 |
| 6 | The tool states what the rules allow for the lot and building type (permitted or not, and the limits). It does not say a specific design is approved or will pass plan check | No statement of what is allowed (brief: "a clear statement of what is allowed"); or an approve/reject verdict on a drawn design (U§7: architect decides). This reverses the earlier "constraints only" wording | FR-2.11, FR-4.2, VR-12 |
| 7 | Mock data is allowed but every datum is tagged real or mock | Silent mock data (handoff: must be clear) | VR-8 |
| 8 | Regulatory ingestion into structured, cited rule fragments is a requirement of its own | Treating ingestion as build detail (brief: "core focus") | IN-1 to IN-3, FR-2.4 |
| 9 | Re-answering as inputs change stays in v1 although the brief lists it as a bonus | Dropping it (PRD impact metric: "real-time buildability answers as inputs change") | FR-3, GC-2 |
| 10 | Usability of the screen is a requirement, not a styling matter | Leaving it to the builder's judgement (brief must-have: "easily see the outputs") | UX-1 |

---

## 1. Givens

### 1.1 Given constraints (GC-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| GC-1 | The proof of concept is delivered within 1 week of start [src U§8]. | Delivery date against start date | U§8 | Must |
| GC-2 | If only half is built, the order is: the brief's must-haves first (FR-1, FR-2, the map, the eval set, the architecture diagram, the five deliverables), then FR-3 (a brief bonus kept because the PRD lists it as an impact metric), then the other bonus items: feedback beyond the confirmed/wrong mark, setback geometry, chat, admin view [src U§8, A-10, brief]. | Build order review at the midpoint | U§8, A-10 | Must |
| GC-3 | One builder; no row requires an operations team or on-call [src U§8]. | Review of deployment steps for manual ops work | U§8 | Should |
| GC-4 | Backend is Python; frontend is JavaScript or TypeScript [src U§8]. | Repository language check | U§8 | Must |
| GC-4a | Parcel geometry is stored and processed with PostgreSQL, PostGIS and Shapely, as listed in the PRD's "Other Requirements" (the brief calls them recommended; the builder adopts them) [src PRD, user decision]. | Review of the technical docs and repository dependencies | U§8 | Must |
| GC-5 | AWS is the default cloud; GCP is used only for geocoding; any other cloud service is recorded with its reason [src U§8]. | Review of the services list in the technical docs | U§8 | Should |
| GC-6 | Only residential parcels inside the City of Los Angeles are assessed. All other parcels (outside the city, non-residential) return an explicit "unsupported" outcome in 100% of test cases [src U§8, U§7]. | 6 test inputs: 3 outside the city, 3 non-residential | U§7, U§8 | Must |
| GC-7 | Data is public, or mock data that is labelled as such. Every datum an answer uses is tagged real or mock (see VR-8) [src U§8]. | Source inventory in the docs | U§8 | Must |
| GC-8 | The five deliverables exist at submission: source code, technical documentation, demo video, test results, an architecture diagram showing productionization [src U§8]. | Checklist at submission | U§8 | Must |
| GC-10 | The PRD marks AI-usage documentation optional; a short note on where AI was used in building this is included in the technical docs [src PRD]. | Present in the docs | PRD | Should |
| GC-9 | The licence of every data source and model used is recorded; the use is internal only [src U§8, A-11]. | Source inventory shows a licence entry for 100% of sources | A-11 | Should |
| GC-11 | A current large language model is used for regulatory synthesis: reading prose for interpreted constraints and extracting rule fragments from text [src brief "use the latest in LLM/AI/ML"]. | The technical docs name the model and each place it is used | Brief | Must |

### 1.2 Dependencies and environment (DE-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| DE-1 | If the geocoder fails or times out, the user is told within 5 s [prop] and no parcel is guessed. | Fault injection: geocoder returns error and delays | UC-1, U§6 | Must |
| DE-2 | Every answer states the date of the data snapshot it used; 100% of answers carry it. | Assert on all eval outputs | UC-1 | Must |
| DE-6 | A data snapshot older than 30 days [prop] is flagged on screen in 100% of answers that use it. | Test with an aged snapshot | DE-2, KEY_METRICS | Should |
| DE-3 | If the language-model service is unavailable, the deterministic constraints are still returned within PF-1 and the interpreted ones are listed as not determined with reason "interpretation unavailable". | Fault injection: model calls fail | UC-1, UC-2 | Gate |
| DE-4 | A clean machine runs the system and the eval suite from the README in ≤ 30 min [prop]. | Timed run on a fresh container | U§8 | Should |
| DE-5 | Each run records the language-model name and version and the prompt version [prop]. | Log query on 100% of runs | UC-1 | Must |

---

## 2. Functional requirements

*Each is verified by a user seeing or using the feature at the story's demo. Numbers are in §3.*

### FR-1 Start an assessment (UC-1)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-1.1 | She enters an address or an APN and chooses single-family home, ADU or guest house. | UC-1 | Must |
| FR-1.2 | She sees which parcel was resolved (address and APN) alongside the answer, so she can confirm it is the lot she meant. | UC-1, U§6 | Must |
| FR-1.3 | If the input matches no parcel or more than one, she is told so, and shown the candidates when there are several. No assessment is shown until one parcel is chosen. | UC-1, U§6 | Gate |
| FR-1.4 | If the parcel is outside the City of Los Angeles or not residential, she is told it is unsupported and why. | U§7 | Must |
| FR-1.5 | She sees the parcel outline and existing buildings on a map. If no existing building is recorded for the parcel, the map says so. | UC-1, brief | Must |
| FR-1.6 | The resolved parcel and its map appear as soon as the parcel is found, before the assessment finishes. | UC-1, PF-3 | Should |

### FR-2 Read the constraints (UC-1)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-2.1 | She sees a list of constraints for the chosen building type on that parcel, covering the rule set. | UC-1, A-8 | Must |
| FR-2.2 | Each constraint shows its value and units, and which building type it applies to. | UC-1 | Must |
| FR-2.3 | Each constraint has a citation she can open to the source passage. | UC-1 | Gate |
| FR-2.4 | Each constraint shows its derivation: the parcel data, the structured rule fragment applied, and the source text that fragment was taken from. | UC-1, brief | Must |
| FR-2.5 | Each constraint is labelled deterministic or interpreted, and the two kinds are visually separated. | UC-1 | Gate |
| FR-2.6 | Each constraint shows a confidence signal. | UC-1 | Must |
| FR-2.7 | Where the text is ambiguous or incomplete, she sees the reasonable readings (decision paths) laid out, apart from verified constraints, each with its source and what would settle it. | UC-1, brief | Must |
| FR-2.8 | She sees a list of rules in the rule set that could not be determined, each with a reason. | UC-1 | Gate |
| FR-2.10 | She marks each constraint "confirmed" or "wrong", and can change the mark. Free-text feedback is not in v1 (LI-1). | UC-1, UC-2, KEY_METRICS | Must |
| FR-2.9 | The answer states that it covers only the rule set and is not the full set of regulations on the lot. | UC-1 | Gate |
| FR-2.11 | The answer opens with a buildability summary for the chosen building type: whether that type is permitted on this parcel (permitted, not permitted, or not determined) and the headline limits (height, setbacks, floor area, and ADU size where it applies). Each item links to the constraint below that backs it. | UC-1, brief, PRD | Must |

### FR-3 Change the project (UC-2)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-3.1 | On the open parcel she changes project inputs (building type, attached/detached, bedrooms, bathrooms, footprint/floor area) without re-entering the parcel. | UC-2 | Must |
| FR-3.2 | The answer updates for the new inputs. | UC-2 | Must |
| FR-3.3 | She sees what changed compared with the answer that was on screen: constraints whose value changed, and constraints that now apply or no longer apply. | UC-2 | Must |
| FR-3.4 | While a new answer is pending, the previous one is visibly marked out of date. | UC-2 | Gate |
| FR-3.5 | The new answer carries the same citations, labels, confidence and not-determined list as FR-2. | UC-2 | Must |

### FR-4 v1 boundaries

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-4.1 | Not in v1: free-text feedback on bad answers (LI-1; the confirmed/wrong mark in FR-2.10 is in v1), drawn setbacks and annotations (LI-2), chat (LI-3), admin pipeline view (LI-4), other jurisdictions (LI-5). | U§6 | Must |
| FR-4.2 | The tool states what the rules allow for the lot and building type. It does not say that a specific design is approved or will pass plan check. | U§7, brief | Must |

---

## 3. Non-functional requirements

*Columns: ID · Requirement · Verified by · Trace · Priority. Sections 3.9 (ingestion) and 3.10 (usability) were added after checking this document against the brief. Groups absent: Authorization (Assumption 3, no login), Compliance and data handling (public parcel data only; licences are in GC-9), Deployment (covered by DE-4 and GC-8).*

### 3.1 Verification / correctness (VR-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| VR-1 | 100% of constraints carry at least one citation that resolves to a stored source passage [src U§5 "must never"]. | Automated check on every eval output | FR-2.3 | Gate |
| VR-2 | 100% of constraints are labelled. With the language model disabled, every constraint labelled deterministic is unchanged, and no constraint labelled deterministic needs the model [src U§5]. | Run the eval set twice (model on, model off) and diff | FR-2.5 | Gate |
| VR-3 | 100% of deterministic values match hand-checked expected values on the eval set [prop]. | Eval suite (EV-1) | FR-2.2 | Must |
| VR-4 | 0 wrong-parcel answers across the eval inputs; an input that cannot be resolved to one parcel is reported as such [src U§6]. | Eval inputs include 3 ambiguous and 3 non-matching inputs | FR-1.2, FR-1.3 | Gate |
| VR-5 | For interpreted constraints, the cited passage supports the stated value in ≥ 90% of cases on manual review [prop]. | Builder reviews all interpreted constraints in the eval set | FR-2.3, FR-2.4 | Must |
| VR-6 | Constraints marked high confidence are correct at least [tbd: set from the first full eval run; interim 90% [prop]] of the time. | Eval suite: accuracy of high-confidence constraints | FR-2.6 | Must |
| VR-7 | In eval cases where a rule's data is deliberately removed (≥ 3 cases), the rule appears in the not-determined list in 100% of cases and never as a constraint [src U§5]. | Seeded-gap eval cases | FR-2.8 | Gate |
| VR-8 | 100% of data items an answer uses are tagged real or mock; any answer using mock data shows that on screen [src handoff]. | Assert on eval outputs; one UI check | GC-7 | Must |
| VR-9 | After any input change, the displayed answer matches the current inputs in 100% of tested change sequences; 0 cases of an old answer shown as current [src U§5]. | Scripted sequences of ≥ 10 changes, including rapid successive changes | FR-3.4 | Gate |
| VR-10 | The change summary equals the set difference between old and new answers in 100% of ≥ 10 eval pairs. | Eval pairs with expected diffs | FR-3.3 | Must |
| VR-11 | The same inputs give identical deterministic values in 10 of 10 repeated runs; interpreted values agree with each other in ≥ 90% of repeated runs [prop]. | Repeat each eval case 10 times | U§5 consistency | Must |
| VR-12 | In 100% of eval outputs, every value in the buildability summary equals the value of the constraint it links to. A summary item is labelled deterministic only if every constraint behind it is deterministic, and shows the lowest confidence among them [prop]. | Automated check on every eval output | FR-2.11 | Gate |

### 3.2 Performance (PF-)

*Not requirements: how the budget is met. The 10 s is one shared budget for geocoding, data retrieval, interpretation and rendering; the split is the high-level design's.*

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| PF-1 | First answer for a parcel: p95 ≤ 10 s from submit to full assessment [prop (U§5 A-4)]. | Timed run over the 30 eval cases, repeated 3 times | UC-1 | Must |
| PF-2 | Re-answer after a project input change: p95 ≤ 10 s [prop (U§5 A-4)]. | Timed run over the 10 change sequences | UC-2 | Must |
| PF-3 | The map shows the parcel within 3 s p95 of a resolved parcel [prop]. | Timed run over eval inputs | FR-1.5 | Should |

### 3.3 Reliability and failure modes (RL-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| RL-1 | When any one dependency (geocoder, data source, model) is made to fail, 100% of cases return either an explicit error or a partial answer that names each missing part; 0 cases return an answer with a silent gap [src U§5]. | Fault injection on each dependency across 5 eval cases | FR-2.8 | Gate |
| RL-2 | Every request ends in an answer or an error within 30 s [prop (3 × PF-1)]. An error names the stage that failed or was still running. | Fault injection with stalled dependencies | UC-1, UC-2 | Must |
| RL-3 | A request that fails leaves the previously displayed answer clearly marked as out of date, never as current. | UI test after a forced failure | FR-3.4 | Must |

### 3.4 Observability (OB-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| OB-1 | For 100% of answers a record is kept of: inputs, data snapshot, model and prompt versions, cited passage identifiers, and timestamps. Replaying a record reproduces the deterministic values. | Replay 10 records and diff | UC-1, UC-2 | Must |
| OB-4 | Citation opens, input changes and confirmed/wrong marks are each recorded as events tied to the answer identifier for 100% of occurrences. | Perform each action 10 times and query the log | FR-2.10, KEY_METRICS | Must |
| OB-2 | Elapsed time per request is recorded for 100% of requests, with a breakdown by stage. | Log query | PF-1, PF-2 | Should |
| OB-3 | Model and cloud cost per request is recorded for 100% of requests. | Log query | CO-1 | Should |

### 3.5 Contracts and interfaces (CT-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| CT-1 | 100% of constraints in every output validate against one published schema containing: rule, value, units, building type, label, confidence, citations, derivation, jurisdiction. | Schema validation on every eval output | FR-2.2 to FR-2.6 | Must |

### 3.6 Evaluation suite (EV-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| EV-1 | The eval set has ≥ 10 parcels × 3 building types = ≥ 30 cases, covering ≥ 3 zones and ≥ 2 parcels with an overlay, each with expected constraints checked by hand against the code [prop]. | Inspect the fixture file | GC-8, A-8 | Must |
| EV-2 | The set includes ≥ 3 ambiguous-text cases, ≥ 3 seeded-gap cases, ≥ 3 ambiguous or non-matching inputs and ≥ 10 input-change pairs [prop]. | Inspect the fixture file | VR-4, VR-7, VR-10 | Must |
| EV-3 | One command runs the whole suite on a clean checkout and writes a results file (the "test results" deliverable). | Run on a fresh container | GC-8 | Must |
| EV-4 | Line coverage ≥ 80% on the code that evaluates rules and computes the change summary [prop]. | Coverage report | U§8 "production-ready" | Should |

### 3.7 Cost (CO-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| CO-1 | Median model and cloud cost per assessment ≤ [tbd: no budget was given; set after the first full eval run; interim $0.25 [prop]]. | Cost log (OB-3) | A-11 | Should |

### 3.8 Scalability (SC-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| SC-1 | Supports 5 users asking at once with p95 within PF-1 [prop (internal team size)]. | Load test with 5 parallel sessions | UC-1 | Should |
| SC-2 | Any residential parcel in the City of Los Angeles can be assessed with 0 parcel-specific code or configuration: ≥ 5 parcels not used during development produce valid assessments with 0 code changes [src brief: scale beyond sample parcels]. | Hold-out run | GC-6 | Must |

### 3.9 Regulatory ingestion (IN-)

*The brief names ingestion and synthesis as the core focus: turning municipal zoning text into structured, explainable rule fragments.*

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| IN-1 | 100% of rules in the rule set exist as structured rule fragments, each with a topic, the building types it covers, parameters, a jurisdiction identifier and the identifier of the source passage it was taken from [src brief]. | Query of the rule store; schema check | FR-2.4, VR-1 | Gate |
| IN-2 | One command rebuilds the rule store and the passage store from the stored source documents on a clean checkout, and the rebuilt rules equal the committed ones in 100% of fragments (no hidden manual edits). | Rebuild and diff on a fresh container | GC-8, DE-4 | Must |
| IN-3 | The builder checks 100% of rule fragments against their source passage before the fragment is used, and each fragment records how it was made (read from a table, or extracted by the language model) and that it was checked [prop]. | Query: fragments without a checked flag = 0 | FR-2.4, VR-5 | Must |

### 3.10 Usability (UX-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| UX-1 | A person who has not seen the tool, given only the README, enters a parcel, picks a building type, reads the buildability summary and opens one citation within 2 minutes, in 3 of 3 trials [prop (brief: "easily see the outputs")]. | Three timed walkthroughs by people other than the builder | FR-1, FR-2 | Must |

### 3.11 Change and later iterations (CH-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| CH-1 | A new rule can be added to the rule set with 0 changes to the output schema (CT-1) and 0 changes to the UI code. | Add one held-back rule as a test | LI-5 | Should |
| CH-2 | 100% of constraints and sources carry a jurisdiction identifier, so a second jurisdiction can coexist with LA. | Schema check (CT-1) | LI-5 | Should |
| CH-5 | The architecture diagram (GC-8) shows how the same design extends from LA City to LA County and other regions [src PRD "scalable architecture"]. | Diagram review at submission | LI-5 | Must |
| CH-3 | 100% of answers and constraints have a stable identifier retrievable from the record in OB-1, so feedback and chat can refer to them. | Retrieve 10 random constraints by identifier | LI-1, LI-3 | Should |
| CH-4 | 100% of setback constraints give distance, units and which lot side (front, side, rear), so they can later be drawn on the map. | Schema check on setback constraints | LI-2 | Should |

---

## 4. Traceability matrix

| Requirement group | UC-1 | UC-2 | Later iterations | Source |
|---|---|---|---|---|
| GC | ● | ● | LI-5 | U§8 |
| DE | ● | ● | | U§5, U§6 |
| FR-1, FR-2 | ● | | | U§5, U§6 |
| FR-3 | | ● | | U§5 |
| FR-2.10 | ● | ● | LI-1 (partial) | KEY_METRICS |
| FR-4 | | | LI-1 to LI-5 | U§6, U§7 |
| VR | ● | ● | | U§5 "must never" |
| PF | ● | ● | | A-4 |
| RL, OB | ● | ● | | U§5 |
| CT, EV | ● | ● | | U§8 |
| CO, SC | ● | | | A-11, brief |
| FR-2.11, VR-12 | ● | ● | | brief, PRD |
| IN | ● | ● | LI-5 | brief scope |
| UX | ● | ● | | brief must-have |
| CH | | | LI-1 to LI-5 | U§6 |

---

## 5. Proposed thresholds

Every **[prop]** and **[tbd]** number. Status is *accepted*, *revised*, or *pending prototype*. [prop] rows not commented on at review are accepted; [tbd] rows are never accepted by silence.

| Row | Value | Reasoning | Status |
|---|---|---|---|
| DE-1 | Geocoder failure reported ≤ 5 s | Half the shared budget; leaves time to retry or show an error | draft |
| DE-4 | Clean install ≤ 30 min | A reviewer should not give up before seeing it run | draft |
| DE-6 | Snapshot older than 30 days flagged | Zoning and parcel data change slowly, so a month is stale enough to warn; the real change rate is unknown | draft |
| DE-5 | Model and prompt version recorded | Needed to reproduce an interpreted answer | draft |
| VR-3 | 100% deterministic match | A table lookup has no excuse for an error | draft |
| VR-5 | ≥ 90% cited passage supports the claim | Interpreted rules are fuzzier; below 90% an architect would stop trusting citations | draft |
| VR-6 | High confidence accurate ≥ [tbd] (interim 90%) | Depends on how the confidence signal is built | pending prototype |
| VR-11 | 10/10 deterministic; ≥ 90% interpreted | Brief asks for consistency across parcels; the model adds variation | draft |
| VR-12 | Summary equals constraints in 100%; weakest label and confidence carried up | A summary that is surer than its parts would break the honesty rule | draft |
| IN-3 | 100% of fragments checked by hand | The rule set is small, so a full check is affordable and the brief stresses traceability | draft |
| UX-1 | 2 minutes, 3 of 3 trials | "Easily see the outputs" has no number in the brief; two minutes is a guess at what an architect would accept before giving up | draft |
| PF-1, PF-2 | p95 ≤ 10 s | The brief says "real-time" with no number; 10 s from A-4 | draft |
| PF-3 | Map ≤ 3 s | Something appears quickly, so the wait feels shorter | draft |
| RL-2 | 30 s hard stop | 3 × PF-1 | draft |
| EV-1, EV-2 | 10 parcels, 30 cases and the listed seeded cases | The brief says "a range of provided parcels"; replace when Cover sends them | draft |
| EV-4 | Coverage ≥ 80% | The brief asks for production-ready code with proper testing | draft |
| CO-1 | Median cost ≤ [tbd] (interim $0.25) | No budget given; keeps eval runs affordable | pending prototype |
| SC-1 | 5 concurrent users | Internal team size, guessed | draft |

**Open items only a running system (or an upstream fix) can answer:**
1. Which public datasets give LA parcels, zoning and buildings, and how current they are. This decides DE-2 and the real shape of FR-1.5.
2. How much of the rule set is numeric tables versus prose (A-5). This decides how many interpreted constraints exist and whether VR-5 and VR-6 matter much.
3. Whether ZIMAS or the city's data can be queried programmatically, which affects PF-1.
4. Cover's own parcels, once received, to replace the EV-1 fixture.
5. A real latency tolerance from Cover to replace A-4.
