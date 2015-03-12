# How to Docker our way to awesome

---

### What was the Definition of Awesome?

- Deploy _whatever_ we want, _whenever_ we want
- One-click (dev) environment setups
- Create/modify test/production envs instantly

Goal with Docker: PoC using Docker for hosting applications + service dependencies (database, queue, etc.) <!-- .element: class="fragment" data-fragment-index="1" -->

---

### What I thought I'd do...

- Understand how Docker works
- Understand best practices
- Have Docker host @ Amazon / use Amazon Elastic Container Service
- Dockerize AssMan API & run on AWS Docker host / Amazon ECS
- (maybe) Dockerize AssMan UI & run on AWS Docker host / Amazon ECS
- (doubtful) Have a private Docker registry

---

### What I really did

- Understand how Docker works <!-- .element: class="fragment" data-fragment-index="1" -->
- Understand some of the best practices <!-- .element: class="fragment" data-fragment-index="2" -->
- Docker host @ Amazon (no ECS) <!-- .element: class="fragment" data-fragment-index="3" -->
- AssMan API -> Dockerized + easy release script <!-- .element: class="fragment" data-fragment-index="4" -->
- AssMan UI -> Dockerized + easy release script <!-- .element: class="fragment" data-fragment-index="5" -->
- Container running NginX acting as reverse proxy for other containers <!-- .element: class="fragment" data-fragment-index="6" -->
- Configure new containers with virtual host for NginX with only 1 Environment variable <!-- .element: class="fragment" data-fragment-index="7" -->
- Private Docker registry, in Docker container, backed by S3 <!-- .element: class="fragment" data-fragment-index="8" -->

---

### The tools of the trade

- `docker` client: run commands against Docker host
- `docker` daemon: client running as daemon == Docker host
- `docker-machine`: easily provision Docker hosts locally or in the cloud (DO, AWS)
- `docker-compose`: a.k.a. _Fig_, for setting up local development environments
- `docker-swarm`: have many Docker hosts, access them as 1
- _Docker registry_: run a private image registry

---

## Docker - The good stuff

---

- Running new containers is deceptively easy
- So many great images in the wild
- Diff-based file system == small incremental updates for images
- Freakishly fast! (after initial image pull)
- Easy local development setup (Fig/docker-compose)
- Docker itself (host & client) is stable: v1.5

---

## Docker - The bad

---

- Auxiliary tooling (machine, swarm, registry) not mature
- 3rd party service offerings lacking (looking at you, Amazon ECS)
- (opinion) Must have a host running for pushing images to the registry

---

## Docker - The gotchas

---

- Must have a host running for pushing images to the registry
- Private Docker registry must use SSL
    - Clients will refuse to push otherwise
    - Docker host can be configured to ignore this
- For persistency: mount volume from host, for databases. Variation: have dedicated "storage" containers, shared with other containers

---

# DEMO TIME

---

## So, what will the workflow look like?

---

### Development

- Service dependencies (database, queue, etc.) run in local Docker containers, 
- Setup using `docker-compose` (Fig)
- App itself runs "natively", using `sbt run` or standalone in your favourite Java container.

---

### Build

Each pushed commit will trigger the following on Team City:

- Run unit tests
- Pull Docker images for dependencies and run them in containers
- Build image for app and run in container
- Run integration/acceptance tests
- If all succeeds, push app image to private registry
- Deploy this latest image to Docker test environment

Images will be tagged with git commit hash and/or with version number for proper releases

---

### Deployment

For now: pull image with proper version tag and run in container(s)

Later: part of TC process. True continuous delivery

---

## Emergencies & Failures 

## or 

## love2log, monitor your metrics

---

- Best practise: don't SSH into a container, there isn't much there anyways
- Logging: collect logs in a central place (ELK). See Sacha for that ;) Options:
    - Syslog on host
    - Syslog in container
    - Lumberjack (logstash appender)
    - *Application logs: use Logback -> Logstash appender*
- Monitoring:
    - Use Graphite and StatsD to measure *everything*.
    - Especially on the application level.
    - Monitor & set triggers

---

# Questions?