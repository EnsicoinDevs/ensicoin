# Guide d’implémentation

Ce guide sert à aider celles et ceux qui souhaitent créer une implémentation de l’ENSICOIN. Il donne des pistes et des directions qui ne sont pas forcément les meilleures. Rien ne vous oblige à le suivre.

## Quel langage choisir ?

Dans un projet de blockchain réel, il faudrait choisir un langage efficace, probablement du C. Cependant, ce projet est un projet à but éducatif. Les performances ne sont pas très importantes.

Vous pouvez donc choisir n’importe quel langage, vraiment. Vous pouvez choisir un langage où vous êtes à l’aise, ou encore un langage que vous souhaitez apprendre.

Nous avons des implémentation en Python, Rust, Go, JS, C par exemple !

## Par où commencer ?

Créer un nœud complet n’est pas une mince affaire, mais il faut bien commencer quelque part.

Un nœud complet est constitué des modules suivants :

- Gestion du réseau et des messages
- Gestion de la piscine des transactions (mempool)
- Gestion de la blockchain
- Gestion de la communication par [gRPC](https://grpc.io/) utilisant le [service standard](https://github.com/EnsicoinDevs/ensicoin-proto)
- Bootstraping (C'est pas encore tres bien defini)

Nous vous proposons de commencer par être capable de vous connecter à un autre nœud et d’échanger quelques messages. Cela simplifiera les tests par la suite, et sera motivant.

Ensuite, il deviendra possible de valider et de transmettre des transactions, c’est la base de toutes les cryptomonnaies : la propagation des transactions.

Afin de bien propager les transactions, vous devrez implémenter une mempool.

Finalement, vous pourrez vous concentrer sur l’implémentation de la blockchain.

## Le réseau et les messages

Le réseau est ici pair à pair. Cela signifie que votre nœud doit jouer à la fois le rôle d’un client et d’un serveur : il doit savoir accepter des connexions, et se connecter de lui-même à d’autres nœuds.

Le protocole réseau utilisé et le TCP via des sockets. Tout les langages possedent des bibliothèque pour utiliser celles ci. Une socket c'est quasiment comme un fichier sauf que l'on doit attendre pour lire son contenu, et on ecrit des octets bruts dedans.

Ce guide est un bon point de départ pour du python par exemple: https://realpython.com/python-sockets/.

On peut faciliment creer une socket en se connectant a quelqu’un, et on peut ecouter pour de nouvelles connections ce qui renvoie des sockets. Par exemple en python pour accepter de nouvelles connections on peut faire
```python
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind((0.0.0.0, TCP_PORT))
s.listen(1)
 
while 1:
	conn, addr = s.accept()
	handle(conn, addr) # handle est la fonction qui va gerer tout ca
```

Si vous n'avez pas compris ce code, lisez la documentation ou copiez le, mais la premiere etape est plutot de savoir se connecter a un autre noeud !

Ainsi voici la fonction la plus simple qui permet de se connecter a un autre noeud !
```python
def connect(addr, port):
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((addr, port))
	return s
```

Vous pouvez essayer ca en utilisant un des noeuds connus (regardez a la fin du fichier c'est explique !) .... mais ne vous etonnez pas il ne va rien se passer.

Pour qu'il se passe quelque chose il va falloir que vous puissiez envoyer des messages !

### Les messages

Tout les messages sont definis a partir des types primitifs suivants: `uint16`, `uint32`, `uint64`, `char/uint8`, `var_uint`.

Un bon point de depart c'est donc de convertir les types de votre langage en ces types, qui ne sont que des sequences de bits, et dans l'autre sens. Par exemple supposons que l'on ait un object python `bytes`
```python
def deserialize_uint16(raw_data):
	return (raw_data[0] << 8) + raw_data[1]
```
(Pour ceux qui se demandent c'est du big-endian, et si vous avez pas compris cette phrase cela ne devrait pas vous poser de problème)

On va pas vous ecrire tout le code, mais il faut faire de meme pour tout les types !

Inversement on peut ecrire par exemple
```python
def serialize_uint64(num):
	if num > 2**64:
		num = 2**64 - 1
	if num < 0:
		num = 0
	return num.to_bytes(8, byteorder='big')
```

Maintenant que l'on a tout ces blocs de base, on peut commencer a ecrire des fonctions qui creent des messages. Par exemple tout les message commencent par un header, defini par
| Field Size | Description | Data Type | Comments                                                                                                                                 |
| ---------- | ----------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 4          | magic       | uint32    | Cette constante magique permet de différencier plusieurs réseaux différents. De plus, elle peut servir de séparateur entre les messages. |
| 12         | type        | char[12]  | Une chaîne de caractères indiquant le type du message                                                                                    |
| 8          | length      | uint64    | Taille de la `payload` en octets.                                                                                                        |

Ceci est votre premier apercu de [messages.md](messages.md) qui sera votre ami pour un moment ! Ou pas.

On voit alors que le premier element du header fait 4 octets, c'est un `uint32` il s'appelle magic et sa valeur est marquee dans [consensus.md](consensus.md). Et oui la doc ca fait ouvrir plein de fichiers. Vous pensiez qu'on etait pas organise ?

La valeur suivante est une suite (un tableau) de 12 `char` qui donne le type du message. Il va y avoir beaucoup de tableau donc comprenez bien ce point. Et ensuite on a la longeure du message. Voici alors une fonction qui permet de creer un header a un message deja fabrique
```python
def create_header(magic, message_type, raw_message):
	return serialize_uint32(magic) + serialize_type(message_type) + serialize_uint64(raw_message.len())
```

Les fonctions de creation de message permetent donc de creer les bytes a envoyer aux autres noeuds !

Votre premiere mission est ainsi d'implementer le `whoami` et de voir ce qu'il se passe quand vous envoyez un `whoami` a un autre noeud. Si il ne se passe rien c'est que votre `whoami` n'est pas correct, ou que le noeud que vous avez contacte est casse (c'est fort possible aussi (deadlock hum hum, demandez a johyn)).

Une fois que vous aurez implementer les messages `whoami` et `whoamiack` vous pouvez essayer de faire une connection complete a un noeud et il devrais vous envoyer un `getblocks`.

Ensuite il va vous falloir implementer le protocole de ping: `2plus2is4` `minus1thats3`. Si on vous envoye le premier vous devez repondre avec le second (sinon par exemple mon noeud vous degage).

Vous pouvez implementer la suite des messages tout de suite, ou attendre d'en avoir besoin c'est comme vous le sentez. Ne vous inquitez pas, rien que cette partie prends deja un temps certain quand on a jamais fait de programme en reseau. Et un conseil: rajoutez bien des fonctions qui vous permetent de voir tout les details des envois de message, et peut etre meme le contenu. Cela vous sera tres utile.

Ensuite pour envoyer des message c'est aussi simple que `s.send(message_as_bytes)`. Il est conseille de faire une structure (`class` par exemple) autour de vos socket qui gere automatiquement les `whoami` et les ping. Et pensez au fonctions de debug ! vous en aurez besoin dans la suite !

Vous etes maintenant prets a gerer des données !

## Le Bootstraping : Comment trouver des amis ?

Faut que je le refasse mais flemme

Pour pouvoir decouvir d'autres pairs il faut déja être connecté à quelqu'un pour pouvoir demander d'autres adresse. C'est donc un peu circulaire, pour trouver d'autres pairs il être dans le réseau. Pour pallier ce problème il est défini un [protocole](decouverte_nœuds.md) sur IRC pour découvrir des nœuds. Chaque nœud publie son adresse en tant que son pseudo sur IRC. Il suffit alors de trouver un nœud et de lui envoyer un message `getaddr`.

## Le stockage

Avant de continuer, vous devez savoir que vous aurez très vite besoin de stocker des données (la blockchain, les transactions de la mempool…).

Au tout début, je vous conseille de simplement stocker ces données dans des dictionnaires, directement dans la mémoire.

```python
blockchain = {} # création du dictionnaire

blockchain["<hash du block>"] = block # stocker un block

block = blockchain.get("<hash du block>") # récupérer un block

blockchain.has_key("<hash du block>") # est-ce que je connais ce block ?

del blockchain["<hash du block>"] # supprimer un block
```

Comme vous pouvez le voir, cette structure de données est très pratique.

Stocker les blocks directement dans la mémoire implique une perte des données si le nœud est relancé. Il faudra donc plus tard passer à une autre solution, comme des fichiers ou une base de données.

## La propagation des transactions

À cette étape du guide, vous devriez pouvoir :

- Vous connecter à un nœud
- Accepter une connexion d’un autre nœud
- Recevoir et envoyer des messages `whoami` en suivant l’ordre défini dans le protocole

Pour être capable de propager correctement des transactions, vous allez devoir implémenter (au moins en partie) les messages suivants :

- `inv`
- `getdata`
- `notfound`
- `transaction`

Vous pouvez aussi décider d’implémenter le message `getmempool`, mais c’est un plus à ce stade.

Étant donné que vous n’êtes concernés que par les transactions, vous ne devez pas implémenter complètement les messages `inv` et `getdata`.

Pour commencer, vous pouvez tenter de propager des transactions sans les valider. Concrètement, voilà le scénario que vous devez gérer dans ce cas :

- Un nœud vous annonce des transactions (ou une seule) avec un message `inv`
- Vous regardez dans votre mempool si vous connaissez déjà ou non cette transaction
- Si vous ne la connaissez pas, vous envoyez un message `getdata` pour récupérer cette transaction

En théorie, vous allez recevoir un peu plus tard un message de type `transaction` juste après.

- Vous enregistrer cette transaction dans la mempool
- Vous propagez cette transaction aux nœuds voisins en utilisant un message `inv`

Une fois que cette partie du protocole fonctionne, il devient important d’être capable de valider les transactions. C’est la prochaine chose que vous devriez réaliser, et c’est probablement la plus complexe.

N’oubliez pas qu’une transaction peut arriver avant que l’une des transactions référencée dans ses entrées soit arrivée. Dans ce cas particulier, la transaction est dite « orpheline ». Vous ne devez cependant pas la marquer comme invalide, car vous recevrez peut-être la transaction manquante plus tard.

## La blockchain

Si vous avez bien implémenté les étapes précédentes, cette étape ne devrait pas être particulièrement difficile à réaliser.

À cette étape du guide, vous devriez pouvoir :

- Vous connecter à un nœud et accepter une connexion d’un autre nœud
- Recevoir et envoyer des messages `whoami`, `inv`, `getdata`, `notfound` et `transaction`
- Valider une transaction

Afin de maintenir une blockchain, votre nœud devra supporter les messages suivants :

- `block`
- `getblocks`

En plus de ça, vous devrez terminer l’implémentation des messages `inv` et `getdata`.

La validation des blocks étant plus simple à réaliser que celle des transactions, vous ne devriez pas reporter celle-ci à plus tard.

Voici, dans l’ordre, ce que vous pouvez faire :

- Terminer l’implémentation des messages `inv` et `getdata`
- Implémenter le message `blocks` - Sauvegarder les blocks - Sauvegarder les blocks orphelins - Gérer correctement la chaîne la plus « longue », et donc les « forks »
- Implémenter le message `getblocks`

Ce dernier message est celui qui va permettre à votre nœud de ce synchroniser au réseau. Il est très important, et vous permettra aussi de tester votre nœud sur un autre nœud.

Si vous être arrivé jusqu’ici, félicitation, votre nœud est fonctionnel ! \o/

## Le protocole gRPC

Avec les fonction définies [içi](https://github.com/EnsicoinDevs/ensicoin-proto), on peut faire communiquer des application externes avec son nœuds, comme un mineur ou un explorateur de blocs.
Pour cela il suffit d'implementer les fonctions définies avec une bibliothèque pour son langage, un certain nombres sont listées [içi](https://packages.grpc.io/).

## Et ensuite ?

Voici quelques idées de choses à faire ensuite :

- Créer un portefeuille pour être capable de créer des transactions et de consulter son solde
- Créer un explorateur de block pour visualiser et analyser la blockchain de l’ENSICOIN

## Trouver de l’aide

Vous pouvez trouver de l’aide sur le serveur Discord de la promotion 2021 de l’Ensimag dans le salon #ensicoin, sur le Discord ensicoin, ou en ouvrant une nouvelle issue sur ce dépôt github.
