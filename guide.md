# Guide d’implémentation

Ce guide sert à aider celles et ceux qui souhaitent créer une implémentation de l’ENSICOIN. Il donne des pistes et des directions qui ne sont pas forcément les meilleures. Rien ne vous oblige à le suivre.

## Quel langage choisir ?

Dans un projet de blockchain réel, il faudrait choisir un langage efficace, probablement du C. Cependant, ce projet est un projet à but éducatif. Les performances ne sont pas très importantes.

Vous pouvez donc choisir n’importe quel langage, vraiment. Vous pouvez choisir un langage où vous êtes à l’aise, ou encore un langage que vous souhaitez apprendre.

Dans la suite de ce guide, le langage python sera utilisé.

## Par où commencer ?

Créer un nœud complet n’est pas une mince affaire, mais il faut bien commencer quelque part.

Un bon point de départ est peut-être la réception des messages. Vous pouvez commencer par réaliser une fonction pour chaque message pouvant être reçu.

Par exemple :

```python
def handleWhoami(message):
	pass
```

En partant de là, vous vous rendrez vite compte de ce que vous devez coder :

- la validation du message
- le stockage de la blockchain et de la mempool
- l’envoi des réponses
- ...

Vous devriez relativement facilement pouvoir découper tout ça en plein de petites fonctions. Si vous ne savez pas faire l’une d’elle, revenez dessus plus tard.

## Le stockage

Très vite, vous aurez besoin de stocker des données (la blockchain, les transactions de la mempool…).

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

## Le réseau

Une autre question importante est la suivante : comment interargir avec le réseau ? En effet, c’est bien joli de savoir comment traiter les messages mais si on ne les reçoit jamais…

Cette étape, plus complexe, doit venir après les autres.

Pour communiquer avec les autres nœuds, il faut établir une connexion socket avec eux. Il existe des tas de guides très bien fait sur les sockets en python sur internet.

La difficulté ici est que le nœud est à la fois un serveur et un client. En effet, il doit pouvoir accepter les connexions d’autres nœuds, mais aussi pouvoir se connecter à d’autres nœuds.

Vous pouvez commencer par ne gérer que le serveur, c’est-à-dire le cas où d’autres nœuds souhaitent se connecter à votre nœud.
