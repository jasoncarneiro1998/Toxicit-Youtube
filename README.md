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

**Suppression de certaines features**

• “video_id_court” et “video_id” (pas d’intérêt car la ligne réfère déjà la vidéo en question)

• “channel_id” (redondant avec “channel_name” car aucun doublon)

• “X” (dernière colonne inutile) (enlevée directement dans le fichier Excel)

**Valeurs manquantes**

Vérification mais aucune valeur ne manquait.
Il nous reste alors un Dataset (46102,23) avec donc 23 features.

## Analyse des variables indépendamment les unes des autres

**Variables quantitatives (20)**

Après une rapide exploration des données, nous avons déterminé la variance (.var()) de l’ensemble des features quantitatives (celles qui ne correspondent pas aux catégories).
Nous pouvons remarquer que certaines variables ont une variance très faible notamment: nbrMotInsulteMoyenne, nbrMotAllongMoyenne, nbrMotMAJMoyenne, nbrExclMarkMoyenne.
En faisant une boîte à moustache pour analyser la distribution de ces variables, on voit que la plupart des données ont une valeur entre 0 et 1 pour ces colonnes. On voit aussi qu’il existe beaucoup de valeurs extrêmes.
En utilisant la commande pour déterminer le nombre d’outliers, on voit par exemple que pour la variable nbrMotInsulteMoyenne, la proportion d’outliers représente 15%. Nous ne pouvons donc pas les supprimer du dataset.
Si nous nous intéressons à présent à ces mêmes variables en ne considérons plus leurs moyennes mais seulement leur quantité à travers les features: nbrMotInsulte, nbrMotAllong, nbrMotMAJ, nbrExclMark, nbrQuestMark. Ces quantités seules ne sont pas exploitables car elles dépendent du nombre de commentaires de la vidéo, du nombre de mots par commentaire.

**Catégorisation des vidéos**

Le dataset permet de catégoriser les vidéos en 3 types de catégories:
• categorie_new : “Coeur”, “Niche” ou “Partisans qui réfèrent au type de débat que suscite la vidéo.
• categ_inst: presse nationale, presse régionale, presse magazine, alternatif, TV et presse uniquement sur le web qui réfèrent au type de média de la vidéo.
• channel_name qui réfère le nom de la chaîne, ainsi toutes les vidéos d’une même chaîne sont reconnaissables.
En traçant la répartition de chacune de ces catégorisations, on peut observer :
• pour channel_name: 5 chaînes youtube représentent à elles seules 60% du dataset (un approfondissement de l’étude pourrait être axé sur un élargissement du dataset en termes de chaînes youtube)
• pour categ_inst:
• pour categorie_new:
