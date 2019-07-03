# Découverte des nœuds (bootstraping)

Afin de se connecter au réseau, il est nécessaire de connaître au moins un autre nœud. Une fois connecté à celui-ci, il est possible d’utiliser le message `getaddr` pour se connecter à d’autres nœuds.

Ce document présente une méthode utilisant Matrix permettant de découvrir des nœuds.

## Principe

Matrix est un protocole de chat décentralisé, l'idée est donc de se connecter a un salon nommé ensicoin, et dans la liste des personnes connectées il suffit de trouver un autre noeud. Pour cela le pseudo du noeud doit etre de la forme `422021_ip:port`.

## Détails

Le salon Matrix recommandé est `#ensicoin:matrix.org`. Pour communiquer avec matrix il faut alors faire des requetes HTTP avec des données en JSON.

Comment peut on faire de telles requêtes ? par exemple en python il y'a `requests`.

### Choix d'un 'homeserver'

Soit on peut faire tourner matrix en local grâce a [synapse](https://matrix.org/docs/projects/server/synapse), soit on peut utiliser par exemple [riot](riot.im) pour se créer un compte. Dans la suite on notera `{HOMESERVER}` le chemin vers l'API. Par exemple cela peut être pour les serveurs officiels (donc riot) `https://matrix.org/_matrix/client/r0`.

### Connexion et Token

Une fois un compte créer avec un mot de passe il convient de se connecter au reseau. Pour cela il faut faire une requête POST à `{HOMESERVER}/login`.

Par exemple avec curl `curl -XPOST -d '{"type":"m.login.password", "user":"example", "password":"wordpass"}' "{HOMESERVER}/login"`. La partie apres le `-d` est le JSON nécessaire a la requête. Cela retourne un dictionnaire qui contient `access_token`, que l'on note `{TOKEN}`. Ce token va servir d'identifiant a votre bot ** il doit rester secret. **

### Changement de son nom

Il y'a deux noms à un utilisateur, le `user_id` original, par exemple `@bob:matrix.org` et le displayname que l'on peut changer. 

Pour changer ce nom il faut faire un PUT à `{HOMESERVER}/profile/{user_id}/displayname?access_token={TOKEN}` avec en données `{"displayname":"un_nouveau_nom"}`.

### Connexion au salon #ensicoin:matrix.org

Pour se connecter au salon il suffit de faire un POST sans données à `{HOMESERVER}/join/%23ensicoin%3Amatrix.org?access_token={TOKEN}`. Toujours avec curl cela donne `curl -XPOST -d '{}' {HOMESERVER}/join/%23ensicoin%3Amatrix.org?access_token={TOKEN}`. En retour on a un dictionnaire qui contient `room_id` que l'on va noter `{ROOM}`

### Récuperation de la liste des membres

On a alors à faire un GET à `{HOMESERVER}/rooms/{ROOM}/members?access_token={TOKEN}`. Cela renvoye une [structure](https://matrix.org/docs/spec/client_server/r0.5.0#get-matrix-client-r0-rooms-roomid-members) qui contient entre autres les displayname de tout le monde.

Voilà, vous êtes maintenant prêts a découvrir des noeuds ! Pour plus d'informations sur l'API de matrix lisez le [guide](https://matrix.org/docs/guides/client-server-api) et la [reference](https://matrix.org/docs/spec/client_server/r0.5.0).
