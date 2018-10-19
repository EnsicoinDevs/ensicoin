# Blocs

Un bloc est composé de deux parties : Son en-tête et les transactions. Une transaction est identifiée par son empreinte, un bloc par l'empreinte de son header.

## Format général

```json
{
	"header": {
		"version": 0,
		"flags": [
			"b",
			"olali"
		],
		"hashPrevBlock": "",
		"hashTransactions": "",
		"timestamp": 0,
		"nonce": 42
	},
	"transactions": []
}
```

La liste des transactions est dans cet exemple vide, mais elle doit toujours contenir une (et une seule) (et commencer) par une transaction de génération (coinbase). Cette transaction spéciale créer des ENSICOIN comme récompense pour le mineur ([voir ici](transactions.md)).

On peut classer les blocs dans trois catégories :

1. Les blocs de la chaîne principale (la plus longue)
2. Les blocs d’une chaîne secondaire (une chaîne moins longue que la chaîne principale)
3. Les blocs orphelins (des blocs dont on ne connais pas au moins un des ancêtres)

Tous ces blocs (et chaînes) DOIVENT être stockées par les nœuds, car ils peuvent éventuellement devenir la chaîne principale.

## Calcul de l'empreinte d’un bloc

Pour calculer l'empreinte d’un bloc (son identifiant) il convient de suivre l’algorithme suivant :

1. Construire la chaîne de caractères suivante :
	1. Ajouter `version`
	2. Ajouter `flags` en collant les valeurs une à une (sans utiliser de caractère de séparation)
	3. Ajouter `hashPrevBlock`, `hashTransactions`, `timestamp` et `nonce`
2. Hacher cette chaîne de caractère avec l’algorithme SHA-256
3. Hacher l'empreinte obtenu à nouveau
4. Encoder cette dernier empreinte en hexadécimal

Il n’y a pas besoin de signer un bloc car il ne peut pas être modifié : il faudrait recalculer le `nonce`, ce qui devrait être difficile.

## Règles de validation

1. Vérifier le format
2. Rejeter si ce bloc est déjà dans la base de données
3. Rejeter s’il n’est dans aucune des trois catégories
4. La liste des transactions doit être non-vide
5. L'empreinte de l’en-tête du bloc doit être inférieur à l'objectif actuel
6. Le timestamp du bloc doit être inférieur à 2 heures dans le futur
7. La première transaction doit être une coinbase, le reste ne doit pas l’être
8. Valider chaque transaction ([voir ici](transactions.md))
9. Vérifier l'empreinte des transactions
10. Vérifier que le bloc précédent est dans la branche principale (cat 1) ou dans une branche secondaire (cat 2). Sinon, ajouter ce bloc à la liste des blocs orphelins (cat 3), puis demander au nœud qui nous a envoyé ce bloc son bloc précédent. C’est terminé pour ce bloc
11. Ajouter le bloc à la base de données. Trois cas : 1. le bloc agrandit la chaîne principale ; 2. le bloc agrandit une chaîne secondaire mais pas assez pour qu’elle devienne la branche principale ; 3. le bloc agrandit une chaîne secondaire et celle-ci devient la branche principale.
12. Pour le cas 1, ajouter à la branche principale :
	1. Enlever les transactions de la piscine des transactions
	2. Transmettre le bloc aux autres nœuds
13. Pour le cas 2, on ne fait rien.
14. Pour le cas 3, une branche secondaire devient la branche principale.
	1. Trouver le « fork-bloc », c’est-à-dire le bloc ou la chaîne principale et la chaîne secondaire se séparent
	2. Modifier la chaîne principale pour n’aller qu’à ce bloc
	3. Ajouter les blocs de la branche secondaire à la branche principale en validant
	4. À partir du fork-bloc, enlever les transactions de la piscine des transactions
	5. Transmettre le bloc aux autres nœuds
15. Pour chaque bloc orphelin dont le bloc prcédent est celui-ci, recommencer tout ça

