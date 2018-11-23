# Découverte des nœuds (bootstraping)

Afin de se connecter au réseau, il est nécessaire de connaître au moins un autre nœud. Une fois connecté à celui-ci, il est possible d’utiliser le message `getaddr` pour se connecter à d’autres nœuds.

Ce document présente une méthode inspiré de la méthode IRC pour découvrir des nœuds.

## Principe

L’idée est d’utiliser Discord pour découvrir les autres nœuds.

## Protocole

Tout serveur Discord possédant un salon textuel `#ensicoin` est compatible avec le protocole.

Bob est un nouveau nœud du réseau, et Alice un nœud déjà connecté.

```
Bob -> #ensicoin : [magic] hello_world_i_m_an_ensicoin_peer [address_bob]
Alice -> #ensicoin : [magic] hello [address_bob] please_connect [address_alice]
Bob : Se connecte alors à Alice en utilisant l’adresse de son message.
```

Le champ `magic` doit contenir la contante magique du réseau définie dans [ce fichier](consensus.md).

Les champs `address_???` sont des structures de type `address` comme défini dans [ce fichier](messages.md).
