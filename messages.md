# Messages

Ce document décrit les différents messages échangés entre les nœuds.

**Ce document est encore incomplet.** En effet, il y manque les messages permettant de découvrir les nœud du réseaux. Sans ces messages, un nœud restera seul ou au mieux connecté aux mêmes nœuds.

## Format général

Un message doit toujours suivre ce format :

```json
{
	"magic": 42,
	"type": "olala",
	"timestamp": 0,
	"message": {}
}
```

Le champ `magic` contient un nombre identifiant le réseau. Cela permet de s’assurer que le message est bien destiné à un nœud ENSICOIN. Ce nombre est donné dans le fichier [consensus.md](consensus.md).

Le champ `type` indique le type du message. Par exemple `whoami` pour indiquer son identité.

Le champ `timestamp` contient la date de création du message.

Finalement, le champ `message` contient le message en lui-même.

## Messages de contrôles

Ces messages servent à assurer le bon fonctionnement du réseau P2P.

### `whoami`

Lorsque qu’un client tente de se connecter à un nœud, il doit envoyer un message du type `whoami` :

```json
{
	"version": 0.1,
}
```

Ce message permet à l’autre nœud de savoir qui est le nœud qui se connecte. L’autre nœud devra alors répondre avec le même message afin d’établir la connexion.

Le champ `version` contient le numéro de version du protocole utilisé par le nœud.

## Blocks et transactions

Le partage de données (blocks / transactions) est géré par deux messages : `inv` et `getdata`.

Le message de type `inv` permet de transmettre les identifants de certaines ressources. Cela évite de transmettre les ressources complètes, ce qui serait trop lourd pour le réseau.

Le message de type `getdata` est en principe utilisé suite à la réception d’un message de type `inv`. Si l’une des ressources de ce message `inv` n’est pas connue, alors il faudra utiliser un message `getdata` pour récupérer la ressource complète.

Deux messages renvoient des messages `inv` :

- `getblocks`
- `getmempool`

Ces messages sont décrits plus bas.

### `inv`


```json
{
	"type": "b/t",
	"hashes": [],
}
```

Le champ `type` indique le type de ressource d’un message `inv` (`b` pour blocks ou `t` pour transactions).

Le champ `hashes` contient les identifiants des ressources (le hash de header d’un block, ou celui d’une transaction).


### `getdata`

Le message `getdata` demande à un nœud des données (blocks ou transactions). Ce message contient un message de type `inv`.

```json
{
	"inv": {}
}
```

Le champs `inv` contient un message de type `inv`. Cela permet de très facilement répondre à un message de type `inv`.

En réponse à ce message, un ou plusieurs messages du type `block`, `transaction` ou encore `notfound` peuvent être envoyés.

### `notfound`

Ce message est envoyé suite à un message `getdata`. Il indique qu’une ressource n’est pas connue par le nœud.

```json
{
	"type": "b/t",
	"hash": ""
}
```

Comme pour le message de type `inv`, il contient le type de la ressource et son identifiant.

### `block`

Ce message est décrit dans [blocks.md](blocks.md).

### `transaction`

Ce message est décrit dans [transactions.md](transactions.md).

### `getblocks`

Ce message est utilisé par un nœud pour demander un message de type `inv`. Il permet de récupérer les identifiants des blocks à partir d’un certain point dans la blockchain. Il est par exemple utilisé lors de la première connexion d’un nœud.

```json
{
	"hashes": [],
	"stop_hash": ""
}
```

Le champ `hashes` contient un tableau d’identifiants de blocs, triés du plus haut au plus bas. L’idée est que le nœud recevant se message répondra à partir du hash le plus grand qu’il connaît. Plus clairement, le but est de trouver le dernier block commun entre les deux nœuds. Si jamais le nœud recevant ne connait aucun des identifiants, alors il répondra à partir du genesis block, c’est-à-dire le premier block de la blockchain.

Le champ `stop_hash` indique à quel block arrêter de répondre (compris). Si ce champs est vide, TOUS les blocks connus devront être envoyés.

### `getmempool`

Ce message est utilisé par un nœud pour récupérer l’ensemble des transactions valides reçues par un nœud qui ne sont pas encore dans un block. Autrement dit, la mempool d’un nœud.

Comme pour `getblocks`, la réponse de ce message est un message de type `inv`.

```json
{}
```

Ce message ne contient aucun champ particulier.

## Exemples

Nous en avons terminé avec les messages. Voici maintenant quelques exemple d’échanges entre des nœuds.

### Nouvelle connexion d’un nœud

1. Le nouveau nœud envoie un message `whoami` à un autre nœud.
2. Cet autre nœud répond avec un message `whoami`. La connexion est établie.
3. Le nouveau nœud envoie un message `getblocks` vide à ce nœud pour effectuer sa première synchronisation.
4. Celui-ci répond avec un message `inv` contenant la liste des hashs de tous les blocks de la blockchain.
5. Le nouveau nœùd répond avec un message `getdata` contenant le message `inv` précédent.
6. L’autre nœud va répondre avec de nombreux messages `block`.
7. Le nouveau nœud peut aussi envoyer un message `getmempool` pour remplir sa mempool.

