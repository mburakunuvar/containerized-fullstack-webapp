# GETTING STARTED

## PART1-A Dockerizing the MongoDB Service

```bash
$ docker run --name mongodb --rm -d -p 27017:27017 mongo
$ sudo node app.js  # run the node app locally
$ docker logs mongdb # check connection
# "'Node.js v15.11.0, LE (unified)","version":"3.6.1|5.10.3"
```

## PART2-A Dockerizing the Node App

- In order to connect node-app running on locally to containerized mongodb use `mongodb://localhost:27017/course-goals` and publish the ports of mongodb container by `-p 27017:27017`

- In order to connect dockerized node-app to containerized mongodb use `mongodb://host.docker.internal:27017/course-goals`. That's because, `localhost` will refer to the host of the container itself in this setup which should be replaced by the host of your system. In order to be able to resolve to localhost of your system, use `host.docker.internal` and publish the ports of mongodb container by `-p 27017:27017`.

So update app code for connection string.

```js
mongoose.connect(
  // 'mongodb://localhost:27017/course-goals',
  "mongodb://host.docker.internal:27017/course-goals",
  // ...
```

```bash
$ docker build -t node-app-image .
$ docker run --name node-backend --rm -d -p 80:80 node-app-image
```

## PART3-A Dockerizing the React App

- In order to connect react-app running locally to a containerized node-app use `http://PUBLIC-IP/goals/` and publish the ports of node-backend container by `-p 80:80`

- In order to connect dockerized react-app to containerized node-app there is **no need to use** `http://host.docker.internal/goals/`. React App will not be running on a container but on your browser which can communicate directly with the localhost of your system. Therefore `localhost` within app code will not refer to the host of the container itself but to the the host of your system. There's no need for additional resolution in this case, but you'll still need to publish the ports of node-backend container by `-p 80:80`

```bash
$ docker build -t react-app-image .
## REACT Apps will require -it flag
$ docker run --name react-frontend --rm -d -p 3000:3000 -it react-app-image
```

# USING DOCKER NETWORKS

So far all communication is possible through localhost machine by publishing ports, there's no cross container communication yet

## Create Networks

```bash
$ docker network create fullstack-webapp-network
$ docker network ls
```

## PART1-B Use Docker Networks for MongoDB Container

```bash
$ docker run --name mongodb --rm -d --network fullstack-webapp-network mongo
# use volumes and environment variables
## FULL-FINAL VERSION MONGODB
$ docker run --name mongodb \
-v data:/data/db \
--rm -d \
--network fullstack-webapp-network \
-e MONGO_INITDB_ROOT_USERNAME=buraku \
-e MONGO_INITDB_ROOT_PASSWORD=password \
mongo
```

## PART2-B Docker Networks for Node-Backend Container

- Previously in order to connect dockerized node-app to containerized mongodb we had used `mongodb://host.docker.internal:27017/course-goals` and we had to publish ports to enable communication through local system.

- Now in order to connect dockerized node-app to containerized mongodb within same network we will use `mongodb://container-name:27017/course-goals` and there will be no need to publish a port for communication through local system.

So update app code for connection string.

```js
mongoose.connect(
  // 'mongodb://localhost:27017/course-goals',
  // "mongodb://host.docker.internal:27017/course-goals",
  "mongodb://mongodb:27017/course-goals",
```

- Finally we'll need to enable environment variables in Dockerfile and use them in app code for a secure connection

```js
mongoose.connect(
  `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/course-goals?authSource=admin`,
```

```bash
# use networks in order to connect containerized node-app to containerized mongodb
# use volumes and environment variables
## FULL-FINAL VERSION NODE-BACKEND
$
docker run --name node-backend \
-v logs:/home/node/app/logs \
-v $(pwd):/home/node/app \
-v /home/node/app/node_modules \
-e MONGODB_USERNAME=buraku \
-e MONGODB_PASSWORD=password \
--rm -d \
--network fullstack-webapp-network \
-p 80:80 \
node-app-image
```

### PART3B Use Docker Networks for React-Frontend Container (not needed)

- Previously in order to connect dockerized react-app to containerized node-app there was **no need to use** `http://host.docker.internal/goals/` so we had proceeded with `http://PUBLIC-IP/goals/`. React App will not be running on a container but on your browser which can communicate directly with the localhost of your system. Therefore `localhost` within app code will not refer to the host of the container itself but to the the host of your system. There's no need for additional resolution in this case, but you'll still need to publish the ports of node-backend container by `-p 80:80`

- Now in order to connect dockerized react-app to containerized node-app within same network, we are unable to use `container-name` because of the same reason. React App will not be running on a container but on your browser which can communicate directly with the localhost of your system.

- Therefore, not it's not even needed to place react-app container within network, since it is a browser side code. But, we'll need to publish a port for communication through local system which is the only way in order to connect dockerized react-app (whose JS will be running on browser) to containerized node-app.

```bash
$ docker build -t react-app-image .

$ \ #FULL VERSION
docker run --name react-frontend \
-v $(pwd)/src:/home/node/app/src \
--rm -d \
# --network fullstack-webapp-network \
-p 3000:3000 \
-it \
react-app-image
```

### OPTIONAL Update for Development Environment

- Use `nodemon` for development environment of dockerized node-app (update package.json and Dockerfile )

```json

  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "nodemon app.js"
  },
  ...
  ...
  "devDependencies": {
    "nodemon": "^2.0.4"
  }
}
```

- This is not needed for react environment as it's a built in feature.

### Appendix

[Setting Up Node.js on an Amazon EC2 Instance](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html)

[Super User for Node](https://stackoverflow.com/questions/4976658/on-ec2-sudo-node-command-not-found-but-node-without-sudo-is-ok)

```bash
whereis node
sudo ln -s /home/<my_user>/.nvm/versions/node/v8.9.4/bin/node /usr/bin/node
sudo ln -s /home/ec2-user/.nvm/versions/node/v8.11.3/bin/npm /usr/bin/npm
```

[Amazon EC2 Error: listen EACCES 0.0.0.0:80](https://stackoverflow.com/questions/51020596/amazon-ec2-error-listen-eacces-0-0-0-080)

[What is linux equivalent of “host.docker.internal”](https://stackoverflow.com/questions/48546124/what-is-linux-equivalent-of-host-docker-internal)
[Support '--add-host=host.docker.internal:host-gateway](https://github.com/containers/podman/issues/8466)
