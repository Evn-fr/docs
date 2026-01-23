# Prérequis
- Un serveur avec Windows Server 2022
- Hyper-V installé dessus
- 2 VM Windows Server 2022 avec également Hyper-V (Serv1 et Serv2) en 172.17.1.1 et .2
- 1 VM pour chaque VM mère (Serv1 et Serv2)

# 1. Résolution de noms (Fichier Hosts)
Sans serveur DNS de domaine, les serveurs doivent pouvoir se "voir" par leur nom d'hôte.
●	Sur chaque serveur, éditer le fichier : ```C:\Windows\System32\drivers\etc\hosts.```
●	Ajouter l'adresse IP et le nom de l'autre serveur.
●	Exemple depuis Serv1 : ```172.17.1.2 Serv2```
●	Exemple depuis Serv2 : ```172.17.1.1 Serv1```
