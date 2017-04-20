# Docker Workshop

<img data-src="https://cdn.worldvectorlogo.com/logos/docker.svg" width="300px" style="background: white">

### 21/04/2017

---

# Introduction

----

## What is docker?

* Open platform for developing, shipping, and running applications
* Package a service with its dependencies (Container)
* Execution environment isolation

----

## Advantages

* Cross environment consistency
* Responsive deployment and scaling
* Expand development team easily
* Easy upgrades
* Utility to test new technologies

----

## Architecture

<img data-src="https://docs.docker.com/engine/article-img/engine-components-flow.png" style="background: white">

----

## Architecture

* Image:  read-only template with instructions for creating a Docker container
* Container: runnable instance of an image
* Registry: Repository of images
* Volumes: persistent storage for containers

----

## Docker vs VM

<img data-src="https://www.docker.com/sites/default/files/VM%402x.png" style="background: white" height="300px">
<img data-src="https://www.docker.com/sites/default/files/Container%402x.png" style="background: white" height="300px">

* Virtual machines run guest operating systems
* Containers can share a single kernel

---

# Docker Basic CLI

----

## Check docker is installed

```
docker version
```

<pre>
Client:
 Version:      17.03.0-ce
 API version:  1.26
 Go version:   go1.7.5
 Git commit:   3a232c8
 Built:        Tue Feb 28 08:21:51 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.03.0-ce
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   3a232c8
 Built:        Tue Feb 28 08:21:51 2017
 OS/Arch:      linux/amd64
 Experimental: false

</pre>

----

## Listing images and containers

```
docker images
```

<pre style="font-size: 10px">
REPOSITORY                   TAG                                        IMAGE ID            CREATED             SIZE
repo/img1                    6a438d4785a7485b2cb3d78f6d021be36a695e55   2663816b6829        40 minutes ago      195 MB
repo/img1                    latest                                     2663816b6829        40 minutes ago      195 MB
repo/img2                    &lt;none&gt;                                     86229db3bf50        About an hour ago   195 MB
postgres                     9.5                                        bd44e8a44ab2        3 weeks ago         265 MB
swaggerapi/swagger-ui        latest                                     2f6c97012fd7        4 weeks ago         10.9 MB
...
</pre>

```
docker ps
```

<pre style="font-size: 10px">
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                  NAMES
9cfb6d2a8004   repo/img1              "/bin/sh /entrypoi..."   42 minutes ago   Up 42 minutes   0.0.0.0:80->3000/tcp   registry_app
3d1cbd8ef6a9   repo/img2              "nginx -g 'daemon ..."   42 minutes ago   Up 42 minutes   80/tcp, 443/tcp        registry_web
499d5c80e917   swaggerapi/swagger-ui  "/bin/sh -c 'exec ..."   6 hours ago      Up 6 hours      0.0.0.0:8089->8080/tcp swagger-ui
f2f42facf79b   postgres:9.5           "docker-entrypoint..."   6 days ago       Up 42 minutes   5432/tcp               registry_db
</pre>

----

## Getting images

* Images available in docker hub

```
docker search --limit=10 nginx
```
<pre style="font-size: 13px">
NAME                      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                     Official build of Nginx.                        5788      [OK]       
jwilder/nginx-proxy       Automated Nginx reverse proxy for docker c...   998                  [OK]
richarvey/nginx-php-fpm   Container running Nginx + PHP-FPM capable ...   367                  [OK]
webdevops/php-nginx       Nginx with PHP-FPM                              77                   [OK]
bitnami/nginx             Bitnami nginx Docker Image                      26                   [OK]
webdevops/nginx           Nginx container                                 7                    [OK]
blacklabelops/nginx       Dockerized Nginx Reverse Proxy Server.          4                    [OK]
1science/nginx            Nginx Docker images that include Consul Te...   4                    [OK]
frekele/nginx             docker run --rm --name nginx -p 80:80 -p 4...   3                    [OK]
xutongle/nginx            nginx http                                      1                    [OK]
</pre>

* [Docker Hub](https://hub.docker.com/)

* Select one and download it

```
docker pull nginx
```

----

## Running an independent container

```
docker run alpine echo "Hello World"
```

* It pulls the image if not already in cache
* It executes a single command in a container and exits
* No tty or interactive mode by default

----

## Running an interactive container

```
docker run -it --rm alpine /bin/sh
/ # whoami
root
```

* Beware of cli argument order
* The container is still alive after execution unless rm flag used
* In a container even unprivileged user can be root

----

## Running a container as a service

```
docker run -d -p 4532:5432 --name pg95 postgres:9.5
psql -p 4532 -h localhost -U postgres
docker kill pg95; docker rm pg95
```

* Creates a postgres db
* Exposes port 4532 in the host and connects it to 5432 in the container
* When the container is recreated the database completely dissapears
* A database should have persistent storage...

----

## Defining volumes when running containers

```
docker run -v pgdata:/var/lib/postgresql/data \
           -d -p 4533:5432 --name pg95-2 postgres:9.5
psql -p 4533 -h localhost -U postgres
```

* Data persists while recreating the container

----

## Mounting volumes in host filesystem

```
docker run -v pgdata:/var/lib/postgresql/data \
           -d -p 4533:5432 --name pg95-2 postgres:9.5
psql -p 4533 -h localhost -U postgres
```

* Beware of permissions (directory will probably be owned by root)

----

## Defining the environment

```
docker run --name mypgdb \
           -e POSTGRES_PASSWORD=mypass -d postgres:9.5
```

* Some docker images provide configuration via environment variables
* Check the docs for available options

----

## Whats is happening in my detached container?

```
docker logs mypgdb
```

* Usually containers provide activity info to the stdout
* In detach mode we don't see stdout
* docker logs allows us to access these logs

----

## Execution on running container

```
docker exec -it mypgdb /bin/bash
```

* Sometimes in comes in handy to execute commands on a running container
* Usefull for troubleshooting

----

## Cleaning up

* Cleaning dead containers

```
docker ps -qf status=exited | xargs docker rm
```

* Cleaning intermidiate images

```
docker images -qf dangling=true | xargs docker rmi
```

----

## Exercise 1 (10 min)

* Create a postgres:9.5 database named dockerws with *NO* persistent volume storage
* Create a table and insert some data
* Destroy the container and recreate it
* Check that the database data dissapeared
* Now give it a named volume and check the data persists performing same steps
* [Postgres Official Image](https://hub.docker.com/_/postgres/)

----

## Solution 1

```
docker run --rm --name ej01 -p 5423:5432 \
           -e POSTGRES_DB=dockerws -d postgres:9.5
psql -p 5423 -h localhost -d dockerws -U postgres <<SQL
CREATE TABLE test(value integer);
INSERT INTO test SELECT generate_series(0, 100);
SQL
docker kill ej01
docker run --rm --name ej01 -p 5423:5432 \
           -e POSTGRES_DB=dockerws -d postgres:9.5
psql -p 5423 -h localhost -d dockerws -U postgres \
     -c "SELECT count(*) FROM test"
```

<pre>
ERROR:  relation "test" does not exist
</pre>

```
docker run --rm --name ej01 -p 5423:5432 \
           -v ej01data:/var/lib/postgresql/data \
           -e POSTGRES_DB=dockerws -d postgres:9.5
```

---

# Docker Advanced CLI

----

## Creating images

Building images basing it on other image

```docker
FROM ubuntu:12.04
RUN apt-get update -y && apt-get install -y netcat
EXPOSE 3333
CMD ["netcat", "-l", "-p", "3333", "-c", "echo Welcome to docker!!"]
```

Paste that into a file named Dockerfile

```
docker build -t myfirstimg .
docker run --rm -d -p 3333:3333 --name myfirstrun myfirstimg
curl localhost:3333
```

----

## Exercise 2 (10 min)

* Create a image myfirstimg based on ruby:2.3.1
* The image provides a webserver in rack that returns md5 of query parameter named data.
* Expected result
```
curl localhost:9292?data=3
# eccbc87e4b5ce2fe28308fd9f2a7baf3
```
* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

----

## Exercise 2 (Part 2)

```
require 'digest'
class App
  def call(env)
    req = Rack::Request.new(env)
    data = req.params["data"]
    content_type = {'Content-Type' => 'text/plain'}
    if data.nil?
      ['400', content_type, ["Missing data"]]
    else
      ['200', content_type, [Digest::MD5.hexdigest(data)]]
    end
  end
end
run App.new
```

----

## Solution

```
# Dockerfile
FROM ruby:2.3.1
RUN gem install rack
COPY config.ru /app/config.ru
EXPOSE 9292
CMD ["rackup", "-o", "0.0.0.0", "/app/config.ru"]
```

```
docker build -t ej02 .
docker run --rm -p 9292:9292 --name ej02 ej02
```

----

## Linking containers

You can connect containers an access them by host name

----

A simple script to connect to a db

```ruby
# script.rb
require 'pg'
conn = PG.connect( dbname: 'testdb', user: 'postgres',
                   password: 'secret', host: 'db' )
conn.exec( "SELECT * FROM pg_stat_activity" ) do |result|
  puts "     PID | User             | Query"
  result.each do |row|
    puts " %7d | %-16s | %s " %
      row.values_at('pid', 'usename', 'query')
  end
end
```

----

Let's create a db to run our script

```
# Dockerfile
FROM ruby:2.3.1
RUN gem install pg
COPY script.rb /script.rb
CMD ["ruby", "/script.rb"]
```

```
docker run --rm -e POSTGRES_PASSWORD=secret \
                -e POSTGRES_DB=testdb -d \
                --name testdb postgres:9.5
docker build -t rbstat .
docker run --rm --link testdb:db rbstat
```
<pre>
PID | User             | Query
116 | postgres         | SELECT * FROM pg_stat_activity
</pre>

----

## Exercise 3 (10 min)

* Create a nginx proxy that redirects to our previous web server (md5)
* The web server is accessible only from nginx
* Nginx can be accessed from the host

----

## Solution

```
# nginx.conf
upstream service {
  server proxyto:9292;
}

server {
    listen       80;
    server_name  localhost;

    location / {
      proxy_pass http://service;
    }
}

```

```
docker run --rm -d --name ej02 ej02
docker run --rm -d -p 8080:80 \
           -v $PWD/nginx.conf:/etc/nginx/conf.d/default.conf\
           --link ej02:proxyto nginx
curl localhost:8080?data=3
```

----

## Other interesting docker commands

* docker volume
* docker inspect
* docker cp
* docker stats
* docker tag

---

# Docker Compose

----

## What?

* As the number of containers grow managment with cli becomes unbearable
* Docker Compose allows us to define and run multi-container applications
* The service definition is handled in a file
* Docker commands have a compose definition equavalency

----

## Docker Compose File

Rewrite of rbstat example

```yaml
# docker-compose.yml
version: '2'
services:
  testdb:
    image: postgres:9.5
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: testdb
  rbstat:
    build: ../example02
    links:
      - testdb:db
```

```
docker-compose run --rm rbstat
```

----


## Exercise 4 (10 min)

* Rewrite exercise 3 using docker compose
* Start service with docker-compose up -d
* [Docker compose file reference](https://docs.docker.com/compose/compose-file/compose-file-v2/)

----

## Solution

```yaml
# docker-compose.yml
version: '2'
services:
  web:
    image: nginx
    volumes:
      - "./nginx.conf:/etc/nginx/conf.d/default.conf:ro"
    ports:
      - "8080:80"
    links:
      - server:proxyto
  server:
    build: ../ej02
```

```
docker-compose up -d
```

---

# Rails with Docker

----

## Getting started

Initialize the application
```
docker run --rm -v $PWD:/usr/src/myapp:Z \
           --user $(id -u):$(id -g) \
           -it  ruby:2.3.1 /bin/bash
$> gem install rails -v 5.0.1
$> rails new /usr/src/myapp -T -d postgresql
$> exit
```

----

## Dockerfile

```
# server.Dockerfile
FROM ruby:2.3.1

RUN apt-get update \
    && apt-get install -y --no-install-recommends nodejs \
    && rm -rf /var/lib/apt/lists/*

# throw errors if Gemfile has been modified since Gemfile.lock
RUN bundle config --global frozen 1

# Define where our application will live inside the image
ENV RAILS_ROOT /usr/src/app

RUN mkdir -p $RAILS_ROOT
WORKDIR $RAILS_ROOT

COPY Gemfile .
COPY Gemfile.lock .
RUN bundle install

COPY . .
EXPOSE 3000
CMD ["rails", "s", "-b", "0.0.0.0"]
```

----

## Compose

```
version: '2'
services:
  server:
    build:
      context: .
      dockerfile: server.Dockerfile
    ports:
      - 3000:3000
    volumes:
      - .:/usr/src/app:Z
    links:
      - db
  db:
    image: postgres:9.5
    env_file: .env
    volumes:
      - wsdbdata:/var/lib/postgresql/data
volumes:
  wsdbdata:
```

----

## Invoking rails commands

```
# OSX/Windows users will want to remove
# --­­user "$(id -­u):$(id -­g)"
docker-compose run --user $(id -u):$(id -g) \
                   server rails g controller Pages home
```

----

## Exercise 5 (20 minutes)

```
git clone \
  git@github.com:nico-acidtango/docker_workshop_20170421_rails.git
```

* Add redis instance
* Use sidekiq to execute FactorialController#compute on background

```
# unfreeze bundle
bundle config --global frozen 0
```

----

## Solution

* on the sidekiq branch

---

# That's all!
