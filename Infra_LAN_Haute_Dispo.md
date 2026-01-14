# Administration de l'infrastructure LAN & Haute disponibilité

Ce projet présente la conception et la configuration d’une infrastructure LAN hautement disponible réalisée sous Cisco Packet Tracer, capable de supporter des applications critiques et temps réel sans interruption de service.​
L’objectif est de mettre en œuvre les principaux mécanismes de redondance et de services d’infrastructure (VLAN, routage, FHRP, EtherChannel, DHCP, DNS, etc.) pour assurer la continuité de fonctionnement du réseau.

---

<img width="1351" height="714" alt="topo" src="https://github.com/user-attachments/assets/15cd2cfc-c779-4275-b77e-8ffe6b5702d2" />

---

## Sommaire

  - [Objectifs du projet](#objectifs-du-projet)
  - [Topologie et technologies utilisées](#topologie-et-technologies-utilisées)
  - [Plan d'adressage IP](#plan-dadressage-ip)
  - [Extraits de configuration](#extraits-de-configuration)
    - [Vérification HSRP – switch L3 actif (SW-CORE-1)](#vérification-hsrp--switch-l3-actif-sw-core-1)
    - [Vérification HSRP – switch L3 de secours (SW-CORE-2)](#vérification-hsrp--switch-l3-de-secours-sw-core-2)
    - [Vérification EtherChannel – agrégation de liens (PAgP)](#vérification-etherchannel--agrégation-de-liens-pagp)
    - [Vérification OSPF](#vérification-ospf)
    - [Passerelles L3, HSRP et relais DHCP](#passerelles-l3-hsrp-et-relais-dhcp)
    - [Service DHCP centralisé](#service-dhcp-centralisé)
    - [DNS et serveur web interne](#dns-et-serveur-web-interne)
    - [Scénarios de tests réalisés](#scénarios-de-tests-réalisés)
   

## Objectifs du projet

- Assurer la haute disponibilité de la passerelle par VLAN via HSRP (FHRP) sur les équipements L3.
​

- Mettre en place un routage inter‑VLAN (SVI + OSPF) pour permettre la communication entre tous les sous‑réseaux du LAN.
​

- Garantir la redondance des liens entre les commutateurs via EtherChannel (LACP/PAgP) pour éviter les coupures en cas de panne de lien.
​

- Centraliser les services d’infrastructure (DHCP, DNS, serveur web) avec un plan d’adressage IP structuré par VLAN et des ip helper‑address.
​

- Vérifier la tolérance aux pannes (perte d’un lien, d’un switch, d’un routeur) sans interruption pour les utilisateurs (ping, accès web par nom, etc.)

---

## Topologie et technologies utilisées




| Élément      | Rôle principal                                                                  |
| ------------ | ------------------------------------------------------------------------------- |
| VTP          | Gestion centralisée des VLAN sur l’ensemble des switches.​                      |
| VLAN + SVI   | Segmentation du réseau par services et inter‑VLAN via switch L3.               |
| EtherChannel | Agrégation de liens pour redondance et augmentation de bande passante.         |
| HSRP (FHRP)  | Passerelle par défaut virtuelle redondante par VLAN.                           |
| OSPF         | Routage dynamique entre routeurs / cœurs de réseau.                            |
| DHCP         | Attribution automatique des adresses IP par VLAN avec ip helper‑address.       |
| DNS          | Résolution des noms (accès au serveur web via nom de domaine).                           |
| Serveur Web  | Application test pour vérifier la connectivité applicative.                    |

---

## Plan d'adressage IP


<img width="513" height="321" alt="plan_adressage_IP" src="https://github.com/user-attachments/assets/afaf7363-92cb-433e-82e3-1693b7dce4c2" />


---

## Extraits de configuration

Les extraits suivants illustrent la mise en œuvre des principaux mécanismes de haute disponibilité et de services d’infrastructure dans ce projet : HSRP, EtherChannel, routage OSPF, DHCP, etc.

### Vérification HSRP – Switch L3 actif (SW-CORE-1)


<img width="548" height="98" alt="HSRP_Active" src="https://github.com/user-attachments/assets/81133820-d90a-4776-b68a-bf13a3f20cd4" />


- Ce cœur de réseau joue le rôle de switch L3 actif pour les VLAN 10/20/30, et assure la passerelle par défaut via la VIP 10.0.x.254.

---

### Vérification HSRP – Switch L3 de secours (SW-CORE-2)


<img width="544" height="96" alt="HSRP_Standby" src="https://github.com/user-attachments/assets/145196ca-eaee-481a-bd7a-94bef17034ea" />


---

### Vérification EtherChannel – agrégation de liens (PAgP)


<img width="413" height="242" alt="Etherchannel-SW1" src="https://github.com/user-attachments/assets/6925019d-597d-4c60-b5e6-fcab43e1c11c" />


<img width="402" height="242" alt="Etherchannel-SW2" src="https://github.com/user-attachments/assets/04a918cb-962d-4933-87f5-2dc5d0bcd91c" />


---

### Vérification OSPF


<img width="567" height="345" alt="ospf" src="https://github.com/user-attachments/assets/1a83c87e-dd38-42ab-ac92-6902dc8260fc" />


<img width="600" height="158" alt="ospf2" src="https://github.com/user-attachments/assets/bef0fcc1-cbe5-4747-8e79-e8bea9f87f71" />


---

### Passerelles L3, HSRP et relais DHCP


<img width="256" height="306" alt="interfaces-vlan" src="https://github.com/user-attachments/assets/2fa5268d-28d1-4bd6-bc9f-b0277a0c88ae" />


- Ces interfaces VLAN sur les switchs L3 assurent le routage inter-VLAN, la passerelle HSRP (VIP .254) et le relais DHCP vers le serveur 10.0.10.3 pour chaque sous-réseau.

---

### Service DHCP centralisé


<img width="662" height="514" alt="pool_dhcp" src="https://github.com/user-attachments/assets/756d9c22-e9b3-40b8-afe2-54e2dc44810b" />


- Le serveur DHCP central fournit un pool dédié par VLAN, avec distribution de la passerelle HSRP (.254) et du serveur DNS pour tous les clients du LAN.

---

### DNS et serveur web interne

- Le serveur DNS (10.0.10.3) résout le nom du serveur web interne afin que les clients accèdent au site par nom et non uniquement par adresse IP.  
- Le serveur web héberge une page de test permettant de valider la connectivité applicative depuis les différents VLAN.


<img width="660" height="357" alt="server_DNS" src="https://github.com/user-attachments/assets/8e610db0-c634-4062-b4a9-feba0ad05aad" />


<img width="673" height="318" alt="server_dns_web_IP" src="https://github.com/user-attachments/assets/170a544d-f02d-4a6c-90c6-009ef4bc66c0" />


<img width="671" height="329" alt="server_dns_web_nom" src="https://github.com/user-attachments/assets/35ec414f-9ef3-43e3-8b1b-b15488e99395" />


---

## Scénarios de tests réalisés

- Bascule HSRP : après l’arrêt de SW-CORE-1 (switch L3 initialement actif), SW-CORE-2 devient actif sur tous les VLAN tout en conservant la même VIP, ce qui maintient la connectivité des clients.

- SW-CORE-1 éteint :


<img width="917" height="710" alt="Test_hsrp_SW1_OFF" src="https://github.com/user-attachments/assets/af411a74-c0a1-46aa-b132-1e26cb42da62" />


- SW-CORE-2 devenu actif HSRP :


<img width="541" height="94" alt="Test_hsrp" src="https://github.com/user-attachments/assets/f21fda1a-81ae-4d5d-ab8f-5012889ced26" />


- DNS / Web : depuis un poste client, accès réussi au serveur web interne d’abord par adresse IP, puis par nom (résolution DNS) comme illustré dans la section « DNS et serveur web interne ».


- Routage inter-VLAN : test de ping entre deux PC situés dans des VLAN différents (par exemple un PC en VLAN 20 et un PC en VLAN 30), validant le rôle des switchs L3 comme passerelles routées entre les sous-réseaux.


<img width="418" height="323" alt="ping-intervlan" src="https://github.com/user-attachments/assets/62647700-a8a2-49af-b8f1-b0c3da5a57dc" />


