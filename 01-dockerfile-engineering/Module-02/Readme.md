# 🏗️ Module 02: Advanced Compose Orchestration

This folder houses validated blueprints for production-grade multi-container orchestrations. Each file addresses a critical physical host infrastructure standard, focusing on state and persistence decoupling, strict resource governance, and proactive host storage protection.

### 📦 Prerequisites for Testing

For Scenario 1, manually create the external orchestration volume before initializing the stack:

```bash
docker volume create shared_scratch
```

---

## 🛡️ 1. State & Persistence: Isolated Named Volume Lifecycles

* **File:** `docker-compose.persistence.yml`
* **Concept:** Decouples persistent states from ephemeral application runtimes, leveraging distinct internal and external volume lifecycles to protect production application data across aggressive stack teardowns.

### 📄 Compose Configuration Content:

```yaml
version: '3.8'

services:
  database:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: app_db
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secure_password_here
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - backend_isolated
    restart: always

  app_worker:
    image: alpine:latest
    command: >
      sh -c "while true; do echo 'Processing ephemeral task...' >> /data/logs.txt; sleep 10; done"
    volumes:
      - shared_scratch:/data
    networks:
      - backend_isolated

networks:
  backend_isolated:
    driver: bridge

volumes:
  db_data:
    name: prod_postgres_persistent_volume
  
  shared_scratch:
    external: true
```

```bash
# Setup & data injection command:
docker compose -f docker-compose.persistence.yml up -d
docker compose -f docker-compose.persistence.yml exec database sh -c "echo 'CRITICAL_PROD_DATA' > /var/lib/postgresql/data/state.txt"

# Infrastructure teardown command (Aggressive purge):
docker compose -f docker-compose.persistence.yml down --volumes

# Verification command:
docker compose -f docker-compose.persistence.yml up -d
docker compose -f docker-compose.persistence.yml exec database cat /var/lib/postgresql/data/state.txt
docker volume ls | grep -E "prod_postgres_persistent_volume|shared_scratch"
```

📋 **Expected Reference Output:**

```text
CRITICAL_PROD_DATA
local     prod_postgres_persistent_volume
local     shared_scratch
```

> 💡 **Security Note:** If `state.txt` throws a file not found error, your volume lifecycles are conjoined with the container execution layer, risking state erasure!

---

## 🎛️ 2. Resource Governance: Hard Capping Cores and Memory Limits

* **File:** `docker-compose.governance.yml`
* **Concept:** Implements kernel-level cgroups boundaries directly within the orchestration manifest to eliminate host CPU starvation vectors and restrict runaway applications from causing a global Out-Of-Memory (OOM) host crash.

### 📄 Compose Configuration Content:

```yaml
version: '3.8'

services:
  heavy_processor:
    image: alpine:latest
    command: sleep 3600
    deploy:
      resources:
        limits:
          cpus: '1.5'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

```bash
# Setup & stress testing injection command:
docker compose -f docker-compose.governance.yml up -d heavy_processor
# Artificially force a heavy multi-core workload and allocate memory past the 512M cap
docker compose -f docker-compose.governance.yml exec heavy_processor sh -c "apk add --no-cache stress-ng && stress-ng --cpu 3 --vm 1 --vm-bytes 700M"

# Verification command (Execute in secondary terminal window during load test):
docker stats --no-stream heavy_processor
```

📋 **Expected Reference Output:**

```text
CONTAINER ID   NAME              CPU %     MEM USAGE / LIMIT   MEM %
a1b2c3d4e5f6   heavy_processor   150.00%   512MiB / 512MiB     100.00%
```

> 💡 **Architecture Note:** The container CPU utilization locks precisely at 150.00% (1.5 cores) and the application processing terminal cleanly throws an Out Of Memory error, proving host safety constraints applied successfully.

---

## 🩺 3. Storage Protection: High-Frequency Log Rotation Policies

* **File:** `docker-compose.logging.yml`
* **Concept:** Restricts stdout/stderr logging engine capture metrics per container to stop chatty, multi-container tracking payloads from saturating storage disk paths on the physical bare-metal hardware.

### 📄 Compose Configuration Content:

```yaml
version: '3.8'

services:
  chatty_service:
    image: alpine:latest
    command: >
      sh -c "while true; do echo \"[INFO] \$(date) - Generating high-frequency telemetry data stream\"; sleep 0.1; done"
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

```bash
# Setup command:
docker compose -f docker-compose.logging.yml up -d chatty_service
# Let the service loop freely for ~45 seconds to generate logging volume

# Verification command:
# Locate physical container log JSON tracking node path directly on the host engine
LOG_PATH=$(docker inspect --format='{{.LogPath}}' $(docker compose -f docker-compose.logging.yml ps -q chatty_service))

# View rotation architecture list
sudo ls -lh $(dirname $LOG_PATH)
```

📋 **Expected Reference Output:**

```text
-rw-r----- 1 root root  10M Oct  6 11:51 a1b2c3...json
-rw-r----- 1 root root  10M Oct  6 11:50 a1b2c3...json.1
-rw-r----- 1 root root  4.2M Oct  6 11:50 a1b2c3...json.2
```

> 💡 **Infrastructure Note:** You will notice at most 3 logging instances, with none exceeding 10M in size, confirming that background disk saturation risks have been effectively mitigated.

---

## 🛠️ 4. Combined Architecture Blueprint: The Unified Host Protection Stack

* **File:** `docker-compose.yml`
* **Concept:** Consolidates state persistence, rigid cgroups boundaries, and standard high-frequency log rotation parameters into a single production orchestration model.

### 📄 Compose Configuration Content:

```yaml
version: '3.8'

services:
  db:
    image: redis:7-alpine
    container_name: prod_redis_backend
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 256M
    volumes:
      - redis_state:/data
    logging:
      driver: "json-file"
      options:
        max-size: "5m"
        max-file: "2"
    networks:
      - core_network

  api:
    image: nginx:alpine
    container_name: prod_nginx_edge
    ports:
      - "8080:80"
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 512M
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "5"
    networks:
      - core_network

networks:
  core_network:
    driver: bridge

volumes:
  redis_state:
    name: redis_persistent_storage
```

```bash
# Execution Stack Command:
docker compose up -d
```

📋 **Expected Reference Output:**

```text
[+] Running 3/3
 ✔ Network project_core_network      Created
 ✔ Volume "redis_persistent_storage" Created
 ✔ Container prod_redis_backend      Started
 ✔ Container prod_nginx_edge         Started
```
