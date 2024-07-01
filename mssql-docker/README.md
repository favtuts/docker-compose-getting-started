# How to Run Microsoft SQL Server|MSSQL Express on Docker

Create `docker-compose.yml` with the following instructions
```yml
version: '3.9'
services:
  sql-server-express:
    image: mcr.microsoft.com/mssql/server:latest
    container_name: docker-sql-server-express
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=MyPass@word
      - MSSQL_PID=Express
    ports:
      - "1433:1433"
```

Docker compose has it specific command that will run the `docker-compose.yml` file and set up a container.
```
$ docker-compose up --build -d
WARN[0000] /home/tvt/techspace/docker/docker-compose-getting-started/mssql-docker/docker-compose.yml: `version` is obsolete
[+] Running 2/4
 ⠋ sql-server-express [⣿⣿⡀] 196.2MB / 575.3MB Pulling                                                                                     41.1s
   ✔ e4340e72fdf2 Download complete                                                                                                       37.7s
   ✔ bf5a0d774f7b Download complete                                                                                                       10.5s
   ⠹ db4f0305501a Downloading   [=========>                                         ]  94.37MB/472.6MB                                    39.2s
```

To stop the container
```
$ docker compose down
```


