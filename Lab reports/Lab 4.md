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
             └─1905 /sbin/suricata -c /etc/suricata/suricata.yaml --pidfile /var/run/suricata.pid -i eth0 --user surica>

Dec 24 15:08:26 companyrouter systemd[1]: Starting Suricata Intrusion Detection Service...
Dec 24 15:08:26 companyrouter systemd[1]: Started Suricata Intrusion Detection Service.
```

Create your own alert rules:

- What is the difference between the fast.log and the eve.json files?  

fast.log:  
 gaat beknopt de informatie weergeven en dient om snel doorzocht te worden met eenvoudige output. De log bevat de timestamp, event ID, classificatie, beschrijving van de signatuur die het event heeft geactiveerd, en de bron- en bestemmings-IP's en poorten.  
 
 eve.json: is een uitgebreid logbestand in JSON-formaat dat gedetailleerde informatie biedt over events. Het is een rijke, gestructureerde en gedetailleerde output biedt die beter geschikt is voor geautomatiseerde verwerking en integratie met geavanceerde monitoring- en analysehulpmiddelen.  


- Create a rule that alerts as soon as a ping is performed between two machines (for example red and database)

- Test your out-of-the-box configuration and browse on your red machine to http://www.cybersec.internal/cmd and enter "id" as an evil command. Does it trigger an alert? If not, are you able to make it trigger an alert?

- Create an alert that checks the mysql tcp port and rerun a hydra attack to check this rule. Can you visually see this bruteforce attack in the fast.log file? Tip: monitor the file live with an option of tail.

- Go have a look at the Suricata documentation. What is the default configuration of Suricata, is it an IPS or IDS?

- What do you have to change to the setup to switch to the other (IPS or IDS)? You are free to experiment more and go all out with variables (for the networks) and rules. Make sure you can conceptually explain why certain rules would be useful and where (= from which subnet to which subnet) they should be applied?

- To illustrate the difference between an IPS and firewall, enable the firewall and redo the hydra attack through an SSH tunnel. Can you make sure that Suricata detects this attack as an IPS? Do you understand why Suricata can offer this protection whilst a firewall cannot? What is the difference between an IPS and firewall? On which layers of the OSI-model do they work?

- An IPS/IDS needs to be tuned thoroughly to prevent alerting fatigue, so it is good that you learn how to write your own rules. Fortunately, there are also rule sets out there that can help you. Read the rule management documentation for Suricata and import the Emerging Threats Open ruleset. Find a way to demonstrate that these additional rules work.