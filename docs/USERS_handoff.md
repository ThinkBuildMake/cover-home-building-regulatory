# Handoff: design and requirement content removed from USERS.md

These items came up while defining the user but describe how the product is built, not who it serves. Carry them into requirements, metrics and architecture. Each maps to a use case ID.

## Verification and honesty
- Every constraint must carry a citation to a source passage and a derivation (UC-1, UC-2).
- Deterministic parcel/zoning data must be kept distinct from LLM-interpreted conclusions, and every conclusion must have a confidence signal (UC-1, UC-2).
- The tool must report what it could not determine, not omit it (UC-1).

## Capabilities traced to use cases
| Capability | Required by |
|---|---|
| Geocode an address or APN to a parcel | UC-1 |
| Ingest municipal zoning text into structured, citable rule fragments | UC-1, UC-2 |
| Parcel and building geometry and visualization | UC-1 |
| Recompute on changed project inputs | UC-2 |
| Feedback on bad responses (bonus) | UC-1 |
| Chat follow-up, setback geometry, admin pipeline view (bonus) | UC-2 |

## Time budget
- The brief says "real-time" and gives no number; the target is set in requirements after the meeting (UC-1, UC-2).

## Scope boundaries with their design reasons
- Subset of rules over full coverage: the brief prioritizes clarity and traceability over completeness; the rule model must scale to LA County, other California jurisdictions and beyond (UC-1).
- Mock data allowed "if needed": the architecture must make it clear which data is real and which is mock.

## Open questions for the audit / research step (assumptions A-1 to A-11 are in USERS.md Section 10)
- Which public datasets exist for LA City parcels, zoning and existing buildings, and how current are they? (the extra resources promised in the brief have not arrived)
- How much of the relevant regulation is numeric tables versus prose? This decides how much the LLM synthesis is needed (UC-1).
- Whether ZIMAS or other city sources can be queried programmatically.

## Existing products in the same space
- Not researched. ZIMAS is the city's own parcel lookup and the obvious alternative. Third-party ADU feasibility sites exist (for example propertyzoned.com); check their pricing and what they cover before claiming a gap.
