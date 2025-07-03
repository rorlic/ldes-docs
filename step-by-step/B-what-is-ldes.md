# What is LDES?
When publishing data for use by data clients, the actual problem that we are trying to solve is data _sharing_, not data _querying_. So, why would we try to solve that with a _querying_ API? We need a _generic API_ that allows for sharing data sets instead. LDES provides a way to do exactly that: LDES allows a data client to copy the _historical_ and _current_ state of a data set (*replicate*) and to keep up to date with the _changes_ made to it in the future (*synchronize*).

If you look at the name LDES you can see that it actually consists of 2 parts: _Linked Data_ (LD) and _Event Streams_ (ES).

> [!NOTE]
> [Linked Data](https://en.wikipedia.org/wiki/Linked_data) is [structured data](https://www.ibm.com/think/topics/structured-vs-unstructured-data) that can be (inter)linked with other (linked) data.

In order to link related data sets together, all data items must have a unique identifier[^1] and all data properties need to have a well-defined meaning (semantics). Linked data is therefore very suited for things like [semantic queries](https://en.wikipedia.org/wiki/Semantic_query).

[^1]: An identifier is preferably globally unique but if multiple identifiers refer to the same physical or conceptual thing, we at least need a way to relate the identifiers.

> [!NOTE]
> An [(Event) Stream](https://en.wikipedia.org/wiki/Stream_(computing)) is a continuous flow of data produced over time by some system.

In software development, we typically handle such a flow using [stream processing](https://en.wikipedia.org/wiki/Stream_processing). Each data item contains an [event](https://en.wikipedia.org/wiki/Event_(computing)), which is basically a change of the state of a thing. An event can either represent only the portion of this thing that has changed, or the full new state of the thing. The LDES specification requires that the data items contain the _complete_ new state of a thing. We call this a _version_ of an entity.

> [!IMPORTANT]
> In essence, an LDES is _an event stream of (fully-represented) entity versions, expressed as linked data_.

---
<p align="right">Next: <a href="C-linked-data-basics.md">Linked Data Basics</a></p>
