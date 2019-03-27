## Crit�res

### Autorit�
> Qui a autorit� sur le projet (Une ou plusieurs organisations ou soci�t�s ? Qui ma�trise l'information ?)

Le client ma�trise l'information, il est capable d'aministrer les droits des utilisateurs et de diffuser ou non les cartes et annotations. L'information est ma�tris�e par l'ensemble des personnes autoris�es � y acc�der, mais un adminisatrateur local � les droits d'administration.

**Par exemple :**
*�tape 1 :* Un laboratoire italien a besoin de mettre en place une plateforme de visualisation 3D et choisit notre solution. Cette plateforme serait accesible dans un premier temps seulement par les chercheurs italiens.

*�tape 2 :* Au moment de l'installation de la solution, l'administrateur definit les informations utilisateurs (login, mot de passe). Les utilisateurs pourront changer leur mot de passe par la suite.*

*�tape 3 :* Un chercheur **italien** d�pose un fichier LIDAR : Ce fichier est stock� sur MINIO en `object storage` sur un serveur du pays (d'une machine universitaire ou chez un h�bergeur national)*

*�tape 4 :* Un chercheur **italien** d�pose une annotation : L'annotation est stock� sur un serveur CouchDB sur un serveur du pays (d'une machine universitaire ou chez un h�bergeur national)*

*�tape 5 :* Un laboratoire **fran�ais** sollicite un acc�s aux donn�es italiennes : L'administrateur r�cup�re les emails des chercheurs fran�ais et transmet les login et les mots de passe. Ainsi le chercheur **fran�ais** pourra d�poser du contenu sur la plateforme (fichier LIDAR ou annotation) selon les droits d�finis par l'administrateur italien*


### R�silience
> Tol�rances aux pannes issues de bug, crash, comportement malicieux ou indisponibilit� de services ?

La base de donn�es des annotations propose un syst�me de journal de transaction qui permet d'avoir un historique des requ�tes sous forme de logs.

Notre application n'a pas besoin de "load balancing" l'ex�cution de l'application ne sera pas amen� � faire face � de fortes charges.

Pour limiter les transits de Points Clouds, nous avons d�cider de les stocker dans le navigateur. Ainsi les chercheurs localis�s dans des zones � faibles connexion internet ne seront pas ou peu impact�s par cette connexion.

        @TODO
        **Response :**
            - S'appuyer sur des backup historiques (sauvegarde 1j, 1 semaine, 1 mois ...)
            - Syst�me de log des transactions CouchDB
            - Pas besoin de load balancer car il n'y aura pas beaucoup de visiteurs
            - L'approche Microservice conduit � une d�centralisation du risque technique


### Evolutivit�
> Sous quelle forme le syst�me va-t-il �voluer ?

Le syst�me se reposera sur plusieurs solutions libres. L'application sera d�coup�e en microservices, ainsi chaque service sera isol� et plus facile � faire �voluer. Une modification d'un service ne provoquera aucun probl�me sur le fonctionnement de l'application s'il respecte les sp�cificit� d'input et d'output.

Par exemple, un administrateur syst�me pourrait choisir de placer le service Minio (ou un autre service) sur un serveur diff�rent afin d'all�ger son serveur web.


### R�partition
> O� l�information est elle situ�e ?

Les diff�rents types d'informations sont distribu�s et/ou r�pliqu�s sur plusieurs disques. Les donn�es sont r�pliqu�es mais toutes les copies restent sur le m�me serveur. 

Mais il est cependant possible de configurer Minio de mani�re � ce que les fichiers soient heberg�s sur plusieurs serveurs. De m�me pour CouchDB.


### Organisation
> Les acteurs ont ils les moyens d�acc�der � l�information ?
* Open Source
* Closed Source

Les acteurs sont les ma�tres de leur installation. Ils sont donc l'acc�s aux informations. Chaque organisation est ma�tresse des informations qu'elle manipule, elle peut les administrer et elle est la seule � y avoir acc�s.

@TODO

### Apprentissage
> Quelle est la nature des comp�tences n�cessaires ?

Les comp�tences g�n�ralistes : 
- Installation de serveur
- SQL/NoSQL
- CouchDB / PouchDB
- Minio
- Docker
- NodeJS / ExpressJS

Comp�tences sp�cifiques : 
- /
@TODO

            
### Prix
> Combien et quand doit on payer ? pourquoi (logiciel, service)

Le prix varie si vous h�bergez vous-m�me l'application ou si vous d�l�guez l'h�bergement � un prestataire informatique.

**Dans le cas ou vous souhaiteriez d�l�guer l'h�bergement � un prestataire.** Voici la grille tarifaire de DigitalOcean en envisageant une d�pense minimum.

| Grille tarifaire pour un h�bergement sur DigitalOcean | Co�t / mois
|-------------------------------------------------------|----------------
| H�bergement web (avec acc�s SSH)                      | 5$/mois
| H�bergement Object Storage                            | 15$/mois
| Droplet Debian (CouchDB, PosgtreSQL, PotreeConverter) | 5$/mois
|                                                       |
| TOTAL                                                 | 15$/mois

Voici les d�penses concernant la promotion et le d�veloppement de l'application Aryballe.

| Co�ts de d�veloppement de l'application               | Co�t / mois
|-------------------------------------------------------|----------------
| H�bergement web (Site vitrine de promotion)                        | 5$/mois
| H�bergement Object Storage (Distribution de l'application)| 15$/mois
| �quipe produit (5 personnes � temps plein)            | 18250�/mois
|                                                       |
| TOTAL                                                 | 18270$/mois


### D�pendance
D�pendances ponctuelles sans impact �conomique, le logiciel d�pend de plusieurs projets open source : 
    - CouchDB
    - PouchDB
    - Minio
    - Potree (ThreeJS, WebGL)
    - PostgrSQL
    - Traefic
    - Docker


### S�curit�
> O� se situe la s�curisation du syst�me (cryptage) ?
    * Au repos (stockage)
    * A l��change (communication)

L'acc�s � la plateforme n�cessite une cl� USB s�curis� par le protocole U2F.

Les donn�es quant � elles, ne sont pas toutes chiffr�es : Les annotations, qui repr�sente la majorit� des informations sensibles le sont.

**Au repos** les annotations et les nuages de points sont stock�s dans le navigateur de l'utilisateur en clair et sont chiffr�es en base de donn�es.

**� l'�change** les informations sont chiffr�es par le serveur.


### Redondance
> Comment l�information est-elle repliqu�e ?

Les fichiers Potree (.bins) sont repliqu�es par Minio sur diff�rents disques de m�me que les fichiers LIDAR si 4 disques sont install�es sur la machine.

Si l'administrateur installe plusieurs instances Docker de CouchDB, il peut cr�er un cluster et de dupliquer les annotations sur ces diff�rentes instances. Si une seule instance est cr��e, il n'y aura alors pas de r�plication.


### Dur�e du projet
> Dans quels termes la r�alisation du projet s�inscrit elle ?
    * Court terme
    * Moyen terme
    * Long terme

Le projet s'inscrit � moyen terme.

### Interfaces
> Quelles sont les modalit�s d�acc�s aux informations ?
    * Format
    * API
    * Protocole

Les donn�es ne sont pas accessibles en mode d�connect�. Les fichiers statiques (.bin et LIDAR) peuvent �tre requ�t� par le biais de l'API de Minio mais requiert d'�tre authentifi�.

Les annotations sont r�cup�r�es par le serveur web au format JSON via l'API CouchDB

Les fichiers LIDAR et .bins sont r�cup�r�s dans leur format d'origine et retrouv�s en requ�tant le contr�leur Minio gr�ce au client JavaScript Minio qui communique avec le protocole s3 d'Amazon.


### � Formes � de stockage
> Sous quelle forme est stock�e l�information ?
    * Block, Fichier, Clef-Valeur
    * Relationnelle, Orient�e Ligne (SGBD), Orient�e colonne
    * Objet, Document, Graphe & Triple Store
    * "Format" des donn�es (JSON, csv)
    * � Forme � des informations (tabulaire, graph, cubique)
    * Portabilit� des donn�es ?

Les annotations sont stock�es au format JSON sous la forme de documents
Les donn�es utilisateurs sont stock�es dans une base de donn�es orient�e ligne PostgreSQL
Les fichiers .bin et .las (LIDAR)sont stock�s au format objet sur un serveur Minio.