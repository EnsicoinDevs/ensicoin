# Consensus

Les règles de l’ENSCOIN doivent être suivies par tous les nœuds afin d’atteindre un consensus. Elles sont basées sur les règles de litecoin.

## Difficulté

Avant d’expliquer la difficulté, expliquons ce qu’est le target.

Il s’agit d’un nombre très grand (256-bit) que tous les nœuds doivent partager. Le hash de l’header d’un block doit être inférieur ou égal à ce nombre au moment ou le block est créé pour être valide. Plus le target est bas, plus il est difficile de trouver un block valide.

Le target est ajustée tous les 2016 blocks afin que le temps moyen entre les blocks soit d’environ 2.5 minutes.

2016 blocks * 2.5 minutes = 302400 secondes = 84 heures = 3.5 jours

Quand 2016 blocks sont passés, on remonte l’historique de 2016 blocks et on calcule la différence entre le timestamp du block actuel et du block précédent. Cette différence doit être bornée dans [84 heures / 4, 84 heures * 4] afin d’éviter un changement de difficulté trop fort.

Finalement, on récupère le target de la fenêtre précédente et on définit le nouveau target comme :

new target = old target * time for 2016 blocks / 84 hours

Maintenant, qu’est-ce que la difficulté dont on parle tant ? Il s’agit d’une mesure inversement proportionnelle au target. Elle est plus compréhensible pour les humains, mais inutile pour l’implémentation.

Pour avoir plus de détails sur calcul de cette mesure, vous pouvez lire cette page : [en.bitcoin.it/wiki/Difficulty](https://en.bitcoin.it/wiki/Difficulty).

## Récompense et création des ENSICOIN

La récompense des mineurs est mise à jour tous les 210000 blocks. Elle est calculée ainsi :

(42 * 100000000) >> (height / 210000)
