FROM ubuntu:bionic

RUN apt-get update \
  && apt-get install --no-install-recommends -y python nodejs 

COPY docker-entrypoint.sh /app/docker-entrypoint.sh
WORKDIR /app

ENTRYPOINT ["/app/docker_entrypoint.sh"]

RUN  apt-get purge -y --auto-remove
RUN rm -rf /var/lib/apt/lists/*
