# Lab 9: VPN

## OpenVPN - practical installation

- Installing the server software can be done using dnf:
```
[vagrant@companyrouter ~]$ sudo dnf install --assumeyes openvpn easy-rsa
```

- Make sure you have at least OpenVPN v2.5.9 and EasyRSA v3.1.6 installed:

```
[vagrant@companyrouter ~]$ openvpn --version
OpenVPN 2.5.11 x86_64-redhat-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] built on Jul 18 2024
library versions: OpenSSL 3.0.7 1 Nov 2022, LZO 2.10
Originally developed by James Yonan
Copyright (C) 2002-2022 OpenVPN Inc <sales@openvpn.net>
Compile time defines: enable_async_push=yes enable_comp_stub=no enable_crypto_ofb_cfb=yes enable_debug=yes enable_def_auth=yes enable_dependency_tracking=no enable_dlopen=unknown enable_dlopen_self=unknown enable_dlopen_self_static=unknown enable_fast_install=yes enable_fragment=yes enable_iproute2=no enable_libtool_lock=yes enable_lz4=yes enable_lzo=yes enable_management=yes enable_multihome=yes enable_pam_dlopen=no enable_pedantic=no enable_pf=yes enable_pkcs11=yes enable_plugin_auth_pam=yes enable_plugin_down_root=yes enable_plugins=yes enable_port_share=yes enable_selinux=yes enable_shared=yes enable_shared_with_static_runtimes=no enable_silent_rules=yes enable_small=no enable_static=yes enable_strict=no enable_strict_options=no enable_systemd=yes enable_werror=no enable_win32_dll=yes enable_x509_alt_username=yes with_aix_soname=aix with_crypto_library=openssl with_gnu_ld=yes with_mem_check=no with_openssl_engine=auto with_sysroot=no
```
```
[vagrant@companyrouter ~]$ sudo /usr/share/easy-rsa/3/easyrsa --version
EasyRSA Version Information
Version:     3.1.6
Generated:   Fri Aug 18 09:28:23 CDT 2023
SSL Lib:     OpenSSL 3.0.7 1 Nov 2022 (Library: OpenSSL 3.0.7 1 Nov 2022)
Git Commit:  9850ced8bec5e0a065d9c576f59c3f372f82f4a9
Source Repo: https://github.com/OpenVPN/easy-rsa
```

## Set up the PKI

- Maak een nieuwe PKI

```
[vagrant@companyrouter ~]$ export PATH=$PATH:/usr/share/easy-rsa/3.1.6
[vagrant@companyrouter ~]$ echo $PATH
/home/vagrant/.local/bin:/home/vagrant/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/usr/share/easy-rsa/3.1.6
[vagrant@companyrouter ~]$ echo 'export PATH=$PATH:/usr/share/easy-rsa/3.1.6' >> ~/.bashrc
[vagrant@companyrouter ~]$  source ~/.bashrc
[vagrant@companyrouter ~]$ easyrsa init-pki

Notice
------
'init-pki' complete; you may now create a CA or requests.

Your newly created PKI dir is:
* /home/vagrant/pki

Using Easy-RSA configuration:
* /home/vagrant/pki/vars

IMPORTANT:
  Easy-RSA 'vars' template file has been created in your new PKI.
  Edit this 'vars' file to customise the settings for your PKI.
  To use a global vars file, use global option --vars=<FILE>
```

- Build the CA:
```
[vagrant@companyrouter ~]$ easyrsa build-ca
Using Easy-RSA 'vars' configuration:
* /home/vagrant/pki/vars

Using SSL:
* openssl OpenSSL 3.0.7 1 Nov 2022 (Library: OpenSSL 3.0.7 1 Nov 2022)

Enter New CA Key Passphrase:

Confirm New CA Key Passphrase:
.+...+......+.+.........+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*........+......+......+..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..........+.....+.+...........+...+....+...+...............+.....+....+.....+...................+..+....+...+............+......+..+......+.+...+..+...+.............+...+..+...+....+..+.+...+.........+..+......+....+......+.....+.......+.................+..........+...............+...........+...................+.....+.........+....+.....+......+....+...........+.........+...+.........+.+.....+.......+..+.........+.......+.........+.....+...+.+.....+..........+......+........+.+...........+....+......+...+..+......+....+.....+......+...+.......+..................+.....+.......+...............+........+......+.+.........+..............+.+.....+.+.....+.......+..+...+...+.......+......+.....+.......+..............+..................+.........................+..+.+...+..+...+....+.....+.+.........+...............+...........+..........+...........+.+...+.....+.+........+......+....+........+..........+..+...............+.......+......+..+.....................+.......+...+..+.......+...+........+....+......+...........+...+...+....+........................+..+...+.+.....+.+..+.+......+.....+.+..+...+..........+..+......+...............+......+....+......+.........+.........+......+.....+.+...+..+.........+.+..................+..+...................+..........................+..........+...+.....+.+.........+..+...+............+....+.........+...+........+.......+...+.........+......+...........+...+...+....+........+...+....+.........+.................+...+.......+..+...+...+.......+........+......+.............+..+..........+........+.............+...+.....+.+.................................+......+..+...............+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
...+...+.......+........+.......+........+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*...+..+...+....+........+....+......+...+.....+.+........+.+..+.......+..+......+...+.......+...+.....+......+.+............+.....+.......+......+............+..+.+..+......+.+.........+.....+......+.+..+.......+...+..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+..........+.........+....................+.+..+...+.......+...............+..+.+......+......+...+.........+..+...+...+...+....+........+..........+............+.....+......+....+.....+...+.........+...+......+............+....+...+..+.........+....+...............+........+....+........+.+......+........+...+...+.+...............+......+.........+.....+.+........+.......+...+.....+....+.....+...+....+...+..+....+..+....+...+...+........+....+...+...+.....+.+.....+.+..+......+.+.....+.+..+.....................+....+......................................+....+...+...+.........+.....+....+..+....+...........+...+....+........+.........+...+.......+........+.............+..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:companyrouter

Notice
------
CA creation complete. Your new CA certificate is at:
* /home/vagrant/pki/ca.crt
```

## Generate the server keys and certificate
```
[vagrant@companyrouter ~]$ easyrsa gen-req server
Using Easy-RSA 'vars' configuration:
* /home/vagrant/pki/vars

Using SSL:
* openssl OpenSSL 3.0.7 1 Nov 2022 (Library: OpenSSL 3.0.7 1 Nov 2022)
...+...+..+...+...+......+....+.....+....+........+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*...+.......+...+......+..+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+...+....................+....+...+.....+....+........+.......+.....+...+.+.................+....+...+..+....+..+.........+...+................+.........+........+.........+....+.....+......+.+........+....+..+...+.......+...+........+...+.........+..........+..+...+.+..+....+...+......+..+....+.....+....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
.........+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.....+..........+..+.......+...+..+.......+.....+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+...+...........+.+...+..+...+.+...+...............+...+.....+...+....+...+..+.......+..................+.....+.+..+...+....+..+....+.....+.......+..+.+..+............+............+....+..............+............+....+..+...+.+.........+......+.....+...+.+.....+.......+............+.....+...+.......+.................+.+.....+.......+......+..+...+.........+.+...........+...............+....+......+......+..+......+.+...+........+.......+..+.+...+......+........+...+....+...........+...+.+...+..+...+.+......+.....+.+...+..+....+...+..+...+...+.........+...................+...........+......+....+...+.....................+.....+.......+...+.....+....+...+..+...+.......+...............+...+..+.......+.....+..................+.............+........+.+........+......+......+.+..............+.........+....+......+..+.........+...+.+..+...+....+...+.....+..........+...+..............+......+.......+.........+.........+..+...+.........+.+...........+...+...+..........+.....+..................+.+..............+....+..+...+......+....+..+......+.............+..+.......+.....+.+..+.......+...........+.+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [server]:companyrouter

Notice
------
Private-Key and Public-Certificate-Request files created.
Your files are:
* req: /home/vagrant/pki/reqs/server.req
* key: /home/vagrant/pki/private/server.key
```

PEM PASSPHRASE: wachtwoord

- Signeer de server request en genereer het certificaat

```
[vagrant@companyrouter ~]$ easyrsa sign-req server server
Using Easy-RSA 'vars' configuration:
* /home/vagrant/pki/vars

Using SSL:
* openssl OpenSSL 3.0.7 1 Nov 2022 (Library: OpenSSL 3.0.7 1 Nov 2022)
You are about to sign the following certificate:
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.
Request subject, to be signed as a server certificate
for '825' days:

subject=
    commonName                = companyrouter

Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes

Using configuration from /home/vagrant/pki/openssl-easyrsa.cnf
Enter pass phrase for /home/vagrant/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'companyrouter'
Certificate is to be certified until Apr  4 15:30:21 2027 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

Notice
------
Certificate created at:
* /home/vagrant/pki/issued/server.crt
```

- Verifieer het certificaat:
```
[vagrant@companyrouter ~]$ sudo openssl verify -CAfile /home/vagrant/pki/ca.crt  /home/vagrant/pki/issued/server.crt
/home/vagrant/pki/issued/server.crt: OK
```

## Generate client keys and certificate

```
[vagrant@companyrouter ~]$ easyrsa gen-req client nopass
Using Easy-RSA 'vars' configuration:
* /home/vagrant/pki/vars

Using SSL:
* openssl OpenSSL 3.0.7 1 Nov 2022 (Library: OpenSSL 3.0.7 1 Nov 2022)
.....+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.....+......+...+..+..................+...+...+.......+..+......+......+.+..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*....+.......+..+.+..............+....+..+...+....+....................+.+...........+..........+......+..+..........+...+.....+...+......+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
.....+......+..+...+....+...........+..........+..+.+..+.......+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*....+...+....+.....+....+.........+..+...+..........+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.......+.+...........+.+.....+.......+..+....+.........+......+......+.....+....+.....+.......+.....+....+...+...+.........+........+.+..+...+.+......+.........+...+..+.+..............+...+.+......+........+...+...................+.....+...+......+.+...............+.....+......+.......+..+......+...+.........+.......+.....+......+...+.+...+............+...+.....+.+.....+......+.+.........+.....+.+.....+....+.........+............+...+..+.........+......+.......+............+...+...........+............+...+.......+........+......+.........+......+.+..+............+....+......+...............+.....+...+............+.+.........+...........+.......+........+.........+..................+......+....+.....+.+..+...+....+..+....+.....+...............+.........+.......+...+........+.........+.+............+.....+.+..+...+.........+...+.........+.+.........+.....+.........+.+...+............+...+..+.........+............+.........+......+.......+...+...+...........+..........+......+..+.+......+......+............+..+....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [client]:employee

Notice
------
Private-Key and Public-Certificate-Request files created.
Your files are:
* req: /home/vagrant/pki/reqs/client.req
* key: /home/vagrant/pki/private/client.key
```

- Signeren:

```
[vagrant@companyrouter ~]$ easyrsa sign-req client client
Using Easy-RSA 'vars' configuration:
* /home/vagrant/pki/vars

Using SSL:
* openssl OpenSSL 3.0.7 1 Nov 2022 (Library: OpenSSL 3.0.7 1 Nov 2022)
You are about to sign the following certificate:
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.
Request subject, to be signed as a client certificate
for '825' days:

subject=
    commonName                = employee

Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes

Using configuration from /home/vagrant/pki/openssl-easyrsa.cnf
Enter pass phrase for /home/vagrant/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'employee'
Certificate is to be certified until Apr  4 15:35:02 2027 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

Notice
------
Certificate created at:
* /home/vagrant/pki/issued/client.crt
```

- Verifiëren:

```
[vagrant@companyrouter ~]$ sudo openssl verify -CAfile /home/vagrant/pki/ca.crt  /home/vagrant/pki/issued/client.crt
/home/vagrant/pki/issued/client.crt: OK
```

## Generate Diffie Hellman parameters

```
[vagrant@companyrouter ~]$ easyrsa gen-dh
Using Easy-RSA 'vars' configuration:
* /home/vagrant/pki/vars

Using SSL:
* openssl OpenSSL 3.0.7 1 Nov 2022 (Library: OpenSSL 3.0.7 1 Nov 2022)
Generating DH parameters, 2048 bit long safe prime
.......+.............................................................................................................................................................................................................................................................................................................+........................+.................+...............+...........+............................................................................................................................+.........................................................................................................................+..............+..........................................+..........................................................................................................................................................................................................................................+........................................................................................................................................................................................................................................................................+.........................................................................................................................................................+.....+...+............................+.................................................................................................+.........................................................+............................+.......................................................+..............................................................................................................................................................................................................................+..........................................................................................+...............................+.........................................+..............................+...................................................+....................................+..........................................+..................................................................................................................++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*
DH parameters appear to be ok.

Notice
------

DH parameters of size 2048 created at:
* /home/vagrant/pki/dh.pem
```

- If everything has gone well, you should see the following files:

```
[vagrant@companyrouter ~]$ sudo ls -a -l /home/vagrant/pki/ca.crt
-rw-------. 1 vagrant vagrant 1212 Dec 30 15:24 /home/vagrant/pki/ca.crt
[vagrant@companyrouter ~]$ sudo ls -a -l /home/vagrant/pki/dh.pem
-rw-------. 1 vagrant vagrant 424 Dec 30 15:39 /home/vagrant/pki/dh.pem
[vagrant@companyrouter ~]$ sudo ls -a -l /home/vagrant/pki/issued
total 20
drwx------. 2 vagrant vagrant   42 Dec 30 15:35 .
drwx------. 8 vagrant vagrant 4096 Dec 30 15:39 ..
-rw-------. 1 vagrant vagrant 4507 Dec 30 15:35 client.crt
-rw-------. 1 vagrant vagrant 4651 Dec 30 15:30 server.crt
[vagrant@companyrouter ~]$ sudo ls -a -l /home/vagrant/pki/private
total 16
drwx------. 2 vagrant vagrant   56 Dec 30 15:34 .
drwx------. 8 vagrant vagrant 4096 Dec 30 15:39 ..
-rw-------. 1 vagrant vagrant 1874 Dec 30 15:24 ca.key
-rw-------. 1 vagrant vagrant 1704 Dec 30 15:34 client.key
-rw-------. 1 vagrant vagrant 1874 Dec 30 15:27 server.key
[vagrant@companyrouter ~]$ sudo ls -a -l /home/vagrant/pki/reqs
total 12
drwx------. 2 vagrant vagrant   42 Dec 30 15:34 .
drwx------. 8 vagrant vagrant 4096 Dec 30 15:39 ..
-rw-------. 1 vagrant vagrant  891 Dec 30 15:34 client.req
-rw-------. 1 vagrant vagrant  895 Dec 30 15:28 server.req
```

## Configure the server

- You can find sample configuration files on your own system:

```
[vagrant@companyrouter ~]$ ls -la /usr/share/doc/openvpn/sample/sample-config-files/
total 84
drwxr-xr-x. 2 root root  4096 Dec 30 15:12 .
drwxr-xr-x. 5 root root    77 Dec 30 15:12 ..
-rw-r--r--. 1 root root   131 Jul 18 12:46 README
-rw-r--r--. 1 root root  3589 Jul 18 12:46 client.conf
-rw-r--r--. 1 root root  3562 Jul 18 12:46 firewall.sh
-rw-r--r--. 1 root root    62 Jul 18 12:46 home.up
-rw-r--r--. 1 root root 11386 Jul 18 12:46 loopback-client
-rw-r--r--. 1 root root   694 Jul 18 12:46 loopback-server
-rw-r--r--. 1 root root    62 Jul 18 12:46 office.up
-rw-r--r--. 1 root root    63 Jul 18 12:46 openvpn-shutdown.sh
-rw-r--r--. 1 root root   776 Jul 18 12:46 openvpn-startup.sh
-rw-r--r--. 1 root root   820 Jul 18 14:36 roadwarrior-client.conf
-rw-r--r--. 1 root root  1498 Jul 18 14:36 roadwarrior-server.conf
-rw-r--r--. 1 root root 10784 Jul 18 12:46 server.conf
-rw-r--r--. 1 root root  1990 Jul 18 12:46 tls-home.conf
-rw-r--r--. 1 root root  2019 Jul 18 12:46 tls-office.conf
-rw-r--r--. 1 root root   199 Jul 18 12:46 xinetd-client-config
-rw-r--r--. 1 root root   989 Jul 18 12:46 xinetd-server-config
```


SERVER CONFIG:
```
[vagrant@companyrouter sample-config-files]$ sudo nano server.conf
#################################################
# Sample OpenVPN 2.0 config file for            #
# multi-client server.                          #
#                                               #
# This file is for the server side              #
# of a many-clients <-> one-server              #
# OpenVPN configuration.                        #
#                                               #
# OpenVPN also supports                         #
# single-machine <-> single-machine             #
# configurations (See the Examples page         #
# on the web site for more info).               #
#                                               #
# This config should work on Windows            #
# or Linux/BSD systems.  Remember on            #
# Windows to quote pathnames and use            #
# double backslashes, e.g.:                     #
# "C:\\Program Files\\OpenVPN\\config\\foo.key" #
#                                               #
# Comments are preceded with '#' or ';'         #
#################################################

# Which local IP address should OpenVPN
# listen on? (optional)
local 192.168.62.253

# Which TCP/UDP port should OpenVPN listen on?
# If you want to run multiple OpenVPN instances
# on the same machine, use a different port
# number for each one.  You will need to
# open up this port on your firewall.
port 1194

# TCP or UDP server?
;proto tcp
proto udp

# "dev tun" will create a routed IP tunnel,
# "dev tap" will create an ethernet tunnel.
# Use "dev tap0" if you are ethernet bridging
# and have precreated a tap0 virtual interface
# and bridged it with your ethernet interface.
# If you want to control access policies
# over the VPN, you must create firewall
# rules for the the TUN/TAP interface.
# On non-Windows systems, you can give
# an explicit unit number, such as tun0.
# On Windows, use "dev-node" for this.
# On most systems, the VPN will not function
# unless you partially or fully disable
# the firewall for the TUN/TAP interface.
;dev tap
dev tun

# Windows needs the TAP-Win32 adapter name
# from the Network Connections panel if you
# have more than one.  On XP SP2 or higher,
# you may need to selectively disable the
# Windows firewall for the TAP adapter.
# Non-Windows systems usually don't need this.
;dev-node MyTap

# SSL/TLS root certificate (ca), certificate
# (cert), and private key (key).  Each client
# and the server must have their own cert and
# key file.  The server and all clients will
# use the same ca file.
#
# See the "easy-rsa" directory for a series
# of scripts for generating RSA certificates
# and private keys.  Remember to use
# a unique Common Name for the server
# and each of the client certificates.
#
# Any X509 key management system can be used.
# OpenVPN can also use a PKCS #12 formatted key file
# (see "pkcs12" directive in man page).
ca /home/vagrant/pki/ca.crt
cert /home/vagrant/pki/issued/server.crt
key /home/vagrant/pki/private/server.key  # This file should be kept secret

# Diffie hellman parameters.
# Generate your own with:
#   openssl dhparam -out dh2048.pem 2048
dh /home/vagrant/pki/dh.pem

# Network topology
# Should be subnet (addressing via IP)
# unless Windows clients v2.0.9 and lower have to
# be supported (then net30, i.e. a /30 per client)
# Defaults to net30 (not recommended)
;topology subnet

# Configure server mode and supply a VPN subnet
# for OpenVPN to draw client addresses from.
# The server will take 10.8.0.1 for itself,
# the rest will be made available to clients.
# Each client will be able to reach the server
# on 10.8.0.1. Comment this line out if you are
# ethernet bridging. See the man page for more info.
server 10.8.0.0 255.255.255.0

# Maintain a record of client <-> virtual IP address
# associations in this file.  If OpenVPN goes down or
# is restarted, reconnecting clients can be assigned
# the same virtual IP address from the pool that was
# previously assigned.
ifconfig-pool-persist ipp.txt

# Configure server mode for ethernet bridging.
# You must first use your OS's bridging capability
# to bridge the TAP interface with the ethernet
# NIC interface.  Then you must manually set the
# IP/netmask on the bridge interface, here we
# assume 10.8.0.4/255.255.255.0.  Finally we
# must set aside an IP range in this subnet
# (start=10.8.0.50 end=10.8.0.100) to allocate
# to connecting clients.  Leave this line commented
# out unless you are ethernet bridging.
;server-bridge 10.8.0.4 255.255.255.0 10.8.0.50 10.8.0.100

# Configure server mode for ethernet bridging
# using a DHCP-proxy, where clients talk
# to the OpenVPN server-side DHCP server
# to receive their IP address allocation
# and DNS server addresses.  You must first use
# your OS's bridging capability to bridge the TAP
# interface with the ethernet NIC interface.
# Note: this mode only works on clients (such as
# Windows), where the client-side TAP adapter is
# bound to a DHCP client.
;server-bridge

# Push routes to the client to allow it
# to reach other private subnets behind
# the server.  Remember that these
# private subnets will also need
# to know to route the OpenVPN client
# address pool (10.8.0.0/255.255.255.0)
# back to the OpenVPN server.
push "route 172.30.0.0 255.255.0.0"
push "route 172.10.10.0 255.255.255.0"
push "route 192.168.62.0 255.255.255.0"
# To assign specific IP addresses to specific
# clients or if a connecting client has a private
# subnet behind it that should also have VPN access,
# use the subdirectory "ccd" for client-specific
# configuration files (see man page for more info).

# EXAMPLE: Suppose the client
# having the certificate common name "Thelonious"
# also has a small subnet behind his connecting
# machine, such as 192.168.40.128/255.255.255.248.
# First, uncomment out these lines:
;client-config-dir ccd
;route 192.168.40.128 255.255.255.248
# Then create a file ccd/Thelonious with this line:
#   iroute 192.168.40.128 255.255.255.248
# This will allow Thelonious' private subnet to
# access the VPN.  This example will only work
# if you are routing, not bridging, i.e. you are
# using "dev tun" and "server" directives.

# EXAMPLE: Suppose you want to give
# Thelonious a fixed VPN IP address of 10.9.0.1.
# First uncomment out these lines:
;client-config-dir ccd
;route 10.9.0.0 255.255.255.252
# Then add this line to ccd/Thelonious:
#   ifconfig-push 10.9.0.1 10.9.0.2

# Suppose that you want to enable different
# firewall access policies for different groups
# of clients.  There are two methods:
# (1) Run multiple OpenVPN daemons, one for each
#     group, and firewall the TUN/TAP interface
#     for each group/daemon appropriately.
# (2) (Advanced) Create a script to dynamically
#     modify the firewall in response to access
#     from different clients.  See man
#     page for more info on learn-address script.
;learn-address ./script

# If enabled, this directive will configure
# all clients to redirect their default
# network gateway through the VPN, causing
# all IP traffic such as web browsing and
# and DNS lookups to go through the VPN
# (The OpenVPN server machine may need to NAT
# or bridge the TUN/TAP interface to the internet
# in order for this to work properly).
;push "redirect-gateway def1 bypass-dhcp"

# Certain Windows-specific network settings
# can be pushed to clients, such as DNS
# or WINS server addresses.  CAVEAT:
# http://openvpn.net/faq.html#dhcpcaveats
# The addresses below refer to the public
# DNS servers provided by opendns.com.
;push "dhcp-option DNS 208.67.222.222"
;push "dhcp-option DNS 208.67.220.220"

# Uncomment this directive to allow different
# clients to be able to "see" each other.
# By default, clients will only see the server.
# To force clients to only see the server, you
# will also need to appropriately firewall the
# server's TUN/TAP interface.
;client-to-client

# Uncomment this directive if multiple clients
# might connect with the same certificate/key
# files or common names.  This is recommended
# only for testing purposes.  For production use,
# each client should have its own certificate/key
# pair.
#
# IF YOU HAVE NOT GENERATED INDIVIDUAL
# CERTIFICATE/KEY PAIRS FOR EACH CLIENT,
# EACH HAVING ITS OWN UNIQUE "COMMON NAME",
# UNCOMMENT THIS LINE OUT.
;duplicate-cn

# The keepalive directive causes ping-like
# messages to be sent back and forth over
# the link so that each side knows when
# the other side has gone down.
# Ping every 10 seconds, assume that remote
# peer is down if no ping received during
# a 120 second time period.
keepalive 10 120

# For extra security beyond that provided
# by SSL/TLS, create an "HMAC firewall"
# to help block DoS attacks and UDP port flooding.
#
# Generate with:
#   openvpn --genkey tls-auth ta.key
#
# The server and each client must have
# a copy of this key.
# The second parameter should be '0'
# on the server and '1' on the clients.
#tls-auth ta.key 0 # This file is secret

# Select a cryptographic cipher.
# This config item must be copied to
# the client config file as well.
# Note that v2.4 client/server will automatically
# negotiate AES-256-GCM in TLS mode.
# See also the ncp-cipher option in the manpage
cipher AES-256-CBC

# Enable compression on the VPN link and push the
# option to the client (v2.4+ only, for earlier
# versions see below)
;compress lz4-v2
;push "compress lz4-v2"

# For compression compatible with older clients use comp-lzo
# If you enable it here, you must also
# enable it in the client config file.
;comp-lzo

# The maximum number of concurrently connected
# clients we want to allow.
;max-clients 100

# It's a good idea to reduce the OpenVPN
# daemon's privileges after initialization.
#
# You can uncomment this out on
# non-Windows systems.
;user nobody
;group nobody

# The persist options will try to avoid
# accessing certain resources on restart
# that may no longer be accessible because
# of the privilege downgrade.
persist-key
persist-tun

# Output a short status file showing
# current connections, truncated
# and rewritten every minute.
status /var/log/openvpn-status.log

# By default, log messages will go to the syslog (or
# on Windows, if running as a service, they will go to
# the "\Program Files\OpenVPN\log" directory).
# Use log or log-append to override this default.
# "log" will truncate the log file on OpenVPN startup,
# while "log-append" will append to it.  Use one
# or the other (but not both).
;log         openvpn.log
;log-append  openvpn.log

# Set the appropriate level of log
# file verbosity.
#
# 0 is silent, except for fatal errors
# 4 is reasonable for general usage
# 5 and 6 can help to debug connection problems
# 9 is extremely verbose
verb 3

# Silence repeating messages.  At most 20
# sequential messages of the same message
# category will be output to the log.
;mute 20

# Notify the client that when the server restarts so it
# can automatically reconnect.
explicit-exit-notify 1
```

## Configure the client

- Make sure the client can ping companyrouter and isprouter, and has a working internet connection.
```
[vagrant@remote-employee ~]$ ping 192.168.62.254
PING 192.168.62.254 (192.168.62.254) 56(84) bytes of data.
64 bytes from 192.168.62.254: icmp_seq=1 ttl=63 time=0.359 ms
64 bytes from 192.168.62.254: icmp_seq=2 ttl=63 time=0.448 ms
64 bytes from 192.168.62.254: icmp_seq=3 ttl=63 time=0.424 ms
64 bytes from 192.168.62.254: icmp_seq=4 ttl=63 time=0.449 ms
^C
--- 192.168.62.254 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3091ms
rtt min/avg/max/mdev = 0.359/0.420/0.449/0.036 ms
[vagrant@remote-employee ~]$ ping 192.168.62.253
PING 192.168.62.253 (192.168.62.253) 56(84) bytes of data.
64 bytes from 192.168.62.253: icmp_seq=1 ttl=62 time=4.70 ms
64 bytes from 192.168.62.253: icmp_seq=2 ttl=62 time=3.22 ms
64 bytes from 192.168.62.253: icmp_seq=3 ttl=62 time=1.20 ms
64 bytes from 192.168.62.253: icmp_seq=4 ttl=62 time=7.32 ms
^C
--- 192.168.62.253 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 1.201/4.110/7.322/2.231 ms
[vagrant@remote-employee ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=27.1 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=18.8 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=17.5 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=113 time=15.6 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 15.630/19.769/27.121/4.395 ms
```

- Install OpenVPN client software on the client.
```
[vagrant@remote-employee ~]$ sudo yum install -y epel-release
```
```
[vagrant@remote-employee ~]$ sudo yum install -y openvpn
```

- Kopieer Certificaten en Sleutels naar de Client

```
[vagrant@remote-employee ~]$ sudo scp vagrant@192.168.62.253:/home/vagrant/pki/ca.crt /etc/openvpn/
```

```
[vagrant@remote-employee ~]$ sudo scp vagrant@192.168.62.253:/home/vagrant/pki/issued/client.crt /etc/openvpn/
```

```
[vagrant@remote-employee ~]$ sudo scp vagrant@192.168.62.253:/home/vagrant/pki/private/client.key /etc/openvpn/
```

- Controleer de certificaat bestanden:
```
[vagrant@remote-employee ~]$ ls -l /etc/openvpn/
total 20
-rw-------. 1 root root    1212 Dec 31 12:43 ca.crt
drwxr-x---. 2 root openvpn    6 Jul 18 14:36 client
-rw-r--r--. 1 root root    3687 Dec 31 12:42 client.conf
-rw-------. 1 root root    4507 Dec 31 12:43 client.crt
-rw-------. 1 root root    1704 Dec 31 12:44 client.key
drwxr-x---. 2 root openvpn    6 Jul 18 14:36 server
```