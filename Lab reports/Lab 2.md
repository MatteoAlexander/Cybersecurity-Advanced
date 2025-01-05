# LAB 2: firewalls

## Attacker virtual machine red

- What routes do you need to add?

op ISProuter:
```bash 
sudo ip route add 172.30.0.0/16 via 192.168.100.253
```

- What is the default gateway?

192.168.62.253

- Does your red has internet? If not, is it possible? Why not OR how?

Ja hij heeft internet.

## The insecure "fake internet" host only network

- Use a web browser to browse to http://www.cybersec.internal

![hierarchy](/images/webserver.png)

- Use a web browser to browse to http://www.cybersec.internal/cmd and test out this insecure application.

![hierarchy](/images/webservercmd.png)

![hierarchy](/images/test.png)

- Perform a default nmap scan on all machines.
Red:  
```bash 
┌──(osboxes㉿osboxes)-[~]
└─$ sudo nmap -sn 172.30.0.0/16
[sudo] password for osboxes: 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-23 06:54 EST
Nmap scan report for 172.30.0.4
Host is up (0.00047s latency).
Nmap scan report for 172.30.0.10
Host is up (0.00033s latency).
Nmap scan report for 172.30.0.15
Host is up (0.00053s latency).
Nmap scan report for 172.30.0.123
Host is up (0.00057s latency).
```
Companyrouter: 
```bash  
┌──(osboxes㉿osboxes)-[~]
└─$ sudo nmap 172.30.255.254
[sudo] password for osboxes:                                                                                                                                                                                                                
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-23 07:18 EST                                                                                                                                                                          
Nmap scan report for 172.30.255.254                                                                                                                                                                                                         
Host is up (0.00012s latency).                                                                                                                                                                                                              
Not shown: 998 closed tcp ports (reset)                                                                                                                                                                                                     
PORT    STATE SERVICE                                                                                                                                                                                                                       
22/tcp  open  ssh                                                                                                                                                                                                                           
111/tcp open  rpcbind                                                                                                                                                                                                                       
                                                                                                                                                                                                                                     
Nmap done: 1 IP address (1 host up) scanned in 0.16 seconds    
```
DNS:
```bash 
──(osboxes㉿osboxes)-[~]                                                                                                                                                                                                                   
└─$ sudo nmap 172.30.0.4                                                                                                                                                                                                                    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-23 07:21 EST                                                                                                                                                                          
Nmap scan report for 172.30.0.4                                                                                                                                                                                                             
Host is up (0.00022s latency).                                                                                                                                                                                                              
Not shown: 998 closed tcp ports (reset)                                                                                                                                                                                                     
PORT   STATE SERVICE                                                                                                                                                                                                                        
22/tcp open  ssh                                                                                                                                                                                                                            
53/tcp open  domain                                                                                                                                                                                                                         
                                                                                                                                                                                                                                            
Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds                                                                                                                                                                                 
                                                            
```
Employee:
```bash 
┌──(osboxes㉿osboxes)-[~]                                                                                                                                                                                                                   
└─$ sudo nmap 172.30.0.123
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-23 07:22 EST
Nmap scan report for 172.30.0.123
Host is up (0.00032s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 0.17 seconds

```
WEB:
```bash 
┌──(osboxes㉿osboxes)-[~]
└─$ sudo nmap 172.30.0.10
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-23 07:23 EST
Nmap scan report for 172.30.0.10
Host is up (0.00026s latency).
Not shown: 995 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
111/tcp  open  rpcbind
8000/tcp open  http-alt
9200/tcp open  wap-wsp

Nmap done: 1 IP address (1 host up) scanned in 0.17 seconds

```
DATABASE:
```bash 
┌──(osboxes㉿osboxes)-[~]
└─$ sudo nmap 172.30.0.15
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-23 07:23 EST
Nmap scan report for 172.30.0.15
Host is up (0.00024s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
3306/tcp open  mysql

Nmap done: 1 IP address (1 host up) scanned in 0.18 seconds

```

- Enumerate the most interesting ports (you found in the previous step) by issuing a service enumeration scan (banner grab scan).  

Companyrouter:
```bash 
┌──(osboxes㉿osboxes)-[~]
└─$ sudo nmap -sV 172.30.255.254                                                                                                                                                                                                          
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-23 07:26 EST
Nmap scan report for 172.30.255.254
Host is up (0.00010s latency).
Not shown: 998 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.7 (protocol 2.0)
111/tcp open  rpcbind 2-4 (RPC #100000)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.44 seconds

```

DNS:
```bash 
┌──(osboxes㉿osboxes)-[~]
└─$ sudo nmap -sV 172.30.0.4                                                                                                                                                                                                       
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-23 07:28 EST
Nmap scan report for 172.30.0.4
Host is up (0.00024s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.3 (protocol 2.0)
53/tcp open  domain  ISC BIND 9.18.32

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.35 seconds

```

Employee:
```bash
┌──(osboxes㉿osboxes)-[~]
└─$ sudo nmap -sV 172.30.0.123                                                                                                                                                                                                     
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-23 07:29 EST
Nmap scan report for 172.30.0.123
Host is up (0.00025s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.3 (protocol 2.0)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.37 seconds
```

WEB:
```bash
┌──(osboxes㉿osboxes)-[~]
└─$ sudo nmap -sV 172.30.0.10                                                                                                                                                                                                      
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-23 07:29 EST
WARNING: Service 172.30.0.10:8000 had already soft-matched rtsp, but now soft-matched sip; ignoring second value
Nmap scan report for 172.30.0.10
Host is up (0.00024s latency).
Not shown: 995 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.7 (protocol 2.0)
80/tcp   open  http     Apache httpd 2.4.62 ((AlmaLinux))
111/tcp  open  rpcbind  2-4 (RPC #100000)
8000/tcp open  rtsp
9200/tcp open  wap-wsp?
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port8000-TCP:V=7.94SVN%I=7%D=12/23%Time=676957C4%P=x86_64-pc-linux-gnu%
SF:r(GetRequest,32A,"HTTP/1\.0\x20200\x20OK\r\naccept-ranges:\x20bytes\r\n
SF:content-length:\x20577\r\ncache-control:\x20public,\x20immutable,\x20ma
SF:x-age=86400\r\nlast-modified:\x20Mon,\x2023\x20Dec\x202024\x2011:49:33\
SF:x20GMT\r\ndate:\x20Mon,\x2023\x20Dec\x202024\x2012:29:56\x20GMT\r\ncont
SF:ent-type:\x20text/html;charset=UTF-8\r\n\r\n<!DOCTYPE\x20html>\n<html\x
SF:20lang=\"en\">\n<head>\n\x20\x20\x20\x20<meta\x20charset=\"UTF-8\">\n\x
SF:20\x20\x20\x20<title>Command\x20Injection</title>\n\x20\x20\x20\x20<scr
SF:ipt\x20src=\"assets/javascript/index\.js\"></script>\n</head>\n<body>\n
SF:<h1>Command\x20Injection</h1>\n<form\x20id=\"ping\">\n\x20\x20\x20\x20<
SF:h2>Ping\x20a\x20device</h2>\n\x20\x20\x20\x20<label\x20for=\"ip\">Enter
SF:\x20an\x20IP\x20address</label>\n\x20\x20\x20\x20<input\x20id=\"ip\"\x2
SF:0type=\"text\"/>\n\x20\x20\x20\x20<input\x20type=\"submit\"\x20value=\"
SF:PING\">\n</form>\n\n<form\x20id=\"exec\">\n\x20\x20\x20\x20<h2>Execute\
SF:x20a\x20command</h2>\n\x20\x20\x20\x20<label\x20for=\"cmd\">Enter\x20a\
SF:x20command</label>\n\x20\x20\x20\x20<input\x20id=\"cmd\"\x20type=\"text
SF:\"/>\n\x20\x20\x20\x20<input\x20type=\"submit\"\x20value=\"EXEC\">\n</f
SF:orm>\n\n<pre></pre>\n</body>\n</html>\n")%r(FourOhFourRequest,8B,"HTTP/
SF:1\.0\x20404\x20Not\x20Found\r\ncontent-type:\x20text/html;\x20charset=u
SF:tf-8\r\ncontent-length:\x2053\r\n\r\n<html><body><h1>Resource\x20not\x2
SF:0found</h1></body></html>")%r(Socks5,2F,"HTTP/1\.0\x20400\x20Bad\x20Req
SF:uest\r\ncontent-length:\x200\r\n\r\n")%r(HTTPOptions,8B,"HTTP/1\.0\x204
SF:04\x20Not\x20Found\r\ncontent-type:\x20text/html;\x20charset=utf-8\r\nc
SF:ontent-length:\x2053\r\n\r\n<html><body><h1>Resource\x20not\x20found</h
SF:1></body></html>")%r(RTSPRequest,33,"RTSP/1\.0\x20501\x20Not\x20Impleme
SF:nted\r\ncontent-length:\x200\r\n\r\n")%r(SIPOptions,32,"SIP/2\.0\x20501
SF:\x20Not\x20Implemented\r\ncontent-length:\x200\r\n\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port9200-TCP:V=7.94SVN%I=7%D=12/23%Time=676957BF%P=x86_64-pc-linux-gnu%
SF:r(GetRequest,4E5,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/3\.1\.3
SF:\x20Python/3\.9\.21\r\nDate:\x20Mon,\x2023\x20Dec\x202024\x2012:29:51\x
SF:20GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length
SF::\x201078\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20l
SF:ang=\"en\">\n<head>\n<meta\x20charset=\"UTF-8\">\n<meta\x20name=\"viewp
SF:ort\"\x20content=\"width=device-width,\x20initial-scale=1\.0\">\n<title
SF:>Cybersec\x20Internal\x20Services</title>\n<style>\n\x20\x20\x20\x20\x2
SF:0\x20\x20\x20body\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0font-family:\x20Arial,\x20sans-serif;\n\x20\x20\x20\x20\x20\x20\x20\x2
SF:0}\n\x20\x20\x20\x20\x20\x20\x20\x20\.content\x20{\n\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20text-align:\x20center;\n\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20margin-top:\x2050px;\n\x20\x20\x20\x20\x2
SF:0\x20\x20\x20}\n\x20\x20\x20\x20\x20\x20\x20\x20button\x20{\n\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20font-size:\x2018px;\n\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20padding:\x2010px\x2020px;\n\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20border:\x20none;\n\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20cursor:\x20pointer;\n\x20\x20\x20\x2
SF:0\x20\x20\x20\x20}\n\x20\x20\x20\x20\x20\x20\x20\x20\.success\x20{\n\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20background-color:\x20green
SF:;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20color:\x20white;\n\x
SF:20\x20\x20\x20\x20\x20\x20\x20}\n\x20\x20\x20\x20\x20\x20\x20\x20\.fail
SF:ure\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20background-co
SF:lor:\x20red;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20color:\x2
SF:0white;\n\x20\x20\x20\x20\x20\x20\x20\x20}\n\x20\x20\x20\x20\x20\x20\x2
SF:0\x20img\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20width:\x
SF:20200px;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20heig")%r(HTTP
SF:Options,C7,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/3\.1\.3\x20Py
SF:thon/3\.9\.21\r\nDate:\x20Mon,\x2023\x20Dec\x202024\x2012:29:51\x20GMT\
SF:r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nAllow:\x20OPTIONS,\x
SF:20GET,\x20HEAD\r\nContent-Length:\x200\r\nConnection:\x20close\r\n\r\n"
SF:);

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 87.56 seconds

```

DATABASE:
```bash
┌──(osboxes㉿osboxes)-[~]
└─$ sudo nmap -sV 172.30.0.15                                                                                                                                                                                                      
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-23 07:32 EST
Nmap scan report for 172.30.0.15
Host is up (0.00030s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.3 (protocol 2.0)
3306/tcp open  mysql   MySQL 5.5.5-10.11.10-MariaDB

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.36 seconds
```

- What database software is running on the database machine? What version?  

MySQL 5.5.5-10.11.10-MariaDB  

- Try to search for a nmap script to brute-force the database. Another (even easier tool) is called hydra (https://github.com/vanhauser-thc/thc-hydra). Search online for a good wordlist. For example "rockyou" or https://github.com/danielmiessler/SecLists We suggest to try the default username of the database software and attack the database machine. Another interesting username worth a try is "toor".  

RED Debian:  

```bash
root@red:~# sudo hydra -s 3306 -l toor -P SecLists/Passwords/xato-net-10-million-passwords-100000.txt mysql://172.30.0.15
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-12-23 13:43:01
[INFO] Reduced number of tasks to 4 (mysql does not like many parallel connections)
[DATA] max 4 tasks per 1 server, overall 4 tasks, 100000 login tries (l:1/p:100000), ~25000 tries per task
[DATA] attacking mysql://172.30.0.15:3306/
[3306][mysql] host: 172.30.0.15   login: toor   password: summer
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-12-23 13:43:02
```

- What webserver software is running on web?

Apache httpd 2.4.62 ((AlmaLinux))

- Try the -sC option with nmap. What is this option?

SC = Script scan  

De optie -sC bij Nmap staat voor "Default Scripts". Wanneer je deze optie gebruikt, voert Nmap een reeks standaard scripts uit die deel uitmaken van de Nmap Scripting Engine (NSE). Deze scripts zijn ontworpen om aanvullende informatie over het doelwit te verzamelen.  

Deze scripts kunnen verschillende taken uitvoeren, zoals:  
- Versie-informatie van services achterhalen.
- Detecteren van kwetsbaarheden.
- Openstaande poorten verder onderzoeken.
- Identificeren van bekende protocollen en configuratiefouten.

```bash
┌──(osboxes㉿osboxes)-[~]
└─$ sudo nmap -sC 172.30.0.123
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-23 08:23 EST
Nmap scan report for 172.30.0.123
Host is up (0.00024s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
| ssh-hostkey: 
|   256 e3:de:29:1b:4d:f5:78:1b:44:78:19:e0:e8:8b:c6:5d (ECDSA)
|_  256 7d:f5:77:a9:c5:cf:db:41:41:7b:7e:c1:0a:de:da:40 (ED25519)

Nmap done: 1 IP address (1 host up) scanned in 0.53 seconds
```

- Try to SSH (using vagrant/vagrant) from red to another machine. Is this possible?

Ja:
```bash
┌──(osboxes㉿osboxes)-[~]
└─$ ssh vagrant@192.168.62.253
The authenticity of host '192.168.62.253 (192.168.62.253)' can't be established.
ED25519 key fingerprint is SHA256:nzj+ffgg9TeSlENQNO98Sp7KnMBiS/QTFO+kAxr0sas.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.62.253' (ED25519) to the list of known hosts.
vagrant@192.168.62.253's password: 
Last login: Mon Dec 23 12:16:00 2024 from 192.168.62.1
[vagrant@companyrouter ~]$ 
```

## Network Segmentation

- What is meant with the term "attack vector"?

Een attack vector (aanvalsvector) is een methode, pad, of techniek die een aanvaller gebruikt om ongeoorloofd toegang te krijgen tot een systeem, netwerk, applicatie, of gegevens. Het verwijst naar de manier waarop een cyberaanval wordt uitgevoerd, inclusief de zwakke punten of kwetsbaarheden die worden uitgebuit.  

- Is there already network segmentation done on the company network?

Nee het is 1 groot netwerk namelijk: 172.30.0.0/16  

- Remember what a DMZ is? What machines would be in the DMZ in this environment?

Enkel de web VM moet in een DMZ  

- What could be a disadvantage of using network segmentation in this case? Tip: client <-> server interaction.

Al het verkeer moet door de companyrouter waardoor dit een single point of failure wordt    

  

Configure the environment, and especially companyrouter, to make sure that the red machine is not able to interact with most systems anymore. The only requirements that are left for the red machine are:  

- Browsing to http://www.cybersec.internal should work. Note: you are allowed to manually add a DNS entry to the red machine to tell the system how to resolve "www.cybersec.internal" if necessary. Do be mindful why/when this is needed!
- All machines in the company network should still have internet access
- You should verify what functionality you might lose implementing the network segmentation. List out and create an overview of the advantages and disadvantages.
- You should be able to revert back easily: Create proper documentation!

Aanpassingen:




## Firewall

Maak de nftables.conf file aan. Deze ziet er zo uit:
```bash
#!/usr/sbin/nft -f

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;

        # Sta loopback-verkeer toe
        iif "lo" accept

        # Sta verkeer toe dat deel uitmaakt van bestaande of gerelateerde sessies
        ct state established,related accept

        # Sta SSH-verkeer toe (vervang <your_ssh_port> met uw SSH-poort)
        tcp dport 22 accept

        # Sta HTTP (80) en HTTPS (443) toe
        tcp dport 80 accept
        tcp dport 443 accept

        # Log geblokkeerd verkeer (optioneel)
        # log prefix "Blocked: " level debug
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
    }

    chain output {
        type filter hook output priority 0; policy accept;
    }
}
```

Activeer de firewall regels:
```bash
[vagrant@companyrouter ~]$ sudo nft -f /etc/nftables.conf
[vagrant@companyrouter ~]$ sudo systemctl enable nftables
Created symlink /etc/systemd/system/multi-user.target.wants/nftables.service → /usr/lib/systemd/system/nftables.service.
[vagrant@companyrouter ~]$ sudo systemctl start nftables
```


## Open, closed, filtered ports

```bash
──(osboxes㉿osboxes)-[~]
└─$ sudo nmap -p 80,22,666 172.30.0.10                                                                                                                                                                                                      
[sudo] password for osboxes: 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-23 10:42 EST
Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
Nmap done: 1 IP address (0 hosts up) scanned in 3.09 seconds

┌──(osboxes㉿osboxes)-[~]
└─$ sudo nmap -Pn -p 80,22,666 172.30.0.10                                                                                                                                                                                                  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-23 10:43 EST
Nmap scan report for www.cybersec.internal (172.30.0.10)
Host is up.

PORT    STATE    SERVICE
22/tcp  filtered ssh
80/tcp  filtered http
666/tcp filtered doom

Nmap done: 1 IP address (1 host up) scanned in 3.10 seconds
```