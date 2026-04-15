# Prometheus & Grafana Monitoring Stack (Local Storage/Bind Mounts)

This guide sets up a production-ready monitoring stack using **Bind Mounts**. All persistent data (Grafana dashboards, Prometheus metrics) will be stored in folders inside your current project directory.

## 📋 Prerequisites

- Docker Engine installed (v20.10+)
- Docker Compose plugin installed (`docker compose`)
- Linux-based host (Ubuntu/CentOS/RHEL)
- Sudo privileges

## 📂 Project Structure

Ensure your directory looks like this before starting:

```text
./monitoring-project/
├── docker-compose.yml
├── prometheus.yml      # You must create this file (see below)
├── .env                # You must create this file (see below)
├── grafana-data/       # Created automatically (or manually)
└── prometheus-data/    # Created automatically (or manually)
```

## ⚙️ Step 1: Configuration Files

### 1. Create `.env` File
Create a file named `.env` to store sensitive credentials securely.

```bash
nano .env
```

Add the following content:
```env
GRAFANA_USER=admin
GRAFANA_PASSWORD=SecurePassword@2026
```

### 2. Create `prometheus.yml`
Create a file named `prometheus.yml` in the same directory. This tells Prometheus what to monitor.

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
```

## 🚀 Step 2: Prepare Directories & Permissions

Since we are using **Bind Mounts** (`./grafana-data`) and running containers as non-root user (`1001`), we **must** set correct permissions manually. If you skip this, the containers will fail with "Permission Denied".

Run these commands in your terminal:

#### 1. Create directories 
```bash
mkdir -p grafana-data prometheus-data
```
#### 2. Set ownership to UID 1001 (Grafana/Prometheus internal user)
```
sudo chown -R 1001:1001 grafana-data prometheus-data
```

## ▶️ Step 3: Start the Services

Run the following command to start the stack in detached mode:

```bash
docker compose up -d
```

## ✅ Step 4: Verification

1. **Check Status:**
   ```bash
   docker compose ps
   ```
   All services should show `Up`.

2. **Access Grafana:**
   - Open browser: `http://<YOUR_SERVER_IP>:3000`
   - Login with credentials from `.env`.

3. **Access Prometheus:**
   - Open browser: `http://<YOUR_SERVER_IP>:9090`
   - Go to **Status > Targets** to ensure all endpoints are green.

## 🛠️ Management & Maintenance

### View Logs
```bash
docker compose logs -f grafana
docker compose logs -f prometheus
```

### Stop Services
```bash
docker compose down
```

### Backup Data
Since data is in local folders, backup is simple:
```bash
tar -czvf monitoring-backup-$(date +%F).tar.gz ./grafana-data ./prometheus-data
```

### Restore Data
1. Stop services: `docker compose down`
2. Extract backup: `tar -xzvf monitoring-backup-DATE.tar.gz`
3. Fix permissions: `sudo chown -R 1001:1001 grafana-data prometheus-data`
4. Start services: `docker compose up -d`

## ⚠️ Troubleshooting

- **Container exits immediately?** Check permissions. Run `ls -ld grafana-data`. It should be owned by `1001`. If not, run `sudo chown -R 1001:1001 grafana-data`.
- **Port already in use?** Ensure ports 3000, 9090, 8080, and 9100 are free. Use `sudo ss -tlnp | grep <port>` to check.

### 🔄 Updating Prometheus Configuration

When you modify `prometheus.yml` (e.g., adding new targets or changing scrape intervals), you need to apply these changes to the running container. The method depends on whether the `--web.enable-lifecycle` flag is enabled in your `docker-compose.yml`.

### Scenario A: If `--web.enable-lifecycle` is Enabled (Recommended)
*Your current `docker-compose.yml` already includes this flag.*

This allows you to reload the configuration **without restarting** the Prometheus container, ensuring zero downtime for data collection.

1. **Edit the prometheus.yml file:**
   ```bash
   nano prometheus.yml
   ```
   > Make your changes and save

2. **Trigger a Hot Reload:**
   Send a `POST` request to the Prometheus API using `curl`:
   ```bash
   curl -X POST http://localhost:9090/-/reload
   ```

3. **Verify:**
   Check the Prometheus logs to confirm successful reload:
   ```bash
   docker logs prometheus
   ```

### Scenario B: If `--web.enable-lifecycle` is NOT Enabled
If you remove the flag or use a basic setup, you must restart the container to apply changes. This causes a brief interruption in scraping.

1. **Edit the prometheus.yml file:**
   ```bash
   nano prometheus.yml
   ```
   > Make your changes and save

2. **Restart the Service:**
   ```bash
   docker compose restart prometheus
   ```

### ⚡ Quick Comparison

| Method | Command | Downtime | Requirement |
| :--- | :--- | :--- | :--- |
| **Hot Reload** | `curl -X POST http://localhost:9090/-/reload` | None (Zero) | `--web.enable-lifecycle` flag must be set |
| **Restart** | `docker compose restart prometheus` | Brief (Seconds) | No special flags needed |

> **💡 Pro Tip:** Always prefer **Hot Reload** in production environments to maintain continuous monitoring data integrity.

---
