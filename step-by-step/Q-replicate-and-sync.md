# LDES Replication & Synchronization
Let us assume that you have found an LDES about things that interest you. Suppose you want to retrieve the complete data set in order to analyze the history of these things. Maybe you even want to predict future values based on the historical knowledge. If so, you probably want to retrieve the future versions as well in order to compare your predicted values. But how do you do this?

The LDES specification defines the procedure (algorithm) that needs to be followed in order to read an event stream. There are basically three phases: initialization, replication and synchronization. Let us look at these phases in more detail to understand what they are for and how they work.

## Initialization
The whole process starts with a URL, which points to an event stream or one of its views, that we have retrieved in some way. Most likely we have been given this URL or we have discovered it from some data catalog.

The initialization phase is needed to determine what this URL refers to and based on that determine both the event stream IRI as well as the view's root node IRI. We need these IRIs to retrieve and extract the event stream and the root node definitions.

These definitions are needed to known where to start following the underlying TREE structure and how to extract the members from the responses.

The [initialization algorithm](https://treecg.github.io/specification/#init) to determine the event stream and root node IRIs is described in the TREE specification and boils down to:
* accept an initial URL, which may be the LDES or (the root node of) the view
* request the initial URL response, following all redirects[^1]
* query the response for an event stream IRI based on its type (`ldes:EventStream`) and if only one IRI is found then that is the event stream IRI (otherwise the response is not a valid LDES)
* query the response for the available view(s) using the view predicate (`tree:view`) and if only one IRI is found then that is the root node IRI[^2]
* verify that the found root node IRI matches the request IRI that resulted in the response (current page IRI) and if they do not match, you need to restart this algorithm using the found root node IRI

[^1]: Redirects can be used for various reasons, e.g. putting an LDES behind a reverse proxy for caching or security.

[^2]: If multiple views are found then a view must be chosen based on the [Tree Discovery and Context Information](https://treecg.github.io/specification/discovery) document (work in progress).

### Extract the Event Stream
The algorithm above says to query the response. So, how does that work? 

When we use a HTTP request to retrieve an LDES by its URL, we may get redirected to some other URL, but eventually we get back a HTTP response containing a linked data model describing our LDES, including its views. As we have seen earlier in the [LDES Specification Basics](./E-ldes-specs.md#empty-ldes) an event stream is defined as an entity of type `ldes:EventStream`.

Remember that any linked data model is just a collection of triples as we have seen in the [Linked Data Basics](./C-linked-data-basics.md#triples)? Know that the LDES is a an entity of type `ldes:EventStream`, we simply lookup the (unique!) triple with predicate `rdfs:type` and object `ldes:EventStream` which gives us a subject.

We can write the query using variables names starting with `?` to indicate things we do not know and add some keywords to get the following:
```
prefix ldes: <https://w3id.org/ldes#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

select ?e 
where {
    ?e rdfs:type ldes:EventStream .
}
```

Congratulations! You have written your first [SPARQL](https://en.wikipedia.org/wiki/SPARQL) query that gives us the event stream IRI.[^3]

[^3]: There are other ways to lookup things in RDF. If you need to do this from within a software system, you need to use one of the [Software Development Kits](https://en.wikipedia.org/wiki/Software_development_kit) (SDKs) for RDF. These exist for many programming languages such as [Java](https://jena.apache.org/), [.NET](https://dotnetrdf.org/), [JavaScript](https://rdf.js.org/N3.js/), etc.

> [!TIP]
> As we have seen in the [Linked Data Basics](./C-linked-data-basics.md#interlinking-data), you can abbreviate the `rdfs:type` predicate as `a`. Doing so, we can rewrite the above query as:
> ```
> prefix ldes: <https://w3id.org/ldes#>
> 
> select ?e 
> where {
>     ?e an LDES:EventStream .
> }
> ```

> [!TIP]
> If no event stream can be found in the responses based on the `ldes:EventStream` type, you can try to get the event stream IRI based on one of the optional predicates (`?e tree:view ?v`, `?e tree:shape ?s` & `?e tree:member ?m`). E.g.:
> ```
> prefix tree: <https://w3id.org/tree#>
> 
> select ?e 
> where {
>     ?e tree:shape ?s .
> }
> ```
> If still no event stream IRI is found this way then the response is not a invalid LDES.

### Extract the Root Node
We can extract the root node IRI in a similar way. Given that there can only be one event stream in the response, we can lookup the triples matching predicate `tree:view`. The objects from these triples are the root node IRIs from the views. I.e.:
```
prefix tree: <https://w3id.org/tree#>

select ?v 
where {
    ?e tree:view ?v .
}
```
Note that we do not select the event stream IRI (`?e`) as we only need the root node IRI here.

> [!TIP]
> If no view(s) can be found in the responses based on the `tree:view` predicate, you can try to get the root node IRI based on its type (`?v a tree:Node`) or one of the optional predicates (`?v tree:relation ?r`, `?v viewDescription ?d` & `?v treeSearch ?o`).
> ```
> prefix tree: <https://w3id.org/tree#>
> 
> select distinct ?v
> where {
>     ?v tree:relation ?r .
> }
> ```
> Note that we use `distinct` to find the unique node as a node can have multiple relations.
> 
> If still no or multiple root node IRIs are found this way then the response is not a invalid LDES.

## Retrieving the history (Replication)
After you have extracted the definitions of the event stream and the view, you have enough information to process the TREE structure.

The initialization algorithm ensures that you have the content of the root node. You can now extract the members form the root node response as well as the related node(s) from the relation(s). For each node found, you retrieve the node content to extract the additional members and any contained relations, which again gives you more node IRIs. You need to repeat this until you have retrieved all the node IRIs. Now you have successfully _replicated_ the data set.

How exactly should we extract the node IRIs and the members themselves? 

### Extract the Related Nodes
To get the links to the related nodes, we need to lookup the subject that is a `tree:Node`, get the (anonymous) objects of type `tree:Relation` (or a sub-class thereof) using the node's `tree:relation` predicate, and finally take the relation's `tree:node` predicate to get the URL in the object position.

Using SPARQL we can almost literary express this as follows:
```
prefix ldes: <https://w3id.org/ldes#>
prefix tree: <https://w3id.org/tree#>

select ?n 
where {
    ?v a tree:Node .
    ?v tree:relation ?r .
    ?r tree:node ?n .
}
```
We do not really care about the relation type here as we need to get all the nodes and therefore follow all the links we find.

> [!NOTE]
> The where clause of a SPARQL query is basically a chain of conditions. In this case, we first look for all `?v` of type `tree:Node` (there can be only one) and use that to find all relations `?r` of that `?v` and use those `?r` to find all the `?n` (there can be only one per relation). If no `?r` exist, then the last condition return no results.

### Extract the Members
We already know how to retrieve all the nodes containing members, but we still need to extract the triples per members from the triples collection. First, you need to know which members are contained in the current node. You can use the following query for that:
```
prefix ldes: <https://w3id.org/ldes#>

select ?m 
where {
    ?e tree:member ?m .
}
```

As you see, getting the URIs of the members is easy. However, collecting all the triples that belong to each member is non trivial. The TREE specification describes the [member extraction algorithm](https://treecg.github.io/specification/#member-extraction-algorithm), which in essence works this way:
* we take all the quads (or triples) with predicate `tree:member` as the object from that triple gives us the focus node URI of a member, that is, the member identifier
* for each member identifier we collect all the quads that are part of the member as follows:
    * take all the quads with the member identifier in the subject position and recursively collect all the quads for each blank node found in the object position (essentially implementing the [CBD](https://www.w3.org/submissions/CBD/) mechanism)
    * add all the quads within the named graph with the member identifier as its name
    * if the above steps did not yield any quads, dereference the member identifier (request it using the HTTP protocol) and take the quads from the response

For now, the algorithm also uses the `tree:shape` to limit or to include quads to the member, but this behavior will be obsoleted in the future.

## Retrieving the future (Synchronization)
Usually, an API that allows you to get notified of changes will provide some way of subscribing for these notifications. This requires management of these notifications on the server side, possibly including retries if the client side cannot be notified. On the client side we need to unsubscribe in order to stop receiving notifications. In addition, the server and client side need to have a common understanding what the notification means and how to interpret what (part of the) data has changed. As you can imagine, this is pretty complex all things considered.

Again, LDES takes a different approach: there are no subscriptions and no notifications, but instead the server side can indicate to the client side which node is "full" and which node may contain additional members in the future. Please remember that an LDES is a collection of immutable members. In other words, the members themselves will never change as they are versions of entities. That is, if an entity changes, a new version object is added to the LDES as a member.

We have seen that an LDES is retrieved using nodes, which contain a number of members. So, how many members does a node contain? Well, that depends on the LDES server implementation, but typically there will be a configured maximum member count. When a node contains that many members, it is marked "full" and linked to a new node that is created to contain additional members. In other words, once the node contains the configured maximum number of members, that node will not change anymore, that is, no more members will be added to it.

Can members be removed? Yes, they can! Due to retention, some members in a node may be removed. Is that not a problem? No, not at all! Any data client that has already processed this node, will not process it again as it will never receive any additional member. Any data client that processes the node later, will not see the removed members, which is indeed what you would expect.

When the node reaches its maximum capacity, it is marked _immutable_. This indicates to the data client that this node should never be re-requested again. But the opposite is also true: if a node is not marked _immutable_, it means that members may be added in the future and that the data client should periodically re-request the nodes in order to receive new members, that is, _synchronize_ with the LDES changes.

How does the server indicate that a node is immutable? For this, currently LDES relies on the [cache-control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) HTTP header. This header allows to specify the `immutable` directive, which indicated that the (HTTP) response will not be updated while it is "fresh". The "freshness" is determined by another directive: the `max-age`, which is a value (in seconds) specifying how long the response is valid after the response creation. It is typically used for cache servers that sit between a HTTP client and a HTTP server, to communicate how long to respond with the same response (caching) and when to re-request the response from the server.

To indicate that a node may receive members in the future, an LDES server returns the `cache-control` header with a `max-age` set to a _polling_ interval, which is the (minimum) time interval for re-requesting a node. A data client may wait longer if desired, but re-requesting it sooner will probably return the same result. If fact, to check if the HTTP response changed, the data client will first check the [etag](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag) header value. If the value is the same as in the previous HTTP response, then the content did not change and there is no need to re-extract anything. In fact, if an LDES server is behind a reverse proxy with caching capabilities, instead of requesting the content of a nodes (HTTP GET command) you can ask for only the response HTTP headers (HEAD command), which is a lot faster, and then request the node content only if the `etag header` has changed.

Example header returned for a _mutable_ node:
> `Cache-Control: max-age=60`

This node can be re-requested every minute to receive new members.

To indicate that a node will not receive any more members, an LDES server returns the `cache-control` header with the _immutable_ directive included and the (mandatory) max-age set to a large value (typically a week or a year). A data client should ignore the max-age value in this case and never re-request the node again.

Example header returned for a _immutable_ node:
> `Cache-Control: immutable, max-age=604800`

This node should not be re-requested (as it will never contain new members).

When a mutable node is re-requested and it contains new members, a data client should only consider those new members for further processing and ignore the previously received ones. Clearly, the data client needs to remember which members have already been encountered and only process the unprocessed ones. This brings up an interesting point: any number of nodes can contain the exact same member. Which is why a data client must keep a list of _all_ the members already encountered.

> [!TIP]
> It is possible to retrieve a one-dimensional forward-linked list of nodes without keeping a list of visited members and nodes, because each node URL and each member can only appear once in this view. However, retrieving such a view will be slower as you can not benefit from retrieving the nodes in a parallel way.

## LDES Client
Now, what happens to the extracted members? Some data clients may want the receive the history of things, while other data clients may simply want the (latest) state of things. Some data clients may even want to alter the data in another model more appropriate for their use case. Some may want to feed it into a custom system while other may simply want to store the data in a database.

No matter which use case a data client has, their data pipeline starts with replicating and (optionally) synchronizing an LDES, extract the members and, if no history is needed, unwrap a version object back to its entity, also called _materialization_. Because the starting point of an LDES pipeline is always the same, it makes sense to have a reuseable _LDES client_ which implements this functionality.

As shown above, an LDES client must implement the replication logic keeping a list of visited nodes. In addition, it must keep a list of processed members and handle polling mutable nodes to receive new members. Finally, it must extract members from a node, optionally materialize members, and pass them further downstream in the data pipeline.

Ideally, an LDES client should come in the form of a SDK, which allows embedding in existing data pipeline systems, or alternatively, as a standalone executable pushing members using HTTP or another standard protocol towards downstream systems.


These data pipeline systems can then store the data members in any database type, be it a traditional one, a document or perhaps a graph database. Could we store the data members again as an LDES? We sure could! But why would we want that? Well, there may be different reasons to do this. Maybe the data publisher only offers a one-dimensional, forward-linked view and you want to offer your data clients alternative [views](#partitioning-types) which allow them to retrieve the data set much faster or retrieve only a subset of the data set. Maybe the data publisher has a short [retention policy](#retention-policies) on its view(s) and you want to offer a longer history to your data clients. Or maybe you want to combine different event streams into a new LDES offering a richer data model.

In all these cases, you should host your own LDES server and create a data pipeline which starts with an LDES client to replicate and synchronize the original LDES, transform the members in the way you see fit and push the members to your LDES server, which is setup to (re-)partition the LDES or offer the LDES with different models or retention policies. If your LDES server implementation cannot ingest version objects, you need to ensure the data pipeline does materialization and the LDES server creates the versions automatically. In fact, in this scenario you are a _data broker_, which is an individual or department who offers combined or otherwise altered data.

> [!TIP] 
> In the current TREE specification it is not allowed anymore to have more than one incoming link for a node. This means that you do not need to keep a list of visited nodes as each node can be reached only once. However, for backwards compatibility and to detect (what are now) faulty TREE structures (such as circular links), it is recommended to keep track of the URLs that you already visited and only request the new ones.

## Summary
> [!IMPORTANT]
> A _LDES client_ is used to _replicate and synchronize a data set_, which allows you to easily retrieve the history and the latest state of the entities in the data set.
