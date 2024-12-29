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

Pas deze file aan op de agent waar je de files wilt monitoren:
```
[vagrant@companyrouter etc]$ sudo nano /var/ossec/etc/ossec.conf
<!--
  Wazuh - Agent - Default configuration for almalinux 9.2
  More info at: https://documentation.wazuh.com
  Mailing list: https://groups.google.com/forum/#!forum/wazuh
-->

<ossec_config>
  <client>
    <server>
      <address>172.30.0.20</address>
      <port>1514</port>
      <protocol>tcp</protocol>
    </server>
    <config-profile>almalinux, almalinux9, almalinux9.2</config-profile>
    <notify_time>10</notify_time>
    <time-reconnect>60</time-reconnect>
    <auto_restart>yes</auto_restart>
    <crypto_method>aes</crypto_method>
    <enrollment>
      <enabled>yes</enabled>
      <agent_name>companyrouter</agent_name>
      <groups>default</groups>
      <authorization_pass_path>etc/authd.pass</authorization_pass_path>
    </enrollment>
  </client>

  <client_buffer>
    <!-- Agent buffer options -->
    <disabled>no</disabled>
    <queue_size>5000</queue_size>
    <events_per_second>500</events_per_second>
  </client_buffer>

  <!-- Policy monitoring -->
  <rootcheck>
    <disabled>no</disabled>
    <check_files>yes</check_files>
    <check_trojans>yes</check_trojans>
    <check_dev>yes</check_dev>
    <check_sys>yes</check_sys>
    <check_pids>yes</check_pids>
    <check_ports>yes</check_ports>
    <check_if>yes</check_if>

    <!-- Frequency that rootcheck is executed - every 12 hours -->
    <frequency>43200</frequency>

    <rootkit_files>etc/shared/rootkit_files.txt</rootkit_files>
    <rootkit_trojans>etc/shared/rootkit_trojans.txt</rootkit_trojans>

    <skip_nfs>yes</skip_nfs>
  </rootcheck>

  <wodle name="cis-cat">
    <disabled>yes</disabled>
    <timeout>1800</timeout>
    <interval>1d</interval>
    <scan-on-start>yes</scan-on-start>

    <java_path>wodles/java</java_path>
    <ciscat_path>wodles/ciscat</ciscat_path>
  </wodle>

  <!-- Osquery integration -->
  <wodle name="osquery">
    <disabled>yes</disabled>
    <run_daemon>yes</run_daemon>
    <log_path>/var/log/osquery/osqueryd.results.log</log_path>
    <config_path>/etc/osquery/osquery.conf</config_path>
    <add_labels>yes</add_labels>
  </wodle>

  <!-- System inventory -->
  <wodle name="syscollector">
    <disabled>no</disabled>
    <interval>1h</interval>
    <scan_on_start>yes</scan_on_start>
    <hardware>yes</hardware>
    <os>yes</os>
    <network>yes</network>
    <packages>yes</packages>
    <ports all="no">yes</ports>
    <processes>yes</processes>

    <!-- Database synchronization settings -->
    <synchronization>
      <max_eps>10</max_eps>
    </synchronization>
  </wodle>

  <sca>
    <enabled>yes</enabled>
    <scan_on_start>yes</scan_on_start>
    <interval>12h</interval>
    <skip_nfs>yes</skip_nfs>
  </sca>

  <!-- File integrity monitoring -->
  <syscheck>
    <disabled>no</disabled>

    <!-- Frequency that syscheck is executed default every 12 hours -->
    <frequency>5</frequency>

    <scan_on_start>yes</scan_on_start>

    <!-- Directories to check  (perform all possible verifications) -->
    <directories check_all="yes">/etc,/usr/bin,/usr/sbin</directories>
    <directories>/bin,/sbin,/boot</directories>
    <directories>/home/vagrant/donotchange</directories>
  <!-- Files/directories to ignore -->
    <ignore>/etc/mtab</ignore>
    <ignore>/etc/hosts.deny</ignore>
    <ignore>/etc/mail/statistics</ignore>
    <ignore>/etc/random-seed</ignore>
    <ignore>/etc/random.seed</ignore>
    <ignore>/etc/adjtime</ignore>
    <ignore>/etc/httpd/logs</ignore>
    <ignore>/etc/utmpx</ignore>
    <ignore>/etc/wtmpx</ignore>
    <ignore>/etc/cups/certs</ignore>
    <ignore>/etc/dumpdates</ignore>
    <ignore>/etc/svc/volatile</ignore>

    <!-- File types to ignore -->
    <ignore type="sregex">.log$|.swp$</ignore>

    <!-- Check the file, but never compute the diff -->
    <nodiff>/etc/ssl/private.key</nodiff>

    <skip_nfs>yes</skip_nfs>
    <skip_dev>yes</skip_dev>
    <skip_proc>yes</skip_proc>
    <skip_sys>yes</skip_sys>

    <!-- Nice value for Syscheck process -->
    <process_priority>10</process_priority>

    <!-- Maximum output throughput -->
    <max_eps>50</max_eps>

    <!-- Database synchronization settings -->
    <synchronization>
      <enabled>yes</enabled>
      <interval>5m</interval>
      <max_eps>10</max_eps>
    </synchronization>
  </syscheck>

  <!-- Log analysis -->
  <localfile>
    <log_format>command</log_format>
    <command>df -P</command>
    <frequency>360</frequency>
  </localfile>

  <localfile>
    <log_format>full_command</log_format>
    <command>netstat -tulpn | sed 's/\([[:alnum:]]\+\)\ \+[[:digit:]]\+\ \+[[:digit:]]\+\ \+\(.*\):\([[:digit:]]*\)\ \+\([0-9\.\:\*]\+\).\+\ \([[:digit:]]*\/[[:alnum:]\-]*\).*/\1 \2 == \3 == \4 \5/' | sort -k 4 -g | sed 's/ == \(.*\) ==/:\1/' | sed 1,2d</command>
    <alias>netstat listening ports</alias>
    <frequency>360</frequency>
  </localfile>

  <localfile>
    <log_format>full_command</log_format>
    <command>last -n 20</command>
    <frequency>360</frequency>
  </localfile>

  <!-- Active response -->
  <active-response>
    <disabled>no</disabled>
    <ca_store>etc/wpk_root.pem</ca_store>
    <ca_verification>yes</ca_verification>
  </active-response>

  <!-- Choose between "plain", "json", or "plain,json" for the format of internal logs -->
  <logging>
    <log_format>plain</log_format>
  </logging>

</ossec_config>

<ossec_config>
  <localfile>
    <log_format>audit</log_format>
    <location>/var/log/audit/audit.log</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/ossec/logs/active-responses.log</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/messages</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/secure</location>
  </localfile>

</ossec_config>
```

Restart Wazuh agent:
```
[vagrant@companyrouter logs]$ sudo /var/ossec/bin/wazuh-control restart
Killing wazuh-modulesd...
Killing wazuh-logcollector...
Killing wazuh-syscheckd...
Killing wazuh-agentd...
Killing wazuh-execd...
Wazuh v4.7.1 Stopped
Starting Wazuh v4.7.1...
Started wazuh-execd...
Started wazuh-agentd...
Started wazuh-syscheckd...
Started wazuh-logcollector...
Started wazuh-modulesd...
Completed.
```

DEMO:

1) Maak een file aan:
```
[vagrant@companyrouter etc]$ sudo touch test.txt
```
2) Schrijf iets in het bestand:
```
[vagrant@companyrouter etc]$ sudo echo "test" >> /etc/test.txt
```
3) Kijk in de logs op da manager node of je deze zaken kan zien:
```
vagrant@Wazuh:~$ sudo grep syscheck /var/ossec/logs/alerts/alerts.log
** Alert 1735476115.48096: - ossec,syscheck,syscheck_entry_added,syscheck_file,pci_dss_11.5,gpg13_4.11,gdpr_II_5.1.f,hipaa_164.312.c.1,hipaa_164.312.c.2,nist_800_53_SI.7,tsc_PI1.4,tsc_PI1.5,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,
2024 Dec 29 12:41:55 (companyrouter) any->syscheck
** Alert 1735476122.50132: - ossec,syscheck,syscheck_entry_modified,syscheck_file,pci_dss_11.5,gpg13_4.11,gdpr_II_5.1.f,hipaa_164.312.c.1,hipaa_164.312.c.2,nist_800_53_SI.7,tsc_PI1.4,tsc_PI1.5,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,
2024 Dec 29 12:42:02 (companyrouter) any->syscheck
** Alert 1735476124.52267: - ossec,syscheck,syscheck_entry_modified,syscheck_file,pci_dss_11.5,gpg13_4.11,gdpr_II_5.1.f,hipaa_164.312.c.1,hipaa_164.312.c.2,nist_800_53_SI.7,tsc_PI1.4,tsc_PI1.5,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,
2024 Dec 29 12:42:04 (companyrouter) any->syscheck
** Alert 1735476195.57671: - ossec,syscheck,syscheck_entry_deleted,syscheck_file,pci_dss_11.5,gpg13_4.11,gdpr_II_5.1.f,hipaa_164.312.c.1,hipaa_164.312.c.2,nist_800_53_SI.7,tsc_PI1.4,tsc_PI1.5,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,
2024 Dec 29 12:43:15 (companyrouter) any->syscheck
** Alert 1735476401.66192: - ossec,syscheck,syscheck_entry_added,syscheck_file,pci_dss_11.5,gpg13_4.11,gdpr_II_5.1.f,hipaa_164.312.c.1,hipaa_164.312.c.2,nist_800_53_SI.7,tsc_PI1.4,tsc_PI1.5,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,
2024 Dec 29 12:46:41 (companyrouter) any->syscheck
** Alert 1735476404.68228: - ossec,syscheck,syscheck_entry_modified,syscheck_file,pci_dss_11.5,gpg13_4.11,gdpr_II_5.1.f,hipaa_164.312.c.1,hipaa_164.312.c.2,nist_800_53_SI.7,tsc_PI1.4,tsc_PI1.5,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,
2024 Dec 29 12:46:44 (companyrouter) any->syscheck
** Alert 1735476408.70363: - ossec,syscheck,syscheck_entry_modified,syscheck_file,pci_dss_11.5,gpg13_4.11,gdpr_II_5.1.f,hipaa_164.312.c.1,hipaa_164.312.c.2,nist_800_53_SI.7,tsc_PI1.4,tsc_PI1.5,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,
2024 Dec 29 12:46:48 (companyrouter) any->syscheck
Dec 29 12:53:48 Wazuh sudo:  vagrant : TTY=pts/0 ; PWD=/home/vagrant ; USER=root ; COMMAND=/usr/bin/grep syscheck /var/ossec/logs/alerts/alerts.log
command: /usr/bin/grep syscheck /var/ossec/logs/alerts/alerts.log
** Alert 1735477120.91677: - ossec,syscheck,syscheck_entry_added,syscheck_file,pci_dss_11.5,gpg13_4.11,gdpr_II_5.1.f,hipaa_164.312.c.1,hipaa_164.312.c.2,nist_800_53_SI.7,tsc_PI1.4,tsc_PI1.5,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,
2024 Dec 29 12:58:40 (companyrouter) any->syscheck
** Alert 1735477147.93700: - ossec,syscheck,syscheck_entry_modified,syscheck_file,pci_dss_11.5,gpg13_4.11,gdpr_II_5.1.f,hipaa_164.312.c.1,hipaa_164.312.c.2,nist_800_53_SI.7,tsc_PI1.4,tsc_PI1.5,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,
2024 Dec 29 12:59:07 (companyrouter) any->syscheck
```

- What is meant with Regulatory Compliance? Give 2 frameworks that can be explored.

Regulatory Compliance verwijst naar het naleven van wetten, regels, voorschriften en richtlijnen die zijn opgelegd door overheidsinstanties of andere regelgevende organen. Organisaties moeten voldoen aan deze vereisten om juridische boetes te vermijden, vertrouwen op te bouwen en risico's te minimaliseren. Het is een essentieel onderdeel van risicobeheer en goede bedrijfsvoering.

Twee voorbeelden van frameworks die kunnen worden onderzocht zijn:

GDPR (General Data Protection Regulation):
Dit is een Europese wetgeving die regels stelt voor het verzamelen, verwerken en opslaan van persoonsgegevens van individuen binnen de Europese Unie. Het doel is om de privacy van personen te beschermen.

ISO 27001:
Dit is een internationaal erkend framework voor informatiebeveiliging. Het biedt richtlijnen en eisen voor het opzetten, implementeren, onderhouden en continu verbeteren van een Information Security Management System (ISMS).

Beide frameworks helpen organisaties om hun compliance-inspanningen te structureren en risico's te beheersen.

- Threat hunting: discover the CLI commands that were executed on your machines. For example perform an install of a package or a download a file and create an overview that lists all commands that have been ran on that machine. Create a demo for Linux and optional for a Windows host.

1) Zorg dat auditd installed is op de machine waarop je wilt thread hunten
```
[vagrant@companyrouter etc]$ sudo systemctl status auditd
● auditd.service - Security Auditing Service
     Loaded: loaded (/usr/lib/systemd/system/auditd.service; enabled; preset: enabled)
     Active: active (running) since Sun 2024-12-29 12:20:05 UTC; 1h 5min ago
       Docs: man:auditd(8)
             https://github.com/linux-audit/audit-documentation
   Main PID: 629 (auditd)
      Tasks: 2 (limit: 24482)
     Memory: 5.8M
        CPU: 79ms
     CGroup: /system.slice/auditd.service
             └─629 /sbin/auditd

Dec 29 12:20:05 companyrouter augenrules[649]: enabled 1
Dec 29 12:20:05 companyrouter augenrules[649]: failure 1
Dec 29 12:20:05 companyrouter augenrules[649]: pid 629
Dec 29 12:20:05 companyrouter augenrules[649]: rate_limit 0
Dec 29 12:20:05 companyrouter augenrules[649]: backlog_limit 8192
Dec 29 12:20:05 companyrouter augenrules[649]: lost 0
Dec 29 12:20:05 companyrouter augenrules[649]: backlog 4
Dec 29 12:20:05 companyrouter augenrules[649]: backlog_wait_time 60000
Dec 29 12:20:05 companyrouter augenrules[649]: backlog_wait_time_actual 0
Dec 29 12:20:05 companyrouter systemd[1]: Started Security Auditing Service.
```

2) Maak een nieuwe audit rule aan:
```
[vagrant@companyrouter etc]$ sudo nano /etc/audit/rules.d/audit.rules
## First rule - delete all
-D

## Increase the buffers to survive stress events.
## Make this bigger for busy systems
-b 8192

## This determine how long to wait in burst of events
--backlog_wait_time 60000

## Set failure mode to syslog
-f 1

### shell & commando gebruik
-a always,exit -F arch=b64 -S execve -k shell_activity
```

3) Doe volgende commandos om de rule actief te krijgen:
```
[vagrant@companyrouter etc]$ sudo augenrules --load
No rules
enabled 1
failure 1
pid 629
rate_limit 0
backlog_limit 8192
lost 0
backlog 4
backlog_wait_time 60000
backlog_wait_time_actual 0
enabled 1
failure 1
pid 629
rate_limit 0
backlog_limit 8192
lost 0
backlog 4
backlog_wait_time 60000
backlog_wait_time_actual 0
enabled 1
failure 1
pid 629
rate_limit 0
backlog_limit 8192
lost 0
backlog 4
backlog_wait_time 60000
backlog_wait_time_actual 0
[vagrant@companyrouter etc]$ sudo auditctl -l
-a always,exit -F arch=b64 -S execve -F key=shell_activity
```

4) Zorg dat in de Wazuh agents config dit stukje in de file staat (nrml default)
```
[vagrant@companyrouter etc]$ sudo nano /var/ossec/etc/ossec.conf
 <localfile>
    <log_format>audit</log_format>
    <location>/var/log/audit/audit.log</location>
  </localfile>
```

5) Bekijk nu de logs:
```
[vagrant@companyrouter etc]$ sudo tail -f /var/log/audit/audit.log
type=USER_ACCT msg=audit(1735478942.382:716): pid=6425 uid=1000 auid=1000 ses=1 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=PAM:accounting grantors=pam_unix acct="vagrant" exe="/usr/bin/sudo" hostname=? addr=? terminal=/dev/pts/0 res=success'UID="vagrant" AUID="vagrant"
type=USER_CMD msg=audit(1735478942.382:717): pid=6425 uid=1000 auid=1000 ses=1 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='cwd="/etc" cmd=7461696C202D66202F7661722F6C6F672F61756469742F61756469742E6C6F67 exe="/usr/bin/sudo" terminal=pts/0 res=success'UID="vagrant" AUID="vagrant"
type=CRED_REFR msg=audit(1735478942.382:718): pid=6425 uid=1000 auid=1000 ses=1 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=PAM:setcred grantors=pam_env,pam_unix acct="root" exe="/usr/bin/sudo" hostname=? addr=? terminal=/dev/pts/0 res=success'UID="vagrant" AUID="vagrant"
type=USER_START msg=audit(1735478942.385:719): pid=6425 uid=1000 auid=1000 ses=1 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=PAM:session_open grantors=pam_keyinit,pam_limits,pam_systemd,pam_unix acct="root" exe="/usr/bin/sudo" hostname=? addr=? terminal=/dev/pts/0 res=success'UID="vagrant" AUID="vagrant"
type=SYSCALL msg=audit(1735478942.385:720): arch=c000003e syscall=59 success=yes exit=0 a0=55b656b93ca8 a1=55b656b914d8 a2=55b656bcad80 a3=0 items=2 ppid=6425 pid=6427 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=1 comm="tail" exe="/usr/bin/tail" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="shell_activity"ARCH=x86_64 SYSCALL=execve AUID="vagrant" UID="root" GID="root" EUID="root" SUID="root" FSUID="root" EGID="root" SGID="root" FSGID="root"
type=EXECVE msg=audit(1735478942.385:720): argc=3 a0="tail" a1="-f" a2="/var/log/audit/audit.log"
type=CWD msg=audit(1735478942.385:720): cwd="/etc"
type=PATH msg=audit(1735478942.385:720): item=0 name="/bin/tail" inode=50495650 dev=08:04 mode=0100755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:bin_t:s0 nametype=NORMAL cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0OUID="root" OGID="root"
type=PATH msg=audit(1735478942.385:720): item=1 name="/lib64/ld-linux-x86-64.so.2" inode=221238 dev=08:04 mode=0100755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:ld_so_t:s0 nametype=NORMAL cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0OUID="root" OGID="root"
type=PROCTITLE msg=audit(1735478942.385:720): proctitle=7461696C002D66002F7661722F6C6F672F61756469742F61756469742E6C6F67
```