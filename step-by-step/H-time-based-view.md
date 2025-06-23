# TREE Time-based View
We can create a view that allows filtering our members on time (_historical filtering_) if we use some of the numerical comparison relations and a timestamp property of our members for the relation's `tree:path`. The relevant numerical comparison relations are:
* tree:GreaterThanRelation
* tree:GreaterThanOrEqualToRelation
* tree:LessThanRelation
* tree:LessThanOrEqualToRelation

With these relations we can create a structure where the collection view contains relations to a year value to do a first rough partitioning of our data members. Each year node can then be subdivided into its month values and further down to any required level by linking to day nodes, hour nodes, etc.

Node `disney:by-time` (the time-based view):
```
@prefix tree: <https://w3id.org/tree#> .
@prefix wiki: <http://en.wikipedia.org/wiki/> .
@prefix schema: <http://schema.org/> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix disney: <http://en.wikipedia.org/wiki/disney#> .
wiki:disney a tree:Collection ;
  tree:view disney:by-time .
disney:by-time a tree:Node ;
  tree:relation
      [ a tree:GreaterThanOrEqualRelation; tree:node disney:1900 ;
          tree:path schema:birthDate ; tree:value "1900-01-01"^^xsd:date ] , 
      [ a tree:LessThanRelation; tree:node disney:1900 ;
          tree:path schema:birthDate ; tree:value "2000-01-01"^^xsd:date ] ,
      [ a tree:GreaterThanOrEqualRelation; tree:node disney:2000 ;
          tree:path schema:birthDate ; tree:value "2000-01-01"^^xsd:date ] ,
      [ a tree:LessThanRelation; tree:node disney:2000 ;
          tree:path schema:birthDate ; tree:value "2100-01-01"^^xsd:date ] .
```
Node `disney:1900` (all Disney figures 'born' in the previous century):
```
@prefix tree: <https://w3id.org/tree#> .
@prefix wiki: <http://en.wikipedia.org/wiki/> .
@prefix schema: <http://schema.org/> .
@prefix void: <http://rdfs.org/ns/void#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix disney: <http://en.wikipedia.org/wiki/disney#> .
wiki:disney a tree:Collection ;
  void:subset disney:1900 .
disney:1900 a tree:Node ;
  tree:relation
      [ a tree:GreaterThanOrEqualRelation; tree:node disney:1928 ;
          tree:path schema:birthDate ; tree:value "1928-01-01"^^xsd:date ] , 
      [ a tree:LessThanRelation; tree:node disney:1928 ;
          tree:path schema:birthDate ; tree:value "1929-01-01"^^xsd:date ] .
```
Node `disney:1928` (all Disney figures 'born' in 1928):
```
@prefix tree: <https://w3id.org/tree#> .
@prefix wiki: <http://en.wikipedia.org/wiki/> .
@prefix schema: <http://schema.org/> .
@prefix void: <http://rdfs.org/ns/void#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix disney: <http://en.wikipedia.org/wiki/disney#> .
wiki:disney a tree:Collection ;
  void:subset disney:1928 ;
  tree:member wiki:Mickey_Mouse, wiki:Minnie_Mouse .
disney:1928 a tree:Node  .
wiki:Mickey_Mouse a schema:Person ; schema:birthDate "1928-11-18"^^xsd:date.
wiki:Minnie_Mouse a schema:Person ; schema:birthDate "1928-11-18"^^xsd:date .
```

```mermaid
flowchart LR
  collection
  view
  p1900["1900's"]
  p2000["2000's"]
  p1928["1928"]
  collection --> view
  view -- "date >= 1/1/1900" --> p1900
  view -- "date < 1/1/2000" --> p1900
  p1900 -- "date >= 1/1/1928" --> p1928
  p1900 -- "date < 1/1/1929" --> p1928
  view -- "date >= 1/1/2000" --> p2000
  view -- "date < 1/1/2100" --> p2000
```
Fig 1. Example of time-based structure

> [!NOTE]
> In this example we create a time-based view starting from a century and then by year. We could also create buckets for each decade, e.g. 1920-1930, 1930-1940, etc., or even buckets of non-equal size, e.g. 1950-1970, 1970-2000. Because we specify the boundaries explicitly a data client can determine the exact buckets that are needed.

## Summary
A _time-based view_ allows us to create a _hierarchical structure_ where we can _filter a data set based on a time interval_.

---
<p align="right">Next: <a href="I-geospatial-view.md">TREE Geospatial View</a></p>
