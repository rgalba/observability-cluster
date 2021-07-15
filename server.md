# Observability cluster server-based

## Setup Prometheus
```
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.7.1/prometheus-2.7.1.linux-amd64.tar.gz
tar -xvf prometheus-2.7.1.linux-amd64.tar.gz

cd prometheus-2.7.1.linux-amd64/
sudo mv console* /etc/prometheus
sudo mv prometheus.yml /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus

sudo mv prometheus /usr/local/bin/
sudo mv promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo vim /etc/systemd/system/prometheus.service
```

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus
```

## Setup Alertmanager

```
sudo useradd --no-create-home --shell /bin/false alertmanager
sudo mkdir /etc/alertmanager
wget "https://github.com/prometheus/alertmanager/releases/download/v0.16.1/alertmanager-0.16.1.linux-amd64.tar.gz"
tar -xvf alertmanager-0.16.1.linux-amd64.tar.gz
cd alertmanager-0.16.1.linux-amd64/
sudo mv alertmanager /usr/local/bin/
sudo mv amtool /usr/local/bin/
sudo chown -R alertmanager:alertmanager /usr/local/bin/alertmanager
sudo chown -R alertmanager:alertmanager /usr/local/bin/amtool
sudo mv alertmanager.yml /etc/alertmanager/
sudo chown -R alertmanager:alertmanager /etc/alertmanager/
sudo vim /etc/systemd/system/alertmanager.service
```

```
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
WorkingDirectory=/etc/alertmanager/
ExecStart=/usr/local/bin/alertmanager \
    --config.file=/etc/alertmanager/alertmanager.yml

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl stop prometheus
sudo vim /etc/prometheus/prometheus.yml
```

```
alerting:
  alertmanagers:
    - static_configs:
      - targets:
        - localhost:9093
```

```
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl start alertmanager
sudo systemctl enable alertmanager
```

## Setup Grafana

```
sudo apt -y install libfontconfig
wget https://dl.grafana.com/oss/release/grafana_5.4.3_amd64.deb
sudo dpkg -i grafana_5.4.3_amd64.deb
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
```

> Login using admin/admin

### Add enpoints

```
sudo vim /etc/prometheus/prometheus.yml
```

```
- job_name: 'alertmanager'
  static_configs:
  - targets: ['localhost:9093']
- job_name: 'grafana'
  static_configs:
  - targets: ['localhost:3000']
```

```
sudo systemctl restart prometheus
sudo systemctl status prometheus
```
