# SMARTLEAD

Django REST Framework + PostgreSQL service discovery and lead management, with a simple Django frontend and an educational Logistic Regression predictor.

## Included

- Registration, JWT login/refresh/logout, password recovery/change, own profile and super-admin account management.
- Catalogue search/filter/sort, categories, packages, images and SEO; staff catalogue management.
- Wishlist, moderated reviews, transactional enquiry-to-lead creation and idempotent retries.
- Lead assignment/status/history, scheduled follow-ups and in-app notifications.
- Dashboard analytics and versioned prediction storage.
- Simple browser user/staff workspace, recovery portal, Django Admin and OpenAPI reference.
- PostgreSQL migrations, tests, Postman collection, TypeScript client, source PDF and planning documents.

## Start here

Read [setup](docs/SETUP.md), [API contract](docs/API_CONTRACT.md), [decisions](docs/IMPLEMENTATION_DECISIONS.md) and [verification](docs/VERIFICATION_CURRENT.md).

```text
backend/        Django source, migrations, tests and simple frontend
ml/             Training code, trusted model artifact and metrics
scripts/        Dataset download and delivery tools
docs/           Current contracts, source reports and historical plans
react-handoff/  TypeScript reference client
```

No credentials, virtual environments, customer databases, uploaded media or raw Kaggle records are shipped. Supply local PostgreSQL credentials and a generated secret using backend/.env.example. Create your own superuser.

Historical planning/review files describe the earlier authentication-only project. Current implementation behavior is documented in IMPLEMENTATION_DECISIONS.md and API_CONTRACT.md. Older backend React_API_Handoff.md/Postman YAML/verification records are history, not current instructions.

## Machine learning

Source: [X Education lead-scoring dataset](https://www.kaggle.com/datasets/lakshmikalyan/lead-scoring-x-online-education). Downloaded locally: 9,240 rows, 37 columns. Logistic Regression uses four genuine source/engagement inputs; no service/budget fields are fabricated. Raw data is not redistributed; scripts/download_dataset.py reproduces the download after reviewing applicable terms.

Held-out education data: ROC-AUC about 0.8000; accuracy at threshold .5 about 0.7576. See ml/artifacts/metrics.json for full evaluation and limitations. These results do not establish digital-service conversion accuracy. Predictions carry is_demo=true and model_version.

Production hosting, manual Postman execution, real SMTP delivery and separate React integration are not claimed. See setup for deployment prerequisites.
