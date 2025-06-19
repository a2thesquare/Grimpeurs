Basé sur le fichier `wdt_import_citizenships.sparqlbook.md`

## Nombre de personnes dans notre population
```sparql
### Number of persons in our population
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
        {?s a wd:Q5.}
}
```
## Quelque exemples de personnes
```sparql
### Some examples of persons
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?s ?label ?birthYear
WHERE {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
        {?s a wd:Q5;
            rdfs:label ?label;
            wdt:P569 ?birthYear}
}
ORDER BY ?s
LIMIT 3
```
## Prepare et inspecte les données à etre importées

```sparql
### Prepare and inspect the data to be imported


PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

CONSTRUCT {?item wdt:P27 ?citizenship.
            ?citizenship rdfs:label ?citizenshipLabel}
WHERE
    {
        GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>

        ## Find the persons in the imported graph
        {SELECT ?item
        WHERE 
                {?item a wd:Q5.}
        ORDER BY ?item      
        OFFSET 0
        #OFFSET 10000
        LIMIT 10

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                ?item wdt:P27 ?country.
                BIND (?citizenshipLabel as ?citizenshipLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
            }
                
        }
```
## Jsp

```sparql
### To be sure, the insert query has to be carried out directly on the Allegrograph server
# but it also could work if executed in this notebook
## Also, you have to carry it out in three steps. The accepted limit by Wikidata 
## of instances in a variable ('item' in our case) appears to be 10000.
## You therefore have to have three steps for a population of around 23000 persons  

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>


WITH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
INSERT {?item wdt:P27 ?citizenship.}
WHERE
    {
        ## Find the persons in the imported graph
        {SELECT ?item
        WHERE 
                {?item a wd:Q5.}
        ORDER BY ?item      
        #OFFSET 8000
        #OFFSET 16000
        #OFFSET 24000
        #OFFSET 32000
        LIMIT 10000

        }
        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                ?item wdt:P27 ?citizenship.
            }
                
        }
```
## Insertion du label de la propriété
```sparql
### Insert the label of the property
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>


INSERT DATA {
  GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
  {wdt:P27 rdfs:label 'country of citizenship'.}
}
```
## Obtention du nombre de citoyenneté crées
```sparql
### Get the number of created 'citizenships'
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>


    SELECT (COUNT(*) as ?n) 
    WHERE {
        GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
            {
                ?s wdt:P27 ?o.
            }
            }
```
## Nombre de personnes sans pays de citoyenneté
```sparql
## Persons without a country of citizenship
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT (COUNT(*) as ?n)
WHERE 
{GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
        
    {?item a wd:Q5;
        rdfs:label ?label.
    MINUS {
            ?item wdt:P27 ?country   .
        }     
    }
}
```
## Test une personne spécifique
```sparql
### test a specific person

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>


SELECT ?item ?o ?p ?statement_o
    {

        ## 
        SERVICE <https://query.wikidata.org/sparql>
            {
                 
                BIND(<http://www.wikidata.org/entity/Q1001072> as ?item)
                {
                    ?item ?p ?statement_o.
                    FILTER(contains(str(?p), 'P27'))
                }
                OPTIONAL{
                    ?item wdt:P27 ?o.
                }

            }
                
        }
```
## Obtenir la valeur du pays 
### Get the country value
```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX p: <http://www.wikidata.org/prop/>
PREFIX ps: <http://www.wikidata.org/prop/statement/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>



SELECT ?item ?os ?osLabel

WHERE
    {

        ## 
        SERVICE <https://query.wikidata.org/sparql>
        {
            {

                BIND(<http://www.wikidata.org/entity/Q1001072> as ?item)
                ?item p:P27 [ps:P27 ?os]

                BIND(?osLabel AS ?osLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }
            }        
                
        }
    }
```
## Obtenir la valeur du pays (again?)
```sparql
### Get the country value
PREFIX franzOption_serviceTimeout: <franz:240>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX p: <http://www.wikidata.org/prop/>
PREFIX ps: <http://www.wikidata.org/prop/statement/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>



SELECT ?item ?os ?osLabel

WHERE
    {

        {SELECT ?item
        WHERE 
                {?item a wd:Q5.
                MINUS {
                        ?item wdt:P27 ?country   .
                    }  
        }
                
        ORDER BY ?item  
       
        }

        ## 
        SERVICE <https://query.wikidata.org/sparql>
        {
            {

                ?item p:P27 [ps:P27 ?os]

                BIND(?osLabel AS ?osLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }
            }        
                
        }
    }
    LIMIT 20
```

## Propriétés d'une personne
```sparql
### Basic query about persons' properties

PREFIX franzOption_defaultDatasetBehavior: <franz:rdf>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?p ?label (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
        {?s a wd:Q5;
            ?p ?o.
        OPTIONAL {?p rdfs:label ?label}    
          }
}
GROUP BY ?p ?label
ORDER BY DESC(?n)
```
## Obtenir les labels des différents pays 
### Get the labels of the countries 
# Prepare the insert

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

CONSTRUCT  {
    ?country rdfs:label ?countryLabel.
}
#SELECT DISTINCT ?country ?countryLabel
WHERE {

    {
    SELECT DISTINCT ?country
    WHERE {
        GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
            {
                ?s wdt:P27 ?country.
            }
            }
    LIMIT 5
    }

    SERVICE <https://query.wikidata.org/sparql>
                {
                BIND (?country as ?country)
                BIND (?countryLabel as ?countryLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
                }



}
```
## Jsp encore une fois
```sparql
### Execute the INSERT, from the sparqlbook or on Allegrograph

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

WITH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md> 
INSERT  {
    ?citizenship rdfs:label ?citizenshipLabel.
}
WHERE {

    {
    SELECT DISTINCT ?citizenship
    WHERE {
            {
                ?s wdt:P27 ?citizenship.
            }
          }
    }

    SERVICE <https://query.wikidata.org/sparql>
                {
                BIND (?citizenship as ?citizenship)
                BIND (?citizenshipLabel as ?citizenshipLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
                }



}
```

## Inspection des citoyennetés

```sparql
###  Inspect the citizenships
# number of persons having this citizenship

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>


SELECT ?citizenship ?citizenshipLabel (COUNT(*) as ?n)
WHERE {
GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
{
   ?s wdt:P27 ?citizenship.
   ?citizenship rdfs:label ?citizenshipLabel.
}

}
GROUP BY ?citizenship ?citizenshipLabel
ORDER BY DESC(?n)
limit 20
```

## Inspection des pays, nombre de pays différents
```sparql
###  Inspect the countries:
# number of different countries

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT (COUNT(*) as ?n)
WHERE
   {
   SELECT DISTINCT ?country
   WHERE {
      GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
         {
            ?s wdt:P27 ?country.
         }
      }
   }
```
## Insertion de la classe "country" pour chaque pays
```sparql
### Insert the class 'country' for all countries
# Please note that strictly speaking Wikidata has no ontology,
# therefore no classes. We add this for our convenience

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

WITH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
INSERT {
   ?country rdf:type wd:Q6256.
}
WHERE
   {
   SELECT DISTINCT ?country
   WHERE {
         {
            ?s wdt:P27 ?country.
         }
      }
   }
```
## Ajout de label 
```sparql
### Ajouter le label pour le concept Country

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

INSERT DATA {
GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
    {    wd:Q6256 rdfs:label "Country".
    }    
}

```
## Personnes avec plus d'une citoyenneté

```sparql
### Persons with more than one citizenship
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>


SELECT ?item (COUNT(*) as ?n) ( GROUP_CONCAT(?citizenshipLabel; separator=", ") AS ?countries )
WHERE {
GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
{
   ?item wdt:P27 ?citizenship.
    ?citizenship rdfs:label ?citizenshipLabel.
}

}
GROUP BY ?item
HAVING (?n > 1)
ORDER BY DESC(?n)
OFFSET 10
LIMIT 5
```

## Jspppp

```sparql
### Number of persons with more than one citizenship
# We see that we have an issue: 1/5 of population with more than one citizenship
# How to treat this ?

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT (COUNT(*) AS ?no)
WHERE {
    SELECT ?item (COUNT(*) as ?n)
    WHERE {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
    {
    ?item wdt:P27 ?citizenship.
        ?citizenship rdfs:label ?citizenshipLabel.
    }

    }
    GROUP BY ?item
    HAVING (?n > 1)
}
```
## Obtention des continents

```sparql
### Get the continents — prepare the data

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

CONSTRUCT  {
    ?citizenship wdt:P30 ?continent.
    ?continent rdfs:label ?continentLabel.
}
#SELECT DISTINCT ?citizenship ?citizenshipLabel
WHERE {

    {
    SELECT DISTINCT ?citizenship
    WHERE {
        GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
            {
                ?s wdt:P27 ?citizenship.
            }
            }
    LIMIT 5
    }

    SERVICE <https://query.wikidata.org/sparql>
                {

                ?citizenship wdt:P30 ?continent.
                # BIND (?continent as ?citizenship)
                BIND (?continentLabel as ?continentLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
                }



}
```

## Obtention des labels des pays ou citoyennetés
```### Get the labels of the countries or citizenships
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

WITH  <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>   
INSERT  {
    ?citizenship wdt:P30 ?continent.
    ?continent rdfs:label ?continentLabel.
}
#SELECT DISTINCT ?citizenship ?citizenshipLabel
WHERE {

    {
    SELECT DISTINCT ?citizenship
    WHERE {
            {
                ?s wdt:P27 ?citizenship.
            }
            }
    }

    SERVICE <https://query.wikidata.org/sparql>
                {

                ?citizenship wdt:P30 ?continent.
                # BIND (?continent as ?citizenship)
                BIND (?continentLabel as ?continentLabel)
                SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 
                }



}
```
## Insertion du label de propriété

```sparql
### Insert the property label
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>


INSERT DATA {
  GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
  {wdt:P30 rdfs:label 'continent'.}
}
```

## Inspection des continents

```sparql
###  Inspect the continents:
# number of different continents

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT (COUNT(*) as ?n)
WHERE
   {
   SELECT DISTINCT ?continent
   WHERE {
      GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
         {
            ?s wdt:P30 ?continent.
         }
      }
   }
```

## Insertion de la classe continent
```sparql
## Insert the class 'continent' for all continents
# Please note that strictly speaking Wikidata has no ontology,
# therefore no classes. We add this for our convenience

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

WITH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
INSERT {
   ?continent rdf:type wd:Q5107.
}
WHERE
   {
   SELECT DISTINCT ?continent
   WHERE {
         {
            ?s wdt:P30 ?continent.
         }
      }
   }
```
## Bruh

```sparql
### Ajouter le label pour la classe "Continent"

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

INSERT DATA {
GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
    {    wd:Q5107 rdfs:label "Continent".
    }    
}
```

## Inspections des continents 

```sparql
###  Inspect the persons in continents
# number of persons having this citizenship

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>


SELECT ?continent ?continentLabel (COUNT(*) as ?n)
WHERE {
GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
{
   ?s wdt:P27 ?country.
   ?country wdt:P30 ?continent.
   ?continent rdfs:label ?continentLabel.
}

}
GROUP BY ?continent ?continentLabel
ORDER BY DESC(?n)
```

## Personnes avec plus d'une citoyenneté
```sparql
### Persons with more than one citizenship
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>


SELECT ?item (COUNT(*) as ?n) ( GROUP_CONCAT(?continentLabel; separator=", ") AS ?continents )
    ( GROUP_CONCAT(?countryLabel; separator=", ") AS ?countries )
WHERE {
    SELECT DISTINCT ?item ?continentLabel ?countryLabel
    WHERE 
        {
        GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
            {
            ?item wdt:P27 ?country.
            ?country wdt:P30 ?continent;
                rdfs:label ?countryLabel.
            ?continent rdfs:label ?continentLabel.
            ## Excluding Eurasia, Australia and Oceania insular
            FILTER ( ?continent NOT IN (wd:Q538, wd:Q3960, wd:Q5401))
            }
        }
}
GROUP BY ?item
HAVING (?n > 1)
ORDER BY DESC(?n)
#OFFSET 10
LIMIT 10

```
