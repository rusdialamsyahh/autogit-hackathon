---
title: FastAPI CRUD Starter
app_type: fastapi-crud-starter
wallet: 0x8eAA683726787ff63b4Ba507B381a3c16e7a16c4
---

# what it is

A ready to fork FastAPI project that ships with one resource, one user, one auth flow, one test file, and nothing else. The README has six lines. The Dockerfile has eleven. You read every file in under five minutes and from that point on the project belongs to you, not to whoever wrote the boilerplate.

# why this shape

Most FastAPI starters drown the reader. They wire Alembic, Celery, Redis, Sentry, OpenTelemetry, JWT refresh rotation, role based access, and Pydantic settings layered three deep, all before the first endpoint exists. Then you spend an afternoon deleting things you do not need. This one assumes you can add what you need, and that adding is easier than deleting.

# the stack

Python 3.12. FastAPI 0.115. Pydantic 2.9 for request and response models. SQLAlchemy 2.0 async with aiosqlite for the dev database. Uvicorn for the dev server. Pytest with httpx AsyncClient for tests. Ruff and mypy in strict mode. No requirements.txt, only pyproject.toml. uv as the package manager.

# directory shape

```
app/
  main.py
  database.py
  models.py
  schemas.py
  auth.py
  routes/
    notes.py
    auth.py
tests/
  conftest.py
  test_notes.py
  test_auth.py
pyproject.toml
Dockerfile
README.md
.env.example
```

# the one resource

A note. Five fields. id is a uuid string. title is a string up to 200 chars. body is a string up to 10000 chars. created_at is a UTC datetime set on insert. owner_id is a foreign key to users. The schema mirrors the model exactly except id, created_at, and owner_id are read only and stripped from the create and update request models.

# endpoints

```
POST   /auth/register     create user, return 201 with bearer token
POST   /auth/login        verify password, return bearer token
GET    /auth/me           return current user

POST   /notes             create, returns 201 with the new note
GET    /notes             list current user notes, paginated by cursor
GET    /notes/{id}        single note, 404 if not yours
PATCH  /notes/{id}        partial update, 422 if any unknown field
DELETE /notes/{id}        soft delete, returns 204

GET    /health            returns {"ok": true, "version": app_version}
```

# auth flow

Passwords hashed with argon2 via passlib argon2_cffi. Tokens are JWT, HS256, ten minute expiry, no refresh. Subject claim is the user id. A single dependency reads the Authorization header, verifies the token, loads the user, returns the user. Every protected route depends on it. If the header is missing, 401 with a www authenticate hint. If invalid, 401. If user is deleted, 401.

# pagination

Cursor based, opaque base64 of the last seen created_at and id pair. Default limit 50, max 200. Response wraps the array in an object with items, next, prev. next is null when no more pages exist. This keeps the api stable as new notes arrive between requests.

# errors

A single exception handler turns any HTTPException into a json body shaped as `{"error": {"code": slug, "message": text}}`. Validation errors become 422 with a field array. Database integrity errors become 409. Anything else becomes 500 and logs the traceback. No stack traces leak to the client.

# tests

Three files. conftest.py spins up an in memory SQLite, runs migrations, yields an AsyncClient bound to the app. test_auth.py covers register, login, me, expired token, bad token, deleted user. test_notes.py covers happy path crud, isolation between users, pagination edges, partial update, soft delete idempotency. Tests run in under one second on a cold cache. Coverage report goes to the terminal, not html.

# config

A single Settings class subclasses BaseSettings. Reads from environment, falls back to .env. Five values. database url, secret key, token ttl seconds, debug flag, app version. No nested settings, no profiles, no overrides per environment. If you need staging versus prod, set the values from the platform you deploy on.

# logging

stdlib logging configured once in main.py at INFO. Format includes timestamp, level, logger name, message. A request id middleware adds a uuid per request, included in log records via a contextvar. No structlog, no loguru, no json formatter by default. Easy to add when you need it.

# the dev loop

```
uv sync
uv run uvicorn app.main:app --reload
uv run pytest
uv run ruff check
uv run mypy app
```

# the docker image

Single stage, python:3.12 slim base, uv installed via the official binary, dependencies installed from the pyproject lock file, app copied last so layer cache survives. CMD runs uvicorn bound to 0.0.0.0:8000 with one worker. No gunicorn, no nginx, no supervisor. Run multiple containers behind whatever load balancer your platform provides.

# what it does not do

No background jobs, no email, no rate limiting, no caching, no admin panel, no audit log, no password reset, no email verification, no oauth, no roles, no permissions, no opentelemetry, no metrics endpoint, no graceful shutdown beyond what uvicorn provides. Each of these is a half day project to add when you actually need it.

# the README

Six lines.

```
# project

set up: uv sync
run: uv run uvicorn app.main:app --reload
test: uv run pytest
docs: open http://localhost:8000/docs
deploy: docker build and push
```

The auto generated openapi at /docs is the rest of the documentation.
