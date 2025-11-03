# Mise en place d’un bastion d’administration avec Guacamole

## Contexte et objectifs du projet

L’entreprise a été sollicitée pour concevoir une infrastructure réseau **robuste**, **performante** et **hautement disponible**, capable de supporter des applications critiques et temps réel.

L’objectif est d’assurer une **continuité de service** tout en garantissant la **sécurité des accès administratifs** aux serveurs internes.

---

## Qu'est-ce qu'un bastion informatique ?

Un **bastion** est un serveur spécialement sécurisé, placé entre un réseau interne protégé et un réseau non sécurisé, comme Internet.  
Il agit comme une **barrière de protection** en filtrant le trafic et en empêchant les intrusions malveillantes.

Il permet également de **contrôler et surveiller les accès** au réseau interne, en vérifiant l’identité et les autorisations des utilisateurs afin de **renforcer la sécurité globale**.

---

## Étude comparative : Teleport vs Apache Guacamole

| **Critères**       | **Teleport (solution retenue)**                          | **Apache Guacamole**                    |
|--------------------|----------------------------------------------------------|-----------------------------------------|
| **Sécurité**       | Zero Trust, MFA, certificats éphémères                   | Authentification classique, mot de passe |
| **Audit**          | Journaux + enregistrements vidéo                         | Logs texte simples                      |
| **Accès**          | SSH, RDP, Web, DB, K8s                                   | RDP, SSH, VNC                           |
| **Administration** | RBAC granulaire + accès temporaire                       | ACL basique                             |
| **Déploiement**    | Modulaire (Auth, Proxy, Node)                            | Stack Java + Tomcat + SQL              |
| **Objectif**       | Bastion d’administration complet                         | Passerelle d’accès Web                 |

---

## Architecture technique

| **Composant**       | **Rôle**                                    | **Configuration**                                                            |
|---------------------|---------------------------------------------|------------------------------------------------------------------------------|
| **VM Bastion (Guacamole)** | Serveur proxy d’accès et d’authentification | Debian, guacamole, IP `192.168.1.115`, `guacamole.jiji.local`           |
| **VM Client**       | Poste d’administration simulant un utilisateur externe | Windows 10, IP `192.168.1.178` 
| **VM Ressources**   | Windows Server 22 simulant des ressources externes | Windows Server 2022, '192.168.1.153' |

---

**Page d’accueil Guacamole accessible via navigateur depuis la VM client.**

<img width="1097" height="552" alt="GUI_Guacamole" src="https://github.com/user-attachments/assets/a461ff3f-db28-404a-9bbd-a2f8b7a8f4df" />

<img width="1096" height="552" alt="connexion_guacamole" src="https://github.com/user-attachments/assets/31bfe8fd-e02e-4ef2-9a93-1043d26a6684" />

---

**Création d'un utilsateur sur l'interface web de Guacamole.** 

<img width="1096" height="552" alt="creation_user" src="https://github.com/user-attachments/assets/72e6f331-be13-44d7-8d25-b97674ea1986" />

---

**Création d'une ressource externe sur l'interface web de Guacamole.** 

<img width="1096" height="552" alt="creation_ressource1" src="https://github.com/user-attachments/assets/3fc15245-162c-46cf-9305-5ebe245dc7e5" />

<BVR><BVR>

<img width="1096" height="552" alt="creation_ressource2" src="https://github.com/user-attachments/assets/9c33c5cb-4b2f-4293-9330-3230c4da22a7" />

<BVR><BVR>

<img width="1096" height="552" alt="ajout_machine_client" src="https://github.com/user-attachments/assets/1334f450-7f9a-4b4f-bf4b-02ca2e12ff63" />

---

**Activation du bureau à distance sur ma VM client**

<img width="1920" height="966" alt="activation_bureau_distance_vm_client" src="https://github.com/user-attachments/assets/6aabf8d8-3b53-45d2-831d-09671815b786" />

---

**Connexion  depuis ma VM client à mes ressources externes (VM WindServer22) en passant par le bastion Guacamole.**

<img width="1920" height="966" alt="connexion_depuis_client_a_bastion" src="https://github.com/user-attachments/assets/8800e6cd-ebd0-488e-bb73-690a6ed73307" />

---

**Connexion RDP réussie.**

<img width="958" height="946" alt="connexion_RDP" src="https://github.com/user-attachments/assets/8e23dac6-6126-48f8-b4b0-285fd7b02df3" />
