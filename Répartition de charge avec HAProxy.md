# SRV-WEB1
```
crm resource stop serviceWeb
crm resource stop IPFailover
crm configure delete servweb
crm resource start IPFailover
crm configure clone cServiceWeb serviceWeb
crm resource start serviceWeb
```

# Dans /etc/network/interfaces
```
address 192.168.0.1/24
gateway 192.168.0.254
```

# Dans /etc/network/interfaces de SRV-WEB2
```
address 192.168.0.2/24
gateway 192.168.0.254
```

# Retour sur SRV-WEB1
```
crm configure edit
```

# Dans CRM Configure Edit (ouvert utilisant Vi)
# Dans node 2 après le paramètre "params ip" changer l'adresse :
```
192.168.0.3
```

# Dans les paramètre VMWare de SRV-WEB1 (Network Adapter) créer un LAN Segment DMZ Sodecaf puis le sélectionner
# Dans les paramètre VMWare de SRV-WEB2, connecter le LAN Segment DMZ SODECAF

# Désactivation d'un site en HTTPS pour en activer un autre en HTTP (à  faire sur SRV-WEB1 et 2)
```
a2dissite appliFrais.conf
a2ensite sodecaf.conf
systectl reload apache2
```

# Création d'une VM Debian12 shell
```
apt update
apt upgrade
```

# Renommer la machine dans /etc/hosts et /etc/hostname
```
haproxy

```
_reboot pour appliquer les changements_

# Sur SRV-WEB1 et 2
```
cd /var/www/sodecaf
nano sodecaf.html
```

_Ctrl + W > "Accueil"_

_Ajouter un signe pour distinguer les deux SRV-WEB (exemple ici Accueil 1 pour server 1 et Accueil 2 pour server 2)_

# Ajouter dans les paramètres VMWare de haproxy une carte réseau (Network Adapter)
# Network Adapter 1 en Host-only
# Network Adapter 2 en LAN Segment (DMZ SODECAF)

# Modifier les IPs des interfaces pour avoir un pied dans le LAN Sodecaf et un autre dans la DMZ
_nano /etc/network/interfaces_

```
allow-hotplug [NomCarteReseau1]
iface [NomCarteReseau1] inet static
  address 172.16.0.13/24
  gateway 172.16.0.254

allow-hotplug [NomCarteReseau2]
iface [NomCarteReseau2] inet static
  address 192.168.0.254
```

# Config du fichier haproxy.cfg
```
nano /etc/haproxy/haproxy.cfg
```

# Ajouter à la fin
```
listen httpProxy
  bind 172.16.0.13
  balance roundrobin
  option httpclose
  option httpchk HEAD / HTTP/1.0
  server serv1 192.168.0.1:80 check
  server serv2 192.168.0.2:80 check
```
# Téléchergement du paquet HAProxy
```
apt update
apt install haproxy
```

# Tester le basculement des 2 serveurs web

# Config du fichier haproxy.cfg
```
nano /etc/haproxy/haproxy.cfg
```

# Ajouter à la fin
```
  stats uri /statsHaproxy
  stats auth root:Btssio2017
  stats refresh 30s
```

# Tester le fonctionnement du site
```
http://[IpAddr]/statsHaproxy
  > Login
  > MDP
```

# Revenir dans le fichier nano /etc/haproxy/haproxy.cfg et ajouter avant listen httpProxy
```
frontend websodecaf
  bind 172.16.0.13:80
  default_backend clusterweb
```

# Puis modifier listen httpProxy en
```
backend clusterweb
```

# Affecter un poid sur chaque serveur (entre 0 et 255) dans backend clusteweb (roundrobin pondéré)
```
  server serv1 192.168.0.1:80 weight 100 check
  server serv2 192.168.0.2:80 weight 50 check
```

# Installation d'OpenSSL pour générer des clés
```
apt install openssl
```

# Création d'un dossier et cd dans celui-ci
```
mkdir /etc/haproxy/cert
cd /etc/haproxy/cert
```

# Génération des clés
```
openssl genrsa -out /etc/haproxy/cert/privateKey.pem 4096
openssl req -new -x509 -days 365 -key /etc/haproxy/cert/privateKey.pem -out /etc/haproxy/cert/cert.pem
  > Language (FR)
  > Country
  > City
  > Organization name
  > Unit name
  > FQDN
  > Email
```

# On fusionne ensuite le certificat et la clé privée dans un même fichier :
```
cat cert.pem privateKey.pem > sodecaf.pem
```

# Dans /etc/haproxy/haproxy.cfg modifier la ligne dans frontend websodecaf
```
  bind 172.16.0.13:443 ssl crt /etc/haproxy/cert/sodecaf.pem
```

# Rediriger les requêtes HTTP (80) vers HTTPS (443)
```
frontend sodecafhttp
  bind 172.16.0.13 :80
  http-request redirect scheme https unless { ssl_fc }
  default_backend fermeweb
```

# Passage sur OPNsense
Dans les paramètres de VMWare ajouter un Network Adapter en LAN Segment (DMZ Sodecaf)

Mettre à jour OPNsense

Interface > Assignation > Assigner la nouvelle carte réseau (DMZ)

Système > Firmware > Greffons > Chercher un Greffons et cochant la case show community plugins
  > haproxy

# Création de 2 serveurs réels
```
✅ Activé    SRV-WEB1    statique    192.168.0.1    80
✅ Activé    SRV-WEB2    statique    192.168.0.2    80
```

# Dans Système > Gestion des certificats > Autorités
```
Ajouter

Importer une CA existante
CA SODECAF
[Données du certificat]
[Données de la clé privée]
```

# Dans Services > HAProxy > Paramètres et Serveurs réels
```
> Sélectionner Backend Pools
```

# Création d'une solution Backend
```
> Backend Pools

> Activé
Nom : BACKEND-WEB-SODECAF
Mode : HTTP (Couche 7) [par défaut]
Algo : "Round-Robin"
Serveurs : [SRV-WEB1] [SRV-WEB2]
```

# Services > HAProxy > Paramètres et Service Public
```
> Activé
> frontend-web-https
> [IP WAN OPNsense]:443
> SSL /HTTPS (mode TCP)
> ✅ Activer le délestage SSL
> [Certificat WEB]
> Vérification : requis
```

# Ouverture du port 443 
```
Pare-feu > Règles > WAN

✅ Ajouter

> Action : Autoriser
> Interface : WAN
> Version TCP/IP : IPv4
> Protocole : TCP
> Source : any
> Destionation : WAN adresse
> Plage de ports de destination : HTTPS
```
