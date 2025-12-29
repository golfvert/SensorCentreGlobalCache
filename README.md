SensorCentreGlobalCache
=================

### About

This is a tool based on Node Red and Redis to assess the performance of the WIS2 Global Cache.
The metrics defined here https://github.com/wmo-im/wis2-metric-hierarchy/blob/main/metrics/scgc.csv are implemented.

The easiest way to use it is with docker.

In the docker-compose.yml add:
- the connection information to **one** Global Broker. This will determine the baseline when new Notification Messages are published
- up to 10 connections information to the Global Cache. 
In Decembre 2024, there are 6 Global Cache. 
In the docker compose file, add the details to up to the 6 GC:

      - MQTT_GB_BROKER=mqtts:...
      - MQTT_GB_USERNAME=...
      - MQTT_GB_PASSWORD=...
      - MQTT_GC_1_BROKER=mqtts:/... # The URL to connect to the local broker of the Global Cache
      - MQTT_GC_1_USERNAME=...
      - MQTT_GC_1_PASSWORD=...
      - GC_1_ID=... # The centre-id of the Global Cache xx-abc-global-cache
      - SENSOR_CENTRE_ID=xx-yyyyy-sensor-centre-global-cache
      - REDIS_URL=redis:6379        # How to reach redis container
      - CACHE_DELAY_LOW=120
      - CACHE_DELAY_MED=300
      - CACHE_DELAY_MAX=600
      - WNM_LOGGING=true            # If present and treu, this will save WNM, timestamp of reception on origin and on each cache. 10 nines means that the file was not cached.
      - MQTT_TOPIC=#                # is the where the connection to the Global Broker and all Global Cache will be made. 

Access to the management interface or to the metrics endpoint is NOT protected.
It suggested to run this docker container behind the traefik proxy.
With the following labels: 

    labels:
      - traefik.enable=true
      - traefik.http.routers.nodered.entrypoints=websecure
      - traefik.http.routers.nodered.rule=Host(`sensorcentreglobalcache.yyyyy.xx`) && PathPrefix(`/admin`)
      - traefik.http.services.nodered.loadbalancer.server.port=1880
      - traefik.http.services.nodered.loadbalancer.server.scheme=http
      - traefik.http.routers.nodered.tls=true
      - traefik.http.routers.nodered.middlewares=auth@file

The "usual" nodered interface of the tool will be available using https://sensor.wis2dev.io/admin and the metrics https://sensor.wis2dev.io/admin/metrics.
Both will be protected using user/pwd as defined in traefik.

Using prometheus, it is then possible to collect the metrics and to provide dashboard using grafana.
- MQTT_TOPIC is the where the connection to the Global Broker and all Global Cache will be made. 
If `#` then the subscription will be `origin/a/wis2/#` for the GB and `cache/a/wis2/#` for all GCs.
