# PART1- GETTING STARTED

## Building Docker Networks

```bash
$ docker network ls
$ docker network create fullstack-webapp-network
```

## Dockerizing the MongoDB Service

```bash
# step1: connect nodeapp running on locally to containerized mongodb
$ docker run --name mongodb --rm -d -p 27017:27017 mongo
# step2: use networks in order to connect containerized nodeapp to containerized mongodb
$ docker run --name mongodb --rm -d --network fullstack-webapp-network mongo
# step3: use volumes and environment variables
```

### FULL-FINAL VERSION

```bash
$
docker run --name mongodb \
-v data:/data/db \
--rm -d \
--network fullstack-webapp-network \
-e MONGO_INITDB_ROOT_USERNAME=buraku \
-e MONGO_INITDB_ROOT_PASSWORD=password \
mongo
```

## Dockerizing the Node App
