# Install MySQL With PhpMyAdmin Using Docker
* https://tuts.heomi.net/install-mysql-with-phpmyadmin-using-docker/

# Step 1 — Installing MySQL

Grap MySQL image
```
$ docker pull mysql:8.0.19
```

Run MySQL container
```
$ docker run --name mysql -d -e MYSQL_ROOT_PASSWORD=my-secret-pw mysql:8.0.19
```

Verify MySQL container is running
```
$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                 NAMES
70894186f36c   mysql:8.0.19   "docker-entrypoint.s…"   38 seconds ago   Up 37 seconds   3306/tcp, 33060/tcp   mysql
```

# Step 2 — Installing phpMyAdmin

Pull phpMyAdmin from Docker Hub
```
$ docker pull phpmyadmin/phpmyadmin:5.0.2
```

phpMyAdmin uses the `PMA_HOST` environment variable to know where to connect to the MySQL database. We set this variable to be equal to the MySQL container name, namely `mysql`.

Start a phpMyAdmin container
```
$ docker run --name phpmyadmin -d -p 8080:80 -e PMA_HOST=mysql phpmyadmin/phpmyadmin:5.0.2
```

Let’s verify both containers are running side to side with `docker ps`:
```
$ docker ps
CONTAINER ID   IMAGE                         COMMAND                  CREATED         STATUS         PORTS                                   NAMES
b34a43abee87   phpmyadmin/phpmyadmin:5.0.2   "/docker-entrypoint.…"   7 seconds ago   Up 6 seconds   0.0.0.0:8080->80/tcp, :::8080->80/tcp   phpmyadmin
70894186f36c   mysql:8.0.19                  "docker-entrypoint.s…"   5 minutes ago   Up 5 minutes   3306/tcp, 33060/tcp                     mysql
```

# Step 3 — Creating a Docker network

At this point, phpMyAdmin can’t connect to MySQL because the `mysql` hostname we used in the previous step doesn’t translate to the IP address of the MySQL container.

To translate hostname to IP Address, or DNS resolution, we need to create Docker network. All containers inside that network will be able to communicate with each other using their names.

Create a Docker network
```
$ docker network create my-network
```

Now, let’s connect both MySQL and phpMyAdmin containers to this network.
```
$ docker network connect my-network mysql
$ docker network connect my-network phpmyadmin
```

You won’t see anything printed out in the console, which is good. It means both containers are now part of our network and can communicate with each other using their names.

Navigate to `localhost:8080` in your browser, and log in with username: `root` and password: `my-secret-pw`. You should now be inside the phpMyAdmin dashboard. 


# Step 4 — (Bonus) Using Docker Compose

With Compose, you use one file to configure all of your application’s containers. Then, with a single command, you create and start all containers from your configuration.

```yaml
# We're using version 3.7 of the Docker Compose file format
version: "3.7"

# Define services/containers
services:
  # MySQL container
  mysql:
    # Use mysql:8.0.19 image
    image: mysql:8.0.19
    # Connect to "my-network" network, as defined below
    networks:
      - my-network
    # Pass a list of environment variables to the container
    environment:
      MYSQL_ROOT_PASSWORD: my-secret-pw

  # phpMyAdmin container
  phpmyadmin:
    # Use phpmyadmin/phpmyadmin:5.0.2 image
    image: phpmyadmin/phpmyadmin:5.0.2
    # Connect to "my-network" network, as defined below
    networks:
      - my-network
    # Map port 8080 on the host to port 80 inside the container
    # Syntax is: "HOST_PORT:CONTAINER_PORT"
    ports:
      - "8080:80"
    # Pass a list of environment variables to the container
    environment:
      PMA_HOST: mysql
    # Wait for "mysql" container to start first
    depends_on:
      - mysql

# Define networks
networks:
  my-network:
```

Stop all running Docker containers
```
$ docker stop $(docker ps -q)
```

Remove the containers
```
$ docker rm $(docker ps -a -q)
```


To start the application stack, in the same folder as the `docker-compose.yml` file, run:
```
$ docker-compose up -d
```
The command to stop and remove all containers and networks is:
```
$ docker-compose down
```