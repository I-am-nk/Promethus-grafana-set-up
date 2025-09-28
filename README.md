
# 🚀 End-to-End Monitoring Setup with Prometheus, Node Exporter, and Blackbox

This repository documents how to set up **application + monitoring** on two AWS EC2 instances:  
- **Application Server** → Java + Maven + Demo Application + Node Exporter  
- **Monitoring Server** → Prometheus + Blackbox Exporter + Grafana  

---

## 🔹 Step 1: Provision Infrastructure
Launch **2 EC2 instances** (Ubuntu 22.04 LTS recommended):

- **Application Server**  
  Runs a Java-based demo app and Node Exporter (for system metrics).  

- **Monitoring Server**  
  Runs Prometheus, Blackbox Exporter, and Grafana.  

✅ Make sure to allow inbound ports in **Security Groups**:  
`22 (SSH), 8080 (App), 9100 (Node Exporter), 9090 (Prometheus), 9115 (Blackbox), 3000 (Grafana)`

---

## 🔹 Step 2: Set Up Application Server

Update system:
```bash
sudo apt update && sudo apt upgrade -y
```

Install Java & Maven:
```bash
sudo apt install openjdk-17-jdk maven -y
```

Verify:
```bash
java -version
mvn -v
```

Clone and build a sample Java app:
```bash
git clone https://github.com/I-am-nk/Promethus-grafana-set-up.git
cd Promethus-grafana-set-up
mvn package
```

Run the app:
```bash
java -jar target/*.jar
```

Check app in browser:  
👉 `http://<Application-Server-Public-IP>:8080`

---

## 🔹 Step 3: Install Node Exporter (on Application Server)

Download Node Exporter:
```bash
cd /opt
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
tar -xvzf node_exporter-1.8.1.linux-amd64.tar.gz
mv node_exporter-1.8.1.linux-amd64 node_exporter
cd node_exporter
```

Run Node Exporter:
```bash
./node_exporter &
```

Verify:
👉 `http://<Application-Server-Public-IP>:9100/metrics`

---
<img width="1913" height="858" alt="image" src="https://github.com/user-attachments/assets/e2d68ba4-c1b0-44e5-8410-2e8d146042c2" />

## 🔹 Step 4: Install Prometheus (on Monitoring Server)

```bash
cd /opt
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.53.0/prometheus-2.53.0.linux-amd64.tar.gz
tar -xvzf prometheus-2.53.0.linux-amd64.tar.gz
mv prometheus-2.53.0.linux-amd64 prometheus
cd prometheus
```

Run Prometheus:
```bash
./prometheus --config.file=prometheus.yml &
```

Verify:
👉 `http://<Monitoring-Server-Public-IP>:9090`

---
<img width="1915" height="985" alt="image" src="https://github.com/user-attachments/assets/1105a6b1-6bcd-485c-acd1-893b0b939720" />

## 🔹 Step 5: Configure Prometheus (`prometheus.yml`)

Edit `prometheus.yml`:
- a) Add the Blackbox Exporter target in Prometheus YAML and specify module as probe (for HTTP/HTTPS checks)
- b) Add a job for Node Exporter with the Application Server’s IP and port 9100 
- Note( change the < Application-Server-Public-IP > with your Ec2 Public IP )
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["<Application-Server-Public-IP>:9100"]

  - job_name: "blackbox"
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - http://<Application-Server-Public-IP>:8080
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9115  # Blackbox Exporter
```

Restart Prometheus:
```bash
pkill prometheus
./prometheus --config.file=prometheus.yml &
```

---

## 🔹 Step 6: Install Blackbox Exporter (on Monitoring Server)

```bash
cd /opt
curl -LO https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
tar -xvzf blackbox_exporter-0.25.0.linux-amd64.tar.gz
mv blackbox_exporter-0.25.0.linux-amd64 blackbox_exporter
cd blackbox_exporter
./blackbox_exporter &
```

Verify:  
👉 `http://<Monitoring-Server-Public-IP>:9115/probe?target=http://google.com&module=http_2xx`

---
<img width="1919" height="714" alt="image" src="https://github.com/user-attachments/assets/b1eb23f1-8dfb-4bde-95cb-100291a2de2a" />

## 🔹 Step 7: Install Grafana (on Monitoring Server)

```bash
sudo apt-get install -y apt-transport-https software-properties-common
sudo mkdir -p /etc/apt/keyrings/
wget -qO- https://packages.grafana.com/gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/grafana.gpg
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install grafana -y
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

Access Grafana:  
👉 `http://<Monitoring-Server-Public-IP>:3000` (default user: `admin` / `admin`)

---
<img width="1919" height="920" alt="image" src="https://github.com/user-attachments/assets/4cfa1911-780b-4a35-a2c6-289b6bf62769" />

## 🔹 Step 8: Import Grafana Dashboards

- **Node Exporter Dashboard** → Import ID `1860`  
- **Blackbox Exporter Dashboard** → Import ID `9965`  

Set refresh interval to `5s`.  

---

## 🔹 Step 9: Final Validation

- **Node Exporter Metrics**: CPU, RAM, Disk usage from Application Server.  
- **Blackbox Metrics**: App uptime, HTTP status, SSL expiry checks.  
- **Grafana Dashboards**: Real-time visualization.  

---
- Grafana Dashboards
<img width="1919" height="972" alt="image" src="https://github.com/user-attachments/assets/99c69e3f-b7e1-45f0-b64c-62b5957cbb96" />

- Blackbox Metrics
<img width="1919" height="981" alt="image" src="https://github.com/user-attachments/assets/ea3b95fc-480b-401b-8ed4-88a9a6d4c274" />


## ✅ Key Notes
- Open required ports in AWS Security Groups:
  ```
  22, 8080, 9100, 9090, 9115, 3000
  ```
- Run exporters and Prometheus with `nohup` or systemd for persistence.  
- Use labels in `prometheus.yml` to distinguish multiple servers.  
