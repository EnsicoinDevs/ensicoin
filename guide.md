# Guide d’implémentation

Ce guide sert à aider celles et ceux qui souhaitent créer une implémentation de l’ENSICOIN. Il donne des pistes et des directions qui ne sont pas forcément les meilleures. Rien ne vous oblige à le suivre.

## Quel langage choisir ?

Dans un projet de blockchain réel, il faudrait choisir un langage efficace, probablement du C. Cependant, ce projet est un projet à but éducatif. Les performances ne sont pas très importantes.

Vous pouvez donc choisir n’importe quel langage, vraiment. Vous pouvez choisir un langage où vous êtes à l’aise, ou encore un langage que vous souhaitez apprendre.

Dans la suite de ce guide, le langage python sera utilisé.

## Par où commencer ?

Créer un nœud complet n’est pas une mince affaire, mais il faut bien commencer quelque part.

Un nœud complet est constitué des modules suivants :

- Gestion du réseau et des messages
- Gestion de la piscine des transactions (mempool)
- Gestion de la blockchain

Nous vous proposons de commencer par être capable de vous connecter à un autre nœud et d’échanger quelques messages. Cela simplifiera les tests par la suite, et sera motivant.

Ensuite, il deviendra possible de valider et de transmettre des transactions, c’est la base de toutes les cryptomonnaies : la propagation des transactions.

Afin de bien propager les transactions, vous devrez implémenter une mempool.

Finalement, vous pourrez vous concentrer sur l’implémentation de la blockchain.

## Le réseau et les messages

Le réseau est ici pair à pair. Cela signifie que votre nœud doit jouer à la fois le rôle d’un client et d’un serveur : il doit savoir accepter des connexions, et se connecter de lui-même à d’autres nœuds.

Le protocole réseau utilisé et le TCP via des sockets. Il existe des librairies en python pour utiliser ce protocole.

Ce guide est un bon point de départ : https://realpython.com/python-sockets/.

La difficulté principale ici est d’être capable de gérer à la fois les connexions entrantes et sortantes. Pour faire ça, nous vous conseillons de créer une classe `Peer` ou `Node` par exemple. Cette classe encapsulera la connexion socket, et permettra de gérer un nœud entrant de la même façon qu’un nœud sortant.

Bien sûr, il y a quelques différences entre les nœuds entrants et sortants : par exemple, c’est au nœud entrant d’envoyer le message `whoami` en premier. Vous pouvez simplement ajouter un booléen `is_ingoing` dans votre classe pour savoir si un nœud est entrant ou sortant.

Vous vous rendrez compte plus tard que le fait de ne pas séparer les nœuds entrants des nœuds sortants dans votre implémentation va vous simplifier grandement le développement.

Afin de tester votre implémentation, vous pouvez tenter de vous connecter sur ce nœud : `78.248.188.120:4224`. Il attendra alors, comme convenu dans le protocole, un message `whoami` de votre part. Si vous envoyez ce message, il répondra avec un message `whoami`. Si vous arrivez jusqu’ici, alors votre implémentation est fonctionnelle.

Pour tester le réseau dans l’autre sens, c’est-à-dire de l’extérieur vers votre nœud, demandez à quelqu’un dans une issue github ou sur le salon #ensicoin du serveur Discord de la promo 2021 de l’Ensimag de tenter de se connecter à votre nœud.

Si votre nœud est fonctionnel dans les deux sens, félicitation.

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
- Implémenter le message `blocks`
	- Sauvegarder les blocks
	- Sauvegarder les blocks orphelins
	- Gérer correctement la chaîne la plus « longue », et donc les « forks »
- Implémenter le message `getblocks`

Ce dernier message est celui qui va permettre à votre nœud de ce synchroniser au réseau. Il est très important, et vous permettra aussi de tester votre nœud sur un autre nœud.

Si vous être arrivé jusqu’ici, félicitation, votre nœud est fonctionnel ! \o/

## Et ensuite ?

Voici quelques idées de choses à faire ensuite :

- Créer un portefeuille pour être capable de créer des transactions et de consulter son solde
- Créer un explorateur de block pour visualiser et analyser la blockchain de l’ENSICOIN

## Trouver de l’aide

Vous pouvez trouver de l’aide sur le serveur Discord de la promotion 2021 de l’Ensimag dans le salon #ensicoin, ou en ouvrant une nouvelle issue sur ce dépôt github.
