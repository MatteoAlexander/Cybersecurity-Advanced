# LAB 1: DNS

## Recap Wireshark

- What layers of the OSI model are captured in this capturefile?

Protocollen vanaf layer 2:  

Ethernet: layer 2  
ARP: layer 2  
ipv4: layer 3  
ipv6: layer 3  
ICMP: layer 3  
UDP: layer 4  
TCP: layer 4  
http: layer 7  
LDAP: layer 7  
SSH: layer 6  

![hierarchy](/images/wireshark%20protocols.png)

- Take a look at the conversations. What do you notice?

Veel ssh verkeer, een gesprek tussen een client & server,...

![hierarchy](/images/conversations.png)

- Take a look at the protocol hierarchy. What are the most "interesting" protocols listed here?

De conversaties die niet encrypted zijn: http

- Can you spot an SSH session that got established between 2 machines? List the 2 machines. Who was the SSH server and who was the client? What ports were used? Are these ports TCP or UDP?

client: 172.30.128.10:37700 (kali)
server: 172.30.42.2:22 (webserver)
  
![hierarchy](/images/ssh%20sessie%20wireshark.png)

- Some cleartext data was transferred between two machines. Can you spot the data? Can you deduce what happened here?

Er is een reverse shell waarbij de client (doelapparaat) surft naar een onveilige website

![hierarchy](/images/reverse%20shell.png)

- Someone used a specific way to transfer a png on the wire. Is it possible to export this png easily? Is it possible to export other HTTP related stuff?

JA: File -> export objects http -> save images  

images:  
![hierarchy](/images/http%20image1)  
![hierarchy](/images/http%20image2)  


## Capture traffic using the CLI

### Start at least the isprouter, the companyrouter, the dns and the employee in your environment.

- Install the tcpdump utility on the companyrouter and figure out a way to sniff traffic origination from the employee using tcpdump on the companyrouter.
  
```bash
[vagrant@companyrouter ~]$ sudo dnf install -y tcpdump
```

- Have a look at the ip configurations of the dns machine, the employee client and the companyrouter.

| vm | interface | ip | default gateway | dns |
| :- | :- | :- | :- | :- |
| companyrouter | eth1 | 192.168.62.253 |  192.168.62.254 |  172.30.0.4 |
| companyrouter | eth2| 172.30.255.254 | |  172.30.0.4 |
| dns | eth1 | 172.30.0.4 | 172.30.255.254 | 192.168.62.254 |
| employee | eth1 | 172.30.0.123 | 172.30.255.254 |  172.30.0.4 |

- Which interface on the companyrouter will you use to capture traffic from the dns to the internet?

Beide interfaces zijn mogelijk, maar aangezien eth2 in hetzelfde netwerk zit als dns kiezen we voor eth2. Anders zal de ssh traffic nog gefilterd moeten worden.

```bash
[vagrant@companyrouter ~]$ ip -4 a | grep inet
    inet 127.0.0.1/8 scope host lo
    inet 192.168.62.253/24 brd 192.168.62.255 scope global noprefixroute eth1
    inet 172.30.255.254/16 brd 172.30.255.255 scope global noprefixroute eth2
[vagrant@companyrouter ~]$ sudo tcpdump -D | grep -i running
1.eth1 [Up, Running, Connected]
2.eth2 [Up, Running, Connected]
3.any (Pseudo-device that captures on all interfaces) [Up, Running]
4.lo [Up, Running, Loopback]
```

- Which interface on the companyrouter would you use to capture traffic from dns to employee?

Hier zal ook gekozen worden voor eth2, aangezien deze verbonden is met het interne netwerk "company". Dit is ook het netwerk dat gebruikt wordt voor de dns, employee.

- Test this out by pinging from employee to the companyrouter and from employee to the dns. Are you able to see all pings in tcpdump on the companyrouter?

Enkel de pings naar companyrouter en internet zijn zichtbaar.

```bash
[vagrant@companyrouter ~]$ sudo tcpdump -i eth2
```
```bash
19:34:47.943189 IP 172.30.0.123 > companyrouter: ICMP echo request, id 3, seq 0, length 64
19:34:47.943204 IP companyrouter > 172.30.0.123: ICMP echo reply, id 3, seq 0, length 64
19:34:47.943276 IP 172.30.0.123.ssh > companyrouter.32988: Flags [P.], seq 3237:3369, ack 1552, win 2003, options [nop,nop,TS val 2028566523 ecr 1179489480], length 132
19:34:47.943301 IP companyrouter.32988 > 172.30.0.123.ssh: Flags [.], ack 3369, win 501, options [nop,nop,TS val 1179489481 ecr 2028566523], length 0
19:34:47.943363 IP 172.30.0.123.ssh > companyrouter.32988: Flags [P.], seq 3369:3469, ack 1552, win 2003, options [nop,nop,TS val 2028566523 ecr 1179489480], length 100
19:34:47.943366 IP companyrouter.32988 > 172.30.0.123.ssh: Flags [.], ack 3469, win 501, options [nop,nop,TS val 1179489481 ecr 2028566523], length 0
19:34:48.943016 IP 172.30.0.123 > companyrouter: ICMP echo request, id 3, seq 1, length 64
19:34:48.943057 IP companyrouter > 172.30.0.123: ICMP echo reply, id 3, seq 1, length 64
19:34:48.943312 IP 172.30.0.123.ssh > companyrouter.32988: Flags [P.], seq 3469:3569, ack 1552, win 2003, options [nop,nop,TS val 2028567523 ecr 1179489481]
```

- Figure out a way to capture the data in a file. Copy this file from the companyrouter to your host and verify you can analyze this file with wireshark (on your host).

```bash
[vagrant@companyrouter ~]$ sudo tcpdump -i eth2 -w /tmp/lab01.pcap -v
dropped privs to tcpdump
tcpdump: listening on eth2, link-type EN10MB (Ethernet), snapshot length 262144 bytes
^C145 packets captured
145 packets received by filter
0 packets dropped by kernel
```

scp lukt momenteel niet: scp /tmp/lab01.pcap matte@192.168.1.27:C:/Users/matte/Downloads

- SSH from employee to the companyrouter. When scanning with tcpdump you will now see a lot of SSH traffic passing by. How can you start tcpdump and filter out this ssh traffic?

Zonder filter:

```bash
[vagrant@companyrouter tmp]$ sudo tcpdump -i eth2
13:58:01.785648 IP companyrouter.32820 > 172.30.0.123.ssh: Flags [.], ack 241, win 501, options [nop,nop,TS val 1436788849 ecr 283650060], length 0
13:58:01.826978 IP 172.30.0.123.49122 > companyrouter.ssh: Flags [.], ack 1570, win 2003, options [nop,nop,TS val 1763325315 ecr 4228225305], length 0
13:58:02.619785 IP companyrouter.32820 > 172.30.0.123.ssh: Flags [P.], seq 88:124, ack 241, win 501, options [nop,nop,TS val 1436789683 ecr 283650060], length 36
13:58:02.620146 IP 172.30.0.123.ssh > companyrouter.32820: Flags [P.], seq 241:277, ack 124, win 2003, options [nop,nop,TS val 283650895 ecr 1436789683], length 36
13:58:02.620175 IP companyrouter.32820 > 172.30.0.123.ssh: Flags [.], ack 277, win 501, options [nop,nop,TS val 1436789683 ecr 283650895], length 0
13:58:02.807439 IP companyrouter.32820 > 172.30.0.123.ssh: Flags [P.], seq 124:160, ack 277, win 501, options [nop,nop,TS val 1436789870 ecr 283650895], length 36
13:58:02.807710 IP 172.30.0.123.ssh > companyrouter.32820: Flags [P.], seq 277:313, ack 160, win 2003, options [nop,nop,TS val 283651083 ecr 1436789870], length 36
13:58:02.807721 IP companyrouter.32820 > 172.30.0.123.ssh: Flags [.], ack 313, win 501, options [nop,nop,TS val 1436789871 ecr 283651083], length 0
13:58:02.922778 IP companyrouter.32820 > 172.30.0.123.ssh: Flags [P.], seq 160:196, ack 313, win 501, options [nop,nop,TS val 1436789986 ecr 283651083], length 36
13:58:02.923074 IP 172.30.0.123.ssh > companyrouter.32820: Flags [P.], seq 313:349, ack 196, win 2003, options [nop,nop,TS val 283651198 ecr 1436789986], length 36
13:58:02.923084 IP companyrouter.32820 > 172.30.0.123.ssh: Flags [.], ack 349, win 501, options [nop,nop,TS val 1436789986 ecr 283651198], length 0
13:58:03.074625 IP companyrouter.32820 > 172.30.0.123.ssh: Flags [P.], seq 196:232, ack 349, win 501, options [nop,nop,TS val 1436790138 ecr 283651198], length 36
```

met filter:

```bash
[vagrant@companyrouter tmp]$ sudo tcpdump -i eth2 port not ssh
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth2, link-type EN10MB (Ethernet), snapshot length 262144 bytes
14:00:17.512368 IP 172.30.0.10.60536 > ip-193-121-135-196.dsl.scarlet.be.ntp: NTPv4, Client, length 48
14:00:17.528197 IP ip-193-121-135-196.dsl.scarlet.be.ntp > 172.30.0.10.60536: NTPv4, Server, length 48
14:00:17.566408 IP companyrouter.38518 > 172.30.0.4.domain: 19458+ PTR? 196.135.121.193.in-addr.arpa. (46)
14:00:17.566821 IP 172.30.0.4.domain > companyrouter.38518: 19458 1/0/0 PTR ip-193-121-135-196.dsl.scarlet.be. (93)
14:00:17.566915 IP companyrouter.39965 > 172.30.0.4.domain: 52439+ PTR? 10.0.30.172.in-addr.arpa. (42)
14:00:17.567147 IP 172.30.0.4.domain > companyrouter.39965: 52439 NXDomain* 0/1/0 (96)
14:00:17.670409 IP companyrouter.44309 > 172.30.0.4.domain: 30592+ PTR? 4.0.30.172.in-addr.arpa. (41)
14:00:17.670784 IP 172.30.0.4.domain > companyrouter.44309: 30592 NXDomain* 0/1/0 (95)
14:00:17.670947 IP companyrouter.33311 > 172.30.0.4.domain: 25188+ PTR? 254.255.30.172.in-addr.arpa. (45)
14:00:17.671182 IP 172.30.0.4.domain > companyrouter.33311: 25188 NXDomain* 0/1/0 (99)
14:00:18.504340 IP 172.30.0.10.42294 > ntp.ulyssis.student.kuleuven.be.ntp: NTPv4, Client, length 48
14:00:18.515616 IP ntp.ulyssis.student.kuleuven.be.ntp > 172.30.0.10.42294: NTPv4, Server, length 48
14:00:18.606280 IP companyrouter.41558 > 172.30.0.4.domain: 57158+ PTR? 214.253.190.193.in-addr.arpa. (46)
14:00:18.608764 IP 172.30.0.4.48373 > _gateway.domain: 16976+% [1au] PTR? 214.253.190.193.in-addr.arpa. (69)
14:00:18.637290 IP _gateway.domain > 172.30.0.4.48373: 16976 2/0/1 CNAME 214.192.253.190.193.in-addr.arpa., PTR ntp.ulyssis.student.kuleuven.be. (124)
14:00:18.637820 IP 172.30.0.4.56536 > _gateway.domain: 43745+% [1au] DS? 190.193.in-addr.arpa. (61)
14:00:18.658080 IP _gateway.domain > 172.30.0.4.56536: 43745 0/1/1 (109)
14:00:18.658876 IP 172.30.0.4.37063 > pri.authdns.ripe.net.domain: 35385 [1au] DS? 190.193.in-addr.arpa. (61)
14:00:18.672075 IP pri.authdns.ripe.net.domain > 172.30.0.4.37063: 35385*- 0/4/1 (375)
14:00:18.673309 IP 172.30.0.4.55198 > _gateway.domain: 38246+% [1au] PTR? 214.192.253.190.193.in-addr.arpa. (73)
14:00:18.673540 IP _gateway.domain > 172.30.0.4.55198: 38246 1/0/1 PTR ntp.ulyssis.student.kuleuven.be. (106)
14:00:18.673837 IP 172.30.0.4.domain > companyrouter.41558: 57158 2/0/0 CNAME 214.192.253.190.193.in-addr.arpa., PTR ntp.ulyssis.student.kuleuven.be. (137)
14:00:18.710716 IP companyrouter.42371 > 172.30.0.4.domain: 4034+ PTR? 254.62.168.192.in-addr.arpa. (45)
14:00:18.711148 IP 172.30.0.4.domain > companyrouter.42371: 4034 NXDomain* 0/1/0 (100)
14:00:18.712100 IP companyrouter.50379 > 172.30.0.4.domain: 32912+ PTR? 5.9.0.193.in-addr.arpa. (40)
14:00:18.712375 IP 172.30.0.4.domain > companyrouter.50379: 32912 1/0/0 PTR pri.authdns.ripe.net. (74)
14:00:20.315054 IP 172.30.0.10.36654 > webserver.discosmash.com.ntp: NTPv4, Client, length 48
14:00:20.354641 IP webserver.discosmash.com.ntp > 172.30.0.10.36654: NTPv4, Server, length 48
14:00:20.374311 IP companyrouter.36935 > 172.30.0.4.domain: 23809+ PTR? 219.227.82.81.in-addr.arpa. (44)
14:00:20.374957 IP 172.30.0.4.45916 > _gateway.domain: 1257+% [1au] PTR? 219.227.82.81.in-addr.arpa. (67)
14:00:20.388724 IP _gateway.domain > 172.30.0.4.45916: 1257 1/0/1 PTR webserver.discosmash.com. (93)
14:00:20.389126 IP 172.30.0.4.domain > companyrouter.36935: 23809 1/0/0 PTR webserver.discosmash.com. (82)
14:00:22.528385 ARP, Request who-has companyrouter tell 172.30.0.10, length 46
14:00:22.528394 ARP, Reply companyrouter is-at 08:00:27:a7:5d:df (oui Unknown), length 28
14:00:22.576363 ARP, Request who-has companyrouter tell 172.30.0.4, length 46
14:00:22.576376 ARP, Reply companyrouter is-at 08:00:27:a7:5d:df (oui Unknown), length 28
14:00:22.589987 ARP, Request who-has 172.30.0.10 tell companyrouter, length 28
14:00:22.590167 ARP, Reply 172.30.0.10 is-at 08:00:27:79:c3:ed (oui Unknown), length 46
14:00:23.048777 IP 172.30.0.123.52286 > ntp.rack66.net.ntp: NTPv4, Client, length 48
14:00:23.057706 IP ntp.rack66.net.ntp > 172.30.0.123.52286: NTPv4, Server, length 48
14:00:23.078361 IP companyrouter.45900 > 172.30.0.4.domain: 2887+ PTR? 5.20.89.185.in-addr.arpa. (42)
14:00:23.078761 IP 172.30.0.4.domain > companyrouter.45900: 2887 1/0/0 PTR ntp.rack66.net. (70)
14:00:23.078878 IP companyrouter.41260 > 172.30.0.4.domain: 22022+ PTR? 123.0.30.172.in-addr.arpa. (43)
14:00:23.079182 IP 172.30.0.4.domain > companyrouter.41260: 22022 NXDomain* 0/1/0 (97)
14:00:25.057578 IP 172.30.0.123.45541 > ntp.devrandom.be.ntp: NTPv4, Client, length 48
14:00:25.066228 IP ntp.devrandom.be.ntp > 172.30.0.123.45541: NTPv4, Server, length 48
14:00:25.158104 IP companyrouter.38895 > 172.30.0.4.domain: 47327+ PTR? 3.76.87.45.in-addr.arpa. (41)
14:00:25.158663 IP 172.30.0.4.57661 > _gateway.domain: 60229+% [1au] PTR? 3.76.87.45.in-addr.arpa. (64)
14:00:25.171694 IP _gateway.domain > 172.30.0.4.57661: 60229 1/0/1 PTR ntp.devrandom.be. (82)
14:00:25.172245 IP 172.30.0.4.39712 > _gateway.domain: 39927+% [1au] DS? 45.in-addr.arpa. (56)
14:00:25.186598 IP _gateway.domain > 172.30.0.4.39712: 39927 2/0/1 DS, RRSIG (264)
14:00:25.187053 IP 172.30.0.4.55630 > _gateway.domain: 55171+% [1au] DNSKEY? in-addr.arpa. (53)
14:00:25.276615 IP _gateway.domain > 172.30.0.4.55630: 55171| 0/0/1 (41)
14:00:25.276969 IP 172.30.0.4.45237 > _gateway.domain: Flags [S], seq 765832025, win 64660, options [mss 1220,sackOK,TS val 655595594 ecr 0,nop,wscale 5], length 0
14:00:25.277141 IP _gateway.domain > 172.30.0.4.45237: Flags [S.], seq 1576935977, ack 765832026, win 65160, options [mss 1460,sackOK,TS val 2459449847 ecr 655595594,nop,wscale 5], length 0
14:00:25.277272 IP 172.30.0.4.45237 > _gateway.domain: Flags [.], ack 1, win 2021, options [nop,nop,TS val 655595594 ecr 2459449847], length 0
14:00:25.277385 IP 172.30.0.4.45237 > _gateway.domain: Flags [P.], seq 1:56, ack 1, win 2021, options [nop,nop,TS val 655595594 ecr 2459449847], length 55 47919+% [1au] DNSKEY? in-addr.arpa. (53)
14:00:25.277498 IP _gateway.domain > 172.30.0.4.45237: Flags [.], ack 56, win 2035, options [nop,nop,TS val 2459449848 ecr 655595594], length 0
14:00:25.277612 IP _gateway.domain > 172.30.0.4.45237: Flags [P.], seq 1:1344, ack 56, win 2035, options [nop,nop,TS val 2459449848 ecr 655595594], length 1343 47919 5/0/1 DNSKEY, DNSKEY, DNSKEY, RRSIG, RRSIG (1341)
14:00:25.277723 IP 172.30.0.4.45237 > _gateway.domain: Flags [.], ack 1344, win 1998, options [nop,nop,TS val 655595595 ecr 2459449848], length 0
14:00:25.277901 IP 172.30.0.4.45237 > _gateway.domain: Flags [F.], seq 56, ack 1344, win 2011, options [nop,nop,TS val 655595595 ecr 2459449848], length 0
14:00:25.278044 IP _gateway.domain > 172.30.0.4.45237: Flags [F.], seq 1344, ack 57, win 2035, options [nop,nop,TS val 2459449848 ecr 655595595], length 0
14:00:25.278149 IP 172.30.0.4.45237 > _gateway.domain: Flags [.], ack 1345, win 2011, options [nop,nop,TS val 655595595 ecr 2459449848], length 0
14:00:25.278587 IP 172.30.0.4.45655 > _gateway.domain: 43013+% [1au] DS? 87.45.in-addr.arpa. (59)
14:00:25.292918 IP _gateway.domain > 172.30.0.4.45655: 43013 0/4/1 (499)
14:00:25.293377 IP 172.30.0.4.48964 > _gateway.domain: 44222+% [1au] DNSKEY? 45.in-addr.arpa. (56)
14:00:25.407911 IP _gateway.domain > 172.30.0.4.48964: 44222| 0/0/1 (44)
14:00:25.408275 IP 172.30.0.4.36277 > _gateway.domain: Flags [S], seq 1048044439, win 64660, options [mss 1220,sackOK,TS val 655595725 ecr 0,nop,wscale 5], length 0
14:00:25.408430 IP _gateway.domain > 172.30.0.4.36277: Flags [S.], seq 3329613547, ack 1048044440, win 65160, options [mss 1460,sackOK,TS val 2459449979 ecr 655595725,nop,wscale 5], length 0
14:00:25.408554 IP 172.30.0.4.36277 > _gateway.domain: Flags [.], ack 1, win 2021, options [nop,nop,TS val 655595726 ecr 2459449979], length 0
14:00:25.408665 IP 172.30.0.4.36277 > _gateway.domain: Flags [P.], seq 1:59, ack 1, win 2021, options [nop,nop,TS val 655595726 ecr 2459449979], length 58 47941+% [1au] DNSKEY? 45.in-addr.arpa. (56)
14:00:25.408784 IP _gateway.domain > 172.30.0.4.36277: Flags [.], ack 59, win 2035, options [nop,nop,TS val 2459449979 ecr 655595726], length 0
14:00:25.408871 IP _gateway.domain > 172.30.0.4.36277: Flags [P.], seq 1:1373, ack 59, win 2035, options [nop,nop,TS val 2459449979 ecr 655595726], length 1372 47941 6/0/1 DNSKEY, DNSKEY, DNSKEY, DNSKEY, RRSIG, RRSIG (1370)
14:00:25.408982 IP 172.30.0.4.36277 > _gateway.domain: Flags [.], ack 1373, win 1998, options [nop,nop,TS val 655595726 ecr 2459449979], length 0
14:00:25.409119 IP 172.30.0.4.36277 > _gateway.domain: Flags [F.], seq 59, ack 1373, win 2011, options [nop,nop,TS val 655595726 ecr 2459449979], length 0
14:00:25.409284 IP _gateway.domain > 172.30.0.4.36277: Flags [F.], seq 1373, ack 60, win 2035, options [nop,nop,TS val 2459449980 ecr 655595726], length 0
14:00:25.409407 IP 172.30.0.4.36277 > _gateway.domain: Flags [.], ack 1374, win 2011, options [nop,nop,TS val 655595726 ecr 2459449980], length 0
14:00:25.410006 IP 172.30.0.4.54770 > _gateway.domain: 39087+% [1au] DS? 76.87.45.in-addr.arpa. (62)
14:00:25.428641 IP _gateway.domain > 172.30.0.4.54770: 39087 0/1/1 (104)
14:00:25.429345 IP 172.30.0.4.33284 > f.in-addr-servers.arpa.domain: 52921 [1au] DS? 76.87.45.in-addr.arpa. (62)
14:00:25.442241 IP f.in-addr-servers.arpa.domain > 172.30.0.4.33284: 52921- 0/8/1 (390)
14:00:25.442696 IP 172.30.0.4.33315 > u.arin.net.domain: 50192 [1au] DS? 76.87.45.in-addr.arpa. (78)
14:00:25.457964 IP u.arin.net.domain > 172.30.0.4.33315: 50192*- 0/4/1 (540)
14:00:25.460820 IP 172.30.0.4.domain > companyrouter.38895: 47327 1/0/0 PTR ntp.devrandom.be. (71)
14:00:25.470148 IP companyrouter.39250 > 172.30.0.4.domain: 34280+ PTR? 1.9.0.193.in-addr.arpa. (40)
14:00:25.470542 IP 172.30.0.4.domain > companyrouter.39250: 34280 1/0/0 PTR f.in-addr-servers.arpa. (76)
14:00:25.471102 IP companyrouter.34154 > 172.30.0.4.domain: 45872+ PTR? 50.216.61.204.in-addr.arpa. (44)
14:00:25.471377 IP 172.30.0.4.domain > companyrouter.34154: 45872 1/0/0 PTR u.arin.net. (68)
^C
88 packets captured
88 packets received by filter
0 packets dropped by kernel
```

- Start the web VM. Find a way to capture only HTTP traffic and only from and to the webserver-machine. Test this out by browsing to http://www.cybersec.internal from the isprouter machine using curl. This is a website that should be available in the lab environment. Are you able to see this HTTP traffic? Browse on the employee client, are you able to see the same HTTP traffic in tcpdump, why is this the case?

pas de dns server aan van isprouter naar: 172.30.0.4 (/etc/resolv.conf)

```bash
isprouter:~$ curl http://www.cybersec.internal
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome to Example Test Environment</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #f4f4f4;
        }
        .container {
            width: 80%;
            max-width: 600px;
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        p {
            font-size: 1.2em;
        }
        a {
            color: #007bff;
            text-decoration: none;
        }
        a:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>

    <div class="container">
        <h1>Welcome to the cybersec webserver!</h1>
        <p>This is the webpage of the "cybersec" test environment. We are glad to have you here.</p>
        <p>If you want to learn more about our services, visit this link:
            <a href="http://www.cybersec.internal/services">http://www.cybersec.internal/services</a>
        </p>
    </div>

</body>
</html>
```
```bash
[vagrant@companyrouter tmp]$ sudo tcpdump -i eth2 -n "port http and (src host 172.30.0.10 or dst host 172.30.0.10)" -n
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth2, link-type EN10MB (Ethernet), snapshot length 262144 bytes
14:17:43.745389 IP 192.168.62.254.58390 > 172.30.0.10.80: Flags [S], seq 741627673, win 64240, options [mss 1460,sackOK,TS val 3045403897 ecr 0,nop,wscale 5], length 0
14:17:43.745592 IP 172.30.0.10.80 > 192.168.62.254.58390: Flags [S.], seq 38521328, ack 741627674, win 65160, options [mss 1460,sackOK,TS val 746586768 ecr 3045403897,nop,wscale 7], length 0
14:17:43.745740 IP 192.168.62.254.58390 > 172.30.0.10.80: Flags [.], ack 1, win 2008, options [nop,nop,TS val 3045403897 ecr 746586768], length 0
14:17:43.745868 IP 192.168.62.254.58390 > 172.30.0.10.80: Flags [P.], seq 1:85, ack 1, win 2008, options [nop,nop,TS val 3045403898 ecr 746586768], length 84: HTTP: GET / HTTP/1.1
14:17:43.745988 IP 172.30.0.10.80 > 192.168.62.254.58390: Flags [.], ack 85, win 509, options [nop,nop,TS val 746586769 ecr 3045403898], length 0
14:17:43.746297 IP 172.30.0.10.80 > 192.168.62.254.58390: Flags [.], seq 1:1449, ack 85, win 509, options [nop,nop,TS val 746586769 ecr 3045403898], length 1448: HTTP: HTTP/1.1 200 OK
14:17:43.746364 IP 172.30.0.10.80 > 192.168.62.254.58390: Flags [P.], seq 1449:1672, ack 85, win 509, options [nop,nop,TS val 746586769 ecr 3045403898], length 223: HTTP
14:17:43.746386 IP 192.168.62.254.58390 > 172.30.0.10.80: Flags [.], ack 1449, win 2003, options [nop,nop,TS val 3045403898 ecr 746586769], length 0
14:17:43.746462 IP 192.168.62.254.58390 > 172.30.0.10.80: Flags [.], ack 1672, win 1997, options [nop,nop,TS val 3045403898 ecr 746586769], length 0
14:17:43.746811 IP 192.168.62.254.58390 > 172.30.0.10.80: Flags [F.], seq 85, ack 1672, win 2003, options [nop,nop,TS val 3045403899 ecr 746586769], length 0
14:17:43.747053 IP 172.30.0.10.80 > 192.168.62.254.58390: Flags [F.], seq 1672, ack 86, win 509, options [nop,nop,TS val 746586770 ecr 3045403899], length 0
14:17:43.747185 IP 192.168.62.254.58390 > 172.30.0.10.80: Flags [.], ack 1673, win 2003, options [nop,nop,TS val 3045403899 ecr 746586770], length 0
^C
12 packets captured
12 packets received by filter
0 packets dropped by kernel
```

Logisch, want de data gaat niet door de companyrouter.  

```bash
[vagrant@companyrouter tmp]$ sudo tcpdump -i eth1 -n "port http and (src host 172.30.0.10 or dst host 172.30.0.10)" -n
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth1, link-type EN10MB (Ethernet), snapshot length 262144 bytes
^C
0 packets captured
1 packet received by filter
0 packets dropped by kernel
```

## Understanding the network + Attacker machine red

### Part 1

vagrantfile for RED:
```bash
Vagrant.configure("2") do |config|
    config.vm.define "red" do |my_machine|
      config.vm.box = "debian/bookworm64"
      config.vm.hostname = "red"
      config.vm.network "private_network", virtualbox_intnet: "VirtualBox Host-Only Ethernet Adapter #3", type: "static", ip: "192.168.62.166"
      config.vm.provider "virtualbox" do |vb|
        vb.gui = true
        vb.name = "red_vagrant"
        vb.memory = "1024"
        vb.cpus = "1"
        vb.customize ["modifyvm", :id, "--clipboard-mode", "bidirectional"]
      end
    end
  end
```

- Unplug de Nat adapter en voeg de hostonly adapter toe  
- Pas de netwerk interfaces config aan:
```bash 
root@red:~# sudo cat /etc/network/interfaces

# interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
source-directory /etc/network/interfaces.d

# The loopback network interfaciface lo inet loopback

# The primary network interface
allow-hotplug eth0
auto lo
iface lo inet loopback
iface eth0 inet dhcp
dns-nameserver 10.0.2.3
pre-up sleep 2
auto eth1
iface eth1 inet static
        address 192.168.62.166
        netmask 255.255.255.0
        gateway 192.168.62.253
#VAGRANT-END
```

- Voeg de volgende route toe: 
```bash 
sudo ip route add 172.30.0.0/16 via 192.168.100.253
```

### Part 2

- What did you have to configure on your red machine to have internet and to properly ping web?

Een ip adres binnen het host only netwerk en een route toegevoegd op de isprouter.  

- What is the default gateway of each machine?  
  
| vm | interface | default gateway |
| isprouter | eth1 | 10.0.2.2 |
| isprouter | eth1 | |
| companyrouter | eth1 | 192.168.62.254 |
| dns | eth1 | 172.30.255.254 |
| web | eth1 | 172.30.255.254 |
| database | eth1 | 172.30.255.254 |
| employee | eth1 | 172.30.255.254 |
| red | eth1 | 192.168.61.254 |

- What is the DNS server of each machine?  
  
| vm | interface | dns |
| isprouter | eth1 | 10.0.2.3 |
| isprouter | eth1 | 10.0.2.3 |
| companyrouter | eth1 | 10.0.2.3 |
| dns | eth1 | 10.0.2.3 |
| web | eth1 | 172.30.0.4 |
| database | eth1 | 172.30.0.4 |
| employee | eth1 | 172.30.255.254 |
| red | eth1 | 10.0.2.3 |

- Which machines have a static IP and which use DHCP?

Alles statisch behalve de employee

- What (static) routes should be configured and where, how do you make it persistent?

```bash
sudo ip route add 172.30.0.0/16 via 192.168.100.253
```
Deze route moet toegevoegd worden op de isprouter.  

```bash
isprouter:~$ sudo cat /etc/network/interfaces
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
    hostname alpine318.localdomain
#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
auto eth1
iface eth1 inet static
      address 192.168.62.254
      netmask 255.255.255.0
#VAGRANT-END
up ip route add 172.30.0.0/16 via 192.168.62.253
down route del -net 172.30.0.0/16 gw 192.168.62.253
up ip route add 172.10.10.0/24 via 192.168.62.42
```

- What is the purpose (which processes or packages for example are essential) of each machine?  
  
isprouter = data routeren tussen netwerken, en toegang tot internet  
companyrouter =  data routeren tussen netwerken, en firewall regels voor company  
dns = dns server en is centraal punt in de router  
employee = cliënt binnen de company  
web = biedt webservices aan binnen de company  
database = biedt data aan voor de company  
red = hacker, probeert data uit het company netwerk te halen  


- Investigate whether the DNS server of the company network is vulnerable to a DNS zone transfer "attack" as discussed above. What exactly does this attack involve? If possible, try to configure the server to allow & prevent this attack. Document this update: How can you execute this attack or check if the DNS server is vulnerable and how can you fix it? Can you perform this "attack" both on Windows and Linux? Document your findings properly.