# Lab 9: IPsec

## Set up and verify the network

Make sure (and troubleshoot if necessary) that:

- Routing is enabled on homerouter.
```
[vagrant@homerouter ~]$ cat /proc/sys/net/ipv4/ip_forward
1
```

- remote_employee has internet connection (including DNS).
```
[vagrant@homerouter ~]$ dig 8.8.8.8

; <<>> DiG 9.16.23-RH <<>> 8.8.8.8
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40173
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;8.8.8.8.                       IN      A

;; ANSWER SECTION:
8.8.8.8.                0       IN      A       8.8.8.8

;; Query time: 5 msec
;; SERVER: 192.168.62.254#53(192.168.62.254)
;; WHEN: Sun Dec 29 13:47:06 UTC 2024
;; MSG SIZE  rcvd: 52

[vagrant@homerouter ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=14.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=14.0 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=14.4 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=114 time=13.8 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 13.832/14.124/14.412/0.226 ms
```

- remote_employee can reach all other VM's (you may disable the firewall on companyrouter.).

YES!!!

- Important (!) add routes in such a way that traffic from the home network from and to the company network directly goes between the two routers (company and home) and not via the isprouter.

Pas de default gateway aan van de homerouter hiervoor:
```
 sudo ip route add default via 192.168.62.253
```

Controleren:
```
[vagrant@homerouter ~]$ traceroute 172.30.255.254
traceroute to 172.30.255.254 (172.30.255.254), 30 hops max, 60 byte packets
 1  172.30.255.254 (172.30.255.254)  1.096 ms !X  0.981 ms !X  0.968 ms !X
```

## MitM attack


Start ARP spoofing:

```
┌──(osboxes㉿osboxes)-[~]
└─$ sudo ettercap -Tq -i eth0 -M arp:remote /192.168.62.42// /192.168.62.253//
[sudo] password for osboxes: 

ettercap 0.8.3.1 copyright 2001-2020 Ettercap Development Team

Listening on:
  eth0 -> 08:00:27:CB:C1:E1
          192.168.62.165/255.255.255.0
          fe80::ef29:130a:fc9f:c25c/64

SSL dissection needs a valid 'redir_command_on' script in the etter.conf file
Privileges dropped to EUID 65534 EGID 65534...

  34 plugins
  42 protocol dissectors
  57 ports monitored
28230 mac vendor fingerprint
1766 tcp OS fingerprint
2182 known services
Lua: no scripts were specified, not starting up!

Scanning for merged targets (2 hosts)...

* |==================================================>| 100.00 %
                                                                                                                                                                                                                                           
2 hosts added to the hosts list...                                                                                                                                                                                                         
                                                                                                                                                                                                                                           
ARP poisoning victims:                                                                                                                                                                                                                     
                                                                                                                                                                                                                                           
 GROUP 1 : 192.168.62.42 08:00:27:47:31:F3                                                                                                                                                                                                 
                                                                                                                                                                                                                                           
 GROUP 2 : 192.168.62.253 08:00:27:2A:96:62                                                                                                                                                                                                
Starting Unified sniffing...                                                                                                                                                                                                               
                                                                                                                                                                                                                                           
                                                                                                                                                                                                                                           
Text only Interface activated...                                                                                                                                                                                                           
Hit 'h' for inline help
```


ARP spoofing in wireshark:

![hierarchy](/images/arpspoofing.png)

- <interface>: the interface you use on red.

eth0

- <leftIP>: the one side of the connection you try to capture, in casu homerouter.

Homerouter

- <rightIP>: the other side of the connection you try to capture, in casu companyrouter.

Companyrouter

- take a look at the man page of ettercap why there is one slash (/) in front and 2 slashes (//) after the IPs?

Eén slash (/) voor de IP-adressen: Dit geeft aan dat het hier om een 'regular expression' gaat die het IP-adres moet matchen. In ettercap wordt dit gebruikt om aan te geven dat je een patroon zoekt in het netwerk. Het gebruik van slashes om een patroon te omgeven is een veelvoorkomende syntaxis in programmeertalen en tools die met reguliere expressies werken.

Twee slashes (//) na de IP-adressen: Dit is een syntactisch kenmerk van ettercap. De twee slashes worden gebruikt om het einde van de reguliere expressie aan te geven. Het is vergelijkbaar met het idee dat je het einde van de match-aangeeft.


- Task: can you intercept a ping from remote_employee to a VM behind companyrouter?

Ja, maar geen response

![hierarchy](/images/ping%20arp%20spoof.png)


## IPsec set-up

Maak het ipsec.sh script op homerouter:
```
[vagrant@homerouter ~]$ sudo vi ipsec.sh
#!/usr/bin/env sh

# Manual IPSec

## Clean all previous IPsec stuff

ip xfrm policy flush
ip xfrm state flush

## The first SA vars for the tunnel from homerouter to companyrouter

SPI7=0x007
ENCKEY7=0xFEDCBA9876543210FEDCBA9876543210

## Activate the tunnel from homerouter to companyrouter

### Define the SA (Security Association)

ip xfrm state add \
    src 192.168.62.42 \
    dst 192.168.62.253 \
    proto esp \
    spi ${SPI7} \
    mode tunnel \
    enc aes ${ENCKEY7}

### Set up the SP using this SA

ip xfrm policy add \
    src 172.10.10.0/24 \
    dst 172.30.0.0/16 \
    dir out \
    tmpl \
    src 192.168.62.42 \
    dst 192.168.62.253 \
    proto esp \
    spi ${SPI7} \
    mode tunnel
```

Maak het ipsec_fwd.sh script op companyrouter:
```
#!/usr/bin/env sh

# Manual IPSec

## The first SA vars for the tunnel from remoterouter to companyrouter

SPI7=0x007
ENCKEY7=0xFEDCBA9876543210FEDCBA9876543210 # pre shared key -> used for IKE

## Activate the tunnel from remoterouter to companyrouter

### Define the SA (Security Association)

ip xfrm state add \
    src 192.168.62.42 \
    dst 192.168.62.253 \
    proto esp \
    spi ${SPI7} \
    mode tunnel \
    enc aes ${ENCKEY7}

### Set up the SP using this SA

# 'in' policy only works with own interfaces
ip xfrm policy add \
    src 172.10.10.0/24 \
    dst 172.30.0.0/16 \
    dir in \
    tmpl \
    src 192.168.62.42 \
    dst 192.168.62.253 \
    proto esp \
    spi ${SPI7} \
    mode tunnel

# extra 'fwd' policy needed to reach hosts in the internal network
ip xfrm policy add \
    src 172.10.10.0/24 \
    dst 172.30.0.0/16 \
    dir fwd \
    tmpl \
    src 192.168.62.42 \
    dst 192.168.62.253 \
    proto esp \
    spi ${SPI7} \
    mode tunnel
```

Voer beide scripts uit, zorg ervoor dat je nftables dissabled zijn!!!

- Ping nu van remote-employee naar de web en bekijk het in wireshark

![hierarchy](/images/ESP.png)


## Encryption from companyrouter to homerouter

Maak een nieuw script aan op beide routers:
  
COMPANYROUTER:
```
#!/usr/bin/env sh

# Manual IPSec

## The first SA vars for the tunnel from companyrouter to remoterouter

SPI7=0x007
ENCKEY7=0xd280167a0f8af26085005d87a034e5f43b58b8a0106e3e09 # new pre shared key -> used for IKE

## Activate the tunnel from companyrouter to remoterouter

### Define the SA (Security Association)

ip xfrm state add \
    src 192.168.62.253 \
    dst 192.168.62.42 \
    proto esp \
    spi ${SPI7} \
    mode tunnel \
    enc aes ${ENCKEY7}

### Set up the SP using this SA

ip xfrm policy add \
    src 172.30.0.0/16 \
    dst 172.10.10.0/24 \
    dir out \
    tmpl \
    src 192.168.62.253 \
    dst 192.168.62.42 \
    proto esp \
    spi ${SPI7} \
    mode tunnel
```

HOMEROUTER:
```
#!/usr/bin/env sh

# Manual IPSec

## The first SA vars for the tunnel from companyrouter to remoterouter

SPI7=0x007
ENCKEY7=0xd280167a0f8af26085005d87a034e5f43b58b8a0106e3e09 # pre shared key -> used for IKE

## Activate the tunnel from companyrouter to remoterouter

### Define the SA (Security Association)

ip xfrm state add \
    src 192.168.62.253 \
    dst 192.168.62.42 \
    proto esp \
    spi ${SPI7} \
    mode tunnel \
    enc aes ${ENCKEY7}

### Set up the SP using this SA

ip xfrm policy add \
    src 172.30.0.0/16 \
    dst 172.10.10.0/24 \
    dir in \
    tmpl \
    src 192.168.62.253 \
    dst 192.168.62.42 \
    proto esp \
    spi ${SPI7} \
    mode tunnel

ip xfrm policy add \
    src 172.30.0.0/16 \
    dst 172.10.10.0/24 \
    dir fwd \
    tmpl \
    src 192.168.62.253 \
    dst 192.168.62.42 \
    proto esp \
    spi ${SPI7} \
    mode tunnel
```

Voer beide scripts uit en bekijk het verkeer in wireshark op de RED vm:

![hierarchy](/images/esp%20beide%20kanten.png)