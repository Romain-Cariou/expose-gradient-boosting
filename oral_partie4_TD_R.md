# Partie 4 : le gradient boosting dans R (oral)

Durée prévue : environ 10 minutes.

## Slide 1 : le problème

- Maintenant qu'on a vu la théorie, on va voir comment ça se passe dans R. On reprend l'exemple d'Emile sur les diamants.
- On utilise le jeu de données `diamonds` du package ggplot2. Il contient environ 54 000 diamants, mais on en garde 300 pour que ce soit plus lisible, et seulement 2 variables : le prix et le poids.
- Sur le graphique, on voit que plus un diamant est lourd, plus il est cher, mais la relation n'est pas une droite : une régression linéaire serait donc mal adaptée. On va voir comment s'en sort le gradient boosting.

## Slide 2 : `gbm` et `predict`

- Dans R, le gradient boosting s'utilise avec la fonction `gbm` du package du même nom.
- Bloc 1 : on lui donne la formule (ici le prix en fonction du poids), les données, le nombre d'arbres B (`n.trees`) et le pas λ (`shrinkage`).
- Bloc 2 : une fois le modèle ajusté, on peut prédire le prix de nouveaux diamants avec `predict`. `gbm` garde en mémoire tous les modèles intermédiaires, donc on peut prédire avec n'importe quel nombre d'arbres entre 1 et 50.
- Pour la suite, j'ai rangé ce code dans une fonction, `trace_boost`, qui ajuste le boosting et trace la courbe des prix prédits.

## Slide 3 : l'effet du nombre d'arbres B

- On utilise cette fonction pour voir l'effet de B.
- Gauche (1 arbre) : le modèle part du prix moyen et fait une seule coupure. Comme λ = 0,1, on obtient une toute petite marche. Le modèle prédit à peu près le même prix pour tous les diamants : il est beaucoup trop simple.
- Milieu (50 arbres) : chaque arbre a corrigé un peu les erreurs des précédents. En ajoutant toutes ces petites marches, la courbe suit bien la tendance des prix.
- Droite (20 000 arbres) : on pourrait penser que ce serait encore mieux, mais non. Une fois la tendance apprise, le modèle fait des pics pour coller à certains diamants en particulier : c'est le sur-ajustement. Sur de nouveaux diamants, ces pics ne correspondraient à rien.
- Donc plus d'arbres n'est pas toujours mieux : B est un paramètre à choisir avec soin, on verra plus loin comment.

## Slide 4 : l'effet du pas λ

- Deuxième réglage : le pas λ. On garde 50 arbres, mais on divise le pas par 10.
- Gauche (λ = 0,1) : le modèle qu'on vient de voir, qui suit bien la tendance.
- Droite (λ = 0,01) : chaque arbre n'apporte qu'un centième de sa correction. Au bout de 50 arbres, le modèle est encore loin des données : il sous-estime nettement les gros diamants.
- Avec un petit pas, il faut donc plus d'arbres pour obtenir le même résultat, ce qui prend plus de temps. Mais en avançant prudemment, le modèle généralise souvent un peu mieux.
- Donc B et λ se règlent ensemble. En pratique, on fixe souvent un petit λ (entre 0,01 et 0,1), et on cherche le meilleur B.

## Slide 5 : toutes les variables

- Jusqu'ici, on n'utilisait que le poids. Maintenant, on utilise toutes les caractéristiques du diamant : taille, couleur, pureté, dimensions…
- Gauche : on tire 10 000 diamants au hasard, et on les sépare en un échantillon d'apprentissage (70 %) et un échantillon test (30 %).
- Droite : le code est le même qu'avant, avec 3 changements :
  - le point dans la formule (`price ~ .`) veut dire qu'on prend toutes les variables ;
  - des arbres plus profonds (3 coupures), pour capter les effets combinés entre variables ;
  - `cv.folds` sert à choisir le nombre d'arbres (slide suivante).

## Slide 6 : choisir B par validation croisée

- Il reste à choisir B. Avec `cv.folds = 5`, `gbm` a fait une validation croisée à 5 blocs, ce qui donne l'erreur de validation pour chaque nombre d'arbres, de 1 à 1000.
- Code : `gbm.perf` cherche le nombre d'arbres qui minimise cette erreur. Ici, B = 905.
- Graphique :
  - en rouge, l'erreur d'entraînement, mesurée sur les diamants qui ont servi à construire le modèle. Elle baisse sans arrêt, donc elle ne peut pas servir à choisir B : elle nous dirait toujours de prendre le maximum ;
  - en bleu, l'erreur de validation, mesurée sur des diamants de l'échantillon d'apprentissage mis de côté à tour de rôle (donc non vus par le modèle). Elle baisse vite au début, puis se stabilise vers 500 arbres.
- Donc on choisit B avec l'erreur de validation, jamais avec l'erreur d'entraînement.

## Slide 7 : résultats sur l'échantillon test

- Dernière étape : on juge le modèle sur l'échantillon test, les 3000 diamants qu'il n'a jamais vus. On compare 3 approches.
- Code : on prédit les prix du test avec le boosting (B = 905). Pour comparer, on ajuste aussi une forêt aléatoire sur l'échantillon d'apprentissage, avec le package `ranger`.
- Graphique : on mesure l'erreur de chaque méthode par la racine de l'erreur quadratique moyenne, pour avoir une erreur en dollars :
  - en prédisant toujours le prix moyen, on se trompe d'environ 3900 $ ;
  - la forêt aléatoire se trompe d'environ 680 $, le boosting d'environ 650 $ : les deux divisent l'erreur par 6.
- Le boosting fait un peu mieux que la forêt, avec environ 4 % d'erreur en moins. L'écart est faible, mais sur cet exemple, le boosting est meilleur.

## Slide 8 : à retenir

Pour résumer, voici la démarche pour utiliser le boosting dans R, en 4 étapes :

1. **Séparer** les données en un échantillon d'apprentissage et un échantillon test.
2. **Ajuster** le boosting avec `gbm` sur l'échantillon d'apprentissage, en fixant le nombre d'arbres maximum, le pas λ, la profondeur des arbres, et `cv.folds` pour la validation croisée.
3. **Choisir B** avec `gbm.perf` : c'est le nombre d'arbres qui minimise l'erreur de validation.
4. **Prédire et évaluer** sur l'échantillon test avec `predict`, sans oublier de préciser le nombre d'arbres, puis mesurer l'erreur.

En pratique, on prend en général un petit λ, des arbres de 2 à 5 coupures, et on laisse la validation croisée choisir B.

Transition vers la partie 5 (Diecy).
