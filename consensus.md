# Consensus

Les règles de l’ENSCOIN doivent être suivies par tous les nœuds afin d’atteindre un consensus. Elles sont basées sur les règles de litecoin.

## Difficulté

Avant d’expliquer la difficulté, expliquons ce qu’est l'objectif.

Il s’agit d’un nombre très grand (256-bit) que tous les nœuds doivent partager. Le hachage de l’en-tête d’un bloc doit être inférieur ou égal à ce nombre au moment ou le bloc est créé pour être valide. Plus l'objectif est bas, plus il est difficile de trouver un block valide.

L'objectif est ajusté tous les 2016 blocs afin que le temps moyen entre les blocs soit d’environ 2.5 minutes.

```
2016 blocs * 2.5 minutes = 302400 secondes = 84 heures = 3.5 jours
```

Quand 2016 blocs sont passés, on remonte l’historique de 2016 blocs et on calcule la différence entre le timestamp du bloc actuel et du bloc précédent. Cette différence doit être bornée dans [84 heures / 4, 84 heures * 4] afin d’éviter un changement de difficulté trop fort.

Finalement, on récupère l'objectif de la fenêtre précédente et on définit le nouveau target comme :

```
new target = old target * time for 2016 blocks / 84 hours
```

Maintenant, qu’est-ce que la difficulté dont on parle tant ? Il s’agit d’une mesure inversement proportionnelle à l'objectif. Elle est plus compréhensible pour les humains, mais inutile pour l’implémentation.

Pour avoir plus de détails sur calcul de cette mesure, vous pouvez lire cette page : https://en.bitcoin.it/wiki/Difficulty.

## Récompense et création des ENSICOIN

La récompense des mineurs est mise à jour tous les 210000 blocks. Elle est calculée ainsi :

```
(42 * 100000000) >> (height / 210000)
```
