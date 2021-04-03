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
