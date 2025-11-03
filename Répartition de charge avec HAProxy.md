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
# Dans node 2 après le paramètre "params ip" changer l'adresse en :
192.168.0.3

