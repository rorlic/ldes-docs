# LDES Time-based Retention
Another typical use case for retention is the availability of members for a relative time interval, e.g. keep members for a day, a week, a month, a year, etc.

Instead of using an absolute point-in-time, this policy uses a dynamic point-in-time as a cutoff point on which members to keep and which to remove from the view. This dynamic point is simply some period of time before the `ldes:timestamp` of the latest published member, effectively creating a sliding time-window as members are added.

To define this retention, the LDES specification defines a `ldes:fullLogDuration` predicate specifying a duration (`xsd:duration` - see [ISO 8601 durations](https://en.wikipedia.org/wiki/ISO_8601#Durations)). This duration is subtracted from the _current date and time_ to calculate the dynamic cutoff point.

> [!NOTE]
> Members whose `ldes:timestampPath` results in a `xsd:dateTime` that is _equal or higher than this dynamic value_ will be retained (available) in the view.
> 
As the dynamic cutoff point is calculated based on the current date and time, it in fact creates a sliding time window maintaining a history of members for the given amount of time when new members are being published. If publishing is paused or stopped, the oldest members are removed possibly leaving the event stream empty after a while.

> [!TIP]
> This retention is identical to the obsolete retention policy of type `ldes:DurationAgoPolicy`, which has a predicate `tree:value` whose value is also a duration value (`xsd:duration`). A data client must interpret this predicate as the `ldes:fullLogDuration` predicate value and map it if needed.

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
  tree:view disney:sliding-time-window .

disney:sliding-time-window a tree:Node ;
  ldes:retentionPolicy [
    ldes:fullLogDuration "P30Y"^^xsd:duration 
  ] .
```

The above `disney:sliding-time-window` view applied to [this example](./E-ldes-specs.md#naming-members) would only contain member `wiki:Minnie_Mouse#v2` (last published member) because any member with a `dct:created` value before today (June 27th, 2025) minus 30 years is removed. This remaining member should be removed in about five years (around January 1st, 2030).

> [!NOTE]
> This retention predicate is _influenced_ by the `ldes:startingFrom` predicate: if that _absolute timestamp falls within the sliding time window_ then those members before this absolute timestamp are _not retained_.

The reason for this behavior is that the two following conditions need to be checked:

* C1: Members whose `ldes:timestampPath` results in a `xsd:dateTime` that is _equal or higher than the absolute timestamp_ given by _`ldes:startingFrom`_ will be retained (available) in the view.
* C2: Members whose `ldes:timestampPath` results in a `xsd:dateTime` that is _equal or higher than the dynamic cutoff point_ given by _t<sub>now</sub> minus `ldes:fullLogDuration`_ will be retained (available) in the view.

Basically, there are only two options:
1. the absolute timestamp falls before the dynamic cutoff point, OR
2. the absolute timestamp falls on or after the dynamic cutoff point

In the first case, the absolute value has no effect on which members are retained or not (C2), while in the second case, the members between the cutoff point an the absolute timestamp are _not_ retained by the first condition above (C1).

Technically speaking, the absolute timestamp can be a future date and in that case, by definition, it falls after the sliding time window. In this case, the first condition (C1) still applies and the second condition (C2) does not, which results in an empty view until that absolute moment in time.

> [!IMPORTANT]
> We can use a _retention policy with a `ldes:fullLogDuration` predicate_ to _keep (retain) all members on or after a time period relative before the current date and time_.

---
<p align="right">Next: <a href="N-version-based-retention.md">LDES Version-based Retention</a></p>
