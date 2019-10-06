# Validation des blocs et des transactions

Ce fichier explique comment valider les blocs et les transactions.

## Blocs

### Coinbase

Un bloc est toujours constitué d’au moins une transaction. Cette transaction doit être une transaction de génération (coinbase), et doit toujours être au début de la liste des transactions. Cette transaction spéciale créer des Ensicoin comme récomponse pour le mineur. Finalement, cette transaction doit être unique au sein d’un bloc.

### Chaînes

Il est possible de classer les blocs dans trois catégories :

1. Les blocs de la chaîne principale (la chaîne la plus longue au sens de la difficulté).
2. Les blocs d’ùne chaîne secondaire (une chaîne moins longue que la chaîne principale).
3. Les blocs orphelins (des blocs dont on ne connais pas au moins un des ancêtres).

Tous ces blocs (et chaînes) DOIVENT être stockés par les nœuds, car ils peuvent éventuellement devenir la chaîne principale.

Vous trouverez le premier bloc de la chaîne principale dans [ce fichier](consensus.md). Il est communément nommé le « genesis block ».

### Calcul de l’empreinte d’un bloc

Pour calculer l’empreinte (le hash) d’un bloc, il convient de suivre l’algorithme suivant :

1. Récupérer l’entête du bloc, c’est-à-dire tous les champs du message `block` moins la liste de transaction.
2. Calculer le hash de cet entête en utilisant l’algorithme SHA-256.
3. Calculer le hash de ce hash.
4. Encoder ce dernier en hexadécimal.

Un bloc n’a pas besoin de signature car il ne peut pas être modifié : il faudrait alors recalculer son `nonce`, ce qui est difficile.

### Validation

À la réception d’un bloc, il faut appliquer l’algorithme suivant :

1. Vérifier le format général du bloc.
2. Rejeter si ce bloc est déjà dans la base de données.
3. Rejeter s’il n’est dans aucune des trois catégories.
4. La liste des transactions doit être non-vide.
5. L’empreinte de l’en-tête du bloc doit être inférieur à l’objectif actuel.
6. La première transaction doit être une coinbase, les autres non.
7. Valider chaque transaction.
8. Vérifier que le bloc précédent est dans la branche principale (catégorie 1) ou dans une branche secondaire (catégorie 2). Sinon, ajouter ce bloc à la liste des blocs orphelins (catégorie 3).
9. Ajouter le bloc à la base de données. Trois cas : 1. le bloc agrandit la chaîne princpale ; 2. le bloc agrandit une chaîne secondaire ; 3. le bloc agrandit une chaîne secondaire et celle-ci devient la branche principale.
10. Pour le cas 1, ajouter à la branche principale :
    1. Enlever les transactions de la piscine des transactions.
    2. Transmettre le bloc aux autres nœuds.
11. Pour le cas 2, on ne fait rien.
12. Pour le cas 3, une branche secondaire devient la branche principale.
    1. Trouver le « fork-block », c’est-à-dire le bloc où la chaîne princpale et la chaîne secondaire se séparent.
    2. Modifier la chaîne princpale pour n’aller qu’à ce bloc.
    3. Ajouter les blocs de la branche secondaire à la branche princpale en validant.
    4. À partir du « fork-block », enlever les transactions de la psicine des transactions.
    5. Transmettre le bloc aux autres nœuds.
13. Pour chaque bloc orphelin dont le bloc précédent est celui-ci, recommencer l’algorithme.

## Transactions

Pour pouvoir valider une transaction, vous aurez aussi besoin de lire ce document : [scripts.md](scripts.md).

### La piscine des transactions ou « mempool »

La mempool est la liset des transactions valides mais qui ne sont pas encore incorporées dans la blockchain. IL est très important que tous les nœuds maintiennent une telle liste afin d’assurer la propagation des transactions.

## Calcul de l’empreinte d’une transaction

Pour calculer l’empreinte (le hash) d’une transaction, il convient de suivre l’algorithme suivant :

1. Récupérer le message `tx` de cette transaction.
2. Calculer le hash de ce message en utilisant l’algorithem SHA-256.
3. Calculer le hash de ce hash.
4. Encoder ce dernier en hexadécimal.

### Calcul des signatures

Il n’est pas possible de calculer la signature d’une transaction à partir de son empreinte étant donnée que la signature fait partie de l’empreinte elle-même (il peut y avoir plusieurs signatures s’il y a plusieurs entrées).

Pour calculer la signature d’une transaction, il faut donc remplacer le script des entrées par celui des sorties qu’elles dépensent.

### Validation

À la réception d’une transaction, il faut appliquer l’algorithme suivant :

1. Vérifier le format général de la transaction.
2. La transaction doit posséder au monis une entrée et une sortie.
3. Vérifier que cette transaciton n’est pas une coinbase, sauf si elle est au début d’un bloc.
4. La somme des sorties doit être strictement inférieure à celle des entrées.
5. Rejeter si on a déjà une transaction avec la même empreinte dans la chaîne princpale ou dans la piscine des transactions.
6. Pour toutes les entrées, si la sortie référencée l’est déjà dans une entrée de la chaîne principale alors rejeter.
7. Pour toutes les entrées, regarder dans la chaîne principale ou dans la piscine des transactions si la transaction référencée existe. Si ce n’est pas le cas, il s’agit d’une transaction orpheline. Il faut alors l’ajouter dans la liste des transactions orphelines sauf s’il y a déjà une transaction qui correspond à celle-ci dans cette liste.
8. Pour toutes les entrées, si une des sorties reférencée est une coinbase, alors il faut vérifier qu’au moins 42 blocs sont passés depuis son bloc.
9. Pour toutes les entrées, si la transaciton reférencée existe mais pas la sortie, alors rejeter.
10. Vérifier les scripts des entrées.
11. Ajouter à la piscine des transactions.
12. Transmettre la transaction aux autres nœuds.
13. Pour chaque transaction orpheline qui utilisent cette transaction, recommencer l’algorithme.

Le probleme de cet algorithme c'est que parcourir la chaine c'est extrement cher. Pour cela il existe une maniere de faire qui simplifie l'algorithme, celle des `utxo`. Ce n'est surement pas la seule maniere de faire mais c'est une maniere simple a implementer.

#### Utxo

Cela veux dire "unspent transaction output" (u - tx - o). Cela consiste a garder un index a jour de toutes les sorties non encore depensee dans la chaine. Les utxos sont donc `(hash, index)`. Il faut stoker avec ceci la valeur, le script, la hauteur et si c'est une coinbase.

On a donc une map de `utxo -> utxodata`.

Recevoir une nouvelle transaction entraine donc l'algorithme suivant:

- Sanity check (ne demande pas d'aller chercher les utxo)
    1. Verifier que la transaction possede une entree et une sortie
    2. Verifier que ce n'est pas une coinbase si elle n'est pas au debut d'un bloc
    3. Verifier que toute les sorties sont strictement positives
- Verification complete
    1. Rechercher pour chaque entree si l'utxo est dans la base de données, si l'utxo n'y ait pas cela signfie que une de trois choses: On a pas encore recu cette entrée, la transaction essaye de depenser de l'argent qu'elle n'as pas, ou elle essaye de depenser de l'argent deja depensee. On met alors la transaction de cote dans les orpheline et si on obtient le parent manquant on peut recommencer a la traiter. Si elle reste orpheline trop longtemps on peut alors l'enlever en se disant quelle est invalide.
    2. La somme des sorties doit etre strictement plus petite que la somme des entree
    3. Si une des sorties est une coinbase il faut regarder si 42 blocs sont passes
    4. Verifier tout les scripts
- Traitement de la transaction
    1. Rajouter la transcation a la piscine (mempool)
    2. Transmettre la transcation
    3. Pour chaque orpheline qui ne l'est plus, recommencer l'algorithme

Mais ce n'est pas tout ! Quand on obtient un bloc valide il faut penser a enlever de la base des utxo toutes les sorties reférencées par les entrées du bloc, sinon on aurait des transacations depensables plusieurs fois. Si on gere bien cet invariant on a jamais besoin d'aller regarder dans la chaîne si la transaction existe

Si la transaction est une coinbase, il convient de se référer à ce document [consensus.md](consensus.md), où des détails sont donnés sur la création des ENSICOIN.
