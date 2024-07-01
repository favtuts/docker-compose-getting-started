# WordPress (Apache/PHP and MySQL) with Docker Compose
* https://tuts.heomi.net/docker-compose-what-is-it-example-tutorial/

# Docker Compose

```yml
services:
  wordpress:
    image: wordpress:${WORDPRESS_TAG:-6.2}
    depends_on:
      - mysql
    ports:
      - ${WORDPRESS_PORT:-80}:80
    environment:
      - WORDPRESS_DB_HOST=mysql
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=${DATABASE_USER_PASSWORD}
      - WORDPRESS_DB_NAME=wordpress
    volumes:
      - wordpress:/var/www/html
    restart: unless-stopped
  mysql:
    image: mysql:8.0
    environment:
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=${DATABASE_USER_PASSWORD}
      - MYSQL_RANDOM_ROOT_PASSWORD="1"
    volumes:
      - mysql:/var/lib/mysql
    restart: unless-stopped

volumes:
  wordpress:
  mysql:
```

# Run Compose

Env directly:
```
$ DATABASE_USER_PASSWORD=abc123 docker compose up -d
```

Env from file:
```
$ docker compose --env-file=config.env up -d


$ docker compose --env-file=config.env up -d
[+] Running 34/34
 ✔ mysql Pulled
 ✔ wordpress Pulled                                                                                                                       57.0s   
[+] Running 5/5
 ✔ Network wordpress-demo_default        Created                                                                                           0.0s
 ✔ Volume "wordpress-demo_wordpress"     Created                                                                                           0.0s
 ✔ Volume "wordpress-demo_mysql"         Created                                                                                           0.0s
 ✔ Container wordpress-demo-mysql-1      Started                                                                                           0.7s
 ✔ Container wordpress-demo-wordpress-1  Started                                                                                           0.7s
```

# Visit WordPress side

```
http://localhost:8765
```