# Lab 11: CA

## Certificate authority

- Does the CA uses a private key?

- Does the CA uses a certificate?

- Does the web server uses a private key?

- Does the web server uses a certificate?

- When using openssl commands to generate files, are you able to easily spot the function/goal of each file?

- How can you view a certificate using openssl?

- Does the webserver need a specific configuration change to allow HTTPS traffic?

- What is meant by a CSR?

- Tip: Do not forget the SAN (Subject Alternative Name) attribute!

- What is a wildcard certificate?

- Tip²: (For the optional part below you might want to support a wildcard certificate)

- What file(s) did you add to the browser (or computer) and how?

- Can you easily retrieve your certificates after adding them?

- Can you review and explain all the files from the OpenVPN lab and what they represent? CA? Keys? Certificates?


STAPPENPLAN:

1) Maak een root private key op ISPROUTER:
```
isprouter:~$ openssl genrsa -out rootCA.key 2048
```
2) Maak een root-certificaat op ISPROUTER:
```
isprouter:~$ openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 3650 -out rootCA.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:BE
State or Province Name (full name) [Some-State]:Oost-Vlaanderen
Locality Name (eg, city) []:Gent
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Hogent
Organizational Unit Name (eg, section) []:IT
Common Name (e.g. server FQDN or YOUR name) []:172.30.0.10
Email Address []:matteo.alexander@student.hogent.be
isprouter:~$ ls
ansible     rootCA.key  rootCA.pem
```
3) Genereer een private key voor je webserver:
```
[vagrant@web private]$ sudo openssl genrsa -out webserver.key 2048
```
4) Maak een CSR (Certificate Signing Request):
```
[vagrant@web private]$ sudo openssl req -new -key webserver.key -out webserver.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:BE
State or Province Name (full name) []:Oost-Vlaanderen
Locality Name (eg, city) [Default City]:Gent
Organization Name (eg, company) [Default Company Ltd]:Hogent
Organizational Unit Name (eg, section) []:IT
Common Name (eg, your name or your server's hostname) []:www.cybersec.internal
Email Address []:matteo.alexander@student.hogent.be

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:wachtwoord
An optional company name []:
[vagrant@web private]$ ls
webserver.csr  webserver.key
```
5) Kopieer de csr file inhoud en plak het in een nieuw .csr bestand op de isprouter
  
6) Onderteken de CSR met de CA (ISPROUTER):
```
isprouter:~$ openssl x509 -req -in webserver.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out webserver.crt -days 365 -sha256s 365 -sha256
Certificate request self-signature ok
subject=C = BE, ST = Oost-Vlaanderen, L = Gent, O = Hogent, OU = IT, CN = www.cybersec.internal, emailAddress = matteo.alexander@student.hogent.be
```
7) Kopieer het ondertekende .crt bestand naar de webserver
  
8) Pas de config file aan en verwijs naar de juiste paden waar de keys staan van het certificaat
```
[vagrant@web private]$ sudo nano /etc/httpd/conf.d/ssl.conf
```
Pas volgende zaken aan naar he juiste pad:
```
SSLCertificateFile /etc/pki/tls/private/webserver.crt
SSLCertificateKeyFile /etc/pki/tls/private/webserver.key
```

9) Herstart de httpd service
```
[vagrant@web private]$ sudo systemctl restart httpd
```

10) SCP de .PEM file van de CA (ISPROUTER) naar de kali machine of de machine waar je de website zal opzoeken
```
scp rootCA.pem osboxes@192.168.62.165:/home/osboxes/Downloads/
The authenticity of host '192.168.62.165 (192.168.62.165)' can't be established.
ED25519 key fingerprint is SHA256:OSC1KQRv5v/ILa1FbIgKFdl8H7dTEXUxPVTpABUqBag.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.62.165' (ED25519) to the list of known hosts.
osboxes@192.168.62.165's password:
rootCA.pem                                                                                                                100% 1489   781.7KB/s   00:00
```
11) Ga naar Firefox -> settings -> Privacy & Security zoek naar de Certificaten sectie en klik op 'view certificates'

12) Import de .PEM file die je net hebt overgezet.

13) Normaal zou nu alles in orde moeten zijn en kan je de site bereiken zonder ssl error: https://www.cybersec.internal/

![hierarchy](/images/https%20werkt.png)