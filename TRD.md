# Technical Requirements Document (TRD)
## Hospital Service Comparison

**Document status:** Proposed technical baseline; revise before production  
**Version:** 1.0  
**Last updated:** 2026-10-17

## 1. Confirmed decisions vs. proposed choices

### Confirmed
- Product serves patients/families and hospital administrators.
- Begin with sample data, then migrate to verified real data.
- Include maps, ratings, booking, admin capabilities, and ML prediction in the advanced scope.

### Proposed technical choices (not yet confirmed)
- Python 3.12+.
- Streamlit for the first user interface.
- FastAPI for backend REST APIs.
- PostgreSQL for persistent data.
- SQLAlchemy 2.x and Alembic for database access/migrations.
- Pydantic for request/response validation.
- pytest for tests and Ruff for lint/format checks.
- Docker Compose for reproducible local development.
- A configurable map provider (for example, Google Maps Platform or Mapbox) only after reviewing terms, cost, quotas, and key restrictions.
- Scikit-learn for ML experimentation, behind a separate service module.
- GitHub Actions for automated checks if the repository is hosted on GitHub.

These choices are a recommendation, not a claim that the project already uses them.

## 2. High-level architecture

```text
Patient / Admin Browser
        |
        v
Streamlit UI (presentation and session state)
        |
        v
FastAPI REST API
        |
        +------ Authentication / Authorization
        |
        +------ Hospital & Service Service
        |
        +------ Price / Provenance Service
        |
        +------ Appointment Service
        |
        +------ Comparison / Ranking Service
        |
        +------ ML Prediction Service
        |
        v
PostgreSQL
        |
        +------ Migration / seed scripts
        +------ Audit records

External adapters (optional, configured separately):
Map / Geocoding API | Verified data import | Email/SMS notification
```

For a portfolio prototype, Streamlit may initially call service modules directly while FastAPI is added as a separate backend. Avoid implementing the same business rules twice; keep core rules in reusable service functions.

## 3. Proposed repository layout

```text
hospital-service-comparison/
├── app/
│   ├── Home.py
│   └── pages/
├── backend/
│   └── main.py
├── hospital_service/
│   ├── core/config.py
│   ├── db/
│   ├── models/
│   ├── schemas/
│   ├── repositories/
│   ├── services/
│   ├── integrations/
│   └── ml/
├── migrations/
├── scripts/
│   ├── seed_demo_data.py
│   └── import_verified_data.py
├── tests/
├── docs/
│   ├── PRD.md
│   ├── TRD.md
│   ├── UI_UX_DESIGN_BRIEF.md
│   └── APP_FLOW.md
├── brain.md
├── .env.example
├── .gitignore
├── requirements.txt
├── Dockerfile
├── compose.yaml
└── README.md
```

The actual structure may change during implementation; keep `brain.md` synchronized with the repository.

## 4. Core data model

### `users`
- `id` UUID primary key
- `email` unique, normalized
- `password_hash` (never plaintext)
- `role` enum: `patient`, `hospital_admin`, `platform_admin`
- `created_at`, `updated_at`, `is_active`

### `hospitals`
- `id` UUID primary key
- `name`, `slug`
- `address_line`, `area`, `city`, `state`, `postal_code`
- `latitude`, `longitude` nullable
- `phone`, `website` nullable
- `description` nullable
- `verification_status` enum: `demo`, `unverified`, `pending_review`, `verified`
- `source_name`, `source_url` nullable
- `last_verified_at` nullable
- `created_at`, `updated_at`

### `hospital_admins`
- `user_id` foreign key
- `hospital_id` foreign key
- unique pair; scope admin permissions to assigned hospitals

### `services`
- `id` UUID primary key
- `name`, `category`, `description`
- `is_active`

### `hospital_services`
- `id` UUID primary key
- `hospital_id`, `service_id`
- `price_amount` numeric(10,2), nullable
- `currency` default `INR`
- `price_basis` (e.g. per test/package)
- `price_status` enum: `demo`, `indicative`, `verified`, `unavailable`
- `inclusions`, `exclusions` nullable
- `source_name`, `source_url` nullable
- `last_verified_at` nullable
- `updated_at`
- Unique hospital/service/basis rule to prevent accidental duplicates.

### `reviews`
- `id` UUID primary key
- `hospital_id`, `user_id` nullable
- `rating` constrained to 1–5
- `review_text` nullable
- `source_type` enum: `verified_user`, `external_import`, `demo`
- `source_reference` nullable
- `created_at`, `moderation_status`
- External review imports require provider permission and source attribution.

### `appointment_requests`
- `id` UUID primary key
- `patient_user_id` nullable for guest flow if supported
- `hospital_id`, `hospital_service_id` nullable
- `requested_start_at`, `requested_end_at` nullable
- `patient_name`, `contact_phone`, `contact_email`
- `status` enum: `pending`, `accepted`, `rejected`, `reschedule_proposed`, `cancelled`, `completed`
- `admin_note` restricted; do not store unnecessary clinical details
- `created_at`, `updated_at`

### `availability_slots` (optional initial table)
- `id`, `hospital_id`, `hospital_service_id` nullable
- `start_at`, `end_at`
- `capacity`, `booked_count`
- `status`, `last_updated_at`
- Do not represent a slot as confirmed unless the data source/admin actually confirms it.

### `audit_logs`
- `id`, `actor_user_id`, `entity_type`, `entity_id`, `action`
- `before_json`, `after_json` with sensitive fields redacted
- `created_at`

## 5. API design proposal

Prefix: `/api/v1`

| Method | Endpoint | Purpose | Access |
|---|---|---|---|
| GET | `/health` | Service health | Public, minimal |
| GET | `/hospitals` | Search/filter hospitals | Public |
| GET | `/hospitals/{hospital_id}` | Hospital detail | Public |
| GET | `/services` | List supported services | Public |
| GET | `/compare` | Compare selected hospital/service IDs | Public |
| POST | `/appointment-requests` | Submit appointment request | Public/authorized with rate limits |
| GET | `/appointment-requests/{request_id}` | Check request status | Owner/admin only or secure reference token |
| POST | `/auth/register` | Register user if enabled | Public, protected |
| POST | `/auth/login` | Authenticate | Public, protected |
| GET | `/admin/hospitals/{hospital_id}` | Admin hospital data | Assigned admin only |
| PATCH | `/admin/hospitals/{hospital_id}` | Update profile | Assigned admin only |
| POST/PATCH | `/admin/hospital-services` | Manage service/price | Assigned admin only |
| GET | `/admin/appointment-requests` | List requests | Assigned admin only |
| PATCH | `/admin/appointment-requests/{id}` | Change status | Assigned admin only |
| POST | `/ml/price-estimate` | Optional estimate | Public, with limitations |
| POST | `/ml/rank-hospitals` | Optional preference ranking | Public |

Use pagination for list endpoints, explicit filter schemas, consistent error responses, and OpenAPI documentation. Do not expose internal admin notes or other patients' information.

## 6. Search and comparison logic

- Normalize service names and location input.
- Filter by active hospital/service and data status.
- Do not silently mix unlike price bases (e.g. single test vs package).
- Missing prices remain null and display “Not available.”
- Use INR formatting for rupee prices.
- Sort by price only among comparable, available values; label whether unverified prices are included.
- Distance requires coordinates. Use a geodesic/Haversine calculation for approximate straight-line distance; label it as approximate, not driving distance.
- Route distance/time should come from a map provider and must not be invented.
- Search results should return `data_status`, `source`, and `last_verified_at` where relevant.

## 7. ML design

### Start with a baseline
Do not claim ML value before collecting a suitable dataset. First implement transparent filters and a deterministic ranking baseline.

### Candidate models (choose one after data review)
- **Price estimate:** Regression, only if sufficient comparable historical prices exist and target definition is consistent.
- **Preference-based ranking:** A transparent weighted score based on user-selected distance, price, and sourced rating.
- **Availability prediction:** Only if timestamped historical slot/booking data is available and permitted.

The initial recommended feature is **preference-based ranking**, because it can work with limited data and can be explained. If a true predictive model is later selected, record the dataset, target, features, train/test split, baseline, metrics, limitations, and model version.

### Guardrails
- Clearly label estimates as estimates.
- Show the factors that influenced a ranking.
- Do not use protected/sensitive personal attributes.
- Do not predict medical outcomes or recommend treatment.
- If input data is insufficient or outside the model's supported range, return “Insufficient data” rather than a fabricated number.
- Keep ML optional; core search and comparison must work when ML is disabled.

## 8. External integrations

### Maps/geocoding
- Put provider calls behind an adapter.
- Keep server keys in environment variables; restrict browser keys by allowed domains and API scope.
- Respect terms, quotas, attribution requirements, and privacy.
- Provide a list-based fallback when map services fail.

### Real-data onboarding
- Import only data with a lawful/authorized source.
- Store source and verification timestamps.
- Add validation for duplicate hospitals, services, and prices.
- Add a review queue before changing a record to `verified`.
- Define a re-verification/expiry interval with the data owner.

### Notifications
Email/SMS can be added after provider selection. Use a background task/queue if delivery becomes asynchronous. Never expose provider secrets in the frontend.

## 9. Security and privacy

- Use a vetted authentication library; hash passwords using a modern password-hashing algorithm.
- Enforce role and hospital-level authorization on the backend, not only in the UI.
- Validate every request and parameter; use ORM/parameterized queries.
- Add rate limits to authentication and public request endpoints.
- Keep secrets in `.env` locally and deployment secrets in the hosting platform; commit only `.env.example`.
- Avoid collecting diagnoses or detailed medical history.
- Use HTTPS in production.
- Configure backups, retention, deletion, and incident response before real patient data is used.
- Do not log passwords, tokens, full contact details, or sensitive form payloads.
- Apply applicable Indian privacy and healthcare requirements after legal review before production deployment.

## 10. Testing strategy

- Unit tests: filtering, price comparability, sorting, distance, validation, ML fallback.
- API tests: status codes, schema validation, auth and authorization.
- Integration tests: database migrations, seed/import workflow, appointment state changes.
- Security tests: patient cannot access another patient's request; admin cannot change another hospital's records.
- UI tests/manual checklist: mobile layout, empty states, map failure, API failure, invalid form.
- ML evaluation: compare to baseline; report MAE/RMSE for regression as applicable, or ranking metrics and human-review outcomes for ranking.
- Use synthetic test data and avoid committing real patient information.

## 11. Local development commands (proposed)

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
cp .env.example .env
uvicorn backend.main:app --reload
streamlit run app/Home.py
pytest
```

Commands must be updated to match the implemented package paths. On Windows, virtual-environment activation differs.

## 12. Configuration variables

Example names only; do not put real values in Git:

```dotenv
APP_ENV=development
DATABASE_URL=postgresql+psycopg://user:password@localhost:5432/hospital_compare
SECRET_KEY=replace-me
MAP_PROVIDER=
MAP_API_KEY=
DATA_SOURCE_MODE=demo
LOG_LEVEL=INFO
```

Use a secret manager or deployment environment variables in production.

## 13. Deployment proposal

- Separate frontend and backend if hosting requirements make this necessary.
- Provision PostgreSQL with backups and restricted network access.
- Run migrations as a controlled deployment step.
- Use HTTPS, environment secrets, health checks, logs, and basic monitoring.
- Do not enable real booking or verified-data claims until the data source and operational workflow have been tested end-to-end.

## 14. Technical decision log

| Decision | Status | Reason / next step |
|---|---|---|
| Streamlit UI | Proposed | Fast Python-based portfolio UI; confirm after prototype. |
| FastAPI backend | Proposed | Typed REST API and separation of concerns. |
| PostgreSQL | Proposed | Relational data and transaction support. |
| Map provider | Open | Compare terms, price, quota, and coverage before selection. |
| ML target | Open | Start with preference ranking; validate data before predictive claims. |
| Authentication | Open implementation choice | Use vetted library; enforce server-side authorization. |
