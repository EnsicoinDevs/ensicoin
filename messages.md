# Messages

Ce document décrit les différents messages échangés entre les nœuds.

Tous les entiers sont stockés en [Big Endian](https://en.wikipedia.org/wiki/Endianness).

## Header d’un message

Un message doit toujours commencer par cette structure :

| Field Size | Description | Data Type | Comments                                                                                                                                 |
| ---------- | ----------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 4          | magic       | uint32    | Cette constante magique permet de différencier plusieurs réseaux différents. De plus, elle peut servir de séparateur entre les messages. |
| 12         | type        | char[12]  | Une chaîne de caractères indiquant le type du message                                                                                    |
| 8         | length      | uint64  | Taille de la `payload` en octets.                                                                                                        |
| length     | payload     | char\[]   | Le message proprement dit.                                                                                                               |

Le champ `magic` contient un nombre identifiant le réseau. Cela permet de s’assurer que le message est bien destiné à un nœud ENSICOIN. Ce nombre est donné dans le fichier [consensus.md](consensus.md).

## Structures communes

Cette section décrit des structures qui sont communes à plusieurs types de messages.

### `address`

Cette structure doit être utilisée quand l’adresse d’un nœud doit être partagée.

| Field Size | Description | Data Type | Comments                                                                                                                                          |
| ---------- | ----------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| 8          | timestamp   | uint64    | Timestamp standard UNIX indiquant la dernière fois où ce nœud a été actif.                                                                        |
| 16         | IPv6/4      | char[16]  | Une adresse IPv6. Pour transmettre une adresse IPv4, il faut utiliser [ce format](https://en.wikipedia.org/wiki/IPv6#IPv4-mapped_IPv6_addresses). |
| 2          | port        | uint16    | Le port réseau.                                                                                                                                   |

### Variable length integer (`var_uint`)

| Value                                    | Bytes Used | Format                                |
| ---------------------------------------- | ---------- | ------------------------------------- |
| >= 0 && \<= 252                          | 1          | uint8                                 |
| >= 253 && \<= 0xFFFF                     | 3          | 0xFD followed by the number as uint16 |
| >= 0x10000 && \<= 0xFFFFFFFF             | 5          | 0XFE followed by the number as uint32 |
| >= 0x100000000 && \<= 0xFFFFFFFFFFFFFFFF | 9          | 0XFF followed by the number as uint64 |

### Variable length string (`var_str`)

| Field Size | Description | Data Type | Comments                             |
| ---------- | ----------- | --------- | ------------------------------------ |
| 1+         | length      | var_uint  | Longueur de la chaîne de caractères. |
| length     | string      | char\[]   | La chaîne de caractères.             |

### `inv_vect`

| Field Size | Description | Data Type | Comments                                       |
| ---------- | ----------- | --------- | ---------------------------------------------- |
| 4          | type        | uint32    | Type de l’objet de ce vecteur de l’inventaire. |
| 32         | hash        | char[32]  | Hash de l’objet.                               |

Le champ `type` peut pour le moment contenir deux valeurs :

| Value | Name  | Description                          |
| ----- | ----- | ------------------------------------ |
| 0     | TX    | Le hash est celui d’une transaction. |
| 1     | BLOCK | Le hash est celui d’un block.        |

## Messages de contrôles

Ces messages servent à assurer le bon fonctionnement du réseau P2P.

### `whoami` et `whoamiack`

Lorsque qu’un client tente de se connecter à un nœud, il doit envoyer un message du type `whoami` :

| Field Size | Description | Data Type | Comments                                               |
| ---------- | ----------- | --------- | ------------------------------------------------------ |
| 4          | version     | uint32    | Le numéro de version du protocole utilisé par le nœud. |
| 8          | timestamp   | uint64    | Un timestamp UNIX standard en secondes.                |

Afin d’établir la connexion, les deux nœuds devront aussi échanger des messages de type `whoamiack`. Ce message ne possède pas de `payload`. Il suffit de définir `type` à `whoamiack`.

Voici le protocole utilisé lors de la connexion entre deux nœuds (un nœud local `L` se connecte à un nœud distant `R`):

    L -> R : Envoie un message whoami avec sa version.
    R -> L : Répond avec un message whoami avec sa version.
    R -> L : Envoie aussi un message de type whoamiack.
    R :      Défini la version du protocole au minimum des deux.
    L -> R : Envoie un message de type whoamiack.
    L :      Défini la version du protocole au minimum des deux.

Cela permet de s’assurer que les deux nœuds vont communiquer correctement.

### `getaddr` et `addr`

Le message `addr` permet de donner des informations à propos de nœuds connus du réseau.

| Field Size | Description | Data Type  | Comments                      |
| ---------- | ----------- | ---------- | ----------------------------- |
| 1+         | count       | var_uint   | Nombre d’adresses du message. |
| 22 \* ?    | addresses   | address\[] | Liste de nœuds.               |

Le message `getaddr` permet de demander à un ordre nœud des informations sur les nœuds du réseaux que ce nœud connait. En réponse à ce message, un message `addr` doit être envoyé. On considère qu’un nœud est vivant si on a reçu un message de lui il y a moins de trois heures. Passé ce délai, le nœud doit être oublié et ne doit donc pas être partagé.

Ce message ne contient aucun champ particulier.

## Blocs et transactions

Le partage de données (blocks / transactions) est géré par deux messages : `inv` et `getdata`.

Le message de type `inv` permet de transmettre les identifants de certaines ressources. Cela évite de transmettre les ressources complètes, ce qui serait trop lourd pour le réseau.

Le message de type `getdata` est en principe utilisé suite à la réception d’un message de type `inv`. Si l’une des ressources de ce message `inv` n’est pas connue, alors il faudra utiliser un message `getdata` pour récupérer la ressource complète.

Deux messages renvoient des messages `inv` :

- `getblocks`
- `getmempool`

Ces messages sont décrits plus bas.

### `inv`

| Field Size  | Description | Data Type   | Comments                            |
| ----------- | ----------- | ----------- | ----------------------------------- |
| 1+          | count       | var_uint    | Nombre d’entrées dans l’inventaire. |
| 36 \* count | inventory   | inv_vect\[] | Vecteurs de l’inventaire.           |

### `getdata`

Le message `getdata` demande à un nœud des données (blocks ou transactions). Ce message contient les mêmes champs qu’un message de type `inv`.

Ce message est généralement envoyé en réponse à un message de type `inv`.

En réponse à ce message, un ou plusieurs messages du type `block`, `transaction` ou encore `notfound` peuvent être envoyés.

### `notfound`

Ce message est envoyé suite à un message `getdata`. Il contient les mêmes champs qu’un message de type `inv`. Il indique que certaines ressources sont inconnues de ce nœud.

Il est envoyé en réponse à un message de type `getdata`.

### `block`

Ce message est envoyé en réponse d’un `getdata`, il représente un bloc. Pour avoir plus de détails sur les blocs, veuillez lire (ce fichier)[validation.md].

| Field Size | Description | Data type  | Comments                                                           |
| ---------- | ----------- | ---------- | ------------------------------------------------------------------ |
| 4          | version     | uint32     | La version de la transaction.                                      |
| 1+         | flags_count | var_uint   | Nombre de drapeaux.                                                |
| ?          | flags       | var_str\[] | Liste des drapeaux.                                                |
| 32         | prev_block  | char[32]   | Le hash du block précédent.                                        |
| 32         | merkle_root | char[32]   | Le hash de l’arbre de Merkle des transactions.                     |
| 8          | timestamp   | uint64     | Un timestamp standard UNIX correspondant à la création de ce bloc. |
| 4          | height      | uint32     | La hauteur du bloc.                                                |
| 4          | bits        | uint32     | Le target utilisé pour calculer ce bloc.                           |
| 8          | nonce       | uint64     | Le nonce utilisé pour générer ce bloc.                             |
| 1+         | txs_count   | var_uint   | Le nombre de transactions du bloc.                                 |
| ?          | txs         | tx\[]      | La liste des transactions du bloc.                                 |

### `tx`

Ce message est envoyé en réponse d’un `getdata`, il représente une transaction. Pour avoir plus de détails sur les transactions, veuillez lire (ce fichier)[validation.md].

| Field Size | Description   | Data type  | Comments                             |
| ---------- | ------------- | ---------- | ------------------------------------ |
| 4          | version       | uint32     | La version de la transaction.        |
| 1+         | flags_count   | var_uint   | Nombre de drapeaux.                  |
| ?          | flags         | var_str\[] | Liste des drapeaux.                  |
| 1+         | inputs_count  | var_uint   | Nombre d’entrées de la transaction.  |
| 37+        | inputs        | tx_in\[]   | Liste des entrées de la transaction. |
| 1+         | outputs_count | var_uint   | Nombre de sorties de la transaction. |
| 9+         | outputs_count | tx_out\[]  | Liste des sorties de la transaction. |

Une `tx_in` suit ce format :

| Field Size | Description     | Data type | Comments                                          |
| ---------- | --------------- | --------- | ------------------------------------------------- |
| 36         | previous_output | outpoint  | Pointeur vers la sortie que dépense cette entrée. |
| 1+         | script length   | var_uint  | La taille du script.                              |
| ?          | script          | uchar\[]  | Le script en lui-même.                            |

Un `outpoint` suit ce format :

| Field Size | Description | Data type | Comments                           |
| ---------- | ----------- | --------- | ---------------------------------- |
| 32         | hash        | char[32]  | Le hash de transaction référencée. |
| 4          | index       | uint32    | L’index de la sortie référencée.   |

Une `tx_out` suit ce format :

| Field Size | Description   | Data type | Comments                |
| ---------- | ------------- | --------- | ----------------------- |
| 8          | value         | uint64    | La valeur de la sortie. |
| 1+         | script length | var_uint  | La taille du script.    |
| ?          | script        | uchar\[]  | Le script en lui-même.  |

### `getblocks`

Ce message est utilisé par un nœud pour demander un message de type `inv`. Il permet de récupérer les identifiants des blocks à partir d’un certain point dans la blockchain. Il est par exemple utilisé lors de la première connexion d’un nœud.

| Field Size | Description          | Data Type  | Comments                                                               |
| ---------- | -------------------- | ---------- | ---------------------------------------------------------------------- |
| 1+         | count                | var_uint   | Nombre de hashs dans le champ `block locator`.                         |
| 32 \* ?    | block locator object | char[32][] | Liste de hashs partant du block le plus haut vers le genesis block.    |
| 32         | hash_stop            | char[32]   | Hash du dernier block désiré. Définir à 0 pour ne pas fixer de limite. |

Lorsqu’un nœud reçoit ce message, il doit parcourir le block locator jusqu’au premier hash connu. Ensuite, il doit envoyer un message de type `inv` contenant une liste de blocks qui commence juste après ce hash, et qui se termine à `hash_stop` ou au dernier block connu.

### `getmempool`

Ce message est utilisé par un nœud pour récupérer l’ensemble des transactions valides reçues par un nœud qui ne sont pas encore dans un block. Autrement dit, la mempool d’un nœud.

Comme pour `getblocks`, la réponse de ce message est un message de type `inv`.

Ce message ne contient aucun champ particulier.
