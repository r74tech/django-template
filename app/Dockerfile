# Build stage
FROM python:3.12-slim-bookworm as builder

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# set work directory
WORKDIR /usr/src/app

# install build dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/* \
    && pip install --upgrade pip \
    && pip install uv

# install dependencies
COPY ./requirements.txt .
RUN uv pip install --system -r requirements.txt

# Final stage
FROM python:3.12-slim-bookworm

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
ENV APP_HOME=/usr/src/app

# create necessary directories
RUN mkdir -p $APP_HOME && \
    mkdir $APP_HOME/staticfiles && \
    mkdir $APP_HOME/mediafiles

# create the app user
RUN groupadd -r app && useradd -r -g app app

# set work directory
WORKDIR $APP_HOME

# install runtime dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libpq5 \
    netcat-traditional \
    && rm -rf /var/lib/apt/lists/*

# copy dependencies from builder
COPY --from=builder /usr/local/lib/python3.12/site-packages/ /usr/local/lib/python3.12/site-packages/
COPY --from=builder /usr/local/bin/ /usr/local/bin/

# copy entrypoint shell file and make it executable
COPY ./entrypoint.sh $APP_HOME
RUN chmod +x $APP_HOME/entrypoint.sh

# copy project
COPY . $APP_HOME

# chown all the files to the app user
RUN chown -R app:app $APP_HOME

# change to the app user
USER app

# run entrypoint shell file
ENTRYPOINT ["/usr/src/app/entrypoint.sh"]