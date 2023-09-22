# How To Install and Use Docker Compose on Ubuntu 22.04
* https://www.favtuts.com/how-to-install-and-use-docker-compose-on-ubuntu-22-04/

# Instroduction
To demonstrate how to set up a `docker-compose.yml`` file and work with Docker Compose, you’ll create a web server environment using the official Nginx image from Docker Hub, the public Docker registry. This containerized environment will serve a single static HTML file.

# Setup a demo app

Creating a new directory
```
$ mkdir ~/compose-demo
$ cd ~/compose-demo
```

Setup application folder to serve as the document root for Nginx environment
```
$ mkdir app
```

Create new `index.html` file within the `app` folder
```
$ touch app/index.html
$ nano app/index.html
```

Place the following content into this file
```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Docker Compose Demo</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/kognise/water.css@latest/dist/dark.min.css">
</head>
<body>

    <h1>This is a Docker Compose Demo Page.</h1>
    <p>This content is being served by an Nginx container.</p>

</body>
</html>
```

Next, create the `docker-compose.yml` file
```yml
version: '3.7'
services:
  web:
    image: nginx:alpine
    ports:
      - "8000:80"
    volumes:
      - ./app:/usr/share/nginx/html
```

The `docker-compose.yml` file typically starts off with the `version` definition. This will tell Docker Compose which configuration version you’re using.

You then have the `services` block, where you set up the services that are part of this environment. In your case, you have a single service called `web`. This service uses the `nginx:alpine` image and sets up a port redirection with the `ports` directive. All requests on port `8000` of the host machine (the system from where you’re running Docker Compose) will be redirected to the `web` container on port `80`, where Nginx will be running.

The `volumes` directive will create a shared volume between the host machine and the container. This will share the local `app` folder with the container, and the volume will be located at /`usr/share/nginx/html` inside the container, which will then overwrite the default document root for Nginx.

# Running Docker Compose

With the `docker-compose.yml` file in place, you can now execute Docker Compose to bring your environment up. The following command will download the necessary Docker images, create a container for the `web` service, and run the containerized environment in background mode:
```
$ docker compose up -d

[+] Running 9/9
 ✔ web 8 layers [⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                          15.8s 
   ✔ 7264a8db6415 Pull complete                                                             4.5s 
   ✔ 518c62654cf0 Pull complete                                                             2.6s 
   ✔ d8c801465ddf Pull complete                                                             2.6s 
   ✔ ac28ec6b1e86 Pull complete                                                             5.0s 
   ✔ eb8fb38efa48 Pull complete                                                             4.4s 
   ✔ e92e38a9a0eb Pull complete                                                             5.4s 
   ✔ 58663ac43ae7 Pull complete                                                             5.4s 
   ✔ 2f545e207252 Pull complete                                                            11.1s 
[+] Building 0.0s (0/0)                                                           docker:default
[+] Running 2/2
 ✔ Network compose-demo_default  Created                                                    0.1s 
 ✔ Container compose-demo-web-1  Started                                                    0.2s
```

Your environment is now up and running in the background. To verify that the container is active, you can run:
```
$ docker compose ps
```

You can now access the demo application by pointing your browser to either `localhost:8000` if you are running this demo on your local machine, or `your_server_domain_or_IP:8000` if you are running this demo on a remote server.

# Docker Compose Commands

To check the logs produced by your Nginx container, you can use the `logs` command:
```
$ docker compose logs
```

If you want to pause the environment execution without changing the current state of your containers, you can use:
```
$ docker compose pause
```

To resume execution after issuing a pause:
```
$ docker compose unpause
```

The `stop` command will terminate the container execution, but it won’t destroy any data associated with your containers:
```
$ docker compose stop
```

If you want to remove the containers, networks, and volumes associated with this containerized environment, use the `down` command:
```
$ docker compose down
```

In case you want to also remove the base image from your system, you can use:
```
$ docker image rm nginx:alpine
```