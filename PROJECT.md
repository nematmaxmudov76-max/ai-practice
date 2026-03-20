# Django Backend Engineer — Execution Plan

## Stage 1 — Foundations: Data Modeling, Admin, Core CRUD

### Task 1: Project Bootstrap with Environment Discipline
**Problem (real-world confusion):** “It works on my machine” because settings and dependencies are inconsistent across environments.
**Task instructions (what to build):**
- Create a new Django project with a `config` settings package and environment-based settings (local/staging/prod).
- Use `.env` for secrets and `python-decouple` or `django-environ` for configuration.
- Add `pre-commit` with formatting and linting (e.g., `black`, `isort`, `ruff`).
**Expected mistakes:**
- Committing secrets to git.
- Hardcoding DEBUG and database credentials.
- Mixing local and production settings.
**Solution (code or approach):**
- Split settings into `base.py`, `local.py`, `prod.py`.
- Load secrets with environment variables; fail fast if missing.
- Configure `ALLOWED_HOSTS`, `CSRF_TRUSTED_ORIGINS`, and `DATABASES` via env.
**Why it works (deep explanation, trade-offs):**
- Environment-specific configuration prevents accidental production misconfigurations.
- `.env` improves local developer experience but must never be used for production secrets.
- Trade-off: Slight complexity up front, but reduces operational risk long-term.

### Task 2: Blog Models + Admin Workflow
**Problem (real-world confusion):** Junior devs often misuse Django ORM fields and skip constraints, leading to data integrity bugs.
**Task instructions (what to build):**
- Build a `Blog` app with `Post`, `Category`, and `Tag` models.
- Add constraints (unique slugs, non-null fields) and indexes on commonly filtered fields.
- Customize admin with list displays, filters, and search.
**Expected mistakes:**
- Forgetting to create a custom `__str__`.
- No indexes on `slug` or `published_at`.
- Overusing `TextField` where `CharField` is appropriate.
**Solution (code or approach):**
- Use `SlugField(unique=True, db_index=True)`.
- Add `published_at = DateTimeField(db_index=True, null=True, blank=True)`.
- Customize admin with `list_display`, `list_filter`, `search_fields`.
**Why it works (deep explanation, trade-offs):**
- Constraints enforce correctness at the DB layer, which is your last line of defense.
- Indexes improve read performance but increase write cost and storage. Use only where reads dominate.
- Admin customization makes back-office workflows reliable and efficient.

### Task 3: CRUD Views with Form Validation
**Problem (real-world confusion):** “My forms save invalid data” because validation is skipped or handled in the view.
**Task instructions (what to build):**
- Implement Create/Update/Delete views with Django forms for `Post`.
- Add custom validation (e.g., title length, slug format).
- Provide success messages and redirect patterns.
**Expected mistakes:**
- Putting validation directly in views.
- Forgetting to call `form.is_valid()`.
- Not handling `IntegrityError` for unique constraints.
**Solution (code or approach):**
- Use `ModelForm` with `clean_<field>` methods.
- Add `unique=True` at model-level and handle DB errors in view.
- Use class-based views (`CreateView`, `UpdateView`) with `form_valid` overrides.
**Why it works (deep explanation, trade-offs):**
- Validation in forms centralizes business rules and keeps views clean.
- DB constraints guarantee correctness even if validation is bypassed.
- Class-based views reduce boilerplate but add indirection; know when to override.

## Stage 2 — Authentication & Authorization

### Task 1: Custom User Model + Registration
**Problem (real-world confusion):** Swapping user models late causes migration pain and broken auth flows.
**Task instructions (what to build):**
- Create a custom user model with email login.
- Implement registration, login, password reset, and email verification.
**Expected mistakes:**
- Creating a custom user model after initial migrations.
- Forgetting to update `AUTH_USER_MODEL`.
- Using username when email is required.
**Solution (code or approach):**
- Create a `users` app and custom `AbstractBaseUser` or `AbstractUser`.
- Set `AUTH_USER_MODEL = "users.User"` in settings before migrations.
- Use `django-allauth` or a custom flow with email verification.
**Why it works (deep explanation, trade-offs):**
- Early user-model customization avoids painful migrations later.
- Email-based auth aligns with modern systems.
- Trade-off: More setup complexity vs. long-term flexibility.

### Task 2: Role-Based Access Control (RBAC)
**Problem (real-world confusion):** “I checked login, but users can still access admin actions.”
**Task instructions (what to build):**
- Define roles (`admin`, `editor`, `viewer`) and permissions for `Post` actions.
- Enforce permissions in views and DRF endpoints.
**Expected mistakes:**
- Using `is_staff` or `is_superuser` for everything.
- Forgetting to restrict update/delete.
- Hardcoding roles without extensibility.
**Solution (code or approach):**
- Use Django groups and permissions.
- Add `@permission_required` or custom permission classes for DRF.
- Store role metadata in the database.
**Why it works (deep explanation, trade-offs):**
- RBAC is a scalable approach for permissions in production.
- Group-based permissions separate authentication from authorization.
- Trade-off: More moving parts but future-proofed access control.

## Stage 3 — REST APIs with DRF

### Task 1: API for Posts with Pagination/Filtering
**Problem (real-world confusion):** “My API returns everything and gets slow.”
**Task instructions (what to build):**
- Build DRF endpoints for `Post` with pagination, filtering, and ordering.
- Add validation and permissions.
**Expected mistakes:**
- Returning unpaginated lists.
- No filtering or ordering.
- Missing serializer validation.
**Solution (code or approach):**
- Use `ModelViewSet` with `PageNumberPagination`.
- Add `django-filter` for filtering and `OrderingFilter`.
- Validate required fields in serializer.
**Why it works (deep explanation, trade-offs):**
- Pagination reduces load and improves response time.
- Filtering/ordering keeps logic in the API layer and out of the UI.
- Trade-off: More configuration but essential for real APIs.

### Task 2: API Versioning + Schema Docs
**Problem (real-world confusion):** “We changed a response and broke clients.”
**Task instructions (what to build):**
- Add API versioning using URL namespaces.
- Generate OpenAPI docs with `drf-spectacular`.
**Expected mistakes:**
- Changing serializers without versioning.
- No schema docs for frontend/clients.
**Solution (code or approach):**
- Create `api/v1/` routes and versioned serializers.
- Add schema endpoint and generated docs.
**Why it works (deep explanation, trade-offs):**
- Versioning protects clients from breaking changes.
- OpenAPI docs standardize communication across teams.

## Stage 4 — Async Work & Background Jobs

### Task 1: Email Queue with Celery
**Problem (real-world confusion):** “User signup is slow because email sending blocks the request.”
**Task instructions (what to build):**
- Configure Celery with Redis.
- Queue welcome email tasks.
- Add retries and logging.
**Expected mistakes:**
- Running tasks synchronously.
- No retry strategy.
- Not handling idempotency.
**Solution (code or approach):**
- Use `@shared_task` with `autoretry_for` and `max_retries`.
- Store task metadata and ensure task is idempotent.
**Why it works (deep explanation, trade-offs):**
- Async tasks improve response times and reliability.
- Retries prevent transient failures from breaking workflows.
- Trade-off: Adds infrastructure complexity (Redis + workers).

### Task 2: Periodic Cleanup Job
**Problem (real-world confusion):** “Old data piles up and slows the DB.”
**Task instructions (what to build):**
- Add a periodic Celery beat task to clean expired drafts.
**Expected mistakes:**
- Deleting without confirmation or logs.
- Running heavy deletes during peak hours.
**Solution (code or approach):**
- Use Celery beat with off-peak schedule.
- Soft-delete or archive first.
**Why it works (deep explanation, trade-offs):**
- Scheduled maintenance keeps data size controlled.
- Trade-off: Soft-deletes add complexity but reduce risk of data loss.

## Stage 5 — Caching & Performance

### Task 1: Cache Post List API
**Problem (real-world confusion):** “Popular endpoints are slow and expensive.”
**Task instructions (what to build):**
- Add per-view caching to `Post` list endpoint.
- Invalidate cache on create/update/delete.
**Expected mistakes:**
- Caching without invalidation.
- Over-caching user-specific data.
**Solution (code or approach):**
- Use `cache_page` with timeout for public list endpoints.
- Clear keys in model signals for writes.
**Why it works (deep explanation, trade-offs):**
- Caching reduces DB load and improves latency.
- Trade-off: Invalidation complexity and stale data risk.

### Task 2: ORM Optimization
**Problem (real-world confusion):** “N+1 queries in production.”
**Task instructions (what to build):**
- Identify N+1 query in post list.
- Add `select_related` and `prefetch_related`.
**Expected mistakes:**
- Overusing `select_related` for many-to-many.
- Fetching too much data.
**Solution (code or approach):**
- Use `select_related` for FK, `prefetch_related` for M2M.
- Limit fields with `.only()` or `.values()` if needed.
**Why it works (deep explanation, trade-offs):**
- Reduces DB round-trips and speeds responses.
- Trade-off: Larger single queries, careful tuning needed.

## Stage 6 — File Handling & External Integrations

### Task 1: User Uploads with S3
**Problem (real-world confusion):** “Files work locally but fail in production.”
**Task instructions (what to build):**
- Configure S3 storage with `django-storages`.
- Enable direct upload with signed URLs.
**Expected mistakes:**
- Storing files on local disk in production.
- Leaking credentials.
**Solution (code or approach):**
- Use S3 backend and IAM credentials.
- Generate signed URLs for upload.
**Why it works (deep explanation, trade-offs):**
- Offloads file handling from the app server.
- Trade-off: Added dependency on AWS infrastructure.

### Task 2: Webhook Handler
**Problem (real-world confusion):** “Webhook retries created duplicate records.”
**Task instructions (what to build):**
- Build a webhook endpoint that validates signatures.
- Ensure idempotency and safe retries.
**Expected mistakes:**
- No signature validation.
- No idempotency keys.
**Solution (code or approach):**
- Validate signatures with shared secret.
- Store processed event IDs and ignore duplicates.
**Why it works (deep explanation, trade-offs):**
- Prevents forged requests and duplicate processing.
- Trade-off: Requires extra storage for event history.

## Stage 7 — Observability & Reliability

### Task 1: Structured Logging + Error Monitoring
**Problem (real-world confusion):** “We can’t debug production issues.”
**Task instructions (what to build):**
- Add structured logging (JSON) and integrate Sentry.
**Expected mistakes:**
- Logging sensitive data.
- Using plain text logs.
**Solution (code or approach):**
- Configure logging formatters with JSON.
- Add Sentry SDK with environment tagging.
**Why it works (deep explanation, trade-offs):**
- Structured logs enable filtering and correlation.
- Trade-off: Slight overhead and privacy considerations.

### Task 2: Health Checks + Readiness
**Problem (real-world confusion):** “Deployments break because services aren’t ready.”
**Task instructions (what to build):**
- Add `/healthz` and `/readyz` endpoints for DB and cache checks.
**Expected mistakes:**
- Returning 200 even when DB is down.
- Doing expensive checks on every request.
**Solution (code or approach):**
- Use light queries for readiness.
- Cache readiness results briefly.
**Why it works (deep explanation, trade-offs):**
- Helps orchestration tools route traffic correctly.
- Trade-off: Adds endpoints that must be secured.

## Stage 8 — Deployment & Operations

### Task 1: Dockerize the App
**Problem (real-world confusion):** “Production behaves differently than local dev.”
**Task instructions (what to build):**
- Create a Dockerfile and docker-compose for app, DB, Redis, Celery.
- Ensure environment variables are passed correctly.
**Expected mistakes:**
- Baking secrets into the image.
- Not pinning dependencies.
**Solution (code or approach):**
- Use multi-stage Dockerfile with `pip-compile` lock.
- Configure `docker-compose` for local parity.
**Why it works (deep explanation, trade-offs):**
- Containers ensure runtime consistency.
- Trade-off: Steeper learning curve but essential for production.

### Task 2: CI/CD Pipeline
**Problem (real-world confusion):** “Migrations and tests aren’t running consistently.”
**Task instructions (what to build):**
- Set up CI pipeline to run tests and lint.
- Add deployment step with migrations.
**Expected mistakes:**
- Running migrations manually.
- Skipping tests in CI.
**Solution (code or approach):**
- Use GitHub Actions or GitLab CI.
- Add migration command in release step.
**Why it works (deep explanation, trade-offs):**
- Automated pipelines reduce deployment risk.
- Trade-off: Requires time to set up and maintain.

## Stage 9 — Capstone: Production-Ready Service

### Task 1: Build a SaaS-Style Blog Platform
**Problem (real-world confusion):** “Projects lack end-to-end production rigor.”
**Task instructions (what to build):**
- Combine auth, APIs, async jobs, caching, uploads, and deployment.
- Add multi-tenant support (organization-based).
**Expected mistakes:**
- Mixing tenant data.
- Missing tenant checks in queries.
**Solution (code or approach):**
- Add `Organization` model and tenant scoping in queries.
- Enforce tenant filtering in permissions and managers.
**Why it works (deep explanation, trade-offs):**
- Multi-tenant design mirrors real SaaS systems.
- Trade-off: Increased complexity and higher testing burden.

### Task 2: Production Readiness Checklist
**Problem (real-world confusion):** “We launched without monitoring or rollback.”
**Task instructions (what to build):**
- Create a checklist covering security, backups, monitoring, and rollback.
**Expected mistakes:**
- No backup strategy.
- Lack of observability.
**Solution (code or approach):**
- Document backup strategy and restore drill.
- Define SLOs and alerts.
**Why it works (deep explanation, trade-offs):**
- Reduces incident impact and downtime.
- Trade-off: More upfront planning.
