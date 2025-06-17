# Rend data base correcte 
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

    
