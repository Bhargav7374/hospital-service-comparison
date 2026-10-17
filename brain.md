# brain.md — AI Project Context
## Hospital Service Comparison

This file is the compact, maintained context for AI coding assistants. Read it before making code changes. Update it when architecture, files, commands, data models, or decisions change.

**Status:** Planning baseline — implementation has not been confirmed by this document.  
**Last updated:** 2026-10-17

## 1. Project summary

Build a portfolio-ready web application that helps patients/families search for hospitals, compare diagnostic-service prices and other sourced information, view locations, and request appointments. Hospital administrators manage their own hospital information, service prices, availability, and appointment requests. Advanced scope includes an ML feature, but core search/comparison must work without ML.

## 2. Confirmed decisions

- Target users: patients/families and hospital administrators.
- Data strategy: start with sample/demo data, then transition to verified real data.
- Desired scope: advanced project with maps, ratings, booking, admin features, and ML prediction.
- Required planning files:
  - `docs/PRD.md`
  - `docs/TRD.md`
  - `docs/UI_UX_DESIGN_BRIEF.md`
  - `docs/APP_FLOW.md`
  - `brain.md`

## 3. Proposed choices — not confirmed until implemented

- Python 3.12+
- Streamlit UI
- FastAPI REST backend
- PostgreSQL database
- SQLAlchemy + Alembic
- Pydantic validation
- pytest and Ruff
- Scikit-learn for ML experiments
- Map/geocoding provider selected only after reviewing cost, terms, quota, coverage, and privacy.
- Initial launch geography may be Ahmedabad/Gujarat; this remains an assumption.

Do not claim these technologies, providers, or features are already working until code and tests prove it.

## 4. Non-negotiable product rules

1. Demo data must be clearly labelled as demo data.
2. Never invent real hospital prices, ratings, certifications, addresses, or available slots.
3. Missing values are `null`/“Not available,” never zero by default.
4. Every real-data price should include a source, verification status, and last-verified timestamp where available.
5. A submitted appointment is `pending` until an authorized hospital/admin confirms it.
6. Do not give diagnoses or treatment recommendations.
7. ML output is an estimate/ranking, not a guarantee; show limitations and fallback when data is insufficient.
8. Keep patient information minimal. Do not store unnecessary clinical details.
9. Enforce authorization in the backend, not only by hiding UI controls.
10. Never commit secrets, real patient data, API keys, or passwords.

## 5. Expected architecture

```text
Streamlit UI → FastAPI → reusable services/repositories → PostgreSQL
                                 ├── data-source adapters
                                 ├── map adapter
                                 └── optional ML module
```

Keep business logic in reusable service functions. Avoid duplicating comparison and validation logic across UI and API.

## 6. Core entities

- User and role
- Hospital
- Hospital-admin assignment
- Service
- Hospital service/price record
- Review/rating with provenance
- Appointment request
- Optional availability slot
- Audit log

See `docs/TRD.md` for proposed fields and API endpoints.

## 7. ML plan

Start with transparent filters and a deterministic baseline. The preferred first advanced feature is an explainable preference-based ranking (price, distance, and sourced rating weights), because genuine historical data may be limited. A true predictive price/availability model must not be selected until the dataset, target, quality, and evaluation plan are established.

If ML is unavailable or data is insufficient, the normal search and comparison experience must still work.

## 8. Development workflow

1. Inspect the existing repository before changing files.
2. Confirm actual package paths and current implementation; do not assume the proposed structure exists.
3. Implement in small, testable phases.
4. Add/update tests for every business rule.
5. Run tests and linters where configured.
6. Update README and this file when commands or architecture change.
7. Never overwrite user work without inspecting it first.
8. Keep secrets in environment variables and `.env.example` placeholders only.
9. Use migrations for schema changes once a database is introduced.
10. Report clearly what was changed, what passed, and what remains incomplete.

## 9. Suggested implementation phases

### Phase 0 — Repository and setup
- Inspect repository and Git status.
- Create the documentation files.
- Add `.gitignore`, `.env.example`, setup instructions.
- Decide package structure and environment.

### Phase 1 — Demo-data search and comparison
- Define hospital/service/price schemas.
- Add synthetic seed data with explicit demo labels.
- Build search, filters, sorting, detail view, and side-by-side comparison.
- Add unit tests for comparable prices, missing values, and filters.

### Phase 2 — Backend and persistence
- Add FastAPI, PostgreSQL, migrations, repositories, and API tests.
- Move business rules into service layer.
- Add validation and useful error responses.

### Phase 3 — Admin and appointments
- Add authentication and hospital-scoped authorization.
- Add profile, service/price, slot, and request management.
- Add appointment state transitions and audit logs.
- Test unauthorized access and invalid transitions.

### Phase 4 — Maps, ratings, verified data
- Select approved providers/sources.
- Add map adapter and list fallback.
- Add sourced rating workflow; do not fabricate ratings.
- Add data provenance, verification review, and staleness handling.

### Phase 5 — ML, deployment, and polish
- Define ML target and evaluation.
- Compare against a simple baseline.
- Add explanation and insufficient-data fallback.
- Deploy with secrets, health checks, logging, and backups.
- Complete accessibility, security, and user-journey tests.

## 10. Proposed local commands

These are placeholders until the actual structure is created and validated:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
cp .env.example .env
uvicorn backend.main:app --reload
streamlit run app/Home.py
pytest
```

Do not tell the user a command works until the corresponding file/dependency exists. On macOS/Linux, `source .venv/bin/activate` is typical; Windows uses a different activation command.

## 11. Open decisions to resolve

- Final launch geography and supported languages.
- Map provider and geocoding source.
- Approved sources for hospital/service/pricing data.
- Authentication method and whether patients need accounts.
- Exact ML target and dataset.
- Appointment notification provider.
- Production data retention, deletion, consent, and compliance requirements.
- Hosting platform and database provider.

## 12. AI response expectations

When coding this project:
- Prefer simple, correct, maintainable implementation over unnecessary complexity.
- Explain setup steps in clear, beginner-friendly English or Gujarati-English when useful.
- Provide exact file paths and complete code where requested.
- Distinguish confirmed facts, proposed decisions, and assumptions.
- Do not fabricate successful tests, deployments, APIs, data, or links.
- Surface security/privacy concerns before adding real user data.
- Keep project scope practical for a student portfolio and implement incrementally.
