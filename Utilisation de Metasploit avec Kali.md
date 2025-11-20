# Prérequis :
> Une machine Kali Linux (ici Kali v2025.2) et une machine Metasploitable

# Dans la console Kali
```
msfconsole
```

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

