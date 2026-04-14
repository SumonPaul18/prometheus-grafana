
# 📄 Guide 2: Managed Volume Setup (Named Volumes)
**File:** `docker-compose1.yml`  
**Strategy:** Data is managed by Docker in `/var/lib/docker/volumes`. Best for stability, security, and avoiding permission issues.

```markdown
# Prometheus & Grafana Monitoring Stack (Named Volumes)

This guide sets up a robust monitoring stack using **Docker Named Volumes**. Docker manages the storage location internally. This method avoids manual permission errors and is recommended for long-term production use.

## 📋 Prerequisites

- Docker Engine installed (v20.10+)
- Docker Compose plugin installed (`docker compose`)
- Linux-based host (Ubuntu/CentOS/RHEL)

## 📂 Project Structure

```text
./monitoring-project/
├── docker-compose1.yml
├── prometheus.yml      # You must create this file
└── .env                # You must create this file
```
*Note: No data folders will appear in this directory. Data is stored in Docker's internal volume path.*

## ⚙️ Step 1: Configuration Files

### 1. Create `.env` File
Store your admin credentials securely.

```bash
nano .env
```

Add the following:
```env
GRAFANA_USER=admin
GRAFANA_PASSWORD=SecurePassword@2026
```

### 2. Create `prometheus.yml`
Define your scrape targets. Note that we use **service names** (`node-exporter`, `cadvisor`) as hosts because they are on the same Docker network.

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

## ▶️ Step 2: Start the Services

No manual folder creation or permission changes are needed. Docker handles everything.

```bash
docker compose -f docker-compose1.yml up -d
```

## ✅ Step 3: Verification

1. **Check Status:**
   ```bash
   docker compose -f docker-compose1.yml ps
   ```

2. **Verify Volumes:**
   Check that Docker created the volumes:
   ```bash
   docker volume ls | grep monitoring
   ```
   You should see `monitoring_grafana-data` and `monitoring_prometheus-data`.

3. **Access Interfaces:**
   - **Grafana:** `http://<YOUR_SERVER_IP>:3000`
   - **Prometheus:** `http://<YOUR_SERVER_IP>:9090`
   - **cAdvisor:** `http://<YOUR_SERVER_IP>:8080` (Useful for debugging container metrics)

## 🛠️ Management & Maintenance

### View Logs
```bash
docker compose -f docker-compose1.yml logs -f prometheus
```

### Stop Services
```bash
docker compose -f docker-compose1.yml down
```
*Note: This stops containers but **keeps** the data volumes intact.*

### Remove Data Completely (Reset)
If you want to wipe all data and start fresh:
```bash
docker compose -f docker-compose1.yml down -v
```

### Backup Data (Production Grade)
Since data is in internal Docker volumes, use a temporary container to back it up.

**Backup Prometheus:**
```bash
docker run --rm -v monitoring_prometheus-data:/data -v $(pwd):/backup alpine tar czf /backup/prometheus-backup.tar.gz -C /data .
```

**Backup Grafana:**
```bash
docker run --rm -v monitoring_grafana-data:/data -v $(pwd):/backup alpine tar czf /backup/grafana-backup.tar.gz -C /data .
```

### Restore Data
1. Stop services: `docker compose -f docker-compose1.yml down`
2. Restore Prometheus:
   ```bash
   docker run --rm -v monitoring_prometheus-data:/data -v $(pwd):/backup alpine sh -c "rm -rf /data/* && tar xzf /backup/prometheus-backup.tar.gz -C /data"
   ```
3. Start services: `docker compose -f docker-compose1.yml up -d`

## ⚠️ Troubleshooting

- **Permission Errors?** Unlikely with Named Volumes. If they occur, check if you manually changed permissions in `/var/lib/docker/volumes/`. Avoid doing this.
- **Data Not Persisting?** Ensure you are NOT using `down -v` unless you intend to delete data.
- **cAdvisor Privileged Mode:** This setup uses `privileged: true` for cAdvisor to access host devices. If security is a strict concern, consider using `cap_add` instead (see advanced docs).
```

---

### 💡 Key Differences Summary for You

| Feature | Guide 1 (`docker-compose.yml`) | Guide 2 (`docker-compose1.yml`) |
| :--- | :--- | :--- |
| **Storage Type** | Bind Mount (`./folder`) | Named Volume (`docker volume`) |
| **Setup Steps** | Requires `mkdir` & `chown` | Just `up -d` |
| **Data Location** | Visible in current folder | Hidden in `/var/lib/docker/volumes` |
| **Best For** | Quick tests, easy manual backup | Production, stability, security |
| **Permission Risk**| High (if `chown` is skipped) | Low (Docker manages it) |

Choose **Guide 1** if you want to see your data files directly. Choose **Guide 2** if you want a "set it and forget it" production setup.