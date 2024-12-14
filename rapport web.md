# Rapport d'analyse réseau : HTTP/1.1 et HTTP/3 (QUIC)

Ce rapport analyse les échanges réseau capturés avec Wireshark en se concentrant sur deux protocoles applicatifs majeurs : **HTTP/1.1** et **HTTP/3 (QUIC)**. Nous détaillerons les étapes des échanges, les rôles des intervenants (client et serveur), et les particularités de chaque protocole.

---

## **Introduction**

### **HTTP/1.1**
Introduit en 1997, **HTTP/1.1** est l’une des versions les plus utilisées du protocole HTTP. Il repose sur le protocole de transport **TCP** pour garantir la fiabilité des échanges entre le client et le serveur. Cependant, il présente des limitations, notamment l’absence de multiplexage (une seule requête à la fois par connexion TCP).

### **HTTP/3**
**HTTP/3** est une évolution majeure reposant sur le protocole de transport **QUIC**, lui-même basé sur **UDP**. QUIC apporte des améliorations significatives :
- Multiplexage natif des requêtes/réponses.
- Réduction de la latence grâce au **0-RTT**.
- Intégration directe du chiffrement, éliminant la négociation TLS classique.

---

## **1. Analyse HTTP/1.1**

### **Capture analysée**
La capture montre une série de requêtes HTTP/1.1 de type **POST** avec des réponses **200 OK**, échangées entre un client (**192.168.1.2**) et un serveur (**192.168.1.1**).

### **Étapes de l'échange HTTP/1.1**
1. **Requête POST :**
   - Exemple de trame analysée : **406**.
   - **Détails des en-têtes HTTP :**
     - **Host** : `192.168.1.1`  
       - Spécifie le serveur ciblé.  
     - **User-Agent** : Informations sur le client (navigateur ou application).  
     - **Content-Type** : Type de données envoyées au serveur (`text/xml`).  
     - **Content-Length** : Longueur des données transmises dans le corps de la requête.

2. **Réponse 200 OK :**
   - Exemple de trame analysée : **406**.
   - **Détails des en-têtes HTTP :**
     - **Status Code :** `200 OK` (indique que la requête a réussi).
     - **Content-Type :** Type de données retournées au client.
     - **Content-Length :** Taille du contenu renvoyé.

3. **Contenu échangé :**
   - Requêtes POST envoyant des données à une ressource spécifique :  
     `/upnp/control/WANCommonIFC1` ou `/upnp/control/WANPPPConn0`.  
   - Le serveur répond avec des données correspondant à la requête.

### **Protocole de transport : TCP**
- HTTP/1.1 repose sur une connexion **TCP** persistante.
- **Caractéristiques observées :**
  - Une connexion TCP est utilisée pour plusieurs requêtes/réponses HTTP.
  - La fiabilité des échanges est garantie par les mécanismes de retransmission et d'acquittement de TCP.

---

## **2. Analyse HTTP/3 (QUIC)**

### **Capture analysée**
La capture montre des communications HTTP/3 reposant sur le protocole de transport **QUIC** entre un client (`2a02:a03f:c9c7:1800:...`) et un serveur (`2a00:1450:400e:805::2004`).

### **Étapes de l'échange HTTP/3**
1. **Établissement de la connexion QUIC :**
   - Exemple de trames analysées : **74** (Initial) et **91** (Handshake).
   - **Détails des trames :**
     - **Initial** :
       - Sert à négocier la connexion QUIC et initier l’échange de clés de chiffrement.
       - Contient les paramètres nécessaires au protocole (comme les ID de connexion : SCID et DCID).
     - **Handshake** :
       - Finalise l’établissement de la connexion sécurisée.
       - Le protocole QUIC intègre directement le chiffrement TLS dans cette phase.

2. **Envoi de données avec 0-RTT :**
   - Exemple de trames analysées : **77 à 81** (0-RTT).
   - **Optimisation observée :**
     - Avec le mode **0-RTT**, le client peut envoyer des données immédiatement après l'envoi de l'Initial.
     - Cela réduit la latence en évitant d’attendre la confirmation complète de la connexion.

3. **Échange de données chiffrées :**
   - Exemple de trames analysées : **86 à 89** (Protected Payload).
   - Les données applicatives HTTP/3 sont échangées sous forme de **Payload protégé**.
   - La capture montre que les flux sont multiplexés, permettant de gérer plusieurs requêtes/réponses dans une même connexion QUIC.

### **Protocole de transport : UDP**
- Contrairement à HTTP/1.1 qui repose sur TCP, HTTP/3 utilise **QUIC**, basé sur UDP.
- **Caractéristiques observées :**
  - Multiplexage natif des flux, chaque flux étant indépendant.
  - Réduction de la latence grâce à des mécanismes comme 0-RTT.
  - Absence de "head-of-line blocking" (blocage dû à la perte d’un segment dans TCP).

---

## **3. Comparaison intermédiaire entre HTTP/1.1 et HTTP/3**

### **Protocole de transport**
- **HTTP/1.1** repose sur **TCP**, garantissant la fiabilité mais introduisant une latence plus élevée en raison des mécanismes de contrôle (trois-way handshake, retransmissions, etc.).
- **HTTP/3** repose sur **QUIC (UDP)**, optimisé pour réduire la latence et améliorer les performances grâce au multiplexage et à l'intégration du chiffrement.

### **Latence**
- **HTTP/1.1** :
  - Nécessite une connexion TCP (three-way handshake).
  - Chaque requête/réponse est traitée de manière séquentielle.
- **HTTP/3** :
  - Avec QUIC, les flux sont multiplexés, permettant des transferts parallèles.
  - Le 0-RTT permet de réduire la latence lors des premières requêtes.

### **Multiplexage**
- **HTTP/1.1** : Une seule requête est traitée à la fois par connexion.
- **HTTP/3** : Plusieurs requêtes/réponses sont gérées simultanément dans une même connexion.

### **Chiffrement**
- **HTTP/1.1** : Repose sur TLS en couches distinctes (TLS + TCP).
- **HTTP/3** : Intègre TLS directement dans QUIC, réduisant la complexité et le temps de négociation.

---

## **4. Conclusion**

Cette analyse met en évidence les différences fondamentales entre **HTTP/1.1** et **HTTP/3** :
- HTTP/1.1 repose sur un protocole fiable (TCP) mais est limité par l’absence de multiplexage et une latence plus élevée.
- HTTP/3 introduit QUIC, un protocole moderne basé sur UDP, conçu pour améliorer les performances du Web, notamment grâce à des optimisations comme 0-RTT et le multiplexage natif.

### **Prochaine étape : Intégration de HTTP/2**
Une fois une capture HTTP/2 disponible, nous compléterons cette analyse en comparant HTTP/1.1, HTTP/2 et HTTP/3.

