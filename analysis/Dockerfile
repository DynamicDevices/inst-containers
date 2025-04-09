FROM ubuntu:focal

RUN apt-get update \
  && apt-get install --no-install-recommends -y python3 nodejs python3-paho-mqtt

COPY docker-entrypoint.sh /app/docker-entrypoint.sh
WORKDIR /app

ENTRYPOINT ["/app/docker_entrypoint.sh"]

RUN  apt-get purge -y --auto-remove
RUN rm -rf /var/lib/apt/lists/*
