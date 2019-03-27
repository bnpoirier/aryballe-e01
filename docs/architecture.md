# ARYBALLE PROJECT : Proposition technique
Par Brendan, Antonin, Charles, Landry

[TOC]

## Demande
Le CNRS demande à mettre en ligne une application web qui permettrait de visualiser des monuments historiques en 3 dimensions.

Cette plateforme web permettrait d'ajouter un fichier LIDAR afin de le convertir en carte à points, qui permettent la visualisation 3D dans le navigateur. 

Cette visualisation 3D peut être annotée mais des tracés peuvent également être effectué.

L'application web devrait accueillir chaque année 12 cartes supplémentaires pouvant elles-même contenir une centaine d'annotations.

Cette application est à destination de 50 chercheurs du CNRS et 2 nouvelles organisations devraient mettre en place ce système à la fin de l'année.

L'organisation souhaite en savoir plus sur la démarche à adopter pour le déploiement de cette application.


## Approche
A partir de la demande et des enjeux liés au contexte d'activité du CNRS, nous avons dégagé trois paradigmes à garder en tête :

1. Une organisation nationale sera aux commandes de la plateforme
2. Les données doivent être hébergeable sur le territoire de l'organisation
3. Permettre à l'organisation d'être autonome dans l'administration de la plateforme

Ainsi, nous proposons un système autonome à héberger **soi-même (self hosted)** afin de répondre aux exigences de chacun des acteurs du projet (CNRS et futures organisations) en leur laissant le choix de l'hébergement.

Par ailleurs, notre système suit les principes des `Microservices` qui morcellent notre application en plusieurs processus afin d'avoir un meilleur contrôle dessus et de répartir la charge.  


## Stack technologiques
### Front End
#### AngularJS / VueJS
L'application utilisera un framework appliquant une logique de "composants" qui permet d'améliorer la maintenabilité du projet sur la partie front. 
Nous avons choisi VueJS étant realtivement simple et ayant une logique de DOM virtuel.


#### Potree
L'application utilise la technologie **WebGL**, assez récente qui n'est pas supportée par tous les navigateurs. Afin de pouvoir utiliser la plateforme, l'utilisateur doit disposer d'une machine possèdant la configuration minimum suivante : 
* Système d'exploitation : Windows / Mac / Linux
* Navigateur minimum requis : Chrome 9 / Firefox 4 / Internet Explorer 11

Aperçu de la compatibilité de Potree : 
| Browser       | OS       | Result | Comments     |
| ---------     | -----    | ------ | --------     |
| Chrome 64     | Win10	   | Works  |              |	
| Firefox 58    | Win10    | Works  |              |	
| Edge          | Win10    | Not supported |       |
| IE 11         | Win7     | Not supported |       |
| Chrome        | Android  | Works | Reduced functionality due to unsupported WebGL extensions. |
| Opera         | Android  | Works | Reduced functionality due to unsupported WebGL extensions. |


### Back End
Nous avons choisi le couple **NodeJS / ExpressJS** car ils nous offrent de manière native un système de gestion de **flux** qui rend plus efficace l'upload de grands fichiers tels que des fichiers LIDAR ou Points Clouds.


### Bases de données
#### Stockage Annotations : CouchDB
Nous avons choisi de stocker les annotations dans une base de données **CouchDB** afin de profiter de la synchronisation des données **PouchDB** pour l'ensemble des clients.

Nous avons besoin d'une base de données **NoSQL** notamment pour la flexibilité des données. Notre base de données doit être capable de gérer des points (une seule coordonnée) mais aussi des tracés et des zones qui possèdent plusieurs coordonnées.


#### Stockage des fichiers statiques : Minio
Nous avons choisi un mode de stockage de type **Object Storage** pour les fichiers **Points Clouds** (Potree) et les fichiers **LIDAR**.

Nous avons donc choisi de mettre en place plusieurs buckets dédiés aux fichiers Points Clouds et aux fichiers LIDAR sur une serveur **Minio**. Chaque buckets de données est répliqué sur une machine différente.

Nous avons exclu les prestataires d'hébergement comme Amazon Web Services pour ne pas imposer de services payant et laisser le choix de l'hébergement des données aux organisations.


#### Stockage des données utilisateurs : PostgreSQL
Afin de stocker les données utilisateurs, nous avons choisi d'utiliser **PostgreSQL**.

La base de données stockera 3 tables :

- users
    - userId
    - login
    - public_key
    - laboratoryId
    - roleId
=> Données utilisateur

Le champ "public_key" contient la clé public de déchiffrage pour l'authentification du scientifique qui se fera par insertion d'une clé USB **FIDO U2F** lors de l'accès au site. En savoir plus sur le système de connexion, voir [Sécurité / Authentification](#securite-authentification)

- roles
    - roleId
    - name
=> Données droits utilisateurs.

- laboratory
    - laboratoryId
    - name
    - address
    - zipcode
    - city
    - telephone
=> Données laboratoires.


## Infrastructure

### Hébergement sur GitHub
La solution sera mis à disposition sur GitHub afin que le public puisse la télécharger et connaître la composition de la solution.

2 dépôts seront crées :
    - Dépôt **aryballe/aryballe** contiendra la solution
    - Dépôt **aryballe/server** contiendra la solution NodeJS/Express

### Application conteneurisée
Notre application suit une architecture microservices. L'application et chacun des services associés sont conteneurisés. Docker est donc nécéssaire pour faire fonctionner l'application.

### Architecture de la solution

**Bloc Minio :** Stockage de fichiers statiques (Lidar et Points Clouds)
**Bloc PostgreSQL :** Stockage en base de données des informations pour les utilisateurs
**Bloc CouchDB :** Stockage en base de données des annotations
**Bloc Potree Converter :** Service de conversion de fichier Lidar
**Bloc Web Application :** Serveur de la web application

![](https://i.imgur.com/DmaoWW9.png)

@TODO

### Configuration matériel minimale du serveur requise
### Ressources

- Structure des ressources pour notre organisation : 
    - Site web : Hébergement mutualisé 
    - Service de distribution de l'application => Available for cloud 
    - CI / CD => Available for cloud 

- Structure des ressources pour les organisations clientes : 
    - Mise en place :
        - Installation
        - Clés compatibles U2F
        - Création d'un sous domaine privé (Peut-être pas de notre ressort)
    - Stockage Lidar / Points clouds
    - Plateforme
    - BDD :
        - CouchDB : 1GB
        - PostgreSQL : 250MB
        - Traefic + PotreeConverter + NodeJS WebApp : 1 serveur dédié ou accessible en SSH
        - Minio : Obect Based Storage 1 ou 2 serveur(s) dédié(s) ou accessible(s) en SSH (2 * 250GB) => Stockage de Lidar `.las` et de Points Clouds `.bin`
    - Services : 
        - Conversion Lidar to Points clouds
        - Upload Lidar
        - Download Lidar
    - Hébergement :
        On propose 2 choix : 
        - Laboratoire universitaire (ou autre solution que l'organisation estime)
        - Hébergement tiers accessible en SSH -> Digital Ocean

### Installation
@TODO
1. S'assurer d'avoir les ressources necessaires en terme de serveur et de disques durs
2. Préparer les machines de réplications (pour Minio et CouchDB)
    - Installer des instances Minio / CouchDB
3. Pull le projet sur la machine principale
4. Exécuter `docker-compose` pour orchestrer les différents conteneurs de la machine.


## Sécurité / Authentification
Pour répondre à des problématiques de sécurité, nous avons pensé à sécuriser la plateforme de 2 manières différentes
- L'authentification par mot de passe
- L'authentification par clé USB

L'utilisateur de notre solution pourra opter une des deux manière ou les deux en même temps. Cela dépendra des exigences de l'utilisateur.

### Authentification par mot de passe
Un système d'identification classique par mot de passe. L'utilisateur a été préalablement inscrit par l'administrateur et se connecte en utilisant **identifiant** et **mot de passe**.

Point faible : L'utilisateur peut se faire voler son mot de passe. La sécurisation est donc plus faible que la proposition suivante.


### Authentification par clé USB (WebAuthn)
Afin d'assurer l'authentification des scientifiques sans pour autant avoir à saisir ses informations de connexion, nous avons pensé à une deuxième manière de se connecter : l'authentification par clé **USB U2F** (Universal 2nd Factor).

L'authentification est simple :
- L'utilisateur affiche le site sur son écran
- L'application demande sa connexion par le biais du navigateur
- L'utilisateur insère sa clé d'authentification U2F + Pose son doigt sur le lecteur d'empreintes de la clé USB.
- L'application l'authentifie.

Il est également possible de choisir un authentifiant différent d'une clé USB U2F comme **un smartphone Android 7.0+** ou bien **une montre connecté**.

L'authentification fonctionne par **chiffrement asymétrique** et requiert l'intervention des 3 parties suivants :
- Le Partie de confiance (**Relying Party**) : l'application NodeJS / ExpressJS
- Le Navigateur (**Browser**)
- L'authentifiant (**Authenticator**) : la clé USB U2F du scientifique

Lors de la connexion, notre application renvoi un "challenge", une donnée aléatoire de 16 bytes minimum. Celui-ci est chiffré par une clé privée enregistrée sur la clé USB U2F lors de l'inscription. Ce challenge chiffré est renvoyé à notre serveur d'application. Il est déchiffré par la clé public enregistrée lors de l'inscription. Si le challenge correspond, l'authentification est effectué.

Plus plus d'informations : 
https://fidoalliance.org/how-fido-works/


## Perspectives
À plus long terme nous avons imaginé des évolutions au projet.

### Fonctionnalités en réflexion
- Mettre en place un système de vérification de mise à jour qui permet de vérifier si l'instance du logiciel Aryball est à jour.
- Ajouter de nouvelles fonctionnalités : 
    - Interface de choix des cartes
    - Interface de choix des versions de cartes
    - Mettre à jour l'URL en fonction de l'angle de la caméra et de l'annotation sélectionnée pour faciliter la collaboration et le partage de lien
    - Interface de mise à disposition des cartes publiques
- Mettre en place le RDF sous la forme d'un JSON-LD afin d'établir des relations sémantiques entre les annotations. Les objets annotation seront au préalable définis selon le formalisme RDF. Ainsi lors de l'ajout de nouveaux objets au fur et à mesure des découvertes sur les cartes, les annotations seront identifiables sur la carte par des parsers comme étant des objets ayant rôle particuliers. 

```json
{
    "@context": "http://aryballe.org",
    "@type": "Annotation",
    "id" : "344",
    "position": {
        "@type": "3DCoordinates",
        "x": "40.75",
        "y": "73.98",
        "z": "37.57"
    },
    "name": "Sphinx",
    "map_id": "378340",
    "author" : {
        "id": "3146574",
        "name": "Champollion"
    }
}
```

### Maintenance évolutive
Le projet sera amené à être amélioré via des mises à jours que nous rendrons disponibles au client. Nous ne pourrons pas, de part l'infrastructure technique et une question d'éthique forcer les mises à jour du logiciel chez les organismes. Le principe est donc de rendre disponible les nouvelles versions sur nos serveurs. L'instance du logiciel client fera une simple requête sur nos serveurs pour vérifier qu'il ait la dernière version, sinon l'utilisateur en sera informé via une notification sur l'interface du logiciel.


## Licence

Nous proposons d'appliquer la licence GNU GPLv3 car nous souhaitons que la communauté puisse proposer des améliorations ou modifications sur notre système. Néanmoins nous ne souhaitons pas qu'une personne physique ou morale reproduise notre solution et la propose en `closed source`.