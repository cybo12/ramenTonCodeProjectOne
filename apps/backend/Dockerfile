FROM python:3.14.4-slim AS builder
RUN pip install --upgrade pip uv
WORKDIR /build
COPY pyproject.toml ./
RUN uv sync --verbose && ls -la .venv/bin/ || (echo "uv sync failed" && exit 1)

FROM builder
WORKDIR /app
COPY --from=builder /build/.venv /app/.venv
ENV PATH="/app/.venv/bin:$PATH"
COPY app ./app
RUN which uvicorn || (echo "uvicorn not found" && exit 1)
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0"]