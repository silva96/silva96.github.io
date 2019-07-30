---
layout: post
title: Elixir Phoenix development using Docker and Docker-Compose
author: silva96
categories: [elixir, phoenix, docker, docker-compose]
image: assets/images/elixir-phoenix-development-using-docker.jpg
---
Quick and dirty, this is my recipe for local development of elixir phoenix apps using docker and docker-compose.

3 important files:

- Dockerfile
- docker-compose.yml
- dev.exs

### Dockerfile

```docker
FROM bitwalker/alpine-elixir-phoenix:latest

# Cache elixir deps
ADD mix.exs mix.lock ./
RUN mix do deps.get, deps.compile

# Same with npm deps
ADD assets/package.json assets/
RUN cd assets && npm install

CMD ["mix", "phx.server"]
```


### docker-compose.yml
```yml
version: '3.7'
services:
  web:
    build: '.'
    ports:
      - "4000:4000"
    volumes:
      - /opt/app/assets/node_modules
      - /opt/app/deps
      - .:/opt/app
    depends_on:
      - db
    environment:
      MIX_ENV: 'dev'
  db:
    image: postgres:alpine
    ports:
      - 5432:5432
```

here we mount 3 volumes, the first two hold the deps so we don't recompile elixir deps or install npm packages every time we build or run the service. The last volume is our app itself, this way, changes reflect immediately without rebuilding or restarting the service.

### dev.exs

```elixir
config :yourapp, Yourapp.Repo,
  username: "postgres",
  password: "postgres",
  database: "yourapp_dev",
  hostname: "db",
  show_sensitive_data_on_connection_error: true,
  pool_size: 10
```

Just make sure to override the hostname with `db` and use the default user and password for postgres.

after this you can do `docker-compose build` and the `docker-compose up` and head to [http://localhost:4000](http://localhost:4000) in your browser.

if you want to enter to the container that is running:

`docker container ls -a`

Find the one with the _web suffix, mine is `myapp_web`

A one liner to enter to it would be something like this:

`docker exec -it $(docker ps | grep 'myapp_web' | awk '{print $1}') /bin/bash`
