# Deploy a Go Web Application with Docker and Nginx
* https://www.favtuts.com/how-to-deploy-a-go-web-application-with-docker-and-nginx-on-ubuntu-22-04/

# Create an Example Go Web App

You’ll store all data under ~/go-docker. Run the following command to create this folder:
```
$ mkdir ~/go-docker
$ cd ~/go-docker
```

You’ll store your example Go web app in a file named `main.go`. Create it using your text editor:
```
$ nano main.go
```

Add the following lines into the file `~/go-docker/main.go`
```go
package main

import (
    "fmt"
    "net/http"

    "github.com/gorilla/mux"
)

func main() {
    r := mux.NewRouter()

    r.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "<h1>This is the homepage. Try /hello and /hello/Sammy\n</h1>")
    })

    r.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "<h1>Hello from Docker!\n</h1>")
    })

    r.HandleFunc("/hello/{name}", func(w http.ResponseWriter, r *http.Request) {
        vars := mux.Vars(r)
        title := vars["name"]

        fmt.Fprintf(w, "<h1>Hello, %s!\n</h1>", title)
    })

    http.ListenAndServe(":80", r)
}
```

# Deploy nginx-proxy with Let's Encrypt

You’ll be storing the Docker Compose configuration for `nginx-proxy` in a file named `nginx-proxy-compose.yaml`. Create it by running:
```
$ nano nginx-proxy-compose.yaml
```

Add the following lines to the file:`~/go-docker/nginx-proxy-compose.yaml`

```yaml
version: '3'

services:
  nginx-proxy:
    restart: always
    image: jwilder/nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "/etc/nginx/vhost.d"
      - "/usr/share/nginx/html"
      - "/var/run/docker.sock:/tmp/docker.sock:ro"
      - "/etc/nginx/certs"

  letsencrypt-nginx-proxy-companion:
    restart: always
    image: jrcs/letsencrypt-nginx-proxy-companion
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    volumes_from:
      - "nginx-proxy"
```

Deploy the `nginx-proxy` by running:
```
$ docker compose -f nginx-proxy-compose.yaml up -d
```

# Dockerizing the Go Web App

Create the `Dockerfile` with your text editor:
```
$ nano Dockerfile
```

Add the following lines: `~/go-docker/Dockerfile`

```Dockerfile
FROM golang:alpine AS build
RUN apk --no-cache add gcc g++ make git
WORKDIR /go/src/app
COPY . .
RUN go mod init webserver
RUN go mod tidy
RUN GOOS=linux go build -ldflags="-s -w" -o ./bin/web-app ./main.go

FROM alpine:3.17
RUN apk --no-cache add ca-certificates
WORKDIR /usr/bin
COPY --from=build /go/src/app/bin /go/bin
EXPOSE 80
ENTRYPOINT /go/bin/web-app --port 80
```

# Creating and Running the Docker Compose File

You will store the Docker Compose configuration for the Go web app in a file named `go-app-compose.yaml`. Create it by running:

```
$ nano go-app-compose.yaml
```

Add the following lines to this file: `~/go-docker/go-app-compose.yaml`

```yaml
version: '3'
services:
  go-web-app:
    restart: always
    build:
      dockerfile: Dockerfile
      context: .
    environment:
      - VIRTUAL_HOST=your_domain
      - LETSENCRYPT_HOST=your_domain
```

Replace `your_domain` both times with your domain name.

Now run your Go web app in the background via Docker Compose:
```
$ docker compose -f go-app-compose.yaml up -d
```

You can now navigate to `https://your_domain/` to access your homepage
