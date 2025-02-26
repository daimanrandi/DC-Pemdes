# DC-103-Prometheus-Grafana-installation-Guide
Prometheus &amp; Grafana installation guide on Ubuntu 24.04 LTS

## 1 : Creating ``Prometheus`` System Users and Directory
Create a system user for ``Prometheus`` using below commnds:
```
sudo useradd --no-create-home --shell /bin/false prometheus
```
Create the directories in which we will be storing our configuration files and libraries:
```
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```
Set the ownership of the ``/var/lib/prometheus`` directory with below command:
```
sudo chown prometheus:prometheus /var/lib/prometheus
```

## 2 : Download ``Prometheus`` Binary File
Download the latest version of ``Prometheus`` from ``https://prometheus.io/download/``, copy the download link as per our Operating System
We need to inside ``/tmp``:
```
cd /tmp/
```
Download the ``Prometheus`` setup using wget
```
wget https://github.com/prometheus/prometheus/releases/download/v3.2.0/prometheus-3.2.0.linux-amd64.tar.gz
```
Extract the files using tar :
```
tar -xvf prometheus-3.2.0.linux-amd64.tar.gz
```
Move the configuration file and set the owner to the prometheus user:
```
cd prometheus-3.2.0.linux-amd64
sudo mv console* /etc/prometheus
sudo mv prometheus.yml /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus
```
Move the binaries and set the owner:
```
sudo mv prometheus /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
```

## 3 : ``Prometheus`` configuration file
We have already copied ``/opt/prometheus-3.2.0.linux-amd64/prometheus.yml`` file ``/etc/prometheus`` directory, verify if it present and should look like below and modify it as per your requirement.
```
sudo nano /etc/prometheus/prometheus.yml
```

## 4 : Creating ``Prometheus`` Systemd file
Create the service file using below command:
```
sudo nano /etc/systemd/system/prometheus.service
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
Reload systemd:
```
sudo systemctl daemon-reload
```
Start, Enable, Status Prometheus service:
```
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus
```

## 5 : Accessing ``Prometheus`` in Browser
Check weather port 9090 is UP in firewall.
Use below command to enable ``prometheus`` service in firewall
```
sudo ufw allow 9090/tcp
```
Now ``Prometheus`` service is ready to run and we can access it from any web browser.
```
http://server-IP-or-Hostname:9090
```
We can check the target at the ``Prometheus`` dashboards. As we can observe Current state is UP and we can also see the last scrape.

## 6 : Download ``Node Exporter``
Go to the official release page of ``Prometheus Node Exporter`` at ``https://github.com/prometheus/node_exporter/releases`` and copy the link of the latest version of the ``Node Exporter`` package according to your OS type.
```
cd /tmp
```
Now lets run the copied URL with wget command
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.0/node_exporter-1.9.0.linux-amd64.tar.gz
```
Unzip the downloaded the file using below command
```
sudo tar xvfz node_exporter-*.*-amd64.tar.gz
```
Move the binary file of ``node exporter`` to ``/usr/local/bin`` location
```
sudo mv node_exporter-*.*-amd64/node_exporter /usr/local/bin/
```
Create a ``node_exporter`` user to run the ``node exporter`` service
```
sudo useradd -rs /bin/false node_exporter
```

## 7 : Creating ``Node Exporter`` Systemd service
Create a ``node_exporter`` service file in the ``/etc/systemd/system`` directory
```
sudo nano /etc/systemd/system/node_exporter.service
```
```
[Unit]

Description=Node Exporter

After=network.target

 

[Service]

User=node_exporter

Group=node_exporter

Type=simple

ExecStart=/usr/local/bin/node_exporter

 

[Install]

WantedBy=multi-user.target
```
Now lets start and enable the ``node_exporter`` service using below commands
```
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
sudo systemctl status node_exporter
```

## 8 : Configure the ``Node Exporter`` as a ``Prometheus`` target
Now to scrape the ``node_exporter`` lets instruct the ``Prometheus`` by making a minor change in ``prometheus.yml`` file

So go to ``etc/prometheus`` and open ``prometheus.yml``
```
sudo nano /etc/prometheus/prometheus.yml
```
```
- job_name: 'Node_Exporter'

    scrape_interval: 5s

    static_configs:

      - targets: ['<Server_IP_of_Node_Exporter_Machine>:9100']
```
Now restart the ``Prometheus`` Service
```
sudo systemctl restart prometheus
```
Hit the URL in your web browser to check weather our target is successfully scraped by ``Prometheus`` or not

![Screenshot_50](https://github.com/user-attachments/assets/14d2b6e8-9c91-4d5b-8d7d-c214f8db6659)

## 9 : Install ``Grafana`` on Ubuntu 24.04 LTS
Add the ``Grafana`` GPG key in Ubuntu using wget
```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```
Next, add the ``Grafana`` repository to your APT sources:
```
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
```
Refresh your APT cache to update your package lists:
```
sudo apt update
```
You can now proceed with the installation:
```
sudo apt install grafana
```
Once ``Grafana`` is installed, use systemctl to start the ``Grafana`` server:
```
sudo systemctl start grafana-server
```
Next, verify that ``Grafana`` is running by checking the service’s status:
```
sudo systemctl status grafana-server
sudo systemctl enable grafana-server
```
Access ``Grafana`` Dashboard in your browser, type server IP or Name followed by ``grafana`` default port 3000.
```
http://your_ip:3000
```
Here you can see Login page of ``Grafana`` now you will have to login with below ``Grafana`` default UserName and Password.

| username | admin |
|--|--|
| password | admin |

## 10 : Configure ``Prometheus`` as ``Grafana`` DataSource
Click on Add Data sources and select ``Prometheus``

![Screenshot_51](https://github.com/user-attachments/assets/a27d5522-14e2-497c-9521-bb91238d0d92)

Now configure ``Prometheus`` data source by providing ``Prometheus`` URL

![Screenshot_52](https://github.com/user-attachments/assets/95fc80e9-d255-477d-8f52-78c17bb8d9d0)

Now click on ``Save`` & ``test`` so it will prompt a message Data Source is working.

## 11 : Creating ``Grafana`` Dashboard to Monitor Linux Server
So we will use ``14513`` to import Grafana.com, Lets come to ``Grafana`` Home page and you can see a “+” icon. Click on that and select ``“Import”``

![Screenshot_53](https://github.com/user-attachments/assets/91c2613d-18ac-42e5-8e1e-3d7eb578f3f2)

Now provide the Grafana.com Dashboard ID which is ``14513`` and click on ``Load``

![Screenshot_54](https://github.com/user-attachments/assets/59ff877e-4186-4882-85bc-4a9f18ad0da1)

Now provide the name and select the ``Prometheus`` Datasource and click on ``Import``.

![Screenshot_55](https://github.com/user-attachments/assets/e488ee9f-56c4-44f1-80fd-0a6bb121e630)

