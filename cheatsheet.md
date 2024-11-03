
```
apt-get --help
```

# General Commands

Start the docker daemon

```
docker -d
```

Get help with Docker. Can also use `--help` on all subcommands

```
docker --help
```

Display system-wide information

```
docker info
```

# images

Build an Image from a Dockerfile

```
docker build -t <image_name>
```

Build an Image from a Dockerfile without the cache

```
docker build -t <image_name> . --no-cache
```

List local images

```
docker images
```

Delete an Image

```
docker rmi <image_name>
```

Remove all unused images

```
docker image prune
```

# Docker Hub

Login into Docker

```
docker login -u <username>
```

Publish an image to Docker Hub

```
docker push <username>/<image_name>
```

Search Hub for an image

```
docker search <image_name>
```

Pull an image from a Docker Hub

```
docker pull <image_name>
```

# Containers

Create and run a container from an image, with a **custom** name:

```
docker run --name <container_name> <image_name>
```

Run a container with and publish a container’s port(s) to the host.

```
docker run -p <host_port>:<container_port> <image_name>
```

Run a container in the background

```
docker run -d <image_name>
```

Start or stop an existing container:

```
docker start|stop <container_name> (or <container-id>)
```

Remove a stopped container:

```
docker rm <container_name>
```

Open a shell inside a running container:

```
docker exec -it <container_name> sh
```

Fetch and follow the logs of a container:

```
docker logs -f <container_name>
```

To inspect a running container:

```
docker inspect <container_name> (or <container_id>)
```

To list currently running containers:

```
docker ps
```

List all docker containers (running and stopped):

```
docker ps --all
```

View resource usage stats

```
docker container stats
```

# compose

To start all the services defined in your `compose.yaml` file:

```
docker compose up
```

To stop and remove the running services:

```
docker compose down 
```

If you want to monitor the output of your running containers and debug issues, you can view the logs with:

```
docker compose logs
```

To list all the services along with their current status:

```
docker compose ps
```

https://docs.docker.com/reference/cli/docker/compose/#subcommands

# image

To view an image's labels, use the `docker image inspect` command. You can use the `--format` option to show just the labels:

```
docker image inspect --format='{{json .Config.Labels}}' myimage
```

# Use a bind mount


---

---

---

In a multi-stage Dockerfile, the final stage is the default target for building. This means that if you don't explicitly specify a target stage using the `--target` flag in the `docker build` command, Docker will automatically build the last stage by default.

**Example**:

```
docker build -t <name:tag> --target <target_build_stage> .
```

---

Look at the image size difference by using the `docker images` command:

```
docker images
```

---

Define the working directory by using the `WORKDIR` instruction. This will specify where future commands will run and the directory files will be copied inside the container image.

```Dockerfile
WORKDIR /app
```

---

`COPY`: Copy files into the current working directory within the Docker container.

---

Copy the `src` directory from your project on the host machine to the working directory within the container.

```Dockerfile
COPY src ./src
```

---

`RUN`: Execute a command within the container.

---

`CMD`: Set the default command to be executed when the container starts.

---


---

# Use Docker Compose

1. Create a new directory and inside that directory, create a `compose.yaml` file with the following contents:

```yaml
services:
  app:
    image: docker/welcome-to-docker
    ports:
      - 8080:80
```

2. Open a terminal and navigate to the directory you created in the previous step.

3. Use the `docker compose up` command to start the application.

4. Open your browser to http://localhost:8080.

# Managing volumes

Volumes have their own lifecycle beyond that of containers and can grow quite large depending on the type of data and applications you’re using. The following commands will be helpful to manage volumes:

list all volumes:

```
docker volume ls
```

remove a volume (only works when the volume is not attached to any containers):

```
docker volume rm <volume-name-or-id>
```

remove all unused (unattached) volumes:

```
docker volume prune
```

---

The `-f` will stop the container first and then remove it:

```
docker rm -f <the-container>
```
