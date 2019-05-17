# Ensicoin

Ensicoin est un projet à but éducatif. L’idée est de réaliser une crypto-monnaie simple, inspirée de Bitcoin, afin de mieux en comprendre le fonctionnement.

Ce dépôt contient les détails du protocole de l’Ensicoin. En théorie, les documents de ce dépôt devraient suffire pour créer un nœud capable d’interargir sans problèmes avec les autres nœuds.

## Documents

Les règles de base du consensus sont lisibles ici : [consensus](consensus.md).

Les messages échangés via le réseau sont décrits ici : [messages](messages.md).

Les règles de validation sont décrites ici : [validation](validation.md).

Finalement, les scripts sont détaillés ici : [scripts](scripts.md).

Un protocole de découverte du réseau utilisant Discord est défini ici : [découverte des nœuds](decouverte_nœuds.md).

Un guide d’implémentation : [guide](guide.md).

Le glossaire : [glossaire](glossaire.md).

## État du projet

### Nœuds

Voici un tableau récapitulant les fonctionnalités des implémentations connues :

| Dépôt                                                              | Handshake          | Synchronisation    | Validation         | gRPC               |
| ------------------------------------------------------------------ | ------------------ | ------------------ | ------------------ | ------------------ |
| [eccd](https://github.com/EnsicoinDevs/eccd)                       | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| [arcd](https://github.com/EnsicoinDevs/arcd)                       | :heavy_check_mark: | :heavy_check_mark: | :x:                | :heavy_check_mark: |
| [ensicoin-rust](https://github.com/EnsicoinDevs/ensicoin-rust)     | :heavy_check_mark: | :x:                | :x:                | :x:                |
| [ensicoin-python](https://github.com/EnsicoinDevs/ensicoin-python) | :heavy_check_mark: | :x:                | :x:                | :x:                |
| ensicoin-swift                                                     | :x:                | :x:                | :x:                | :x:                |

### Contrôleurs (ctl)

Pour administrer les nœuds, il est possible d’utiliser un de ces contrôleurs à partir du moment où le nœud supporte gRPC.

| Dépôt                                              | UI  |
| -------------------------------------------------- | --- |
| [arc-cli](https://github.com/EnsicoinDevs/arc-cli) | TUI |parallélisé
| [eccctl](https://github.com/EnsicoinDevs/eccctl)   | TUI |

### Mineurs

Les mineurs peuvent se connecter à un nœud afin de générer des blocs.

| Dépôt                                                            | Parallélisé        | gRPC               |
| ---------------------------------------------------------------- | ------------------ | ------------------ |
| [ensicoin-simon](https://github.com/EnsicoinDevs/ensicoin-simon) | :x:                | :heavy_check_mark: |
| [cuda-miner](https://github.com/EnsicoinDevs/cuda-miner)         | :heavy_check_mark: | :x:                |

### Wallets

Les wallets permettent d’échanger des ensicoins.

| Dépôt                                                      | Plate-forme |
|------------------------------------------------------------|-------------|
| [MaybeWallet](https://github.com/EnsicoinDevs/maybewallet) | Mobile      |

### Utilitaires

| Dépôt                                                                  | Fonction | Description          |
| ---------------------------------------------------------------------- | -------- | -------------------- |
| [ensicoin-explorer](https://github.com/EnsicoinDevs/ensicoin-explorer) | Explorer | Explorateur de blocs |

## Par où commencer ?

Si vous souhaitez simplement utiliser l’Ensicoin, vous pouvez choisir un nœud ou un wallet dans les tableaux ci-dessus.

Si vous souhaitez participer au projet, n’hésitez pas à contacter l’un des développeurs.
