# LDES Time-based Retention
Another typical use case for retention is the availability of members for a relative time interval, e.g. keep members for a day, a week, a month, a year, etc.

Instead of using an absolute point-in-time, this policy uses a dynamic point-in-time as a cutoff point on which members to keep and which to remove from the view. This dynamic point is simply some period of time before the `ldes:timestamp` of the latest published member, effectively creating a sliding time-window as members are added.

To define this retention, the LDES specification defines a `ldes:fullLogDuration` predicate specifying a duration (`xsd:duration` - see [ISO 8601 durations](https://en.wikipedia.org/wiki/ISO_8601#Durations)). This duration is subtracted from the last member's timestamp to calculate the dynamic cutoff point. Members whose `ldes:timestampPath` results in a `xsd:dateTime` that is _equal or higher than this dynamic value_ will be retained (available) in the view.

> [!TIP]
> This retention is _similar_ to the obsolete retention policy of type `ldes:DurationAgoPolicy`, which has a predicate `tree:value` whose value is also a duration value (`xsd:duration`). A data client must interpret this predicate as the `ldes:fullLogDuration` predicate value and map it if needed.

> [!NOTE]
> There is a difference in behavior with the obsolete retention policy of type `ldes:DurationAgoPolicy`.
> 
> In previous LDES versions, the dynamic cutoff point was calculated based on _the current date and time_ resulting in a sliding time window relative to the actual date and time and therefore maintaining a history of members for the given amount of time _only_ if new members were being published. If publishing paused or stopped, the oldest members were removed possibly leaving the LDES empty after a while.
> 
> Now, the policy uses the _last member's timestamp_ to calculate the dynamic cutoff point. As long as new members are being published, this is roughly the same (depending on the frequency of publication). However, for a event stream that pauses or stops publishing members, the retention policy now guarantees to _always_ keep the most recent members that fall within the relative time period before the last published member. Therefore, the LDES will not be emptied.

```
@prefix tree:      <https://w3id.org/tree#> .
@prefix ldes:      <https://w3id.org/ldes#> .
@prefix sh:        <http://www.w3.org/ns/shacl#> .
@prefix schema:    <http://schema.org/> .
@prefix dct:       <http://purl.org/dc/terms/> .
@prefix xsd:       <http://www.w3.org/2001/XMLSchema#> .
@prefix wiki:      <http://en.wikipedia.org/wiki/> .
@prefix disney:    <http://en.wikipedia.org/wiki/disney/> .

wiki:disney a ldes:EventStream ;
  tree:shape [ a sh:NodeShape; sh:targetClass schema:Person ] ;
  ldes:versionOfPath dct:isVersionOf ;
  ldes:timestampPath dct:created ;
  tree:view disney:sliding-time-window .

disney:sliding-time-window a tree:Node ;
  ldes:retentionPolicy [
    ldes:fullLogDuration "P10Y"^^xsd:duration 
  ] .
```

The above `disney:sliding-time-window` view applied to [this example](./E-ldes-specs.md#naming-members) would only contain member `wiki:Minnie_Mouse#v2` (last published member) because any member with a `dct:created` value before its timestamp minus 10 years is removed. This remaining member will never be removed (as it is the last published one).

> [!IMPORTANT]
> We can use a _retention policy with a `ldes:fullLogDuration` predicate_ to _keep (retain) all members on or after a time period relative before the last published member_.
>
> This retention predicate is only influenced by the `ldes:startingFrom` predicate: if that _absolute timestamp falls within the relative time period_ before the last member's timestamp then those members before this absolute timestamp are _not retained_.
