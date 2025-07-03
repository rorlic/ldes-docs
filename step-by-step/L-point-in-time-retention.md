# LDES Point-in-time Retention
One of the most simple use cases for retention is the availability of members after some absolute point in time which marks some event, some change in legislation, etc.

To define this retention, the LDES specification defines a `ldes:startingFrom` predicate specifying the absolute point in time as a `xsd:dateTime` value.

> [!NOTE]
> Members whose `ldes:timestampPath` results in a `xsd:dateTime` that is _equal or higher than this absolute value_ will be retained (available) in the view.

Technically speaking, the absolute timestamp could be a value in the future and until that moment the view will contain no members. A possible use case is that a view contains members after some future legislation date, or members representing future car models or accessories, future prices, etc.

> [!TIP]
> This retention is identical to the obsolete retention policy of type `ldes:PointInTimePolicy`, which has a predicate `ldes:pointInTime` whose value is also a timestamp (`xsd:dateTime`). A data client must interpret this predicate as the `ldes:startingFrom` predicate value and map it if needed.

```
@prefix tree:      <https://w3id.org/tree#> .
@prefix ldes:      <https://w3id.org/ldes#> .
@prefix sh:        <http://www.w3.org/ns/shacl#> .
@prefix schema:    <http://schema.org/> .
@prefix dct:       <http://purl.org/dc/terms/> .
@prefix xsd:       <http://www.w3.org/2001/XMLSchema#> .
@prefix ex:        <http://example.org/> .
@prefix disney:    <http://example.org/disney/> .

ex:DisneyFeed a ldes:EventStream ;
  tree:shape [ a sh:NodeShape; sh:targetClass schema:Person ] ;
  ldes:versionOfPath dct:isVersionOf ;
  ldes:timestampPath dct:created ;
  tree:view disney:point-in-time .

disney:point-in-time a tree:Node ;
  ldes:retentionPolicy [
    ldes:startingFrom "1950-01-01T00:00:00Z"^^xsd:dateTime 
  ] .
```

The above `disney:point-in-time` view applied to [this example](./E-ldes-specs.md#naming-members) would only contain members `wiki:Minnie_Mouse#v2` and `wiki:Mickey_Mouse#v2` because the other two members have a `dct:created` value which is lower than our `ldes:startingFrom` value.

> [!IMPORTANT]
> We can use a _retention policy with a `ldes:startingFrom` predicate_ to _keep (retain) all members on or after an absolute date and time_.

---
<p align="right">Next: <a href="M-time-based-retention.md">LDES Time-based Retention</a></p>
