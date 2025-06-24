# TREE Partitioning Types
The TREE specification defines a number of special relation types (`tree:GreaterThanRelation`, `tree:GeospatiallyContainsRelation`, ...) that allow to better define the nature of the relation between the nodes.

These relations have additional predicates defining a constraint on the members that can be reached by following the relation's link. Such a relation has a predicate `tree:path`, which is a [property path](https://www.w3.org/TR/shacl/#property-paths) of the data items, and a predicate `tree:value`, which is the value to compare with (after retrieving a data item's value by resolving the property path). By using these special relations we actually construct a _search tree_, which allows a data client to retrieve a subset of the data set (_filtering_).

These special [relation types](https://w3id.org/tree/specification/#Relation) allow to compare text, numerical and date/time values, in addition to do geo-spatial comparison. Using these relations we can define multiple views on a collection, each defining a different partitioning of the data set and as such allow filtering the data set in a different way.

In fact, we can look at this as creating a number of virtual buckets and sorting our data items into one or more of these buckets (_bucketizing_). We label each bucket virtually with the constraints applied when sorted a data item in that bucket. A data client can then use the bucket labels to search and use only the buckets of interest.

Basically, a specialized relation _constraints the data items that can be found_ by following the link to its node.

Since there are many relation types and we can even combine these relation, the possibilities are endless. For example, we could first partition by location (geo-spatial or reference) and then by time. There are even relations that allow filtering on prefix, postfix and sub-string of a string which can be used to build some kind of dictionary.

> [!NOTE]
> The TREE specification allows more than one root node and therefore multiple partitions on the same collection. A data publisher can offer partitions with different pages sizes or different search trees by using these special relation types.

The most important thing to remember is that relations allow to _prune the search tree_ and effectively allow a data client to _retrieve a subset of the data items_.

---
<p align="right">Next: <a href="H-time-based-view.md">TREE Time-based View</a></p>
