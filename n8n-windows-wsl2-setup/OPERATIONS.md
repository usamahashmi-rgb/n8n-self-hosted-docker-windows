# 🛠️ n8n Day-2 Operations & Maintenance Manual

> **Back to Installation Guide:** [README.md](README.md)

A practical operations reference for managing, monitoring, backing up, and safely updating a locally hosted **n8n** instance running on **Windows 11 + WSL 2 (Ubuntu 24.04 LTS) + Docker Desktop**.

![Platform](https://img.shields.io/badge/Platform-Windows%2011%20%7C%20WSL%202-0078D6?logo=windows)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04%20LTS-E95420?logo=ubuntu&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)
![n8n](https://img.shields.io/badge/n8n-Operations%20Manual-EA4B71?logo=n8n&logoColor=white)

---

## 📋 Environment Summary

| Parameter | Configuration |
| :--- | :--- |
| **Host Operating System** | Windows 10 / Windows 11 |
| **Linux Environment** | WSL 2 (`Ubuntu-24.04`) |
| **Container Engine** | Docker Desktop (WSL 2 Backend) |
| **Project Directory** | `~/n8n` (inside Ubuntu filesystem) |
| **Container Name** | `n8n` |
| **Local Web URL** | `http://localhost:5678` |
| **Persistent Volume** | `n8n_n8n_data` *(Docker Compose prefixed)* |
| **Timezone** | `Asia/Karachi` |

---

## 📑 Table of Contents

- [1. System Overview & Data Persistence](#1-system-overview--data-persistence)
- [2. Accessing the n8n Dashboard](#2-accessing-the-n8n-dashboard)
- [3. Daily Service Management Commands](#3-daily-service-management-commands)
- [4. Logs, Diagnostics & Resource Monitoring](#4-logs-diagnostics--resource-monitoring)
- [5. Safe n8n Update Procedure](#5-safe-n8n-update-procedure)
- [6. Docker Volume Management & Backups](#6-docker-volume-management--backups)

---

## 1. System Overview & Data Persistence

This manual covers routine administration for your local n8n stack:

```text
Windows 11 (Host OS)
 └── Docker Desktop (WSL 2 Engine)
      └── Ubuntu 24.04 LTS (~/n8n)
           ├── Docker Container: n8n (Stateless Application Runtime)
           └── Docker Volume: n8n_n8n_data (Persistent Workflows & Credentials)
```

> **How Persistence Works:** The `n8n` container holds only the application runtime, while your workflows, credentials, encryption key, and settings live inside the Docker volume (`n8n_n8n_data`). Stopping, restarting, or recreating the container will **not** delete your data as long as you preserve this volume.

---

## 2. Accessing the n8n Dashboard

Open **Google Chrome**, **Microsoft Edge**, or your preferred browser and navigate to:

```text
http://localhost:5678
```

**Prerequisites for access:**
1. **Docker Desktop** must be running on Windows 11.
2. The `n8n` container must be in the `Up` state (`docker ps`).

---

## 3. Daily Service Management Commands

Open **Ubuntu 24.04** from the Windows Start menu to run the commands below.

### 3.1 Start n8n
Starts the existing `n8n` container:
```bash
docker start n8n
```
*(Or via Docker Compose inside `~/n8n`: `docker compose up -d`)*

### 3.2 Stop n8n
Gracefully stops the n8n service without deleting workflows or stored data:
```bash
docker stop n8n
```

### 3.3 Restart n8n
Restarts the running `n8n` container (useful after environment or network changes):
```bash
docker restart n8n
```

### 3.4 Check Running Containers
Displays currently active containers, uptime, and mapped ports (`5678`):
```bash
docker ps
```

### 3.5 Check All Containers (Running & Stopped)
Displays all containers on the system, including stopped ones:
```bash
docker ps -a
```

---

## 4. Logs, Diagnostics & Resource Monitoring

Use these commands inside your Ubuntu terminal to troubleshoot workflow execution errors, inspect configuration, or monitor system resource usage.

### 4.1 View All Container Logs
```bash
docker logs n8n
```

### 4.2 Stream Live Logs in Real Time
```bash
docker logs -f n8n
```
> Press `Ctrl + C` to exit the live log stream. Exiting logs does **not** stop the `n8n` container.

### 4.3 View the Last 100 Log Lines
```bash
docker logs --tail 100 n8n
```

### 4.4 Inspect Full Container Configuration
Displays environment variables, volume mounts, network settings, and restart policies in JSON format:
```bash
docker inspect n8n
```

### 4.5 Check Installed n8n Version
Queries the exact n8n version running inside the container:
```bash
docker exec n8n n8n --version
```

### 4.6 Monitor Live CPU & Memory Usage
Displays real-time CPU, RAM, network I/O, and disk I/O consumption:
```bash
docker stats n8n
```
*(Press `Ctrl + C` to exit.)*

---

## 5. Safe n8n Update Procedure

Follow these steps whenever you want to upgrade n8n to the latest release.

> **⚠️ Important Volume Note:** Because the installation guide deploys n8n using **Docker Compose** inside `~/n8n`, Docker Compose automatically prefixes your volume with the folder name (`n8n_n8n_data`). Always update using **Docker Compose** as shown below so your existing workflows remain attached. *(Running a standalone `docker run -v n8n_data:...` command would mount a separate empty volume.)*

### Step 1: Navigate to Your Project Directory
```bash
cd ~/n8n
```

### Step 2: (Recommended) Create a Quick Volume Backup
Back up your `n8n_n8n_data` volume before upgrading:
```bash
mkdir -p ~/n8n-backups
docker run --rm \
  -v n8n_n8n_data:/data \
  -v ~/n8n-backups:/backup \
  ubuntu tar czf /backup/n8n-data-$(date +%F).tar.gz -C /data .
```

### Step 3: Pull the Latest n8n Docker Image
```bash
docker compose pull
```

### Step 4: Recreate & Start the Updated Container
Docker Compose automatically stops the old container, creates the updated one, and reattaches `n8n_n8n_data`:
```bash
docker compose up -d
```

### Step 5: Verify Container Status & New Version
```bash
docker ps
docker exec n8n n8n --version
```

### Step 6: Open n8n in Your Browser
Visit `http://localhost:5678` and verify that your workflows and credentials are intact.

---

## 6. Docker Volume Management & Backups

### 6.1 List All Docker Volumes
```bash
docker volume ls
```

### 6.2 Inspect the n8n Volume
Displays driver details, creation timestamp, labels, and mount path:
```bash
docker volume inspect n8n_n8n_data
```

### 6.3 Display the Volume Mount Location
Prints only the internal WSL 2 filesystem path where Docker stores the volume data:
```bash
docker volume inspect n8n_n8n_data --format '{{.Mountpoint}}'
```

### 6.4 Check Whether the n8n Volume Exists
Filters your Docker volumes to confirm the `n8n` volume is present:
```bash
docker volume ls --filter name=n8n
```

### 6.5 Restore from a Backup Archive *(If Needed)*
If you ever need to restore a backup created in Step 2:
```bash
cd ~/n8n
docker compose down
docker run --rm \
  -v n8n_n8n_data:/data \
  -v ~/n8n-backups:/backup \
  ubuntu bash -c "rm -rf /data/* && tar xzf /backup/n8n-data-YYYY-MM-DD.tar.gz -C /data"
docker compose up -d
```
*(Replace `YYYY-MM-DD` with your backup file's date.)*

> **🚨 Data Safety Reminder:** Never run `docker compose down -v` or `docker volume rm n8n_n8n_data` unless you intentionally want to permanently erase all workflows, credentials, and user accounts.