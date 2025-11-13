

# ===================================
# STAGE 1: Base
# ===================================
FROM python:3.14-slim AS base

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /app

# System dependencies with cache
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        gcc \
        g++ \
        libpq-dev \
        curl && \
    rm -rf /var/lib/apt/lists/*

# ===================================
# STAGE 2: Dependencies
# ===================================
FROM base AS dependencies

# Copy requirements FIRST (for layer caching)
COPY requirements.txt .

# Install Python packages with pip cache
RUN --mount=type=cache,target=/root/.cache/pip,sharing=locked \
    pip install --upgrade pip setuptools wheel && \
    pip install -r requirements.txt

# ===================================
# STAGE 3: Development
# ===================================
FROM dependencies AS development

# Copy application code LAST
COPY . .

RUN mkdir -p /app/uploads

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]

# ===================================
# STAGE 4: Production
# ===================================
FROM dependencies AS production

COPY . .

RUN mkdir -p /app/uploads && \
    useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app

USER appuser

EXPOSE 8000

CMD ["gunicorn", "app.main:app", \
     "--workers", "4", \
     "--worker-class", "uvicorn.workers.UvicornWorker", \
     "--bind", "0.0.0.0:8000"]