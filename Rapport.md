## Introduction
Ce projet etudie la population de Grimpeurs de Wikidata sous différents angles :

Évolution temporelle de la proportion de femmes inventrices
Relations entre variables qualitatives (genre, pays, siècle)
Structure de réseau de co-invention et détection de communautés

En premier lieu les données ont été extraites via SPARQL, stockées dans un triplestore (AllegroGraph Cloud), analysées en Python  et documentées dans des notebooks Jupyter.

## Objectifs du projet

Ce projet vise à nous initier aux méthodes numériques appliquées aux sciences humaines et sociales, en s’appuyant sur des outils d’extraction et d’analyse de données comme Wikidata. Il s’agit d’apprendre à interroger, structurer et interpréter des données dans une démarche prosopographique exploratoire.

## Questions de recherche
### 1) Quelle est la répartition géographique des grimpeurs notables ?

Pour répondre à cette question, la démarche se décompose en plusieurs étapes :  
1) Importation de la base de données initiale([voir détail](SPARQL-Documentation/Importation-des-donnees.md)) pour nettoyer et clarifier le DataFrame original.  
2) Importation des citoyennetés ([voir détail](SPARQL-Documentation/Import_des_citoyennetes.md)) afin d’obtenir le nombre total d’individus par citoyenneté, grâce au fichier extrait [Citoyennetes.csv](data/Citoyennetes.csv).  
3) Visualisation des citoyennetés [voir détail](Jupyter-Notebooks/Distribution_nationalites) pour observer la distribution géographique.

Grâce à un simple diagramme en barres horizontales, on constate que les États-Unis sont le pays le plus représenté parmi les grimpeurs notables, l'Allemagne puis la France en deuxième et troisième place.  

Une autre observation intéressante est la présence multiple du Royaume-Uni, sous différentes dénominations historiques : *United Kingdom of Great Britain and Ireland*, *United Kingdom*, *Kingdom of Great Britain*, etc.  

De même, la Russie moderne apparaît sous plusieurs formes : *Russian Empire*, *Soviet Union*, *Russian Socialist Federative Soviet Republic*, et d’autres appellations.  

Ces variations ne concernent pas seulement ces deux pays ; plusieurs nations ont vu leurs noms et frontières évoluer au fil du temps. Cette diversité dans les appellations nous permet de retracer ces changements historiques sur une même zone géographique.  

Par ailleurs, certains grimpeurs portent plusieurs citoyennetés correspondant à différents noms d’un même pays au cours de leur vie, reflétant ainsi les transformations politiques et territoriales qu’ils ont traversées.



### 2) Y a-t-il des différences de genre selon les disciplines (bloc, difficulté, vitesse) ?

*Méthode :*

* Analyse croisée entre le genre et la discipline principale.
* Tableau de contingence + test du Chi².
* Objectif : voir s’il y a une répartition genrée dans les différentes sous-disciplines.

### 3. Est-ce qu'il y a un plus grosse proportion de femmes grimpeuses à partir d'une certaine année ? 


## Conclusion
Limitations ?
