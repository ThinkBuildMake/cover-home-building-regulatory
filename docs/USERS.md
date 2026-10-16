# USERS.md — Home Building Regulatory Engine

## Scope

**This document covers:** who the engine is for and who it is not for; what goes wrong for them today, with sources; the moments it enters their day; the use cases and why this shape of product is the one they'd choose; the minimum first iteration and what comes after; what is out of scope; and the constraints the project was given.

**This document does not cover:** how the product is built, verified, measured or hosted. Requirements, key metrics, stack and architecture are separate documents that trace back to the use case IDs (UC-n) and the constraints here.

**Status:** Provisional draft, written from the Portal PRD and the Cover brief (`docs/Cover - Home Building Regulatory Engine.pdf`) only. The builder (Michael) has no firsthand knowledge of how Cover's architects work today, so everything about their day is marked *(guess)*. Starred items in `MEETING_QUESTIONS.md` are open.

---

## 1. Target user

**A Cover design architect (or engineer) who knows the zoning code and building regulations, working on a residential lot in the City of Los Angeles at the start of, and during, a project.** The brief names the initial target user as "an Architect or Engineer with knowledge of the code and regulations" who can "validate the claims made by the tool", reads maps and parcels, and understands technical regulatory language. That this person is on Cover's own design team (rather than a client's) comes from the brief's "speed up the design team's ability to assess property buildability". *(guess: in-house; confirm.)*

Their job at this point is to decide what can realistically be built on the site before design effort, pricing and timelines are committed. They are responsible for noticing what a parcel's zoning, overlays and other rules allow, and for not promising a client something the city will refuse. *(guess: how they do this today, which tools and sources they open, how long it takes.)*

**Not the target user:** a homeowner or client reading the answer directly; a sales or intake person who needs a quick yes/no and cannot check the citations (the brief requires that the user can validate the claims, which excludes them for now); a city plan checker.

---

## 2. The problem, in numbers

There are no figures for this problem from Cover. The brief says the process is "slow and manual due to fragmented regulatory research" and that it creates uncertainty affecting "early design exploration, client conversations, pricing, and timelines", and sets "reduce regulatory research effort to zero" as a goal. It gives no hours, frequency or cost of a surprise. *(guess: all of it; asked in the meeting questions.)*

What the public sources do show, about how the research is structured:

### The research spans separate sources by design
The City of Los Angeles says most parcels are subject to some kind of overlay (Historic Preservation Overlay Zones, Specific Plans, Community Design Overlays) that "limits or further restricts what can be built beyond what the zoning allows" [1]. The City's own steps are to look the zone up in ZIMAS, then in the Zoning Code (published separately through American Legal Publishing), then to check additional regulations; ZIMAS identifies the zone but does not replace reading the code [1][2]. ADUs add a third layer, state law, with its own deadlines (for example, a 15-business-day completeness decision from the city under SB 543, effective 2026-01-01) [3].

### The downstream cost shows up as plan-check corrections
LADBS issues correction sheets for single-family and duplex plans and for ADUs, with supplemental sheets for fire district, grading and hillside, and requires compliance "prior to the issuance of the permit" [4]. This shows that compliance with zoning and building code is checked after design, not that Cover's projects fail it often. I did not find an official rate of correction for zoning issues. A private consultancy publishes a breakdown (zoning and setback issues at 22% of corrections) [5], but its source is its own unverified data and I do not use the figure.

---

## 3. Why we target these moments

The engine enters two moments of the architect's work: the first answer about a parcel, and the re-answer when the project changes. It does not try to replace permitting, plan check, or the architect's judgement; the goal is to put correct, sourced constraints in front of them in the minutes where a wrong assumption would shape the design, not to save the most hours overall.

**Reason 1: the cost of a wrong early assumption lands later.** Early design, pricing and client expectations rest on what the lot allows (brief). If the assumption is wrong, it surfaces at plan check [4]. *(guess: how often and how expensive; unquantified.)*

**Reason 2: the answer is spread across rules that interact.** Base zone, overlays, specific plans and ADU state law each constrain the same lot [1][3]. That they must be combined is documented; how error-prone it is for Cover is not.

**Reason 3: the question changes as the project does.** The brief says the understanding "must be continuously refined rather than treated as a one-time determination" and lists responding to project inputs (a bedroom count) as a goal. The second question builds on the first.

### Out of scope: moments not targeted
Permit submission, plan-check correction responses, structural, energy and civil compliance, and construction-phase inspections. These are different work at a different time and are not part of deciding what can be built.

---

## 4. Workflow: where the engine enters the day *(guess: all of it)*

**Monday, 9:30 AM.** Dana, a Cover design architect, has just been handed a new lead: a single-story house on a lot in Los Angeles. The client wants an ADU behind it.

**The 30 seconds before opening it:** she has an address and the client's wish. She does not yet know the zone, the overlays, or the setbacks, and she does not want to promise the client anything. → **UC-1**

**What she needs from it:** what zone and overlays apply, what that means for a house or ADU on this lot, where each rule came from, and which parts are fixed by the code versus the tool's reading of it. She expects to check the parts that matter against the code herself.

**What she does with the output:** she decides whether to proceed, what to tell the client, and what she'll confirm with the city. She then asks, "what if it's two bedrooms, and attached?" → **UC-2**

**Thursday, 2:00 PM.** The client wants to add a second bedroom and a larger footprint. She needs to know whether the lot still allows it, and what changed since Monday. → **UC-2**

---

## 5. Use cases

### UC-1: First answer for a parcel

| | |
|---|---|
| **Trigger** | She enters an address or APN for a lot in the City of Los Angeles and picks a building type (single-family home, ADU, guest house). |
| **Input** | The address or APN and the building type. Nothing else. |
| **What they need to see** | The parcel and any existing buildings on a map; the constraints that apply to that building type on that lot; for each constraint, where it came from (which source and which passage); which constraints are read straight from parcel and zoning data and which are the tool's interpretation of ambiguous text; how confident the tool is in each; and what it could not determine. |
| **What they do with it** | Decide whether the lot can take what the client wants, what to tell the client, and what to confirm with the city. The tool does not make the decision. |
| **Time available** | *(guess: unknown; to ask.)* |
| **Must never** | State an interpretation as if it were a verified fact. Give an answer with no source she can check. Look equally sure when it is not. Present a rule as complete when it covered only part of the lot's regulations. |

**Why this shape, not ZIMAS plus the municipal code:** An architect who knows the code can already do this by hand, and a lookup of the zone and its numeric standards (height, setbacks, lot coverage) is deterministic and a rule table could return it. The product has to earn its place on what a table can't: (1) much of the governing text is prose spread across the code, overlays and specific plans, and applying it to one lot means reading and combining passages, which is the "regulatory ingestion and synthesis" the brief names as the core; (2) where the text is ambiguous or incomplete, a table has nothing to say, while the architect needs the reasonable readings laid out and kept apart from verified constraints; (3) the same answer must come out the same way for every parcel, with citations, which hand research cannot promise. This depends on how much of Cover's real research is prose-reading versus lookup *(guess; asked in the meeting questions)*. If it is nearly all lookup, the value is speed and citations, not synthesis, and the shape should be reconsidered.

---

### UC-2: Re-answer when the project changes

| | |
|---|---|
| **Trigger** | The client or architect changes something about the project, such as building type, bedroom or bathroom count, or size. |
| **Input** | The parcel already open and the changed project detail. |
| **What they need to see** | What changed in the answer, which constraints now bind that did not before, and the same sourcing and confidence as UC-1. |
| **What they do with it** | Decide whether the changed design is still allowed, and tell the client. |
| **Time available** | *(guess: asked in the middle of a client conversation or while designing; unknown.)* |
| **Must never** | Silently keep an old answer after an input changed. Present the new answer without saying what it replaced. |

**Why this shape, not a fixed report:** A report is accurate for one set of inputs and goes stale the moment the project changes; the architect would redo the research each time. A rule table can recompute numeric limits for the new inputs, and the tool should not claim more than that for those. What a table can't do is re-read the prose rules that depend on the new details (an ADU's rules differ from a guest house's) and show the architect what moved. This case weakens if, in Cover's real projects, the inputs that change are mostly ones that only move numeric limits *(guess)*.

---

## 6. The minimum product

**First iteration.** An architect would open this on a Tuesday to get, for a lot in the City of Los Angeles, a cited and confidence-marked list of the constraints on a single-family home, ADU or guest house, with the parcel on a map, and to change a few project details and see the answer update. If it covered a handful of rules well, with every claim checkable, the architect would still use it; if its claims could not be checked, they would not. *(guess; to confirm.)*

| # | Feature | Use case | What it does for the user | Acceptable in v1 |
|---|---|---|---|---|
| 1 | Enter an address or APN and a building type | UC-1 | Starts an assessment for a lot | *(to ask)*: address/APN must resolve to the right parcel |
| 2 | Parcel and existing-building map | UC-1 | Lets her see the lot she is asking about | *(to ask)* |
| 3 | List of applicable constraints for a small set of rules | UC-1 | Gives the constraints that apply | Fewer rules is acceptable if each is correct *(from brief: "small, well-reasoned subset")* |
| 4 | Citation and derivation for each constraint | UC-1 | Lets her check it against the code | Every constraint has a source she can open *(guess)* |
| 5 | Deterministic versus interpreted label, plus confidence | UC-1 | Shows what to trust and what to check | Distinction always present *(from brief)* |
| 6 | Change a few project details and re-answer | UC-2 | Keeps the answer current | *(to ask: how long she'd wait)* |

**Later iterations,** in the order the brief suggests (to confirm with the user's order):

| # | Feature | Use case | What it adds for the user | Why not first |
|---|---|---|---|---|
| 1 | Feedback on a bad answer | UC-1 | The tool improves where it was wrong | Needs the minimum in use first (brief: bonus) |
| 2 | Visual setbacks and annotations on the map | UC-1, UC-2 | Sees the constraint on the lot | Needs reliable constraints first; adds geometry work |
| 3 | Chat for follow-up questions | UC-2 | Asks about the answer in plain language | Needs the structured answer first |
| 4 | Admin view of the ingestion pipeline | n/a (builder/operator) | Lets operators adjust the system | Not something the architect uses |
| 5 | More jurisdictions (LA County, other California cities) | UC-1 | Covers more of Cover's projects | Brief says design for it, but build one city first |

---

## 7. Explicitly out of scope

| Not doing | Why |
|---|---|
| Full regulatory coverage of the City of Los Angeles | The brief asks for a small, well-reasoned subset of rules, and traceability over completeness |
| Jurisdictions outside the City of Los Angeles in the first iteration | The brief limits the proof of concept to residential parcels in the city |
| Non-residential parcels | The brief limits scope to residential parcels |
| Permit submission or plan-check corrections | Different work at a different moment (Section 3) |
| Telling the architect what to design or approving a design | The architect makes the decision; the tool gives the constraints |
| Serving clients or sales staff directly | The brief requires a user who can validate the claims |

---

## 8. Constraints we were given

| Constraint | What it is | Given by |
|---|---|---|
| Timeline | 1 week for the proof of concept. Date, time and time zone not stated *(to ask)*. What to drop first if half done: *(to ask)* | The brief |
| Team | *(to ask: Michael, alone?)* | |
| Services / vendors | Python backend, JavaScript/TypeScript frontend (required); AWS preferred, GCP may be needed for geocoding; everything else is up to the builder. The PRD's "inferred" items (React, FastAPI, LangChain, OpenAI) are not mandates | The brief, the PRD |
| Geography | Residential parcels in the City of Los Angeles | The brief |
| Data | Public parcel, zoning and regulatory information, "or reasonable mock data if needed"; more specific resources "will be provided before the project starts" *(not yet received)* | The brief |
| Deliverables | Source code, technical documentation, demo video, test results; an architecture diagram showing how it could be productionized; tested on a range of provided parcels and building types | The brief, the PRD |
| Intended use | Proof of concept for Cover; commercial use later is possible but *(to ask)* | |

Spend, hardware and regulatory/contractual rows are deleted because they were not asked.

---

## 9. Evidence limitations

- The problem has no quantified evidence. Cover supplied no hours, frequency or cost, and I found no public study of architects' zoning research time.
- The City's pages [1][2] establish that rules are split across sources and overlays; they do not say how hard that is for an architect who knows them.
- The LADBS correction sheets [4] show what plan check examines, not how often zoning mistakes occur.
- The ADU statutory timelines [3] come from a third-party site and should be checked against the statute.
- The consultancy breakdown [5] is not used because its data cannot be verified.
- Nothing here measures whether the engine improves outcomes for Cover.

---

## References

1. City of Los Angeles, Department of City Planning. Zoning Search / Zoning Code. https://planning.lacity.org/zoning/zoning-search and https://planning.lacity.org/zoning/zoning-code
2. City of Los Angeles, Department of City Planning. ZIMAS guidance pages. https://planning.lacity.gov/zoning/zoning-search
3. Cali ADU. ADU permitting process in California (citing SB 543, Gov. Code §66317). https://cali-adu.com/blog/adu-permitting-process-california/ (secondary source)
4. LADBS. Single family dwelling / duplex plan review correction list and ADU correction sheet. https://dbs.lacity.gov/sites/default/files/efs/forms/pc17/adu-correction-sheet.pdf
5. JDJ Consulting. How to Read LADBS Correction Notices. https://jdj-consulting.com/?p=10727 (private, unverified; not relied on)
