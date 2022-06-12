In this post I'll share three methods how to enable different environments in Docker Compose and which of then I prefer most.

I use Docker Compose when I need to run multi-container Docker application.
Often I need to have different setups for different environments. 
For example, I want to set `debug` variable to `false` for a production environment and have an 
additional service for database administration for a development. 

Docker Compose is a great instrument, and it natively enables it in different ways.
But generally we need to define common and environment specific configurations and tell Compose
how to merge them.

### Extending using [multiple Compose files](https://docs.docker.com/compose/extends/#multiple-compose-files)

With this method we just define environment specific variables with the same service name across
different configs.

> If a service is defined in both files, Compose merges the configurations using the rules described in Adding and overriding configuration.

So if we have `docker-compose.yml`
```bash
version: "3.9"

services:

  db:
    image: postgres
    environment:
      - POSTGRES_PASSWORD

  app:
    image: python:3.9
    depends_on:
      - db
    ports:
      - 8080:8080
```

And `docker-compose.dev.yml`
```bash
services:

  db:
    ports:
      - 5432:5432

  app:
    environment:
      - DEBUG=true

  adminer:
    image: adminer
```

The merged services are the following:

```bash
$ docker-compose -f docker-compose.yml -f docker-compose.dev.yml config

services:
  adminer:
    image: adminer
  app:
    depends_on:
      db:
        condition: service_started
    environment:
      DEBUG: "true"
    image: python:3.9
    ports:
    - published: 8080
      target: 8080
  db:
    environment:
      POSTGRES_PASSWORD: null
    image: postgres
    ports:
    - published: 5432
      target: 5432
version: '3.9'
```

