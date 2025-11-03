# SRV-WEB1
crm resource stop serviceWeb
crm resource stop IPFailover
crm configure delete servweb
crm resource start IPFailover
crm configure clone cServiceWeb serviceWeb
crm resource start serviceWeb

# Dans /etc/network/interfaces
address 192.168.0.1/24
gateway 192.168.0.254

# Dans /etc/network/interfaces de SRV-WEB2
address 192.168.0.2/24
gateway 192.168.0.254

# Retour sur SRV-WEB1
crm configure edit

# Dans CRM Configure Edit (ouvert utilisant Vi)
# Dans node 2 après le paramètre "params ip" changer l'adresse :
192.168.0.3

# Dans les paramètre VMWare de SRV-WEB1 (Network Adapter) créer un LAN Segment DMZ Sodecaf puis le sélectionner
# Dans les paramètre VMWare de SRV-WEB2, connecter le LAN Segment DMZ SODECAF

# Désactivation d'un site en HTTPS pour en activer un autre en HTTP (à  faire sur SRV-WEB1 et 2)
a2dissite appliFrais.conf
a2ensite sodecaf.conf
systectl reload apache2

# Création d'une VM Debian12 shell
apt update
apt upgrade

# Renommer la machine dans /etc/hosts et /etc/hostname
haproxy
# Sur SRV-WEB1 et 2
cd /var/www/sodecaf
nano sodecaf.html

Ctrl + W

Ajouter un signe pour distinguer les deux SRV-WEB (exemple ici Accueil 1 pour server 1 et Accueil 2 pour server 2)

# Création d'une VM HAProxy
# Renommer la VM en haproxy puis reboot pour appliquer les changements
nano /etc/hosts
nano /etc/hostname

reboot

# Ajouter dans les paramètres VMWare de haproxy une carte réseau (Network Adapter)
# Network Adapter 1 en Host-only
# Network Adapter 2 en LAN Segment (DMZ SODECAF)

# Modifier les IPs des interfaces pour avoir un pied dans le LAN Sodecaf et un autre dans la DMZ
nano /etc/network/interfaces

allow-hotplug [NomCarteReseau1]
iface [NomCarteReseau1] inet static
  address 172.16.0.13/24
  gateway 172.16.0.254

allow-hotplug [NomCarteReseau2]
iface [NomCarteReseau2] inet static
  address 192.168.0.254

# Config du fichier haproxy.cfg
nano /etc/haproxy/haproxy.cfg

# Ajouter à la fin
listen httpProxy
  bind 172.16.0.13
  balance roundrobin
  option httpclose
  option httpchk HEAD / HTTP/1.0
  server serv1 192.168.0.1:80 check
  server serv2 192.168.0.2:80 check
