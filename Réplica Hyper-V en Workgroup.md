# Prérequis
- Un serveur avec Windows Server 2022
- Hyper-V installé dessus
- 2 VM Windows Server 2022 avec également Hyper-V (Serv1 et Serv2) en 172.17.1.1 et .2
- 1 VM pour chaque VM mère (Serv1 et Serv2)

<img width="602" height="354" alt="Picture" src="https://github.com/user-attachments/assets/0aea0bdc-1b0a-4308-9699-b7c3e0e58885" />


# 1. Résolution de noms (Fichier Hosts)
Sans serveur DNS de domaine, les serveurs doivent pouvoir se "voir" par leur nom d'hôte.
- Sur chaque serveur, éditer le fichier : ```C:\Windows\System32\drivers\etc\hosts.```
- Ajouter l'adresse IP et le nom de l'autre serveur.
- Exemple depuis Serv1 : ```172.17.1.2 Serv2```
- Exemple depuis Serv2 : ```172.17.1.1 Serv1```

# 2. Création de certificats (PowerShell)
Il faut générer des certificats auto-signés. Pour que cela fonctionne, le certificat doit supporter à la fois l'**Authentification Client** et l'**Authentification Serveur**.
Exécuter ces commandes sur le **Serveur Principal** :
```
# Création d'un certificat racine (Root CA)
$rootCert = New-SelfSignedCertificate -DnsName "HyperVReplicaRoot" -CertStoreLocation "Cert:\LocalMachine\My" -TestRoot 

New-SelfSignedCertificate -DnsName "Serv1" ` -CertStoreLocation "Cert:\LocalMachine\My" ` 
-Signer $rootCert ` 
-KeyUsage DigitalSignature, KeyEncipherment ` 
-TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.1,1.3.6.1.5.5.7.3.2")
```
**Répéter l'opération sur le serveur de réplica** en **adaptant** le nom de -DnsName.

# 3. Importation et confiance

Pour que la réplication fonctionne, chaque serveur doit faire confiance au certificat de l'autre :

1. **Exporter** le certificat racine (.cer) créé à l'étape précédente (avec la clé privée)
2. Le **copier** sur l'autre serveur
3. **L'importer** dans le magasin **Autorités de certification racines de confiance** (Trusted Root Certification Authorities) de chaque machine

**Note importante** : Après l’importation, Modifier le registre comme suit. Car comme ce sont des certificats auto-signés, il n'y a pas de liste de révocation (CRL). Il faut dire à Windows de ne pas la vérifier en exécutant cette commande sur les deux serveurs :

```
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Replication" /v DisableCertRevocationCheck /d 1 /t REG_DWORD /f
```

# 4. Configuration du serveur de réplica
Sur le serveur qui va recevoir la VM (le réplica) :
1.	Ouvrez le Gestionnaire Hyper-V.
2.	Faire un clic droit sur le serveur > Paramètres Hyper-V.
3.	Aller dans Configuration de la réplication.
4.	Cocher Activer ce ordinateur en tant que serveur de réplication.
5.	Cocher Utiliser l'authentification par certificat (HTTPS) (Port 443).
6.	Cliquer sur Sélectionner un certificat et choisir celui qui a été créé.
7.	Sélectionner Autoriser la réplication à partir de n'importe quel serveur authentifié ou sélectionner des partenaires de réplication spécifiques.
