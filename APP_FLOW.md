# App Flow
## Hospital Service Comparison

**Document status:** Proposed interaction specification  
**Version:** 1.0  
**Last updated:** 2026-10-17

## 1. Confirmed scope

- Patient/family and hospital-admin user types.
- Demo data first, then verified real data.
- Search, comparison, maps, ratings, appointment requests, admin features, and ML capability.

Specific routes, authentication rules, and the exact ML target are proposed until implemented and validated.

## 2. Top-level navigation

```text
Landing / Search
  ├── Search results
  │    ├── Hospital details
  │    │    ├── Compare selection
  │    │    └── Appointment request
  │    └── Compare hospitals
  ├── Appointment request status
  ├── Sign in / Register (if enabled)
  └── Admin sign in
       └── Admin dashboard
            ├── Hospital profile
            ├── Services & prices
            ├── Availability
            ├── Appointment requests
            └── Audit history
```

## 3. Patient journey

### Flow P1 — Search for a service
1. User opens the home page.
2. User selects or types a service (e.g. X-ray, blood test, MRI).
3. User enters a city/area or uses a location feature if available and permission is granted.
4. User clicks **Search hospitals**.
5. UI shows a loading state.
6. API validates the query and returns matching hospitals.
7. Results display price status, location, and available sourced rating/distance fields.
8. If no matches exist, show an empty state with options to change service or broaden area.
9. If the API fails, show an error and retry action; do not show fabricated results.

### Flow P2 — Filter and sort
1. User selects filters.
2. Results update.
3. Sorting by price excludes or separates missing/uncomparable prices.
4. Distance sorting is available only when coordinates exist.
5. Rating sorting is available only when ratings have a source and value.
6. Applied filters are visible and can be cleared.

### Flow P3 — View hospital details
1. User clicks **View details**.
2. Detail screen shows profile, services, price basis, source/status, and last verification date if present.
3. Map is shown only if provider and coordinates are available.
4. If map fails, retain address and list content.
5. User can return to results without losing search/filter context where possible.

### Flow P4 — Compare hospitals
1. User clicks **Compare** on a result.
2. Hospital is added to comparison selection.
3. UI indicates selected count and prevents duplicate selection.
4. User opens **Compare selected**.
5. Comparison screen shows comparable service and price fields side by side.
6. Missing values show **Not available**.
7. Incompatible package/test bases show a warning and are not presented as directly equivalent.
8. User may remove hospitals or return to results.

### Flow P5 — Request an appointment
1. User clicks **Request appointment**.
2. Form is prefilled with hospital/service when known.
3. User enters required contact details and preferred date/time.
4. Client and server validate input.
5. On validation error, show field-specific messages; do not submit.
6. On success, create a request with status `pending`.
7. Show a request reference and the message: “Request submitted; this is not confirmed until the hospital accepts it.”
8. Notify hospital admin through the chosen mechanism, if configured.
9. If submission fails, retain form values safely and offer retry without duplicating requests.

### Flow P6 — Track request
1. User signs in or enters a secure request reference, depending on chosen access model.
2. System verifies ownership/access.
3. Display status timeline.
4. If status is `reschedule_proposed`, show the proposed time and response options if implemented.
5. Unauthorized or unknown references show a generic message without exposing private information.

## 4. Hospital administrator journey

### Flow A1 — Authentication
1. Admin opens admin sign-in.
2. Server validates credentials.
3. Server loads assigned hospital permissions.
4. Unauthorized users cannot open admin pages or call admin APIs.
5. Session expiry redirects to sign-in with a clear message.

### Flow A2 — Manage profile
1. Admin opens **Hospital profile**.
2. Existing profile values are loaded.
3. Admin edits permitted fields.
4. Server validates input and authorization.
5. Save succeeds and creates an audit entry.
6. UI displays updated timestamp.
7. If validation fails, preserve edits and show field-level errors.

### Flow A3 — Manage services and prices
1. Admin opens **Services & prices**.
2. Admin selects a service or adds an allowed service.
3. Admin enters amount, currency, price basis, inclusions/exclusions, and effective/update details.
4. Server validates non-negative amount and required fields.
5. Save updates the record and audit history.
6. Price status must follow the verification workflow; admin editing alone must not automatically make an externally sourced price “verified.”
7. UI shows saved status and timestamp.

### Flow A4 — Manage availability
1. Admin opens **Availability**.
2. Admin creates or edits slots.
3. Server validates time ranges, capacity, and conflicts.
4. UI shows updated slot state and timestamp.
5. Expired/stale slots are not represented as live availability without rechecking.

### Flow A5 — Handle appointment requests
1. Admin opens request queue.
2. Admin filters by pending/status/date.
3. Admin opens a request.
4. Admin accepts, rejects, or proposes a different time.
5. Server checks authorization and valid state transition.
6. Status and audit record are saved atomically.
7. Patient sees updated status through the chosen notification/status mechanism.
8. Failed updates preserve the previous state and show an error.

## 5. ML interaction flow

1. User selects an explicit objective, such as preference-based ranking or price estimate.
2. System checks whether sufficient supported data exists.
3. If not, show “Insufficient data” and use standard filters/comparison.
4. If available, compute output using a versioned model or documented scoring logic.
5. Show the result with relevant factors, data date, and limitations.
6. Do not block core search if ML service is unavailable.
7. Never present an estimate as a guaranteed final price or medical recommendation.

## 6. Screen states

| Screen | Loading | Empty | Error | Success |
|---|---|---|---|---|
| Search results | Loading indicator | No hospitals match filters | Search failed + retry | Result cards |
| Hospital detail | Detail skeleton | Missing sections labelled unavailable | Could not load detail | Detail content |
| Comparison | Comparison loading | Select at least two hospitals | Comparison failed | Side-by-side table/cards |
| Appointment form | Submit progress | Not applicable | Field/API error | Pending reference |
| Admin queue | Loading requests | No requests yet | Could not load requests | Request list |
| ML panel | Calculating | Insufficient data | Prediction unavailable | Explained output |

## 7. State transitions

### Appointment request
```text
pending ──> accepted
   │       rejected
   │       reschedule_proposed ──> accepted / rejected / pending (if user responds)
   └──────> cancelled (if policy permits)

accepted ──> completed
accepted ──> cancelled (subject to policy)
```

Enforce transitions on the backend. Define cancellation and rescheduling policy before production.

### Price status
```text
demo → (replace with sourced record) → unverified
unverified → pending_review → verified
verified → stale/unverified when review interval expires or source changes
any state → unavailable when no current value can be supported
```

Do not change a record to `verified` without a documented verification event.

## 8. Proposed route/page map

Actual implementation may use Streamlit pages or a separate frontend.

| Page | Proposed path/name | Access |
|---|---|---|
| Home/search | `/` or `Home.py` | Public |
| Search results | `/search` | Public |
| Hospital detail | `/hospitals/{id}` | Public |
| Comparison | `/compare` | Public |
| Appointment request | `/appointments/new` | Public/authorized |
| Request status | `/appointments/{reference}` | Owner/authorized |
| Sign in | `/login` | Public |
| Admin dashboard | `/admin` | Hospital admin |
| Services & prices | `/admin/services` | Assigned admin |
| Availability | `/admin/availability` | Assigned admin |
| Requests | `/admin/requests` | Assigned admin |

## 9. Interaction acceptance checklist

- [ ] Every visible button has a defined action.
- [ ] Every API-backed action has loading, success, and failure behaviour.
- [ ] Search filters do not silently reset unexpectedly.
- [ ] Comparison supports missing values and incompatible price bases.
- [ ] Appointment requests are pending until confirmed.
- [ ] Admin authorization is enforced server-side.
- [ ] Invalid state transitions are rejected.
- [ ] ML failure does not break standard search.
- [ ] Map/API failure has a usable fallback.
- [ ] No sensitive details leak through error messages.
