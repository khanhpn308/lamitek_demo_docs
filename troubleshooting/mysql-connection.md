# MySQL Connection Troubleshooting Guide

## Error: "Can't connect to MySQL server on '127.0.0.1:3306'"

### Overview

This error occurs when the FastAPI backend cannot establish a connection to the MySQL database during the lifespan startup phase. The error manifests in `app/core/db_wait.py` after 60 retry attempts at 1-second intervals.

**Error Message:**

```
sqlalchemy.exc.OperationalError: (pymysql.err.OperationalError) (2003, "Can't connect to MySQL server on '127.0.0.1'")
RuntimeError: Database unavailable after 60 attempts at 127.0.0.1:3306
```

---

## Root Causes & Diagnosis

### 1. **Database Service Not Running**

**Symptom:** Backend container starts but cannot reach database immediately.

**Diagnosis:**

```bash
# Check if database_service containers are running
cd database_service
docker compose ps

# Expected output: MySQL container should be RUNNING and HEALTHY
# Example:
# NAME             IMAGE              STATUS               PORTS
# database_service-db-1  mysql:8.0   Up 2 minutes (healthy)  0.0.0.0:3306->3306/tcp
```

**Solution:**

```bash
# Start the database service
cd database_service
docker compose up -d

# Wait for health check to pass (30-45 seconds)
docker compose logs db --follow

# Once healthy, start app_service
cd ../app_service
docker compose up -d
```

---

### 2. **Wrong DB_HOST Value in .env**

**Symptom:** Database is running but backend still cannot connect, even after waiting.

**Diagnosis:**

Check your `.env` file in `app_service/`:

```bash
# View current configuration
cat app_service/.env | grep DB_

# You should see:
DB_HOST=???
DB_PORT=3306
DB_NAME=iot_db
DB_USER=root
DB_PASSWORD=your_password
```

**Root Cause Analysis:**

The correct `DB_HOST` value depends on whether MySQL is on the **same Docker network** as the backend or on a **different machine**:

| Case                                | Correct `DB_HOST`          | Reason                                                                |
| ----------------------------------- | -------------------------- | --------------------------------------------------------------------- |
| Same machine, same Docker network   | `db`                       | Use the MySQL service name from `database_service/docker-compose.yml` |
| Different machine or remote server  | IP address or DNS name     | Example: `192.168.1.100`, `mysql.example.com`                         |
| MySQL on the host OS, not in Docker | `127.0.0.1` or `localhost` | Only if MySQL runs directly on the host machine                       |

**Current project setup:**
`database_service` and `app_service` are separate Compose stacks connected through the external network `iot-net`. If both stacks run on the same machine, the backend must use:

✅ `DB_HOST=db`

Use `127.0.0.1` only when the database is not running as a container service on the shared Docker network.

**Solution:**

```bash
# 1. Edit .env in app_service
cd app_service
cat > .env << 'EOF'
# Database Configuration
DB_HOST=db
DB_PORT=3306
DB_NAME=iot_db
DB_USER=root
DB_PASSWORD=your_password

# ... other environment variables
EOF

# 2. Restart backend with new configuration
docker compose up -d --build backend
```

---

### 3. **External Network Not Created**

**Symptom:** Both stacks appear to be running, but backend cannot reach database at all (no timeout, immediate failure).

**Diagnosis:**

```bash
# List all Docker networks
docker network ls

# Look for "iot-net" in the output
# Expected:
# NETWORK ID     NAME              DRIVER    SCOPE
# abc123def456   iot-net           bridge    local
# ...
```

**If `iot-net` is missing:**

```bash
# Create the external network
docker network create iot-net

# Verify creation
docker network ls | grep iot-net
docker network inspect iot-net
```

**Verify Both Stacks Connected:**

```bash
# Check which networks app_service uses
docker network inspect iot-net | grep -A 20 "Containers"

# You should see both:
# - database_service MySQL container
# - app_service backend container
```

**Solution:**

If recreating the network:

```bash
# Stop both stacks
cd database_service && docker compose down
cd ../app_service && docker compose down

# Create network
docker network create iot-net

# Restart both stacks
cd database_service && docker compose up -d
cd ../app_service && docker compose up -d
```

---

### 4. **Network Connectivity (Docker Desktop / WSL)**

**Symptom:** Configuration looks correct, but still cannot connect.

**Diagnosis for Docker Desktop on Windows/Mac:**

```bash
# From inside backend container, test connectivity
docker compose exec backend bash

# Inside container:
nc -zv db 3306
# Or:
mysql -h db -u root -p<password> -e "SELECT 1"
```

**Common Issues:**

- **WSL2 network isolation:** External networks must be explicit
- **VPN interference:** VPN can block container-to-container communication
- **Host.docker.internal:** Don't use for inter-container communication

---

## Verification Checklist

Before debugging further, verify each step:

```bash
# 1. Database service is running
cd database_service
docker compose ps
# ✓ db container is RUNNING and healthy

# 2. Database is accepting connections
docker compose exec db mysql -u root -e "SELECT 1"
# ✓ Returns: 1

# 3. External network exists
docker network ls | grep iot-net
# ✓ iot-net is listed

# 4. Backend container is on the network
docker network inspect iot-net | grep "app_service-backend"
# ✓ Backend container is listed

# 5. DB_HOST is correct in .env
cd ../app_service
grep DB_HOST .env
# ✓ Should show: DB_HOST=db

# 6. Test connectivity from backend container
docker compose exec backend bash
nc -zv db 3306
# ✓ Connected to db:3306 OK

# 7. Try connecting with MySQL client
docker compose exec backend \
  mysql -h db -u root -pYourPassword -e "SELECT 1"
# ✓ Returns: 1
```

---

## Step-by-Step Recovery

If you're stuck, follow this sequence:

### Step 1: Stop Everything Cleanly

```bash
# From app_service directory
docker compose down

# From database_service directory
cd ../database_service
docker compose down
```

### Step 2: Verify Network

```bash
# Create fresh network
docker network create iot-net 2>/dev/null || echo "Network already exists"

# Clean up any orphaned containers
docker container prune -f
```

### Step 3: Start Database

```bash
cd database_service
docker compose up -d

# Wait for healthy status
sleep 30
docker compose ps

# Verify database is responsive
docker compose exec db mysql -u root -proot -e "SELECT VERSION();"
```

### Step 4: Start App Service

```bash
cd ../app_service

# Update .env if needed
cat .env | grep DB_HOST
# Make sure it says: DB_HOST=db

# Start the stack
docker compose up -d

# Monitor startup
docker compose logs -f backend
```

### Step 5: Verify Application Started

```bash
# Check backend container logs for successful startup
docker compose logs backend | tail -20

# Expected final lines:
# Uvicorn running on http://0.0.0.0:8000
# [INFO] MQTT Subscriber started
# [INFO] InfluxDB connection established

# Test API health endpoint
curl http://localhost:8000/api/health
# Response: {"status": "healthy"}
```

---

## Common Scenarios & Solutions

### Scenario A: Fresh Deployment on New Machine

**Steps:**

1. Clone repository
2. Copy `.env` files from `.env.example`
3. Update passwords in both `.env` files
4. Create external network: `docker network create iot-net`
5. Start database_service first
6. Wait 30 seconds for health check
7. Start app_service

---

### Scenario B: Restarting After System Restart

**Issue:** Network might not persist across docker daemon restarts.

**Solution:**

```bash
# Recreate network
docker network create iot-net 2>/dev/null || echo "Network exists"

# Restart all services
cd database_service && docker compose up -d
cd ../app_service && docker compose up -d
```

---

### Scenario C: Using Different Database Host

**If your MySQL is hosted elsewhere (cloud, different server):**

```bash
# In app_service/.env:
DB_HOST=your-mysql-server.example.com  # or IP like 10.0.0.5
DB_PORT=3306
DB_USER=iot_user
DB_PASSWORD=strong_password
DB_NAME=iot_db

# Restart backend
docker compose up -d --build backend
```

---

## Related Files

- **Database Initialization:** `database_service/sql/schema.sql`
- **Connection Waiting Logic:** `app_service/backend/app/core/db_wait.py`
- **Database Configuration:** `app_service/backend/app/core/config.py`
- **FastAPI Lifespan:** `app_service/backend/app/main.py` (lines with `lifespan()` function)
- **Docker Compose Setup:**
  - `app_service/docker-compose.yml`
  - `database_service/docker-compose.yml`

---

## Debug Logs

To get detailed connection information, temporarily modify `db_wait.py`:

```python
# In app_service/backend/app/core/db_wait.py
# Add after the import statements:

import os
logger = logging.getLogger(__name__)

# In the wait_for_db() function, before the loop:
host = os.getenv("DB_HOST", "localhost")
port = os.getenv("DB_PORT", "3306")
logger.info(f"Attempting to connect to {host}:{port}")
```

Then check logs:

```bash
docker compose logs backend | grep "Attempting to connect"
```

---

## Contact & Support

If the issue persists after following all steps:

1. **Collect diagnostic information:**

   ```bash
   # Network status
   docker network inspect iot-net > network-debug.log

   # Container status
   docker compose ps > containers-debug.log

   # Backend logs
   docker compose logs backend > backend-debug.log

   # Database connectivity test
   docker compose exec backend nc -zv db 3306 2>&1 > connectivity-debug.log
   ```

2. **Check environment variables:**

   ```bash
   # Verify .env values
   cat app_service/.env | grep DB_
   cat database_service/.env | grep -E "MYSQL_|MYSQL_"
   ```

3. **Verify MySQL version compatibility:**
   - Check `database_service/docker-compose.yml` for MySQL image version
   - Ensure Python MySQL driver (PyMySQL) is compatible

---

## Prevention Tips

- Always start `database_service` before `app_service`
- Keep `docker-compose.yml` files in sync regarding network definitions
- Use health checks in docker-compose to ensure ordered startup
- Document environment-specific `DB_HOST` values in your deployment notes
- Test database connectivity before deploying to production
