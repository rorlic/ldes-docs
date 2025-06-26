# LDES Retention Policies
An event stream will by definition have an ever increasing number of members. Obviously this causes issues for both for a data publisher and a data client. The storage cost is the main problem on data publisher side, while replicating a huge data set is costly and time-consuming for a data client.

A view partitioned with the special relations (`rdf:subClassOf tree:Relation`) allows to retrieve a subset of the data set. This only partially solves the problem for a data client: if no view is defined that allows to approximate the required subset, a data client still needs to replicate the whole data set and filter on client side. In addition, view partitioning does not help with some use cases: a data client may only need the most recent members or a number of last members, may not be interested in entities that were deleted, etc. We need another way to support these use cases on data publisher side.

Moreover, a partitioned view does not solve the main problem for the data publisher: the data set can still grow to an unmanageable size, driving up storage costs. On data publisher side there are also use cases that cannot be solved with view partitioning, e.g. a data publisher may want to differentiate how much historical data is made available on a freeware basis as opposed to for paying clients, a data publisher may only want to offer the most recent members for fast-moving data, etc.

This is where retention policies come to aide: a data publisher can set a retention policy on a view to both manage how much history of the data set is available in a view and to actually purge data items that are unavailable in all views.

View retention basically means that a member is only available in the view for a limited amount of time. If no retention policy is put on a view, all members will be available forever in that view.

> [!TIP]
> Because an LDES can have multiple views, all members must be kept indefinitely unless a retention policy is put on _all_ views of the collection. In this case a data item can only be removed from actual storage if no retention policy applies to it.

By keeping a limited history of data items, a data publisher can manage the storage costs. A data client can use the retention policy definition to select the appropriate view or to archive the members for which the retention policy will expire and therefore may be unavailable in the future.

The LDES specification defines that a root node can contain a single predicate `ldes:retentionPolicy`, which refers to an entity that can contain zero or more retention related predicates to describing which members are available and for how long.

> [!TIP]
> The `ldes:retentionPolicy` can alternatively be supplied on the root node's `tree:viewDescription` instead of on the root node itself, but obviously not on both.

If the entity contains no predicates, no members will be available in the view. Adding predicates allows us to define which members must be retained in the view and will be available. Each predicate helps us to cover some use case: keep members from a fixed point in time, keep members for a fixed time interval, keep a limited number of versions for a member, keep deleted members for a fixed time interval, etc. However, by combining these retention predicates additional use cases are possible.

By adding (combining) retention predicates, typically we retain more members, but some combinations interact and therefore decrease the retained member count.

> [!IMPORTANT]
> We can use _retention policies_ to _manage the members that are available in a view_ and _keep the storage cost under control_ by actually removing members from a data set for which no retention policy applies.

> [!TIP]
> In previous LDES versions, the specification defined an abstract base class `ldes:RetentionPolicy` and three concrete types of retention policies to help with specific retention use cases: `ldes:PointInTimePolicy`, `ldes:DurationAgoPolicy` and `ldes:LatestVersionSubset`. These policies can easily be mapped onto the retention predicates in the current LDES specification.

We will cover these and some other retention use cases in subsequent steps in this guide.

> [!CAUTION]
> In previous LDES versions, you could override the `ldes:timestampPath` predicate on the retention policy level but this is _not_ allowed anymore to ensure that the retention policy always compares to the last member’s timestamp.
> 
> This timestamp is defined as the chronological order in which members of the event stream are added. Using a different timestamp would risk using a non-chronological timestamp even if it is theoretically chronological, e.g. out-of-order arrivals.

---
<p align="right">Next: <a href="L-point-in-time-retention.md">LDES Point-in-time Retention</a></p>
