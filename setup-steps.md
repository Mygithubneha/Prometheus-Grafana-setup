
# Prometheus-Grafana Monitoring Server Setup

## Table of Contents

Install Prometheus
Install Node Exporter
Install and Configure Grafana
Access Grafana Dashboard

## Install Prometheus

1. Create Prometheus User

```markdown
sudo useradd \
 --system \
 --no-create-home \
 --shell /bin/false prometheus
```

## 2. Download Prometheus Package

```markdown
wget https: //github.com/prometheus/prometheus/releases/download/v2.49.0-rc.1/prometheus-2.49.0-rc.1.linux-amd64.tar.gz
```


## 3. Extract Prometheus Package


```python
tar -xvf prometheus-2.49.0-rc.1.linux-amd64.tar.gz
```

## 4. Create Required Directories

```python
sudo mkdir -p /data /etc/prometheus
```

## 5. Move Files to Appropriate Locations


```python
cd prometheus-2.49.0-rc.1.linux-amd64/
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles console_libraries/ prometheus.yml /etc/prometheus/
```

## 6. Set Permissions


```python
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```

## 7. Validate Prometheus Installation

```python
prometheus --version
```

## 8. Create Systemd Service File for Prometheus

Create and edit the file /etc/systemd/system/prometheus.service:

```python
sudo vim /etc/systemd/system/prometheus.service

```
Add the following content:

```python
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target
StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
 --config.file=/etc/prometheus/prometheus.yml \
 --storage.tsdb.path=/data \
 --web.console.templates=/etc/prometheus/consoles \
 --web.console.libraries=/etc/prometheus/console_libraries \
 --web.listen-address=0.0.0.0: 9090 \
 --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```


## 9. Enable and Start Prometheus Service

```python
sudo systemctl enable prometheus.service
sudo systemctl start prometheus.service
systemctl status prometheus.service
```

## 10. Access Prometheus

Open a web browser and access Prometheus using the Monitoring Server's public IP on port 9090.

## Install Node Exporter

### 1. Create Node Exporter User


```markdown
sudo useradd \
 --system \
 --no-create-home \
 --shell /bin/false node_exporter
```


### 2. Download Node Exporter Package


```python
wget https: //github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
```

### 3. Extract and Move Node Exporter


```python
tar -xvf node_exporter-1.7.0.linux-amd64.tar.gz
sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
```

### 4. Validate Node Exporter Installation

```python
node_exporter --version
```

### 5. Create Systemd Service File for Node Exporter
Create and edit the file /etc/systemd/system/node_exporter.service:


```python
sudo vim /etc/systemd/system/node_exporter.service

```
Add the following content:



```python
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter \
 --collector.logind

[Install]
WantedBy=multi-user.target
```

### 6. Enable and Start Node Exporter Service


```python
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
systemctl status node_exporter.service
```

### 7. Add Node Exporter to Prometheus Targets
Edit the Prometheus configuration file:

```python
sudo vim /etc/prometheus/prometheus.yml

```
Add the following content:


```python
- job_name: "node_exporter"
  static_configs:
    - targets: [
  "localhost:9100"
]
```

### 8. Validate Configuration and Reload Prometheus


```python
promtool check config /etc/prometheus/prometheus.yml
curl -X POST http: //localhost:9090/-/reload
```

Check if the node exporter target is up and running on the Prometheus server.

# Install and Configure Grafana

### 1. Add Grafana APT Repository


```python
sudo apt-get install -y apt-transport-https software-properties-common wget
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https: //apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com beta main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
```

### 2. Install Grafana


```python
sudo apt-get install grafana

```
### 3. Enable and Start Grafana Service

```python
sudo systemctl enable grafana-server.service
sudo systemctl start grafana-server.service
sudo systemctl status grafana-server.service
```

### Access Grafana Dashboard

Open Browser: Use the Monitoring Server's public IP on port 3000.

### Login Credentials:

Username: admin
Password: admin (Change the password upon initial login)

### Add Data Source:

Click on Data sources.

Select Prometheus.

Provide the Monitoring Server Public IP with port 9090.

Click on Save and test.

### Import Dashboard:

Go to Dashboard section.

Click on Import dashboard.

Add 1860 for the node exporter dashboard and click on Load.

Select Prometheus from the drop-down menu and click on Import.