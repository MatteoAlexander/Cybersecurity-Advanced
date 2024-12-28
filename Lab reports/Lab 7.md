# Lab 7: Backups

## BorgBackup


- Create a folder on the web VM and store the files (e.g. ~/important-files). Use curl with the --location and --remote-name-all options. What do these options do? Why do you need them? Do you really need them? What happens without them?

```bash
[vagrant@web ~]$ mkdir important-files
[vagrant@web ~]$ cd !$
cd important-files
[vagrant@web important-files]$ curl --remote-name-all https://video.blender.org/download/videos/bf1f3fb5-b119-4f9f-9930-8e20e892b898-720.mp4 https://www.gutenberg.org/ebooks/100.txt.utf-8 https://www.gutenberg.org/ebooks/996.txt.utf-8 https://upload.wikimedia.org/wikipedia/commons/4/40/Toreador_song_cleaned.ogg
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 36196  100 36196    0     0   178k      0 --:--:-- --:--:-- --:--:--  179k
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   300  100   300    0     0    447      0 --:--:-- --:--:-- --:--:--   447
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   300  100   300    0     0   2500      0 --:--:-- --:--:-- --:--:--  2500
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 1662k  100 1662k    0     0  1562k      0  0:00:01  0:00:01 --:--:-- 1562k
[vagrant@web important-files]$ mv 100.txt.utf-8 100.txt
[vagrant@web important-files]$ mv 996.txt.utf-8 996.txt
[vagrant@web important-files]$ ll
total 1708
-rw-r--r--. 1 vagrant vagrant     300 Dec 28 12:17 100.txt
-rw-r--r--. 1 vagrant vagrant     300 Dec 28 12:17 996.txt
-rw-r--r--. 1 vagrant vagrant 1702187 Dec 28 12:17 Toreador_song_cleaned.ogg
-rw-r--r--. 1 vagrant vagrant   36196 Dec 28 12:17 bf1f3fb5-b119-4f9f-9930-8e20e892b898-720.mp4
```

Je gebruikt --location als je verwacht dat de URL die je opvraagt, kan omleiden naar een andere URL. Dit is gebruikelijk op het web, vooral met download links.

--remote-name-all is handig als je meerdere bestanden downloadt en je wilt dat curl de originele bestandsnamen behoudt zonder dat je ze handmatig hoeft op te geven.

Zonder --location, als de URL omleidt, zal curl de omleiding niet volgen en krijg je mogelijk niet het gewenste bestand.
Zonder --remote-name-all, moet je handmatig een bestandsnaam specificeren voor elke download, wat onpraktisch kan zijn bij het downloaden van meerdere bestanden.

- Create a folder on the db VM to store the backups (e.g. ~/backups).
  
```bash
database:~$ mkdir backups
database:~$ cd backups/
database:~/backups$
```

- Install borg on both the machine were the files are used, and the machine were the backups will be stored. As borg is only available on linux machines 1 and we don't want to introduce another VM to burden your laptop further, we will store the active versions of the files on web and the backup db VM. It is important that both machines have borg installed.

There are always multiple ways to install software on a machine, such as pip, npm, a downloadable executable, flatpak, ... . The best way is always through the distro's package manager, which you always should try first:

WEB:
```bash
[vagrant@web important-files]$ sudo dnf search epel-release
```
```bash
[vagrant@web important-files]$ sudo dnf install borgbackup
```
  
  
DATABASE:
```bash
database:~/backups$ sudo apk add alpine-sdk
```
```bash
database:~/backups$ sudo apk add borgbackup
```

- From the webserver, initialize a backup repository on db. This repository will contain all the created backups. Make sure you use therepokey` encryption mode!
```bash
[vagrant@web important-files]$ ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/vagrant/.ssh/id_rsa
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:kBKncoQRVfN8OzrY3/anZmS2aQngBX/PZufleNnoVyc vagrant@web
The key's randomart image is:
+---[RSA 4096]----+
|  o=+.+          |
|  .. + =  .      |
|  . + o o .o     |
|   o . . ...o .  |
|        S.oo . o |
|       o .... E X|
|      . +    = %B|
|         o .. X.B|
|          ...*+= |
+----[SHA256]-----+
[vagrant@web important-files]$ ssh-copy-id vagrant@172.30.0.15
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
The authenticity of host '172.30.0.15 (172.30.0.15)' can't be established.
ED25519 key fingerprint is SHA256:so8Nwt0+y0MowuzwQMdK+oMZPUbaPrXk/wRBOzgJtHE.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@172.30.0.15's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@172.30.0.15'"
and check to make sure that only the key(s) you wanted were added.

[vagrant@web important-files]$ (umask 0077; head -c 32 /dev/urandom | base64 -w 0 > ~/.borg-passphrase)
[vagrant@web important-files]$ export BORG_PASSCOMMAND="cat $HOME/.borg-passphrase"
[vagrant@web important-files]$ source ~/.bashrc
[vagrant@web important-files]$ borg init --encryption=repokey vagrant@172.30.0.15:~/backups

By default repositories initialized with this version will produce security
errors if written to with an older version (up to and including Borg 1.0.8).

If you want to use these older versions, you can disable the check by running:
borg upgrade --disable-tam 'ssh://vagrant@172.30.0.15/~/backups'

See https://borgbackup.readthedocs.io/en/stable/changes.html#pre-1-0-9-manifest-spoofing-vulnerability for details about the security implications.

IMPORTANT: you will need both KEY AND PASSPHRASE to access this repo!
If you used a repokey mode, the key is stored in the repo, but you should back it up separately.
Use "borg key export" to export the key, optionally in printable format.
Write down the passphrase. Store both at safe place(s).
```

Export the borg keyfile in a readable format so you can store it on a safe location. The keyfile was generated automatically when you initialized the backup repository and stored on the web server (because that's from where you initialized the backup repository). You need both the borg key and the borg keyfile to access the backups. If you lose the borg keyfile (e.g. the SSD/HDD on web has been damaged or wiped), you won't be able to access the backups anymore. You best keep a backup of the keyfile outside your backup repository. Make sure you don't lock yourself out by "leaving your keys inside your car".
```bash
[vagrant@web important-files]$ scp ~/.borg-passphrase vagrant@172.30.0.4:~
The authenticity of host '172.30.0.4 (172.30.0.4)' can't be established.
ED25519 key fingerprint is SHA256:IAD3g7Enx4S7u09b1lRiGV2ncKaAx1qno5pn6GgWXlI.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.30.0.4' (ED25519) to the list of known hosts.
vagrant@172.30.0.4's password:
.borg-passphrase                                                                                                          100%   44   101.7KB/s   00:00
```

- Create a backup, make sure it is called first.

```bash
[vagrant@web ~]$ borg create vagrant@172.30.0.15:~/backups::first important-files/
```

- List all backups. You should see a similar output as the following:
```bash
[vagrant@web ~]$ borg info vagrant@172.30.0.15:~/backups
Repository ID: c634ff4bb2a0f1ecb510a0d76a963047565868a3f335824b778c7ad73fc9f709
Location: ssh://vagrant@172.30.0.15/~/backups
Encrypted: Yes (repokey)
Cache: /home/vagrant/.cache/borg/c634ff4bb2a0f1ecb510a0d76a963047565868a3f335824b778c7ad73fc9f709
Security dir: /home/vagrant/.config/borg/security/c634ff4bb2a0f1ecb510a0d76a963047565868a3f335824b778c7ad73fc9f709
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
All archives:                1.74 MB              1.72 MB              1.72 MB

                       Unique chunks         Total chunks
Chunk index:                       7                    7
```


- Add a file test.txt with as content Hello world. Create another backup, make sure it is called second. You should see a similar output as the following:
```bash
[vagrant@web important-files]$ echo "Hello world" > test.txt
[vagrant@web ~]$ borg create vagrant@172.30.0.15:~/backups::second important-files/
Archive second already exists
[vagrant@web ~]$ borg list vagrant@172.30.0.15:~/backups
first                                Sat, 2024-12-28 13:22:33 [e78ef4099e0d02fd1ae44a44d377f37902c075f65e06b3140f987898c5d6f6fc]
second                               Sat, 2024-12-28 13:24:16 [86ed91030bb5e83eabcac4d70a45d325377e6b4ef660adc8bc3f0d56ef3f147f]
```
```bash
[vagrant@web ~]$ borg list vagrant@172.30.0.15:~/backups::first
drwxr-xr-x vagrant vagrant        0 Sat, 2024-12-28 12:52:06 important-files
-rw-r--r-- vagrant vagrant    36196 Sat, 2024-12-28 12:51:49 important-files/bf1f3fb5-b119-4f9f-9930-8e20e892b898-720.mp4
-rw-r--r-- vagrant vagrant      300 Sat, 2024-12-28 12:51:49 important-files/100.txt
-rw-r--r-- vagrant vagrant      300 Sat, 2024-12-28 12:51:49 important-files/996.txt
-rw-r--r-- vagrant vagrant  1702187 Sat, 2024-12-28 12:51:50 important-files/Toreador_song_cleaned.ogg
```
```bash
[vagrant@web ~]$ borg list vagrant@172.30.0.15:~/backups::second
drwxr-xr-x vagrant vagrant        0 Sat, 2024-12-28 13:24:07 important-files
-rw-r--r-- vagrant vagrant    36196 Sat, 2024-12-28 12:51:49 important-files/bf1f3fb5-b119-4f9f-9930-8e20e892b898-720.mp4
-rw-r--r-- vagrant vagrant      300 Sat, 2024-12-28 12:51:49 important-files/100.txt
-rw-r--r-- vagrant vagrant      300 Sat, 2024-12-28 12:51:49 important-files/996.txt
-rw-r--r-- vagrant vagrant  1702187 Sat, 2024-12-28 12:51:50 important-files/Toreador_song_cleaned.ogg
-rw-r--r-- vagrant vagrant       12 Sat, 2024-12-28 13:24:07 important-files/test.txt
```
```bash
[vagrant@web ~]$ borg info vagrant@172.30.0.15:~/backups
Repository ID: c634ff4bb2a0f1ecb510a0d76a963047565868a3f335824b778c7ad73fc9f709
Location: ssh://vagrant@172.30.0.15/~/backups
Encrypted: Yes (repokey)
Cache: /home/vagrant/.cache/borg/c634ff4bb2a0f1ecb510a0d76a963047565868a3f335824b778c7ad73fc9f709
Security dir: /home/vagrant/.config/borg/security/c634ff4bb2a0f1ecb510a0d76a963047565868a3f335824b778c7ad73fc9f709
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
All archives:                3.48 MB              3.44 MB              1.72 MB

                       Unique chunks         Total chunks
Chunk index:                      10                   15
```

- With which bash command can you see the size of the folder with the files on the webserver? How big is that folder? Tip: try with and without the --si option. Which corresponds with the output of borg? Where do you find this in the BorgBackup documentation?

```bash
https://borgbackup.readthedocs.io/en/stable/usage/general.html 
--iec
format using IEC units (1KiB = 1024B)

deze correspondeert met Borgbackup, deze gebruikt base-2 units (1 KB is equal to 1024 bytes):
[vagrant@web ~]$ du -hs important-files/
1.7M    important-files/

–si reports base-10 units (where 1 kB is equal to 1000 bytes):
[vagrant@web ~]$ du -h --si important-files/
1.8M    important-files/
```

- Now check the size of the backups folder on the database server.
```bash
database:~$ du -hs backups/
1.8M    backups/
```

- What is the difference between Original size, Compressed size and Deduplicated size? Can you link this with the sizes you found for the folders on the web and db VM's? Make sure you comprehend this!

Original size:
effectieve grootte van het bestand bv 100MB

Compressed size:
Dit is de grootte van je back-up nadat compressie is toegepast om ruimte te besparen. Het is meestal kleiner dan de originele grootte. bv 50MB.


Deduplicated size:
Dit is de grootte van je back-up nadat duplicaten zijn verwijderd om ruimte te besparen. Het is vaak kleiner dan de gecomprimeerde grootte als er vergelijkbare gegevens in meerdere back-ups staan. bv bij 2 backups met duplicaten kan de grootte kleiner worden dan zonder duplicaten

- What are chunks?

Chunks zijn kleine stukjes of eenheden waarin data wordt verdeeld. Ze worden gebruikt voor efficiëntere opslag, overdracht en verwerking van gegevens.

- It is necessary to periodically check the integrity of the borg repository. With which command can this be done? When should I use the --verify-data option? Tip: use --verbose to see more information.

Het periodiek controleren van de integriteit van een BorgBackup repository is een belangrijke best practice om de betrouwbaarheid en consistentie van uw back-ups te waarborgen.

gebruik –verify-dat als wanneer je zorgen hebt over de integriteit van je back-up gegevens, dit is wel resource-intensief


Doe dit om het ww te achterhalen:

```bash
[vagrant@web ~]$ cat ~/.borg-passphrase
tUH5MhNmMY0CsdNIE44ebs4+Qi/SLNiDAdrLHo3mB9g=[vagrant@web ~]$
```
```bash
database:~$ borg check --verbose backups/
Starting repository check
finished segment check at segment 17
Starting repository index check
Index object count match.
Finished full repository check, no problems found.
Starting archive consistency check...
Enter passphrase for key /home/vagrant/backups:
Enter passphrase for key /home/vagrant/backups:
Enter passphrase for key /home/vagrant/backups:
Analyzing archive first (1/2)
Analyzing archive second (2/2)
Archive consistency check complete, no problems found.
```
```bash
database:~$ borg check --verify-data --verbose backups/
Starting repository check
finished segment check at segment 17
Starting repository index check
Index object count match.
Finished full repository check, no problems found.
Starting archive consistency check...
Enter passphrase for key /home/vagrant/backups:
Starting cryptographic data integrity verification...
Finished cryptographic data integrity verification, verified 11 chunks with 0 integrity errors.
Analyzing archive first (1/2)
Analyzing archive second (2/2)
Archive consistency check complete, no problems found.
```

- Delete the original files on web.
```bash
[vagrant@web ~]$ rm --recursive --verbose important-files/
removed 'important-files/bf1f3fb5-b119-4f9f-9930-8e20e892b898-720.mp4'
removed 'important-files/Toreador_song_cleaned.ogg'
removed 'important-files/100.txt'
removed 'important-files/996.txt'
removed 'important-files/test.txt'
removed directory 'important-files/'
```

- Restore the original files using the first backup on the database server (without the test.txt file) to the same place on web so it seems like nothing has happened. --strip-elements may come in handy here as borg uses absolute paths inside backups. You should see a similar output after restoring the backup:

```bash
[vagrant@web ~]$ borg extract --strip-components=0 vagrant@172.30.0.15:~/backups::first important-files/
[vagrant@web ~]$ ll important-files/
total 1708
-rw-r--r--. 1 vagrant vagrant     300 Dec 28 12:51 100.txt
-rw-r--r--. 1 vagrant vagrant     300 Dec 28 12:51 996.txt
-rw-r--r--. 1 vagrant vagrant 1702187 Dec 28 12:51 Toreador_song_cleaned.ogg
-rw-r--r--. 1 vagrant vagrant   36196 Dec 28 12:51 bf1f3fb5-b119-4f9f-9930-8e20e892b898-720.mp4
```

- Automate the backups and set an appropriate retention policy. Look at the documentation to have a starting point. What is the retention policy here? Have you ever heard of the Grandfather-Father-Son policy? The automation should create a backup every 5 minutes. There are various ways to do this, but we prefer a systemd timer to execute the script on the time intervals.

```bash
[vagrant@web ~]$ sudo cat /usr/local/bin/borg-backup.sh
#!/bin/sh

# Setting this, so the repo does not need to be given on the commandline:
export BORG_REPO='vagrant@172.30.0.15:~/backups'

# See the section "Passphrase notes" for more infos.
export BORG_PASSPHRASE='IloveCybersec'

# some helpers and error handling:
info() { printf "\n%s %s\n\n" "$( date )" "$*" >&2; }
trap 'echo $( date ) Backup interrupted >&2; exit 2' INT TERM

info "Starting backup"

# Backup the most important directories into an archive named after
# the machine this script is currently running on:

borg create                         \
    --verbose                       \
    --filter AME                    \
    --list                          \
    --stats                         \
    --show-rc                       \
    --compression lz4               \
    --exclude-caches                \
    ${BORG_REPO}::'{hostname}-{now}' \
    /home/vagrant/important-files     \

backup_exit=$?

info "Pruning repository"

# Use the `prune` subcommand to maintain 7 daily, 4 weekly and 6 monthly
# archives of THIS machine. The '{hostname}-*' matching is very important to
# limit prune's operation to this machine's archives and not apply to
# other machines' archives also:

borg prune                          \
    --list                          \
    --glob-archives '{hostname}-*'  \
    --show-rc                       \
    --keep-daily    7               \
    --keep-weekly   4               \
    --keep-monthly  6 \
    ${BORG_REPO}

prune_exit=$?

# actually free repo disk space by compacting segments

info "Compacting repository"

borg compact ${BORG_REPO}

compact_exit=$?

# use highest exit code as global exit code
global_exit=$(( backup_exit > prune_exit ? backup_exit : prune_exit ))
global_exit=$(( compact_exit > global_exit ? compact_exit : global_exit ))

if [ ${global_exit} -eq 0 ]; then
    info "Backup, Prune, and Compact finished successfully"
elif [ ${global_exit} -eq 1 ]; then
    info "Backup, Prune, and/or Compact finished with warnings"
else
    info "Backup, Prune, and/or Compact finished with errors"
fi

exit ${global_exit}
```

- Maak de borg service aan:
```bash
[vagrant@web ~]$ sudo cat /etc/systemd/system/borg-backup.service
[Unit]
Description=Borg Backup Service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/borg-backup.sh
```

- Maak een timer service aan:
```bash
[vagrant@web ~]$ sudo cat /etc/systemd/system/borg-backup.timer
[Unit]
Description=Run Borg Backup every 5 minutes

[Timer]
OnUnitActiveSec=5min
Persistent=true

[Install]
WantedBy=timers.target
```

- Start de service:
```bash
[vagrant@web ~]$ sudo systemctl enable borg-backup.timer
Created symlink /etc/systemd/system/timers.target.wants/borg-backup.timer → /etc/systemd/system/borg-backup.timer.
[vagrant@web ~]$ sudo systemctl start borg-backup.timer
[vagrant@web ~]$ sudo systemctl status borg-backup.timer
● borg-backup.timer - Run Borg Backup every 5 minutes
     Loaded: loaded (/etc/systemd/system/borg-backup.timer; enabled; preset: disabled)
     Active: active (elapsed) since Sat 2024-12-28 13:55:22 UTC; 10s ago
      Until: Sat 2024-12-28 13:55:22 UTC; 10s ago
    Trigger: n/a
   Triggers: ● borg-backup.service

Dec 28 13:55:22 web systemd[1]: Started Run Borg Backup every 5 minutes.
```


retention policy is a common backup strategy that involves keeping three sets of backups: a "grandfather" monthly backup, a "father" weekly backup, and a "son" daily backup.

Be careful, prune is a potentially dangerous command, it will remove backup archives.


- What does the borg compact command do?

borg compact is een BorgBackup-opdracht die ruimte in het repository vrijmaakt door ongebruikte gegevensblokken te verwijderen en de opslagefficiëntie te optimaliseren.


- can I use tools like borg to backup an active database? Why (not)? Should I take any extra measures to do this safely?

Ja, je kunt tools zoals Borg gebruiken om een actieve database te back-uppen, maar er zijn belangrijke overwegingen. Borg legt de staat van bestanden vast op het moment dat het deze leest. Als de databasebestanden veranderen tijdens het back-upproces, kan de back-up een inconsistente staat van de database vastleggen. Om een consistente back-up te waarborgen, is het aanbevolen om databasespecifieke tools te gebruiken om een dump van de database te maken, die vervolgens met Borg kan worden geback-upt. Dit zorgt ervoor dat de back-up een consistente staat van de database weerspiegelt.


- There is a tool that has been built on top of borg called borgmatic. What does it do? Could it be useful to you? Why (not)?

Borgmatic is een tool die bovenop BorgBackup is gebouwd. Het vereenvoudigt het configureren en automatiseren van backups met Borg. Borgmatic kan handig zijn voor het back-uppen van databases omdat het helpt bij het consistent houden van back-ups, vooral door het automatiseren van het back-upproces en het toepassen van retentiebeleid. Dit is belangrijk voor databases om ervoor te zorgen dat de back-ups een betrouwbare staat van de data weerspiegelen.
