# Lab 5: Honeypots

## Cowrie

Installeer docker:
```bash
[vagrant@companyrouter ~]$ sudo yum install -y docker-ce docker-ce-cli containerd.io
```
```bash
[vagrant@companyrouter ~]$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: disabled)
     Active: active (running) since Thu 2024-12-26 13:39:18 UTC; 6s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 6137 (dockerd)
      Tasks: 7
     Memory: 26.0M
        CPU: 188ms
     CGroup: /system.slice/docker.service
             └─6137 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Dec 26 13:39:17 companyrouter systemd[1]: Starting Docker Application Container Engine...
Dec 26 13:39:17 companyrouter dockerd[6137]: time="2024-12-26T13:39:17.639680310Z" level=info msg="Starting up"
Dec 26 13:39:17 companyrouter dockerd[6137]: time="2024-12-26T13:39:17.640599827Z" level=info msg="OTEL tracing is not configured, using no-op tracer provi>
Dec 26 13:39:17 companyrouter dockerd[6137]: time="2024-12-26T13:39:17.668518732Z" level=info msg="Loading containers: start."
Dec 26 13:39:17 companyrouter dockerd[6137]: time="2024-12-26T13:39:17.965265485Z" level=info msg="Loading containers: done."
Dec 26 13:39:17 companyrouter dockerd[6137]: time="2024-12-26T13:39:17.980257835Z" level=info msg="Docker daemon" commit=c710b88 containerd-snapshotter=fal>
Dec 26 13:39:17 companyrouter dockerd[6137]: time="2024-12-26T13:39:17.980441888Z" level=info msg="Daemon has completed initialization"
Dec 26 13:39:18 companyrouter dockerd[6137]: time="2024-12-26T13:39:18.003567887Z" level=info msg="API listen on /run/docker.sock"
Dec 26 13:39:18 companyrouter systemd[1]: Started Docker Application Container Engine.
lines 1-21/21 (END)
```


- Why is companyrouter, in this environment, an interesting device to configure with a SSH honeypot? What could be a good argument to NOT configure the router with a honeypot service?

Aangezien dit apparaat verbonden is met het internet, is de kans groot dat het gescand zal worden op standaardpoorten. Als aanvallers hier tijd verliezen en niet verder zoeken naar bijvoorbeeld de bastionserver: winst. Het zou ook een strikte firewall moeten hebben die draait. Aan de andere kant, als deze router gecompromitteerd wordt, zitten we in serieuze problemen. Deze machine heeft ook andere functies (routeren) die essentieel zijn. Misschien is het een goed idee om een ​​toegewijde, geïsoleerde machine op te zetten die als honeypot fungeert, zowel binnen als buiten de firewall.

- Change your current SSH configuration in such a way that the SSH server (daemon) is not listening on port 22 anymore but on port 2222.

Doe deze commando's om het in de SELinux settings aan te passen:
```bash
[vagrant@companyrouter ~]$ sudo yum install policycoreutils-python-utils
```
```bash
[vagrant@companyrouter ~]$ sudo semanage port -a -t ssh_port_t -p tcp 2222
```

Pas het ook aan in de sshd config file:
```bash
[vagrant@companyrouter ~]$ sudo nano /etc/ssh/sshd_config
```
```bash
Port 2222
```

- Install and run the cowrie software on the router and listen on port 22 - the default SSH server port.
```bash
[vagrant@companyrouter ~]$  sudo docker run -p 22:2222 cowrie/cowrie:latest

[vagrant@companyrouter ~]$ ss -tulpn
Netid          State           Recv-Q          Send-Q                    Local Address:Port                     Peer Address:Port          Process
udp            UNCONN          0               0                               0.0.0.0:111                           0.0.0.0:*
udp            UNCONN          0               0                             127.0.0.1:323                           0.0.0.0:*
udp            UNCONN          0               0                                  [::]:111                              [::]:*
udp            UNCONN          0               0                                 [::1]:323                              [::]:*
tcp            LISTEN          0               4096                            0.0.0.0:22                            0.0.0.0:*
tcp            LISTEN          0               4096                            0.0.0.0:111                           0.0.0.0:*
tcp            LISTEN          0               128                             0.0.0.0:2222                          0.0.0.0:*
tcp            LISTEN          0               4096                               [::]:22                               [::]:*
tcp            LISTEN          0               4096                               [::]:111                              [::]:*
tcp            LISTEN          0               128                                [::]:2222                             [::]:*
```

- Once configured and up and running, verify that you can still SSH to the router normally, using port 2222.
```bash
PS C:\Users\matte> ssh vagrant@companyrouter -p 22
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:e+xlkJ/eyEGLE8JU+IHk6uEQZ6MNH2iA4X8p6BZhYG4.
Please contact your system administrator.
Add correct host key in C:\\Users\\matte/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in C:\\Users\\matte/.ssh/known_hosts:6
Host key for 192.168.62.253 has changed and you have requested strict checking.
Host key verification failed.
```
```bash
2024-12-26T14:26:47+0000 [-] Reading configuration from ['/cowrie/cowrie-git/etc/cowrie.cfg.dist']
2024-12-26T14:26:47+0000 [-] Python Version 3.11.2 (main, Sep 14 2024, 03:00:30) [GCC 12.2.0]
2024-12-26T14:26:47+0000 [-] Twisted Version 24.10.0
2024-12-26T14:26:47+0000 [-] Cowrie Version 2.6.1
2024-12-26T14:26:47+0000 [-] Loaded output engine: jsonlog
2024-12-26T14:26:47+0000 [twisted.scripts._twistd_unix.UnixAppLogger#info] twistd 24.10.0 (/cowrie/cowrie-env/bin/python3 3.11.2) starting up.
2024-12-26T14:26:47+0000 [twisted.scripts._twistd_unix.UnixAppLogger#info] reactor class: twisted.internet.epollreactor.EPollReactor.
2024-12-26T14:26:47+0000 [-] CowrieSSHFactory starting on 2222
2024-12-26T14:26:47+0000 [cowrie.ssh.factory.CowrieSSHFactory#info] Starting factory <cowrie.ssh.factory.CowrieSSHFactory object at 0x7f9ac62dfd90>
2024-12-26T14:26:47+0000 [-] Generating new RSA keypair...
2024-12-26T14:26:47+0000 [-] Generating new ECDSA keypair...
2024-12-26T14:26:47+0000 [-] Generating new ed25519 keypair...
2024-12-26T14:26:47+0000 [-] Ready to accept SSH connections
2024-12-26T14:26:58+0000 [cowrie.ssh.factory.CowrieSSHFactory] No moduli, no diffie-hellman-group-exchange-sha1
2024-12-26T14:26:58+0000 [cowrie.ssh.factory.CowrieSSHFactory] No moduli, no diffie-hellman-group-exchange-sha256
2024-12-26T14:26:58+0000 [cowrie.ssh.factory.CowrieSSHFactory] New connection: 192.168.62.1:55371 (172.17.0.2:2222) [session: 68801eef6e16]
2024-12-26T14:26:58+0000 [HoneyPotSSHTransport,0,192.168.62.1] Remote SSH version: SSH-2.0-OpenSSH_for_Windows_9.5
2024-12-26T14:26:58+0000 [HoneyPotSSHTransport,0,192.168.62.1] SSH client hassh fingerprint: 701158e75b508e76f0410d5d22ef9df0
2024-12-26T14:26:58+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2024-12-26T14:26:58+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2024-12-26T14:26:58+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2024-12-26T14:26:58+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2024-12-26T14:26:58+0000 [HoneyPotSSHTransport,0,192.168.62.1] Connection lost after 0.1 seconds
```


- Attack your router and try to SSH normally. What do you notice?

Nu ssh je naar de honeypot. (op ip adres met user 'root')

- What credentials work? Do you find credentials that don't work?

Alle credentials werken

```bash
PS C:\Users\matte> ssh root@192.168.62.253
root@192.168.62.253's password:

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@svr04:~#
```

```bash
2024-12-26T14:37:56+0000 [cowrie.ssh.connection.CowrieSSHConnection#info] sending close 0
2024-12-26T14:37:56+0000 [cowrie.ssh.session.HoneyPotSSHSession#info] remote close
2024-12-26T14:37:56+0000 [HoneyPotSSHTransport,6,192.168.62.1] Got remote error, code 11 reason: b'disconnected by user'
2024-12-26T14:37:56+0000 [HoneyPotSSHTransport,6,192.168.62.1] avatar root logging out
2024-12-26T14:37:56+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2024-12-26T14:37:56+0000 [HoneyPotSSHTransport,6,192.168.62.1] Connection lost after 7.0 seconds
2024-12-26T14:37:57+0000 [cowrie.ssh.factory.CowrieSSHFactory] No moduli, no diffie-hellman-group-exchange-sha1
2024-12-26T14:37:57+0000 [cowrie.ssh.factory.CowrieSSHFactory] No moduli, no diffie-hellman-group-exchange-sha256
2024-12-26T14:37:57+0000 [cowrie.ssh.factory.CowrieSSHFactory] New connection: 192.168.62.1:55529 (172.17.0.2:2222) [session: ec0e456a353f]
2024-12-26T14:37:57+0000 [HoneyPotSSHTransport,7,192.168.62.1] Remote SSH version: SSH-2.0-OpenSSH_for_Windows_9.5
2024-12-26T14:37:57+0000 [HoneyPotSSHTransport,7,192.168.62.1] SSH client hassh fingerprint: 701158e75b508e76f0410d5d22ef9df0
2024-12-26T14:37:57+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2024-12-26T14:37:57+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2024-12-26T14:37:57+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2024-12-26T14:37:57+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2024-12-26T14:37:57+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#debug] starting service b'ssh-userauth'
2024-12-26T14:37:57+0000 [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'root' trying auth b'none'
2024-12-26T14:37:57+0000 [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'root' trying auth b'publickey'
2024-12-26T14:37:57+0000 [HoneyPotSSHTransport,7,192.168.62.1] public key attempt for user b'root' of type b'ssh-ed25519' with fingerprint 9b:f8:22:a6:65:9c:42:6c:42:dd:1d:05:69:34:82:b5
2024-12-26T14:37:57+0000 [HoneyPotSSHTransport,7,192.168.62.1] public key login attempt for [b'root'] failed
2024-12-26T14:37:57+0000 [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'root' failed auth b'publickey'
2024-12-26T14:37:57+0000 [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] reason: ('Incorrect signature', None)
2024-12-26T14:37:59+0000 [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'root' trying auth b'password'
2024-12-26T14:37:59+0000 [HoneyPotSSHTransport,7,192.168.62.1] Could not read etc/userdb.txt, default database activated
2024-12-26T14:37:59+0000 [HoneyPotSSHTransport,7,192.168.62.1] login attempt [b'root'/b'vagrant'] succeeded
2024-12-26T14:37:59+0000 [HoneyPotSSHTransport,7,192.168.62.1] Initialized emulated server as architecture: linux-x64-lsb
2024-12-26T14:37:59+0000 [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'root' authenticated with b'password'
2024-12-26T14:37:59+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#debug] starting service b'ssh-connection'
2024-12-26T14:37:59+0000 [cowrie.ssh.connection.CowrieSSHConnection#debug] got channel b'session' request
2024-12-26T14:37:59+0000 [cowrie.ssh.session.HoneyPotSSHSession#info] channel open
2024-12-26T14:37:59+0000 [cowrie.ssh.connection.CowrieSSHConnection#debug] got global b'no-more-sessions@openssh.com' request
2024-12-26T14:37:59+0000 [twisted.conch.ssh.session#info] Handling pty request: b'xterm-256color' (41, 156, 640, 480)
2024-12-26T14:37:59+0000 [SSHChannel session (0) on SSHService b'ssh-connection' on HoneyPotSSHTransport,7,192.168.62.1] Terminal Size: 156 41
2024-12-26T14:37:59+0000 [twisted.conch.ssh.session#info] Getting shell
2024-12-26T14:38:05+0000 [HoneyPotSSHTransport,7,192.168.62.1] CMD: exit
2024-12-26T14:38:05+0000 [HoneyPotSSHTransport,7,192.168.62.1] Command found: exit
2024-12-26T14:38:05+0000 [twisted.conch.ssh.session#info] exitCode: 0
2024-12-26T14:38:05+0000 [cowrie.ssh.connection.CowrieSSHConnection#debug] sending request b'exit-status'
2024-12-26T14:38:05+0000 [HoneyPotSSHTransport,7,192.168.62.1] Closing TTY Log: var/lib/cowrie/tty/2638f1c1c2018567a46a4cae049dd90db2d468e1538d60d328f2707d071f73c5 after 6.1 seconds
2024-12-26T14:38:05+0000 [cowrie.ssh.connection.CowrieSSHConnection#info] sending close 0
2024-12-26T14:38:05+0000 [cowrie.ssh.session.HoneyPotSSHSession#info] remote close
2024-12-26T14:38:05+0000 [HoneyPotSSHTransport,7,192.168.62.1] Got remote error, code 11 reason: b'disconnected by user'
2024-12-26T14:38:05+0000 [HoneyPotSSHTransport,7,192.168.62.1] avatar root logging out
2024-12-26T14:38:05+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2024-12-26T14:38:05+0000 [HoneyPotSSHTransport,7,192.168.62.1] Connection lost after 8.0 seconds
2024-12-26T14:38:10+0000 [cowrie.ssh.factory.CowrieSSHFactory] No moduli, no diffie-hellman-group-exchange-sha1
2024-12-26T14:38:10+0000 [cowrie.ssh.factory.CowrieSSHFactory] No moduli, no diffie-hellman-group-exchange-sha256
2024-12-26T14:38:10+0000 [cowrie.ssh.factory.CowrieSSHFactory] New connection: 192.168.62.1:55531 (172.17.0.2:2222) [session: c18366a14961]
2024-12-26T14:38:10+0000 [HoneyPotSSHTransport,8,192.168.62.1] Remote SSH version: SSH-2.0-OpenSSH_for_Windows_9.5
2024-12-26T14:38:10+0000 [HoneyPotSSHTransport,8,192.168.62.1] SSH client hassh fingerprint: 701158e75b508e76f0410d5d22ef9df0
2024-12-26T14:38:10+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2024-12-26T14:38:10+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2024-12-26T14:38:10+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2024-12-26T14:38:10+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2024-12-26T14:38:10+0000 [cowrie.ssh.transport.HoneyPotSSHTransport#debug] starting service b'ssh-userauth'
2024-12-26T14:38:10+0000 [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'vagrant' trying auth b'none'
2024-12-26T14:38:10+0000 [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'vagrant' trying auth b'publickey'
2024-12-26T14:38:10+0000 [HoneyPotSSHTransport,8,192.168.62.1] public key attempt for user b'vagrant' of type b'ssh-ed25519' with fingerprint 9b:f8:22:a6:65:9c:42:6c:42:dd:1d:05:69:34:82:b5
```

- Do you get a shell?

Ja 

- Are your commands logged? Is the IP address of the SSH client logged? If this is the case, where?

Ja de commando's zijn logged in de container logs: 
```bash
2024-12-26T14:43:31+0000 [HoneyPotSSHTransport,10,192.168.62.1] CMD: pwd
2024-12-26T14:43:31+0000 [HoneyPotSSHTransport,10,192.168.62.1] Command found: pwd
2024-12-26T14:43:33+0000 [HoneyPotSSHTransport,10,192.168.62.1] CMD: ls
2024-12-26T14:43:33+0000 [HoneyPotSSHTransport,10,192.168.62.1] Command found: ls
```
- Can an attacker perform malicious things?

Not really

- Are the actions, in other words the commands, logged to a file? Which file?

```bash
2024-12-26T14:37:56+0000 [HoneyPotSSHTransport,6,192.168.62.1] Closing TTY Log: var/lib/cowrie/tty/2638f1c1c2018567a46a4cae049dd90db2d468e1538d60d328f2707d071f73c5 after 5.7 seconds
```

- If you are an experienced hacker, how would/can you realize this is not a normal environment?

geen pakketbeheerder  
zeer weinig omgevingsvariabelen  
bash-geschiedenis wordt verwijderd na opnieuw inloggen  
geen netwerkconfiguratie te vinden (geen ip-commando, geen /etc/network/interfaces)  


## Critical thinking (security) when using "Docker as a service"

- What are some (at least 2) advantages of running services (for example cowrie but it could be sql server as well) using docker?
  
Isolatie: Docker biedt een geïsoleerde omgeving voor elke service. Dit betekent dat afhankelijkheden en configuraties voor één service geen andere services beïnvloeden.

Consistentie en Draagbaarheid: Docker containers kunnen over verschillende omgevingen heen consistent draaien, wat de draagbaarheid verhoogt. Een container die lokaal werkt, zal op dezelfde manier werken in een productieomgeving.


- What could be a disadvantage? Give at least 1.

Performantie

- Explain what is meant with "Docker uses a client-server architecture."
  
Docker gebruikt een client-server architectuur, wat betekent dat de Docker client communiceert met de Docker daemon, die de zware taken van het bouwen, draaien en distribueren van Docker containers uitvoert.


- As which user is the docker daemon running by default? Tip: https://docs.docker.com/engine/install/linux-postinstall/ .
  
De Docker daemon draait standaard als de root gebruiker.

- What could be an advantage of running a honeypot inside a virtual machine compared to running it inside a container?

Volledige Isolatie: Een virtuele machine biedt volledige isolatie op hardwareniveau, waardoor het moeilijker is voor een aanvaller om toegang te krijgen tot de host of andere virtuele machines, in tegenstelling tot containers die meer afhankelijk zijn van de hostkernel. Dit kan extra beveiliging bieden voor een honeypot-toepassing.

## Other honeypots

- What type of honeypot is "honeyup"?

Een HoneyUP is een specifiek type honeypot dat zich richt op het imiteren van kwetsbare of verouderde systemen of applicaties, vaak gericht op specifieke services of protocollen. Dit maakt het mogelijk om aanvallers aan te trekken en hun gedrag te analyseren.

- What is the idea behind "opencanary"?

Het wordt voornamelijk gebruikt om hackers te pakken nadat ze hebben ingebroken in niet-openbare netwerken.

- Is a HTTP(S) honeypot a good idea? Why or why not?

Ja, een HTTP(S) honeypot is een goed idee omdat het effectief kan zijn bij het detecteren en analyseren van webgebaseerde aanvallen, zoals SQL-injecties, cross-site scripting, en brute force aanvallen.