# How Compose works

https://docs.docker.com/compose/intro/compose-application-model/

With Docker Compose you use a YAML configuration file, known as the Compose file, to configure your application’s services, and then you create and start all the services from your configuration with the Compose CLI.

The Compose file, or `compose.yaml` file, follows the rules provided by the Compose Specification in how to define multi-container applications. This is the Docker Compose implementation of the formal Compose Specification.

## the compose application model

Computing components of an application are defined as ***services***. A **service** is an abstract concept implemented on platforms by running the same container image and configuration, one or more times.

Services communicate with each other through networks. In the Compose Specification, ***a network is a platform capability abstraction to establish an IP route between containers within services connected together***.

Services store and share persistent data into **volumes**. The Specification describes such a persistent data as ***a high-level filesystem mount with global options***.

Some services require ***configuration data that is dependent on the runtime or platform***. For this, the Specification defines a dedicated configs concept. From a service container point of view, ***configs are comparable to volumes, in that they are files mounted into the container***. But the actual definition involves distinct platform resources and services, which are abstracted by this type.

A secret is ***a specific flavor of configuration data*** for sensitive data that should not be exposed without security considerations. ***Secrets are made available to services as files mounted into their containers***, but the platform-specific resources to provide sensitive data are specific enough to deserve a distinct concept and definition within the Compose specification.

**Note**: With ***volumes, configs and secrets*** you can have a simple declaration at the top-level and then add more platform-specific information at the service level.

A project is an individual deployment of an application specification on a platform. A project's name, set with the top-level `name` attribute, is used to group resources together and isolate them from other applications or other installation of the same Compose-specified application with distinct parameters. If you are creating resources on a platform, you must prefix resource names by project and set the label `com.docker.compose.project`.

Compose offers a way for you to set a custom project name and override this name, so that the same `compose.yaml` file can be deployed twice on the same infrastructure, without changes, by just passing a distinct name.

## The Compose file

The default path for a Compose file is `compose.yaml` that is placed in the working directory.

You can use **fragments** and **extensions** to keep your Compose file efficient and easy to maintain.

Multiple Compose files can be merged together to define the application model. The combination of YAML files is implemented by appending or overriding YAML elements based on the Compose file order you set. Simple attributes and maps get overridden by the highest order Compose file, lists get merged by appending.

If you want to ***reuse*** other Compose files, or ***factor out*** parts of your application model into separate Compose files, you can also use `include`. This is useful if your Compose application is dependent on another application which is managed by a different team, or needs to be shared with others.

## compose example

Consider an application split into a frontend web application and a backend service.

The frontend is configured at runtime with an HTTP configuration file managed by infrastructure, providing an external domain name, and an HTTPS server certificate injected by the platform's secured secret store.

The backend stores data in a persistent volume.

Both services communicate with each other on an isolated back-tier network, while the frontend is also connected to a front-tier network and exposes port 443 for external usage.

The example application is composed of the following parts:

- 2 services, backed by Docker images: webapp and database
- 1 secret (HTTPS certificate), injected into the frontend
- 1 configuration (HTTP), injected into the frontend
- 1 persistent volume, attached to the backend
- 2 networks

```yaml
services:
  frontend:
    image: example/webapp
    ports:
      - "443:8043"
    networks:
      - front-tier
      - back-tier
    configs:
      - httpd-config
    secrets:
      - server-certificate

  backend:
    image: example/database
    volumes:
      - db-data:/etc/data
    networks:
      - back-tier

volumes:
  db-data:
    driver: flocker
    driver_opts:
      size: "10GiB"

configs:
  httpd-config:
    external: true

secrets:
  server-certificate:
    external: true

networks:
  # The presence of these objects is sufficient to define them
  front-tier: {}
  back-tier: {}
```

The `docker compose up` command starts the `frontend` and `backend` services, create the necessary networks and volumes, and injects the configuration and secret into the frontend service.

`docker compose ps` provides a snapshot of the current state of your services, making it easy to see which containers are running, their status, and the ports they are using:

```
docker compose ps

NAME                IMAGE                COMMAND                  SERVICE             CREATED             STATUS              PORTS
example-frontend-1  example/webapp       "nginx -g 'daemon ofâ¦"   frontend            2 minutes ago       Up 2 minutes        0.0.0.0:443->8043/tcp
example-backend-1   example/database     "docker-entrypoint.sâ¦"   backend             2 minutes ago       Up 2 minutes
```

# Services top-level elements

A service is an abstract definition of a ***computing resource*** within an application which can be **scaled or replaced independently** from other components. Services are backed by a set of containers, run by the platform according to replication requirements and placement constraints. As ***services are backed by containers***, they are ***defined by a Docker image and set of runtime arguments***. All containers within a service are identically created with these arguments.

A Compose file must declare a `services` top-level element as a map whose keys are string representations of service names, and whose values are service definitions. A service definition contains the configuration that is applied to each service container.

## Simple example

The following example demonstrates how to define two simple services, set their images, map ports, and configure basic environment variables using Docker Compose.

```compose

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"

  db:
    image: postgres:13
    environment:
      POSTGRES_USER: example
      POSTGRES_DB: exampledb
```


---

Docker Compose provides a structured and streamlined approach for managing multi-container deployments. With Docker Compose, you don’t need to run multiple `docker run` commands. All you need to do is define your entire multi-container application in a single YAML file called `compose.yaml`.

---

Use the following command in a terminal to clone the sample application repository.

```
git clone https://github.com/dockersamples/nginx-node-redis
```

Navigate into the `nginx-node-redis` directory. Inside this directory, you'll find a file named `compose.yml`. This YAML file is where all the magic happens. It defines **all the services that make up your application**, along with their **configurations**. Each service specifies its image, ports, volumes, networks, and any other settings necessary for its functionality.

Use the `docker compose up` command to start the application:

```
docker compose up -d --build
```

---

If you remove web1 or web2 and referesh the webpage you see incrementation with only one of them, as opposed to earlier, which was with both.

---

Nginx is typically used as a reverse proxy for web applications, routing traffic to backend servers. In this case, it routes to the Node.js backend containers (web1 or web2).

You might have noticed that Nginx, acting as a reverse proxy, likely distributes incoming requests in a round-robin fashion between the two backend containers. This means each request might be directed to a different container (web1 and web2) on a rotating basis. The output shows consecutive increments for both the web1 and web2 containers and the actual counter value stored in Redis is updated only after the response is sent back to the client.