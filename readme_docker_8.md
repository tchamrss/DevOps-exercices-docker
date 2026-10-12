# Docker & Docker Compose Cheat Sheet

A practical, structured guide covering core **Docker CLI operations**, container networking, data volumes, **private registry handling (Nexus)**, and **Docker Compose multi-container orchestration**.

---

## 1. Installation & Container Lifecycle Management

### Installing Docker
To install Docker via Snap on a Linux host system:
```bash
sudo snap install docker
```

### Core Operations & Lifecycle Commands
* **Pull an image:** Fetch an image from Docker Hub without running it.
  ```bash
  docker pull redis
  ```
* **Run a new container:** Automatically pull (if missing) and start a container.
  ```bash
  # Start Postgres with an environment variable
  docker run -e POSTGRES_PASSWORD=mysecretpassword -d postgres

  # Start a specific version of Redis in the foreground
  docker run redis:8.4
  ```
* **Run in Detached Mode (`-d`):** Run the container in the background to free up the terminal.
  ```bash
  docker run -d redis
  ```
* **Start vs. Run:** 
  * `docker run` creates a *brand new container instance* from an image and boots it.
  * `docker start` turns on an *existing, stopped container*.
  ```bash
  docker start <CONTAINER_ID>
  ```
* **Stop containers:** Stop one or multiple running containers at once.
  ```bash
  docker stop <CONTAINER_ID>
  docker stop <CONTAINER_ID_1> <CONTAINER_ID_2>
  ```
* **Delete resources:** Clean up system storage.
  ```bash
  docker rm <CONTAINER_ID>    # Remove a stopped container
  docker rmi <IMAGE_ID>       # Remove a local image
  ```

---

## 2. Inspections & Diagnostics

* **List running containers:** Shows running containers, their IDs, images, and ports.
  ```bash
  docker ps
  ```
* **List all containers:** Shows all containers on the system (running, paused, or exited). Use this to find the IDs of stopped containers.
  ```bash
  docker ps -a
  ```
* **List local images:** Shows all images currently stored locally on the host.
  ```bash
  docker images
  ```
* **View container logs:** Audit standard output logs for troubleshooting.
  ```bash
  docker logs <CONTAINER_ID_OR_NAME>
  ```
* **Inspect resource details:** Extract deep configuration mapping data (including volume storage paths).
  ```bash
  docker inspect nexus-data
  ```
* **Execute commands inside a container (`exec`):** Open an interactive bash terminal inside a running container to check configurations and environment parameters.
  ```bash
  docker exec -it <CONTAINER_ID> /bin/bash
  
  # Type 'exit' to leave the container terminal
  exit
  ```

---

## 3. Advanced Networking & Naming

### Naming a Container
Assign a custom, readable name to a container using the `--name` flag:
```bash
docker run -d -p 6001:6379 --name redis-older redis:8.4
```

### Port Mapping (`-p`)
Forward network traffic from your laptop/host machine to the internal container ports. Format: `-p <HOST_PORT>:<CONTAINER_PORT>`
```bash
# Bind Redis container default port (6379) to the host port (6000)
docker run -p 6000:6379 redis

# Bind a different Redis container instance to the host port (6001)
docker run -p 6001:6379 redis:8.4
```

### Volume Inspections
List all persistent volumes managed by Docker on the host machine:
```bash
docker volume ls
```

---

## 4. Working with Private Registries (Nexus Example)

### Authentication & Configs
When you log into a private docker registry, authentication tokens are generated and stored locally:
```bash
cat ~/.docker/config.json
```

### Log In, Tag, and Push Workflow
All target infrastructure IP addresses in this workflow are safely masked using `###.###.###.###`.

1. **Log into the remote registry:**
   ```bash
   docker login ###.###.###.###:8083
   ```
2. **Tag your local image:** Assign a registry destination identifier prefix to match your registry location.
   ```bash
   docker tag my-app:1.0 ###.###.###.###:8083/my-app:1.0
   ```
3. **Push the image to your repository:**
   ```bash
   docker push ###.###.###.###:8083/my-app:1.0
   ```
4. **Query components via API:** Verify the pushed artifacts using a curl request.
   ```bash
   curl -u ruru:###### -X GET '###.###.###.###:8081/services/rest/v1/components?repository=docker-hosted'
   ```

---

## 5. Docker Compose Multi-Container Orchestration

Docker Compose lets you define and run multi-container applications using a single YAML configuration file.

### Complete `docker-compose.yml` Template
Create a file named `docker-compose.yml` in your project folder:

```yaml
version: '3.8'

services:
  # Service 1: PostgreSQL Database
  database:
    image: postgres:15
    container_name: production-db
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: mysecretpassword
      POSTGRES_DB: app_data
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data

  # Service 2: Redis Cache
  cache:
    image: redis:8.4
    container_name: production-cache
    restart: unless-stopped
    ports:
      - "6000:6379"

  # Service 3: Web Application
  web-app:
    image: ###.###.###.###:8083/my-app:1.0
    container_name: web-frontend
    restart: always
    ports:
      - "80:8080"
    depends_on:
      - database
      - cache
    environment:
      - DB_HOST=database
      - CACHE_HOST=cache

volumes:
  db_data:
```

### Crucial Docker Compose CLI Commands
Run these commands in the exact same directory where your `docker-compose.yml` file is located:

* **Start all services:** Automatically builds, creates, and launches all defined containers in background detached mode.
  docker compose up -d
* **Stop all services:** Stops and safely removes the containers and internal networks created by `up` (leaves your volume data untouched).
  docker compose down
* **View orchestration logs:** View combined, live-streaming log aggregation from all grouped services simultaneously.
  docker compose logs -f
* **Check runtime states:** List the execution metrics and operational status of all services tracked by the compose file.
  docker compose ps
