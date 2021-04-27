## PART1- Dockerizing the MongoDB Service

```bash
# step1: connect node-app running on locally to containerized mongodb
$ docker run --name mongodb --rm -d -p 27017:27017 mongo
$ sudo node app.js  # run the node app locally
$ docker logs mongdb # check connection
# "'Node.js v15.11.0, LE (unified)","version":"3.6.1|5.10.3"

# step2: use networks in order to connect containerized nodeapp to containerized mongodb
$ docker network ls
$ docker network create fullstack-webapp-network
$ docker run --name mongodb --rm -d --network fullstack-webapp-network mongo

# step3: use volumes and environment variables
## FULL-FINAL VERSION
$
docker run --name mongodb \
-v data:/data/db \
--rm -d \
--network fullstack-webapp-network \
-e MONGO_INITDB_ROOT_USERNAME=buraku \
-e MONGO_INITDB_ROOT_PASSWORD=password \
mongo
```

## PART2- Dockerizing the Node App

- In order to connect node-app running on locally to containerized mongodb use `mongodb://localhost:27017/course-goals`

- In order to connected dockerized node-app to containerized mongodb use `mongodb://host.docker.internal:27017/course-goals`

So update app code for connection string.

```js
mongoose.connect(
  // 'mongodb://localhost:27017/course-goals',
  "mongodb://host.docker.internal:27017/course-goals",
  // ...
```

```bash
# step1: build image
$ docker build -t node-app-image .
# step2:
$ docker run --name node-backend --rm -d -p 80:80 node-app-image

# step3: use networks in order to connect containerized nodeapp to containerized mongodb
# step4: use volumes and environment variables

#FULL VERSION
$
docker run --name node-backend \
-v logs:/app/logs \
-v $(pwd):/home/node/app \
-v /home/node/app/node_modules \
-e MONGODB_USERNAME=buraku \
-e MONGODB_PASSWORD=password \
--rm -d \
--network web-app-network \
-p 80:80 \
node-app-image
```

### Appendix

[Setting Up Node.js on an Amazon EC2 Instance](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html)

[Super User for Node](https://stackoverflow.com/questions/4976658/on-ec2-sudo-node-command-not-found-but-node-without-sudo-is-ok)

```bash
whereis node
sudo ln -s /home/<my_user>/.nvm/versions/node/v8.9.4/bin/node /usr/bin/node
sudo ln -s /home/ec2-user/.nvm/versions/node/v8.11.3/bin/npm /usr/bin/npm
```

[Amazon EC2 Error: listen EACCES 0.0.0.0:80](https://stackoverflow.com/questions/51020596/amazon-ec2-error-listen-eacces-0-0-0-080)
