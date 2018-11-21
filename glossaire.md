# Glossaire

- bloc (block): Un ensemble de données, qui doivent être validées par chaque nœud de la chaine de blocs, chaque
  bloc change et pointe vers sa version "précédente". Dès qu'un nœud est modifié, il doit se faire valider.

- nœud (node): Un élément du réseau qui vérifie, valide, crée, et invalide un ou des blocs.

- chaine de blocs (blockchain): Historique le plus long des modifications VALIDES effectuées sur le nœud le plus récent.

- Nœud génèse (genesis-block): Nœud le plus vieux, unique et prédéterminé au sein de la chaine de blocs.

- Objectif (target): Il s’agit d’un nombre très grand (256-bit) que tous les nœuds doivent partager.
  L'empreinte de l’en-tête d’un bloc doit être inférieur ou égal à ce nombre au moment ou le bloc est créé pour être valide.
  Plus l'objectif est bas, plus il est "difficile" de trouver un bloc valide.
