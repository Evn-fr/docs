# Prérequis
```
- Une machine Debian 12 avec accès internet
- 4go de RAM alloué
- Site Web de léditeur : https://guacamole.apache.org
```

# Avant de commencer, renommer la machine en "guacamole"
```
nano /etc/hosts
nano /etc/hostname
```
guacamole

# Installation des dépendances
```
apt-get update
apt-get install build-essential libcairo2-dev libjpeg62-turbo-dev libpng-dev libtool-bin uuid-dev libossp-uuid-dev libavcodec-dev
libavformat-dev libavutil-dev libswscale-dev freerdp2-dev libpango1.0-dev libssh2-1-dev libtelnet-dev libvncserver-dev libwebsockets-dev
libpulse-dev libssl-dev libvorbis-dev libwebp-dev
```

# Téléchargement de la dernière version
```
cd /tmp
wget https://downloads.apache.org/guacamole/1.6.0/source/guacamole-server-1.6.0.tar.gz
```

# Décompression de l'archive et cd
```
tar -xzf guacamole-server-1.6.0.tar.gz
cd guacamole-server-1.6.0/
```

# Préparation de la compilation
```
./configure --with-systemd-dir=/etc/systemd/system/
```

# Compilation
```
sudo make
sudo make install
```

# Mise à jour des liens entre Guacamole-server et les librairies
```
sudo ldconfig
```
Normalement, la commande ne retourne rien

# Démarrage du service "guacd" et enable de celui-ci
```
sudo systemctl daemon-reload
sudo systemctl enable --now guacd
```

# Check du status
```
sudo systemctl status guacd
```

# Création d'un dossier guacamole dans /etc/ avec les sous dossier extensions et lib
```
mkdir -p /etc/guacamole/{extensions,lib}
```

# Création d'un dépôt pour télécharger une ancienne version de Tomcat
```
echo "deb http://deb.debian.org/debian/ bullseye main" > /etc/apt/sources.list.d/bullseye.list
```

# Mise à jour des paquets
```
apt update
```

# Téléchargement de Tomcat 9
```
apt-get install tomcat9 tomcat9-admin tomcat9-common tomcat9-user
```

