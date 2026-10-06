# SMARTLEAD Project Flow and Backend Plan

Prepared: 2 October 2026

Source: `C:/Users/HP/Downloads/SmartLead_Project_Report.pdf`.

This document describes the intended final project flow understood from the report. It is a planning document, not a claim that all features are already implemented. Diagrams use Mermaid and can be viewed in a Markdown viewer that supports Mermaid.

## 1. Project purpose

SMARTLEAD is a digital service discovery and lead management platform. A user discovers a service, submits an enquiry, and the organization manages that enquiry as a lead. Staff record follow-ups and actual outcomes; machine learning estimates conversion probability to help prioritize work.

```text
Service Discovery -> Enquiry -> Lead -> Follow-up -> Conversion Outcome
```

Shopping cart, checkout, payments, shipping, inventory, and order fulfilment are outside the initial scope.

## 2. Main business flow

```mermaid
flowchart TD
    A["Marketing / Google search / Direct visit"] --> B["User browses services"]
    B --> C["Search, filter and view service details"]
    C --> D["Register / Log in when required"]
    D --> E["Submit enquiry<br/>Service, requirement, budget, contact and source"]

    subgraph BACKEND["Django backend"]
        E --> F["Authenticate and validate"]
        F --> G["Save enquiry in PostgreSQL"]
        G --> H["Create linked lead"]
        H --> I["Notify authorized staff"]
    end

    I --> J["Admin / Staff views lead"]
    J --> K["Assign responsible staff member"]
    K --> L["Contact customer<br/>Record notes and schedule follow-ups"]
    L --> M["Update lead status and history"]
    M --> N{"Final outcome known?"}
    N -->|"Still open"| L
    N -->|"Converted"| O["Record converted outcome"]
    N -->|"Not converted"| P["Record not-converted outcome"]

    M --> Q["User sees permitted enquiry status<br/>and receives notifications"]
    O --> R["Dashboard and business analytics"]
    P --> R

    H --> S["Request ML prediction when available"]
    S --> T["Prepare agreed lead features"]
    T --> U["Trained model / Prediction service"]
    U --> V["Store probability, band and model version"]
    V --> J
```

Implementation proposal: save the enquiry and its linked lead in one database transaction. Deliver notifications after the transaction commits. An unavailable prediction service must not prevent a valid enquiry from being stored.

Whether anonymous enquiry submission is allowed, when prediction is triggered, and how enquiries map to leads must be agreed before implementation.

## 3. User journey

```mermaid
flowchart LR
    A["Browse / Search services"] --> B["View details, features and packages"]
    B --> C["Register / Log in"]
    C --> D["Submit enquiry"]
    D --> E["View own enquiry history and status"]
    E --> F["Receive status notifications"]
    C --> G["Manage own profile"]
    C --> H["Save favourites"]
    C --> I["Submit ratings / feedback"]
```

The user must only access their own private profile, enquiries, wishlist, and notifications. Public review visibility follows the agreed moderation policy. Internal staff notes and predictions should only be exposed under the agreed role policy.

## 4. Administrative journey

```mermaid
flowchart TD
    A["Staff / Admin logs in"] --> B["Authorized dashboard"]
    B --> C["View new enquiries and leads"]
    C --> D["Assign / Reassign lead"]
    D --> E["View details and available prediction"]
    E --> F["Contact customer and record follow-up"]
    F --> G["Schedule next follow-up"]
    G --> H["Update status and preserve history"]
    H --> I["Record final conversion outcome"]
    B --> J["Manage permitted services and categories"]
    B --> K["Moderate reviews and manage SEO content"]
    B --> L["View permitted analytics and alerts"]
    M["Super Admin"] --> N["Manage users, roles, permissions and system settings"]
```

Staff/admin access is operational. Super-admin access includes system-wide administration. Exact assignment, visibility, and role-management permissions require agreement and server-side enforcement.

## 5. System architecture and responsibility

```mermaid
flowchart TB
    A["React user interface"] --> D["Django REST APIs"]
    B["React admin interface"] --> D
    C["Django Admin<br/>Internal administration"] --> E["Business rules and permissions"]
    D --> E
    E --> F[("PostgreSQL")]
    E --- G["Accounts and roles"]
    E --- H["Categories, services, images and packages"]
    E --- I["Wishlist, reviews and moderation"]
    E --- J["Enquiries, leads, follow-ups and history"]
    E --- K["Notifications, SEO and analytics"]
    E --- L["ML prediction integration"]
```

React consumes Django APIs rather than connecting directly to PostgreSQL or the model. Django owns validation, authorization, business operations, persistence, and the prediction integration interface. Django Admin supports internal administration and does not replace React-facing REST APIs.

| Team | Responsibility |
|---|---|
| Django backend | Models, migrations, PostgreSQL integration, APIs, permissions, business rules, notifications, reports, admin and prediction integration |
| React | User/admin screens, routing, forms, API consumption and agreed engagement-data collection |
| ML | Dataset preparation, model training/evaluation, preprocessing contract and versioned prediction capability |
| Marketing | Keywords, SEO content, source/campaign definitions and content strategy |
| UI/UX and HTML/Tailwind | Approved interface designs and responsive templates |
| Database / QA | Coordinate schema/data management and verify the integrated application with the development teams |

## 6. ML training and prediction are separate flows

### Training flow

```mermaid
flowchart LR
    A["Historical leads<br/>with confirmed outcomes"] --> B["ML team cleans and prepares data"]
    B --> C["Split data for training and evaluation"]
    C --> D["Train and evaluate Logistic Regression"]
    D --> E["Versioned model / Prediction service"]
```

### Prediction flow

```mermaid
flowchart LR
    A["New lead in Django"] --> B["Build agreed features"]
    B --> C["Versioned model / Prediction service"]
    C --> D["Conversion probability"]
    D --> E["Store prediction and metadata"]
    E --> F["Authorized staff prioritizes follow-up"]
```

Potential inputs in the report include traffic source, pages viewed, previous enquiries, time on website, selected service, budget, and engagement score. Field definitions, units, availability and preprocessing must match the trained model.

Actual outcome labels are `converted = 1` and `not converted = 0`. Open/unresolved leads must not automatically be labelled as failures. Only features available at prediction time should be used; later status changes and outcomes are not valid inputs for an initial prediction.

The PDF supplies example values, not a training dataset. The selected Kaggle X Education lead-scoring dataset is a candidate for the initial demonstration; its download/import has not completed. Its field meanings and usage terms need inspection before use. A demonstration model trained on education leads does not establish accuracy for digital-service customers. Missing budget/service information must not be fabricated and presented as genuine observations.

The report's example 81% is illustrative. Staff record actual conversion; the prediction is an estimate, not a guarantee or a replacement for the outcome.

## 7. Main data relationships

This is a planning relationship diagram. Exact fields and package/history/notification representations remain design decisions.

```mermaid
flowchart TD
    U["User"] --> E["Enquiry"]
    U --> W["Wishlist"]
    U --> R["Review"]
    C["Category"] --> S["Service"]
    S --> SI["Service images / Packages"]
    S --> E
    S --> W
    S --> R
    S --> SEO["SEO content"]
    E --> L["Lead"]
    L --> F["Follow-ups"]
    L --> H["Status / Assignment history"]
    L --> P["ML predictions"]
    L --> N["Notification events"]
    E --> N
```

## 8. Development plan

| Phase | Deliverable | Completion evidence |
|---|---|---|
| 1 | Agree schema, roles, lead statuses, origins and contracts; configure PostgreSQL | Reviewed decisions and successful database migrations |
| 2 | Extend existing auth with profile, authenticated password change and logout policy | Account and privilege tests pass |
| 3 | Categories and service discovery: details, search, filtering, sorting, media and packages | Public discovery and authorized management APIs work |
| 4 | Wishlist and moderated reviews | Ownership, duplicate and moderation tests pass |
| 5 | Enquiry submission/tracking and transactional lead creation | One submission creates the agreed linked records; rollback is tested |
| 6 | Lead assignment/status/search/filtering, scheduled follow-ups and history | Operational permissions and transition/history tests pass |
| 7 | Status/new-enquiry/due-follow-up notifications | Authorized recipients, timing and duplicate prevention are verified |
| 8 | ML prediction integration | Versioned results are stored; model failures are handled safely |
| 9 | Analytics, SEO and customized Django Admin | Reports match fixtures; required content and admin operations work |
| 10 | Full workflow tests, Postman evidence and React handoff | Documented APIs and integrated acceptance tests pass |

Existing registration, JWT login/refresh, password recovery and basic configuration should be retained and extended. The current authentication foundation does not yet provide the business workflow above.

## 9. Decisions to resolve

- [ ] Exact role model and staff/admin/super-admin permissions.
- [ ] User model/profile migration strategy before adding business relationships.
- [ ] Service fields, packages, price representation and publishing rules.
- [ ] Anonymous versus authenticated enquiries and contact requirements.
- [ ] Lead creation/assignment rules, statuses, allowed transitions and final outcomes.
- [ ] Enquiry-status mapping and notification channels/timing.
- [ ] Prediction interface, feature definitions, model version and probability-band thresholds.
- [ ] Analytics metrics, conversion-rate denominator and reporting windows.
- [ ] API request/response/error format, frontend origins and reset-link routes.

## 10. End-to-end acceptance scenario

```text
1. User browses a published service.
2. User registers/logs in and submits a valid enquiry.
3. PostgreSQL contains the enquiry and its linked lead.
4. Authorized staff receives a new-enquiry alert and accesses the lead.
5. Lead is assigned; staff records a scheduled follow-up.
6. Prediction is requested, returned and stored with model metadata.
7. Staff updates status; history and permitted user notifications are recorded.
8. Staff records the actual converted/not-converted outcome.
9. Dashboard metrics reflect the records under the agreed definitions.
10. Another general user cannot access the private records or administrative operations.
```

Also test invalid input, missing objects, unauthorized requests, transaction rollback, duplicate retries, and unavailable ML/email services. No project server or dataset import needs to be started to review this plan.
