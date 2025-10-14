## Graylog:

Graylog is an open-source log management and analysis platform designed to collect, index, and analyze log data from various sources in real time. It’s widely used for centralized logging, monitoring, security analysis, and troubleshooting across IT systems.



### Prerequisites:

- Required ports are open: 
    - MongoDB: tcp/27017
    - Graylog: tcp/9000 
    - OpenSearch: tcp/9200
    - Data Node: tcp/9300
    - Sidecar: tcp/5044	(Log collectors)
- We strongly recommend the use of an **XFS file system** for storage.
- To set a specific time zone on a Graylog server. 



### Key Components

_Graylog’s architecture has three main parts:_

1. Graylog Server: 
    - The core application that processes and analyzes log messages.
    - Provides the web interface and API for searching and managing logs.

2. Elasticsearch / OpenSearch:
    - Used as the data store to index and search through log data quickly.
    - Graylog queries Elasticsearch when you search or visualize logs.

3. MongoDB:
    - MongoDB serves as a metadata store for Graylog, managing configurations and other operational data. 
    - Stores configuration data, such as user accounts, roles, dashboards, stream rules, and system settings.
    - Not used for actual log messages.




### Main Features:

| Feature                              | Description                                                                       |
| ------------------------------------ | --------------------------------------------------------------------------------- |
| **Centralized Logging**              | Collects logs from multiple sources into one dashboard.                           |
| **Search & Filter**                  | Advanced search using Lucene syntax for quick log queries.                        |
| **Dashboards**                       | Visualize metrics and trends with widgets and charts.                             |
| **Alerts & Notifications**           | Define thresholds or event conditions and get alerts via email, Slack, etc.       |
| **Stream Processing**                | Route logs to specific streams based on rules (e.g., “error” logs to one stream). |
| **Role-Based Access Control (RBAC)** | Granular permissions for users and teams.                                         |
| **Integrations**                     | Works with SIEM tools, Grafana, Prometheus, and security systems.                 |





#### Supported Linux Operating Systems: 
Graylog offers official DEB and RPM package repositories for the following supported Linux operating systems:

| Operating Systems | Versions | 
| ----------------- | -------- | 
| Debian |	10, 11, 12 |
| Ubuntu |	22.04, 24.04 |
| RHEL (AlmaLinux, Rocky Linux, etc.) |	7, 8, 9 | 
| SUSE Linux Enterprise Server | 12, 15 |




#### Compatibility for Graylog Open and Enterprise with MongoDB:
Both Graylog Open and Graylog Enterprise require the following version compatibility for **MongoDB**:

| Graylog Version | Minimum MongoDB Version | Maximum MongoDB Version |
| --------------- | ----------------------- | ----------------------- |
| 4.3.x | 	3.6	  | 5.0 |
| 5.0.x | 	5.0.7 |	6.x |
| 5.1.x |	5.0.7 |	6.x |
| 5.2.x |	5.0.7 |	6.x |
| 6.0.x |	5.0.7 |	7.x |
| 6.1.x |	5.0.7 |	7.x |
| 6.2.x |	5.0.7 |	8.x |
| 6.3.x |	5.0.7 |	8.x |



#### Compatibility for Graylog Open and Enterprise with OpenSearch:
If you are using Graylog with self-managed **OpenSearch**, Graylog Open and Graylog Enterprise require the following version compatibility for software dependencies:

| **Graylog Version** | **Minimum MongoDB Version** | **Maximum MongoDB Version** | **Minimum OpenSearch Version** | **Maximum OpenSearch Version** |
| ------------------- | --------------------------- | --------------------------- | ------------------------------ | ------------------------------ |
| **4.3.x**           | 3.6                         | 5.0                         | 1.1.x                          | 1.3.x                          |
| **5.0.x**           | 5.0.7                       | 6.x                         | 1.1.x                          | 2.13.x                         |
| **5.1.x**           | 5.0.7                       | 6.x                         | 1.1.x                          | 2.13.x                         |
| **5.2.x**           | 5.0.7                       | 6.x                         | 1.1.x                          | 2.13.x                         |
| **6.0.x**           | 5.0.7                       | 7.x                         | 1.1.x                          | 2.15.x                         |
| **6.1.x**           | 5.0.7                       | 7.x                         | 1.1.x                          | 2.15.x                         |
| **6.2.x**           | 5.0.7                       | 7.x                         | 1.1.x                          | 2.19.3                         |
| **6.3.x**           | 5.0.7                       | 8.x                         | 1.1.x                          | 2.19.3                         |





---
---



## Method-1: Install Graylog: Single Graylog Node

_To set a specific time zone:_
```
timedatectl list-timezones

timedatectl set-timezone Asia/Dhaka
```




### Install MongoDB: 

```
apt install -y gnupg curl
```



```
## Import the MongoDB public key:
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor


## Create a list file for MongoDB:
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list


## Install the latest stable version of MongoDB:
apt update
apt install -y mongodb-org

apt-mark hold mongodb-org
```



_Open the MongoDB configuration file:_
```
vim /etc/mongod.conf

# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0
```


```
systemctl daemon-reload
systemctl start mongod
systemctl enable mongod

systemctl status mongod
```







### Install Data Node:

_Add the graylog repo:_
```
wget https://packages.graylog2.org/repo/packages/graylog-6.2-repository_latest.deb

sudo dpkg -i graylog-6.2-repository_latest.deb

sudo apt update
```


_Install datanode package:_
```
sudo apt install graylog-datanode
```


_Ensure that the Linux setting `vm.max_map_count` is set to at least `262144`:_
```
cat /proc/sys/vm/max_map_count
```



```
echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.d/99-graylog-datanode.conf

sudo sysctl --system

cat /proc/sys/vm/max_map_count
```



_Create randomly generated secret:_
```
openssl rand -hex 32

011d8699569e05a85ee0e9b1711xxxxxxxxxxxxxxxxx
```



#### Configure Data Node:

_Open the Data Node configuration file:_
```
vim /etc/graylog/datanode/datanode.conf


password_secret = 011d8699569e05a85ee0e9b1711xxxxxxxxxxxxxxxxx

mongodb_uri = mongodb://localhost:27017/graylog
```


```
systemctl daemon-reload
systemctl start graylog-datanode

systemctl enable graylog-datanode
systemctl status graylog-datanode
```





### Install Graylog: 

```
apt install graylog-server
```



_Create randomly generated SHA-256 password:_
```
echo -n 'admin123' | sha256sum

240be518fabd2724ddb6f04eeb1da596744xxxxxxxxxxxxxxxxxxxxxx
```



#### Configure Graylog: 

```
vim /etc/graylog/server/server.conf


password_secret = 011d8699569e05a85ee0e9b1711xxxxxxxxxxxxxxxxx

root_password_sha2 = 240be518fabd2724ddb6f04eeb1da596744xxxxxxxxxxxxxxxxxxxxxx

root_timezone = Asia/Dhaka

http_bind_address = 0.0.0.0:9000

```




```
systemctl daemon-reload

systemctl start graylog-server

systemctl enable graylog-server
systemctl status graylog-server
```



_Graylog-server logs_
```
tail -f /var/log/graylog-server/server.log
```



### Access Web Interface:

Open your browser: `http://<your-server-ip>:9000`



#### Verify Setup:
```
echo "test log" | nc -u 127.0.0.1 514
```



---
---




## Method-2: Install Graylog: Single Graylog Node


### Install MongoDB: 

```
systemctl status mongod
```


### Install OpenSearch:


```
apt update && apt -y install lsb-release ca-certificates curl gnupg2
```


_Add OpenSearch repository:_
```
curl -o- https://artifacts.opensearch.org/publickeys/opensearch.pgp | sudo gpg --dearmor --batch --yes -o /usr/share/keyrings/opensearch-keyring

echo "deb [signed-by=/usr/share/keyrings/opensearch-keyring] https://artifacts.opensearch.org/releases/bundle/opensearch/2.x/apt stable main" | sudo tee /etc/apt/sources.list.d/opensearch-2.x.list

ll /etc/apt/sources.list.d/opensearch-2.x.list
```


_Export environment variables to skip demo install:_
```
export DISABLE_INSTALL_DEMO_CONFIG=true
export DISABLE_SECURITY_PLUGIN=true
```



```
apt update

sudo env OPENSEARCH_INITIAL_ADMIN_PASSWORD=Istl@#2025# apt install opensearch

apt install -y opensearch
```


#### Configure OpenSearch: 

```
vim /etc/opensearch/opensearch.yml

cluster.name: graylog
node.name: node-01
network.host: 0.0.0.0

discovery.type: single-node
plugins.security.disabled: true
```


```
systemctl start opensearch
systemctl enable opensearch

systemctl status opensearch
```


```
curl http://localhost:9200
```



### Install Graylog: 


```
apt install graylog-server
```



#### Generate password and secret:

_To Generate SHA-256 password:_
```
echo -n 'admin123' | sha256sum

240be518fabd2724ddb6f04eeb1da5967448d7xxxxxxxxxxxxxxxxxxxxxxxxx
```


_Install pwgen utility:_
```
apt install pwgen -y   

dnf install pwgen -y  
```


_To Generate a random secret:_
```
pwgen -N 1 -s 96

DzffuqMVsR18wvkax0XTgrCoC60BJCzZlxxxxxxxxxxxxxxxxxxxxxxx
```



#### Configure Graylog: 

_Edit graylog config file:_
```
vim /etc/graylog/server/server.conf


password_secret = DzffuqMVsR18wvkax0XTgrCoC60BJCzZlxxxxxxxxxxxxxxxxxxxxxxx

root_password_sha2 = 240be518fabd2724ddb6f04eeb1da5967448d7xxxxxxxxxxxxxxxxxxxxxxxxx

root_timezone = Asia/Dhaka

http_bind_address = 0.0.0.0:9000

elasticsearch_hosts = http://127.0.0.1:9200

mongodb_uri = mongodb://localhost/graylog
```




```
systemctl start graylog-server

systemctl enable graylog-server
systemctl status graylog-server
```





### Access Web Interface:

Open your browser: `http://<your-server-ip>:9000`



### Ref: 
- [Graylog Documentation](https://go2docs.graylog.org/current/home.htm)
- [Graylog Install on Ubuntu](https://go2docs.graylog.org/6-2/downloading_and_installing_graylog/ubuntu_installation.htm)

