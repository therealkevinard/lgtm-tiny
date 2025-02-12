# LGTM Tiny
--- 

This is a lightweight LGTM stack for local telemetry.  
it's a boilerplate: it works fine out-of-the-box, but there's room to finesse it for your own preferences.

all components are pre-wired and deployed in single-player local mode: no auth, no clustering, little/no persistence.

interfaces

- [grafana](https://grafana.com/docs/grafana/latest/): serves ui on localhost:3000

ingest

- [alloy](https://grafana.com/docs/alloy/latest/): listens on :4317 (otlp-grpc), :4318 (otlp-http), and :4040 (
  pyro-http)
- [promtail](https://grafana.com/docs/loki/latest/send-data/promtail/): reads syslog and docker container logs; sinks to
  loki (skips alloy)

stores

- [loki](https://grafana.com/docs/loki/latest/): logs
- [mimir](https://grafana.com/docs/mimir/latest/): metrics
- [tempo](https://grafana.com/docs/tempo/latest/): traces
- [pyroscope](https://grafana.com/docs/pyroscope/latest/): profiling

> !! only really tested on mac-arm, but it's as portable as docker is. some platforms _may_ need small adjustments to
tags, but they should work  
> !! i don't _personally_ use otlphttp much/at-all - i _assume_ alloy's http ingest works? 

the main ingestion endpoints are through alloy, listening for grpc on :4317 and http on :4318.  
configure your otlp grpc exporters for localhost:4317, and start observing.

## quickstart

1. run the compose stack with `docker compose up -d`
2. open grafana on http://localhost:3000
3. run an application with telemetry exported to localhost:4317
4. go back to grafana and see what's happened

## notes and cheatsheets

> a few little notes about the OOTB configs

### promtail

**scraper labels**  
discovered logs have a few label/relabel configs in place.
system and docker container logs are discovered via different scrapes, delineated by the `job` label.   
system logs can be found in loki with logql: `{job="varlogs", filename="$path_to_logfile"}`  
docker logs can be found with logql: `{job="docker_sock", container_name="$container_name"}`

**otel vs hard-file**  
!! a common practice that could cause duplicate log ingest:   
using otel-native logging with a forked logger (it's common practice to fork otel logs to stdout/file) can/will cause
double-discovery.  
logs will be captured once from otel ingest, and again through the docker socket.    
that's by design: we capture twice because you're logging twice.   
in this case, you probably want to prefer the otel logs. otel/tempo will add precise labels to the ingested jobs
depending on your otel resource configs.   
also worth noting that the docker.sock logs can be directly excluded with `{job!="docker_sock"}`
