# Application du fuseau horaire
```
timedatectl set-timezone Europe/Paris
```

#  Éditez le fichier /etc/systemd/timesyncd.conf et paramétrez le serveur de temps utilisé
```
[Time]
NTP=ntp.univ-rennes2.fr
```

# Redémarrer le service puis vérifier l'heure
```
systemctl restart systemd-timesyncd.service
timedatectl
```

# Utilisation d'une doc ITconnect
```
 https://www.it-connect.fr/tuto-graylog-sur-debian-centraliser-et-analyser-logs/
```

# Installation des paquets nécessaires
```
sudo apt-get update
sudo apt-get install curl lsb-release ca-certificates gnupg2 pwgen
```

# Téléchaargement de la clé GPG qui correspond au dépôt de MongoDB
```
curl -fsSL https://www.mongodb.org/static/pgp/server-6.0.asc | gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg --dearmor
```

# Ajout du dépôt MongoDB sur la machine Debian12
```
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg] http://repo.mongodb.org/apt/debian bullseye/mongodb-org/6.0 main" | tee /etc/apt/sources.list.d/mongodb-org-6.0.list
```

# Mise à jour des paquets
```
apt-get update
apt-get install -y mongodb-org
```

# L'erreur suivante apparaît
```
Les paquets suivants contiennent des dépendances non satisfaites :
 mongodb-org-mongos : Dépend: libssl1.1 (>= 1.1.1) mais il n'est pas installable
 mongodb-org-server : Dépend: libssl1.1 (>= 1.1.1) mais il n'est pas installable
E: Impossible de corriger les problèmes, des paquets défectueux sont en mode « garder en l'état ».
```

# Téléchargement du paquet DEB libssl1.1_1.1.1f-1ubuntu2.24_amd64.deb
```
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.24_amd64.deb
sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2.24_amd64.deb
```

# On peut alors relancer l'install de MongoDB
```
apt-get install -y mongodb-org
```

# Relance du service MongoDB et activer son démarrage automatique
```
systemctl daemon-reload
systemctl enable mongod.service
systemctl restart mongod.service
systemctl --type=service --state=active | grep mongod
```

# Ajout de la clé de signature OpenSearch
```
curl -o- https://artifacts.opensearch.org/publickeys/opensearch.pgp | gpg --dearmor --batch --yes -o /usr/share/keyrings/opensearch-keyring
```

# Ajout du dépôt d'OpenSearch
```
echo "deb [signed-by=/usr/share/keyrings/opensearch-keyring] https://artifacts.opensearch.org/releases/bundle/opensearch/2.x/apt stable main" | tee /etc/apt/sources.list.d/opensearch-2.x.list
```

# Mettre à jour le cache des paquets
```
apt-get update
```

# Installation d'OpenSearch avec la définition d'un mot de passe (min/maj/special)
```
sudo env OPENSEARCH_INITIAL_ADMIN_PASSWORD=IT-Connect2024! apt-get install opensearch
```

# Ouverture du fichier de conf opensearch.yml
```
sudo nano /etc/opensearch/opensearch.yml

cluster.name: graylog
node.name: ${HOSTNAME}
path.data: /var/lib/opensearch
path.logs: /var/log/opensearch

------------------------------------------------------------------
Ajout dans la section discovery en dessous de discovery.seed :
discovery.type: single-node
------------------------------------------------------------------

network.host: 127.0.0.1
action.auto_create_index: false
plugins.security.disabled: true
```

# Configuration de Java (JVM) pour une config à 4go de RAM max
```
sudo nano /etc/opensearch/jvm.options

-Xms1g
-Xmx1g
```

# Activation du démarrage auto d'OpenSearch
```
systemctl daemon-reload
systemctl enable opensearch
systemctl restart opensearch
```

# Téléchergement et installation de Graylog dans sa dernière version
```
wget https://packages.graylog2.org/repo/packages/graylog-6.1-repository_latest.deb
dpkg -i graylog-6.1-repository_latest.deb
apt-get update
apt-get install graylog-server
```

# Création d'une clé de 96 caractères
```
pwgen -N 1 -s 96

Exemple ici
wVSGYwOmwBIDmtQvGzSuBevWoXe0MWpNWCzhorBfvMMhia2zIjHguTbfl4uXZJdHOA0EEb1sOXJTZKINhIIBm3V57vwfQV59
```

# Coller le mot de passe dans password_secret
```
nano /etc/graylog/server/server.conf
```

# Création du compte admin (avec exemple de clé)
```
echo -n "Password" | shasum -a 256
6b297230efaa2905c9a746fb33a628f4d7aba4fa9d5c1b3daa6846c68e602d71
```

# Coller la valeur retournée dans root_password_sha2
```
nano /etc/graylog/server/server.conf
```

# Ajouter les valeurs suivantes :
```
http_bind_address = 0.0.0.0:9000
--------------------------------
elasticsearch_hosts = http://127.0.0.1:9200
```

# Démarrer automatiquement Graylog au démarrage
```
systemctl enable --now graylog-server
```

# Accéder à l'interface Web
```
admin
Password
```
