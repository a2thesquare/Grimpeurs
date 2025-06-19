Application du document `wdt_import_population.sparqlbook` sur ma base de données de grimpeurs (climbers, mountaineers et rock climbers).
# Rend data base correcte 
```sparql
PREFIX franzOption_serviceTimeout: <franz:120>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT DISTINCT ?item ?itemLabel ?genderLabel (xsd:integer(?year) AS ?birthYear)
WHERE {
  SERVICE <https://query.wikidata.org/sparql> {
    { ?item wdt:P106 wd:Q82594 }  # Mountaineer
    UNION
    { ?item wdt:P106 wd:Q2374149 } # Climber
    UNION
    { ?item wdt:P106 wd:Q10873124 } # Rock Climber

    ?item wdt:P31 wd:Q5;  # Ensure they are humans
          wdt:P569 ?birthDate;
          wdt:P21 ?gender.

    # Extract only the year part from the birth date
    BIND(YEAR(?birthDate) AS ?year)

    # Ensure the birth year is within range
    FILTER(?year > 1750 && ?year < 1951)

    # Get readable labels instead of Wikidata IDs
    SERVICE wikibase:label { 
      bd:serviceParam wikibase:language "en".
      ?item rdfs:label ?itemLabel.
      ?gender rdfs:label ?genderLabel.
    }
  }
}
```

# Ajoutte un label à la classe "Personne"
```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

INSERT DATA {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
    {
        wd:Q5 rdfs:label "Person".
    }
}
```
# Ajoutte la classe "Genre"
```sparql
###  Inspect the genders:
# number of different countries

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT (COUNT(*) as ?n)
WHERE
   {
   SELECT DISTINCT ?gender
   WHERE {
      GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
         {
            ?s wdt:P21 ?gender.
         }
      }
   }
```
# Inserer la classe genre pour tout types de genre 
```sparql
### Insert the class 'gender' for all types of gender
# Please note that strictly speaking Wikidata has no ontology,
# therefore no classes. We add this for our convenience

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

WITH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
INSERT {
   ?gender rdf:type wd:Q48264.
}
WHERE
   {
   SELECT DISTINCT ?gender
   WHERE {
         {
            ?s wdt:P21 ?gender.
         }
      }
   }
```
```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

INSERT DATA {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
    {
        wd:Q48264 rdfs:label "Gender Identity".
    }
}
```
## Bug: personnes sans genre déclaré, combien ?
```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
SELECT ?o (COUNT(*) as ?x) WHERE {
  GRAPH ?g {
    wd:Q21518962 ?p ?o .
  }
}
GROUP BY ?o
```
## Enlever cette personne
```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
DELETE WHERE {
  GRAPH ?g {
    wd:Q21518962 ?p ?o .
  }
}
```
# Verifier triplets importés et ajoutter labels aux genres
```sparql
### Number of triples in the graph
SELECT (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
        {?s ?p ?o}
}
```
```sparql
### Number of persons with more than one label : no person
SELECT (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
        {?s rdf:label ?o}
}
GROUP BY ?s
HAVING (?n > 1)
```
# Explorer le genre
```sparql
### Number of persons having more than one gender
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?s (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
        {?s wdt:P21 ?gen}
}
GROUP BY ?s
HAVING (?n > 1)
```
```sparql
### Number of persons per gender
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?gen (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
        {?s wdt:P21 ?gen}
}
GROUP BY ?gen
#HAVING (?n > 1)
```
```sparql
### Number of persons per gender in relation to a period
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?gen (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
        {?s wdt:P21 ?gen;
            wdt:P569 ?birthDate.
        FILTER (?birthDate < '1900')     
          }
}
GROUP BY ?gen
#HAVING (?n > 1)
```
# Ajoutter des labels aux genres 
```sparql
### Add the label to the gender

# This query will first retrieve all the genders, 
# then fetch in Wikidata the gender's labels

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?gen ?genLabel
WHERE {

    

    {SELECT DISTINCT ?gen
    WHERE {
        GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
            {?s wdt:P21 ?gen}
    }
    }   

    SERVICE  <https://query.wikidata.org/sparql> {
        ## Add this clause in order to fill the variable      
        BIND(?gen as ?gen)
        BIND ( ?genLabel as ?genLabel)
        SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }  
    }
}

## Check how many people with no gender there are (mine)
PREFIX wd: <http://www.wikidata.org/entity/>
SELECT ?o (COUNT(*) as ?x) WHERE {
  GRAPH ?g {
    wd:Q21518962 ?p ?o .
  }
}
GROUP BY ?o

## Remove the guy with no gender 
PREFIX wd: <http://www.wikidata.org/entity/>
DELETE WHERE {
  GRAPH ?g {
    wd:Q21518962 ?p ?o .
  }
}
```
```sparql
### Add the label to the gender

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

CONSTRUCT {
     ?gen rdfs:label ?genLabel
    
} 
WHERE {

    

    {SELECT DISTINCT ?gen
    WHERE {
        GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
            {?s wdt:P21 ?gen}
    }
    }   

    SERVICE  <https://query.wikidata.org/sparql> {
        ## Add this clause in order to fill the variable      
        BIND(?gen as ?gen)
        BIND ( ?genLabel as ?genLabel)
        SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }  
    }
}
```
# Preparer les données pour l'analyse
```
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>


SELECT ?s ?label ?birthDate ?genLabel
WHERE {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
        {
            ## A property path passes through 
            # two or more properties
            ?s wdt:P21 / rdfs:label ?genLabel;
            rdfs:label ?label;
            wdt:P569 ?birthDate.
          }
}
ORDER BY ?birthDate
LIMIT 10
```
```sparql
### Number of persons

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT (COUNT(*) as ?n)
WHERE {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
        {
          # ?s wdt:P31 wd:Q5 
          ?s a wd:Q5
          }
}
```
```sparql
### Personnes avec choix aléatoire de modalités pour variables doubles

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>


SELECT  ?s (MAX(?label) as ?label) (xsd:integer(MAX(?birthDate)) as ?birthDate) 
    (MAX(?gen) as ?gen) (MAX(?genLabel) AS ?genLabel)
WHERE {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
        {?s wdt:P21 ?gen;
            rdfs:label ?label;
            wdt:P569 ?birthDate.
        ?gen rdfs:label ?genLabel    
          }
}
GROUP BY ?s
LIMIT 10
```
```sparql
### Nombre de personnes avec propriétés de base sans doublons (choix aléatoire par MAX)

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT (COUNT(*) as ?n)
WHERE {
SELECT  ?s (MAX(?label) as ?label) (YEAR(MAX(?birthDate)) as ?birthYear)

            (MAX(?gen) as ?gen) (MAX(?genLabel) AS ?genLabel)
WHERE {
    GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
        {?s wdt:P21 ?gen;
            rdfs:label ?label;
            wdt:P569 ?birthDate.
          }
}
GROUP BY ?s
}
```
```sparql
### Ajouter le label pour la propriété "date of birth"

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

INSERT DATA {
GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
{    wdt:P569 rdfs:label "date of birth"
}    
}
```
```sparql
### Nombre de personnes avec propriétés de base sans doublons (choix aléatoire)

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

INSERT DATA {
GRAPH <https://github.com/a2thesquare/Grimpeurs/blob/main/graphs/wikidata_imported_data.md>
{    wdt:P21 rdfs:label "sex or gender"
}    
}
```
