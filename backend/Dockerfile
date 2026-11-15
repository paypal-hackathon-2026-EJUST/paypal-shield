# Server for the FastAPI core (Person 1's app lives in backend/app).
# Owned by Person 4. Python 3.11 slim, non-root, explicit uvicorn entrypoint.
FROM python:3.11-slim AS base

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

WORKDIR /app

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt \
    && chown 1000:1000 /app

# uid 1000 owns the code and the workdir so runtime writes (uploads, caches)
# succeed while everything still installs as root.
COPY --chown=1000:1000 app ./app
# alembic.ini sits at the image workdir root so migrations can be invoked
# flag-free (`alembic upgrade head`) from CWD=/app - no fragile -c paths in
# boot commands or platform pre-deploy hooks.
COPY --chown=1000:1000 app/db/alembic.ini ./alembic.ini

USER 1000

EXPOSE 8000

# Sandbox guard: refuses to boot against a non-Sandbox PayPal base URL unless
# ALLOW_NON_SANDBOX=true is explicitly set by the operator.
# Migrations run on boot (idempotent) so a fresh Postgres volume gets the
# 9 canonical tables before the API binds its port.
# ENTRYPOINT (not CMD) makes the boot sequence immune to a platform-level
# "Docker Command" override: any injected argv lands as ignored positional
# params of `sh -c` instead of replacing the migration+server invocation.
ENTRYPOINT ["sh", "-c", "alembic upgrade head && exec python -m uvicorn app.main:app --host 0.0.0.0 --port 8000"]
CMD []
