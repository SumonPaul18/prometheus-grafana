# Grafana automating the provisioning of Data Sources and Dashboards using Docker-Compose

As a DevOps engineer, automating the provisioning of Data Sources and Dashboards is a best practice. This approach is called **"Provisioning"**. Instead of manually clicking in the UI, you define everything in YAML files and mount them into the Grafana container.

Here is the step-by-step guide to fully automate grafana-provisioning.

### 📂 Step 1: Update Project Structure

Create a new directory named `grafana-provisioning` in your project root. Inside it, create two subdirectories: `datasources` and `dashboards`.

```text
./monitoring-project/
├── docker-compose.yml
├── .env
├── prometheus.yml
└── grafana-provisioning/       <-- New Folder
    ├── datasources/
    │   └── datasource.yml      <-- Auto-adds Prometheus
    └── dashboards/
        ├── dashboard.yml       <-- Tells Grafana where to look
        └── my-dashboard.json   <-- Your exported JSON dashboard
```

---

### 📝 Step 2: Create Provisioning Files

#### 1. Define the Data Source (`grafana-provisioning/datasources/datasource.yml`)
This file tells Grafana to automatically add Prometheus as a data source.

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090  # Uses Docker service name
    isDefault: true
    editable: false             # Prevents accidental manual changes
```

#### 2. Define the Dashboard Provider (`grafana-provisioning/dashboards/dashboard.yml`)
This file tells Grafana to look for JSON files in a specific folder inside the container.

```yaml
apiVersion: 1

providers:
  - name: 'Default'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10   # Checks for new/updated dashboards every 10s
    options:
      path: /var/lib/grafana/dashboards  # Path inside the container
```

#### 3. Prepare Your Dashboard JSON
1. Go to your current running Grafana.
2. Open the dashboard you want to save.
3. Click **Share** (top right) -> **Export** -> **Save to file**.
4. Rename the downloaded file to `my-dashboard.json` and place it in `grafana-provisioning/dashboards/`.

*Note: Ensure the `uid` in the JSON file is unique if you plan to have multiple dashboards.*

---

### 🐳 Step 3: Update `docker-compose.yml`

You need to mount these new folders into the Grafana container. Update your `grafana` service section:

```yaml
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: always
    ports:
      - "3000:3000"
    environment:
      - GF_PANELS_DISABLE_SANITIZE_HTML=true
      - GF_SECURITY_ADMIN_USER=${GRAFANA_USER}
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana-data:/var/lib/grafana
      # Mount Provisioning Configs
      - ./grafana-provisioning/datasources:/etc/grafana/provisioning/datasources:ro
      - ./grafana-provisioning/dashboards:/etc/grafana/provisioning/dashboards:ro
      # Mount the actual dashboard JSON files
      - ./grafana-provisioning/dashboards:/var/lib/grafana/dashboards:ro
    networks:
      - monitoring
```

*Key Changes:*
1. Mounted `datasources` folder to `/etc/grafana/provisioning/datasources`.
2. Mounted `dashboards` config to `/etc/grafana/provisioning/dashboards`.
3. Mounted the actual JSON files to `/var/lib/grafana/dashboards` (as defined in `dashboard.yml`).

---

### ▶️ Step 4: Deploy and Verify

1. **Stop and Remove** the old container (to ensure clean volume mounting):
   ```bash
   docker compose down
   ```

2. **Start** the stack:
   ```bash
   docker compose up -d
   ```

3. **Verify:**
   - Log in to Grafana (`http://<IP>:3000`).
   - Go to **Connections** -> **Data Sources**. You should see **Prometheus** already added and working.
   - Go to **Dashboards**. You should see your imported dashboard ready to use.

---

### 💡 DevOps Best Practices for You

1. **Immutable Dashboards:** Since we set `editable: false` in the datasource and use provisioning, users cannot accidentally delete the data source. For dashboards, if you want to prevent UI edits, you can add `"editable": false` inside the JSON file itself.
2. **Updating Dashboards:** If you change a dashboard in the UI, export it again and replace the `.json` file in your repo. On the next restart (or after 10 seconds), Grafana will update it automatically.
3. **Version Control:** Commit the `grafana-provisioning` folder to your GitHub repository. Now, any new server you deploy with `docker compose up -d` will have the exact same monitoring setup without any manual clicks.

This method ensures your monitoring stack is **Infrastructure as Code (IaC)** compliant.

---