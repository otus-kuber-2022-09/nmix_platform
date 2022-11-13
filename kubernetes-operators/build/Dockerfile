FROM python:3.10.7-alpine3.16 as base

RUN pip install --no-cache-dir pipenv==2022.9.24

ENV PROJECT_DIR /tmp

WORKDIR ${PROJECT_DIR}

COPY Pipfile Pipfile.lock ${PROJECT_DIR}/

RUN pipenv requirements > requirements.txt

FROM base

ENV PROJECT_DIR /app
WORKDIR ${PROJECT_DIR}

RUN apk update && apk add --no-cache build-base=0.5-r3 libpq-dev=14.5-r0 \
    && rm -rf /var/cache/apk/*
RUN pip install --no-cache-dir -r /tmp/requirements.txt

COPY . .

ENTRYPOINT ["kopf", "run", "/app/mysql-operator.py"]
