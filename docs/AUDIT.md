# AUDIT.md — Home Building Regulatory Engine

**Status:** first pass, run 2026-10-17. It tells us what sources exist and what the engine is expected to pull in. It is not legal advice and it is not the rule set.

**How it was done.** Five small research agents did a first scan (zoning code, ADU law, overlays, datasets, geocoding). Three stronger agents then re-did the parts that mattered by downloading the documents and querying the data services directly. Where a fact below says **read** it was seen in a downloaded document or live data response. **Mirror** means it was read on a third-party copy of the code. **Unverified** means it comes from a search snippet or memory and must not be built on yet.

## 1. Bottom line

1. **The data is all public and free to query.** Parcels, zoning, overlays, addresses and building outlines are served by City and County map services. Most need no key and answer by APN or by point. None carries an open-data licence (§3, §9).
2. **The controlling code text is not machine-readable by us.** The American Legal code library (which publishes the LAMC), Municode and the state legislature site all refuse automated access. The City's own planning PDFs and ordinance PDFs on the City Clerk site do work. So citations in v1 point at official City PDFs where possible, and at a mirror otherwise, and the engine says which (§8).
3. **Two zoning codes are in force.** Most house lots are under the old Chapter 1 (R1, RS, RE, RD, R2). Downtown, and some other areas, are under the new Chapter 1A, with different zone strings. The engine needs a first step that decides which code a parcel is under (§7).
4. **For house lots, most of the base rules are numeric.** Height, yards, lot area and floor-area ratio are tables. The prose-heavy parts are the front-yard "prevailing setback" rule, exemptions to floor area, ADU height and parking tests that depend on transit distance, state-versus-City layering, and anything an overlay changes. That supports the plan: a small table-driven core with interpretation reserved for those places (A-5).
5. **The base floor-area question is settled for R1.** The current text gives a flat 0.45 outside hillside and coastal areas. The 0.50 tier was the 2008 version (§4).
6. **A "guest house" is probably handled as an ADU.** The 2019 ordinance repealed the old second-dwelling rules and the City's memo treats those units as ADUs. This is not confirmed from current code text (§5).
7. **A geocoding vendor is probably not needed.** The City publishes about 1.05M address points under CC0, and the parcel layers answer by point. GCP stays optional (§6).

## 2. Scope recommendation for the proof of concept

Under A-8 the rule set is: building type permitted, zone, height, setbacks, lot coverage, floor area, ADU size, overlay flags. The audit suggests narrowing the **parcels** the engine fully answers, and flagging the rest honestly:

| Parcel type | Treatment in v1 |
|---|---|
| Chapter 1 zones RE, RS, R1 (and R2) outside Hillside and Coastal areas | Full answer |
| Same zones inside a Hillside Ordinance area | Answer with the slope-band floor-area table, using slope estimated from public elevation data and labelled "estimate, not a stamped Slope Analysis Map" (§3a) |
| Parcel with a Q, T or D marker in its zone string | Detect it, show the ordinance number and effective date, link the ordinance PDF, and list the actual caps as not determined unless that ordinance has been entered by hand (§3a) |
| Parcel with an HPOZ, specific plan, CPIO or other overlay | Show the overlay flag as a constraint, state that its rules override the base zone, and list the overlay's own limits as not determined unless the plan is ingested |
| Parcel inside the Coastal Zone | Detect it from the Coastal Commission layer, state that a Coastal Development Permit applies, show the height and floor-area figures from the City summary, and list who issues the permit and the Venice sub-area as not determined (§3a) |
| Chapter 1A zone strings (bracketed) | Return "unsupported in v1" with the reason |
| RD and other multi-family zones | Return "unsupported in v1" (GC-6 already excludes non-residential parcels; this is a further cut and needs the user's agreement) |

## 3. Data to pull in

All endpoints below were queried live on 2026-10-17 with no key. Counts are what the service returned.

| Need | Source | What it gives | Access | Status |
|---|---|---|---|---|
| Parcel outline by APN or point (City) | City parcel layer, `maps.lacity.org/lahub/rest/services/Landbase_Information/MapServer/5` | 883,687 parcels; APN in field `BPP`; address text in `TOOLTIP`; no lot-area field (compute from geometry) | Query by `where` and by point | read; item last modified 2020-09-17 |
| Parcel attributes (County) | County Assessor parcels, `public.gis.lacounty.gov/.../LACounty_Parcel/MapServer/0` | 2,433,269 parcels; `AIN`/`APN`, situs address, use code, year built, bedrooms, building area, lot area | Query by `where` and by point; 1,000 rows per request. Monthly download of the full set exists | read |
| Zoning | City zoning polygons, `services5.arcgis.com/7nsPwEMP38bSkCjy/arcgis/rest/services/Zoning/FeatureServer/15` | 58,859 polygons, edited 2026-10-15; fields `Zoning`, `CATEGORY` only. Height district and overlay are suffixes inside the string (`R1V2-HPOZ`) | Query by point | read |
| Zoning, richer | `maps.lacity.org/lahub/rest/services/City_Planning_Department/MapServer/8` | `ZONE_CMPLT`, `ZONE_CLASS` (e.g. R1), `ZONING_DESCRIPTION`; marks Chapter 1A zones | Query | read |
| Address to point | City address points, `data.lacity.org/resource/4ca8-mxuh.json` | 1,045,947 rows, daily refresh, fields `hse_nbr`, `str_nm`, `str_sfx_cd`, `zip_cd`, `lat`, `lon` | Socrata query; no key | read; **CC0** |
| Existing buildings | LARIAC 2020 building outlines (County), `services.arcgis.com/RmCCgQtiZLDCtblq/.../Countywide_Building_Outlines_(2020)/FeatureServer/0` | 3,293,177 outlines with `HEIGHT`, `AREA`; no address or APN, so join by location | Query and bulk download | read; licence points to a County terms page that was not read |
| Overlay flags | HPOZ (36), specific plan areas (61), community design overlay (19), CPIO (9), Hillside Ordinance area (344), hillside grading area (7), Very High Fire Hazard zones (14), Residential Floor Area districts, Neighborhood Stabilization overlay | Mostly name-only flags | Query by point | read (service URLs in the working notes, to be copied into `ARCHITECTURE.md`) |
| Coastal Zone | California Coastal Commission, `services9.arcgis.com/wwVnNW92ZHUIr0V0/arcgis/rest/services/VIEW_Local_Coastal_Program_Segments/FeatureServer/0` | 370 polygons, edited 2026-09-04; the segment name for each City of LA coastal area | Query by point | read (§3a); the Commission's licence requires an "illustrative purposes only" statement on derived products |
| Q, T and D conditions | the zone string itself, plus City "Zoning Archive" layer (`services5.../Zoning_Archive/FeatureServer/2`) | markers on every polygon; ordinance number and effective date on most Q polygons | Query by point | read (§3a); the text of each condition is in an ordinance PDF, not in data |
| Elevation (for slope) | USGS 3D Elevation Program, `elevation.nationalmap.gov/arcgis/rest/services/3DEPElevation/ImageServer` | 1 m ground elevation, public domain | Point query and tile export | read (§3a) |
| Permit history | LADBS permit datasets on `data.lacity.org` | useful later; not needed in v1 | | unverified |

**ZIMAS.** The City's parcel lookup tool appears to sit on these same public services. No page we read says automated use is allowed, and the terms sit behind a script. Treat it as a manual cross-check only and ask City Planning before any automated use.

**Dataset size.** About 0.9M City parcels (the earlier design assumed about 1M parcels and buildings; the real count is 0.9M parcels plus about 1.1M to 3.3M outlines depending on source). Still small for one database.

## 3a. The three data gaps, followed up

Three agents re-ran the gaps found in the first pass. Everything below was queried or downloaded on 2026-10-17.

**Slope (for the Hillside floor-area table).**
- **What the City accepts.** From the 2010 draft of the Hillside ordinance (the adopted wording was not read): a "Slope Analysis Map" signed by a licensed surveyor, with contours at 10 ft or finer. So the engine's number can only be a pre-screen, never the official one.
- **Source that works.** The USGS 3D Elevation Program serves 1 m ground elevation without a key, and its "terms" are the USGS statement that its data is public domain. No City elevation or contour layer exists. The County contour layer returned unclipped lines and was not usable; the 2020 LiDAR is restricted to university users (search snippet, not opened).
- **Test.** For one Hillside-area parcel near Mulholland Drive the agent downloaded the tile, computed slope with numpy, and picked pixels inside the parcel with Shapely. Mean slope was 46.8% against 46.1% from the service's own slope output. The six bands agreed closely except the 100%-and-over band (518 vs 302 sq ft). It was not compared with a surveyor's map. Expected accuracy ("several percentage points per band") is an estimate, not measured.
- **Gotchas.** A point query must pass the spatial reference or it returns no data. The vertical datum is not stated in the service. Tiles from before 2024 are coarser.

**Q, T and D conditions.**
- **Detection.** Every zone string shows them: `(Q)` or `[Q]` prefix, `(T)` prefix, and D inside the height district (for example `[Q]R1-1D-HCR`). In the City layer, 4.8% of R1, RS and RE polygons carry one (about 3% for Q alone), 28.7% of RD polygons and 3.0% of R2. These counts are by polygon, not by parcel, and come from the agent's own parsing. ArcGIS reads `[Q]` as a wildcard, so the string must be parsed in code.
- **Ordinance link.** The City "Zoning Archive" layer has an ordinance number and effective date for most Q polygons (431 of 819 marked R1/RS/RE polygons in one field, 382 in the other, 18 Q-marked R1 polygons with neither). What the two field names mean is a guess.
- **Text.** One real ordinance was read (Northeast LA Hillsides, Ord. 180,403): height 30 ft, or 26 ft if the roof slope is under 25%; a slope-band floor-area formula with a 20%-of-lot or 1,000 sq ft floor; prose design standards; Zoning Administrator adjustments of up to 20% on height and 10% on floor area; and earlier conditions retained from older ordinances, so one parcel can be under several. A prose-only Q ordinance was not sampled.
- **Engine behaviour.** Detect the marker, show the ordinance number and date, link the PDF, and list the actual limits as not determined unless that ordinance has been entered by hand. The agent estimates a day or two to enter the most common ordinances for the test parcels.

**Coastal Zone.**
- **Source.** No City layer exists. The California Coastal Commission publishes one that answers by point: coastal zone boundary and "local coastal programme segments" (370, edited 2026-09-04; 14 are City of LA segments such as Venice, Pacific Palisades and San Pedro). The City's specific-plan layer also flags the Venice Coastal Zone and the Coastal Transportation Corridor. Test points in Venice and Pacific Palisades hit; Van Nuys and Encino did not. The agent used coordinates, not geocoded addresses.
- **Terms.** The Commission's boundary layer says it is not the official boundary and requires derived products to say so ("shown for illustrative purposes only"). The answer must carry that statement.
- **Rules read.** In the City summary: height 45 ft for lots in the Coastal Zone outside Hillside areas; floor area 3:1 in Height District 1; one extra foot of side yard per story above the second. Venice's own specific plan sets height by sub-area (for example 30 ft flat roof / 35 ft varied in North Venice; 28 ft on walk streets) in prose, and the sub-area boundaries are exhibit images.
- **Not available:** no layer for who issues the permit (City, Commission or both); the Venice sub-area; the base floor-area number for R1, RS and RE in the Coastal Zone (it cites LAMC 12.21.1 A, unread); the Mello Act statute text; LAMC 12.20.2. The code may have moved sections under Chapter 1A.
- **Engine behaviour.** Detect the coastal segment, state that a Coastal Development Permit applies, show the figures above with their sources, print the Commission's statement, and list who issues the permit, the Venice sub-area and the coastal floor-area number as not determined.

## 4. Base rules for house lots (Chapter 1)

Source for yards, lot area and height: the City's own January 2026 "Generalized Summary of Zoning Regulations" (an official document that calls itself "only a guide"), checked against the text of the code on a third-party mirror (gocodebook.com, "2026-08 edition"). Where the two agreed the row is marked read; section numbers come from the mirror.

| Rule | R1 (LAMC 12.08) | RS (12.07.1) | RE (12.07.01) | Type |
|---|---|---|---|---|
| Uses | one-family dwelling; "accessory living quarters" only on lots of 10,000 sq ft or more | same, 15,000 sq ft | same, 20,000 sq ft | numeric condition |
| Height, flatland, Height District 1 | 33 ft, or 28 ft if the top roof slope is under 25% | same | RE9 same; RE11–RE40 36 ft or 30 ft | numeric table; coastal and hillside differ |
| Front yard | 20% of lot depth, max 20 ft | 20% of depth, max 25 ft | 20% of depth, max 25 ft | numeric, but the "prevailing setback" averaging rule needs interpretation |
| Side yard | 5 ft (10% of width if lot under 50 ft wide, min 3 ft); +1 ft per 10 ft of height above 18 ft | 5 ft, same reduction | 7 ft (RE9, RE11) to 10 ft (RE20, RE40) | numeric with conditions |
| Rear yard | 15 ft | 20 ft | 25% of depth, max 25 ft | numeric |
| Min lot area / width | 5,000 sq ft / 50 ft | 7,500 / 60 | 9,000 to 40,000 | numeric |
| Residential floor area | **0.45 × lot area** (flatland), all lot sizes | 45% under 9,000 sq ft; 40% or 4,050 sq ft above; +20% bonus for upper-floor step-back | 35–40% depending on zone and lot size; same bonus | numeric; exemptions and bonus are prose |
| Lot coverage | only in R1 variation zones: 50% down to 40% | none | none | numeric |

**Floor area for R1, resolved.** The current text (Ord. 184,802, effective 3/17/2017 per the mirror) gives a flat 0.45. The original 2008 ordinance (179,883) said 0.50 with a 45% or 3,750 sq ft rule for lots of 7,500 sq ft and up, which a City zoning-information file confirms. An unsigned redline on the planning site should not be used. The effective date comes from the mirror, not a signed copy.

**Hillside.** Floor area is the sum of each slope band's area times a ratio (R1: 0.45 at 0–15% slope falling to 0 at 100% and over). Height is an envelope measured from "Hillside Area Grade". These need slope data, which none of the layers above provide.

**Floor-area exemptions** (prose, from the code's definition of residential floor area): covered parking up to 400 sq ft if in the rear half or 55 ft from the front; detached accessory buildings of 200 sq ft or less (cap 400); porches and patios; some basements; ceilings over 14 ft count twice.

## 5. ADUs and guest houses

Sources read: the 2019 City ordinance (186,481, effective 12/19/2019, the only text of LAMC 12.22 A.33 we could read); the City's implementation memo "ZA Memo 143, Rev. 1" (August 2025); the state housing department's handbook (March 2026, covers SB 543); and state Government Code text from a secondary site (`california.public.law`, marked current to 2026-01-01). The current amended A.33 was not reached.

| Rule | Value | Note |
|---|---|---|
| Detached ADU size | 1,200 sq ft | read, ordinance (d)(1) |
| Attached ADU size | 50% of the existing dwelling, but at least 850 sq ft (1,000 sq ft if more than one bedroom) | read; the floor comes from the City memo |
| Lot floor-area and coverage limits | apply, but may not prevent an 800 sq ft ADU with 4 ft side and rear setbacks | read, state law and ordinance |
| Side and rear setback | 4 ft for new construction; none for a conversion | read |
| Front | detached ADU may not sit between the house and the street | read; exceptions exist |
| Height | the City sets no ADU figure; state floors are 16 ft, 18 ft near a major transit stop, +2 ft if the roof pitch matches the house; max two stories | read (state text, City memo). Needs a "major transit stop" test |
| ADUs per lot | one under the City ordinance; state types can add an attached ADU and a junior ADU | read |
| Junior ADU | 500 sq ft, inside the house | read |
| Parking | one space unless exempt; exempt within half a mile walking distance of transit, in an HPOZ, and other cases | read; walking distance is measured on streets "by generally available mapping systems" |
| Owner-occupancy | none for ADUs | read, memo |
| No ADU where the lot is in both a Very High Fire zone and a Hillside area | with exceptions; not enforceable on state-type ADUs | read |

**State versus City.** The memo itself overrides parts of the 2019 ordinance to match state law. An earlier scout's claim of a state-housing-department deficiency letter to Los Angeles was not found and is dropped.

**Guest house.** The word does not appear in the 2019 ordinance. The old second-dwelling provisions (12.24 W.43, W.44) were repealed and the memo says those units "shall be treated as ADUs". The base-zone text uses "accessory living quarters". Whether a separate guest-house rule remains is unconfirmed because LAMC 12.03 and 12.21 C.5 could not be read from an official source. **The engine should handle "guest house" as an ADU or accessory living quarters and say so**; the user should confirm this reading (it changes FR-1.1).

## 6. Geocoding and parcel lookup

- **Address to parcel, in-city:** look the address up in the City address points (CC0), take its point, then ask the parcel layer which parcel contains it. No vendor needed.
- **APN lookup:** `where BPP = '<10 digits>'` on the City layer, or `AIN` on the County layer. LA County APNs are 10 digits shown as 4-3-3 (e.g. 5123-014-022). One third-party page says 12 digits; treat that as an error.
- **Google geocoding:** a free monthly allowance exists (10,000 calls per product, from the vendor's pricing page) but the per-thousand price and the rule on storing results were not found. Use it only as a fallback, after reading those terms.
- **Free alternatives:** US Census geocoder (batch limit 10,000 per file, returns census geography); public Nominatim (1 request per second, results must be cached); self-hosted Pelias (MIT). None is better than the City points for in-city addresses.
- **Failure cases** (engineering judgement, not sourced): unit numbers, several parcels per address, parcels with no street address, street direction and suffix variants. The existing "several parcels" and "no match" states already cover them.

## 7. Chapter 1 and Chapter 1A

Both are in force (read, City Planning documents). Chapter 1A, the new code, "applies only to … lots zoned with districts established in Chapter 1A"; other lots stay under Chapter 1. The two City pages disagree about which areas are already converted (Downtown only, or Downtown plus Boyle Heights), so the status must be checked per parcel. In the zoning layer about 835 of 2,158 distinct zone strings are bracketed Chapter 1A codes (for example `[RG2-2L]`), and every bracketed residential one seen also carried a CPIO overlay suffix. The 1A residential districts use form, frontage and use parts with no RE/RS/R1 equivalent. **Consequence:** the engine needs a zone parser for legacy strings and a separate one for bracketed strings, and v1 supports only the first.

## 8. Citations, and the code text we cannot fetch

A citation must point to a passage the user can open (VR-1). Today the usable sources are:
- City Planning PDFs (zoning summary, memos), fetched by direct download.
- Ordinance PDFs on the City Clerk site (the 2019 ADU ordinance, others).
- A third-party mirror of the LAMC text, which says it is "not a substitute" for official publications.

The official LAMC publisher blocks automated access, and its bulk-use terms were not read. So v1 either stores the City PDFs and cites them, or cites the mirror and labels the source as unofficial in the answer. Either way the answer should say which. The user should decide whether a mirror is acceptable for a demo.

## 9. Licences and terms

| Source | Terms found | Consequence |
|---|---|---|
| City address points | CC0 (read) | free to copy |
| City Planning layers (zoning, HPOZ, specific plans) | standard web-site agreement: "as is", no accuracy guarantee, can change; no open licence; copyright City Planning | fine for an internal proof of concept; ask City Planning before redistributing a copy |
| City parcel layer | licence field empty | same |
| County parcels and building outlines | informational use; subject to the County's terms-of-use page, which was not read | read it before the footprint data is stored |
| Planning site and PDFs | terms say content is City Planning's property and not to be reposted without rights | keep stored copies internal (A-11) |
| Code text | unverified | public law, but the publisher's terms were not read |

## 10. Open items (not resolved by this audit)

1. The controlling LAMC text for 12.08, 12.07, 12.21 C.10 and 12.22 A.33 from an official source, and the signed Ord. 184,802.
2. Whether any ordinance after 2019 amended A.33 (none seen).
3. Whether a separate guest-house rule exists in current code.
4. Whether the City accepts an estimated slope as a screening figure; the adopted ordinance text on slope bands; the vertical datum of the elevation data (§3a).
5. What the two ordinance-number fields in the Zoning Archive layer mean, and prose-only Q ordinances (§3a).
6. Who issues a Coastal Development Permit for a given lot, the Venice sub-area, LAMC 12.20.2 and 12.21.1 A (§3a).
7. The County terms of use (parcels and building outlines).
8. Whether City Planning allows automated ZIMAS use; ask its data-requests contact.
9. Google geocoding price and storage terms (only if used).
10. Exact RE side-yard values and the R1 encroachment-plane geometry (garbled in extraction).

## 11. What this changes in the other documents (to apply once agreed)

| Document | Change |
|---|---|
| `USERS.md` A-5 | Now partly evidenced: base rules are mostly numeric tables; interpretation is needed for prevailing setbacks, floor-area exemptions, ADU height and transit tests, overlay precedence. |
| `USERS.md` A-8, `REQUIREMENTS.md` | Narrow the supported parcels (§2). Add a requirement that a parcel under Chapter 1A, a multi-family zone or a Hillside area without slope data returns a named limitation. Treat "guest house" as an ADU or accessory living quarters (§5). |
| `REQUIREMENTS.md` DE-, GC-9 | Record that code text comes from City PDFs or a mirror and that answers must state which. Record licence entries from §9. |
| `HIGH_LEVEL_DESIGN.md` | Real source sizes (§3). The geocoder is no longer needed; address resolution uses the City points and the parcel layers, so GC-5's GCP clause is not exercised. Add a "which code applies" step before rule evaluation. Parcel count is 0.9M. |
| `REQUIREMENTS.md` open items 1–3 | Item 1 (datasets) and item 3 (ZIMAS) are answered in §3; item 2 (numeric versus prose) in §1. |

## 12. Documents and services read

- City Planning, Generalized Summary of Zoning Regulations (CP-7150, January 2026): `planning.lacity.gov/odocument/eadcb225-a16b-4ce6-bc94-c915408c2b04/Zoning_Code_Summary.pdf`
- City Planning, Zoning Code Chapter 1A book (January 2025): `planning.lacity.gov/odocument/35d8aeca-6be9-463b-b6cd-674049f618d0/AA_Chapter1A_Book_v2.pdf`
- ZIMAS zoning information file ZI 2391: `zimas.lacity.org/zoneinfo/ZI2391.pdf`
- Ordinance 186,481 (ADU, 2019): `cityclerk.lacity.org/onlinedocs/2016/16-1468_ORD_186481_12-19-2019.pdf`
- ZA Memo 143 Rev. 1 (August 2025): `planning.lacity.gov/odocument/be4db130-aa38-4637-b35a-d24304d50ff2/ZA_Memo_143_Revision_1_Chapter1.pdf`
- State housing department ADU Handbook (March 2026): `hcd.ca.gov/sites/default/files/docs/policy-and-research/adu-handbook-update.pdf`
- State Government Code 66313–66333 on a secondary site: `california.public.law/codes/government_code_section_66323` (same pattern for other sections)
- LAMC Chapter 1 text on a third-party mirror: `gocodebook.com/library/us/ca/los-angeles-planning/chapter-i-general-provisions-and-zoning/article-2-specific-planning-zoning-comprehensive-zoning-plan/`
- LA Conservancy page on the mansionization ordinance: `laconservancy.org/save-places/issues/mansionization-and-teardowns-city-of-los-angeles-ordinance/`
- USGS 3D Elevation Program image service: `elevation.nationalmap.gov/arcgis/rest/services/3DEPElevation/ImageServer`; City Planning 2010 draft Hillside ordinance: `planning.lacity.gov/Code_Studies/BaselineHillsideOrd/ProposedBHOrdProvisions.pdf`
- Ordinance 180,403 (Northeast LA Hillsides, Q and D conditions): `cityclerk.lacity.org/onlinedocs/2008/08-2599_ord_180403.pdf`; City "Zoning Archive" layer on `services5.arcgis.com/7nsPwEMP38bSkCjy`
- California Coastal Commission layers on `services9.arcgis.com/wwVnNW92ZHUIr0V0`; Venice Coastal Zone Specific Plan: `planning.lacity.gov/odocument/b76ef566-6a24-4603-a5cc-e040af8646c4/Venice_Coastal_Zone_Specific_Plan.pdf`
- Census geocoder, Nominatim policy and Pelias documentation (geocoding options), and Google Maps Platform pricing and usage pages (price per thousand not found).
