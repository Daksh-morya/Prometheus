#!/bin/bash

# Set versions
PROM_VERSION="2.48.0"
NODE_EXPORTER_VERSION="1.8.0"

echo " Installing Prometheus and Node Exporter..."

# Create directories
mkdir -p /opt/prometheus /opt/node_exporter

# Download Prometheus
cd /opt
wget https://github.com/prometheus/prometheus/releases/download/v$PROM_VERSION/prometheus-$PROM_VERSION.linux-amd64.tar.gz
tar -xvf prometheus-$PROM_VERSION.linux-amd64.tar.gz
mv prometheus-$PROM_VERSION.linux-amd64/* /opt/prometheus/

# Download Node Exporter
cd /opt
wget https://github.com/prometheus/node_exporter/releases/download/v$NODE_EXPORTER_VERSION/node_exporter-$NODE_EXPORTER_VERSION.linux-amd64.tar.gz
tar -xvf node_exporter-$NODE_EXPORTER_VERSION.linux-amd64.tar.gz
mv node_exporter-$NODE_EXPORTER_VERSION.linux-amd64/* /opt/node_exporter/

# Create Prometheus config
cat <<EOF > /opt/prometheus/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
EOF

# Create Prometheus systemd service
cat <<EOF > /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus Monitoring
After=network.target

[Service]
User=root
ExecStart=/opt/prometheus/prometheus --config.file=/opt/prometheus/prometheus.yml
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Create Node Exporter systemd service
cat <<EOF > /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=root
ExecStart=/opt/node_exporter/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Reload and start services
systemctl daemon-reexec
systemctl enable prometheus
systemctl start prometheus

systemctl enable node_exporter
systemctl start node_exporter

echo " Prometheus running on port 9090"
echo " Node Exporter running on port 9100"
echo " Access Prometheus UI: http://<your-server-ip>:9090"
