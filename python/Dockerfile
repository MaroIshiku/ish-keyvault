FROM python:3.12-alpine

ENV PYTHONUNBUFFERED=1 \
    PORT=3000 \
    PUBLIC_DIR=/app/public

WORKDIR /app

COPY python/requirements.txt /app/requirements.txt
RUN pip install --no-cache-dir -r /app/requirements.txt

COPY python/app.py /app/app.py
COPY public /app/public

EXPOSE 3000

CMD ["gunicorn", "--bind", "0.0.0.0:3000", "--workers", "1", "--threads", "8", "--access-logfile", "-", "--error-logfile", "-", "app:app"]
