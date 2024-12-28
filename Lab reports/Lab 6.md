# Lab 6: Hardening

## Ansible

Start of by creating a seperate SSH keypair specifically for ansible (we recommend to also create a seperate ansible user) and copy over the correct (public or private?) key to the remote locations (which file holds this key on the remote machine?). You can use the vagrant user the first time for creating the ansible user if you do not want to do this manually.

```bash
[vagrant@companyrouter ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/vagrant/.ssh/id_rsa
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:Lex/+09mUeW9VenVKuW/zfkHvNr/reiW50jt1SD7oD0 vagrant@companyrouter
The key's randomart image is:
+---[RSA 3072]----+
|                =|
|              .o*|
|             o..*|
|       . .  . o.+|
|        S . .o.+ |
|       . .   +o.+|
|        .   +..+X|
|         . +EBoB=|
|          o+BBB+X|
+----[SHA256]-----+
```

Zet de public key over naar de web en database vm:
```bash
ssh-copy-id -i /home/vagrant/.ssh/id_rsa.pub vagrant@172.30.0.10
```
```bash
ssh-copy-id -i /home/vagrant/.ssh/id_rsa.pub vagrant@172.30.0.15
```

Voer dit commando uit op companyrouter:
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub -p 2222 vagrant@172.30.255.254
```
  
Create an inventory file containing the machines in your environment. Below is an example inventory file. Feel free to extend this. The name between brackets is a group. The name below such brackets is the hostname (or IP). When using [name:vars] it is possible to define extra variables (or settings) for a specific group. Do you notice what protocol is used to authenticate with Windows in this case? What protocol is used for the other machines?

Inventory file: 
```bash
linux:
  hosts:
    web:
      ansible_host: 172.30.0.10
      ansible_ssh_user: vagrant
      ansible_ssh_private_key_file: /home/vagrant/.ssh/id_rsa
      ansible_ssh_public_key: /home/vagrant/.ssh/id_rsa.pub
    database:
      ansible_host: 172.30.0.15
      ansible_ssh_user: vagrant
      ansible_ssh_private_key_file: /home/vagrant/.ssh/id_rsa
      ansible_ssh_public_key: /home/vagrant/.ssh/id_rsa.pub
      ansible_python_interpreter: /usr/bin/python3.11
    companyrouter:
      ansible_host: 192.168.62.253
      ansible_ssh_user: vagrant
      ansible_ssh_private_key_file: /home/vagrant/.ssh/id_rsa
      ansible_ssh_public_key: /home/vagrant/.ssh/id_rsa.pub
      ansible_ssh_port: 2222
```
```bash
[vagrant@companyrouter ansible]$ ansible -i inventory.yml -m ping linux
database | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
web | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
companyrouter | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

- Run an ad-hoc ansible command to check if the date of all machines are configured the same. Are you able to use the same Windows module for Linux machines and vice versa?

```bash
[vagrant@companyrouter ansible]$ ansible -i inventory.yml -m shell -a "date" linux
database | CHANGED | rc=0 >>
Sat Dec 28 11:48:29 UTC 2024
web | CHANGED | rc=0 >>
Sat Dec 28 11:48:29 UTC 2024
companyrouter | CHANGED | rc=0 >>
Sat Dec 28 11:48:30 UTC 2024
```

- Create a playbook (or ad-hoc command) that pulls all /etc/passwd files from all Linux machines locally to the ansible controller node for every machine seperately.

get_passwd.yml:
```bash
---
- name: Retrieving /etc/passwd from all Linux machines and saving it locally
  hosts: linux
  tasks:
    - name: fetching the file for each machine
      fetch:
        src: /etc/passwd
        dest: "/home/vagrant/{{ inventory_hostname }}_etc_passwd"
        flat: yes
```
```bash
[vagrant@companyrouter ansible]$ ansible-playbook -i inventory.yml get_passwd.yml

PLAY [Retrieving /etc/passwd from all Linux machines and saving it locally] ********************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************
ok: [database]
ok: [web]
ok: [companyrouter]

TASK [fetching the file for each machine] ******************************************************************************************************************
changed: [web]
changed: [database]
changed: [companyrouter]

PLAY RECAP *************************************************************************************************************************************************
companyrouter              : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
database                   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web                        : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
```bash
[vagrant@companyrouter ansible]$ cd ~
[vagrant@companyrouter ~]$ ls
companyrouter_etc_passwd  database_etc_passwd  get-docker.sh  web_etc_passwd
[vagrant@companyrouter ~]$ sudo cat web_etc_passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
systemd-coredump:x:999:997:systemd Core Dumper:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
tss:x:59:59:Account used for TPM access:/dev/null:/sbin/nologin
sssd:x:998:995:User for sssd:/:/sbin/nologin
chrony:x:997:994:chrony system user:/var/lib/chrony:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/sbin/nologin
systemd-oom:x:992:992:systemd Userspace OOM Killer:/:/usr/sbin/nologin
vagrant:x:1000:1000::/home/vagrant:/bin/bash
vboxadd:x:991:1::/var/run/vboxadd:/bin/false
rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
geoclue:x:990:989:User for geoclue:/var/lib/geoclue:/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
```

- Create a playbook (or ad-hoc command) that creates the user "walt" with password Friday13th! on all Linux machines.
  
create_user.yml:
```bash
---
- name: Create user 'walt' with password on all Linux machines
  hosts: linux
  become: yes  # Use 'sudo' to perform privileged tasks
  tasks:
    - name: Create user 'walt'
      user:
        name: walt
        password: "{{ 'Friday13th!' | password_hash('sha512') }}"  # Hash the password
        state: present
```
```bash
[vagrant@companyrouter ansible]$ ansible-playbook -i inventory.yml creat_user.yml

PLAY [Create user 'walt' with password on all Linux machines] **********************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************
ok: [database]
ok: [web]
ok: [companyrouter]

TASK [Create user 'walt'] **********************************************************************************************************************************
[DEPRECATION WARNING]: Encryption using the Python crypt module is deprecated. The Python crypt module is deprecated and will be removed from Python 3.13.
Install the passlib library for continued encryption functionality. This feature will be removed in version 2.17. Deprecation warnings can be disabled by
setting deprecation_warnings=False in ansible.cfg.
[DEPRECATION WARNING]: Encryption using the Python crypt module is deprecated. The Python crypt module is deprecated and will be removed from Python 3.13.
Install the passlib library for continued encryption functionality. This feature will be removed in version 2.17. Deprecation warnings can be disabled by
setting deprecation_warnings=False in ansible.cfg.
[DEPRECATION WARNING]: Encryption using the Python crypt module is deprecated. The Python crypt module is deprecated and will be removed from Python 3.13.
Install the passlib library for continued encryption functionality. This feature will be removed in version 2.17. Deprecation warnings can be disabled by
setting deprecation_warnings=False in ansible.cfg.
changed: [database]
changed: [web]
changed: [companyrouter]

PLAY RECAP *************************************************************************************************************************************************
companyrouter              : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
database                   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web                        : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
```bash
[vagrant@companyrouter ansible]$ ansible web -i inventory.yml -m command -a "id walt"
web | CHANGED | rc=0 >>
uid=1001(walt) gid=1001(walt) groups=1001(walt)
```

- Create a playbook (or ad-hoc command) that pulls all users that are allowed to log in on all Linux machines.

```bash
[vagrant@companyrouter ansible]$ ansible -i inventory.yml -m getent -a 'database=passwd' linux
```

- Create a playbook (or ad-hoc command) that calculates the hash (md5sum for example) of a binary (for example the ss binary).

hash.yml:
```bash
---
- hosts: linux
  tasks:
    - name: Check if ss binary exists
      stat:
        path: /sbin/ss
      register: ss_stat

    - name: Calculate MD5 checksum of ss binary
      command: md5sum /sbin/ss
      when: ss_stat.stat.exists
      register: md5_result

    - name: Display MD5 checksum
      debug:
        var: md5_result.stdout_lines
      when: ss_stat.stat.exists

    - name: Inform if ss binary is missing
      debug:
        msg: "ss binary not found on this host"
      when: not ss_stat.stat.exists
```
```bash
[vagrant@companyrouter ansible]$ ansible-playbook -i inventory.yml hash.yml

PLAY [linux] ***********************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************
ok: [database]
ok: [web]
ok: [companyrouter]

TASK [Check if ss binary exists] ***************************************************************************************************************************
ok: [web]
ok: [database]
ok: [companyrouter]

TASK [Calculate MD5 checksum of ss binary] *****************************************************************************************************************
changed: [database]
changed: [web]
changed: [companyrouter]

TASK [Display MD5 checksum] ********************************************************************************************************************************
ok: [web] => {
    "md5_result.stdout_lines": [
        "e13b63ecba3a94e56fe72ad4128757d5  /sbin/ss"
    ]
}
ok: [database] => {
    "md5_result.stdout_lines": [
        "dfa062835f685a008b5402c4b795e97e  /sbin/ss"
    ]
}
ok: [companyrouter] => {
    "md5_result.stdout_lines": [
        "e13b63ecba3a94e56fe72ad4128757d5  /sbin/ss"
    ]
}

TASK [Inform if ss binary is missing] **********************************************************************************************************************
skipping: [web]
skipping: [database]
skipping: [companyrouter]

PLAY RECAP *************************************************************************************************************************************************
companyrouter              : ok=4    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
database                   : ok=4    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
web                        : ok=4    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

- Create a playbook (or ad-hoc command) that copies a file (for example a txt file) from the ansible controller machine to all Linux machines.

copy.yml:
```bash
---
- hosts: linux
  become: yes
  tasks:
    - name: Copy a file to all Linux machines
      copy:
        src: ./TEST.txt
        dest: /home/vagrant/
```

```bash
[vagrant@companyrouter ansible]$ ansible-playbook -i inventory.yml copy.yml

PLAY [linux] ***********************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************
ok: [database]
ok: [web]
ok: [companyrouter]

TASK [Copy a file to all Linux machines] *******************************************************************************************************************
changed: [database]
changed: [web]
changed: [companyrouter]

PLAY RECAP *************************************************************************************************************************************************
companyrouter              : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
database                   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web                        : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
```bash
database:~$ ls
TEST.txt
database:~$ cat TEST.txt
MatteoTheHacker
```