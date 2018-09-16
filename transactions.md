# Transactions

Les transactions sont largement inspirée du système de Bitcoin.

## Format général

```json
{
	"version": 0,
	"flags": [
		"a",
		"olala"
	],
	"inputs": [
		{
			"previousOutput": {
				"transactionHash": "",
				"index": 0
			},
			"script": [
				"signature",
				"pubkey"
			]
		}
	],
	"outputs": [
		{
			"value": 42,
			"script": [
				"OP_DUP",
				"OP_HASH160",
				"hash160(pubkey)",
				"OP_EQUAL",
				"OP_VERIFY",
				"OP_CHECKSIG"
			]
		}
	]
}
```

## Règles de validation

1. Vérifier le format de la transaction
2. Vérifier qu’il y a au moins une entrée et une sortie
3. Taille plus petite ou égale à MAX_BLOCK_SIZE 
4. Les valeurs des sorties doivent être supérieures à 0
5. Vérifier que cette transaction n’est pas une coinbase, sauf si elle est au début d’un block
6. La somme des sorties doit être strictement inférieure à celle des entrées
7. Rejeter si on a déjà une transaction avec le même hash dans la chaîne principale ou dans la piscine des transactions
8. Pour toutes les entrées, si la sortie reférencée l’est déjà dans une entrée de la chaîne principale alors rejeter
9. Pour toutes les entrées, regarder dans la chaîne principale ou dans la piscine si la transaction référencée existe. Si ce n’est pas le cas, il s’agit d’une transaction orpheline. Il faut alors l’ajouter dans la liste des transactions orphelines sauf s’il y a déjà une transaction qui correspond à celle-ci dans cette liste.
10. Pour toutes les entrées, si une des sorties reférencée est une coinbase, alors il faut vérifier qu’au moins 42 blocks sont passés depuis.
11. Pour toutes les entrées, si la transaction reférencée existe mais pas la sortie, alors rejeter
12. Vérifier les scripts de toutes les entrées
13. Ajouter à la piscine des transactions (olala)
14. Envoyer aux autres nœuds
15. Executer tout ça sur les transactions orphelines qui utilisent cette transaction

## Script

Cette section va détailler le langage des scripts situés dans les entrées et les sorties des transactions. L’idée est d’avoir un langage de script simple qui n’est pas turing-complet. Comme pour Bitcoin, on s’inspirera de [FORTH](https://www.forth.com/forth/).

L’idée est donc de faire un langage de script basé sur une pile, exécuté de droite à gauche qui se terminera forcément (pas de boucles, pas turing complet).

Voici la liste des mots-clés du script de l’ENSICOIN :

| Mot-clé     | Description |
|-------------|-------------|
| OP_DUP      | Duplique le haut de la pile.
| OP_HASH160  | Hash le haut de la pile avec RIPEMD-160.
| OP_EQUAL    | Remplace les deux valeurs du haut de la pile par vrai si elles sont égales, par faux sinon.
| OP_VERIFY   | Marque la transaction comme invalide si le haut de la pile est à faux, enlève le haut de la pile sinon.
| OP_CHECKSIG | Utilise la clé publique qui est en haut de la pile pour vérifier que la signature située juste en-dessous est valide. Marque la transaction comme invalide sinon. Enlève ces deux valeurs de la pile.

Si autre chose qu’un mot-clé est présent, alors cette chose est simplement mise en haut de la pile.

Ces mots-clés suffisent à réaliser une transaction basique de type P2PKH :

Script de l’entrée : `OP_DUP OP_HASH160 hash160(pubkey) OP_EQUAL OP_VERIFY OP_CHECKSIG`

Script de la sortie : `<signature> <pubkey>`
