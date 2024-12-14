# Rapport d'analyse réseau : HTTP/1.1, HTTP/2 et HTTP/3 (QUIC)

Ce rapport documente l'analyse des trois versions majeures du protocole HTTP : **HTTP/1.1**, **HTTP/2**, et **HTTP/3 (QUIC)**. Nous détaillerons les étapes des échanges, les caractéristiques de chaque version et leurs différences, basées sur des captures réseau analysées avec Wireshark.

---

## **Introduction**

Le protocole HTTP est au cœur des communications sur le Web, évoluant pour répondre aux besoins croissants de rapidité et d’efficacité.

- **HTTP/1.1** : Une version largement adoptée depuis les années 1990, reposant sur le protocole de transport TCP.
- **HTTP/2** : Introduit en 2015, HTTP/2 améliore HTTP/1.1 avec le multiplexage, la compression des en-têtes, et la persistance des connexions.
- **HTTP/3** : La dernière version, basée sur QUIC (protocole UDP), réduit encore plus la latence grâce à l’intégration du chiffrement et à une gestion optimisée des flux.

En raison de limitations techniques, les traces HTTP/1.1 et HTTP/3 ont été capturées directement. Cependant, pour HTTP/2, nous avons utilisé une capture externe trouvée en ligne, car le décryptage TLS n'a pas pu être effectué sur nos captures.

---

## **1. Analyse HTTP/1.1**

### **Capture analysée**
La capture montre une série de requêtes HTTP/1.1 de type **POST** échangées entre un client (`192.168.1.2`) et un serveur (`192.168.1.1`), avec des réponses **200 OK**.

### **Étapes de l'échange HTTP/1.1**

1. **Requête POST :**
   - Exemple de trame analysée : **406**.
   - **Détails des en-têtes HTTP :**
     - **Host** : `192.168.1.1`  
     - **User-Agent** : Informations sur le client (navigateur ou application).  
     - **Content-Type** : Type de données envoyées au serveur (`text/xml`).  
     - **Content-Length** : Longueur des données transmises dans le corps de la requête.

2. **Réponse 200 OK :**
   - Exemple de trame analysée : **406**.
   - **Détails des en-têtes HTTP :**
     - **Status Code :** `200 OK` (indique que la requête a réussi).
     - **Content-Type** : Type de données retournées au client.
     - **Content-Length** : Taille du contenu renvoyé.

3. **Protocole de transport : TCP**
   - HTTP/1.1 repose sur une connexion **TCP** persistante pour garantir la fiabilité des échanges.

---

## **2. Analyse HTTP/2**

### **Capture analysée**
La capture montre une communication HTTP/2 entre un client (`10.9.0.2`) et un serveur (`139.162.123.134`). En raison de limitations techniques, cette capture a été trouvée en ligne.

### **Étapes de l'échange HTTP/2**

1. **Requête initiale en HTTP/1.1 et transition vers HTTP/2 :**
   - **Trame 1** : Requête **GET** pour `/robots.txt` envoyée en HTTP/1.1.
   - **Trame 2** : Réponse **101 Switching Protocols** pour passer à HTTP/2.

2. **Négociation des paramètres HTTP/2 :**
   - **Trames 3 à 5** : Frames spécifiques au protocole HTTP/2 :
     - `Magic` : Vérification d’initialisation HTTP/2.
     - `SETTINGS` : Échange des paramètres entre client et serveur.

3. **Requête et réponse pour `/robots.txt` :**
   - **Trame 6** : Frame `HEADERS` contenant une requête **GET**.
   - Réponse associée : **200 OK** avec une frame `DATA`.

4. **Requête et réponse pour `/humans.txt` :**
   - **Trame 8** : Frame `HEADERS` pour une requête **GET**.
   - Réponse associée : **404 Not Found**, indiquant que la ressource est introuvable.

5. **Frames de contrôle :**
   - **Trame 7** : Frame `WINDOW_UPDATE` utilisée pour gérer la taille de la fenêtre (contrôle de flux).

### **Caractéristiques de HTTP/2**
- **Multiplexage des requêtes** : Plusieurs requêtes/réponses échangées simultanément dans une seule connexion.
- **Compression des en-têtes** : Réduction de la taille des en-têtes grâce au mécanisme **HPACK**.
- **Frames spécifiques** : Utilisation de frames comme `HEADERS`, `DATA`, `SETTINGS`, et `WINDOW_UPDATE`.

---

## **3. Analyse HTTP/3 (QUIC)**

### **Capture analysée**
La capture montre une communication HTTP/3 reposant sur QUIC entre un client (`2a02:a03f:c9c7:1800:...`) et un serveur (`2a00:1450:400e:805::2004`).

### **Étapes de l'échange HTTP/3**

1. **Établissement de la connexion QUIC :**
   - Exemple de trames analysées : **74** (Initial) et **91** (Handshake).
   - **Frames importantes :**
     - **Initial** : Début de la session QUIC avec négociation des paramètres.
     - **Handshake** : Établissement de la connexion sécurisée avec chiffrement TLS intégré.

2. **Envoi de données avec 0-RTT :**
   - **Trames 77 à 81** : Optimisation **0-RTT**, permettant d’envoyer des données dès l’établissement initial de la connexion.

3. **Échange de données :**
   - Frames `Protected Payload` (exemple : **86 à 89**) contenant les données applicatives chiffrées.

### **Caractéristiques de HTTP/3**
- **Protocole de transport : QUIC** (basé sur UDP).
- **Multiplexage natif** : Plusieurs flux gérés simultanément dans une seule connexion.
- **Optimisations :**
  - Réduction de la latence grâce au 0-RTT.
  - Absence de "head-of-line blocking" (problème rencontré avec TCP).

---

## **4. Comparaison des protocoles HTTP**

| **Critère**               | **HTTP/1.1**            | **HTTP/2**                | **HTTP/3**               |
|---------------------------|-------------------------|---------------------------|--------------------------|
| **Protocole de transport** | TCP                     | TCP                       | QUIC (UDP)              |
| **Multiplexage**           | Non                     | Oui                       | Oui                     |
| **Compression des en-têtes** | Non                     | Oui (HPACK)               | Oui (QPACK)             |
| **Latence**                | Élevée                  | Moyenne                   | Faible                  |
| **Fiabilité**              | Fiabilité assurée par TCP | Fiabilité assurée par TCP | Fiabilité intégrée à QUIC |
| **Chiffrement**            | TLS (séparé)            | TLS (séparé)              | TLS intégré à QUIC      |

---

## **Conclusion**

L'analyse des trois versions du protocole HTTP met en évidence une évolution significative :
- **HTTP/1.1** est limité par une latence élevée et l'absence de multiplexage.
- **HTTP/2** améliore ces aspects en introduisant le multiplexage et la compression des en-têtes.
- **HTTP/3** va encore plus loin avec QUIC, basé sur UDP, réduisant la latence et intégrant des fonctionnalités avancées comme le 0-RTT.

Chaque version apporte des améliorations adaptées aux besoins croissants du Web moderne. Cette étude montre comment ces protocoles répondent aux enjeux actuels en termes de rapidité, sécurité et fiabilité.

