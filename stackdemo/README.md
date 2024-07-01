# docker-compose-getting-started
Quickstart demo of Docker Compose for blog post: https://tuts.heomi.net/docker-compose-overview-steps-to-install-docker-compose/


# prepare environment

```
virtualenv venv --python=python3.7
source venv/bin/activate
```

# install packages

```
pip install -r requirements.txt
```

# Build and run the application

```
$ docker-compose up -d
```

# List the application running
```
$ docker-compose ps
$ docker ps
```

# Bring the application down
```
$ docker-compose down
```