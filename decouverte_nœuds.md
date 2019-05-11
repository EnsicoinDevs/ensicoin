# Découverte des nœuds (bootstraping)

Afin de se connecter au réseau, il est nécessaire de connaître au moins un autre nœud. Une fois connecté à celui-ci, il est possible d’utiliser le message `getaddr` pour se connecter à d’autres nœuds.

Ce document présente une méthode utilisant IRC permettant de découvert des nœuds.

## Principe

L’idée est d’utiliser IRC pour découvrir les autres nœuds. Chaque nœud se connecte à un salon IRC et déféni son pseudo comme son adresse publique préfixée du nombre magique et d’un underscore (par exemple `422021_78.248.18.120:4224`). Il suffit alors de sélectionner un utilisateur au hasard dans le salon, et d’essayer de s’y connecter.

## Détails

Le salon IRC recommandé est hébergé sur [freenode](https://freenode.net/). Il s’agit de `#ensicoin`.
