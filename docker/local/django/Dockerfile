ARG PYTHON_VERSION=3.9-slim-bullseye

FROM python:${PYTHON_VERSION} as python

FROM python as python-build-stage
ARG BUILD_ENVIRONMENT=local

RUN apt-get update && apt-get install --no-install-recommends -y \
    build-essential \
    libpq-dev 

COPY ./requirements .

RUN pip wheel --wheel-dir /usr/src/app/wheels \
    -r ${BUILD_ENVIRONMENT}.txt


FROM python as python-run-stage

ARG BUILD_ENVIRONMENT=local
ARG APP_HOME=/app

ENV PYTHONDONTWRITEBYTECODE 1

ENV PYTHONUNBUFFERED 1

ENV BUILD_ENV ${BUILD_ENVIRONMENT}

WORKDIR ${APP_HOME}


RUN apt-get update && apt-get install --no-install-recommends -y \
    libpq-dev \
    gettext \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && rm -rf /var/lib/apt/lists/*

COPY --from=python-build-stage /usr/src/app/wheels /wheels/

RUN pip install --no-cache-dir --no-index --find-links=/wheels/ /wheels/* \
    && rm -rf /wheels/

# The entry point script allows us to connect to a db first before running the app.
COPY ./docker/loacl/django/entrypoint /entrypoint
# processinfg any Windows line endings to Unix line endings.
RUN sed -i 's/\r$//' /entrypoint
# make the entrypointy shell script executable.
RUN chmod +x /entrypoint

# start sript ernsures models are migrated before running the app , static is collectedand the server is started 
COPY ./docker/local/django/start /start
RUN sed -i 's/\r$//' /start
RUN chmod +x /start

COPY . ${APP_HOME}

ENTRYPOINT [ "/entrypoint" ]