## Critères

### Autorité
> Qui a autorité sur le projet (Une ou plusieurs organisations ou sociétés ? Qui maîtrise l'information ?)

Le client maîtrise l'information, il est capable d'aministrer les droits des utilisateurs et de diffuser ou non les cartes et annotations. L'information est maîtrisée par l'ensemble des personnes autorisées à y accéder, mais un adminisatrateur local à les droits d'administration.

**Par exemple :**
*Étape 1 :* Un laboratoire italien a besoin de mettre en place une plateforme de visualisation 3D et choisit notre solution. Cette plateforme serait accesible dans un premier temps seulement par les chercheurs italiens.

*Étape 2 :* Au moment de l'installation de la solution, l'administrateur definit les informations utilisateurs (login, mot de passe). Les utilisateurs pourront changer leur mot de passe par la suite.*

*Étape 3 :* Un chercheur **italien** dépose un fichier LIDAR : Ce fichier est stocké sur MINIO en `object storage` sur un serveur du pays (d'une machine universitaire ou chez un hébergeur national)*

*Étape 4 :* Un chercheur **italien** dépose une annotation : L'annotation est stocké sur un serveur CouchDB sur un serveur du pays (d'une machine universitaire ou chez un hébergeur national)*

*Étape 5 :* Un laboratoire **français** sollicite un accès aux données italiennes : L'administrateur récupère les emails des chercheurs français et transmet les login et les mots de passe. Ainsi le chercheur **français** pourra déposer du contenu sur la plateforme (fichier LIDAR ou annotation) selon les droits définis par l'administrateur italien*


### Résilience
> Tolérances aux pannes issues de bug, crash, comportement malicieux ou indisponibilité de services ?

La base de données des annotations propose un système de journal de transaction qui permet d'avoir un historique des requêtes sous forme de logs.

Notre application n'a pas besoin de "load balancing" l'exécution de l'application ne sera pas amené à faire face à de fortes charges.

Pour limiter les transits de Points Clouds, nous avons décider de les stocker dans le navigateur. Ainsi les chercheurs localisés dans des zones à faibles connexion internet ne seront pas ou peu impactés par cette connexion.

        @TODO
        **Response :**
            - S'appuyer sur des backup historiques (sauvegarde 1j, 1 semaine, 1 mois ...)
            - Système de log des transactions CouchDB
            - Pas besoin de load balancer car il n'y aura pas beaucoup de visiteurs
            - L'approche Microservice conduit à une décentralisation du risque technique


### Evolutivité
> Sous quelle forme le système va-t-il évoluer ?

Le système se reposera sur plusieurs solutions libres. L'application sera découpée en microservices, ainsi chaque service sera isolé et plus facile à faire évoluer. Une modification d'un service ne provoquera aucun problème sur le fonctionnement de l'application s'il respecte les spécificité d'input et d'output.

Par exemple, un administrateur système pourrait choisir de placer le service Minio (ou un autre service) sur un serveur différent afin d'alléger son serveur web.


### Répartition
> Où l’information est elle située ?

Les différents types d'informations sont distribués et/ou répliqués sur plusieurs disques. Les données sont répliquées mais toutes les copies restent sur le même serveur. 

Mais il est cependant possible de configurer Minio de manière à ce que les fichiers soient hebergés sur plusieurs serveurs. De même pour CouchDB.


### Organisation
> Les acteurs ont ils les moyens d’accéder à l’information ?
* Open Source
* Closed Source

Les acteurs sont les maîtres de leur installation. Ils sont donc l'accès aux informations. Chaque organisation est maîtresse des informations qu'elle manipule, elle peut les administrer et elle est la seule à y avoir accès.

@TODO

### Apprentissage
> Quelle est la nature des compétences nécessaires ?

Les compétences généralistes : 
- Installation de serveur
- SQL/NoSQL
- CouchDB / PouchDB
- Minio
- Docker
- NodeJS / ExpressJS

Compétences spécifiques : 
- /
@TODO

            
### Prix
> Combien et quand doit on payer ? pourquoi (logiciel, service)

Le prix varie si vous hébergez vous-même l'application ou si vous déléguez l'hébergement à un prestataire informatique.

**Dans le cas ou vous souhaiteriez déléguer l'hébergement à un prestataire.** Voici la grille tarifaire de DigitalOcean en envisageant une dépense minimum.

| Grille tarifaire pour un hébergement sur DigitalOcean | Coût / mois
|-------------------------------------------------------|----------------
| Hébergement web (avec accès SSH)                      | 5$/mois
| Hébergement Object Storage                            | 15$/mois
| Droplet Debian (CouchDB, PosgtreSQL, PotreeConverter) | 5$/mois
|                                                       |
| TOTAL                                                 | 15$/mois

Voici les dépenses concernant la promotion et le développement de l'application Aryballe.

| Coûts de développement de l'application               | Coût / mois
|-------------------------------------------------------|----------------
| Hébergement web (Site vitrine de promotion)                        | 5$/mois
| Hébergement Object Storage (Distribution de l'application)| 15$/mois
| Équipe produit (5 personnes à temps plein)            | 18250€/mois
|                                                       |
| TOTAL                                                 | 18270$/mois


### Dépendance
Dépendances ponctuelles sans impact économique, le logiciel dépend de plusieurs projets open source : 
    - CouchDB
    - PouchDB
    - Minio
    - Potree (ThreeJS, WebGL)
    - PostgrSQL
    - Traefic
    - Docker


### Sécurité
> Où se situe la sécurisation du système (cryptage) ?
    * Au repos (stockage)
    * A l’échange (communication)

L'accès à la plateforme nécessite une clé USB sécurisé par le protocole U2F.

Les données quant à elles, ne sont pas toutes chiffrées : Les annotations, qui représente la majorité des informations sensibles le sont.

**Au repos** les annotations et les nuages de points sont stockés dans le navigateur de l'utilisateur en clair et sont chiffrées en base de données.

**À l'échange** les informations sont chiffrées par le serveur.


### Redondance
> Comment l’information est-elle repliquée ?

Les fichiers Potree (.bins) sont repliquées par Minio sur différents disques de même que les fichiers LIDAR si 4 disques sont installées sur la machine.

Si l'administrateur installe plusieurs instances Docker de CouchDB, il peut créer un cluster et de dupliquer les annotations sur ces différentes instances. Si une seule instance est créée, il n'y aura alors pas de réplication.


### Durée du projet
> Dans quels termes la réalisation du projet s’inscrit elle ?
    * Court terme
    * Moyen terme
    * Long terme

Le projet s'inscrit à moyen terme.

### Interfaces
> Quelles sont les modalités d’accès aux informations ?
    * Format
    * API
    * Protocole

Les données ne sont pas accessibles en mode déconnecté. Les fichiers statiques (.bin et LIDAR) peuvent être requêté par le biais de l'API de Minio mais requiert d'être authentifié.

Les annotations sont récupérées par le serveur web au format JSON via l'API CouchDB

Les fichiers LIDAR et .bins sont récupérés dans leur format d'origine et retrouvés en requêtant le contrôleur Minio grâce au client JavaScript Minio qui communique avec le protocole s3 d'Amazon.


### « Formes » de stockage
> Sous quelle forme est stockée l’information ?
    * Block, Fichier, Clef-Valeur
    * Relationnelle, Orientée Ligne (SGBD), Orientée colonne
    * Objet, Document, Graphe & Triple Store
    * "Format" des données (JSON, csv)
    * « Forme » des informations (tabulaire, graph, cubique)
    * Portabilité des données ?

Les annotations sont stockées au format JSON sous la forme de documents
Les données utilisateurs sont stockées dans une base de données orientée ligne PostgreSQL
Les fichiers .bin et .las (LIDAR)sont stockés au format objet sur un serveur Minio.