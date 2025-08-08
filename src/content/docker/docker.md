## Install

[Docker Engine](https://docs.docker.com/engine/install/ubuntu/).

## Commands

### Build image.

```bash
docker build -t getting-started .
```

- `-t` tags our image. A human-readable name for the final image. We can refer to that image when we run a container using that name.
- `.` at the end of the docker build command tells Docker that it should look for the Dockerfile in the current directory.

[Link](https://docs.docker.com/get-started/02_our_app/).

### Run container.

```bash
docker run --rm -d -p 80:80 docker/getting-started
```

- `--rm` automatically remove the container when it exits.
- `-d` run the container in detached mode (in the background).
- `-p 80:80` map port 80 of the host to port 80 in the container. Without the port mapping, we wouldn’t be able to access the application.
- `docker/getting-started` the image to use.

The application is accessible at <http://localhost:80>.

[Link example](https://docs.docker.com/get-started/).

[Link run command](https://docs.docker.com/engine/reference/commandline/run/).

Use a volume mount.

```bash
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```

- `-v` mount the todo-db volume to /etc/todos, which will capture all files created at the path.

[Link](https://docs.docker.com/get-started/05_persisting_data/).

### List containers

```bash
docker ps -a
```

- `-a` show not running containers too.

### Interact with a container

Start an interactive shell:

```bash
docker exec -it $(DOCKER_CONTAINER_NAME) /bin/bash
```

Run command on a running container.

```bash
docker exec <container-id> cat /data.txt
```

Run command on a new container.

```bash
docker run -it ubuntu ls /
```

[Link](https://docs.docker.com/get-started/05_persisting_data/).

### Container logs

```bash
docker logs -f <container-id>
```

[Link](https://docs.docker.com/get-started/06_bind_mounts/).

### Stop and remove a container

First stop and after that, remove.

```bash
docker stop <the-container-id>
docker rm <the-container-id>
```

Stop and remove with only one command.

```bash
docker rm -f <the-container-id>
```

[Link](https://docs.docker.com/get-started/03_updating_app/).

### Remove image

Image with tag none ([link](https://stackoverflow.com/a/33913711)):

```bash
docker rmi $(docker images --filter "dangling=true" -q --no-trunc)
```

### Volumes

#### Create named volume

```bash
docker volume create todo-db
```

[Link](https://docs.docker.com/get-started/05_persisting_data/).

#### Inspect named volume

```bash
docker volume inspect todo-db
```

- Output `Mountpoint`is the actual location on the disk where the data is stored.

[Link](https://docs.docker.com/get-started/05_persisting_data/).

#### Create bind mount

```bash
docker run -dp 3000:3000 \
     -w /app -v "$(pwd):/app" \
     node:12-alpine \
     sh -c "yarn install && yarn run dev"
```

- `-w /app` sets the “working directory” or the current directory that the command will run from.
- `-v "$(pwd):/app"` bind mount the current directory from the host in the container into the /app directory.
- `node:12-alpine` the image to use. Note that this is the base image for our app from the Dockerfile.
- `sh -c "yarn install && yarn run dev"` - the command. We’re starting a shell using sh (alpine doesn’t have bash) and running yarn install to install all dependencies and then running yarn run dev. If we look in the package.json, we’ll see that the dev script is starting nodemon.

[Link](https://docs.docker.com/get-started/06_bind_mounts/).

## Volumes

### Named volume vs Bind volume

- Named volumes are great if we simply want to store data, as we don’t have to worry about where the data is stored.
- Bind mounts, we control the exact mountpoint on the host. We can use this to persist data, but it’s often used to provide additional data into containers. When working on an application, we can use a bind mount to mount our source code into the container and work with it like at our local host.
- [Comparison table](https://docs.docker.com/get-started/06_bind_mounts/#quick-volume-type-comparisons).

[Link](https://docs.docker.com/get-started/06_bind_mounts/).

