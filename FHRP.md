# Analyse de l’infrastructure

<img width="1553" height="730" alt="tp_simplon_FHRP_topo" src="https://github.com/user-attachments/assets/eed23447-d7e6-4353-8d9e-157e63ec2c41" />


### Analyser l'infrastructure et la configuration des équipements.

• Qu’est-ce que HSRP ? Proposer une définition simple.

HSRP (Hot Standby Router Protocol) est un protocole Cisco qui permet à plusieurs routeurs de se faire passer pour une seule passerelle, afin que si le routeur principal tombe en panne, un second prenne automatiquement le relais sans coupure pour les utilisateurs.​ 

---

• Pourquoi utilise-t-on HSRP et quel problème résout-il ? Expliquer l'intérêt de HSRP dans ce réseau.

HSRP est utilisé pour avoir une passerelle par défaut redondante, afin qu’en cas de panne du routeur principal, un autre prenne le relais sans coupure pour les utilisateurs.

Problème que HSRP résout :

  - Sans HSRP :
    • Les PC ont une seule gateway IP (un seul routeur).
    • Si ce routeur tombe, plus de sortie vers l’extérieur tant qu’on ne reconfigure pas les postes ou le réseau.
    ​
  - Avec HSRP :
    • Plusieurs routeurs partagent une IP virtuelle de passerelle.
    • L’un est « Active », un autre est « Standby » prêt à prendre la main automatiquement si l’Active tombe.

---


### Analyse de la configuration existante.

• Identifier les routeurs primaires et les routeurs de secours HSRP, quels sont leurs rôles respectifs ?

<img width="477" height="359" alt="tp_simplon_FHRP" src="https://github.com/user-attachments/assets/b2543ca3-787b-44c7-b805-b33e0aa56a21" />

- R1 = Router Active ; groupe 1 (172.30.128.251/24)
- R3 =Router Standby ; groupe 1 (172.30.128.253/24)

- R3 = Router Active ; groupe 2 (92.60.150.4/24)
- R2 = Router Standby ; groupe 2 (92.60.150.3/24)

---

• Noter les adresses IP virtuelles (VIP) et physiques (R1, R2, R3) utilisées dans les groupes HSRP, à quoi servent ces différentes adresses ?

Groupe 1 => IP Virtuelle : 172.30.128.254

IP Physique : 
- R1 = 172.30.128.251/24
- R2 = 172.30.128.252/24
- R3 = 172.30.128.253/24

Groupe 2 => IP Virtuelle : 92.60.150.1

IP Physique : 
- R1 = 92.60.150.2/24
- R2 = 92.60.150.3/24
- R3 = 92.60.150.4/24

L’adresse IP virtuelle est utilisée par les hôtes comme adresse de passerelle par défaut et “se déplace/permute” entre les routeurs selon l’état Active/Standby.

---

• Identifier les interfaces réseau participant à HSRP sur chaque routeur, leurs priorités, les délais et les autres paramètres HSRP configurés sur les routeurs. Que comprenez-vous ?

R1#show standby 
GigabitEthernet0/0/0 - Group 1
State is Active
Preemption enabled
Priority 120 (configured 120)
Hello time 3 sec, hold time 10 sec 

  - Hello time 3 sec
        ◦ Le routeur Active envoie un paquet « hello » toutes les 3 secondes aux autres routeurs du groupe HSRP.
        ◦ Ça sert à dire « je suis toujours là, je suis l’active ».
    ​
  - Hold time 10 sec
        ◦ Si le routeur Standby ne reçoit plus de hello de l’Active pendant 10 secondes, il considère que l’Active est mort.
        ◦ Au bout de ces 10 secondes, il peut alors prendre le rôle d’Active et la VIP bascule sur lui.
  

R3#GigabitEthernet0/0/1 - Group 2
State is Listen
Hello time 3 sec, hold time 10 sec
Preemption enabled
Priority 100 (default 100)

Avec la commande show standby, j’identifie les interfaces Gx/x participant à HSRP, IP Virtuelle (VIP), la priorité locale, les timers et la préemption. J’en conclus quel routeur est Active/Standby pour chaque groupe, comment est faite la redondance. 

---

### Configuration HSRP

• À l'aide des informations que vous avez collectées, proposer un guide de commandes de configuration HSRP. Expliquer brièvement le rôle de chaque commande utilisée. Identifier les éléments clés tels que le numéro de groupe HSRP, les adresses IP virtuelles, les priorités, les
délais, ainsi que les commandes permettant d'activer HSRP sur l'interface.

Configuration des adresses IP des interfaces physiques :
- R1(config)#interface gigabitEthernet 0/0/0
- R1(config-if)#ip address 172.30.128.251 255.255.255.0
- R1(config-if)#no shutdown

Configuration de l’adresse IP virtuelle :
- R1(config-if)#standby 1 ip 172.30.128.254

Configuration des priorités et activation de la préemption :
-R1(config-if)#standby 1 priority 120
- R1(config-if)#standby 1 preempt 

Configuration des minuteurs :
- R1(config-if)#standby 1 timers 4 12


