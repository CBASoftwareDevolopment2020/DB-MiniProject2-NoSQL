  
  
#  Mini Project 2: NoSQL Databases
  
  
By **Daniel, Jakob, Nikolaj & Stephan**  
Institute **CPHBusiness**  
  
Education **Software Development**  
Course **Databases**  
  
###  Objective
  
The objective of this assignment is to provide you with experience in analysis and design of database
applications that can benefit from NoSQL databases.
  
###  Task
  
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
  
###  Notes
  
This is a group assignment. Some groups will be asked to present their solutions to the class.
There is no requirement for developing client applications, code on a console or dashboard would be
good enough.
  
There is no requirement for writing report, results are to be presented in a readme file.
  
##  Content
  
  
- [Selected Databases and Data Source](#selected-databases-and-data-source )
  - [Neo4j](#neo4j )
  - [Mongo DB](#mongo-db )
  - [Data Source](#data-source )
- [Neo4j](#neo4j-1 )
  - [Load Data](#load-data )
  - [Queries](#queries )
  - [Performance & Storage](#performance-storage )
- [Mongo DB](#mongo-db-1 )
  - [Load Data](#load-data-1 )
  - [Queries](#queries-1 )
  - [Performance & Storage](#performance-storage-1 )
- [Evaluation](#evaluation )
  - [Syntax](#syntax )
  - [Storage & Performance](#storage-performance )
  - [Conclusion](#conclusion )
  
___
##  Selected Databases and Data Source
  
  
###  Neo4j
  
Neo4j is the graph database platform powering mission-critical enterprise applications like artificial intelligence, fraud detection and recommendations.
  
[reference, neo4j.com](https://neo4j.com/ )
  
###  Mongo DB
  
MongoDB is a general purpose, document-based, distributed database built for modern application developers and for the cloud era.
  
[reference, mongodb.com](https://www.mongodb.com/ )
  
###  Data Source
  
  
The movies dataset includes 81,273 movies with attributes such as movie description, average rating, number of votes, genre, etc.
  
The ratings dataset includes 81,273 rating details from demographic perspective.
  
The names dataset includes 175,719 cast members with personal attributes such as birth details, death details, height, spouses, children, etc.
  
The title principals dataset includes 377,848 cast members roles in movies with attributes such as IMDb title id, IMDb name id, order of importance in the movie, role, and characters played.
  
[source, kaggle.com](https://www.kaggle.com/stefanoleone992/imdb-extensive-dataset#IMDb%20movies.csv )
  
___
##  Neo4j
  
  
###  Load Data
  
  
Add `data/data.csv` to the import folder for the selected database.
  
**add nodes**  
_cypher shell_
```sql
LOAD CSV WITH HEADERS FROM 'file:///data.csv' AS row
MERGE (m:Movie {title: row.Title, description: row.Description, runtime: row.Runtime, rating: row. Rating, revenue: coalesce(row.Revenue, 0), metascore: coalesce(row.Metascore, 0)})
MERGE (e:Year {year: row.Year})
MERGE (d:Person {name: row.Director})
FOREACH(n IN split(row.Actors, ",")|MERGE (a:Person {name: trim(n)}))
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
UNWIND split(row.Actors, ",") AS n
MATCH (a:Person {name: trim(n)})
MERGE (m)-[:ACTED]-(a)
```
  
###  Queries
  
  
**Get name of persons who acted in a movie in 2006** | neo4j_query_1  
_cypher shell_
```sql
MATCH(Year {year: "2006"})-[:RELEASED]-(:Movie)-[:ACTED]-(p:Person) 
return p.name
```
  
**Get amount of persons that acted in a movie directed by David Yates** | neo4j_query_2  
_cypher shell_
```sql
MATCH (:Person {name: "David Yates"})-[:DIRECTED]-(:Movie)-[:ACTED]-(a:Person) 
RETURN count(distinct a)
```
  
**Get genres Christian Bale appeared in** | neo4j_query_3  
_cypher shell_
```sql
MATCH(p:Person {name:"Christian Bale"})-[:ACTED]-(:Movie)-[:GENRE]-(g:Genre)
return count(g), g.name
```
  
  
###  Performance & Storage
  
  
**Performance Time**
  
_python_
```python
driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "1234"))
  
# Get name of persons who acted in a movie in 2006
print("neo4j query 1", timeit(neo4j_query_1, number=5000))
  
# Get amount of persons that acted in a movie directed by David Yates
print("neo4j query 2", timeit(neo4j_query_2, number=5000))
  
# Get genres Christian Bale appeared in
print("neo4j query 3", timeit(neo4j_query_3, number=5000))
  
driver.close()
```
  
_response_
```bash
neo4j query 1: 6.957793999999922
neo4j query 2: 5.730350099999896
neo4j query 3: 5.3102682999999615
```
  
**Storage Size**  
_cypher shell_
```sql
:sysinfo
```
  
_response_
```sql
Store Size
Size	2.22 MiB
```
  
___
##  Mongo DB
  
  
  
###  Load Data
  
  
Add `data/data.json` to the selected database collection.
  
**add data**  
_bash shell_
```bash
mongoimport --db mini_project --collection main --file data/data.json --jsonArray
```
  
###  Queries
  
  
**Get name of persons who acted in a movie in 2006** | mongo_query_1  
_mongo shell_
```javascript
db.collection.aggregate([
    { '$match': { 'Year': 2006 } },
    { '$unwind': '$Actors' },
    { '$group': { '_id': None, 'res': { '$addToSet': '$Actors' } } }
])
```
  
**Get amount of persons that acted in a movie directed by David Yates**  | mongo_query_2  
_mongo shell_
```javascript
db.collection.aggregate([
    { '$match': { 'Director': 'David Yates' } },
    { '$unwind': '$Actors' },
    { '$group': { '_id': None, 'res': { '$addToSet': '$Actors' } } },
    { '$project': { 's': { '$size': '$res' } } }
]);
```
  
**Get genres Christian Bale appeared in** | mongo_query_3  
_mongo shell_
```javascript
db.collection.aggregate([
    { '$match': { 'Actors': { '$elemMatch': { '$eq': 'Christian Bale' } } } },
    { '$unwind': '$Genre' },
    { '$group': { '_id': None, 'res': { '$addToSet': '$Genre' } } }
]);
```
  
###  Performance & Storage
  
  
**Performance Time**
  
_python_
```python
mongo_client = MongoClient("mongodb://localhost:27017/")
mongo_db = mongo_client["mini_project"]
mongo_col = mongo_db["main"]
  
# Get name of persons who acted in a movie in 2006
print("mongo query 1", timeit(mongo_query_1, number=5000))
  
# Get amount of persons that acted in a movie directed by David Yates
print("mongo query 2", timeit(mongo_query_2, number=5000))
  
# Get genres Christian Bale appeared in
print("mongo query 3", timeit(mongo_query_3, number=5000))
```
  
_response_
```bash
mongo query 1: 23.863425428000028
mongo query 2: 20.93231320299992
mongo query 3: 22.32942827199986
```
  
  
**Get storage size**  
_mongo shell_
```js
db.collection.totalSize()
```
  
_response_
```
483.6 KB
```
  
___
##  Evaluation
  
  
###  Syntax
  
  
  
  
###  Storage & Performance
  
  
|   |Neo4j|Mongo DB|
|---|---|---|
|Storage|2.22 Mb|0.48 Mb|
|Time 500 Q1|6.958 s|23.863 s|
|Time 500 Q2|5.730 s|20.932 s|
|Time 500 Q3|5.310 s|22.329 s|
  
###  Conclusion
  
  