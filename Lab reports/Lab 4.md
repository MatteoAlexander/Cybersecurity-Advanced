# Lab 4: IDS/IPS

## IDS/IPS

- Ask yourself which system (or systems) in the network layout of the company would be best suited to install IDS/IPS software on. Revert back to the original network diagram of the initial setup and answer the same questions as well.

Aangezien we zoveel mogelijk verkeer willen zien is de companyrouter de beste keus.  

- What traffic can be seen?

De data die volgens de suricata rules gemonitord moet worden en dat door de companyrouter passeert.  

- What traffic (if any) will be missed and when?

Data die niet door de companyrouter gaat.

- For this exercise, disable the firewall so that you can reach the database. Install tcpdump on the machine where you will install Suricata on and increase the memory (temporary if needed) to 4GB. Reboot if necessary.

- Verify that you see packets (in tcpdump) from red to the database. Try this by issuing a ping and by using the hydra mysql attack as seen previously. Are you able to see this traffic in tcpdump? What about a ping between the webserver and the database?
```bash
[vagrant@companyrouter ~]$ sudo tcpdump -i any icmp -n
tcpdump: data link type LINUX_SLL2
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
14:57:19.533598 eth1  In  IP 192.168.62.166 > 172.30.0.15: ICMP echo request, id 433, seq 1, length 64
14:57:19.533633 eth2  Out IP 192.168.62.166 > 172.30.0.15: ICMP echo request, id 433, seq 1, length 64
14:57:19.533865 eth2  In  IP 172.30.0.15 > 192.168.62.166: ICMP echo reply, id 433, seq 1, length 64
14:57:19.533873 eth1  Out IP 172.30.0.15 > 192.168.62.166: ICMP echo reply, id 433, seq 1, length 64
14:57:20.544143 eth1  In  IP 192.168.62.166 > 172.30.0.15: ICMP echo request, id 433, seq 2, length 64
14:57:20.544170 eth2  Out IP 192.168.62.166 > 172.30.0.15: ICMP echo request, id 433, seq 2, length 64
14:57:20.544412 eth2  In  IP 172.30.0.15 > 192.168.62.166: ICMP echo reply, id 433, seq 2, length 64
14:57:20.544417 eth1  Out IP 172.30.0.15 > 192.168.62.166: ICMP echo reply, id 433, seq 2, length 64
14:57:21.568154 eth1  In  IP 192.168.62.166 > 172.30.0.15: ICMP echo request, id 433, seq 3, length 64
14:57:21.568177 eth2  Out IP 192.168.62.166 > 172.30.0.15: ICMP echo request, id 433, seq 3, length 64
14:57:21.568361 eth2  In  IP 172.30.0.15 > 192.168.62.166: ICMP echo reply, id 433, seq 3, length 64
14:57:21.568366 eth1  Out IP 172.30.0.15 > 192.168.62.166: ICMP echo reply, id 433, seq 3, length 64
14:57:22.591784 eth1  In  IP 192.168.62.166 > 172.30.0.15: ICMP echo request, id 433, seq 4, length 64
14:57:22.591811 eth2  Out IP 192.168.62.166 > 172.30.0.15: ICMP echo request, id 433, seq 4, length 64
14:57:22.592002 eth2  In  IP 172.30.0.15 > 192.168.62.166: ICMP echo reply, id 433, seq 4, length 64
14:57:22.592009 eth1  Out IP 172.30.0.15 > 192.168.62.166: ICMP echo reply, id 433, seq 4, length 64
```
```bash
root@red:~# sudo hydra -s 3306 -l toor -P SecLists/Passwords/xato-net-10-million-passwords-100000.txt mysql://172.30.0.15
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-12-24 14:58:36
[INFO] Reduced number of tasks to 4 (mysql does not like many parallel connections)
[DATA] max 4 tasks per 1 server, overall 4 tasks, 100000 login tries (l:1/p:100000), ~25000 tries per task
[DATA] attacking mysql://172.30.0.15:3306/
[3306][mysql] host: 172.30.0.15   login: toor   password: summer
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-12-24 14:58:41
```
```bash
[vagrant@companyrouter ~]$ sudo tcpdump -i any host 172.30.0.15
15:00:11.593281 eth1  Out IP 172.30.0.15.mysql > 192.168.62.166.54544: Flags [P.], seq 95:132, ack 7, win 2037, options [nop,nop,TS val 1035559671 ecr 1569752375], length 37
15:00:11.593388 eth2  In  IP 172.30.0.15.mysql > 192.168.62.166.54576: Flags [P.], seq 1:95, ack 1, win 2037, options [nop,nop,TS val 1035559671 ecr 1569752375], length 94
15:00:11.593391 eth1  Out IP 172.30.0.15.mysql > 192.168.62.166.54576: Flags [P.], seq 1:95, ack 1, win 2037, options [nop,nop,TS val 1035559671 ecr 1569752375], length 94
15:00:11.593489 eth2  In  IP 172.30.0.15.mysql > 192.168.62.166.54544: Flags [R.], seq 132, ack 7, win 2037, options [nop,nop,TS val 1035559671 ecr 1569752375], length 0
15:00:11.593491 eth1  Out IP 172.30.0.15.mysql > 192.168.62.166.54544: Flags [R.], seq 132, ack 7, win 2037, options [nop,nop,TS val 1035559671 ecr 1569752375], length 0
15:00:11.594042 eth1  In  IP 192.168.62.166.54582 > 172.30.0.15.mysql: Flags [S], seq 789403588, win 64240, options [mss 1460,sackOK,TS val 1569752376 ecr 0,nop,wscale 7], length 0
15:00:11.594045 eth2  Out IP 192.168.62.166.54582 > 172.30.0.15.mysql: Flags [S], seq 789403588, win 64240, options [mss 1460,sackOK,TS val 1569752376 ecr 0,nop,wscale 7], length 0
15:00:11.594090 eth1  In  IP 192.168.62.166.54544 > 172.30.0.15.mysql: Flags [R], seq 757774023, win 0, length 0
15:00:11.594091 eth1  In  IP 192.168.62.166.54576 > 172.30.0.15.mysql: Flags [.], ack 95, win 502, options [nop,nop,TS val 1569752376 ecr 1035559671], length 0
15:00:11.594092 eth2  Out IP 192.168.62.166.54544 > 172.30.0.15.mysql: Flags [R], seq 757774023, win 0, length 0
15:00:11.594098 eth2  Out IP 192.168.62.166.54576 > 172.30.0.15.mysql: Flags [.], ack 95, win 502, options [nop,nop,TS val 1569752376 ecr 1035559671], length 0
```

- Install and configure the Suricata software. Keep it simple and stick to the default configuration file(s) as much as possible. Change the interface to the one you want to sniff on in the correct Suricata configuration file. Focus on 1 interface when starting out!

```bash
sudo dnf install -yq epel-release yum-plugin-copr
sudo dnf copr enable @oisf/suricata-6.0
sudo dnf install -yq suricata
sudo suricata-update
sudo suricata -T -c /etc/suricata/suricata.yaml -v
sudo chown -R suricata:suricata /var/log/suricata/
sudo chmod -R 755 /var/log/suricata/
```

BELANGRIJK!!!
Pas de volgende file aan naar het juiste interface (eth1):

```bash
[vagrant@companyrouter rules]$ sudo nano /etc/sysconfig/suricata
```
```bash
# The following parameters are the most commonly needed to configure
# suricata. A full list can be seen by running /sbin/suricata --help
# -i <network interface device>
# --user <acct name>
# --group <group name>

# Add options to be passed to the daemon
OPTIONS="-i eth1 --user suricata "
```

```bash
[vagrant@companyrouter ~]$ sudo systemctl start suricata
[vagrant@companyrouter ~]$ sudo systemctl status suricata
● suricata.service - Suricata Intrusion Detection Service
     Loaded: loaded (/usr/lib/systemd/system/suricata.service; disabled; preset: disabled)
     Active: active (running) since Tue 2024-12-24 15:08:26 UTC; 4s ago
       Docs: man:suricata(1)
    Process: 1904 ExecStartPre=/bin/rm -f /var/run/suricata.pid (code=exited, status=0/SUCCESS)
   Main PID: 1905 (Suricata-Main)
      Tasks: 1 (limit: 24482)
     Memory: 138.2M
        CPU: 4.488s
     CGroup: /system.slice/suricata.service
             └─1905 /sbin/suricata -c /etc/suricata/suricata.yaml --pidfile /var/run/suricata.pid -i eth1 --user surica>

Dec 24 15:08:26 companyrouter systemd[1]: Starting Suricata Intrusion Detection Service...
Dec 24 15:08:26 companyrouter systemd[1]: Started Suricata Intrusion Detection Service.
```

Test:
```bash
[vagrant@companyrouter rules]$ curl http://testmynids.org/uid/index.html
uid=0(root) gid=0(root) groups=0(root)
```

Nu zou je dit moeten zien verschijnen in de fast.log file:
```bash
12/26/2024-10:59:39.156754  [**] [1:2100498:7] GPL ATTACK_RESPONSE id check returned root [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 18.239.208.28:80 -> 192.168.62.253:53050
```

Create your own alert rules:

- What is the difference between the fast.log and the eve.json files?  

fast.log:  
 gaat beknopt de informatie weergeven en dient om snel doorzocht te worden met eenvoudige output. De log bevat de timestamp, event ID, classificatie, beschrijving van de signatuur die het event heeft geactiveerd, en de bron- en bestemmings-IP's en poorten.  
 
 eve.json: is een uitgebreid logbestand in JSON-formaat dat gedetailleerde informatie biedt over events. Het is een rijke, gestructureerde en gedetailleerde output biedt die beter geschikt is voor geautomatiseerde verwerking en integratie met geavanceerde monitoring- en analysehulpmiddelen.  


- Create a rule that alerts as soon as a ping is performed between two machines (for example red and database)

Maak een nieuwe file aan (local.rules):
```bash
[vagrant@companyrouter rules]$  cd  /var/lib/suricata/rules/
[vagrant@companyrouter rules]$ sudo nano local.rules
```
Met de volgende config:
```bash
alert icmp any any -> any any (msg:"ICMP Echo Request Detected"; itype:8; sid:1000001;)
alert icmp any any -> any any (msg:"ICMP Echo Reply Detected"; itype:0; sid:1000002;)
```

Test deze rule door te pingen van 2 machines naar emlaar (database naar RED) en bekijk de logs in de fast.log file:

```bash
[vagrant@companyrouter ~]$ sudo tail -f /var/log/suricata/fast.log
12/26/2024-10:59:30.404532  [**] [1:1000002:0] ICMP Echo Reply Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 192.168.62.166:0 -> 172.30.0.15:0
12/26/2024-10:59:31.404317  [**] [1:1000001:0] ICMP Echo Request Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 172.30.0.15:8 -> 192.168.62.166:0
12/26/2024-10:59:31.404529  [**] [1:1000002:0] ICMP Echo Reply Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 192.168.62.166:0 -> 172.30.0.15:0
12/26/2024-10:59:39.156754  [**] [1:2100498:7] GPL ATTACK_RESPONSE id check returned root [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 18.239.208.28:80 -> 192.168.62.253:53050
12/26/2024-11:07:33.937492  [**] [1:1000001:0] ICMP Echo Request Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 172.30.0.15:8 -> 192.168.62.166:0
12/26/2024-11:07:33.937691  [**] [1:1000002:0] ICMP Echo Reply Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 192.168.62.166:0 -> 172.30.0.15:0
12/26/2024-11:07:34.937508  [**] [1:1000001:0] ICMP Echo Request Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 172.30.0.15:8 -> 192.168.62.166:0
12/26/2024-11:07:34.937664  [**] [1:1000002:0] ICMP Echo Reply Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 192.168.62.166:0 -> 172.30.0.15:0
12/26/2024-11:07:35.937513  [**] [1:1000001:0] ICMP Echo Request Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 172.30.0.15:8 -> 192.168.62.166:0
12/26/2024-11:07:35.937707  [**] [1:1000002:0] ICMP Echo Reply Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 192.168.62.166:0 -> 172.30.0.15:0
12/26/2024-11:07:36.937187  [**] [1:1000001:0] ICMP Echo Request Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 172.30.0.15:8 -> 192.168.62.166:0
12/26/2024-11:07:36.937349  [**] [1:1000002:0] ICMP Echo Reply Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 192.168.62.166:0 -> 172.30.0.15:0
12/26/2024-11:07:37.936828  [**] [1:1000001:0] ICMP Echo Request Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 172.30.0.15:8 -> 192.168.62.166:0
12/26/2024-11:07:37.937018  [**] [1:1000002:0] ICMP Echo Reply Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 192.168.62.166:0 -> 172.30.0.15:0
12/26/2024-11:07:38.936518  [**] [1:1000001:0] ICMP Echo Request Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 172.30.0.15:8 -> 192.168.62.166:0
12/26/2024-11:07:38.936714  [**] [1:1000002:0] ICMP Echo Reply Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 192.168.62.166:0 -> 172.30.0.15:0
12/26/2024-11:07:39.936401  [**] [1:1000001:0] ICMP Echo Request Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 172.30.0.15:8 -> 192.168.62.166:0
12/26/2024-11:07:39.936618  [**] [1:1000002:0] ICMP Echo Reply Detected [**] [Classification: (null)] [Priority: 3] {ICMP} 192.168.62.166:0 -> 172.30.0.15:0
^C
```

- Test your out-of-the-box configuration and browse on your red machine to http://www.cybersec.internal/cmd and enter "id" as an evil command. Does it trigger an alert? If not, are you able to make it trigger an alert?

Ja het triggered een alert:
```bash
12/26/2024-12:17:08.493902  [**] [1:2019284:3] ET ATTACK_RESPONSE Output of id command from HTTP server [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 172.30.0.10:80 -> 192.168.62.165:41066
12/26/2024-12:17:08.493902  [**] [1:2100498:7] GPL ATTACK_RESPONSE id check returned root [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 172.30.0.10:80 -> 192.168.62.165:41066
12/26/2024-12:17:46.439017  [**] [1:2019284:3] ET ATTACK_RESPONSE Output of id command from HTTP server [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 172.30.0.10:80 -> 192.168.62.165:35328
12/26/2024-12:17:46.439017  [**] [1:2100498:7] GPL ATTACK_RESPONSE id check returned root [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 172.30.0.10:80 -> 192.168.62.165:35328
12/26/2024-12:17:51.440346  [**] [1:2019284:3] ET ATTACK_RESPONSE Output of id command from HTTP server [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 172.30.0.10:80 -> 192.168.62.165:35328
12/26/2024-12:17:51.440346  [**] [1:2100498:7] GPL ATTACK_RESPONSE id check returned root [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 172.30.0.10:80 -> 192.168.62.165:35328
```

- Create an alert that checks the mysql tcp port and rerun a hydra attack to check this rule. Can you visually see this bruteforce attack in the fast.log file? Tip: monitor the file live with an option of tail.
```bash
12/26/2024-12:19:42.857246  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.62.166:46366 -> 172.30.0.15:3306
12/26/2024-12:19:42.857250  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.62.166:46362 -> 172.30.0.15:3306
12/26/2024-12:19:42.857356  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.62.166:46376 -> 172.30.0.15:3306
12/26/2024-12:19:42.858940  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.62.166:46388 -> 172.30.0.15:3306
12/26/2024-12:19:42.864840  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.62.166:46400 -> 172.30.0.15:3306
12/26/2024-12:19:42.876012  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:46454
12/26/2024-12:19:42.909629  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:46532
12/26/2024-12:19:42.940293  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:46660
12/26/2024-12:19:42.962017  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:46726
12/26/2024-12:19:42.983775  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:46802
12/26/2024-12:19:43.017353  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:46882
12/26/2024-12:19:43.055272  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:47018
12/26/2024-12:19:43.079532  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:47070
12/26/2024-12:19:43.104267  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:47136
12/26/2024-12:19:43.137415  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:47192
12/26/2024-12:19:43.170815  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:47330
12/26/2024-12:19:43.194407  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:47424
12/26/2024-12:19:43.214830  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:47506
12/26/2024-12:19:43.246093  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:47598
12/26/2024-12:19:43.275409  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:47706
12/26/2024-12:19:43.298077  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:47804
12/26/2024-12:19:43.318840  [**] [1:2010494:5] ET SCAN Multiple MySQL Login Failures Possible Brute Force Attempt [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 172.30.0.15:3306 -> 192.168.62.166:47876
```

- Go have a look at the Suricata documentation. What is the default configuration of Suricata, is it an IPS or IDS?

IDS: Suricata onderneemt zelf geen acties. Het detecteert enkel gevaren.

- What do you have to change to the setup to switch: to the other (IPS or IDS)? You are free to experiment more and go all out with variables (for the networks) and rules. Make sure you can conceptually explain why certain rules would be useful and where (= from which subnet to which subnet) they should be applied?

In /etc/suricata/suricata.yaml “copy-mode” op yes zetten
Modify Ruleset for Active Blocking: Use Suricata’s drop action in rules to block malicious traffic instead of just alerting.
  
Deze kunnen handig zijn om bv. een mysql aanval of een aanval op de webserver  niet alleen te loggen maar ook te verwerpen, deze rules zouden van het externe netwerk naar het internet netwerk geconfigureerd


- To illustrate the difference between an IPS and firewall, enable the firewall and redo the hydra attack through an SSH tunnel. Can you make sure that Suricata detects this attack as an IPS? Do you understand why Suricata can offer this protection whilst a firewall cannot? What is the difference between an IPS and firewall? On which layers of the OSI-model do they work?

Ja suricata kan dit gaan verwerpen als IPS
  
Een firewall controleert voornamelijk IP-adressen, TCP/UDP-poorten en andere kopinformatie terwijl een IPS diepgaande pakketinspectie (Deep Packet Inspection, DPI) uit, waarbij het de inhoud van het netwerkverkeer analyseert, niet alleen de koppen (headers). Het kan patronen herkennen die wijzen op een aanval, zoals bepaalde strings of verkeersgedragingen in de payload van pakketten.  

Firewall: laag 3,4
IPS: laag 3,4,5,7

- An IPS/IDS needs to be tuned thoroughly to prevent alerting fatigue, so it is good that you learn how to write your own rules. Fortunately, there are also rule sets out there that can help you. Read the rule management documentation for Suricata and import the Emerging Threats Open ruleset. Find a way to demonstrate that these additional rules work.

Installeer de Emerging rules:
```bash
[vagrant@companyrouter rules]$ wget https://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz
```
Voeg ze toe aan de suricata config file:
```bash
-emerging-attack_response.rules  
-emerging-icmp_info.rules       
-emerging-retired.rules      
-emerging-web_server.rules
-emerging-chat.rules             
-emerging-imap.rules            
-emerging-rpc.rules          
-emerging-web_specific_apps.rules
-emerging-coinminer.rules        
-emerging-inappropriate.rules   
-emerging-scada.rules        
-emerging-worm.rules
-emerging-current_events.rules   
-emerging-info.rules            
-emerging-scan.rules      
-emerging-deleted.rules          
-emerging-ja3.rules             
-emerging-shellcode.rules             
-emerging-dns.rules              
-emerging-malware.rules         
-emerging-smtp.rules      
-emerging-dos.rules              
-emerging-misc.rules            
-emerging-snmp.rules        
-emerging-exploit.rules          
-emerging-mobile_malware.rules  
-emerging-sql.rules          
-emerging-exploit_kit.rules      
-emerging-netbios.rules         
-emerging-telnet.rules                       
-emerging-ftp.rules              
-emerging-p2p.rules             
-emerging-tftp.rules                     
-emerging-games.rules            
-emerging-phishing.rules        
-emerging-user_agents.rules  
-emerging-activex.rules     
-emerging-hunting.rules          
-emerging-policy.rules          
-emerging-voip.rules        
-emerging-adware_pup.rules  
-emerging-icmp.rules             
-emerging-pop3.rules            
-emerging-web_client.rules
```