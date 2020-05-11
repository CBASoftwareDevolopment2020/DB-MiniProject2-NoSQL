---
export_on_save:
  markdown: true
markdown:
  path: README.md
  ignore_from_front_matter: true
---

# Mini Project 2: NoSQL Databases {ignore=true}

By **Daniel, Jakob, Nikolaj & Stephan**  
Institute **CPHBusiness**  

Education **Software Development**  
Elective **Databases**  

### Objective {ignore=true}
The objective of this assignment is to provide you with experience in analysis and design of database
applications that can benefit from NoSQL databases.

### Task {ignore=true}
Your task is to select two or more databases of different NoSQL types and to compare their features
and performance in storing, scaling, providing, and processing big data.

Your solution includes

* preparing a large data source and loading it into both databases

* selecting relevant database operations, which can be used to compare the databases  

* selecting appropriate criteria for comparison, such as access time, storage space,
complexity, versioning, security, or similar  

* creating demo code for testing the selected database operations against the selected
comparison criteria  

* reporting the results and conclusions.

It is recommended to consider using ACID and CAP as reference.

Your conclusion would contribute to the recommendations for making choices of database types for
specific use cases.

### Notes {ignore=true}
This is a group assignment. Some groups will be asked to present their solutions to the class.
There is no requirement for developing client applications, code on a console or dashboard would be
good enough.

There is no requirement for writing report, results are to be presented in a readme file.

## Content {ignore=true}
[TOC]

___
## Selected Databases and Data Source

### Neo4j
Neo4j is the graph database platform powering mission-critical enterprise applications like artificial intelligence, fraud detection and recommendations.

[reference, neo4j.com](https://neo4j.com/)

### Mongo DB
MongoDB is a general purpose, document-based, distributed database built for modern application developers and for the cloud era.

[reference, mongodb.com](https://www.mongodb.com/)

### Data Source

The movies dataset includes 81,273 movies with attributes such as movie description, average rating, number of votes, genre, etc.

The ratings dataset includes 81,273 rating details from demographic perspective.

The names dataset includes 175,719 cast members with personal attributes such as birth details, death details, height, spouses, children, etc.

The title principals dataset includes 377,848 cast members roles in movies with attributes such as IMDb title id, IMDb name id, order of importance in the movie, role, and characters played.

[source, kaggle.com](https://www.kaggle.com/stefanoleone992/imdb-extensive-dataset#IMDb%20movies.csv)

___
## Neo4j

### Load Data

Add `data.csv` to the import folder for the selected database.

**add nodes**  
_cypher shell_
```sql
LOAD CSV WITH HEADERS FROM 'file:///data.csv' AS row
MERGE (m:Movie {title: row.Title, description: row.Description, runtime: row.Runtime, rating: row. Rating, revenue: coalesce(row.Revenue, 0), metascore: coalesce(row.Metascore, 0)})
MERGE (e:Year {year: row.Year})
MERGE (d:Person {name: row.Director})
FOREACH(n IN split(row.Actors, ", ")|MERGE (a:Person {name: n}))
FOREACH(n IN split(row.Genre, ",")|MERGE (e:Genre {name: n}))
```

**add director and released to movie relations**  
_cypher shell_
```sql
LOAD CSV WITH HEADERS FROM 'file:///data.csv' AS row
MATCH (m:Movie {title: row.Title})
MATCH (d:Person {name: row.Director})
MATCH (y:Year {year: row.Year})
MERGE (d)-[:DIRECTED]-(m)
MERGE (m)-[:RELEASED]-(y)
```

**add genre to movies relations**  
_cypher shell_
```sql
LOAD CSV WITH HEADERS FROM 'file:///data.csv' AS row
MATCH (m:Movie {title: row.Title})
UNWIND split(row.Genre, ",") AS n
MATCH (g:Genre {name: n})
MERGE (m)-[:GENRE]-(g)
```

**add actor to movies relations**  
_cypher shell_
```sql
LOAD CSV WITH HEADERS FROM 'file:///data.csv' AS row
MATCH (m:Movie {title: row.Title})
UNWIND split(row.Actors, ", ") AS n
MATCH (a:Person {name: n})
MERGE (m)-[:ACTED]-(a)
```

### Queries

**Get name of persons who acted in a movie in 2006**  
_cypher shell_
```sql
MATCH(Year {year: "2006"})-[:RELEASED]-(:Movie)-[:ACTED]-(p:Person) 
return p.name
```

**Get amount of persons that acted in a movie directed by David Yates**
_cypher shell_
```sql
MATCH (:Person {name: "David Yates"})-[:DIRECTED]-(:Movie)-[:ACTED]-(a:Person) 
RETURN count(distinct a)
```

**Get genres Christian Bale appeared in**  
_cypher shell_
```sql
MATCH(p:Person {name:"Christian Bale"})-[:ACTED]-(:Movie)-[:GENRE]-(g:Genre)
return count(g), g.name
```


### Performance & Storage

**Performance Time**

_python_
```js
from timeit import timeit
from neo4j import GraphDatabase

driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "1234"))

def func1():
    with driver.session() as session:
        m ='''MATCH(y:Year)-[r:RELEASED]-(m:Movie)-[a:ACTED]-(p:Person)
        WHERE y.year = "2006"
        return p.name'''
        r = session.run(m)

def func2():
    with driver.session() as session:
        m ='''MATCH (d:Person)-[:DIRECTED]-(:Movie)-[:ACTED]-(a:Person) 
        WHERE d.name = "David Yates"
        RETURN count(distinct a)'''
        r = session.run(m)

def func3():
    with driver.session() as session:
        m ='''MATCH (:Person {name:"Christian Bale"})-[:ACTED]-(:Movie)-[:GENRE]-(g:Genre) 
        RETURN count(g), g.name'''
        r = session.run(m)

print("func1", timeit(func1, number=5000))
print("func2", timeit(func2, number=5000))
print("func3", timeit(func3, number=5000))

driver.close()
```

_response_
```sql
func1 6.957793999999922
func2 5.730350099999896
func3 5.3102682999999615
```

**Storage Size**
_cypher shell_
```sql
:sysinfo
```

_response_
```sql
Store Size
Size	23.35 MiB
```

___
## Mongo DB


### Load Data
**add data**  
_mongo shell_
```js
...
```

**alternative add data**  
_bash shell_
```bash
mongoimport --db DB_NAME --collection COL_NAME --file FILE_NAME.json --jsonArray
```

### Queries

**Get name of persons who acted in a movie in 2006**  
_mongo shell_
```js
...
```

**Get amount of persons that acted in a movie directed by David Yates**  
_mongo shell_
```js
...
```

**Get genres Christian Bale appeared in**  
_mongo shell_
```js
...
```


### Performance & Storage




**Get storage size**  
_mongo shell_
```js
db.collection.totalSize()
```

_response_
```
...
```

___
## Conclusion

### Syntax
### Storage
### Performance
### ACID & CAP Theorem

#### Acid
#### CAP Theorem


