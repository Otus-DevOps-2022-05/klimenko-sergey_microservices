FROM python:3.6.0-alpine

WORKDIR /app
COPY . /app

RUN apk --no-cache --update add build-base gcc musl-dev && \
    pip install --no-cache --no-cache-dir --upgrade pip setuptools && \
    pip install --no-cache-dir -r /app/requirements.txt && \
    apk del build-base && \
    rm -rf /var/cache/apk/*

ENV POST_DATABASE_HOST=post_db \
    POST_DATABASE=posts

ENTRYPOINT ["python3", "post_app.py"]
