# ---------- Builder ----------
FROM python:3.12-slim AS builder

WORKDIR /app

RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*


RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir \
    && pip install -r requirements.txt   

# ---------- Runtime ----------
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

RUN useradd -m djangouser
WORKDIR /app

COPY --from=builder /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin
COPY . .

RUN chown -R djangouser:djangouser /app
USER djangouser

EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
