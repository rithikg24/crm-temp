# CRM System – Technical Documentation

## Overview
This document provides a comprehensive technical guide for the CRM system, including architecture, system design, data model, API and web routes, UI views, configuration, testing, operations, troubleshooting, and a user manual. It targets developers, QA, and operators.

## Architecture
- **Framework**: Spring Boot (Java)
- **Layers**:
  - **Controller layer** (`com.example.crm.controller`): Handles web requests (Thymeleaf MVC views) and JSON analytics APIs.
  - **Service layer** (`com.example.crm.service`): Business logic and transactional operations.
  - **Repository layer** (`com.example.crm.repository`): Spring Data JPA repositories for persistence.
  - **Entity layer** (`com.example.crm.entity`): JPA entities for `Customer`, `Lead`, and `Interaction`.
  - **Views** (`src/main/resources/templates`): Thymeleaf HTML templates.
  - **Database**: H2 in-memory DB configured via `application.properties`, schema defined in `schema.sql`, data loaded from `data.sql`.

### High-Level Data Flow
1. Browser requests a page (e.g., `/customers`).
2. Controller queries the service; service uses repositories to load data.
3. Model is rendered with a Thymeleaf template.
4. Mutations (POST/DELETE) go controller → service → repository → DB.
5. Analytics endpoints return JSON for charts on `/analytics`.

## Data Model
Entities mapped by JPA and supported by `schema.sql`.

### Customer
- Fields: `id` (PK), `name`, `emailId` (column: `email_id`), `contactNumber` (column: `contact_number`), `address`, `customerType` (column: `customer_type`).
- Relationships: Referenced by `Lead.customerId` and `Interaction.customerId`.

### Lead
- Fields: `id`, `customerId` (FK → `customer.id`), `source`, `status`, `topic`, `notes`.
- Cascade delete: When a `Customer` is deleted, leads are removed via FK `ON DELETE CASCADE`.

### Interaction
- Fields: `id`, `time` (timestamp), `type`, `topic`, `notes`, `customerId` (FK → `customer.id`).
- Service ensures `time` is set if null on creation.

### SQL Schema (excerpt)
- Defined in `src/main/resources/schema.sql` with proper creation order and `ON DELETE CASCADE` on FKs.

## Services
- `CustomerService`: CRUD and transactional delete with existence check.
- `LeadService`: CRUD, find by customer, delete by customer (transactional).
- `InteractionService`: Create with timestamp normalization, query by customer, delete by customer (transactional).

## Controllers and Routes
All HTML views are server-rendered with Thymeleaf unless noted as JSON.

### Dashboard
- `GET /` → `dashboard.html`.

### Customers
- `GET /customers` → list customers (`customers-list.html`).
- `GET /customers/add` → add form (`add-customer.html`).
- `GET /customers/{id}/edit` → edit form (`update-customer.html`).
- `POST /customers` → create customer.
- `POST /customers/{id}` → update customer.
- `DELETE /customers/{id}` → delete customer.

### Leads
- `GET /leads` → list leads (`leads-list.html`).
- `GET /leads/add` → add form (`add-lead.html`).
- `GET /leads/{id}/edit` → edit form (`update-lead.html`).
- `POST /leads` → create lead (with selected customer).
- `POST /leads/{id}` → update lead.
- `DELETE /leads/{id}` → delete lead.

### Interactions
- `GET /interactions/customers/{id}` → list interactions for customer (`customer-interactions-list.html`).
- `GET /interactions/customer/{id}/add` → add form (`add-interaction.html`).
- `POST /interactions/customers/{id}` → create interaction for customer.

### Analytics
- `GET /analytics` → analytics dashboard (`analytics.html`).
- JSON APIs (used by charts):
  - `GET /api/analytics/customers-over-time`
  - `GET /api/analytics/leads-over-time`
  - `GET /api/analytics/customer-types`
  - `GET /api/analytics/lead-status`
  - `GET /api/analytics/lead-sources`

## Views and UX
Templates are in `src/main/resources/templates/`. Key pages: `dashboard.html`, `customers-list.html`, `add-customer.html`, `update-customer.html`, `leads-list.html`, `add-lead.html`, `update-lead.html`, `customer-interactions-list.html`, `add-interaction.html`, `analytics.html`, `error.html`.

## Configuration
`src/main/resources/application.properties`:
- `spring.datasource.url=jdbc:h2:mem:test`
- `spring.jpa.show-sql=true`
- `spring.jpa.hibernate.ddl-auto=none`
- `spring.jpa.properties.hibernate.format_sql=true`
- `spring.jpa.properties.hibernate.jdbc.time_zone=UTC`

Database scripts:
- `schema.sql`: Defines tables and FKs with cascades.
- `data.sql`: Seeds initial data (optional in test/dev).

## Build & Run
- Prerequisites: Java 17+, Maven.
- Run app: `mvn spring-boot:run`
- Run tests: `mvn test`
- App runs on default port 8080 unless overridden.

## User Manual
### Accessing the application
- Start the app and navigate to `http://localhost:8080`.

### Managing Customers
- View all: `Customers` link → table of customers.
- Add: `Add Customer` → fill form → submit.
- Edit: on list, click `Edit` → modify → submit.
- Delete: on list, click `Delete` (uses HTTP DELETE via form or JS).

### Managing Leads
- View all leads via `Leads` nav.
- Add: choose a customer, set `source`, `status`, `topic`, `notes` → save.
- Edit or Delete similarly via actions on list.

### Managing Interactions
- From a specific customer page: view interactions.
- Add interaction: set `type`, `topic`, `notes`; timestamp is auto-set if omitted.

### Analytics
- Open `Analytics` to view charts. Data loads from `/api/analytics/*` endpoints.

## Error Handling & Constraints
- Deleting a customer cascades to delete dependent leads and interactions (DB FK with `ON DELETE CASCADE`). Services also provide helper bulk deletes by customer ID when necessary.
- Controllers guard against missing IDs by redirecting or returning `error.html` as appropriate.

## Security
- No authentication/authorization is configured. If needed, integrate Spring Security (form login) and scope analytics APIs appropriately.

## Testing
- Unit tests in `src/test/java/com/example/crm/service/` using JUnit 5 + Mockito.
- Integration tests in `src/test/java/com/example/crm/controller/` validating end-to-end flows.
- See `TESTING.md` for commands and structure.

## Operations
- Startup: verify tables created from `schema.sql`, confirm log shows SQL and successful initialization. H2 is ephemeral; use external DB for production.
- Configuration profiles: add `application-*.properties` for different environments as needed.
- Logs: default console; consider `logback-spring.xml` for production logging.

## Troubleshooting
- Schema not applied: ensure `spring.jpa.hibernate.ddl-auto=none` and `schema.sql` is on classpath; clean `target/` and rerun.
- Delete FK errors: ensure you use provided DELETE endpoints; DB is configured with `ON DELETE CASCADE`.
- Timestamp null on interactions: service sets it; ensure controller passes entity without `id`.

## Roadmap (Suggestions)
- Add authentication and role-based access.
- Persist to a production-grade database (PostgreSQL/MySQL) with migrations (Flyway/Liquibase).
- Add REST APIs for all CRUD operations in addition to Thymeleaf views.
- Add validation annotations and server-side error messages.
- Enhance analytics with more dimensions and caching.

## PDF Export (Windows-Friendly)
Choose one of the following:

1) VS Code/Editor Extension
- Open `docs/TECHNICAL_DOCUMENTATION.md`.
- Use a Markdown PDF extension (e.g., "Markdown PDF").
- Command: Export (cmd palette) → "Markdown PDF: Export (pdf)".

2) Pandoc + wkhtmltopdf (CLI)
- Install Pandoc and wkhtmltopdf.
- In project root:
  - Windows CMD:
    ```bat
    pandoc docs/TECHNICAL_DOCUMENTATION.md -o docs/TECHNICAL_DOCUMENTATION.pdf --from gfm --pdf-engine=wkhtmltopdf
    ```

3) Chrome/Edge Print to PDF
- Open the Markdown preview in your IDE or a rendered view.
- Print (Ctrl+P) → Destination: Save as PDF → Save.

---

For updates, maintain this single source of truth in `docs/TECHNICAL_DOCUMENTATION.md`. Ensure endpoints and schema snippets stay in sync with controllers and `schema.sql`.
