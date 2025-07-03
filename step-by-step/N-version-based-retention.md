# LDES Version-based Retention
Instead of using an absolute timestamp or relative time duration, we can retain members with the most recent state changes.

This policy is based on the assumption that more recent data is the most relevant one. As explained in the [LDES Specification Basics](./E-ldes-specs.md) an LDES contains members which are in essence a full state representation at some point in time of an entity. All the state changes of an entity are related to each other by means of the LDES `ldes:versionOfPath` predicate path. In other words, all the state changes of an entity have the same value for this predicate path, which is the identity of the entity.

## Version-subset Retention
The LDES specification defines a `ldes:versionAmount` predicate specifying an amount (`xsd:integer`) of most recent versions to retain per entity. Conceptually, members are grouped by their `ldes:versionOfPath` predicate path value and then ordered (descending) by their `ldes:timestampPath` predicate path value after which the specified amount of newer entity versions is retained while the other, older entity versions, are dropped.

> [!NOTE]
> Members whose _0-based index_ in the above grouped and sorted subsets is _lower than the number of versions to keep_ will be retained (available) in the view.

> [!TIP]
> This retention is a _simplification_ of the obsolete retention policy of type `ldes:LatestVersionSubset`, which has an optional `ldes:amount` predicate (defaulting to one) specifying the number (`xsd:integer` larger than zero) of latest versions to keep per entity. A data client must interpret this predicate as the `ldes:versionAmount` predicate value and map it if needed.

> [!CAUTION]
> Both the `ldes:versionOfPath` and `ldes:timestampPath` predicates _can not be redefined_ on the policy level anymore.
> 
> In addition, `ldes:versionKey` is _not supported_ anymore.
>
> As opposed to the obsolete `ldes:amount` predicate, the `ldes:versionAmount` predicate does not and can not have a default value, therefore you explicitly need to set a value for `ldes:versionAmount`.

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
  tree:view disney:version-based .

disney:version-based a tree:Node ;
  ldes:retentionPolicy [
    ldes:versionAmount 2
  ] .
```
The above `disney:version-based` view applied to [this example](./E-ldes-specs.md#naming-members) would contain all current members. If at some time in the future a third version of an entity would be added, then the first version would be removed from the view.

## Current-state Retention
There is one special use case of the above (version-subset) retention that is very useful: a retention policy which only keeps the latest state of each entity. By doing this we actually drop the history of the entities and have a view on the _current state_ of the source system. This is what other mechanisms typically provide by default through an API. 

To implement this policy we simply set the `ldes:versionAmount` to one.

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
  tree:view disney:latest-state .

disney:latest-state a tree:Node ;
  ldes:retentionPolicy [
    ldes:versionAmount 1
  ] .
```

The above `disney:latest-state` view applied to [this example](./E-ldes-specs.md#naming-members) would only contain members `wiki:Minnie_Mouse#v2` and `wiki:Mickey_Mouse#v2` because these are the latest versions of the entities. If at some time in the future a new version of an entity would be added, then the old version of that entity would be removed from the view.

> [!NOTE]
> Although this policy only keeps one version of an entity, the LDES members are still a _version_ of an entity and not the entity itself. As we will see later, when a data client replicates and synchronizes an LDES, the members emitted need to be converted into their entity further downstream.

## Time-limited Retention
The retentions described above keep one or more versions of an entity in a view, but they do so for an unlimited period of time, i.e. the life-time of the event stream. Therefore, an LDES can still grow without limit if the number of entities increases.

In addition, there are use cases where data freshness is important and it may be pointless even to keep these versions for more than some period of time, e.g. keep the last sensor observation of each sensor in the stream for maximum 1 hour.

The LDES specification defines a `ldes:versionDuration` predicate specifying a duration (`xsd:duration`). This duration is subtracted from the _current date and time_ to calculate the cutoff point that should be used to further limit the entity versions to keep for the [Version-subset Retention](#version-subset-retention) (and consequently the [Current-state Retention](#current-state-retention)).

> [!NOTE]
> Members that are retained after applying the version-subset condition and whose `ldes:timestampPath` results in a `xsd:dateTime` that is _equal or higher_ than the _cutoff point_ will be retained (available) in the view.

```
@prefix tree:      <https://w3id.org/tree#> .
@prefix ldes:      <https://w3id.org/ldes#> .
@prefix sh:        <http://www.w3.org/ns/shacl#> .
@prefix schema:    <http://schema.org/> .
@prefix dct:       <http://purl.org/dc/terms/> .
@prefix xsd:       <http://www.w3.org/2001/XMLSchema#> .
@prefix ex:        <https://example.org/> .
@prefix sosa:      <http://www.w3.org/ns/sosa/> .

ex:observations a ldes:EventStream ;
  tree:shape [ a sh:NodeShape; sh:targetClass sosa:Observation ] ;
  ldes:versionOfPath dct:isVersionOf ;
  ldes:timestampPath sosa:resultTime ;
  tree:view ex:latest-hour .

ex:latest-hour a tree:Node ;
  ldes:retentionPolicy [
    ldes:versionAmount 1;
    ldes:versionDuration "PT1H"^^<xsd:duration>
  ] .
```

In this example, all entities are of type `sosa:Observation` and have a `sosa:resultTime` which is the result time of the observation and a `dct:isVersionOf` value which represents the sensor. The retention will only keep the last observation of a sensor, which is a member with the highest `sosa:resultTime` for each distinct `dct:isVersionOf` value, but only if it happened at most 1 hour ago, i.e. its `sosa:resultTime` is greater or equal than now minus one hour.

## Summary
> [!IMPORTANT]
> We can use a _retention policy with a `ldes:versionAmount` predicate_ to _keep a number of most recent versions of each entity_.
>
> We can use a _latest-state retention policy_ (with `ldes:versionAmount` set to 1) to _retain only the latest version of each entity_, which in essence just keeps the current/latest state of a data set, without the history of changes.
>
> We can combine both these policies with a `ldes:versionDuration` to _further limit the versions retained based on a duration_ against the current date and time, which basically creates a sliding time window (of most recent versions).

---
<p align="right">Next: <a href="P-metadata.md">LDES Availability & Discovery</a></p>
