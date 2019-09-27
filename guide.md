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

Avant de lire la suite il faudrait que vous lisiez au moins tout les messages, ca vous aideras a comprendre.

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

Un tres bon systeme de base de données c'est le K-V DB, key value database. C'est donc tres proche d'un dictionnaire.

## La validation des ressources

Vous pouvez maintenant demander au gens leurs transactions et blocks ! Et voila vous etes prets a vous faire arnaquer par un mec qui depense votre argent. Attendez c'est pas ce que vous voulez ? Bah il faut donc vous assurer que tout le monde fait des operations valides. On va donc verifier tout cela !.

### Les `utxo`

Vous avez sans doute remarque que les entrées des transactions ne contiennent pas la valeur (sinon honte a vous, il fallait lire les messages !).
En effet celle ci est contenue dans la sortie de la transaction dont on a le hash, on a juste a la trouver c'est ca ? Ouais bah quand la blockchain fera 100 GB chercher dans chaque bloc ca va vous prendre un certain temps.

La solution c'est de garder en memoire toutes les sorties qui ne sont pas encore depensees, les unspent transaction output (`utxo`). Ensuite quand vous voyez que quelqu’un depense de l'argent vous pouvez supprimer ces transactions de votre liste, et quand quelqu’un gagne de l'argent vous pouvez l'ajouter a cette liste. Le l'algorithme de validation est plus explique dans [validation](validation.md).

### Les blocks

Apres etre capable de valider une tx, il faut pouvoir valider des blocs. Pour cela on commence par verifier quelques propriete sur le bloc puis on verifie chaque transaction. Et ensuite on supprime des utxos toutes les sorties references par le bloc. Mais tres vite il va arriver un problème....

## Recevoir des ressources

Comment on recois des ressources ? on les demande avec un `getdata` ! Souvent parceque on a recu un `inv` dont on ne connaisais pas certain hash. Une fois qu'on a recu la ressources on la valide comme decrit precedement, et on l'ajoute soit a la mempool si c'est une transcation, soit a la blockchain si c'est un bloc. Mais il y'a un probleme ici, la blockchain c'est la chaine avec le plus de travail sur laquelle tout le monde s'est accordee, mais pourquoi je stockerais un bloc qui n'est pas dans la main chain ?

La raison est tres simple: il pourrait devenir la main chain si suffisament de blocs s'y rajoutent. Il faut donc en permanence stocker tout les blocs, au cas ou ils deviennent principaux. Le seul probleme est donc qu'il ne faut pas dire que ces blocs font gagner de l'argent, ils sont juste la au cas ou.

Gerer une chaine qui a le plus de travail s'apelle un fork et c'est probablement la partie la plus difficile de la blockchain.

Une idee pour gerer ca c'est d'avoir une map (dictionnaire) `(hash de block) -> (travail pour arriver jusque la)`, notons la `work`
On peut alors mettre a jour grace a
```python
def update(new_block, work):
	work[hash(new_block)] = work[previous_block(new_block)] + work_of_block(new_block)
```

Comme ca on peut stocker le travail maximal actuel `current_work` et si `work[hash(new_block)] > current_work` il va falloir forker la chaine.

Comment fait on pour forker la chaine ? On a donc deux branches, disons `A` (actuelle) et `N` (nouvelle). Il faut maintenant trouver le point commun entre `A` et `N`. Pour cela on peut deja remonter la plus haute branche des deux jusqua on soit sur deux blocks de meme hauteur sur `A` et sur `C`. Apres il suffit de faire
```python
def find_common(blockA, blockN):
	while blockA != blockN:
		blockA = previous_block(blockA)
		blockN = previous_block(blockN)
	return blockA # On a blockA == blockN
```

Si on remonte trop loin et qu'on descends dans les hauteurs negatives c'est que les deux chaines n'avaient pas le meme premier bloc. On appelle ce bloc le genesis hash, et il ne peut pas etre un bloc valide vu qu'il n'est precede par personne. Ce bloc est decrit dans [consensus](consensus.md).

Quand on a trouve le point commun `blockCommun` il suffit d'annuler tout les blocs de `A` jusqu’a `blockCommun` et d'ajouter les blocks de `N` a partir de `blockCommun`. Et voila vous avez fait un fork ! Ca avait l'air simple ? Essayer d'implementer ca, y'a plein de details que j'ai passer sous le tapis.

## La propagation

Maintenant qu'on est capable de se mettre a l'etat le plus recent on devrais faire profiter les autres de cet etat. Du coup a chaque fois qu'on obtient des nouvelles ressources on doit envoyer un `inv` qui contient les hash de ces ressources a tout les noeuds auquels on est connectes pour leur demander si ils sont deja au courant, et si ils nous les demandent il faut les envoyer a l'aide de `block` ou `transaction`.

Il y'a un autre problème que je n'ai pas aborde: celui des ressources orphelines. En effet il se peut que une ressources se perde en route, ou qu'on nous envoye des ressources dans le mauvais ordre. Dans ce cas la on risque de refuser de valider quelques chose parceque on ne connait pas ses parents. Pour regler ceci on conseille de garder un petit registre des ressources orphelines et quand elle est orpheline depuis trop longtemps de la virer. Vous inquitez pas vous n'etes pas un monstre c'etait surement un enfant demoniaque.

## Le protocole gRPC

Avec les fonction définies [içi](https://github.com/EnsicoinDevs/ensicoin-proto), on peut faire communiquer des application externes avec son nœuds, comme un mineur ou un explorateur de blocs.
Pour cela il suffit d'implementer les fonctions définies avec une bibliothèque pour son langage, un certain nombres sont listées [içi](https://packages.grpc.io/).

C'est assez utile pour implementer un mineur pour creer des nouveau blocs, je vous propose d'aller regarder `ensicoin-simon` pour cela.

## Et ensuite ?

Voici quelques idées de choses à faire ensuite :

- Créer un portefeuille pour être capable de créer des transactions et de consulter son solde
- Créer un explorateur de block pour visualiser et analyser la blockchain de l’ENSICOIN

## Trouver de l’aide

Vous pouvez trouver de l’aide sur le serveur Discord de la promotion 2021 de l’Ensimag dans le salon #ensicoin, sur le Discord ensicoin, ou en ouvrant une nouvelle issue sur ce dépôt github.
