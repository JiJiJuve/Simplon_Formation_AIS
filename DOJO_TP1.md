
# ***PROJET DOJO : Création d’un Lab dans un environnement à la fois vulnérable et réaliste***
---

## **Sommaire**

- [Introduction](#introduction)
- [Machines](#machines)
- [Descriptifs des différentes étapes](#Descriptifs-des-différentes-étapes)
- [Schéma Infrastructure](#schéma-infrastructure)
- [Verification du hash ISO](#verification-du-hash-iso)
- [Mises à jour](#mises-à-jour)
- [Joindre Client Windows au Domain AD](#joindre-client-windows-au-domain-ad)
- [Statut des Services](#statut-des-services)
- [Mise en place de WinRM, SMB, Web-Linux](#mise-en-place-de-winRM,-SMB,-web-linux)
- [Script BadBlood](#script-badblood)
- [Listes des comptes a risques](#listes-des-comptes-a-risques)
- [Kanban](#kanban)


## **Introduction**

Ce projet consiste à créer un laboratoire informatique dans un environnement à la fois vulnérable et réaliste, 
basé sur des services couramment utilisés en entreprise. L’objectif est de simuler un environnement de travail, 
tout en étudiant la sécurisation, la gestion et l’exploitation des systèmes. 

## **Machines**

- Windows 10  
- Windows Server 2022 
- Debian 11.6

 
## **Descriptifs des différentes étapes**

1. Télécharger les images (fichiers ISO ou machines virtuelles) des systèmes 
Windows et Linux, puis vérifier leur intégrité à l’aide des sommes de contrôle 
(hashs) ou des signatures fournies par les éditeurs. 
2. Installer trois machines virtuelles : un contrôleur de domaine sous Windows 
Server, un poste client Windows, et une machine Linux. 
3. Effectuer toutes les mises à jour de sécurité nécessaires sur ces machines. 
4. Sur la Workstation Windows, joindre le domaine Active Directory (en prenant 
en compte les contraintes liées aux licences). 
5. Créer un utilisateur administrateur (root pour Linux, administrateur pour 
Windows) ainsi qu’un utilisateur local standard non administrateur sur 
chacune des machines. 
6. Mettre en place les services essentiels sur le domaine Active Directory et 
configurer la Workstation afin qu’elle puisse les utiliser efficacement. Ces 
services comprennent : 
- DNS sous Windows, utilisé par défaut dans le domaine AD, 
- WinRM sous Windows, accessible uniquement aux administrateurs du 
domaine, 
- Partages SMB sur Windows, avec un partage en lecture seule et un autre en 
écriture accessible sans authentification, 
- Serveur Web sur Linux, via l’application VulnerableLightApp qui inclut un 
serveur Kestrel accessible sur le réseau, 
- SSH sur Linux, ouvert aux membres du groupe sudoers. 
7. Enfin, exécuter le script automatisé nommé « badblood » pour finaliser la mise 
en place.

## **Schéma Infrastructure**

<img width="824" height="599" alt="DOJO" src="https://github.com/user-attachments/assets/1280a8a9-bbb5-4f50-9220-bd5122a7b342" />


## **Verification du hash ISO**

#### Machine Debian (Server Linux) → 

<img width="778" height="196" alt="hash_debian" src="https://github.com/user-attachments/assets/2c505d8b-fe80-42cb-b362-24b340806a3a" />

#### Machine Windows 10 (Client AD) →

<img width="792" height="355" alt="Hash_W10" src="https://github.com/user-attachments/assets/2eac63b6-5f08-450d-9b72-13a34e2304ab" />

#### Machine Windows Server 2022 (Server AD) → 

<img width="776" height="386" alt="hash_WServer22" src="https://github.com/user-attachments/assets/b6b66296-2150-44f4-907f-1580bd46d48b" />

Pour trouver le hash d’un fichier dans PowerShell :

```
Get-FileHash -Algorithm SHA256 -Path "chemin\vers\ton\fichier"
```

   - Get-FileHash : cmdlet qui calcule la valeur de hachage d’un fichier.

   - Algorithm SHA256 : spécifie l’algorithme de hachage (ici SHA256, mais tu peux aussi utiliser MD5, SHA1, SHA384, SHA512).

   - Path "chemin\vers\ton\fichier" : indique le chemin complet ou relatif du fichier dont tu veux calculer le hash.

## **Mises à jour**

#### Màj Windows Server 2022 → 

<img width="1920" height="966" alt="Maj_WServer22" src="https://github.com/user-attachments/assets/d9a51b7c-a590-420e-97e4-87e99cbfb6af" />

#### Màj Windows 10 → 

<img width="1024" height="768" alt="Maj_W10" src="https://github.com/user-attachments/assets/03d46756-536d-4863-bceb-ff310e68d8bf" />

#### Màj Server Debian → 

<img width="1920" height="966" alt="Maj_Debian" src="https://github.com/user-attachments/assets/36da74be-a44d-4503-9242-c9b7c1d62ff2" />


## **Joindre Client Windows au Domain AD**

#### Joindre le client Windows 10 au domaine Active Directory Windows server 2022 →

<img width="1920" height="966" alt="ClientW10_in_AD" src="https://github.com/user-attachments/assets/0a97735c-6ec6-4895-a540-43234580417d" />


## **Statut des Services**

<img width="1001" height="947" alt="Statut_Service_AD" src="https://github.com/user-attachments/assets/7f1951b7-7ae8-4486-9874-9621d49e4a22" />


## **Mise en place de WinRM, SMB, Web-Linux**

#### SMB Partage ReadOnly →

<img width="1920" height="966" alt="SMB_Partage_ReadOnly" src="https://github.com/user-attachments/assets/9bef11d4-098e-4450-a6ab-d6fe5e06c632" />

#### SMB Partage Write  →

<img width="1920" height="966" alt="SMB_Partage_Write" src="https://github.com/user-attachments/assets/abbaf184-9a1f-4a7b-a840-ba9dbea4deb3" />

#### Activation WinRM → 

<img width="956" height="954" alt="WinRm" src="https://github.com/user-attachments/assets/a554b869-e6fc-4648-9730-08b8145faccd" />

#### Connexion depuis Windows10 → Windows Server 2022 AD en WinRM → 

<img width="1024" height="768" alt="ConnectionWinRM_depuis_ClientW10" src="https://github.com/user-attachments/assets/853cc55d-6ed0-40ee-8729-1cfe4b21ee40" />

#### Connexion depuis Windows10 → Windows Server 2022 AD en WinRM → 

<img width="1024" height="768" alt="ConnectionWinRM_depuis_ClientW10-2" src="https://github.com/user-attachments/assets/b7dd0a1f-9b25-45fa-8c39-e07a6a556ee3" />

#### Connexion depuis Windows10 → Windows Server 2022 AD en SSH→ 

<img width="1024" height="768" alt="SSH_W10_WS22" src="https://github.com/user-attachments/assets/0e527953-be6b-41d4-8c3d-cd447e655c08" />

#### Connexion depuis Windows Server 2022 AD → Windows10 en SSH → 

<img width="1920" height="966" alt="SSH_W22_W10" src="https://github.com/user-attachments/assets/f8b665bd-00c1-48b8-8d42-34b1c9faaf50" />

#### Connexion depuis Windows10 → Debian en SSH → 

<img width="1024" height="768" alt="SSH_W10_Debian" src="https://github.com/user-attachments/assets/85ba6415-580c-4b4c-8321-2ead7ddd8b20" />


#### Web-Linux (VulnerableLightApp) → 

<img width="1920" height="966" alt="VulnerableLightApp" src="https://github.com/user-attachments/assets/e58ecee7-5d77-4bba-a37d-02442dde2e22" />

## **Script BadBlood**

#### Exécution du script → 

<img width="859" height="737" alt="badblood_creation" src="https://github.com/user-attachments/assets/b5f80a75-db72-48ed-b969-36e21156e4c9" />
 
<img width="841" height="726" alt="badblood_creation2" src="https://github.com/user-attachments/assets/1c7740c7-c525-43d6-94fc-f2fd70f64336" />

<img width="768" height="666" alt="badblood_creation3" src="https://github.com/user-attachments/assets/ffeb4176-9a16-481e-a006-1b3b8ce8abab" />


## **Listes des comptes a risques**

- ***Comptes et Groupes Privilégiés ou Critiques – Windows Server, Windows 10 et Linux***

| Plateforme            | Nom du compte/groupe                     | Type                          | Description                                                                                     | Niveau de risque | Risque spécifique / Exemple d’exploitation |
|-----------------------|------------------------------------------|-------------------------------|-------------------------------------------------------------------------------------------------|------------------|-------------------------------------------|
| **Windows Server (AD)** | **Administrateur** | Utilisateur | Compte d’administration local. | Extrême | Accès total au système local. |
| **Windows Server (AD)** | **Administrateurs** | Groupe de sécurité - Global | Contrôle total sur le domaine. | Extrême | Accès complet à toutes les ressources AD. |
| **Windows Server (AD)** | **Administrateurs clés Enterprise** | Groupe de sécurité - Universel | Administration critique dans toute la forêt. | Extrême | Compromission = contrôle complet de la forêt. |
| **Windows Server (AD)** | **Administrateurs de l’entreprise** | Groupe de sécurité - Universel | Contrôle total de la forêt AD. | Extrême | Peut déléguer les droits sur tous les domaines. |
| **Windows Server (AD)** | **Administrateurs du schéma** | Groupe de sécurité - Universel | Peut modifier le schéma AD. | Extrême | Modification du schéma = impact global irréversible. |
| **Windows Server (AD)** | **Admins du domaine / Domain Admins** | Groupe de sécurité - Global | Contrôle total du domaine. | Extrême | Peut élever tout compte au niveau admin. |
| **Windows Server (AD)** | **DnsAdmins** | Groupe de sécurité - Domaine local | Peut exécuter du code arbitraire en tant que SYSTEM sur le serveur DNS. | Extrême | Exploitation classique via DLL malveillante. |
| **Windows Server (AD)** | **Contrôleurs de domaine** | Groupe de sécurité - Global | Tous les DC du domaine. | Extrême | Contrôle complet sur l’annuaire AD. |
| **Windows Server (AD)** | **krbtgt** | Compte de service AD | Utilisé pour le chiffrement Kerberos. | Extrême | Attaque Golden Ticket (forgery de tickets). |
| **Windows Server (AD)** | **Group Policy Creator Owners** | Groupe de sécurité - Domaine local | Peut créer et modifier des GPO. | Élevé | GPO malveillante = exécution à distance. |
| **Windows Server (AD)** | **Backup Operators** | Groupe de sécurité - Domaine local | Peut sauvegarder et restaurer des fichiers. | Élevé | Accès indirect à des données sensibles. |
| **Windows Server (AD)** | **Account Operators** | Groupe de sécurité - Domaine local | Peut créer/modifier des comptes utilisateurs. | Élevé | Création de comptes administrateurs cachés. |
| **Windows Server (AD)** | **Server Operators** | Groupe de sécurité - Domaine local | Peut gérer les services et ressources du serveur. | Élevé | Arrêt, redémarrage, gestion de services sensibles. |
| **Windows Server (AD)** | **Hyper-V Admins** | Groupe de sécurité - Domaine local | Contrôle les machines virtuelles. | Élevé | Accès au disque virtuel = vol de credentials. |
| **Windows Server (AD)** | **Remote Management Users** | Groupe local | Peut exécuter des commandes à distance (PowerShell Remoting). | Élevé | Exécution de code via WinRM. |
| **Windows Server (AD)** | **RDP Users** | Groupe local | Peut se connecter à distance via RDP. | Moyen/Élevé | Accès distant si MFA absent. |
| **Windows Server (AD)** | **Replicator** | Groupe de sécurité - Domaine local | Utilisé pour la réplication de fichiers. | Élevé | Peut exfiltrer des données sensibles. |
| **Windows Server (AD)** | **SYSTEM** | Compte système | Compte système interne Windows. | Extrême | Exécution avec tous les privilèges. |
| **Windows Server (AD)** | **TrustedInstaller** | Compte système | Gère les installations Windows. | Très élevé | Peut modifier des fichiers système. |
| **Windows Server (AD)** | **Network Service** | Compte système | Service réseau avec privilèges limités. | Élevé | Escalade possible via services vulnérables. |
| **Windows Server (AD)** | **Local Service** | Compte système | Service local à permissions restreintes. | Moyen | Risque limité, mais exploitable localement. |
| **Windows 10** | **Administrateur local** | Utilisateur | Accès complet à la machine. | Extrême | Peut désactiver sécurité et exécuter code arbitraire. |
| **Windows 10** | **SYSTEM / TrustedInstaller** | Compte système | Contrôle total du système. | Extrême | Manipulation directe de fichiers système. |
| **Windows 10** | **Remote Desktop Users** | Groupe local | Accès RDP distant. | Élevé | Tentatives de brute-force RDP. |
| **Windows 10** | **Remote Management Users** | Groupe local | Exécution à distance via PowerShell. | Élevé | Exécution de commandes sans MFA. |
| **Windows 10** | **Power Users** | Groupe local | Droits élevés hérités des anciennes versions. | Moyen/Élevé | Peut exécuter ou installer des logiciels. |
| **Windows 10** | **Users (Builtin)** | Groupe local | Groupe de base des comptes locaux. | Moyen | Mauvaise configuration = élévation potentielle. |
| **Linux** | **root** | Utilisateur | Superutilisateur avec accès complet. | Extrême | Contrôle total du système. |
| **Linux** | **sudo** | Groupe | Peut exécuter des commandes en tant que root. | Très élevé | Mauvaise config sudoers = accès root. |
| **Linux** | **wheel** | Groupe | Peut utiliser `sudo` (selon la distro). | Très élevé | Membre = élévation directe. |
| **Linux** | **adm** | Groupe | Peut lire les logs système. | Moyen | Accès à journaux sensibles. |
| **Linux** | **shadow** | Groupe | Accès à `/etc/shadow`. | Extrême | Lecture des mots de passe hachés. |
| **Linux** | **operator** | Groupe | Peut effectuer certaines opérations système. | Élevé | Exécution de commandes critiques. |
| **Linux** | **staff** | Groupe | Peut installer des logiciels localement. | Élevé | Installation de binaires malveillants. |
| **Linux** | **docker** | Groupe | Contrôle indirect de root via conteneur privilégié. | Extrême | Accès root via `--privileged` ou volumes. |
| **Linux** | **lxd / libvirt / kvm** | Groupes | Accès à la virtualisation. | Élevé/Extrême | Accès à l’hôte via VM. |
| **Linux** | **systemd-journal** | Groupe | Peut lire journaux du système. | Moyen | Accès à des infos sensibles. |
| **Linux** | **bin, daemon, sys, sync, games, lp, mail** | Comptes système | Comptes système historiques. | Moyen | Risques faibles mais existants. |
| **Linux** | **nobody / nogroup** | Utilisateur/Groupe | Utilisé pour processus anonymes. | Moyen | Peut être détourné pour exécution anonyme. |
| **Linux** | **Comptes avec UID 0** | Utilisateur | Tout compte avec UID 0 = root. | Extrême | Contournement total des restrictions. |
| **Linux** | **Comptes de service (www-data, mysql, postfix, etc.)** | Utilisateur | Comptes utilisés par des services. | Moyen/Élevé | Vulnérables selon leur rôle et permissions. |

## **Kanban**

<img width="564" height="684" alt="Bankan_DOJO" src="https://github.com/user-attachments/assets/50694581-25c9-42ed-99e4-eee3253ed473" />


https://trello.com/b/WH7sXlZN/creation-de-lab

