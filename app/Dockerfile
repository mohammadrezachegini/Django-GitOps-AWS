# ─────────────────────────────────────────────
# Stage 1: builder
# Install dependencies into a separate layer.
# This keeps the final image clean and small.
# ─────────────────────────────────────────────
FROM python:3.11-slim AS builder

WORKDIR /app

# Install only what's needed to compile Python packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy and install Python dependencies
COPY requirements.txt .
RUN pip install --upgrade pip \
    && pip install --prefix=/install --no-cache-dir -r requirements.txt


# ─────────────────────────────────────────────
# Stage 2: runtime
# Copy only the installed packages and app code.
# No build tools — smaller and more secure.
# ─────────────────────────────────────────────
FROM python:3.11-slim AS runtime

WORKDIR /app

# Runtime system dependencies only (psycopg2 needs libpq)
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 \
    && rm -rf /var/lib/apt/lists/*

# Copy installed packages from the builder stage
COPY --from=builder /install /usr/local

# Copy the Django source code
COPY src/ .

# Create a non-root user — running as root is a security risk
RUN useradd --no-create-home --shell /bin/false appuser
USER appuser

# Gunicorn will serve the app on this port
EXPOSE 8000

# manage.py lives at /app/django_app/manage.py
# Gunicorn looks for config.wsgi inside the django_app/ package
# Copy and permission the entrypoint BEFORE switching to non-root user

COPY --chmod=755 entrypoint.sh /entrypoint.sh

# THEN create and switch to non-root user

ENTRYPOINT ["/entrypoint.sh"]