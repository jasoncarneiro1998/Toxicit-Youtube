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
