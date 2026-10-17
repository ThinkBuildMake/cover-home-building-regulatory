# KEY_METRICS.md — Home Building Regulatory Engine

**Status:** draft. The success sentence is agreed with the user.

## What success means

The user is a Cover design architect who today looks up a parcel in ZIMAS, reads the code and overlay documents, and writes the result up by hand (`USERS.md` §1, A-2). There is no baseline for how long or how error-prone this is; Cover supplied no figures (`USERS.md` §2, A-3).

Success means Cover's architects answer their parcel questions with the engine instead of researching them by hand.

## Success metric: Parcels assessed per week

**What is it?** The number of distinct parcels that Cover architects run an assessment on each week. A parcel counts once per week, however many times the inputs are changed on it.

**How we measure it.** Count of distinct parcel identifiers with at least one completed assessment in the week, from the answer records (OB-1). No manual counts.

**Target.** No baseline exists for how many parcels Cover researches by hand (`USERS.md` §2, A-3), so the target is set after 4 weeks of data.

**Why it shows the need was met.** Each parcel assessed is a parcel the architect did not research by hand, which is the problem in `USERS.md` §2. It counts the work the engine took over, not clicks on a feature.

**What it can't tell you.** It counts use, not correctness: a wrong tool would still be used for a while. The first supporting metric below covers that gap.

### Supporting metrics

| Metric | What it is | Recorded from | Why it explains the success metric |
|---|---|---|---|
| **Assessments with a constraint marked wrong** | Share of reviewed assessments where she marked at least one constraint "wrong" (FR-2.10) | Confirmed/wrong marks (OB-4) | If parcels per week falls and this rose, architects stopped trusting the answers. |
| **Assessments with a citation opened** | Share of assessments where she opens at least one citation | Citation-open events (OB-4) | If parcels per week is steady but this is low, nobody is checking the answers. |
| **Assessments followed by a changed input** | Share of assessments where she changes a project input on the same parcel (UC-2) | Input-change events (OB-4) | If it is low, the re-answer half of the product is not being used (A-6). |

## Guarantee metrics

| Metric | Requirement | Threshold | Measured on | Recorded from |
|---|---|---|---|---|
| **Answer time** | PF-1, PF-2 | p95 ≤ 10 s for a first answer and for a re-answer | Slowest realistic case: a parcel with an overlay and an ADU, with several users at once; weekly | Per-request stage timings (OB-2) |
| **Longest wait before an answer or error** | RL-2 | Every request ends within 30 s | Maximum over all requests; weekly | Per-request stage timings (OB-2) |
| **Geocoder failure report time** | DE-1 | User told within 5 s of a geocoder failure | p95 of failed geocoding requests; weekly | Request log |
| **Cost per assessment** | CO-1 | Median ≤ $0.25 (interim, not yet agreed; see below) | Median over all requests; weekly | Per-request cost record (OB-3) |
| **High-confidence constraints marked wrong** | VR-6 | At most 10% marked wrong (interim 90% correct; not yet agreed) | Constraints shown as high confidence; weekly | The "confirmed / wrong" marks |

## Upstream fixes and open questions

- Done in `REQUIREMENTS.md`: the confirmed/wrong mark (FR-2.10), event recording (OB-4) and a snapshot-age flag (DE-6).
- DE-6 flags old data but is not tracked as a metric. A limit on how stale data may be is still open.
- CO-1 and VR-6 thresholds are still [tbd]; they are set after the first full eval run. Their rows above use the interim values.
- The success target waits on 4 weeks of data.
- FR-2.10 and OB-4 stay in `REQUIREMENTS.md`: the wrong-mark share and the VR-6 guarantee use them.
