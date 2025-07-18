# Phase One Technical Build Plan
This technical plan outlines how to build out the new healthlab repository in Phase One. The goal is to implement the core architecture, initial domains (Scalp and Facial Skin) and infrastructure so that additional domains can be added in later phases. The plan is organized by directory, specifying the files to create, their responsibilities, and the Definition of Done (DoD) criteria for each component.

## 0 Prerequisites
The full documentation folder (with domain specifications) should be copied into healthlab/docs/ before development begins.

Set up Python (e.g., poetry) and Node.js/Expo environments on the development machine.

Acquire necessary API keys (e.g., for timeseries DB, external food APIs) and store them as environment variables.

## 1 Repository Layout
Create the following top‑level directory structure:

```bash
healthlab/
├── apps/
│   ├── api/                    # FastAPI application (server)
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── domains/
│   │   │   ├── scalp/
│   │   │   ├── facial_skin/
│   │   │   ├── body_skin/      # placeholder for phase two
│   │   │   ├── hypertrophy/    # placeholder for phase two
│   │   │   ├── strength/       # placeholder for phase two
│   │   │   ├── nutrition/      # placeholder for phase two
│   │   │   ├── cardio/         # placeholder for phase two
│   │   │   ├── sleep/          # placeholder for phase two
│   │   │   └── time_tracking/  # placeholder for phase two
│   │   ├── scheduling/
│   │   └── automations/
│   └── mobile/                # React Native (Expo) app
├── packages/
│   └── domain_core/
├── docs/
├── tests/
├── infrastructure/
└── pyproject.toml
```
Use the same layout for later phases (nutrition, cardio, etc.). For Phase One, you will implement only the scalp and facial_skin domains; the other domain directories can remain empty with an __init__.py file.

## 2 apps/api – FastAPI Server
## 2.1 main.py
Purpose: Entry point for the FastAPI application.

Tasks:

Import FastAPI, instantiate the app, and load configuration.

Include routers from each domain that is implemented in Phase One (scalp and facial_skin).

Provide the OpenAPI documentation route /docs (FastAPI’s built‑in Swagger UI).

Optionally include middleware for CORS, authentication, and error handling.

Definition of Done:

The server starts via uvicorn apps.api.main:app without runtime errors.

The API root returns a JSON message confirming the service is running.

The OpenAPI docs page loads and shows the available endpoints.

## 2.2 config.py
Purpose: Centralize configuration (environment variables, database connection string, API keys).

Tasks:

Use Pydantic’s BaseSettings to define a Settings class with fields for database URL, secret keys, and environment.

Provide a function get_settings() that returns a cached instance of Settings.

Definition of Done:

Environment variables are loaded from .env or system environment.

Importing get_settings() returns a Settings instance with all required attributes.

Unit tests verify that missing environment variables raise helpful errors.

## 2.3 Domain Router Registration
For each implemented domain, import its router and include it in the FastAPI app:

```python
from .domains.scalp.router import router as scalp_router
from .domains.facial_skin.router import router as facial_skin_router

app.include_router(scalp_router, prefix="/scalp")
app.include_router(facial_skin_router, prefix="/facial-skin")
```
Definition of Done:

Requests to /scalp/events and /facial-skin/events return valid responses.

## 3 packages/domain_core – Shared Models and Utilities
## 3.1 Enums and BaseModel
Tasks:

Create enums.py containing enums for domain names and generic event types.

Create base.py with an abstract DomainBaseModel class that extends pydantic.BaseModel and adds id: int | None, created_at and updated_at timestamps.

Definition of Done:

The enums list all nine domain names (scalp, facial_skin, body_skin, hypertrophy, strength, nutrition, cardio, sleep, time_tracking).

The DomainBaseModel class is importable and sets default timestamps on instantiation.

Unit tests in tests/domain_core/test_base.py assert that the created_at field is populated automatically.

## 4 Domains (Phase One) – scalp and facial_skin
Each domain folder will follow the Domain‑Driven Design structure
:

```bash
api/domains/<domain>/
├── __init__.py
├── models.py
├── service.py
├── domain_api.py   # optional – defines interfaces to other domains
└── router.py
## 4.1 scalp/
models.py
Define SQLModel (or SQLAlchemy/SQLModel) models and Pydantic schemas:

ScalpEvent: event table with fields such as id, timestamp, event_type (enum: wash, conditioner, serum, microneedle), details (JSON/dict), and user_id.

ProductBatch: table for hair products with fields id, product_kind, volume_ml, concentration, lot_number.

ScalpMeasurement: table for measurement data like hair mass index (HMI), shed counts, etc.

Pydantic schemas for request/response models (e.g., ScalpEventCreate, ScalpEventRead).

service.py
Implement business logic:

Validate events against blackout rules (e.g., 24 h after microneedling you cannot schedule another microneedling
).

Provide methods to create, update, and retrieve events and measurements.

Handle cross‑domain checks if necessary (in Phase One there are no cross‑domain interactions).

router.py
Define FastAPI endpoints:

POST /events – create an event (calls the service layer).

GET /events/{id} – retrieve an event by ID.

GET /events – list events with optional filters (date range, event type).

POST /measurements – create measurement data.

GET /measurements – retrieve measurement data.

Definition of Done (Scalp):

Running Alembic migrations creates the scalp tables.

POST /scalp/events stores events in the database and returns 201 Created.

Blackout logic rejects invalid events (tested via unit tests).

GET /scalp/measurements returns measurement records.

Unit tests cover model serialization, service validation, and router endpoints.

Documentation in docs/scalp_project.md exists and is referenced from the README.

## 4.2 facial_skin/
Follow the same pattern as the scalp domain, adapting fields and rules:

models.py
FacialSkinEvent with event types: wash, vitamin C serum, moisturizer, TAAN, etc.

FacialSkinMeasurement with TEWL, sebum, lesion counts.

service.py
Implement barrier_guard logic to block TAAN when TEWL is elevated
.

router.py
Mirror scalp endpoints for events and measurements.

Definition of Done (Facial Skin):

Alembic migrations create facial‑skin tables.

API endpoints create and list events and measurements.

TEWL‐based blackout rules prevent TAAN events when the barrier is compromised (tested via unit tests).

Unit tests cover service logic and endpoints.

## 5 Scheduling Engine – apps/api/scheduling
## 5.1 schedule_service.py
Purpose: Provide functions for generating domain schedules.

Tasks:

Create a ScheduleService class with methods generate_scalp_schedule() and generate_facial_skin_schedule() that output future events based on the execution grids defined in the documentation.

Encode blackout rules into scheduling (skip days after microneedling; skip TAAN when TEWL is high).

Provide helper functions to detect conflicts and reschedule.

Definition of Done:

A call to generate_scalp_schedule(start_date, end_date) returns a list of event dictionaries covering the specified period with no rule violations.

Unit tests verify that schedules respect blackout periods.

Additional domains can be plugged in later via similar methods.

## 5.2 Scheduling API
Implement a router scheduling/router.py:

GET /schedule/{domain} – return the upcoming schedule for the given domain within a date range.

Definition of Done:

The scheduling API returns a 200 OK response with JSON schedule for scalp and facial‑skin domains.

Unit tests cover schedule endpoint and ensure correct schedule generation.

## 6 Automations – apps/api/automations
In Phase One, this folder can remain a placeholder. Create an __init__.py and optionally a stub Celery configuration file for future use.

Definition of Done:

Directory exists with stub files and is ready for future Celery tasks.

## 7 apps/mobile – React Native App
## 7.1 Project Setup
Tasks:

Initialize an Expo project (expo init healthlab-mobile) in apps/mobile.

Configure TypeScript and add a basic tab navigation with two screens: Scalp and Facial Skin.

For Phase One, create simple forms to log scalp and facial‑skin events; connect them to the API using fetch/axios.

Add a schedule view that calls the scheduling API and displays upcoming routines.

Definition of Done:

The mobile app compiles in Expo and runs on iOS/Android simulators.

Users can log scalp and facial‑skin events from the app; events are posted to the API and persisted.

The schedule view fetches and displays upcoming routines for scalp and facial‑skin.

## 8 tests – Unit and Integration Tests
## 8.1 Testing Setup
Tasks:

Install pytest and pytest-asyncio for asynchronous testing.

Create conftest.py with fixtures for FastAPI TestClient and database session.

Write tests for domain models, services, routers, and scheduling service.

Definition of Done:

Running pytest yields 0 failures for Phase One components.

Coverage reports show that at least 80% of scalp, facial_skin, and scheduling modules are tested.

## 9 infrastructure – Deployment Scripts
## 9.1 Local Deployment
Tasks:

Write a docker-compose.yml that spins up the API server, PostgreSQL with TimescaleDB, and an optional Grafana instance.

Provide a .env file template with placeholders for environment variables.

## 9.2 Production Infrastructure (Phase One Placeholder)
Include Terraform/CloudFormation/CDK skeleton files for future production deployment (RDS, ECS/EKS), but actual deployment will occur in later phases.

Definition of Done:

docker-compose up starts all services; the API is reachable at localhost:8000.

The database service includes TimescaleDB extension enabled.

## 10 docs – Documentation
## 10.1 Documentation Import
Tasks:

Copy the user’s existing documentation files into the docs/ directory.

Create mkdocs.yml to configure MkDocs with a custom theme and navigation structure.

For Phase One, ensure at least two domain docs are accessible: scalp_project.md and facial_skin_project.md. Add high‑level pages such as Introduction, Architecture, and API reference.

Definition of Done:

Running mkdocs serve renders the docs site locally; pages load without broken links.

The documentation includes the imported domain specifications and new architecture description.

## 11 Integration and Handover
## 11.1 README
Provide a top‑level README.md with:

Overview of the project and goals.

Setup instructions (Python, Node.js, environment variables).

How to run the API server locally and in Docker.

How to run tests and build the mobile app.

Links to documentation.

Definition of Done:

A developer following the README can clone the repo, install dependencies, set environment variables, run the API server and mobile app, and execute tests without additional guidance.

## 11.2 Code Quality and Definition of Done
This plan implements the DDD principle of encapsulating code within domain modules and minimizing global logic
. The definitions of done ensure each increment meets agreed‑upon criteria and is ready for integration
. For each module, code must compile, tests must pass, and documentation must be in place. Additionally, all features implemented in Phase One must work together end‑to‑end: events logged from the mobile app appear in the API and database, schedules are generated correctly, and the documentation reflects the implemented behavior.



This technical plan should give Codex a precise blueprint for scaffolding the repository, implementing the initial domains (Scalp and Facial Skin), setting up scheduling, database migrations, documentation, testing, and a basic mobile interface. Feel free to let me know if you need further adjustments or more detail on any specific component

