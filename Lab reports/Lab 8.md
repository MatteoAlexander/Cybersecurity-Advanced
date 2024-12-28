# Lab 8: SIEM

## Wazuh server

Installeer Wazuh:
```
vagrant@Wazuh:~$ curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
vagrant@Wazuh:~$ curl -sO https://packages.wazuh.com/4.7/config.yml
vagrant@Wazuh:~$ ls
config.yml  wazuh-install.sh
vagrant@Wazuh:~$ chmod +x wazuh-install.sh
```

Config file:
```
nodes:
  # Wazuh indexer nodes
  indexer:
    - name: node-1
      ip: "172.30.0.20"

  # Wazuh server nodes
  server:
    - name: wazuh-1
      ip: "172.30.0.20"

  # Wazuh dashboard nodes
  dashboard:
    - name: dashboard
      ip: "172.30.0.20"
```

Run de config file:
```
vagrant@Wazuh:~$ sudo ./wazuh-install.sh --generate-config-files --ignore-check
28/12/2024 15:28:33 INFO: Starting Wazuh installation assistant. Wazuh version: 4.7.5
28/12/2024 15:28:33 INFO: Verbose logging redirected to /var/log/wazuh-install.log
28/12/2024 15:28:48 WARNING: Hardware and system checks ignored.
28/12/2024 15:28:48 INFO: --- Configuration files ---
28/12/2024 15:28:48 INFO: Generating configuration files.
28/12/2024 15:28:48 INFO: Created wazuh-install-files.tar. It contains the Wazuh cluster key, certificates, and passwords necessary for installation.
vagrant@Wazuh:~$ ls
wazuh-install-files.tar  wazuh-install.sh
```

Indexer installatie:
```
vagrant@Wazuh:~$ sudo ./wazuh-install.sh --wazuh-indexer node-1 -i -o
28/12/2024 15:40:41 INFO: Starting Wazuh installation assistant. Wazuh version: 4.7.5
28/12/2024 15:40:41 INFO: Verbose logging redirected to /var/log/wazuh-install.log
28/12/2024 15:40:42 INFO: --- Removing existing Wazuh installation ---
28/12/2024 15:40:42 INFO: Removing Wazuh indexer.
28/12/2024 15:40:48 INFO: Wazuh indexer removed.
28/12/2024 15:40:48 INFO: Installation cleaned.
28/12/2024 15:40:53 WARNING: Hardware and system checks ignored.
28/12/2024 15:41:00 INFO: Wazuh repository added.
28/12/2024 15:41:00 INFO: --- Wazuh indexer ---
28/12/2024 15:41:00 INFO: Starting Wazuh indexer installation.
28/12/2024 15:41:41 INFO: Wazuh indexer installation finished.
28/12/2024 15:41:41 INFO: Wazuh indexer post-install configuration finished.
28/12/2024 15:41:41 INFO: Starting service wazuh-indexer.
28/12/2024 15:42:05 INFO: wazuh-indexer service started.
28/12/2024 15:42:05 INFO: Initializing Wazuh indexer cluster security settings.
28/12/2024 15:42:08 INFO: Wazuh indexer cluster initialized.
28/12/2024 15:42:08 INFO: Installation finished.
```

Cluster installatie
```
vagrant@Wazuh:~$ sudo ./wazuh-install.sh --start-cluster -i
28/12/2024 15:42:14 INFO: Starting Wazuh installation assistant. Wazuh version: 4.7.5
28/12/2024 15:42:14 INFO: Verbose logging redirected to /var/log/wazuh-install.log
28/12/2024 15:42:22 WARNING: Hardware and system checks ignored.
28/12/2024 15:42:29 INFO: Wazuh indexer cluster security configuration initialized.
28/12/2024 15:43:11 INFO: Wazuh indexer cluster started.
```

Cluster testen
```
vagrant@Wazuh:~$ sudo tar -axf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt -O | grep -P "\'admin\'" -A 1
  indexer_username: 'admin'
  indexer_password: 'LU?N99r?Hy7hZbui*4YUardW*bV60bDD'
```
```
vagrant@Wazuh:~$ curl -k -u admin:LU?N99r?Hy7hZbui*4YUardW*bV60bDD https://172.30.0.20:9200
{
  "name" : "node-1",
  "cluster_name" : "wazuh-indexer-cluster",
  "cluster_uuid" : "zx-Gn0MFRkuyf0p5YLUe4g",
  "version" : {
    "number" : "7.10.2",
    "build_type" : "rpm",
    "build_hash" : "db90a415ff2fd428b4f7b3f800a51dc229287cb4",
    "build_date" : "2023-06-03T06:24:25.112415503Z",
    "build_snapshot" : false,
    "lucene_version" : "9.6.0",
    "minimum_wire_compatibility_version" : "7.10.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "The OpenSearch Project: https://opensearch.org/"
}
vagrant@Wazuh:~$ curl -k -u admin:LU?N99r?Hy7hZbui*4YUardW*bV60bDD https://172.30.0.20:9200/_cat/nodes?v
ip          heap.percent ram.percent cpu load_1m load_5m load_15m node.role node.roles                               cluster_manager name
172.30.0.20           14          95  14    0.46    0.66     0.31 dimr      data,ingest,master,remote_cluster_client *               node-1
```


Wazuh server cluster installatie:
```
vagrant@Wazuh:~$ sudo ./wazuh-install.sh --wazuh-server wazuh-1 -i
28/12/2024 15:45:31 INFO: Starting Wazuh installation assistant. Wazuh version: 4.7.5
28/12/2024 15:45:31 INFO: Verbose logging redirected to /var/log/wazuh-install.log
28/12/2024 15:45:39 WARNING: Hardware and system checks ignored.
28/12/2024 15:45:44 INFO: Wazuh repository added.
28/12/2024 15:45:44 INFO: --- Wazuh server ---
28/12/2024 15:45:44 INFO: Starting the Wazuh manager installation.
28/12/2024 15:46:33 INFO: Wazuh manager installation finished.
28/12/2024 15:46:33 INFO: Starting service wazuh-manager.
28/12/2024 15:46:53 INFO: wazuh-manager service started.
28/12/2024 15:46:53 INFO: Starting Filebeat installation.
28/12/2024 15:47:05 INFO: Filebeat installation finished.
28/12/2024 15:47:05 INFO: Filebeat post-install configuration finished.
28/12/2024 15:47:13 INFO: Starting service filebeat.
28/12/2024 15:47:14 INFO: filebeat service started.
28/12/2024 15:47:14 INFO: Installation finished.
```

Wazuh dashboard installeren:
```
vagrant@Wazuh:~$ sudo ./wazuh-install.sh --wazuh-dashboard dashboard -i
28/12/2024 15:47:22 INFO: Starting Wazuh installation assistant. Wazuh version: 4.7.5
28/12/2024 15:47:22 INFO: Verbose logging redirected to /var/log/wazuh-install.log
28/12/2024 15:47:30 WARNING: Hardware and system checks ignored.
28/12/2024 15:47:30 INFO: Wazuh web interface port will be 443.
28/12/2024 15:47:35 INFO: Wazuh repository added.
28/12/2024 15:47:35 INFO: --- Wazuh dashboard ----
28/12/2024 15:47:35 INFO: Starting Wazuh dashboard installation.
28/12/2024 15:48:26 INFO: Wazuh dashboard installation finished.
28/12/2024 15:48:26 INFO: Wazuh dashboard post-install configuration finished.
28/12/2024 15:48:26 INFO: Starting service wazuh-dashboard.
28/12/2024 15:48:27 INFO: wazuh-dashboard service started.
28/12/2024 15:48:51 INFO: Initializing Wazuh dashboard web application.
28/12/2024 15:48:52 INFO: Wazuh dashboard web application initialized.
28/12/2024 15:48:52 INFO: --- Summary ---
28/12/2024 15:48:52 INFO: You can access the web interface https://172.30.0.20:443
    User: admin
    Password: LU?N99r?Hy7hZbui*4YUardW*bV60bDD
28/12/2024 15:48:52 INFO: Installation finished.
```

Credentials bekijken:
```
vagrant@Wazuh:~$ sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
wazuh-install-files/wazuh-passwords.txt
# Admin user for the web user interface and Wazuh indexer. Use this user to log in to Wazuh dashboard
  indexer_username: 'admin'
  indexer_password: 'LU?N99r?Hy7hZbui*4YUardW*bV60bDD'

# Wazuh dashboard user for establishing the connection with Wazuh indexer
  indexer_username: 'kibanaserver'
  indexer_password: 'nT4YjT5TqLd0xcn6VZPV0?Rc4PbCafel'

# Regular Dashboard user, only has read permissions to all indices and all permissions on the .kibana index
  indexer_username: 'kibanaro'
  indexer_password: 'R1QCKczq.?*aU1AoweuJ8Lh2*KaKXBY*'

# Filebeat user for CRUD operations on Wazuh indices
  indexer_username: 'logstash'
  indexer_password: 'GdpPOx?e2Qxox5Cig4*qQ8?IcfH4QMx0'

# User with READ access to all indices
  indexer_username: 'readall'
  indexer_password: 'dd+fnya9?0dP2QS+txX8L.o3SShZ*Hf7'

# User with permissions to perform snapshot and restore operations
  indexer_username: 'snapshotrestore'
  indexer_password: 'kEs+j7id1rFKrpCK9Pmb0N?8W9HRKL+l'

# Password for wazuh API user
  api_username: 'wazuh'
  api_password: 'jrY26bUmjgu2dfC9nPGNm1h.MKrQj7+z'

# Password for wazuh-wui API user
  api_username: 'wazuh-wui'
  api_password: 'Ur5ZPC*8q*q4M.4FAhI8UelBQHU?yFun'
```



## Wazug agents


Install de agent op companyrouter (RedHat): 
```
[vagrant@companyrouter ansible]$ curl -o wazuh-agent-4.7.1-1.x86_64.rpm https://packages.wazuh.com/4.x/yum/wazuh-agent-4.7.1-1.x86_64.rpm && sudo WAZUH_MANAGER='172.30.0.20' WAZUH_AGENT_GROUP='default' WAZUH_AGENT_NAME='companyrouter' rpm -ihv wazuh-agent-4.7.1-1.x86_64.rpm
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 9246k  100 9246k    0     0  8514k      0  0:00:01  0:00:01 --:--:-- 8514k
warning: wazuh-agent-4.7.1-1.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 29111145: NOKEY
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:wazuh-agent-4.7.1-1              ################################# [100%]
[vagrant@companyrouter ansible]$ sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent --now
Created symlink /etc/systemd/system/multi-user.target.wants/wazuh-agent.service → /usr/lib/systemd/system/wazuh-agent.service.
[vagrant@companyrouter ansible]$ sudo systemctl status wazuh-agent
● wazuh-agent.service - Wazuh agent
     Loaded: loaded (/usr/lib/systemd/system/wazuh-agent.service; enabled; preset: disabled)
     Active: active (running) since Sat 2024-12-28 16:09:58 UTC; 9s ago
    Process: 8064 ExecStart=/usr/bin/env /var/ossec/bin/wazuh-control start (code=exited, status=0/SUCCESS)
      Tasks: 28 (limit: 24482)
     Memory: 100.4M
        CPU: 671ms
     CGroup: /system.slice/wazuh-agent.service
             ├─8092 /var/ossec/bin/wazuh-execd
             ├─8104 /var/ossec/bin/wazuh-agentd
             ├─8118 /var/ossec/bin/wazuh-syscheckd
             ├─8132 /var/ossec/bin/wazuh-logcollector
             └─8150 /var/ossec/bin/wazuh-modulesd

Dec 28 16:09:51 companyrouter systemd[1]: Starting Wazuh agent...
Dec 28 16:09:51 companyrouter env[8064]: Starting Wazuh v4.7.1...
Dec 28 16:09:52 companyrouter env[8064]: Started wazuh-execd...
Dec 28 16:09:53 companyrouter env[8064]: Started wazuh-agentd...
Dec 28 16:09:54 companyrouter env[8064]: Started wazuh-syscheckd...
Dec 28 16:09:55 companyrouter env[8064]: Started wazuh-logcollector...
Dec 28 16:09:56 companyrouter env[8064]: Started wazuh-modulesd...
Dec 28 16:09:58 companyrouter env[8064]: Completed.
Dec 28 16:09:58 companyrouter systemd[1]: Started Wazuh agent.
```

Op de Wazuh vm kan je dan kijken welke machines er allemaal een Wazzuh agent hebben:
```
vagrant@Wazuh:~$ sudo /var/ossec/bin/agent_control -l

Wazuh agent_control. List of available agents:
   ID: 000, Name: Wazuh (server), IP: 127.0.0.1, Active/Local
   ID: 001, Name: companyrouter, IP: any, Active

List of agentless devices:
```

Install de agent op companyrouter (Alphine):
```
dns:~$ sudo wget -O /etc/apk/keys/alpine-devel@wazuh.com-633d7457.rsa.pub https://packages.wazuh.com/key/alpine-devel%40wazuh.com-633d7457.rsa.pub
```
```
dns:~$ echo "https://packages.wazuh.com/4.x/alpine/v3.12/main" | sudo tee -a /etc/apk/repositories > /dev/null
```
```
dns:~$ sudo apk update
```
```
dns:~$ sudo apk add wazuh-agent=4.7.0-r1
```
```
dns:~$ export WAZUH_MANAGER="172.30.0.20" && sudo sed -i "s|MANAGER_IP|$WAZUH_MANAGER|g" /var/ossec/etc/ossec.conf
```
```
dns:~$ sudo /var/ossec/bin/wazuh-control start
```


Controle:
```
vagrant@Wazuh:~$ sudo /var/ossec/bin/agent_control -l

Wazuh agent_control. List of available agents:
   ID: 000, Name: Wazuh (server), IP: 127.0.0.1, Active/Local
   ID: 001, Name: companyrouter, IP: any, Active
   ID: 002, Name: web, IP: any, Active
   ID: 003, Name: database, IP: any, Active
   ID: 004, Name: dns, IP: any, Active
   ID: 005, Name: employee, IP: any, Active

List of agentless devices:
```

- What is File Integrity Monitoring? Try to monitor the home directory of a user on a specific machine. Create a demo.

File Integrity Monitoring (FIM) is een beveiligingsproces dat veranderingen in bestanden en mappen controleert om ongewenste of schadelijke wijzigingen te detecteren. Het wordt vaak gebruikt om de integriteit van gevoelige of kritieke bestanden te waarborgen en beveiligingsincidenten snel op te sporen.

- What is meant with Regulatory Compliance? Give 2 frameworks that can be explored.


- Threat hunting: discover the CLI commands that were executed on your machines. For example perform an install of a package or a download a file and create an overview that lists all commands that have been ran on that machine. Create a demo for Linux and optional for a Windows host.