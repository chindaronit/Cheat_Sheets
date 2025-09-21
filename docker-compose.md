# 📘 Docker Compose Cheat Sheet

A quick reference for **Docker Compose** — useful commands, file structure, and advanced features.

### 🔹 What is Docker Compose?
- A tool for defining and running **multi-container Docker applications**.  
- Uses a YAML file (`docker-compose.yml`) to configure services, networks, and volumes.  
- Run everything with **one command**:  `docker-compose up`

### 🔹 Basic Commands

```bash
# Start services in foreground
docker-compose up

# Start in background (detached)
docker-compose up -d

# Stop all services
docker-compose down

# Stop services but keep volumes
docker-compose down --volumes

# Rebuild images
docker-compose up --build

# List services
docker-compose ps

# View logs
docker-compose logs -f  # -f stands for file

# Execute a command in a service
docker-compose exec <service_name> <command>

# Stop containers + remove containers + remove networks + remove volumes
docker-compose down -v
```

### 🔹 Basic docker-compose.yml Structure

```bash
version: "3.9"
name: app
services:
  app:
    image: myapp:latest
    build: .
    ports:
      - "8080:80"
    environment: # give env variable like this or through .env file
      - DEBUG=true
    volumes:
      - ./src:/app
    depends_on:
      - db

  db:
    image: mysql:8.0
    restart: unless-stopped
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```
### 🔹 Production ready compose file

```bash
version: "3.9"
name: app # not mandatory
services:
  # 🔹 Reverse Proxy
  nginx:
    image: nginx:latest
    container_name: reverse_proxy
    restart: always
    ports:
      - "${NGINX_HTTP_PORT}:80"
      - "${NGINX_HTTPS_PORT}:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      frontend:
        condition: service_healthy
      backend:
        condition: service_healthy
    networks:
      - frontend_net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/"]
      interval: 30s
      timeout: 10s
      retries: 5

  # 🔹 Frontend
  frontend:
    build: ./frontend
    container_name: frontend_app
    restart: always
    networks:
      - frontend_net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 5

  # 🔹 Backend
  backend:
    build: ./backend
    container_name: backend_app
    restart: always
    environment:
      DB_HOST: db
      DB_PORT: 3306
      DB_NAME: ${MYSQL_DATABASE}
      DB_USER: ${MYSQL_USER}
      DB_PASSWORD: ${MYSQL_PASSWORD}
    ports:
      - "${BACKEND_PORT}:5000"
    depends_on:
      db:
        condition: service_healthy
    networks:
      - frontend_net
      - db_net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 5

  # 🔹 Database
  db:
    image: mysql:8.0
    container_name: mysql_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - db_net
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-p${MYSQL_ROOT_PASSWORD}"]
      interval: 30s
      timeout: 10s
      retries: 5

volumes:
  db_data:

networks:
  frontend_net:
  db_net:
```
### 🔹 Example nginx.conf

```nginx
    events {}

    http {
        server {
            listen 80;

            # Frontend (default route)
            location / {
                proxy_pass http://frontend:80;
            }

            # Backend API
            location /api {
                proxy_pass http://backend:5000;
            }
        }
    }

```

### 🔹 Network Diagram
```
           🌍 External Users
                   |
            +------+------+
            |   NGINX     |  (reverse proxy)
            +------+------+ 
                   |
          [frontend_net]
         /           \
+--------+----+   +---+---------+
|  Frontend   |   |   Backend   |
|  (React)    |   | (Node/Java) |
+-------------+   +-------+-----+
                          |
                        [db_net]
                          |
                     +----+----+
                     |  MySQL  |
                     +---------+
```
### 🔹 How This Works

- Users only hit NGINX (ports 80/443 exposed).
- NGINX proxies requests:
    - `/`→ frontend container (frontend:80)
    - `/api` → backend container (backend:5000)
- Frontend and backend talk freely inside frontend_net.
- Backend connects to database on db_net.
- Database is never exposed to the outside world.

### 🔹 .env File
```bash
# MySQL
MYSQL_ROOT_PASSWORD=supersecret
MYSQL_DATABASE=myappdb
MYSQL_USER=myuser
MYSQL_PASSWORD=mypassword

# Ports
FRONTEND_PORT=3000
BACKEND_PORT=5000
NGINX_HTTP_PORT=80
NGINX_HTTPS_PORT=443
```
***You can also pass .env as `env_file: .env` in docker compose instead of individual environment variable***

--- 
### 🔹 Advanced

- Make base docker image
- Seperate override for dev and production env.

### 🔹 1. docker-compose.base.yml

This contains shared definitions (services, networks, volumes).

```bash
version: "3.9"

services:
  nginx:
    image: nginx:latest
    container_name: reverse_proxy
    restart: always
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      frontend:
        condition: service_healthy
      backend:
        condition: service_healthy
    networks:
      - frontend_net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/"]
      interval: 30s
      timeout: 10s
      retries: 5

  frontend:
    build: ./frontend
    container_name: frontend_app
    restart: always
    networks:
      - frontend_net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 5

  backend:
    build: ./backend
    container_name: backend_app
    restart: always
    environment:
      DB_HOST: db
      DB_PORT: 3306
      DB_NAME: ${MYSQL_DATABASE}
      DB_USER: ${MYSQL_USER}
      DB_PASSWORD: ${MYSQL_PASSWORD}
    depends_on:
      db:
        condition: service_healthy
    networks:
      - frontend_net
      - db_net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 5

  db:
    image: mysql:8.0
    container_name: mysql_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql:ro ## to run initial db scripts
    networks:
      - db_net
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-p${MYSQL_ROOT_PASSWORD}"]
      interval: 30s
      timeout: 10s
      retries: 5

volumes:
  db_data:

networks:
  frontend_net:
  db_net:
```

### 🔹 db init script

```sql
-- Example: create table and seed data
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL
);

INSERT INTO users (name, email) VALUES
('Alice', 'alice@example.com'),
('Bob', 'bob@example.com');

```

### 🔹 2. docker-compose.prod.yml (override for production)

```bash
version: "3.9"

services:
  nginx:
    ports:
      - "${NGINX_HTTP_PORT}:80"
      - "${NGINX_HTTPS_PORT}:443"

  backend:
    ports:
      - "${BACKEND_PORT}:5000"
```

### 🔹 3. docker-compose.dev.yml (override for development)
```bash
version: "3.9"

services:
  frontend:
    volumes:
      - ./frontend:/usr/share/nginx/html:cached
    ports:
      - "${FRONTEND_PORT}:80"

  backend:
    volumes:
      - ./backend:/app
    ports:
      - "${BACKEND_PORT}:5000"

  db:
    ports:
      - "3306:3306"
```

### Start production
```bash
docker-compose -f docker-compose.base.yml -f docker-compose.prod.yml up -d
```

### Start development
```bash
docker-compose -f docker-compose.base.yml -f docker-compose.dev.yml up -d
```
