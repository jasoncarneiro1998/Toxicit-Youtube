# Toxicité-Youtube

# Introduction

L’objectif de cette étude est de déterminer et décrire la toxicité d’une vidéo Youtube. Le dataset fourni pour cette étude réunit plusieurs informations de 46102 vidéos Youtube.
Il y a exactement 28 informations différentes telles que des informations sur le nombre d’insultes, de mots allongés, de points d’exclamation et d’interrogations dans les commentaires des vidéos, ainsi que le nombre de commentaires, ceux qui sont des réponses à d’autres commentaires et ceux non. Des caractéristiques sur les commentateurs sont aussi présentes. Les chaînes d’où proviennent les vidéos sont documentées elles aussi: son nom, le nombre d’abonnés, le nombre de vues et sa catégorie selon deux classifications différentes.
Ainsi beaucoup d’informations ont été récoltées afin de pouvoir estimer des indicateurs de toxicité d’une vidéo Youtube à partir de ce dataset. Après avoir analysé les données et les avoir nettoyées, nous allons déterminer un modèle de machine permettant de déterminer le nombre d’insultes présentes dans les commentaires d’une vidéo Youtube. Par la suite, nous créerons un
indice de toxicité afin de pouvoir classer chaque vidéo Youtube en un certain nombre de catégories, de la moins toxique à la plus toxique.

# Statistiques descriptives et features engineering

## Lecture des données

Nous avons modifié légèrement le fichier Excel contenant l’ensemble des données afin de faciliter sa lecture sur le Notebook:

• modifications des caractères spéciaux comme les accents (ex: Libération → Liberation) ;


• modification des virgules en points pour les flottants (nous aurions aussi pu utiliser le paramètre float = ’,’ dans python).

Après lecture du fichier dans le Notebook, nous obtenons un tableau Panda (46102, 27) : 46102 lignes qui correspondent chacune à une vidéo youtube et 27 colonnes correspondant chacune à une feature.

## Nettoyage du dataset

### Suppression de certaines features

• “video_id_court” et “video_id” (pas d’intérêt car la ligne réfère déjà la vidéo en question)

• “channel_id” (redondant avec “channel_name” car aucun doublon)

• “X” (dernière colonne inutile) (enlevée directement dans le fichier Excel)

### Valeurs manquantes

Vérification mais aucune valeur ne manquait.
Il nous reste alors un Dataset (46102,23) avec donc 23 features.

## Analyse des variables indépendamment les unes des autres

### Variables quantitatives (20)

Après une rapide exploration des données, nous avons déterminé la variance (.var()) de l’ensemble des features quantitatives (celles qui ne correspondent pas aux catégories).
Nous pouvons remarquer que certaines variables ont une variance très faible notamment: nbrMotInsulteMoyenne, nbrMotAllongMoyenne, nbrMotMAJMoyenne, nbrExclMarkMoyenne.
En faisant une boîte à moustache pour analyser la distribution de ces variables, on voit que la plupart des données ont une valeur entre 0 et 1 pour ces colonnes. On voit aussi qu’il existe beaucoup de valeurs extrêmes.
En utilisant la commande pour déterminer le nombre d’outliers, on voit par exemple que pour la variable nbrMotInsulteMoyenne, la proportion d’outliers représente 15%. Nous ne pouvons donc pas les supprimer du dataset.
Si nous nous intéressons à présent à ces mêmes variables en ne considérons plus leurs moyennes mais seulement leur quantité à travers les features: nbrMotInsulte, nbrMotAllong, nbrMotMAJ, nbrExclMark, nbrQuestMark. Ces quantités seules ne sont pas exploitables car elles dépendent du nombre de commentaires de la vidéo, du nombre de mots par commentaire.

### Catégorisation des vidéos

Le dataset permet de catégoriser les vidéos en 3 types de catégories:

• categorie_new : “Coeur”, “Niche” ou “Partisans qui réfèrent au type de débat que suscite la vidéo.

• categ_inst: presse nationale, presse régionale, presse magazine, alternatif, TV et presse uniquement sur le web qui réfèrent au type de média de la vidéo.

• channel_name qui réfère le nom de la chaîne, ainsi toutes les vidéos d’une même chaîne sont reconnaissables.

En traçant la répartition de chacune de ces catégorisations, on peut observer :

• pour channel_name: 5 chaînes youtube représentent à elles seules 60% du dataset (un approfondissement de l’étude pourrait être axé sur un élargissement du dataset en termes de chaînes youtube)

## Comparaison des variables entre elles

### Corrélation

Afin de comparer l’ensemble des variables entre elles, nous avons tracé la matrice de corrélation du dataset ainsi que des diagrammes par paire pour chaque couple de variable. Nous avons pu mettre en couleur chacune des catégories, notamment categorie_new qui nous a particulièrement intéressé car il n’y avait que 3 types de valeurs possibles.

Le “pairplot” nous permet d’avoir une vue générale des données. On peut voir que les graphes se ressemblent beaucoup entre eux et que les points sont très condensés.
De nombreuses variables sont corrélées (coefficient de corrélation supérieur à 0.9) notamment le nombre de commentateurs uniques qui est corrélé avec le nombre de commentaires total et le nombre d’utilisateurs dont les commentaires sont likés (0.97 et 0.96 respectivement).

### Outliers

Ces diagrammes par paires montrent encore beaucoup de valeurs extrêmes dans tout le dataset et pour n’importe quelle feature. Ces outliers sont en grande proportion, environ 15% du dataset pour chaque catégorie. Nous ne pouvons donc pas décider de les supprimer, les conséquences seraient trop importantes pour la qualité de notre dataset.

## Encodage des variables non quantifiées

Afin d’avoir un tableau de données entièrement quantifié (seulement avec des nombres), nous avions plusieurs possibilités pour encoder les variables non quantifiées.
Au départ nous avons opté pour un encodage assez simple en associant un chiffre à chaque valeur différente de la variable catégorie. Par exemple pour categorie_new, à "Coeur" on associait 1, “Niche” 2 et “Partisan” 3.

Nous avons finalement opté pour un autre type d’encodage qui permet des calculs plus rapides par la suite. Pour chaque valeur différente de la variable catégorie, une nouvelle variable est créée valant 1 si la vidéo appartient à cette catégorie, 0 sinon. Ainsi pour une variable catégorie comprenant au départ 3 valeurs différentes, 3 variables sont créées valant chacune 0 ou 1.

Cette méthode permet ensuite de s’adapter à une plus grande base de données car après avoir trouvé notre modèle nous pourrons le tester avec des vidéos qui n’entrent pas dans les catégories initiales, notamment des vidéos dont la chaîne Youtube n’est pas encore présente dans le dataset.


# Benchmark des méthodes de régression pour prédire le nombre d’insultes liées à une vidéo

Le but de cette partie est de prédire la variable “nbrMotInsulte”, en formulant donc le problème sous la forme d’une régression.


Pour commencer nous avons entraîné différents modèles de Régression afin d’avoir une idée des modèles les plus performants sur notre base de données. Le KNeighbors, LinearRegression ainsi que le GradientBoosting présentent les meilleurs résultats. Cependant pour la plupart des modèles, nous sommes en situation d’over-fitting, en effet notre modèle est tellement entraîné sur notre jeu d'entraînement qu’il ne peut pas correspondre à des jeux de données supplémentaires ou ne pas prévoir de manière fiable les observations futures. Pour remédier à ce problème nous allons déterminer les hyper-paramètres optimaux à l’aide d’un GridSearchCv.

Pour le KneighborsRegressor, on trouve qu’il obtient un meilleur score (0.74) avec 2 voisins cependant on est en Sur-Apprentissage.
Avec 11 voisins, le score est plus faible (0.68) mais nous ne sommes pas en situation de Sur-Apprentissage, notre modèle peut donc prédire correctement le nombre d'insultes avec 68% de précision.

Nous nous sommes ensuite intéressés aux variables qui ont eu une importance lors de l’apprentissage de notre modèle. Les variables qui ont eu un rôle majeur sont : ‘nbrMot’ et ‘ViewCount’. Ce résultat paraît plutôt logique car plus il y a de mots et plus il y aura de vues, plus il sera susceptible d’avoir des mots d’insultes.


Nous avons ensuite réalisé un test de Khi 2 pour trouver de quelles variables dépend y_train. Nous avons trouvé des résultats similaires à l'importance des variables durant
l’apprentissage puisque y_train dépend beaucoup de 'nbrMot', de 'viewCount' et de 'subscriberCount'.

Nous avons ensuite optimisé notre GradientBoosting en tentant de trouver les valeurs optimales pour les hyper-paramètres suivant : learning_rate, subsample, n_estimators, max_depth. Cependant faire cela avec un GridSearchCv prendrait beaucoup trop de temps, nous avons donc utilisé un randomizedSearchCv. Nous obtenons finalement un score de 0.73.


Nous avons obtenu de bons résultats pour le KNeighbors (avec deux voisins), pour le GradientBoosting optimisé et pour le randomForest, pourquoi ne pas utiliser la méthode de stacking pour voir ce qu’il en est ?

Le stacking est un procédé qui consiste à appliquer un algorithme de machine learning à des classifieurs/régresseurs générés par un autre algorithme de machine learning. D’une certaine façon, il s’agit de prédire quels sont les meilleurs classifiers/régresseurs et de les pondérer. Cette démarche à l’avantage de pouvoir agréger des modèles très différents et d’améliorer sensiblement la qualité de la prédiction finale.
Nous obtenons en effet un score de 0.75 sans over-fitting.


Nous avons également réalisé un modèle de deep learning, pour ce faire il a fallu dans un premier temps transformer notre dataset en Tensor pour pouvoir utiliser la librairie Pytorch. Nous avons choisi comme modèle un MLP (Multi Layer Perceptron) avec 3 couches cachées et comme fonction d’activation un softmax. Nous avons choisi comme critère pour notre fonction de perte le MSE( Mean Square Error) MSE = sum[(y-y_pred)^2]. 
Une fois notre modèle entraîné et évalué, nous l’avons testé en créant 3 instances, le modèle est censé prédire le nombre d’insultes cependant il retourne trois fois le même résultat alors qu’il ne devrait pas (puisque les test sont totalement différents). Nous n’avons pas réussi à comprendre pourquoi notre modèle de deep Learning n’étais pas précis, il faudrait peut-être augmenter le nombre d’epochs (mais nous somme limité avec nos ordinateurs portables), changer le learning rate, ajouter du dropout ou choisir un autre optimizer plus adapté à notre Loss.

Pour conclure, si nous devons choisir un modèle pour prédire le nombre d'insultes, nous choisirons la méthode de stacking car c’est le modèle qui a le meilleur score sans over-fitting.

# Création d’un indice de toxicité et classification

L’objectif est de créer une nouvelle variable, à partir des variables de départ, qui est plus représentative du caractère toxique d’une vidéo.

Notre méthode consiste à calculer un score qui représente un degré de toxicité. Plus le score est élevé, plus la vidéo est toxique. Puis, ce score prend la forme d’une variable catégorielle : toxicité faible/moyenne/forte.

## Définir les variables de départ

Il s’agit, tout d’abord, de définir à partir de quelles variables de départ le score va être calculé. Le choix de ces variables est mené en se posant la question suivante : “Est-ce que cette variable nous permet de dire si une vidéo est toxique ?” Si non, on la supprime. Si oui, on la garde et on se demande: ”Si l’on combine cette variable avec une
autre variable, la variable obtenue sera-t-elle plus représentative de la toxicité ?” Si non, on garde la variable de base. Si oui, on prend la nouvelle variable obtenue.

Prenons pour commencer le nombre de mots d’insultes. Plus il y a d’insultes, plus on peut considérer qu’une vidéo est toxique. Donc, cette variable permet de dire si une vidéo est toxique ou non. Cependant, il est plus pertinent d’obtenir un pourcentage de mots d’insultes par rapport au nombre de mots total en-dessous d’une vidéo. En effet, par exemple, s’il y a trois mots d’insultes dans l’ensemble des commentaires et que le nombre de mots est peu élevé, nous considérons la vidéo comme toxique. Or, si le nombre de mots très grand devant trois, nous considérerons que la vidéo n’est pas toxique. Nous gardons donc la variable “Pourcentage de mots d’insultes” qui correspond à “nbrMotInsulte/nbrMot” qui est plus représentative de la toxicité d’une vidéo. On effectue exactement le même raisonnement pour les variables nbrMotAllong, nbrMotMAJ, nbrQuestMark et nbrExclMark. Nous gardons donc les variables suivantes :

• nbrMotAllong/nbrMot

• nbrMotMAJ/nbrMot

• nbrQuestMark/nbrMot

• nbrExclMark/nbrMot

Nous gardons les variables suivantes telles quelles car elles sont significatives pour estimer la toxicité d’une vidéo et en les combinant à une autre variable, nous n’obtenons pas de variables plus représentatives:

• nbrMotInsulteMoyenne

• nbrMotAllongMoyenne

• nbrMotMAJMoyenne

• nbrQuestMarkMoyenne

• nbrExclMarkMoyenne

Pour les variables “discussion_count” et “comment_count”, nous considérons qu’il est plus pertinent de les comparer à “message_count”, car à nouveau cela dépend du nombre de commentaires au total. Plus il y a de fils de discussion et de réponses à des commentaires, plus cela signifie qu’il y a débat et donc potentiellement davantage de toxicité. Ainsi, nous retenons les variables suivantes :

• discussion_count/message_count

• comment_count/message_count

Pour la variable “viewCount” soit le nombre de vues d’une vidéo, nous estimons qu’il est plus pertinent de la comparer à “subscriberCount” car si une vidéo a un nombre de vues grand devant le nombre de personnes qui suivent la chaîne, la vidéo est susceptible d’avoir “créé le buzz” et donc d’avoir davantage de commentaires toxiques. Nous retenons donc la variable : “viewCount/subscriberCount”.

Pour les variables “channel_name”, “categ_inst, “categorie_new”, nous pensons qu’elles peuvent influer sur la toxicité. En effet, par exemple, certains journaux sont plus polémiques et peuvent donc être sujets à davantage de toxicité car qui dit polémique dit débat. Cependant, nous n’arrivons pas à définir une méthode pour dire si une chaîne est toxique ou non. On aurait pu faire des statistiques descriptives et trouver quelle chaîne est la plus toxique selon ce datasat. Cependant, cela n’aurait pas pu permettre de généraliser cette information. De plus, notre modèle ne sera peut-être pas capable de prédire la toxicité d’une vidéo dont la chaîne n’est pas présente dans le datasat. Ainsi, pour toutes ces raisons, nous avons décidé de ne pas utiliser ces variables pour le calcul du score de la toxicité.

Toutes les autres variables sont supprimées car nous considérons qu’elles ne nous permettent pas de dire si une vidéo est toxique ou non à partir de ces variables.

## Définir une méthode de calcul du score de toxicité

Il s’agit maintenant de définir une méthode de calcul du score en fonction des variables choisies. Pour cela, nous avons accordé un coefficient de pondération à chaque variable choisie dans la partie précédente en fonction de l’importance de cette variable, selon nous, pour évaluer la toxicité d’une vidéo. Nous sommes tout à fait conscients que ces coefficients sont très subjectifs et dépendent de notre appréciation. Il faudrait se baser sur une étude sociologique et linguistique par exemple afin de déterminer ces coefficients de pondération.

Voici le tableau avec les coefficients de pondération associés à chaque variable :

Nom de la variable                             Coefficient de pondération

nbrMotInsulte/nbrMot                                        5

nbrMotAllong/nbrMot                                         2

nbrMotMAJ/nbrMot                                            3

nbrQuestMark/nbrMot                                         1

nbrExclMark/nbrMot                                          2

nbrMotInsulteMoyenne                                        5

nbrExclMarkMoyenne                                          2

nbrQuestMarkMoyenne                                         1

nbrMotMAJMoyenne                                            3

nbrMotAllongMoyenne                                         2

discussion_count/message_count                              4

comment_count/message_count                                 4

viewCount/subscriberCount                                   3
