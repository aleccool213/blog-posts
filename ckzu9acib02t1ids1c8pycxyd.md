## Running SQL Migrations Before Booting Docker Compose Services

Having a great local development experience is critical to happy engineers. Developers are happy to come into a codebase and start hacking away at problems, as opposed to dreading picking up their laptops. As services become more and more separated into separate "micro-services", new problems arise. When I look back on my short career, one problem that has induced me a lot of pain are SQL Migrations and how they should work in Docker Compose. Ideally, they should run before your web apps boot so they have access to a well-structured database. Your strategy to make this happen may be different for each environment, in this post I'll cover your local experience which easily extends to CI and I'll also touch how to do this in production.

## A Bit of Background

I ran into this problem when we started to adopt [Apollo Federation](https://www.apollographql.com/blog/apollo-federation-f260cf525d21/) into our stack. One gateway service lives in front of our other GraphQL services (which have their own databases). This gateway is what the client-side apps ("Web Browser" in diagram) queries to get its data. For clients to start using the gateway API every dependent service must be up and healthy.

![Service diagram](https://cdn.hashnode.com/res/hashnode/image/upload/v1645300243051/ycyzevgLS.png)

My goal is to start the entire set of services in a deterministic manner, have each service wait until its dependent service is running and healthy. I'll focus on the database and migrations in this post but the concepts here extend to any service having dependencies on another service.

A typical Docker Compose file which runs the products service with it's migrations will look like this:

```yaml
version: "3.7"

postgres:
  image: yourorg/postgres
  command: postgres -c 'max_connections=1024'
  expose:
    - "5432"
  environment:
    POSTGRES_USER: ${DATABASE_USERNAME}
    POSTGRES_PASSWORD: ${DATABASE_PASSWORD}
  volumes:
    - ${YOUR_ORG_INSTALL_PATH}/volumes/services/postgres-data:/var/lib/postgresql/data:delegated

products-run-migrations:
  build:
    context: ./apps/products/migrations/.
  image: yourorg/products-run-migrations:${IMAGE_TAG:-latest}
  env_file:
    - ./apps/products/products.env
  depends_on:
    - postgres

products:
  build:
    context: ./apps/products/.
  image: yourorg/products:${IMAGE_TAG:-latest}
  env_file:
    - ./apps/products/products.env
  depends_on:
    - postgres
    - products-run-migrations
```

A few issues with this setup, focusing on `depends_on`:

1. The `products-run-migrations` script will run right when `postgres` starts to run, not when it's healthy and ready to accept connections. There is a 99% chance that the `postgres` container will not be healthy when the migrations begin to run, causing them to fail.
2. Similar situation with the `products` service, it will run right when `products-run-migrations` starts to run. If you query the `products` service when it's healthy, there is a good chance the migrations have not yet completed.

## What We Want

This is currently not what we want, ideally developers should be able to start the products service and when it's healthy, know that the migrations ran successfully and they can query its API.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1645300244825/MM9TLLexM.gif)

This required me to do some research.

I found out there is a `conditions` argument to `depends_on` I could use to achieve the implementation we wanted. But the next thing I found out is where things went haywire.

**We needed to downgrade our docker-compose version.**

[Turns out, Docker Compose 3.x versions are meant to be used for Docker Swarm and Kubernetes environments where services are not strictly dependent on each other.](https://github.com/peter-evans/docker-compose-healthcheck/issues/3) This promotes a more fault-tolerant environment where services run independently.

[What I saw people suggesting](https://peterevans.dev/posts/how-to-wait-for-container-x-before-starting-y/) was to switch to Docker Compose 2.4 and use a port waiting script like [wait-for-it](https://github.com/vishnubob/wait-for-it).

Our new Docker Compose 2.4 file:

```yaml
version: '2.4'

postgres:
  image: yourorg/postgres
  command: postgres -c 'max_connections=1024'
  expose:
    - '5432'
  environment:
    POSTGRES_USER: ${DATABASE_USERNAME}
    POSTGRES_PASSWORD: ${DATABASE_PASSWORD}
  volumes:
    - ${YOUR_ORG_INSTALL_PATH}/volumes/services/postgres-data:/var/lib/postgresql/data:delegated
  healthcheck:
    test: ['CMD-SHELL', 'pg_isready -U root']
    interval: 60s

products-run-migrations:
  build:
    context: ./apps/products/migrations/.
  image: yourorg/products-run-migrations:${IMAGE_TAG:-latest}
  entrypoint:
      - /bin/bash
      - -c
      - 'wait-for-it $$DATABASE_HOSTNAME:5432 -s -t 60 -- npm run db-migrate:products
  restart: on-failure:5
  env_file:
    - ./apps/products/products.env
  depends_on:
      postgres:
        condition: service_healthy

products:
  build:
    context: ./apps/products/.
  image: yourorg/products:${IMAGE_TAG:-latest}
  env_file:
    - ./apps/products/products.env
  restart: on-failure:5
  depends_on:
      postgres:
        condition: service_healthy
      products-run-migrations:
        condition: service_started
```

A few new additions:

1. Added a `healthcheck` to the `postgres` container. This made it so migration scripts like `products-run-migrations` can start to run only when the database is ready to accept connections.
2. We added an `entrypoint` along with the `condition` arg to `depends_on` to our migration images. This made sure that not only the database is running but that the port is returning a 200 response code.
3. Added `condition` args to both `depends_on` in the `products` service definition.

It's not perfect but works much better than before. The issue still remains of the app not waiting for the migrations to be fully finished before booting up. If they run relatively quickly, local developers may not run into problems. In CI, we have full control over the environment so we can add an extra command to skirt around this issue.

```yaml
# .circleci/config.yml
test-products:
  executor: node-docker
  steps:
    - test-project:
        project-name: products
        pre-test-command: docker-compose run -T products-run-migrations
        tests:
          - run-project-test:
              project-name: products
```

## The Issues That Still Remain

What I didn't touch on is what our production environment looks like in terms of Docker Compose. We decided to maintain two versions of our Docker Compose files, one in 2.4 and one in 3.7. This is because we want to adopt Kubernetes easily in the future. You may stick with one or the other but for a great local development experience, we decided to always have a 2.4 file going.
