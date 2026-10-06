# SMARTLEAD Backend: Additional Development Guide

Updated: 2 October 2026

## 1. Purpose and scope

This is a modified companion to `SMARTLEAD_Backend_Python_Django_Project_Guide.md`, focused on what you still need to develop as the Django backend developer. It builds on your existing project rather than asking you to recreate completed authentication work.

Current project: `C:/Users/HP/Desktop/SDLC/sdlc_django_project/sdlc_django_project`.

Sources:

- Original backend guide: `C:/Users/HP/Downloads/SMARTLEAD_Backend_Python_Django_Project_Guide.md`.
- Project report: `C:/Users/HP/Downloads/SmartLead_Project_Report.pdf`, especially pages 3-5 (functional modules), 7-9 (backend, entities, roles, scope), and 10-12 (connected example).
- Current settings, models, serializers, views, routes, admin files, tests, and handoff documentation.

The required result is a connected backend workflow:

```text
Discover service -> Submit enquiry -> Create lead -> Assign staff
-> Record follow-ups -> Predict conversion -> Update outcome -> Report activity
```

The checkboxes below represent remaining work, not completed implementation. Endpoint names, fields, status values, and numerical thresholds labelled as proposals require agreement with the team. Example credentials and predictions in the PDF are illustrations, not configuration to copy into production.

## 2. What you already have: retain and extend

| Existing capability | Current implementation | Additional action |
|---|---|---|
| Django + DRF | Project and `api` application | Add business modules and models. |
| Registration | `POST /api/user/register/` | Extend agreed profile fields; preserve password hashing and validation. |
| Login | `POST /api/token/` | Keep JWT login; document the role/profile data React needs. |
| Refresh | `POST /api/token/refresh/` | Preserve password-aware token invalidation. |
| Password recovery/reset | Separate user/admin forgot/reset APIs | Verify actual SMTP delivery and the agreed frontend reset URL. |
| Username recovery | `POST /api/user/forgot-username/` | Retain; this is an existing extra capability. |
| Staff user listing | `GET /api/admin/users/` | Review access against super-admin user-management policy. |
| Security foundation | Password hashing, required-field/email/password validation, JWT checks, anonymous throttling | Extend to ownership, business validation, and operational permissions. |
| CORS and secrets | Environment-driven configuration | Configure actual frontend origins and deployment values. |
| Django Admin | Standard auth administration | Register and customize business models. |
| Tests and handoff | Authentication/recovery/portal tests; API documentation and Postman request files | Expand to all SMARTLEAD modules and record execution evidence. |

Current database is SQLite. `api/models.py` has no business models, and `api/admin.py` has no business registrations. A functioning login/register portal does not complete the SMARTLEAD workflow.

Current verification: 24 existing tests were run on 2 October 2026; 23 passed and `test_forgot_password` failed. It expects `http://localhost:5173/admin/reset-password/`, while configured `FRONTEND_URL` is `http://127.0.0.1:8001`. Confirm the intended frontend URL, then make the configuration and test consistent. Do not change the URL solely to silence the test. These tests do not cover the missing business modules.

## 3. Agree these contracts before schema work

- [ ] Decide whether staff and admin are one operational role or have different privileges; define super-admin responsibilities.
- [ ] Decide whether to retain built-in User plus a profile/role model or migrate deliberately to a custom User. A custom user is recommended, but changing it after auth migrations requires a data/migration plan.
- [ ] Agree fields and relationships for services, enquiries, leads, reviews, SEO, notifications, and predictions.
- [ ] Agree lead statuses, allowed transitions, terminal outcomes, and how enquiry status reflects lead progress.
- [ ] Confirm one lead per enquiry and automatic creation; define duplicate request/retry behavior.
- [ ] Agree who may view all leads versus only assigned leads, and who may assign/reassign them.
- [ ] Agree whether anonymous visitors may submit enquiries; the report leaves registration conditional, so do not assume anonymous submission without a decision.
- [ ] Confirm frontend origins, reset routes, API naming, pagination, filtering, and response/error format.
- [ ] Agree ML inputs, preprocessing ownership, artifact/service interface, model version, probability bands, and unavailable-service behavior.
- [ ] Agree notification channels, follow-up reminder timing/timezone, analytics definitions, and marketing-data collection/privacy rules.

The report's API paths are indicative. Existing auth paths can remain if React agrees; do not count a route rename as a new business feature.

## 4. Phase 1: PostgreSQL and schema foundation

- [ ] Configure PostgreSQL as the primary database using environment variables and an appropriate driver.
- [ ] Keep credentials out of source control; update `.env.example` with placeholders only.
- [ ] Back up any SQLite data you need, and define whether to import it or begin with approved demo data. Do not delete/reset the current database as a shortcut.
- [ ] Resolve the user/profile design before creating business foreign keys; use `settings.AUTH_USER_MODEL` for relationships.
- [ ] Create reviewed migrations, relational constraints, useful indexes, and a repeatable demo-data command.
- [ ] Organize code into logical applications or clearly separated modules. Exact app names are flexible.

Recommended relationships (design proposal):

| Entity | Relationship / key responsibility |
|---|---|
| User/Profile | Authentication, role and agreed contact/profile data |
| Category | One category has many services |
| Service | Category link, published details, features, pricing/packages |
| ServiceImage | Many images per service; alt text |
| Wishlist | User + service; database uniqueness for the pair |
| Enquiry | Owner, service, requirement, contact information, budget, source and agreed engagement fields |
| Lead | One-to-one enquiry link; assignee, status and conversion outcome |
| LeadFollowUp | Lead, author, notes, scheduled time, completion state if agreed |
| LeadHistory | Proposed separate audit entity for assignment/status changes; equivalent auditable design is acceptable |
| Review | User, service, rating, comment and moderation state |
| SEOContent | Service/page metadata and marketing-team content |
| MLPrediction | Lead, probability, band, model version and prediction timestamp |
| Notification | Proposed persisted alerts, recipient, event type, read state and related object |

Acceptance: a fresh PostgreSQL database migrates successfully; relationships and constraints are tested; data import, if required, is verified.

## 5. Phase 2: Complete accounts and permissions

- [ ] Add logout with an agreed refresh-token invalidation policy; explain that issued access tokens may remain valid until expiry unless immediate revocation is implemented.
- [ ] Add authenticated password change with current-password verification, matching confirmation, existing validators, and token invalidation.
- [ ] Add own-profile GET and PATCH/PUT; whitelist editable fields so users cannot change privileges.
- [ ] Implement general-user, operational staff/admin, and super-admin permissions on the server.
- [ ] Add super-admin user/role management if required by the React admin interface; distinguish it from existing staff user listing.
- [ ] Verify recovery/reset delivery, expiry, reuse prevention, inactive-account handling, and frontend links.

Proposed additional API contract:

| Method / route | Access | Purpose |
|---|---|---|
| POST `/api/auth/logout/` | Authenticated | Revoke agreed refresh token/session |
| POST `/api/auth/password/change/` | Authenticated | Verify current password and set new password |
| GET/PATCH `/api/auth/profile/` | Authenticated owner | Retrieve/update permitted profile fields |
| GET/PATCH `/api/admin/users/{id}/` | Super-admin under agreed policy | Manage account/role state |

Acceptance: normal users cannot read/update another profile, grant themselves privileges, or access operational APIs; role changes affect subsequent authorized requests.

## 6. Phase 3: Service discovery and content

The PDF explicitly includes search, filtering, sorting, packages, categories, and related services (page 3).

- [ ] Build Category, Service, ServiceImage, and agreed package representation.
- [ ] Provide public list/detail APIs for published services and categories.
- [ ] Add keyword search, category/status/price filters as agreed, ordering, and pagination.
- [ ] Return descriptions, features, pricing/packages, images/alt text, and related services.
- [ ] Add authorized category/service create/update/delete or archive operations.
- [ ] Validate prices, slugs, image uploads, publication state, and category relationships.
- [ ] Protect referenced services from destructive deletion or implement an agreed archive policy.

Proposed routes: `GET /api/categories/`, `GET /api/categories/{id}/`, `GET /api/services/`, `GET /api/services/{id}/`; administrative CRUD under `/api/admin/categories/` and `/api/admin/services/`.

Acceptance: visitors see only allowed published data; keyword/category queries and ordering work; normal users cannot mutate services; pagination and missing-object responses are documented.

## 7. Phase 4: Wishlist and reviews

- [ ] Add own-wishlist list/create/delete APIs and a database constraint preventing duplicate user-service entries.
- [ ] Add service review submission/listing with agreed rating range and review eligibility/duplicate policy.
- [ ] Add admin moderation; users cannot approve their own reviews.
- [ ] Define whether users can edit/delete their reviews, and expose only approved reviews publicly.

Proposed routes: `GET/POST /api/wishlist/`, `DELETE /api/wishlist/{id}/`, `GET/POST /api/services/{id}/reviews/`, and review moderation under `/api/admin/reviews/`.

Acceptance: object ownership is enforced; duplicate wishlist requests are handled consistently; invalid ratings return validation errors; moderation cannot be bypassed.

## 8. Phase 5: Enquiry submission and automatic lead creation

This is the central business transaction to implement.

- [ ] Add enquiry creation, own history, and own detail/status APIs.
- [ ] Capture service, requirement, contact details, budget, traffic/campaign source, keyword, and agreed engagement fields.
- [ ] Validate service availability, mandatory fields, budgets, and engagement ranges; derive trusted user/previous-enquiry data server-side where appropriate.
- [ ] Save Enquiry and its Lead in one database transaction; enforce one lead per enquiry.
- [ ] Return identifiers for both enquiry and lead, with fields visible to the submitting user limited appropriately.
- [ ] Define retry/idempotency behavior so network retries do not accidentally create duplicate leads.
- [ ] Queue or emit notifications only after a successful commit; a notification/ML failure must not leave half-created enquiry/lead records.

Proposed routes: `POST/GET /api/enquiries/`, `GET /api/enquiries/{id}/`. Staff visibility may use an admin route or role-filtered queryset under the agreed contract.

Acceptance: one valid submission creates one linked lead; rollback leaves neither record when lead creation fails; users cannot see others' enquiries; invalid submissions return 400 rather than 500.

## 9. Phase 6: Lead operations, follow-ups and history

- [ ] Provide operational lead list/detail APIs with keyword search, pagination, and filters for status, service, source and date.
- [ ] Implement assignment/reassignment only to eligible active staff/admin accounts.
- [ ] Implement validated status transitions and converted/not-converted outcomes; record conversion time when applicable.
- [ ] Keep enquiry status and user notifications consistent with agreed lead progression.
- [ ] Create/list follow-ups with notes, author and scheduled date/time; define completion/rescheduling behavior.
- [ ] Preserve history of status/assignment changes with actor and timestamp, in addition to follow-up notes.
- [ ] Apply the agreed assigned-lead versus all-lead permission policy to every operation.

Proposed routes: `GET /api/leads/`, `GET/PATCH /api/leads/{id}/`, `GET/POST /api/leads/{id}/followup/`, `GET /api/leads/{id}/history/`.

Acceptance: general users cannot manage leads; invalid assignments/statuses/dates are rejected; filter results are correct; follow-up and transition history remains auditable.

## 10. Phase 7: Notifications

The report defines a notification module on page 4; it must be covered even though the original backend guide does not detail it.

- [ ] Notify users when their enquiry status changes.
- [ ] Notify authorized administrators/staff about new enquiries.
- [ ] Expose due follow-ups and high-probability lead alerts to authorized operational users.
- [ ] Agree persisted in-app alerts versus email and any scheduled worker mechanism.
- [ ] Add own-notification list/read APIs if React needs them, with unread state and ownership enforcement.
- [ ] Prevent repeated reminders from producing duplicate alerts; record delivery/retry state where applicable.

Proposed routes: `GET /api/notifications/`, `PATCH /api/notifications/{id}/`; notification routing and recipients must follow permissions.

Acceptance: each event reaches only authorized recipients; users cannot read others' notifications; due-time boundaries and repeated task execution are tested.

## 11. Phase 8: ML prediction integration

Your responsibility is backend integration. The ML team owns training/evaluating the initial Logistic Regression model (PDF pages 5 and 9).

- [ ] Receive a versioned model artifact or prediction-service contract from the ML team.
- [ ] Prepare features consistently with training: traffic source, pages viewed, previous enquiries, time on website, service, budget, and agreed engagement score.
- [ ] Agree categorical encoding, missing values, units and feature order; do not invent preprocessing or hard-code the example 81%.
- [ ] Add authorized prediction requests and store probability, model version, timestamp, and agreed HIGH/MEDIUM/LOW band.
- [ ] Return prediction data with lead details/dashboard data as agreed.
- [ ] Handle timeout/unavailable/invalid model responses safely and preserve enquiry/lead creation.
- [ ] Record actual converted/not-converted labels for agreed historical-data handoff; expose only approved data to the ML team.

Proposed route: `POST /api/ml/predict/` with a lead identifier. Decide whether prediction is also triggered asynchronously after enquiry creation.

Acceptance: only authorized staff request/view predictions; probabilities are in [0,1]; saved outputs have a model version; service failure returns a documented safe response; tests use a controlled predictor stub.

## 12. Phase 9: Analytics, SEO and Django Admin

- [ ] Add authorized dashboard totals and agreed date-filtered reports for enquiries, leads, conversion activity, services and sources.
- [ ] Define conversion-rate denominator, time window, timezone, and status grouping before implementing calculations.
- [ ] Add due-follow-up and high-probability lead summaries; review/feedback summaries where agreed.
- [ ] Implement SEOContent storage and management for titles, meta descriptions, keywords, slugs and relevant content; coordinate alt text and internal-linking data with marketing/React.
- [ ] Capture campaign/landing-page attribution only where real data is supplied; do not claim full website traffic analytics from enquiry counts.
- [ ] Register business models in Django Admin with useful columns, search, filters, ordering, read-only audit fields and appropriate permissions.

Proposed routes: `GET /api/admin/dashboard/`, `GET /api/analytics/summary/`, and SEO management under `/api/admin/seo/`. Exact reports/routes are team decisions.

Acceptance: analytics match database fixtures; unauthorized users cannot access reports; SEO fields reach the frontend API; admin mutations respect business rules and audit history.

## 13. Phase 10: Tests, API handoff and deployment preparation

- [ ] Resolve the current password-reset URL test mismatch.
- [ ] Test migrations, constraints, permissions, validation, pagination, uploads, and safe error responses against PostgreSQL.
- [ ] Add the complete integration test below and failure/rollback variants.
- [ ] For every endpoint document method, path, purpose, role, auth, request, response, errors and status codes.
- [ ] Expand Postman files and record actual execution results; request files alone are not proof of successful testing.
- [ ] Correct old documentation commands referring to `D:\SDLC`, `.venv`, and `backend/manage.py`.
- [ ] Verify browser CORS with the actual React origin and real reset-link navigation/email delivery.
- [ ] Agree one error contract; do not expose traceback details to API clients in deployment.
- [ ] Prepare production secrets, DEBUG=false, allowed hosts/origins, PostgreSQL, static/media storage, HTTPS and email/worker configuration as required by the selected deployment.

Required integration test:

```text
Create user -> Login -> Fetch published service -> Submit enquiry
-> Verify PostgreSQL enquiry + exactly one linked lead
-> Staff lists/assigns lead -> Adds scheduled follow-up
-> Requests prediction -> Verifies persisted result
-> Updates outcome -> Verifies history + user status/notification + analytics
```

Run commands from the current project folder:

```powershell
cd C:\Users\HP\Desktop\SDLC\sdlc_django_project\sdlc_django_project
.\my_venv\Scripts\python.exe manage.py check
.\my_venv\Scripts\python.exe manage.py makemigrations --check --dry-run
.\my_venv\Scripts\python.exe manage.py migrate --check
.\my_venv\Scripts\python.exe manage.py test
.\my_venv\Scripts\python.exe -m pip check
```

These are verification commands, not a claim that future business features already pass. Review and apply new migrations during development before using `migrate --check`.

## 14. Your responsibility versus team dependencies

| Your Django backend deliverable | Input / deliverable from another team |
|---|---|
| Models, migrations, database integration | Schema and database provisioning agreement |
| REST APIs, permissions and validation | React screen needs, form fields and API contract agreement |
| SEO/content persistence and API fields | Marketing keywords, metadata and content strategy |
| Attribution storage and reporting | Frontend/marketing collection of real campaign/engagement data |
| Notification events and backend delivery | UX notification behavior, agreed email/reminder requirements |
| Prediction adapter and stored outputs | ML team's trained/evaluated model, preprocessing and versioned interface |
| Backend tests and handoff documents | QA manual results and React integration verification |

You do not need to build Figma designs, Tailwind templates, the React application, or train the ML model as part of the Django role. You do need to supply and verify the backend interfaces they consume.

Shopping cart, checkout, online payment, shipping, inventory and order fulfilment remain outside the initial scope.

## 15. Completion checklist for additional work

- [ ] PostgreSQL and reviewed business migrations work.
- [ ] Account logout/change/profile gaps and agreed roles are implemented.
- [ ] Service discovery, search/filter/sort, packages and related services work.
- [ ] Wishlist/reviews and moderation work with ownership controls.
- [ ] Enquiry creation atomically creates its lead; users can track their own enquiries.
- [ ] Lead assignment/status/filtering, scheduled follow-ups and audit history work.
- [ ] Required notifications work.
- [ ] ML integration stores real versioned predictions and handles failures.
- [ ] Dashboard analytics and SEO/content APIs meet agreed scope.
- [ ] Django Admin supports operational business data safely.
- [ ] Automated tests pass, including the full business workflow on PostgreSQL.
- [ ] Postman execution and React integration are recorded.
- [ ] API documentation and deployment configuration are ready for handoff.

Start with decisions and PostgreSQL/user design, then service discovery, then the enquiry-to-lead transaction. Those dependencies make the later lead, notification, ML and analytics work possible.
