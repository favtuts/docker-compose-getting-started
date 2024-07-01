# Docker Compose – What is It, Example & Tutorial
* https://tuts.heomi.net/docker-compose-what-is-it-example-tutorial/

# Check if Docker Compose is installed
```
$ docker compose version

Docker Compose version v2.27.1
```

# Swith Node version

```
$ nvm list
$ nvm use 18.20.3
$ nvm alias default 18.20.3
$ node -v
```

# Create Your Application

Create file `app.js` with following content:
```typescript
const express = require("express");
const {createClient: createRedisClient} = require("redis");

(async function () {

    const app = express();

    const redisClient = createRedisClient({
        url: `redis://redis:6379`
    });

    await redisClient.connect();

    app.get("/", async (request, response) => {
        const counterValue = await redisClient.get("counter");
        const newCounterValue = ((parseInt(counterValue) || 0) + 1);
        await redisClient.set("counter", newCounterValue);
        response.send(`Page loads: ${newCounterValue}`);
    });

    app.listen(80);

})();
```

# Install Dependencies

The code uses the [Express](https://expressjs.com/) web server package to create a simple hit tracking application. Each time you visit the app, it logs your hit in Redis, then displays the total number of page loads.
```
$ npm install express redis
```

# Create Dockerfile

```Dockerfile
FROM node:18-alpine

EXPOSE 80
WORKDIR /app

COPY package.json .
COPY package-lock.json .
RUN npm install

COPY app.js .

ENTRYPOINT ["node", "app.js"]
```

# Create a Docker Compose file

Compose will need 2 containers
* Container 1: The Node.js server app you’ve created.
* Container 2: A Redis instance for your Node.js app to connect to.

```yaml
services:
  app:
    image: app:latest
    build:
      context: .
    ports:
      - ${APP_PORT:-80}:80
  redis:
    image: redis:6
```

# Bring Up Your Containers

```bash
$ docker compose up -d
[+] Running 10/10
 ✔ redis Pulled                                                                                         10.4s
   ✔ 2cc3ae149d28 Already exists                                                                         0.0s
   ✔ e72770150eaa Pull complete                                                                          1.1s
   ✔ 81fd7df17925 Pull complete                                                                          1.1s
   ✔ 923e5e3ba6cf Pull complete                                                                          1.6s
   ✔ a36c397b2e33 Pull complete                                                                          3.5s
   ✔ 891b2e27f729 Pull complete                                                                          3.6s
   ✔ 4f4fb700ef54 Pull complete                                                                          3.6s
   ✔ 6f6c01f42854 Pull complete                                                                          5.2s
 ! app Warning              pull access denied for app, repository does not exist or may...              4.2s
[+] Building 11.7s (9/11)                                                                      docker:default
...
 => [app 2/6] WORKDIR /app                                                                               0.2s
 => [app 3/6] COPY package.json .                                                                        0.0s
 => [app 4/6] COPY package-lock.json .                                                                   0.0s
 => [app 5/6] RUN npm install                                                                            2.1s
 => [app 6/6] COPY app.js .                                                                              0.0s
 => [app] exporting to image                                                                             0.2s
 => => exporting layers                                                                                  0.2s
 => => writing image sha256:8d4f37bbae713f7cef89029d2b352298ed8d62bc0b52c6b4e900931819a119d0             0.0s
 => => naming to docker.io/library/app:latest                                                            0.0s
[+] Running 3/3
 ✔ Network compose-express_default    Created                                                            0.0s
 ✔ Container compose-express-redis-1  Started                                                            0.8s
 ✔ Container compose-express-app-1    Started                                                            0.8s
```

Testing
```
$ curl http://localhost
Page loads: 1
$ curl http://localhost
Page loads: 2
$ curl http://localhost
Page loads: 3
```

# Manage your Docker Compose stack

List all containers
```
$ docker compose ps
NAME                      IMAGE        COMMAND                  SERVICE   CREATED         STATUS         PORTS
compose-express-app-1     app:latest   "node app.js"            app       3 minutes ago   Up 3 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp
compose-express-redis-1   redis:6      "docker-entrypoint.s…"   redis     3 minutes ago   Up 3 minutes   6379/tcp
```

Stop the stack
```
$ docker compose down
[+] Running 3/3
 ✔ Container compose-express-app-1    Removed                                                                                              0.8s
 ✔ Container compose-express-redis-1  Removed                                                                                              0.6s
 ✔ Network compose-express_default    Removed
$ docker compose ps
```

# Change port via env variable

```
$ APP_PORT=8484 docker compose up -d
[+] Running 3/3
 ✔ Network compose-express_default    Created                                                                                              0.0s
 ✔ Container compose-express-app-1    Started                                                                                              0.6s
 ✔ Container compose-express-redis-1  Started
```

Verify 8484 port
```
$ telnet localhost 8484
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
^CConnection closed by foreign host.
```

CURL Testing
```
$ curl http://localhost:8484
Page loads: 1
$ curl http://localhost:8484
Page loads: 2
$ curl http://localhost:8484
Page loads: 3
```

You can override env values by creating environment file `.env`
```
$ cat config.env
APP_PORT=8484

$ docker compose --env-file=config.env up -d
```

# Control service startup order

You can control the order in which services start by setting the `depends_on` field in your docker-compose.yml file. For greater safety, you can wait until the container is passing its [healthcheck](https://docs.docker.com/engine/reference/builder/#healthcheck) by using the long form of `depends_on` instead:

```yaml
services:
  app:
    image: app:latest
    build:
      context: .
    depends_on:
      redis:
        condition: service_healthy
    ports:
      - ${APP_PORT:-80}:80
  redis:
    image: redis:6
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
```


# Use Compose Profiles

Assigning services to profiles allows you to manually activate them when you run Compose commands
```yaml
services:
  app:
    image: app:latest
    build:
      context: .
    ports:
      - ${APP_PORT:-80}:80
  redis:
    image: redis:6
    profiles:
      - with-redis
```

This `docker-compose.yml` file assigns the `redis` service to a profile called `with-redis`. Now the Redis container will only be considered when you include the `--profile with-redis` flag with your `docker compose` commands:
```bash
# Does not start Redis
$ docker compose up -d

# Will start Redis
$ docker compose --profile with-redis up -d
```