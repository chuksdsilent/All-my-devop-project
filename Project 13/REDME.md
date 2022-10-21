# PROMETHEUS AND GRAFANA

## What is prometheus

- Prometheus is an open-source monitoring and alserting system
- Prometheus can be used to monitor highly dynamic container environments like kubernetes, docker swarm or we can also use it as a traditional non0container application
- Targets can be configured with service discovery pattern or static configuration is also supported
- Prometheus scras/pull metrics from targes using pull mechanism. This is what differentiates it from others
- It has got strong query language called PROMQL.
- Prometheus is a stand alone(No dependencies)
- Most prometheus components are written in GO.

## Installing Prometheus

1. Create a file and name it prometheus.yaml then add the following command to it

```
global:
  scrape_interval: 15s
  external_labels:
    monitor: 'prometheus'

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

2. Create a file and name it prometheus.service then add the following command

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

3. Create a file then name it install-prometheus.sh and add the following commands

```
sudo useradd --no-create-home prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

wget  https://github.com/prometheus/prometheus/releases/download/v2.23.0/prometheus-2.23.0.linux-amd64.tar.gz
tar -xvf prometheus-2.23.0.linux-amd64.tar.gz
sudo cp prometheus-2.23.0.linux-amd64/prometheus /usr/local/bin
sudo cp prometheus-2.23.0.linux-amd64/promtool /usr/local/bin
sudo cp -r prometheus-2.23.0.linux-amd64/consoles /etc/prometheus/
sudo cp -r prometheus-2.23.0.linux-amd64/console_libraries /etc/prometheus
sudo cp prometheus-2.23.0.linux-amd64/promtool /usr/local/bin/

rm -rf prometheus-2.23.0.linux-amd64.tar.gz prometheus-2.19.0.linux-amd64
sudo cp prometheus.yml /etc/prometheus/
sudo cp prometheus.service /etc/systemd/system/prometheus.service

sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
sudo chown -R prometheus:prometheus /var/lib/prometheus

sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

4. Run it with sudo ./install-prometheus.sh

5. Allow the following port from your ec2 instance

- port: 9090 and 9000

## Installing node-exporter

1. Create a file then name it node-exporter.service and add the following commands

```
sudo useradd --no-create-home node_exporter

wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz
tar xzf node_exporter-1.0.1.linux-amd64.tar.gz
sudo cp node_exporter-1.0.1.linux-amd64/node_exporter /usr/local/bin/node_exporter
rm -rf node_exporter-1.0.1.linux-amd64.tar.gz node_exporter-1.0.1.linux-amd64

sudo cp node-exporter.service /etc/systemd/system/node-exporter.service

sudo systemctl daemon-reload
sudo systemctl enable node-exporter
sudo systemctl start node-exporter
sudo systemctl status node-exporter
```

2. Create a file then name it install-node-exporter.sh and add the following commands

```
sudo useradd --no-create-home node_exporter

wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz
tar xzf node_exporter-1.0.1.linux-amd64.tar.gz
sudo cp node_exporter-1.0.1.linux-amd64/node_exporter /usr/local/bin/node_exporter
rm -rf node_exporter-1.0.1.linux-amd64.tar.gz node_exporter-1.0.1.linux-amd64

sudo cp node-exporter.service /etc/systemd/system/node-exporter.service

sudo systemctl daemon-reload
sudo systemctl enable node-exporter
sudo systemctl start node-exporter
sudo systemctl status node-exporter
```

3. Run it with sudo ./install-node-exporter.sh

4. 5. Allow the following port from your ec2 instance

- port : 9100

5. you can check metrics by using the following url

```
<ip-address>:9100
```

## To configure prometheus for the Nodes

1. Navigate to the prometheus folder

```
sudo -i
cd /etc/prometheus

2. change the target in prometheus.yaml to the ip-addres of the ec2 you want to monitor. For example
```

global:
scrape_interval: 15s
external_labels:
monitor: 'prometheus'

scrape_configs:

- job_name: 'prometheus'
  static_configs:
  - targets: ['localhost:9090'] => - targets: ['3.24.23.45:9090']

```
3. Restart prometheus service
```

sudo systemctl restart prometheus

```

## Service Discovery
This is used to monitor all ec2 instances in a particular AZ

1. Go to ~/etc/prometheus/prometheus.yml and replace with the following code.

```

global:
scrape_interval: 15s
external_labels:
monitor: 'prometheus'

scrape_configs:

- job_name: 'node'
  ec2_sd_configs:
  - region: us-east-2
    access_key: yourkey
    secret_key: yourkey
    port: 9100

```

## Installin Grafana
1. create a file and name it install-grafana.sh and add the following code
```

sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_7.3.4_amd64.deb
sudo dpkg -i grafana_7.3.4_amd64.deb
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server
sudo systemctl enable grafana-server.service

```
2. Run the code using
```

sudo ./install-grafana.sh

```

3. Open up port 3000 on your ec2 instance
```
