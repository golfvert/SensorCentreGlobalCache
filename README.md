SensorGlobalCache
=================

### About

This is a tool based on Node Red and Redis to assess the performance of the WIS2 Global Cache.
The metrics defined here https://github.com/wmo-im/wis2-metric-hierarchy/blob/main/metrics/sgc.csv are implemented.

The easiest way to use it is with docker.

In the docker-compose.yml add:
- the connection information to **one** Global Broker. This will determine the baseline when new Notification Messages are published
- up to 10 connections information to the Global Cache. 
In Decembre 2024, there are 6 Global Cache. 
So in the docker compose file, add the details to up to the 6 GC:

      - MQTT_GC_1_BROKER=mqtts:/... # The URL to connect to the local broker of the Global Cache
      - MQTT_GC_1_USERNAME=...
      - MQTT_GC_1_PASSWORD=...
      - GC_1_ID=... # The centre-id of the Global Cache xx-abc-global-cache
      
Access to the management interface or to the metrics endpoint is NOT protected.
It suggested to run this docker container behind the traefik proxy.
With the following labels: 

    labels:
      - traefik.enable=true
      - traefik.http.routers.nodered.entrypoints=websecure
      - traefik.http.routers.nodered.rule=Host(`sensor.wis2dev.io`) && PathPrefix(`/nodered`,`/metrics`)
      - traefik.http.services.nodered.loadbalancer.server.port=1880
      - traefik.http.services.nodered.loadbalancer.server.scheme=http
      - traefik.http.routers.nodered.tls=true
      - traefik.http.routers.nodered.middlewares=auth@file

The "usual" nodered interface of the tool will be available using https://sensor.wis2dev.io/nodered and the metrics https://sensor.wis2dev.io/metrics.
Both will be protected using user/pwd as defined in traefik.

Using prometheus, it is then possible to collect the metrics and to provide dashboard using grafana.
- MQTT_TOPIC is the where the connection to the Global Broker and all Global Cache will be made. 
If `#` then the subscription will be `origin/a/wis2/#` for the GB and `cache/a/wis2/#` for all GCs.