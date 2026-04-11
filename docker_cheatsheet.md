# Docker Cheat Sheet for Data Engineering

A practical reference for Docker commands and patterns commonly used in data engineering workflows.

---

## Images

### `docker pull`
Download an image from Docker Hub.

| Command | What it does |
|---------|--------------|
| `docker pull python:3.11` | Pull a specific Python image |
| `docker pull postgres:15` | Pull a specific Postgres image |

---

### `docker build`
Build an image from a Dockerfile.

| Command | What it does |
|---------|--------------|
| `docker build -t my_image .` | Build and tag an image from current directory |
| `docker build -t my_image:v1 .` | Build with a version tag |
| `docker build -f Dockerfile.prod -t my_image .` | Build using a specific Dockerfile |
| `docker build --no-cache -t my_image .` | Build without using cached layers |

---

### `docker images`
List all local images.

| Command | What it does |
|---------|--------------|
| `docker images` | List all images |
| `docker images -a` | Include intermediate images |
| `docker rmi image_name` | Delete an image |
| `docker image prune` | Remove all unused images |

---

## Containers

### `docker run`
Create and start a container from an image.

| Command | What it does |
|---------|--------------|
| `docker run my_image` | Run a container |
| `docker run -it my_image bash` | Run interactively with a terminal |
| `docker run -d my_image` | Run in the background (detached) |
| `docker run --name my_container my_image` | Give the container a name |
| `docker run -p 8080:80 my_image` | Map host port 8080 to container port 80 |
| `docker run -v /host/path:/container/path my_image` | Mount a host folder into the container |
| `docker run -e MY_VAR=value my_image` | Pass an environment variable |
| `docker run --env-file .env my_image` | Pass environment variables from a file |
| `docker run --rm my_image` | Auto-remove container when it exits |

---

### `docker ps`
List containers.

| Command | What it does |
|---------|--------------|
| `docker ps` | List running containers |
| `docker ps -a` | List all containers including stopped ones |

---

### `docker stop` / `docker start` / `docker restart`
Control container state.

| Command | What it does |
|---------|--------------|
| `docker stop container_name` | Gracefully stop a running container |
| `docker start container_name` | Start a stopped container |
| `docker restart container_name` | Restart a container |
| `docker kill container_name` | Force stop immediately |

---

### `docker rm`
Remove containers.

| Command | What it does |
|---------|--------------|
| `docker rm container_name` | Remove a stopped container |
| `docker rm -f container_name` | Force remove a running container |
| `docker container prune` | Remove all stopped containers |

---

### `docker exec`
Run a command inside a running container.

| Command | What it does |
|---------|--------------|
| `docker exec -it container_name bash` | Open a terminal inside the container |
| `docker exec container_name python script.py` | Run a script inside the container |

---

### `docker logs`
View output from a container.

| Command | What it does |
|---------|--------------|
| `docker logs container_name` | Show all logs |
| `docker logs -f container_name` | Follow logs in real time |
| `docker logs --tail 100 container_name` | Show last 100 lines |

---

## Volumes

### `docker volume`
Manage persistent storage that survives container restarts.

| Command | What it does |
|---------|--------------|
| `docker volume create my_volume` | Create a named volume |
| `docker volume ls` | List all volumes |
| `docker volume inspect my_volume` | Show volume details |
| `docker volume rm my_volume` | Delete a volume |
| `docker volume prune` | Remove all unused volumes |

---

## Networks

### `docker network`
Manage container networking.

| Command | What it does |
|---------|--------------|
| `docker network create my_network` | Create a custom network |
| `docker network ls` | List all networks |
| `docker network inspect my_network` | Show network details |
| `docker network connect my_network container_name` | Connect a container to a network |

---

## Docker Compose

### `docker compose up`
Start all services defined in `docker-compose.yml`.

| Command | What it does |
|---------|--------------|
| `docker compose up` | Start all services |
| `docker compose up -d` | Start all services in the background |
| `docker compose up --build` | Rebuild images before starting |
| `docker compose up service_name` | Start only one specific service |

---

### `docker compose down`
Stop and remove containers, networks.

| Command | What it does |
|---------|--------------|
| `docker compose down` | Stop and remove containers and networks |
| `docker compose down -v` | Also remove volumes |
| `docker compose down --rmi all` | Also remove images |

---

### Other Compose commands

| Command | What it does |
|---------|--------------|
| `docker compose ps` | List running compose services |
| `docker compose logs -f` | Follow logs for all services |
| `docker compose exec service_name bash` | Open terminal in a running service |
| `docker compose restart service_name` | Restart a specific service |
| `docker compose build` | Rebuild images without starting |

---

## Dockerfile Reference

Common instructions used in a Dockerfile.

| Instruction | What it does |
|-------------|--------------|
| `FROM python:3.11` | Base image to build from |
| `WORKDIR /app` | Set the working directory inside the container |
| `COPY . .` | Copy files from host into the container |
| `RUN pip install -r requirements.txt` | Run a command during the build |
| `ENV MY_VAR=value` | Set an environment variable |
| `EXPOSE 8080` | Document which port the container listens on |
| `CMD ["python", "pipeline.py"]` | Default command to run when container starts |
| `ENTRYPOINT ["python"]` | Fixed executable — arguments are appended |

---

## ETL Patterns

### Run a one-off pipeline script and remove the container after
```bash
docker run --rm -v $(pwd)/data:/app/data my_pipeline python pipeline.py
```

### Pass database credentials securely via env file
```bash
docker run --env-file .env my_pipeline python pipeline.py
```

### Start Postgres locally for development
```bash
docker run -d \
  --name dev_postgres \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  postgres:15
```

### Start a full pipeline stack with Compose
```bash
docker compose up -d
docker compose logs -f pipeline
```

### Open a shell inside a running pipeline container to debug
```bash
docker exec -it my_pipeline bash
```

### Check how much disk Docker is using
```bash
docker system df
```

### Clean up everything unused (images, containers, volumes, networks)
```bash
docker system prune -a --volumes
```

### Copy a file out of a container to the host
```bash
docker cp container_name:/app/output/result.csv ./result.csv
```

---

*Last updated: 2026*
