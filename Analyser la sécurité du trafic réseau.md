# Analyser la sécurité du trafic réseau avec Wireshark 
 

Il vous est demandé de procéder à l'analyse des protocoles utilisés sur le réseau et de détecter d'éventuelles faiblesses.

A partir du Lab installé, ajouter les services nécessaires et répondre aux questions suivantes.

---

### Rappels

* Quelle est votre adresse IP ? Quelle est sa classe (IPv4) ?

* Quel est votre masque de sous-réseau ?

* Quelle est l'adresse de votre passerelle ?

---


### Questions

0. Quels sont les `flags TCP` ?

Les flags TCP sont des indicateurs sur 1 bit présents dans l’en-tête de chaque segment TCP, utilisés pour gérer l’établissement, le suivi et la fin des connexions, ainsi que le contrôle de la transmission des données.

​
- Liste des principaux flags TCP :
  
    - SYN (Synchronize) : Drapeau utilisé pour initier l’établissement d’une connexion. Premier paquet de la séquence du 3-way handshake.
​
    - ACK (Acknowledgement) : Indique que le paquet est un accusé de réception. Présent dans presque toutes les étapes après l’établissement de la connexion.

    - FIN (Finish) : Signale la fin de la transmission de données. Utilisé pour terminer la connexion de manière ordonnée.
​
    - RST (Reset) : Réinitialise une connexion en cas d’anomalie ou lorsqu’une connexion non attendue est détectée.

    - PSH (Push) : Demande au récepteur de transmettre immédiatement les données à l’application, sans mise en mémoire tampon.

    - URG (Urgent) : Signale la présence de données urgentes qui doivent être traitées immédiatement.

    - ECE (ECN Echo) : Utilisé pour le contrôle de congestion explicite selon RFC 3168 (indique une compatibilité ECN).

    - CWR (Congestion Window Reduced) : Indique la réduction de la fenêtre de congestion lors du contrôle de congestion ECN.

  ---
​
1. Capturer le processus `DORA` du protocole DHCP

<img width="856" height="206" alt="DORA_DHCP" src="https://github.com/user-attachments/assets/e62d3042-0559-4566-b7b2-57ef507f411e" />

---

2. qu’est ce que le `DHCP Starvation` / `snooping` ? `Rogue DHCP` ?

<BVR><BVR>

- Le DHCP Starvation (« épuisement DHCP ») est une attaque qui consiste à inonder un serveur DHCP avec un grand nombre de requêtes DHCP portant des adresses MAC différentes (souvent générées ou usurpées). Le but est d’épuiser la plage d’adresses IP disponibles du serveur DHCP, empêchant ainsi les autres clients légitimes d’obtenir une adresse IP et d’accéder au réseau. Ce type d’attaque est souvent une première étape avant des attaques plus poussées, comme l’introduction d’un serveur DHCP pirate.

​
- Le DHCP Snooping est une fonctionnalité de sécurité proposée par de nombreux équipements réseau (commutateurs/switchs). Elle permet au switch de surveiller le trafic DHCP et de filtrer ou bloquer les réponses venant de serveurs DHCP non autorisés. Les ports réseau sont classés comme « de confiance » (trusted) ou « non fiables » (untrusted). Seuls les ports de confiance peuvent émettre des réponses DHCP, ce qui empêche qu’un appareil non autorisé sature le réseau avec de fausses configurations ou causes des attaques de type Rogue DHCP.

​
- Un Rogue DHCP (serveur DHCP pirate) est un appareil (ou logiciel) non autorisé sur le réseau qui agit comme serveur DHCP. Il peut attribuer des adresses IP et, surtout, fournir de mauvaises informations de configuration (passerelle, DNS…), ce qui redirige le trafic des victimes vers des destinations contrôlées par l’attaquant (pour intercepter ou manipuler le trafic). Cette technique facilite les attaques de type « Man-in-the-middle » ou des redirections frauduleuses, et elle peut aussi causer des défaillances réseau en distribuant des configurations invalides.

---
​
3. Que ce passe-t-il lors de l'execution de la commande `ipconfig /release` (windows) ? D’un point de vue sécurité quel peut etre l'enjeu ?

- La commande ipconfig /release sous Windows demande au client DHCP de libérer son adresse IP actuelle. Concrètement, elle envoie un message DHCP Release au serveur DHCP pour indiquer que l’adresse IP attribuée n’est plus utilisée, puis le système supprime cette adresse IP de son interface réseau. Après cette commande, le poste n’a plus d’adresse IP valide, ce qui interrompt sa connexion réseau tant qu’une nouvelle adresse n’est pas obtenue par un renouvellement DHCP (ex. via ipconfig /renew).

--> Enjeux de sécurité simples :

 - Perte temporaire de connexion : Votre appareil n’a plus d’adresse IP, donc il ne peut plus communiquer sur le réseau tant qu’une nouvelle adresse n’est pas reçue. Cela peut être utilisé pour interrompre temporairement la connexion d’un dispositif.

 - Possibilité d’usurpation IP : Si l’adresse IP libérée est rapidement attribuée à un autre appareil (malveillant par exemple), ce dernier pourrait usurper l’identité réseau de la machine précédente.

 - Perturbation volontaire : Si un attaquant peut exécuter cette commande à distance sur votre machine, il peut provoquer un déni de service local en coupant la connexion réseau.


---

4. Quelle fonctionnalité propose CISCO pour se prémunir des `attaques DHCP` ?

Cisco propose la fonctionnalité appelée DHCP Snooping pour se prémunir des attaques DHCP. Cette fonctionnalité agit comme un pare-feu entre les ports utilisateurs non fiables (non trusted) et les ports des serveurs DHCP fiables, empêchant notamment :

- Les faux serveurs DHCP (DHCP spoofing) de répondre aux clients.
- Les attaques par usurpation d'adresses IP ou de serveurs DNS.
- L'épuisement des ressources DHCP via des requêtes frauduleuses massives.

Concrètement, DHCP Snooping filtre le trafic DHCP en marquant certaines interfaces comme "trusted" (fiables) où seuls les serveurs légitimes peuvent envoyer des messages DHCP Offer, alors que les autres interfaces sont non fiables et bloquent ce type de messages. Il maintient aussi une base de données des associations IP-MAC pour vérifier la légitimité des communications.

De plus, on peut limiter le nombre de paquets DHCP Discover reçus sur une interface pour éviter la saturation, et activer des options telles que la vérification de l'adresse MAC source pour renforcer la sécurité.

---

5. Capturer une `requête DNS` et sa réponse

 <img width="1090" height="152" alt="dns" src="https://github.com/user-attachments/assets/d81f85b8-7c44-4977-af05-56bcd8b1243b" />

<BVR><BVR>

 <img width="491" height="293" alt="dns2" src="https://github.com/user-attachments/assets/eb9954d2-c8bb-4122-8d08-d29c26adb745" />

---

7. Qu’est-ce que le `DNS Spoofing` ? Comment s’en protéger ?

Le DNS Spoofing, ou usurpation DNS, est une attaque informatique où un attaquant manipule les réponses DNS pour rediriger les utilisateurs vers des sites frauduleux ou malveillants à leur insu. En interceptant ou en corrompant les requêtes DNS légitimes, l'attaquant peut fournir une fausse adresse IP à la place de la véritable, ce qui permet de rediriger le trafic vers des sites de phishing ou diffusant des malwares. Cette technique peut se faire notamment par empoisonnement du cache DNS, interception des requêtes en mode "Man-in-the-Middle", ou par la mise en place de faux serveurs DNS.

​
--> Comment se protéger du DNS Spoofing :

- Utiliser des protocoles de sécurité DNS comme DNSSEC (Domain Name System Security Extensions) qui authentifient la source des données DNS et empêchent l'insertion de données falsifiées.

- Configurer correctement les serveurs DNS pour limiter leur cache et appliquer des politiques de sécurité.

- Employer des réseaux sécurisés (VPN) pour réduire le risque d'interception des requêtes.

- Mettre à jour régulièrement les serveurs et les logiciels réseau pour corriger les vulnérabilités connues.

- Surveiller et détecter les anomalies dans le trafic DNS pour identifier rapidement des attaques potentielles.

- Utiliser des solutions de sécurité réseau qui filtrent ou bloquent les réponses DNS non sécurisées ou suspectes.

---

8. Qu’est-ce que `DNSSec` ? `DNS over TLS` ou `DNS over HTTPS` ?

- DNSSEC authentifient la source des données DNS et empêchent l'insertion de données falsifiées.
Il fonctionne en ajoutant une signature numérique aux réponses DNS. Cette signature est générée par le serveur DNS autoritaire qui détient l’information. Quand ton ordinateur reçoit la réponse, il vérifie cette signature à l’aide d’une clé publique, pour s’assurer que la réponse n’a pas été modifiée et vient bien de la source légitime. C’est un peu comme vérifier la signature sur un document officiel avant de lui faire confiance.

  - Le DNSSEC protège contre la falsification des réponses DNS.

  - Il garantit que l’adresse IP reçue pour un nom de domaine est authentique.

  - Il utilise la cryptographie (signatures numériques) pour ça.

  - C’est essentiel pour éviter les attaques qui redirigent vers des faux sites dangereux.

- DNS over TLS (DoT) et DNS over HTTPS (DoH) sont deux protocoles conçus pour sécuriser les requêtes DNS en chiffrant les échanges entre ton appareil et le serveur DNS afin de protéger ta confidentialité et éviter les interceptions ou manipulations.
  
- DNS over TLS (DoT)

   - Utilise le protocole TLS (le même que celui utilisé par HTTPS) pour chiffrer le trafic DNS classique qui est normalement en clair (non chiffré via UDP ou TCP).

   - Le trafic DNS est encapsulé dans une connexion sécurisée et authentifiée sur le port 853.

   - Empêche les attaques de type "man-in-the-middle" (interception/modification par un tiers) car seules les parties légitimes peuvent décoder ces échanges.

   - Cependant, le port spécifique utilisé peut indiquer qu'une requête DNS est en cours même si le contenu reste privé.

    ​

- DNS over HTTPS (DoH)

  - Transporte les requêtes DNS via le protocole HTTPS (port 443).

  - Le DNS est encapsulé dans des requêtes HTTP chiffrées, mélangé avec le trafic web classique, rendant plus difficile de distinguer les requêtes DNS.

  - Permet un chiffrement et une confidentialité comparables, avec une meilleure intégration dans les navigateurs et les applications modernes.

  - Facilite la contournement des blocages réseau qui ciblent spécifiquement le port DNS.

---

9. Dans quels cas trouve-t-on du DNS sur TCP ?

Le DNS utilise généralement le protocole UDP car les requêtes et les réponses sont petites et rapides, et UDP ne nécessite pas d'établissement de connexion, ce qui est efficace pour le trafic léger. Cependant, on trouve du DNS sur TCP dans certains cas spécifiques :

   - Quand la taille d'une réponse DNS dépasse la limite d'UDP (historique 512 octets, aujourd’hui souvent un peu plus grâce à EDNS), la réponse est tronquée en UDP et le client doit refaire la requête en TCP pour récupérer la réponse complète.

   - Pour les transferts de zone DNS entre serveurs DNS, qui impliquent l’envoi de gros volumes de données, TCP est systématiquement utilisé car il garantit la fiabilité et l'intégrité des données.

   - Si une requête DNS échoue en UDP (pas de réponse reçue), le client peut tenter une retransmission via TCP.

   - Certains services et configurations réseau imposent ou préfèrent TCP pour des raisons de sécurité ou de fiabilité.

En résumé, UDP est privilégié pour les requêtes rapides et légères, TCP est utilisé quand la taille de la donnée dépasse UDP ou quand la fiabilité de transfert est essentielle

---

10. Capturer un flux `HTTP`

<img width="1905" height="217" alt="http" src="https://github.com/user-attachments/assets/0c0c0eb0-7ea7-4783-9d91-09b0b0f03533" />

---

11. Qu’est-ce que le `HTTP Smuggling` ? Donner un exemple de `CVE`

--> HTTP Smuggling, ou HTTP Request Smuggling, est une technique d'attaque où un attaquant exploite des différences d'interprétation des requêtes HTTP entre des serveurs frontaux (comme des proxies, load balancers) et des serveurs backend. Ces serveurs peuvent traiter différemment les en-têtes HTTP comme Content-Length et Transfer-Encoding. Cela crée une désynchronisation dans la délimitation des requêtes, permettant à l'attaquant d'injecter une requête malveillante "cachée" dans une requête normale, pouvant contourner des contrôles de sécurité, voler des informations, ou détourner des sessions.

Un exemple typique est l'attaque CL.TE où le serveur frontal utilise Content-Length pour déterminer la fin de la requête alors que le serveur backend utilise Transfer-Encoding. L'attaquant construit une requête multicouche qui est partiellement traitée, laissant une partie malveillante non vue par un serveur mais interprétée par l'autre.

--> Concernant un exemple de CVE lié à HTTP Request Smuggling, un cas notable est :

   - CVE-2021-41773, une vulnérabilité dans Apache HTTP Server qui pouvait être exploitée via HTTP Request Smuggling, permettant un contournement de sécurité (source: communs CVE databases).

En résumé, HTTP Request Smuggling est une attaque qui exploite les différences dans le traitement des en-têtes Content-Length et Transfer-Encoding entre serveurs frontaux et backend, avec des impacts graves en sécurité web.

---

13. Comment mettre en place la confidentialité et l'authenticité pour HTTP ?

Pour assurer la confidentialité et l'authenticité dans HTTP, il faut mettre en place HTTPS (HTTP Secure), qui est HTTP encapsulé dans une couche de chiffrement SSL/TLS.
Mise en place de la confidentialité

   - HTTPS chiffre toutes les données échangées entre le client (navigateur) et le serveur grâce aux protocoles SSL/TLS. Le chiffrement empêche toute interception ou lecture des données par des tiers non autorisés.

Lorsqu'un client se connecte au serveur HTTPS, une négociation SSL/TLS a lieu : le serveur envoie son certificat numérique (émis par une Autorité de Certification reconnue), le client vérifie ce certificat pour valider l'identité du serveur, puis une clé de session symétrique est établie pour chiffrer toute la communication.

Mise en place de l'authenticité

   - Le certificat SSL garantit l'identité du serveur, évitant les attaques de type "man-in-the-middle".

   - Le protocole HTTPS utilise des signatures numériques dans le certificat pour s'assurer que le certificat est authentique et n'a pas été falsifié.

   - La validation du certificat par le client authenticité le serveur, assurant que l'utilisateur communique bien avec le site légitime.

Étapes pratiques

  - Obtenir un certificat SSL/TLS auprès d'une Autorité de Certification.

  - Installer ce certificat sur le serveur web.

  - Configurer le serveur pour activer HTTPS (ex. : liaison du certificat avec le port 443, choix des protocoles TLS à autoriser).

  - Pour renforcer la sécurité, mettre en place HTTP Strict Transport Security (HSTS) qui force les clients à n'utiliser que des connexions HTTPS.

  - Configurer les algorithmes cryptographiques et les versions TLS pour garantir un niveau élevé de sécurité.

Ainsi, HTTPS garantit la confidentialité en chiffrant les données et l'authenticité par la validation du certificat, protégeant à la fois les échanges et les utilisateurs.

​---


14. Qu’est-ce qu’une `PKI` ?

Une PKI, ou Infrastructure à Clé Publique (Public Key Infrastructure en anglais), est un ensemble de technologies, de politiques et de procédures destinées à gérer de manière sécurisée les clés cryptographiques et les certificats numériques. La PKI permet d'authentifier les identités des utilisateurs ou des appareils, de chiffrer les communications pour garantir leur confidentialité, et d'assurer l'intégrité et la non-répudiation des échanges numériques.

Concrètement, la PKI utilise la cryptographie asymétrique basée sur une paire de clés (une clé publique accessible à tous et une clé privée gardée secrète). Les certificats numériques, émis et signés par une autorité de certification (CA), servent de "carte d'identité numérique" permettant de vérifier l'identité des parties en communication. La PKI comprend aussi des composants comme une autorité d'enregistrement (RA), une base de données de certificats, et un système de gestion de certificats, qui ensemble gèrent le cycle de vie complet des certificats (émission, stockage, révocation).

---

16. Capturer un `mot de passe` HTTP via le projet VulnerableLightApp.

Requête HTTP sur Swagger -->

<img width="1024" height="768" alt="requette_HTTP_Swagger_VulnerableLightApp" src="https://github.com/user-attachments/assets/cd98f33d-060a-46bb-b705-8b20ec6727c8" />

<BVR><BVR>

Capture Login/Mot de Passe avec Wireshark -->

<img width="1024" height="768" alt="Capture_Login_MP" src="https://github.com/user-attachments/assets/e91f473c-bbd8-4866-9335-eb74751c48b2" />

---

17. Comment mettre en place la `confidentialité` pour ce service ?

Pour garantir la confidentialité du service HTTP de VulnerableLightApp, il est essentiel de passer à HTTPS en installant un certificat SSL/TLS. Cela chiffre toutes les communications et empêche l’interception des mots de passe.

18. Capturer un `handshake TLS`

<img width="986" height="438" alt="image" src="https://github.com/user-attachments/assets/d0b55657-d58c-444b-99f7-61b645f1a32c" />

---

19. Qu’est-ce qu’une autorité de certification (`AC`) racine ? Qu'est qu'une `AC intermediaire` ?

--> Une autorité de certification (AC) racine est l’autorité la plus élevée et la plus fiable dans la hiérarchie des certificats numériques. Elle est auto-signée, c’est-à-dire qu’elle signe elle-même son propre certificat, attestant ainsi sa propre identité. Ce certificat racine est préinstallé dans les navigateurs web et systèmes d’exploitation, ce qui permet de leur faire automatiquement confiance. Les AC racines délivrent des certificats intermédiaires ou directement des certificats utilisateur, mais pour des raisons de sécurité, elles émettent rarement des certificats finaux directement.

--> Une autorité de certification intermédiaire est une AC située entre l’AC racine et les certificats finaux (utilisateurs ou serveurs). Elle est signée par l’AC racine ou par une autre AC intermédiaire supérieure, et elle a pour rôle de délégué l’émission des certificats. Cela permet de créer une chaîne de confiance : le certificat final est émis par une AC intermédiaire, qui elle-même est signée par l’AC racine. Cette architecture ajoute une couche de sécurité, car la clé privée de l’AC racine est protégée hors ligne, et en cas de compromission d’une AC intermédiaire, les conséquences sont contenues.

---

21. Connectez-vous sur `taisen.fr` et affichez la `chaine de confiance` du certificat

<img width="935" height="696" alt="certificat_autorisation" src="https://github.com/user-attachments/assets/03ca2f73-0b02-44a4-8681-b9cf8dacc83f" />

---

22. Capturer une authentification `Kerberos` (mettre en place le service si nécessaire), identifier l'`AS_REQ`, `AS_REP` et les messages suivants.

<img width="800" height="600" alt="Authenti_Kerberos" src="https://github.com/user-attachments/assets/8fd69188-810a-4ee6-82c5-8aa55e4d19b9" />

<img width="800" height="600" alt="AS_REQ" src="https://github.com/user-attachments/assets/9f86b959-319f-4d81-acd2-1b1a4881e8da" />

<img width="800" height="600" alt="AS_REP" src="https://github.com/user-attachments/assets/e5a4e990-9742-4952-93ea-5f1f3df4fa49" />

---


23. Capturer une `authentification RDP` (mettre en place le service si nécessaire), quel est le protocole d'authentification capturé ?

<img width="475" height="276" alt="rdp2" src="https://github.com/user-attachments/assets/fc42a970-7f56-43b8-9352-4aba868ff427" />

<img width="800" height="600" alt="rdp4" src="https://github.com/user-attachments/assets/f120eafa-8056-48c4-98e5-4655fa733722" />

 --> Le protocole d’authentification capturé est TLS 1.2 pour la sécurité de la session RDP.
 
24. Quelles sont les attaques connues sur `NetLM` ?

Les attaques connues sur le protocole d’authentification NetLM/NTLM sont principalement les suivantes :

--> Attaque par relais NTLM (NTLM Relay)

  - L’attaquant intercepte une requête d’authentification NTLM d’un utilisateur légitime et relaie cette authentification vers un serveur cible sans connaître le mot de passe.

  - Cette attaque permet à l’attaquant de se faire passer pour l’utilisateur, obtenir un accès non autorisé et exécuter des actions avec ses privilèges.

  - Des variantes comme l’attaque PetitPotam exploitent des failles dans les services pour forcer une authentification NTLM sur des serveurs critiques (comme Active Directory Certificate Services) et obtenir un contrôle étendu sur le domaine.

--> Attaques "Pass-the-Hash" (PtH)

   - Extraction des hachages NTLM depuis la mémoire des machines (avec des outils comme Mimikatz), puis utilisation directe de ces hachages pour s’authentifier ailleurs sans connaître le mot de passe en clair.

--> Exploitation des vulnérabilités dans la gestion NTLM

   - Par exemple, une vulnérabilité zero-day récente dans Windows pouvait forcer la machine victime à envoyer ses hachages NTLM à un serveur malveillant simplement en affichant un fichier malveillant dans l’explorateur.

--> Man-in-the-Middle (MitM) et attaques de coercition

   - Techniques permettant à un attaquant sur le réseau local de forcer une réauthentification NTLM, par empoisonnement ARP, DNS spoofing, ou exploitation d’autres services vulnérables liés à NTLM.

---

25. Capturer une `authentification WinRM` (Vous pouvez utiliser EvilWinRM si nécessaire côté client.), quel est le protocole d'authentification capturé ?

<img width="800" height="600" alt="winRM" src="https://github.com/user-attachments/assets/df5d46bc-a1fd-4e29-985b-00903bcdd673" />

--> WinRM utilise NTLM pour l’authentification.

26. Capturer une `authentification SSH` ou SFTP (mettre en place le service si nécessaire)

<img width="1152" height="864" alt="ssh" src="https://github.com/user-attachments/assets/2c475b65-50e9-4cbc-8076-22d0d09d2f41" />

<img width="800" height="600" alt="ssh2" src="https://github.com/user-attachments/assets/8ed82697-ef7f-40da-a243-fd4fd032356f" />

27. Intercepter un `fichier au travers du protocole SMB`
    
<img width="1152" height="864" alt="smb" src="https://github.com/user-attachments/assets/e83b096f-f62a-48f8-80d5-954d5024c7ac" />

<img width="800" height="600" alt="smb2" src="https://github.com/user-attachments/assets/8365df68-08c4-4efb-9e8e-07945744b4b7" />

29. Comment proteger l'`authenticité` et la `confidentialité` d'un partage SMB ?

> [!TIP]
> Bonus : **Déchiffrer le traffic TLS** en important la clé privée du certificat dans Wireshark et **reconstituer le fichier** qui à transité sur le réseau à l'aide de Wireshark
