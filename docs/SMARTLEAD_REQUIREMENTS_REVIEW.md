# SMARTLEAD requirements review

Reviewed 2 October 2026 against `C:/Users/HP/Downloads/SMARTLEAD_Backend_Python_Django_Project_Guide.md`.
Project: `C:/Users/HP/Desktop/SDLC/sdlc_django_project/sdlc_django_project`.

## Verdict

The project does not yet satisfy the SMARTLEAD backend requirements. It provides an authentication/account-recovery foundation, but the core service-discovery and enquiry-to-lead business workflow is absent.

This is an assessment, not an instruction to implement everything in the guide. The guide identifies exact model fields, app layout, lead statuses, several endpoint designs, prediction thresholds, and analytics details as proposals or team decisions. Equivalent routes can satisfy a capability if the frontend contract is agreed; route spelling alone is not the main gap.

## Coverage

| Requirement | Status | Evidence / gap |
|---|---|---|
| Django and DRF setup | Present | Settings install DRF and the `api` app; URL routing exists. |
| PostgreSQL | Missing | `sdlc_django_project/settings.py` configures SQLite; no PostgreSQL driver appears in requirements. |
| Migrations | Partial | Built-in auth/admin/contenttypes/session migrations were applied during this session. `api/migrations` contains only `__init__.py`; no business schema exists. |
| Registration and login | Present | `/api/user/register/` and `/api/token/`; Django password hashing and validation. |
| JWT refresh | Present | `/api/token/refresh/`, password-aware revocation checks. |
| Password reset | Present for development | Separate staff/user recovery and reset endpoints. Console email defaults; actual SMTP delivery not verified. |
| Logout | Missing server API | No logout route or refresh-token blacklist configuration. Client token removal alone does not revoke a JWT. |
| Authenticated password change | Missing API | Recovery reset exists; no authenticated current-password/change endpoint. Django Admin password changes are not the required general-user API. |
| User profile retrieval/update | Missing API | No own-profile GET/PATCH/PUT endpoint. |
| Roles and permissions | Partial | Built-in `User`, `is_staff`, `is_superuser`; `IsAdminUser` protects the user list. No complete SMARTLEAD operational role/permission policy. Staff can list users, which needs review against the guide's super-admin responsibilities. |
| Custom user model | Recommended feature absent | Code imports Django's built-in `User`; no custom role/phone model. The guide prefers a custom user rather than mandating exact fields. Changing this after migrations requires a deliberate migration plan. |
| Categories, services, service images | Missing | `api/models.py` defines no models; no relevant routes. |
| Wishlist and reviews | Missing | No models, routes, duplicate/ownership controls, or moderation. |
| Enquiries | Missing | No creation, history, detail, ownership filtering, or service/budget validation. |
| Automatic enquiry-to-lead creation | Missing | No Enquiry/Lead models or transaction implementing this workflow. Registration's transaction does not implement it. |
| Leads | Missing | No list/detail/update, assignment, status changes, or status/service/source/date filters. |
| Follow-ups | Missing | No model, endpoint, history, or operational permissions. |
| ML prediction integration | Missing | No feature preparation, prediction interface, persistence, versioning, or failure handling. Backend integration is required; training belongs to the ML team. |
| SEO content | Missing / details undefined | Listed as a core entity in the guide, but fields and API expectations are unspecified. |
| Analytics | Missing / scope undefined | No analytics APIs; the team must specify required metrics. |
| Django Admin | Partial | Standard Django admin route and built-in auth administration; `api/admin.py` has no business model registration/customization. |
| CORS | Present configuration | Middleware and environment-driven allowed origins; verify actual React origins before handoff. |
| Secrets/environment configuration | Present locally | `.env` is loaded; `.gitignore` excludes it. Git history and remote credential exposure were not audited. |
| Production configuration | Incomplete | Current local setup uses DEBUG=true, SQLite, and development email. Production deployment behavior not verified. |
| Input validation and safe errors | Partial | Auth serializers validate required fields, emails, and passwords; business validation is absent. Error shapes vary between field errors, `error`, and `detail`. |
| React API documentation | Partial | `React_API_Handoff.md` documents the nine existing endpoints, not SMARTLEAD business APIs. Some commands refer to an older `D:\SDLC`, `.venv`, `backend/manage.py` layout. |
| Postman testing | Evidence incomplete | Request YAML files exist. A collection/checklist does not prove every scenario has been manually executed; handoff documentation explicitly marks unexecuted scenarios as pending. |
| Automated tests | Partial / failing | 24 tests cover authentication/recovery/portal behavior; no SMARTLEAD business tests. Current run: 23 passed, 1 failed because password-reset test expects localhost:5173 while configured FRONTEND_URL is 127.0.0.1:8001. |

## Evidence locations

- `api/models.py`: no application models.
- `api/admin.py`: no application model registrations.
- `api/urls.py`: seven registration/recovery/user-list endpoints.
- `sdlc_django_project/urls.py`: JWT login/refresh and built-in admin; total nine REST endpoints.
- `api/views.py`: built-in User operations and staff-only user list.
- `api/serializers.py`: registration/recovery validation, user list, refresh handling.
- `sdlc_django_project/settings.py`: SQLite, JWT, CORS, dotenv, email settings.
- `api/tests.py`, `api/test_portal.py`: existing tests.
- `VERIFICATION.md`: historical claim of 24 passing tests; the current local test result differs.

## Recommended sequence

1. Agree roles, schema, enquiry-to-lead behavior, lead statuses, API/error contract, actual frontend origins, ML interface/thresholds, and required analytics.
2. Plan PostgreSQL and user-model migration before building relationships. Complete logout, authenticated password change, and profile APIs.
3. Build Category/Service/ServiceImage, then wishlist/reviews.
4. Implement transactional enquiry creation plus lead creation with ownership and staff permission tests.
5. Add lead assignment/status/filtering and follow-up history.
6. Integrate the ML team's prediction model/service and persist results; add specified analytics and SEO features.
7. Customize admin, complete API handoff documentation, record Postman results, and add the guide's full business-workflow integration test.

Do not add shopping cart, payments, shipping, or inventory: the guide explicitly excludes them from the initial scope.
