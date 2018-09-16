# Blocks

Un block est composé de deux parties : le header et les transactions. Une transaction est identifiée par son hash, un block par le hash de son header.

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

La liste des transactions est dans cet exemple vide, mais elle doit toujours contenir une (et une seule) (et commencer) par une transaction de génération (coinbase). Cette transaction spéciale créer des ENSICOIN comme récompense pour le mineur ([voir ici](transations.md)).

## Règles de validation

1. Vérifier le format
2. Rejeter si ce block est déjà dans la base de données
3. Rejeter s’il n’est dans aucune des trois catégorie
4. La liste des transactions doit être non-vide
5. Le hash de l’header du block doit être inférieur au target actuel
6. Le timestamp du block doit être inférieur à 2 heures dans le futur
7. La première transaction doit être une coinbase, le reste ne doit pas l’être
8. Valider chaque transaction ([voir ici](transactions.md))
9. Vérifier le hash des transactions
10. Vérifier que le block précédent est dans la branche principale (cat 1) ou dans une branche secondaire (cat 2). Sinon, ajouter ce block à la liste des blocks orphelins (cat 3), puis demander au nœud qui nous a envoyé ce block son block précédent. C’est terminé pour ce block
11. Ajouter le block à la base de données. Trois cas : 1. le block agrandit la chaîne principale ; 2. le block agrandit une chaîne secondaire mais pas assez pour qu’elle devienne la branche principale ; 3. le block agrandit une chaîne secondaire et celle-ci devient la branche principale.
12. Pour le cas 1, ajouter à la branche principale :
	1. Enlever les transactions de la piscine des transactions
	2. Transmettre le block aux autres nœuds
13. Pour le cas 2, on ne fait rien.
14. Pour le cas 3, une branche secondaire devient la branche principale.
	1. Trouver le « fork-block », c’est-à-dire le block ou la chaîne principale et la chaîne secondaire se séparent
	2. Modifier la chaîne principale pour n’aller qu’à ce block
	3. Ajouter les blocks de la branche secondaire à la branche principale en validant
	4. À partir du fork-block, enlever les transactions de la piscine des transactions
	5. Transmettre le block aux autres nœuds
15. Pour chaque block orphelin dont le block prcédent est celui-ci, recommencer tout ça


