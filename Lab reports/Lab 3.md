# LAB 3: SSH

## SSH client config


Maak een publieke sleutel aan of vraag deze op indien je die al hebt:
```bash
 type $env:USERPROFILE\.ssh\id_rsa.pub
 ```

 Ga naar de machine waarop je makkelijk wilt kunnen ssh'en en doe volgende commando's:
```bash
ssh vagrant@192.168.62.254
mkdir -p ~/.ssh
echo "<gekopieerde_public_key>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

Voeg op je eigen laptop dit toe in de C:\Users\matte\.ssh\config file:

```bash
Host isprouter
  HostName 192.168.62.254
  User vagrant
  IdentityFile ~/.ssh/id_rsa
```

Voor de machines die je via een jumphost moet bereiken is de werking hetzelfde, enkel moet je dit in je lokale config file zetten:
```bash
Host dns
    HostName 172.30.0.4
    User vagrant
    ProxyJump companyrouter
    IdentityFile ~/.ssh/id_rsa
```

Nu zal het zonder problemen en zonder wachtwoord moeten lukken:
```bash
PS C:\Users\matte> ssh vagrant@isprouter
isprouter:~$ exit
logout
Connection to 192.168.62.254 closed.
PS C:\Users\matte> ssh vagrant@dns
dns:~$
```

Wat maakt deze methode veilig?  
  

Beperkingen op direct toegang:

Door via een jump host (zoals companyrouter) verbinding te maken met andere VM's, beperk je directe toegang tot de achterliggende systemen.
Dit voorkomt dat de achterliggende systemen direct blootgesteld worden aan het internet of andere ongecontroleerde netwerken.
Centralisatie van toegang:

Je centraliseert de toegang via de jump host. Dit betekent dat je alle toegang tot interne systemen via één toegangspunt beheert, wat makkelijker is om te monitoren en beveiligen.
Gebruik van SSH-sleutels:

Het gebruik van SSH-sleutels in plaats van wachtwoorden maakt de verbinding veel veiliger. SSH-sleutels zijn veel moeilijker te kraken dan wachtwoorden, vooral als ze goed beveiligd zijn met een passphrase.
Geen wachtwoorden in transit:

Aangezien je alleen SSH-sleutels gebruikt, worden er geen wachtwoorden via het netwerk verstuurd, wat de kans op wachtwoord-interceptie of brute-force aanvallen verkleint.


Verbeteringen:  
  
SSH-Sleutelbeveiliging
Gebruik sterke wachtwoorden voor sleutels: Als je SSH-sleutels gebruikt, zorg dan dat ze beveiligd zijn met een sterke passphrase. Hierdoor wordt de sleutel ook beschermd als iemand toegang krijgt tot het bestand zelf.
Beperk toegang tot private sleutels: Zorg ervoor dat de private sleutels (bijvoorbeeld id_rsa) alleen toegankelijk zijn voor de eigenaar (gebruik chmod 600 ~/.ssh/id_rsa).  
  
Logging op de jump host: Zorg ervoor dat je gedetailleerde logging hebt ingeschakeld op de jump host (bijvoorbeeld met syslog of via Auditd). Dit helpt om inbraakpogingen te detecteren en te reageren.


## SSH Port Forwarding

nmap naar mariadb poort op database vm (no ping)  

```bash
┌──(osboxes㉿osboxes)-[~]                                                                                                                                                                                                                   
└─$ sudo nmap -Pn -p3306 -sV 172.30.0.15
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-24 07:47 EST                                                                                                                                                                          
Nmap scan report for 172.30.0.15                                                                                                                                                                                                            
Host is up (0.00030s latency).                                                                                                                                                                                                              
                                                                                                                                                                                                                                            
PORT     STATE SERVICE VERSION                                                                                                                                                                                                              
3306/tcp open  mysql   MySQL 5.5.5-10.11.10-MariaDB

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.27 seconds
```

Lokale tunnel naar db server met de companyrouter als jumphost

```bash
┌──(osboxes㉿osboxes)-[~]
└─$ ssh -L 3306:172.30.0.15:3306 vagrant@192.168.62.253                                                                                                                                                                                  
vagrant@192.168.62.253's password: 
Last login: Tue Dec 24 11:55:24 2024 from 192.168.62.1
[vagrant@companyrouter ~]$ 
```


- Why is this an interesting approach from a security point-of-view?  

Versleutelde tunnel: Het creëert een versleutelde SSH-tunnel voor het transporteren van netwerkverkeer, wat handig is voor het verzenden van gevoelige gegevens over onbeveiligde of niet-vertrouwde netwerken.

Firewalls omzeilen: Het kan gebruikt worden om firewall regels te omzeilen die bepaalde poorten of diensten blokkeren, waardoor netwerkconnectiviteit mogelijk wordt die anders beperkt zou zijn.

Toegangscontrole: Het biedt een manier om toegang te krijgen tot netwerkdiensten die niet zijn blootgesteld aan het openbare internet, waardoor een extra laag van toegangscontrole wordt toegevoegd.

Veilig testen: Het is een waardevol hulpmiddel om netwerkservices veilig te testen. Zo kun je bijvoorbeeld een lokale server blootstellen aan het internet zonder dat deze voor iedereen toegankelijk is.


- When would you use local port forwarding?  

Het wordt gebruikt om een gebruiker vanaf de lokale computer verbinding te laten maken met een andere server, d.w.z. gegevens veilig door te sturen vanaf een andere clienttoepassing die op dezelfde computer draait als een Secure Shell (SSH) client.  

- When would you use remote port forwarding?  
  
wordt gebruikt wanneer een gebruiker externe toegang moet toestaan tot een service of applicatie die op zijn lokale machine wordt gehost, meestal achter een firewall of router, en deze toegankelijk moet maken voor een service of applicatie die op een externe server draait.  

- Which of the two are more "common" in security?  

Local port forwarding  

- Some people call SSH port forwarding also a "poor man's VPN". Why?  

omdat het je, net als een VPN, in staat stelt om veilig verbinding te maken met externe netwerken en diensten. In tegenstelling tot een VPN creëert SSH port forwarding echter geen netwerkinterface die al het verkeer door de tunnel stuurt, noch biedt het dezelfde mate van flexibiliteit en controle. Het is een eenvoudigere, meer lichtgewicht oplossing die voldoende is voor bepaalde gebruikssituaties zonder de overhead van een volledig VPN.


Example 1: use port forwarding to get to see the webpage from the webserver in the browser on the host (your laptop).
```bash
PS C:\Users\matte> ssh -L 8080:172.30.0.10:80 vagrant@companyrouter
Last login: Tue Dec 24 12:55:33 2024 from 192.168.62.165
[vagrant@companyrouter ~]$
```

![hierarchy](/images/localpf.png)

Example 2: use port forwarding to access the database from the host (your laptop).