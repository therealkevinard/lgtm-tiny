# LGTM Tiny
--- 

This is a lightweight LGTM stack for local telemetry.

includes Loki logs, Grafana dashboards, Tempo tracing, and Mimir metrics, along with promtail and alloy for ingestion.

all components are deployed in single-player local mode: no auth, no clustering, little/no persistence.   
this is a boilerplate: it works out-of-the-box, but there's room to finesse it for your own preferences.

!! only really tested on mac-arm, but it's as portable as docker is. some platforms _may_ need small adjustments to
tags, but they should work

the main ingestion endpoints are through alloy, listening for grpc on :4317 and http on :4318.  
configure your otlp grpc exporters for localhost:4317, and start observing.

## quickstart

1. run the compose stack with `docker compose up -d`
2. open grafana on http://localhost:3000 
3. run an application with telemetry exported to localhost:4317 
4. go back to grafana and see what's happened
