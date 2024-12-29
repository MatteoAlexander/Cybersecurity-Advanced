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