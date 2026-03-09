# Prérequis
```
- Une machine Debian 12 avec accès internet
- 4go de RAM alloué
- Site Web de l'éditeur : https://guacamole.apache.org
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

# Navigation vers le dossier /tmp et téléchargement de la WebApp Guacamole
```
cd /tmp
wget https://downloads.apache.org/guacamole/1.6.0/binary/guacamole-1.6.0.war
```

# Déplacement dans la librairie de Web App de Tomcat9
```
mv guacamole-1.6.0.war /var/lib/tomcat9/webapps/guacamole.war
```
# Restart des services Tomcat9 et Guacamole
```
systemctl restart tomcat9 guacd
```

# Installation du paquet MariaDB Server
```
sudo apt-get install mariadb-server
```

# Sécurisation de l'instance
```
sudo mysql_secure_installation
```

# Connexion en tant que root à MariaDB
```
mysql -u root -p
```

# Création d'une base de données et un utilisateur dédié pour Guacamole
```
CREATE DATABASE guacadb;
CREATE USER 'guaca_user'@'localhost' IDENTIFIED BY 'YourObviouslySecuredPassword';
GRANT SELECT,INSERT,UPDATE,DELETE ON guacadb.* TO 'guaca_user'@'localhost';
FLUSH PRIVILEGES;
EXIT
```

# Toujours depuis le dépôt officiel, on télécharge cette extension
```
cd /tmp
wget https://downloads.apache.org/guacamole/1.6.0/binary/guacamole-auth-jdbc-1.6.0.tar.gz
```

# On décompresse l'archive
```
tar -xzf guacamole-auth-jdbc-1.6.0.tar.gz
```

# Déplacement du fichier ".jar" de l'extension dans le répertoire "/etc/guacamole/extensions/" créé précédemment
```
sudo mv guacamole-auth-jdbc-1.5.5/mysql/guacamole-auth-jdbc-mysql-1.5.5.jar /etc/guacamole/extensions/
```

# Téléchargement du connecteur MySQL
```
cd /tmp
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-j-9.6.0.tar.gz
```

# Décompression du fichier précédemment téléchargé
```
tar -xzf mysql-connector-j-9.6.0.tar.gz
```

# Copie du fichier .jar du connecteur vers le répertoire "lib" d'Apache Guacamole
```
cp mysql-connector-j-9.6.0/mysql-connector-j-9.6.0.jar /etc/guacamole/lib/
```

# Import de la strcture de la base de données Guacamole dans guacadb
```
cd guacamole-auth-jdbc-1.5.5/mysql/schema/
cat *.sql | mysql -u root -p guacadb
```

# Ouverture et edit du fichier guacamole.properties
```
nano /etc/guacamole/guacamole.properties

# MySQL
  mysql-hostname: 127.0.0.1
  mysql-port: 3306
  mysql-database: guacadb
  mysql-username: guaca_user
  mysql-password: YourObviouslySecuredPassword
```

# Déclaration du serveur Guacamole
```
nano /etc/guacamole/guacd.conf

[server] 
  bind_host = 0.0.0.0
  bind_port = 4822
```

# Restart des services tomcat, guacd et mariadb
```
systemctl restart tomcat9 guacd mariadb
```

# Accès à l'interface Web de Guacamole
```
http://<Adresse IP>:8080/guacamole/

Identifiants (par défaut):
>  guacadmin
>  guacadmin
```

# Si erreur de connexion RDP
```
useradd -M -d /var/lib/guacd/ -r -s /sbin/nologin -c "Guacd User" guacd
mkdir /var/lib/guacd
chown -R guacd: /var/lib/guacd
sed -i 's/daemon/guacd/' etc/systemd/system/guacd.service
systemctl daemon-reload
systemctl restart guacd
```

# ===========[ Setup MFA ]===========
```

```
