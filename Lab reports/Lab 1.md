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

- SSH from employee to the companyrouter. When scanning with tcpdump you will now see a lot of SSH traffic passing by. How can you start tcpdump and filter out this ssh traffic?

- Start the web VM. Find a way to capture only HTTP traffic and only from and to the webserver-machine. Test this out by browsing to http://www.cybersec.internal from the isprouter machine using curl. This is a website that should be available in the lab environment. Are you able to see this HTTP traffic? Browse on the employee client, are you able to see the same HTTP traffic in tcpdump, why is this the case?