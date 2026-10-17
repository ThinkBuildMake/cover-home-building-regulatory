# KEY_METRICS.md — Home Building Regulatory Engine

**Status:** draft. The success sentence is agreed with the user.

## What success means

The user is a Cover design architect who today looks up a parcel in ZIMAS, reads the code and overlay documents, and writes the result up by hand (`USERS.md` §1, A-2). There is no baseline for how long or how error-prone this is; Cover supplied no figures (`USERS.md` §2, A-3).

Success means the architect starts from the engine's cited answer and checks it, instead of redoing the research by hand, because she has found its claims to be right.

## Success metric: Assessments the architect did not mark wrong

**What is it?** The share of reviewed assessments in which no constraint was marked wrong. A reviewed assessment is one where the architect marked at least one constraint "confirmed" or "wrong".

**How we measure it.** From the per-constraint "confirmed / wrong" marks the product records. Formula: reviewed assessments with 0 constraints marked wrong ÷ all reviewed assessments, per week. This needs a user action the requirements do not have yet (see upstream fixes).

**Target.** No baseline exists, so the target is set after 4 weeks of use. Until then the number is only watched.

**Why it shows the need was met.** An architect who marks constraints "confirmed" is relying on the answer rather than redoing the work. One who keeps marking them "wrong" is not. It measures whether the answers deserve trust, not whether the page was opened.

**What it can't tell you.** It counts only what the architect chose to mark, and she may not catch a wrong constraint; an unmarked error looks like success.

### Supporting metrics

| Metric | What it is | Recorded from | Why it explains the success metric |
|---|---|---|---|
| **Assessments with a citation opened** | Share of assessments where she opens at least one citation | Citation-open events | If the success metric looks good but this is low, nobody is checking and the number is not trustworthy. |
| **Assessments followed by a changed input** | Share of assessments where she changes a project input on the same parcel (UC-2) | Input-change events | If it is low, UC-2 is not being used, and the "re-answer" half of the product is not serving the need (A-6). |

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
