FROM python:3.7-alpine

RUN set -x && \
  apk update && \
  apk upgrade && \
  apk --no-cache add \
    bash \
    g++ \
    gcc \
    musl-dev \
    curl && \
  curl -SsL https://downloads.gauge.org/stable | sh && \
  gauge install python && \
  gauge install html-report && \
  gauge install screenshot && \
  gauge update -a && \
  gauge telemetry off

COPY gauge.properties /root/.gauge/config/gauge.properties

WORKDIR /workspace
ENTRYPOINT ["/usr/local/bin/gauge", "config", "--list"]

