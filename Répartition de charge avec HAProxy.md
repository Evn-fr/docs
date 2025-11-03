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

# Sur SRV-WEB1 et 2
cd /var/www/sodecaf
nano sodecaf.html

Ctrl + W

Ajouter un signe pour distinguer les deux SRV-WEB (exemple ici Accueil 1 pour server 1 et Accueil 2 pour server 2)
