# TREE Reference-based View
Although a geo-spatial view is very precise and allows us to retrieve a subset by means of a few bounding boxes, it is non-trivial to calculate these square polygons. Instead, we can use the `tree:EqualToRelation` to create a view based on string, or even better, URI equality.

If our member models are [labelled](https://www.w3.org/TR/rdf-schema/#ch_label) or classified in another way using a (limited set of) string or URI value (e.g. "Brugge", "Gent", http://www.brugge.be, http://stad.gent ...), we can create the buckets labelled 'equal to "Brugge"', 'equal to http://stad.gent', etc. This allows us to partition our data set using any arbitrary classification.

Example reference-based view:
```
@prefix tree:      <https://w3id.org/tree#> .
@prefix sh:        <http://www.w3.org/ns/shacl#> .
@prefix schema:    <http://schema.org/> .
@prefix dct:       <http://purl.org/dc/terms/> .
@prefix xsd:       <http://www.w3.org/2001/XMLSchema#> .

@prefix ex:        <http://example.org/> .
@prefix disney:    <http://example.org/DisneyFeed/> .
@prefix by-gender: <http://example.org/DisneyFeed/by-gender/> .

ex:DisneyFeed a tree:Collection ;
  tree:view disney:by-gender .

disney:by-gender a tree:Node ;
  tree:relation [ 
      a tree:EqualToRelation ; 
      tree:path schema:gender ; 
      tree:value "female" ; 
      tree:node by-gender:female 
  ] , [ 
      a tree:EqualToRelation ; 
      tree:path schema:gender ; 
      tree:value "male" ; 
      tree:node by-gender:male 
  ] .
```

```mermaid
flowchart LR
    disney((ex:DisneyFeed))
    collection(("`tree:
    Collection`"))
    view(("`disney:
    by-gender`"))
    pFemale(("`by-gender:
    female`"))
    pMale(("`by-gender:
    male`"))
    node(("`tree:
    Node`"))
    rel1(("`[ ]`"))
    rel2(("`[ ]`"))
    relation(("`tree:
    EqualTo
    Relation`"))
    female(('female'))
    male(('male'))
    gender(("`schema:
    gender`"))
    disney -- a --> collection
    disney -- tree:view --> view
    view -- a --> node
    view -- tree:relation --> rel1
    view -- tree:relation --> rel2
    rel1 -- tree:value --> female
    rel1 -- tree:node --> pFemale
    rel1 -- a --> relation
    rel2 -- a --> relation
    rel1 -- tree:path --> gender
    rel2 -- tree:path --> gender
    rel2 -- tree:node --> pMale
    rel2 -- tree:value --> male
```
Fig 1. Example Reference-based View

> [!IMPORTANT]
> A _reference-based view_ allows us to create a (shallow) _hierarchical structure_ where we can _filter a data set based on some classification_.

---
<p align="right">Next: <a href="K-retention-policies.md">LDES Retention Policies</a></p>
