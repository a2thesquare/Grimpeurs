## Introduction
Ce projet etudie la population de Grimpeurs de Wikidata sous différents angles :

Évolution temporelle de la proportion de femmes inventrices
Relations entre variables qualitatives (genre, pays, siècle)
Structure de réseau de co-invention et détection de communautés

En premier lieu les données ont été extraites via SPARQL, stockées dans un triplestore (AllegroGraph Cloud), analysées en Python  et documentées dans des notebooks Jupyter.

## Objectifs du projet

Ce projet vise à nous initier aux méthodes numériques appliquées aux sciences humaines et sociales, en s’appuyant sur des outils d’extraction et d’analyse de données comme Wikidata. Il s’agit d’apprendre à interroger, structurer et interpréter des données dans une démarche prosopographique exploratoire.

## Questions de recherche
### 1) Quelle est la répartition géographique des grimpeurs notables, et comment a-t-elle évolué dans le temps ?

*Méthode :*

* Distribution des pays de naissance ou de nationalité par décennie de naissance.
* Visualisation sous forme de graphique ou carte.
* Hypothèse explorée : la discipline se diffuse à partir de quelques pays « fondateurs » vers d'autres zones.

### 2) Y a-t-il des différences de genre selon les disciplines (bloc, difficulté, vitesse) ?

*Méthode :*

* Analyse croisée entre le genre et la discipline principale.
* Tableau de contingence + test du Chi².
* Objectif : voir s’il y a une répartition genrée dans les différentes sous-disciplines.

### 3. Est-ce qu'il y a un plus grosse proportion de femmes grimpeuses à partir d'une certaine année ? 


## Conclusion
Limitations ?
