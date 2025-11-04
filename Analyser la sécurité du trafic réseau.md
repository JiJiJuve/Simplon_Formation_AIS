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
​
1. Capturer le processus `DORA` du protocole DHCP

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

5. Capturer une `requête DNS` et sa réponse

6. Qu’est-ce que le `DNS Spoofing` ? Comment s’en protéger ?

7. Qu’est-ce que `DNSSec` ? `DNS over TLS` ou `DNS over HTTPS` ?

8. Dans quels cas trouve-t-on du DNS sur TCP ?

9. Capturer un flux `HTTP`

10. Qu’est-ce que le `HTTP Smuggling` ? Donner un exemple de `CVE`

11. Comment mettre en place la confidentialité et l'authenticité pour HTTP ?

12. Qu’est-ce qu’une `PKI` ?

13. Capturer un `mot de passe` HTTP via le projet VulnerableLightApp.

14. Comment mettre en place la `confidentialité` pour ce service ?

15. Capturer un `handshake TLS`

16. Qu’est-ce qu’une autorité de certification (`AC`) racine ? Qu'est qu'une `AC intermediaire` ?

17. Connectez-vous sur `taisen.fr` et affichez la `chaine de confiance` du certificat

18. Capturer une authentification `Kerberos` (mettre en place le service si nécessaire), identifier l'`AS_REQ`, `AS_REP` et les messages suivants.

19. Capturer une `authentification RDP` (mettre en place le service si nécessaire), quel est le protocole d'authentification capturé ?

20. Quelles sont les attaques connues sur `NetLM` ?

21. Capturer une `authentification WinRM` (Vous pouvez utiliser EvilWinRM si nécessaire côté client.), quel est le protocole d'authentification capturé ?

22. Capturer une `authentification SSH` ou SFTP (mettre en place le service si nécessaire)

23. Intercepter un `fichier au travers du protocole SMB`

24. Comment proteger l'`authenticité` et la `confidentialité` d'un partage SMB ?

> [!TIP]
> Bonus : **Déchiffrer le traffic TLS** en important la clé privée du certificat dans Wireshark et **reconstituer le fichier** qui à transité sur le réseau à l'aide de Wireshark
