# Script

Ce document détaille le langage des scripts situés dans les entrées et les sorties des transactions. L’idée est d’avoir un langage de script simple qui n’est pas turing-complet. Comme pour Bitcoin, on s’inspirera de [FORTH](https://www.forth.com/forth/).

L’idée est donc de faire un langage de script basé sur une pile, exécuté de droite à gauche qui se terminera forcément (pas de boucles, non turing-complet).

## Fonctionnement

Pour exécuter un script, il faut le lire de gauche à droite. En pratique, un script est constitué du script de la sortie concaténé à celui de l’entrée qui la dépense.

Sur le réseau, les scripts doivent être compilés en utilisant le format décrit ci-dessous.

## Mots-clés

Voici la liste des mots-clés du langage de l’ENSICOIN :

### Constantes

| Word     | Opcode | Hex     | Description                                                           |
| -------- | ------ | ------- | --------------------------------------------------------------------- |
| OP_FALSE | 0      | 00      | Pousse le nombre 0 en haut de la pile.                                |
| N/A      | 1 - 75 | 01 - 4b | Les _opcodes_ octets suivants doivent être placés en haut de la pile. |
| OP_TRUE  | 80     | 50      | Pousse le nombre 1 en haut de la pile.                                |

### Gestion de la pile

| Word   | Opcode | Hex | Description                  |
| ------ | ------ | --- | ---------------------------- |
| OP_DUP | 100    | 64  | Duplique le haut de la pile. |

### Logique binaire

| Word     | Opcode | Hex | Description                                                                                            |
| -------- | ------ | --- | ------------------------------------------------------------------------------------------------------ |
| OP_EQUAL | 120    | 78  | Remplace les deux valeurs du haut de la pile par `OP_TRUE` si elles sont égales, par `OP_FALSE` sinon. |

### Contrôle du flot

| Word      | Opcode | Hex | Description                                                                                                                                  |
| --------- | ------ | --- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| OP_VERIFY | 140    | 8c  | Marque la transaction comme invalide si le haut de la pile est à faux (résultat de l’opération `OP_FALSE`), enlève le haut de la pile sinon. |

### Crypto

| Word        | Opcode | Hex | Description                                                                                                                                                                                                                     |
| ----------- | ------ | --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| OP_HASH160  | 160    | a0  | Hash le haut de la pile avec RIPEMD-160.                                                                                                                                                                                        |
| OP_CHECKSIG | 170    | aa  | Utilise la clé publique qui est en haut de la pile pour vérifier que la signature située juste en-dessous est valide. Enlève ces deux valeurs de la pile. Agit comme `OP_TRUE` si la signature est valide, et `OP_FALSE` sinon. |

## Validation

Un script doit être considéré comme **invalide** s’il contient une clé publique qui n’est pas sous une forme compressée.

Ainsi, une clé publique DOIT commencer par `0x02` ou `0x03`, et la taille totale doit être de 33 octets.

## Signature

Pour calculer la signature d’une entrée, il faut tout d’abord calculer le double hash SHA-256 de la structure suivante :

1. `version`
2. `flags_count` puis `flags`
3. le double hash de tous les outpoints des entrées de la transaction
4. l’entrée actuelle moins le script
5. la valeur de la sortie dépensée par cette entrée
6. le hash de toutes les sorties de la transaction

On nomme ce double hash le `shash` (signature hash).

Il convient ensuite de signer ce shash puis de l’intégrer dans le script de l’entrée.

## Exemple

Ces mots-clés suffisent à réaliser une transaction basique de type P2PKH :

Script de la sortie : `OP_DUP OP_HASH160 <hash160(pubKey)> OP_EQUAL OP_VERIFY OP_CHECKSIG`

Script de l’entrée : `<signature> <pubKey>`

Sous forme hexadécimale, on obtient par exemple :

Script de l’entrée : `64 A0 14 89 AB CD EF AB BA AB BA AB BA AB BA AB BA AB BA AB BA AB BA 78 8C AC`
