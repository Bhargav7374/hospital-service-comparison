# Product Requirements Document (PRD)
## Hospital Service Comparison

**Document status:** Initial planning baseline  
**Version:** 1.0  
**Last updated:** 2026-10-17

## 1. Purpose

Hospital Service Comparison helps patients and their families discover hospitals, compare diagnostic-service prices and other practical details, and request appointments. Hospital administrators can maintain their hospital profile, service prices, availability, and booking information.

The initial release uses clearly labelled sample/demo data. The product can transition to verified real-world data only after data sources, permissions, update frequency, and verification procedures are established.

## 2. Confirmed decisions

The following decisions were explicitly selected by the project owner:

- **Target users:** Patients/families and hospital administrators.
- **Data strategy:** Start with sample data, then move to verified real data.
- **Scope:** Advanced resume project including maps, ratings, booking, administrator features, and ML prediction.
- **Documentation:** Maintain PRD, TRD, UI/UX design brief, app flow, and AI context (`brain.md`).

## 3. Assumptions requiring validation

These are planning assumptions, not confirmed facts:

- The first demonstration will focus on Ahmedabad/Gujarat, with location scope configurable.
- The initial interface will be English-first and responsive on desktop and mobile.
- Patients may browse without creating an account; an account may be required to manage bookings.
- Appointment requests will be treated as **requests**, not confirmed bookings, until a hospital/admin confirms them.
- Initial sample prices, ratings, locations, and availability are synthetic and must be labelled as demo data.
- ML will initially provide a clearly labelled estimate or ranking from available data, not medical advice.
- Payment processing is out of scope for the first version unless separately approved.

## 4. Problem statement

Patients may need to contact several hospitals separately to learn whether a diagnostic service is offered, how much it costs, how far away the hospital is, and whether an appointment slot is available. The project aims to make this comparison easier through a single searchable interface.

Hospital administrators need a simple way to maintain service listings and respond to appointment requests.

## 5. Goals and non-goals

### Goals
1. Search hospitals by name, area, service, and distance.
2. Compare selected hospitals and diagnostic services side by side.
3. Show listed prices with source/status and last-updated information when available.
4. Display hospital locations on a map and provide distance/directions links.
5. Display ratings/reviews with clear provenance; do not fabricate real reviews.
6. Let patients submit and track appointment requests.
7. Give administrators tools to manage hospital profiles, services, prices, slots, and requests.
8. Provide an explainable ML-based estimate/ranking using documented data and limitations.
9. Provide a testable, deployable, portfolio-ready application.

### Non-goals for the initial release
- Diagnosis, treatment recommendations, or emergency triage.
- Claiming that displayed prices are guaranteed final bills.
- Automatically confirming real appointments without a reliable hospital integration or admin confirmation.
- Processing medical records or storing detailed clinical histories.
- Payments, insurance claims, or electronic health record integration.
- Claiming a hospital is accredited or a service is available without evidence.

## 6. Personas

### Patient / family member
Needs to find a nearby hospital, compare a service price and travel distance, review available information, and request an appointment without confusion.

### Hospital administrator
Needs to manage the hospital profile, services, indicative prices, available slots, and incoming appointment requests.

### Project maintainer
Needs auditable data, reliable tests, clear configuration, and documentation that helps future development.

## 7. MVP and advanced scope

### Phase 1 — Demonstration MVP
- Home/search page.
- Search by service, location/area, and hospital name.
- Hospital list and details.
- Side-by-side comparison for selected hospitals.
- Sample data clearly labelled as demo.
- Basic filters and sorting.
- Automated tests for core comparison logic.

### Phase 2 — Operational workflow
- Authentication and role-based access for administrators.
- Hospital profile and service/price management.
- Appointment request form and status tracking.
- Admin request review and status updates.
- Data validation and audit timestamps.

### Phase 3 — Advanced capabilities
- Map view and distance calculation.
- Ratings/reviews from a defined, trustworthy source or verified-user workflow.
- Availability/slot display with update timestamps.
- ML prediction/ranking with evaluation and explanation.
- Verified real-data onboarding and source-health monitoring.
- Deployment, monitoring, and accessibility review.

## 8. Functional requirements

| ID | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| FR-01 | Search hospitals | Must | Search by hospital name, location/area, and supported service; show an understandable empty state. |
| FR-02 | Filter and sort results | Must | Filter by service, price range, rating where sourced, and distance where coordinates exist. Sort options must be labelled. |
| FR-03 | View hospital details | Must | Detail page shows address, contact details if verified, services, listed prices, source/status, and last updated date when available. |
| FR-04 | Compare hospitals | Must | User can select at least two hospitals and compare the same service and price basis side by side. Missing values show “Not available,” not zero. |
| FR-05 | Price transparency | Must | Distinguish sample, indicative, and verified prices; show inclusions/exclusions and update date where known. |
| FR-06 | Map and distance | Should | Map pins are based on stored coordinates; distance method and units are clear; map failure does not block list search. |
| FR-07 | Ratings | Should | Display rating only with source, count, and/or verified-review status; otherwise show “No verified rating available.” |
| FR-08 | Appointment request | Should | Validate required fields, create a request with a reference ID, and show that it is pending until confirmed. |
| FR-09 | Admin management | Should | Authorized admin can create/update their hospital's profile, service prices, slots, and booking statuses. |
| FR-10 | ML estimate/ranking | Could/Advanced | Show the prediction type, inputs/features, evaluation status, and disclaimer; never present it as a guaranteed price or medical recommendation. |
| FR-11 | Data provenance | Must | Each real-data record has source, verification status, and last-checked timestamp when applicable. |
| FR-12 | Error handling | Must | API/network/database errors show useful messages without exposing secrets or stack traces. |

## 9. Core user journeys

1. Patient selects a diagnostic service and location.
2. System returns matching hospitals and displays price/data status.
3. Patient filters/sorts results and opens hospital details.
4. Patient selects hospitals for comparison.
5. Patient may submit an appointment request for a selected hospital and time.
6. Hospital administrator reviews the request and accepts, rejects, or proposes a different slot.
7. Patient sees the current request status.

## 10. Data and trust requirements

- Demo data must have a visible “Demo data” label.
- Do not scrape or republish data without checking terms, permissions, and applicable law.
- Never invent prices, ratings, hospital certifications, contact details, or appointment availability.
- Real records should include source, verification status, `last_verified_at`, and `updated_at`.
- Prices must specify currency (INR), service/test name, units or package basis, and known inclusions/exclusions.
- Where no trustworthy data exists, show “Not available” and explain how users can verify directly.
- The app is informational and does not replace contacting the hospital.

## 11. Non-functional requirements

- **Usability:** Clear search, comparison, and booking states; responsive layouts.
- **Performance:** Common searches should feel responsive; target under 2 seconds for typical local database queries in the demo environment (assumption to validate).
- **Reliability:** Graceful handling of unavailable map/API services.
- **Security:** Password hashing through a vetted auth library, role-based authorization, input validation, secrets in environment variables, and least-privilege database access.
- **Privacy:** Collect only information needed for appointment requests; define retention and deletion policy before production.
- **Accessibility:** Keyboard navigation, labels, adequate contrast, and non-colour-only status indicators.
- **Maintainability:** Modular services, typed/validated API schemas, automated tests, and documented setup.
- **Auditability:** Record administrative changes to prices, slots, and booking status.

## 12. Success metrics

Initial project metrics (targets are assumptions):
- At least 90% of core search/compare test cases pass.
- A user can reach a comparison in no more than 4 major interactions after landing.
- Every displayed price has an explicit demo/indicative/verified status.
- No unauthorized admin can modify another hospital's records in security tests.
- ML performance is reported against a baseline before the prediction is enabled by default.

## 13. Risks and mitigations

| Risk | Mitigation |
|---|---|
| Stale or incorrect prices | Source attribution, verification dates, expiry/review workflow, and direct confirmation warning. |
| Fake or misleading ratings | Do not fabricate; show source and count or hide the rating. |
| False availability | Use request status and explicit admin confirmation. |
| Poor ML quality or biased data | Start with baseline, evaluate, explain limitations, and allow disabling predictions. |
| API costs or rate limits | Keep integrations behind service adapters; configure quotas and graceful fallback. |
| Sensitive personal information | Minimize collection, secure access, and define retention before production. |

## 14. Open questions

- Which launch geography and languages are required?
- Which real-data sources will be authorized and reliable?
- Will patients need accounts, or will verified contact details be enough?
- Which map/geocoding provider will be used, subject to terms and quotas?
- What exactly should ML predict: price estimate, hospital ranking, demand, or slot availability?
- What is the data retention period for appointment requests?
- Is payment functionality ever required?

## 15. Definition of done

A feature is done when its requirements are implemented, validation and error states exist, tests pass, documentation is updated, secrets are not committed, and any sample/estimated data is clearly distinguished from verified real data.
