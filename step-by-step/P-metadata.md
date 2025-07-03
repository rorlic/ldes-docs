# LDES Availability & Discovery
So, how do you known which data sets are available as an LDES? How do you announce availability of your LDES data sets?

In the linked data world, we can use the [Data Catalog Vocabulary (version 3)](https://www.w3.org/TR/vocab-dcat-3/) (DCAT v3) to describe the metadata of a data set. This allows a data client to look up information about the data set such as the data offered in the data set, conditions for using it, where to find the data set, etc.

The DCAT specification allows to build registries of data sets by aggregating metadata from different data owners or publishers. This increases the discoverability of data sets. The specification even allows for decentralized data catalogs that make federated searches across these registries.

If you know a metadata service URL (e.g.[Metadata Vlaanderen](https://metadata.vlaanderen.be/srv/eng/catalog.search#/home) or [European Data](https://data.europa.eu/en)), you can lookup the available data sets in the service's catalog.

The main [DCAT concepts](https://www.w3.org/TR/vocab-dcat-3/#dcat-scope) are _catalog_ (`dcat:Catalog`), _data set_ (`dcat:Dataset`), _distribution_ (`dcat:Distribution`) and _data service_ (`dcat:DataService`), where `dcat:` maps to namespace `http://www.w3.org/ns/dcat#`.

Simplified, in the DCAT context, a catalog is a collection of metadata about the available data sets. A data set is a collection of data items available using distributions. A distribution is an accessible form of the data set and can have a data (access) service. A data service is the interface (API) to access the data set. In fact, both a data set and a data service are a resource (`dcat:Resource`) and inherit many properties in addition to a few of their own. A catalog is basically a data set of metadata records, and as such also a resource.

The TREE specifications does not refer to DCAT while the LDES specification briefly mentions [DCAT v2](https://www.w3.org/TR/vocab-dcat-2/) compatibility. Basically, this is still work in progress in the [TREE Discovery and Context Information](https://treecg.github.io/specification/discovery).

So, how do we find (or expose) an LDES using DCAT for now? To find an LDES in a DCAT catalog, you need to look for data sets or data services which have a `dct:conformsTo` set to `ldes:specification` (see [here](https://github.com/SEMICeu/LinkedDataEventStreams/issues/58)). Once you have found the data sets that conform to the LDES specification, you need to get a hold of one of the LDES views.

Technically, LDES does not define an API, therefore a data publisher might use the `dcat:accessURL` predicate on `dcat:Distribution` to refer to the LDES itself, from where the views can be discovered, which all provide the same data set but organized differently. The idea is similar to [this](https://www.w3.org/TR/vocab-dcat-3/#example-landing-page), where the views are "behind" the LDES "landing" page. But an LDES is not a landing page and its views are directly accessible. Therefore, a data publisher might relate a `dcat:Distribution` per LDES view and set its `dcat:downloadURL` to the LDES view (root) node, which boils down to [this](https://www.w3.org/TR/vocab-dcat-3/#a-dataset-available-as-download-and-behind-some-web-page) scenario. You could argue that the root node only returns a portion of the data set, so again this scenario is a partial match. Probably most data publishers will therefore create a `dcat:Distribution` per LDES view and set the `dcat:accessURL` to the LDES view URL directly and set the `dcat:accessService` predicate to a `dcat:DataService`, which has its `dcat:endpointURL` also set to the LDES view URL.

This is the _current recommendation_ until the TREE and LDES specifications contain final DCAT requirements.

There are many DCAT derived specifications that are applicable for local environments. These may require a slightly different approach, or expect some additional mandatory predicates, or limit some values, etc. The problem with some standards is that there are too many (variations) of them.

Example LDES metadata (without the LDES details and with minimal DCAT):
```
@prefix tree:      <https://w3id.org/tree#> .
@prefix ldes:      <https://w3id.org/ldes#> .
@prefix sh:        <http://www.w3.org/ns/shacl#> .
@prefix schema:    <http://schema.org/> .
@prefix dct:       <http://purl.org/dc/terms/> .
@prefix xsd:       <http://www.w3.org/2001/XMLSchema#> .
@prefix ex:        <http://example.org/> .
@prefix disney:    <http://example.org/DisneyFeed/> .
@prefix dcat:      <http://www.w3.org/ns/dcat#> .

ex:catalog a dcat:Catalog
  dcat:dataset ex:DisneyFeed .

ex:DisneyFeed a ldes:EventStream, dcat:Dataset ;
  dcat:conformsTo ldes:specification ;
  dcat:distribution disney:default-distribution ;
  tree:view disney:default-view .

disney:default-distribution a dcat:Distribution ;
  dcat:accessURL disney:default-view ;
  dcat:accessService disney:default-view .

disney:default-view a tree:Node, dcat:DataService ;
  dcat:endpointURL disney:default-view .
```
This may need a bit of explanation. Basically, we have defined a catalog named `ex:catalog` that has a data set `ex:DisneyFeed`, which is doubled typed to an LDES and a data set. The event stream conforms to the LDES specification and has a distribution named `disney:default-distribution` and a view named `disney:default-view`, which is both an identifier (URI) and a locator (URL). The distribution has both its direct access URL set to the view URL and refers to the view as its access service, which is the 'API' allowing access to the data set. Finally, our view is both a (starting) node as well as a data service, which endpoint is, of course, the view URL.

> [!IMPORTANT]
> Both the _TREE and LDES specifications are a living standard_ and do not (yet) define exactly how the LDES metadata should look like. However, they _are_ both _compatible with DCAT_.
> 
> For the time being, the recommended way to expose metadata for an LDES is to _map the LDES to a dataset and a view to a data service_ and relate them using a distribution, and finally, put all of these DCAT resources in a catalog. After that, you provide the endpoint containing the catalog to a metadata service so that a data client can discover the available LDES'es.

---
<p align="right">Next: <a href="Q-replicate-and-sync.md">LDES Replication & Synchronization</a></p>
