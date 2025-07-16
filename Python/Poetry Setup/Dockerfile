# Python base with shared environment settings
FROM python:3.13-slim AS python-base

# Set environment variables
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=off \
    PIP_DISABLE_PIP_VERSION_CHECK=on \
    PIP_DEFAULT_TIMEOUT=100 \
    POETRY_VERSION=2.1.3 \
    POETRY_HOME="/opt/poetry" \
    POETRY_VIRTUALENVS_IN_PROJECT=true \
    POETRY_NO_INTERACTION=1 \
    PYSETUP_PATH="/opt/pysetup" \
    VENV_PATH="/opt/pysetup/.venv"

# Prepend poetry and venv to path
ENV PATH="$POETRY_HOME/bin:$VENV_PATH/bin:$PATH"

# Builder stage for dependencies
FROM python-base AS builder-base

# Install system dependencies
RUN apt-get update && apt-get install --no-install-recommends -y \
    curl \
    build-essential \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install Poetry
RUN pip install poetry==${POETRY_VERSION}

# Set working directory for dependencies
WORKDIR $PYSETUP_PATH

# Copy your existing pyproject.toml
COPY pyproject.toml ./

# Generate lock file and install dependencies
RUN poetry lock && poetry install --no-root

# Production stage
FROM python-base AS production

# Copy virtual environment from builder
COPY --from=builder-base $PYSETUP_PATH $PYSETUP_PATH

# Copy application code
COPY . /app/

# Set working directory
WORKDIR /app

# Expose port
EXPOSE 5002

# Run gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:5002", "--workers", "1", "--timeout", "120", "app:app"]