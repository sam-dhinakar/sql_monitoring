# sql_monitoring
To monitor SQL DB using Prometheus and Grafana


The setup for Prometheus, Grafana, and SQL Exporter for SQL Server monitoring has been completed as per your instructions. From here, we can now pull the required metrics using queries.



SQL Exporter:  https://github.com/burningalchemist/sql_exporter

To enable SQL Exporter to fetch data, we need to create a database user with the necessary permissions. Below are the required SQL commands:

USE master;  
CREATE LOGIN sql_exporter_user WITH PASSWORD = 'Password';  
CREATE USER sql_exporter_user FOR LOGIN sql_exporter_user;  

--For Accessing SQL Jobs--


USE msdb;  
GRANT SELECT ON dbo.sysjobs TO sql_exporter_user;  
GRANT SELECT ON dbo.sysjobhistory TO sql_exporter_user;  
GRANT SELECT ON dbo.sysjobsteps TO sql_exporter_user;  
GRANT SELECT ON dbo.sysjobschedules TO sql_exporter_user;  

Once this user is created, we can configure it in sql_exporter.yml to start collecting the required job metrics.

I have also attached the documentation detailing the entire setup process. Please let me know if any further modifications are needed.


==========================================================================

Installing and Configuring Prometheus, Grafana, and SQL Exporter for SQL Server Monitoring


Installing Prometheus

sudo useradd --no-create-home --shell /bin/false prometheus

sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

cd /tmp/
wget https://github.com/prometheus/prometheus/releases/download/v2.46.0/prometheus-2.46.0.linux-amd64.tar.gz
tar -xvf prometheus-2.46.0.linux-amd64.tar.gz

cd prometheus-2.46.0.linux-amd64
sudo mv console* /etc/prometheus
sudo mv prometheus.yml /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus

sudo mv prometheus /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus

sudo nano /etc/prometheus/prometheus.yml

=============================================
Creating Prometheus Systemd file @ /etc/systemd/system

Prometheus.service

[Unit]
Description=Prometheus
After=network-online.target
Wants=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target



sudo systemctl daemon-reload

sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus

Installing Grafana

sudo apt install -y apt-transport-https
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

sudo apt-get update
sudo apt-get install grafana

sudo systemctl enable --now grafana-server

Installing SQL exporter
Offical  docs: https://github.com/burningalchemist/sql_exporter

sudo wget https://github.com/free/sql_exporter/releases/download/0.5/sql_exporter-0.5.linux-amd64.tar.gz

tar -xzf sql_exporter-0.5.linux-amd64.tar.gz

sudo mv sql_exporter-0.5.linux-amd64/sql_exporter /usr/local/bin/
sudo chmod +x /usr/local/bin/sql_exporter
/usr/local/bin/sql_exporter --version

=======================================================
Creating SQL exporter Systemd file @ /etc/systemd/system

Sql_exporter.service

[Unit]
Description=SQL Exporter for Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=ubuntu
Group=ubuntu
Type=simple
ExecStart=/usr/local/bin/sql_exporter --config.file=/etc/prometheus/sql_exporter.yml
Restart=always

[Install]
WantedBy=multi-user.target


sudo systemctl daemon-reload

sudo systemctl start Sql_exporter
sudo systemctl enable Sql_exporter
sudo systemctl status Sql_exporter

================================================================


============================================================


http://localhost:9090         #Prometheus
http://localhost:3000         #grafana
http://localhost:9399/metrics #SQL exporter
