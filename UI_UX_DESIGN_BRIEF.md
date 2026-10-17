# UI/UX Design Brief
## Hospital Service Comparison

**Document status:** Proposed visual direction  
**Version:** 1.0  
**Last updated:** 2026-10-17

## 1. Product experience

Create a trustworthy, calm, modern healthcare comparison experience. The interface should help users answer three questions quickly:

1. Which hospitals provide the service I need?
2. What is the listed price, and how reliable/current is that price?
3. How far away is the hospital, and how can I request an appointment?

The design must avoid implying that the cheapest hospital is always best or that unverified information is guaranteed.

## 2. Confirmed decisions and assumptions

### Confirmed
- Users include patients/families and hospital administrators.
- Initial data is sample/demo data; verified real data comes later.
- Planned features include search, comparison, maps, ratings, appointment booking, admin tools, and ML prediction.

### Proposed design assumptions
- English-first interface, with later support for Gujarati/Hindi if required.
- Responsive desktop-first dashboard that also works on mobile.
- Light theme by default for readability.
- Healthcare teal/blue with restrained green accents; avoid alarmist visual styling.
- Accessible contrast and visible labels for data status.

## 3. Visual direction

**Keywords:** Trustworthy, clear, calm, practical, transparent, modern.

Avoid:
- Overly decorative gradients that reduce contrast.
- Crowded comparison tables on mobile.
- Fake testimonials or unverified star ratings.
- Bright red used as a general decorative colour.
- Medical claims or “best hospital” badges without a defined evidence basis.

## 4. Suggested design tokens

These are proposed values, not existing implementation facts.

| Token | Suggested value | Use |
|---|---|---|
| Primary | `#0F766E` | Main actions, active navigation |
| Primary dark | `#115E59` | Hover/strong emphasis |
| Secondary | `#2563EB` | Links, selected comparison states |
| Background | `#F8FAFC` | App background |
| Surface | `#FFFFFF` | Cards and panels |
| Main text | `#0F172A` | Headings and body text |
| Muted text | `#475569` | Supporting text |
| Border | `#E2E8F0` | Dividers and card edges |
| Success | `#166534` | Confirmed/complete states |
| Warning | `#92400E` | Indicative/stale data |
| Error | `#B91C1C` | Form errors and failed actions |
| Focus ring | `#2563EB` | Keyboard focus |

Use colour together with text/icon labels. Never rely on colour alone to convey verified, pending, or failed status.

## 5. Typography and spacing

- Use a clean system sans-serif font stack; optionally use Inter if bundled and licensed appropriately.
- Page title: approximately 28–32 px desktop, 24–28 px mobile.
- Section heading: 20–24 px.
- Body: 16 px where possible.
- Supporting labels: 13–14 px, still readable.
- Use an 8 px spacing scale (8, 16, 24, 32).
- Keep paragraphs short and use descriptive labels.
- Price figures should use consistent currency formatting and aligned columns.

## 6. Information architecture

### Patient navigation
- Home / Search
- Compare
- Appointments (if signed in or request reference available)
- Help / Data transparency

### Admin navigation
- Dashboard
- Hospital profile
- Services & prices
- Availability / slots
- Appointment requests
- Audit/history (if appropriate)

Keep admin navigation separate from patient navigation and enforce permissions in the backend.

## 7. Key screens

### A. Home/search
- Clear heading and short explanation.
- Search inputs: service/test, city/area, optional hospital name.
- Primary action: “Search hospitals.”
- Popular test shortcuts may be shown, but must be configurable and not imply availability.
- Visible notice when demo data is active.

### B. Search results
- Result count and applied filters.
- Filters: service, price range, distance, sourced rating.
- Sort: price (comparable records only), distance (if coordinates available), rating (if sourced).
- Hospital cards show name, area, service match, listed price/status, approximate distance if known, rating source if known, and “View details” / “Compare.”
- Empty state suggests broadening filters; never invent results.

### C. Hospital details
- Hospital name, address, map/directions link if coordinates exist.
- Service and price table with data status, source, last verified date, and known inclusions/exclusions.
- Rating/review provenance.
- Appointment request CTA.
- Notice that prices and availability should be confirmed with the hospital.

### D. Comparison
- Side-by-side columns for selected hospitals.
- Rows: service, price, price status, source/update date, distance, sourced rating, availability status, contact/directions.
- Show “Not available” for missing values.
- Make the comparison horizontally scrollable on small screens or convert each hospital into a stacked comparison card.
- Explain when price bases differ and cannot be fairly compared.

### E. Appointment request
- Hospital and service prefilled where possible.
- Requested date/time, patient name, contact method.
- Minimum necessary fields only.
- Consent/privacy note.
- Confirmation page with request reference and status “Pending confirmation.”
- Do not label a request “Booked” until actually confirmed.

### F. Patient request status
- Reference ID, hospital, service, requested slot, status timeline.
- Restrict access through authentication or a secure, unguessable reference mechanism.

### G. Admin dashboard
- Summary counts: pending requests, active services, records needing verification.
- Service/price editor with validation and effective/update dates.
- Availability editor.
- Appointment request queue with status actions.
- Confirmation dialog for destructive/high-impact changes.
- Audit history for important updates.

### H. ML insight panel
- Name the output accurately (e.g. “Preference match,” “Estimated price,” or “Predicted availability”).
- Show factors and data date.
- Display limitations and “insufficient data” fallback.
- Do not use authoritative clinical language or imply a prediction is guaranteed.

## 8. Component behaviour

- Buttons have clear action labels; avoid vague labels like “Go.”
- Loading states: skeleton/list placeholder or spinner plus brief status text.
- Empty states: explain why no results appeared and offer a safe next step.
- Error states: concise message, retry where useful, no raw stack trace.
- Form errors: shown next to the field and summarized when necessary.
- Confirmation: show what happened and the next step.
- Destructive admin actions require confirmation.
- Disable actions only when the reason is visible.
- Keep filter state when returning from details to results where practical.

## 9. Data-status language

Use explicit status text:
- **Demo data** — synthetic data for demonstration only.
- **Unverified** — source has not been checked sufficiently.
- **Indicative price** — may differ from the final amount.
- **Verified on [date]** — verified by the project's documented process on that date.
- **Not available** — no value available.
- **Pending confirmation** — appointment request received, not yet accepted.
- **Confirmed** — only after an authorized confirmation event.

Do not use “verified” merely because a record exists in the database.

## 10. Accessibility and responsive design

- Maintain WCAG-informed contrast; verify actual colour pairs with an accessibility checker.
- All inputs have programmatic labels.
- Keyboard focus is visible and logical.
- Buttons and interactive targets are sufficiently large.
- Tables have headers and meaningful reading order.
- Status uses text/icons as well as colour.
- Do not communicate meaning through colour alone.
- Test at mobile, tablet, and desktop widths.
- Respect browser zoom and avoid fixed-width layouts that require horizontal page scrolling.

## 11. UX acceptance checklist

- [ ] User can start a search without needing to understand technical terms.
- [ ] Demo data is visibly labelled.
- [ ] Each displayed price communicates its status.
- [ ] Missing values never appear as zero or false ratings.
- [ ] Comparison makes unlike price bases obvious.
- [ ] Booking request clearly says pending until confirmed.
- [ ] Admin actions are role-restricted.
- [ ] Map failure does not prevent list-based search.
- [ ] Keyboard-only use is possible for core journeys.
- [ ] Mobile comparison remains usable.
- [ ] ML outputs include explanation and limitations.
