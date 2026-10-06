# SMARTLEAD Backend Developer Project Guide

## 1. Project Overview

**SMARTLEAD --- AI-Powered Digital Service Discovery & Lead Management
System** is a full-stack platform for a digital/technology service
organization.

It is **not an e-commerce application**. Its main business workflow is:

``` text
Service Discovery
      ↓
User Enquiry
      ↓
Lead Creation
      ↓
Admin / Staff Follow-up
      ↓
ML Conversion Prediction
      ↓
Converted / Not Converted
```

A visitor discovers services such as Web Development, Digital Marketing,
UI/UX, or AI/ML services. After the visitor submits an enquiry, the
backend stores the enquiry and turns it into a business lead.
Staff/admin users then manage the lead, record follow-ups, update its
status, and view a machine-learning prediction of its conversion
probability.

------------------------------------------------------------------------

# 2. Where I Fit in the Project

My role is:

> **Backend Developer --- Python + Django + Django REST Framework +
> PostgreSQL**

The overall technical chain is:

``` text
Digital Marketing
      ↓
UI/UX (Figma)
      ↓
HTML + Tailwind CSS
      ↓
React Frontend
      ↓
=================================
      MY BACKEND LAYER
Python + Django + DRF
      ↓
PostgreSQL
      ↓
ML Prediction Integration
=================================
```

The React developer does **not** communicate directly with PostgreSQL or
the ML model.

React communicates with the **Django REST APIs** that I create.

------------------------------------------------------------------------

# 3. My Core Responsibilities

I am responsible for the backend business logic and API layer.

### Backend Development

-   Create the Django project.
-   Divide the backend into logical Django applications.
-   Implement application business rules.
-   Create Django models.
-   Create serializers.
-   Create REST API views/viewsets.
-   Configure URL routing.
-   Validate incoming data.
-   Return proper JSON responses and HTTP status codes.
-   Handle backend exceptions safely.

### Authentication and Authorization

Implement:

-   User registration
-   Login
-   Logout
-   Password change
-   Password reset
-   User profile
-   Authentication tokens/JWT as agreed by the team
-   Role-based permissions

Roles defined by the project:

``` text
GENERAL_USER
ADMIN / STAFF
SUPER_ADMIN
```

A general user must not be able to access administrative lead-management
APIs.

### PostgreSQL

I must connect Django to PostgreSQL and create/manage the database
schema through Django models and migrations.

Core entities include:

``` text
User
Category
Service
ServiceImage
Wishlist
Enquiry
Lead
LeadFollowUp
Review
SEOContent
MLPrediction
```

### REST APIs

The React frontend depends on my APIs.

Therefore I must provide:

``` text
HTTP Method
API URL
Authentication requirement
Request JSON
Response JSON
Possible errors
HTTP status codes
```

for every endpoint handed to the React developer.

### Django Admin

Configure Django Admin so authorized internal users can manage relevant
backend data.

### ML Integration

The ML team is responsible for training/evaluating the lead-conversion
model.

My responsibility is to integrate the resulting prediction capability
into Django so the application can send lead features to the prediction
service/model and return/store the result.

------------------------------------------------------------------------

# 4. Recommended Django Project Structure

``` text
smartlead/
│
├── manage.py
├── requirements.txt
├── .env
│
├── smartlead/
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
│
├── accounts/
├── categories/
├── services/
├── enquiries/
├── leads/
├── reviews/
├── analytics/
└── ml_prediction/
```

Each application can contain:

``` text
models.py
serializers.py
views.py
urls.py
permissions.py
admin.py
tests.py
```

------------------------------------------------------------------------

# 5. Backend Data Flow

## General User Flow

``` text
React
  ↓
Django REST API
  ↓
Authentication / Permission Check
  ↓
Serializer Validation
  ↓
Business Logic
  ↓
Django ORM
  ↓
PostgreSQL
  ↓
JSON Response
  ↓
React
```

## Enquiry-to-Lead Flow

``` text
User selects Service
        ↓
User submits Enquiry
        ↓
POST /api/enquiries/
        ↓
Django validates enquiry
        ↓
Enquiry saved in PostgreSQL
        ↓
Lead created
        ↓
Admin can see Lead
        ↓
ML prediction requested
        ↓
Probability returned/stored
        ↓
Admin follows up
        ↓
Lead status updated
```

A useful implementation improvement is to make enquiry creation and
automatic lead creation **transactional**, so the system does not end up
with an enquiry saved without its corresponding lead if an error occurs.

------------------------------------------------------------------------

# 6. Main Django Models

The exact fields are not fully specified in the supplied project report,
so the following schema is an implementation proposal that should be
finalized with the team before migrations are locked.

## User

Prefer a custom user model from the beginning.

Example fields:

``` text
id
email
username
password
first_name
last_name
role
phone
is_active
created_at
updated_at
```

Possible roles:

``` text
GENERAL_USER
ADMIN
SUPER_ADMIN
```

------------------------------------------------------------------------

## Category

``` text
id
name
description
slug
is_active
created_at
```

Relationship:

``` text
Category
   │
   └── Service (one-to-many)
```

------------------------------------------------------------------------

## Service

``` text
id
category
name
slug
description
features
starting_price
status
created_at
updated_at
```

------------------------------------------------------------------------

## ServiceImage

``` text
id
service
image
alt_text
```

Relationship:

``` text
Service
   │
   └── ServiceImage (one-to-many)
```

------------------------------------------------------------------------

## Wishlist

``` text
id
user
service
created_at
```

A user can save services as favourites.

------------------------------------------------------------------------

## Enquiry

Possible fields:

``` text
id
user
service
requirement
budget
traffic_source
keyword
pages_viewed
time_on_website
previous_enquiries
contact_details
status
created_at
updated_at
```

------------------------------------------------------------------------

## Lead

``` text
id
enquiry
customer
service
source
status
assigned_to
created_at
updated_at
converted_at
```

Example statuses may include:

``` text
NEW
CONTACTED
FOLLOW_UP
QUALIFIED
CONVERTED
NOT_INTERESTED
```

The report does not define a final status enumeration, so the team
should approve the exact values.

------------------------------------------------------------------------

## LeadFollowUp

``` text
id
lead
created_by
notes
follow_up_date
created_at
```

Relationship:

``` text
Lead
  │
  └── LeadFollowUp (one-to-many)
```

------------------------------------------------------------------------

## Review

``` text
id
user
service
rating
comment
is_approved
created_at
```

------------------------------------------------------------------------

## MLPrediction

Possible fields:

``` text
id
lead
probability
probability_band
model_version
predicted_at
```

Example:

``` text
probability = 0.81
probability_band = HIGH
```

The prediction is an estimate, not a guarantee of conversion.

------------------------------------------------------------------------

# 7. Database Relationship Overview

``` text
User
 ├── Enquiry
 ├── Wishlist
 ├── Review
 └── LeadFollowUp (when staff/admin)

Category
 └── Service
       ├── ServiceImage
       ├── Enquiry
       ├── Wishlist
       └── Review

Enquiry
   ↓
Lead
   ├── LeadFollowUp
   └── MLPrediction
```

------------------------------------------------------------------------

# 8. API Groups I Need to Build

## Authentication APIs

``` text
POST /api/auth/register/
POST /api/auth/login/
POST /api/auth/logout/
POST /api/auth/password/change/
POST /api/auth/password/reset/
GET  /api/auth/profile/
PUT/PATCH /api/auth/profile/
```

------------------------------------------------------------------------

## Service APIs

``` text
GET  /api/services/
GET  /api/services/{id}/
```

Administrative CRUD may include:

``` text
POST   /api/admin/services/
PATCH  /api/admin/services/{id}/
DELETE /api/admin/services/{id}/
```

------------------------------------------------------------------------

## Category APIs

``` text
GET /api/categories/
GET /api/categories/{id}/
```

Admin CRUD should be added as required.

------------------------------------------------------------------------

## Wishlist APIs

Possible design:

``` text
GET    /api/wishlist/
POST   /api/wishlist/
DELETE /api/wishlist/{id}/
```

------------------------------------------------------------------------

# 9. Enquiry APIs

``` text
POST /api/enquiries/
GET  /api/enquiries/
GET  /api/enquiries/{id}/
```

Important permission rule:

``` text
GENERAL_USER → only own enquiries
ADMIN/STAFF  → authorized operational enquiries
```

Example request:

``` json
{
  "service_id": 1,
  "requirement": "Responsive business website with admin panel",
  "budget": 50000,
  "traffic_source": "Google Organic Search",
  "keyword": "web development services",
  "pages_viewed": 10,
  "time_on_website": 500
}
```

Possible response:

``` json
{
  "message": "Enquiry created successfully",
  "enquiry_id": 25,
  "lead_id": 25
}
```

------------------------------------------------------------------------

# 10. Lead Management APIs

Administrative endpoints may include:

``` text
GET   /api/leads/
GET   /api/leads/{id}/
PATCH /api/leads/{id}/
POST  /api/leads/{id}/followup/
```

Filtering requirements from the project include:

``` text
status
service
source
date
```

Therefore the API should support a filtering strategy agreed with the
frontend team.

Example:

``` text
GET /api/leads/?status=NEW
```

------------------------------------------------------------------------

# 11. Follow-Up API

Example:

``` text
POST /api/leads/{id}/followup/
```

Request:

``` json
{
  "notes": "Customer requested a follow-up call.",
  "follow_up_date": "2026-10-05"
}
```

Backend responsibilities:

``` text
Authenticate staff/admin
        ↓
Check permission
        ↓
Verify lead exists
        ↓
Validate input
        ↓
Create follow-up
        ↓
Update related state if required
        ↓
Return JSON response
```

------------------------------------------------------------------------

# 12. Machine Learning Prediction Integration

The initial ML problem is binary classification:

``` text
Converted = 1
Not Converted = 0
```

The project proposes Logistic Regression as the initial model.

Potential features include:

``` text
Traffic source
Pages viewed
Previous enquiries
Time on website
Service selected
Budget
Engagement score
```

API:

``` text
POST /api/ml/predict/
```

Conceptual flow:

``` text
Django Lead Data
      ↓
Feature Preparation
      ↓
Trained ML Model / Prediction Service
      ↓
predict_proba()
      ↓
Conversion Probability
      ↓
MLPrediction record
      ↓
Admin Dashboard
```

Example output:

``` json
{
  "lead_id": 25,
  "conversion_probability": 0.81,
  "probability_band": "HIGH"
}
```

The thresholds for HIGH/MEDIUM/LOW are not specified in the report and
must be agreed before implementation.

------------------------------------------------------------------------

# 13. Role-Based Access Control

## GENERAL_USER

Can typically:

``` text
Register
Login/logout
Manage own profile
Browse services
View service details
Manage own wishlist
Submit enquiries
View own enquiry history/status
Submit reviews
```

Cannot manage other users or administrative leads.

## ADMIN / STAFF

Can typically:

``` text
Access operational modules
Manage services/categories as permitted
View/manage enquiries
View/manage leads
Assign leads
Add follow-ups
Update lead status
View ML predictions
View permitted analytics
Moderate reviews
```

## SUPER_ADMIN

Can typically:

``` text
Manage users
Manage roles
Manage permissions
Manage system-wide configuration
Access all administrative modules
```

DRF permissions must enforce these rules on the server. Hiding a React
button is not security.

------------------------------------------------------------------------

# 14. React Developer Handoff

For each API I should give the React developer a specification such as:

``` text
Endpoint:
POST /api/enquiries/

Purpose:
Create a service enquiry.

Authentication:
Required.

Allowed role:
GENERAL_USER.

Content-Type:
application/json

Request:
{ ... }

Success:
201 Created

Validation failure:
400 Bad Request

Unauthenticated:
401 Unauthorized

Forbidden:
403 Forbidden

Response:
{ ... }
```

This should be maintained as an API contract so frontend and backend
development do not drift apart.

------------------------------------------------------------------------

# 15. CORS

Because the React application and Django backend may run on different
development origins, configure CORS for the actual frontend origins
agreed by the team.

For example, during local development this may include:

``` python
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
    "http://localhost:5173",
]
```

The exact production origin must be configured separately during
deployment.

------------------------------------------------------------------------

# 16. Django Admin Responsibilities

Register and customize appropriate models such as:

``` text
User
Category
Service
Enquiry
Lead
LeadFollowUp
Review
MLPrediction
```

Useful admin features:

``` text
list_display
search_fields
list_filter
ordering
readonly_fields
```

Django Admin is an internal administration interface; it does not
replace the REST APIs required by the React application.

------------------------------------------------------------------------

# 17. Validation and Security

Backend validation should cover areas such as:

``` text
Required fields
Valid email
Valid role
Valid service
Valid budget/value ranges
Valid lead status
Valid dates
Duplicate wishlist entries
Ownership checks
Admin-only operations
```

Also implement:

``` text
Password hashing through Django
Authentication
Authorization
Object-level ownership checks
CSRF handling where applicable
CORS configuration
Environment variables for secrets
No credentials/API keys committed to Git
Production DEBUG=False
Input validation
Safe error responses
```

------------------------------------------------------------------------

# 18. Postman Testing Responsibility

Before handing an endpoint to React, test it in Postman.

For each endpoint test:

``` text
1. Correct request
2. Missing required field
3. Invalid input
4. Unauthenticated request
5. Unauthorized role
6. Non-existing object
7. Successful database operation
8. Correct JSON structure
9. Correct HTTP status code
```

Important status codes:

``` text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
500 Internal Server Error
```

A 500 response during normal invalid user input generally indicates a
backend problem that should be fixed before handoff.

------------------------------------------------------------------------

# 19. Backend Testing

Important areas:

``` text
Authentication tests
Permission tests
Model tests
Serializer validation tests
API tests
Enquiry → Lead creation test
Lead follow-up tests
ML integration tests
Database relationship tests
Admin access tests
```

Critical integration test:

``` text
Create User
   ↓
Login
   ↓
Fetch Service
   ↓
Create Enquiry
   ↓
Verify Enquiry in DB
   ↓
Verify Lead created
   ↓
Admin accesses Lead
   ↓
Create Follow-up
   ↓
Request ML Prediction
   ↓
Verify Prediction response/storage
```

------------------------------------------------------------------------

# 20. Recommended Backend Implementation Order

``` text
Phase 1
Project setup
Django + DRF + PostgreSQL
Environment configuration
Custom User model

Phase 2
Authentication
Registration
Login/logout
Profile
Password change/reset
Roles and permissions

Phase 3
Category + Service models/APIs

Phase 4
Wishlist + Review

Phase 5
Enquiry
Enquiry tracking
Automatic Lead creation

Phase 6
Lead management
Assignment
Status changes
Follow-ups

Phase 7
ML prediction integration

Phase 8
Analytics APIs required by frontend

Phase 9
Django Admin customization

Phase 10
Postman + automated API testing

Phase 11
React integration and bug fixing

Phase 12
Deployment preparation
```

------------------------------------------------------------------------

# 21. Team Dependencies

## I receive from React/UI teams

``` text
Required screens
Form fields
Filtering requirements
Expected JSON structures
Frontend routes/use cases
```

## I provide to React

``` text
Base backend URL
API endpoints
Methods
Authentication method
Request JSON
Response JSON
Errors/status codes
Postman collection/documentation
```

## I coordinate with ML team on

``` text
Feature names
Feature types
Preprocessing
Model artifact/service interface
Expected prediction input
Expected prediction output
Model version
Failure handling
```

## I coordinate with database/project team on

``` text
Schema
Relationships
Constraints
Indexes
Migration strategy
Seed/demo data
```

------------------------------------------------------------------------

# 22. What Is Outside the Initial Scope

According to the project definition, the first version does not require:

``` text
Shopping cart
Checkout
Online payment
Shipping
Inventory
Order fulfilment
```

unless the project scope is later changed.

This distinction is important because SmartLead is a **service discovery
and lead management system**, not an online store.

------------------------------------------------------------------------

# 23. My Backend Definition of Done

My backend work is ready for handoff when:

``` text
[ ] Django project runs successfully
[ ] PostgreSQL connection works
[ ] Migrations work
[ ] Authentication works
[ ] Password reset/change flow works
[ ] Roles are implemented
[ ] Permissions are enforced server-side
[ ] Service APIs work
[ ] Enquiry APIs work
[ ] Enquiry correctly creates/links a Lead
[ ] Lead APIs work
[ ] Follow-up APIs work
[ ] Review/wishlist requirements work
[ ] ML prediction integration works
[ ] Django Admin works
[ ] Required analytics APIs work
[ ] CORS is configured
[ ] APIs are tested in Postman
[ ] Automated backend tests pass
[ ] React developer has API documentation
[ ] Secrets are stored in environment variables
[ ] Error responses are consistent
```

------------------------------------------------------------------------

# 24. The Most Important Part of My Role

My job is not simply to create Django pages.

I am building the **central application layer** connecting:

``` text
React
   ↕
REST APIs
   ↕
Django Business Logic
   ↕
PostgreSQL
   ↕
Lead Management
   ↕
ML Prediction
```

The most important business transaction is:

``` text
User submits enquiry
        ↓
Django validates it
        ↓
PostgreSQL stores it
        ↓
Lead is created
        ↓
Admin manages the lead
        ↓
ML predicts conversion probability
        ↓
Follow-up history is recorded
        ↓
Lead becomes Converted / Not Converted
```

If this workflow works correctly, the core SmartLead backend is working.

------------------------------------------------------------------------

# 25. Immediate Backend Starting Point

Before coding all modules, freeze these team decisions:

``` text
1. Exact User roles
2. Authentication mechanism (for example JWT/token strategy)
3. Exact fields for User, Service, Enquiry and Lead
4. Exact lead statuses
5. Whether enquiry automatically creates a lead
6. Admin vs staff permissions
7. Password-reset delivery mechanism
8. ML input/output contract
9. HIGH/MEDIUM/LOW prediction thresholds
10. React development and production origins
11. API response/error format
12. Which analytics are required in version 1
```

Then implement the project in the phase order above.

------------------------------------------------------------------------

## Final Mental Model

``` text
              SMARTLEAD
                  │
        ┌─────────┴─────────┐
        │                   │
   GENERAL USER          ADMIN
        │                   │
        ↓                   ↓
     REACT UI            REACT UI
        │                   │
        └─────────┬─────────┘
                  ↓
             DJANGO REST API
                  │
       ┌──────────┼───────────┐
       ↓          ↓           ↓
 Authentication Business    Permissions
              Logic
                  │
                  ↓
             PostgreSQL
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Enquiry      Lead     Follow-Up
                  │
                  ↓
            ML Prediction
                  │
                  ↓
          Admin Dashboard
```

**Backend ownership:** Django models, ORM/database interaction, business
logic, authentication, authorization, role-based permissions, REST APIs,
Django Admin, ML API/model integration, backend/API testing, and
technical API handoff to React.
