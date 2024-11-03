# Publishing Ports

Publishing ports happens during container creation using the `-p` (or `--publish`) flag with `docker run`. The syntax is:

```
docker run -d -p HOST_PORT:CONTAINER_PORT nginx
```

- `HOST_PORT`: The port number **_on your host machine_** where you want to receive traffic

- `CONTAINER_PORT`: The port number **_within the container_** that's listening for connections

For example, to publish the container's port `80` to host port `8080`:

```
docker run -d -p 8080:80 nginx
```

Now, any traffic sent to port `8080` on your host machine will be forwarded to port `80` within the container.

The first `8080` refers to the host port. This is the port on your local machine that will be used to access the application running inside the container. The second `80` refers to the container port. This is the port that the application inside the container listens on for incoming connections. Hence, the command binds to port `8080` of the host to port `80` on the container system.

---

Start a container using the `httpd` image with the following command:

```
docker run -d -p 8080:80 --name my_site httpd:2.4
```

- `-d`, `--detach`: Run container in background and print container ID
- `-p`, `--publish list`: Publish a container's port(s) to the host

Open the browser and access http://localhost:8080 or use the `curl` command to verify if it's working fine or not.

```
curl localhost:8080
```
