# Découverte des nœuds (bootstraping)

Afin de se connecter au réseau, il est nécessaire de connaître au moins un autre nœud. Une fois connecté à celui-ci, il est possible d’utiliser le message `getaddr` pour se connecter à d’autres nœuds.

Ce document présente une méthode utilisant Matrix permettant de découvert des nœuds.

## Principe

Matrix est un protocole de chat decentralise, l'idee est donc de se connecter a un salon nomme ensicoin, et dans la liste des personnes connectes il suffit de trouver un autre noeud. Pour cela le pseudo du noeud doit etre de la forme `422021_ip:port`.

## Détails

Le salon Matrix recommande est `#ensicoin:matrix.org`. Pour communiquer avec matrix il faut alors faire des requetes HTTP avec des donnees en JSON.

Comment peut on faire de telles requetes ? par exemple en python il y'a `requests`.

### Choix d'un 'homeserver'

Soit on peut faire tourner matrix en local grace a [synapse](https://matrix.org/docs/projects/server/synapse), soit on peut utiliser par exemple [riot](riot.im) pour se creer un compte. Dans la suite on noteras `{HOMESERVER}` le chemin vers l'API. Par exemple cela peut etre pour les serveurs officiels (donc riot) `https://matrix.org/_matrix/client/r0`.

### Connection et Token

Une fois un compte creer avec un mot de passe il convient de se connecter au reseau. Pour cela il faut faire une requete POST a `{HOMESERVER}/login`.

Par exemple avec curl `curl -XPOST -d '{"type":"m.login.password", "user":"example", "password":"wordpass"}' "{HOMESERVER}/login"`. La partie apres le `-d` est le JSON necessaire a la requete. Cela retourne un dictionnaire qui contient `access_token`, que l'on note `{TOKEN}`. Ce token va servir d'identifiant a votre bot il doit rester secret.

### Changement de son nom

Il y'a deux noms a un utilisateur, le `user_id` original, par exemple `@bob:matrix.org` et le displayname que l'on peut changer. 

Pour changer ce nom il faut faire un PUT a `{HOMESERVER}/profile/{user_id}/displayname?access_token={TOKEN}` avec en donnees `{"displayname":"un_nouveau_nom"}`.

### Connection au salon #ensicoin:matrix.org

Pour se connecter au salon il suffit de faire un POST sans donnees a `{HOMESERVER}/join/%23ensicoin%3Amatrix.org?access_token={TOKEN}`. Toujours avec curl cela donne `curl -XPOST -d '{}' {HOMESERVER}/join/%23ensicoin%3Amatrix.org?access_token={TOKEN}`. En retour on a un dictionnaire qui contient `room_id` que l'on va noter `{ROOM}`

### Recuperation de la liste des membres

On a alors a faire un GET a `{HOMESERVER}/rooms/{ROOM}/members?access_token={TOKEN}`. Cela renvoye une [structure](https://matrix.org/docs/spec/client_server/r0.5.0#get-matrix-client-r0-rooms-roomid-members) qui contient entre autres les displayname de tout le monde.

Voila vous etes maintenant prets a decouvrir des noeuds ! Pour plus d'informations sur l'API de matrix lisez le [guide](https://matrix.org/docs/guides/client-server-api) et la [reference](https://matrix.org/docs/spec/client_server/r0.5.0).
