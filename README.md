# Elasticsearch - The Definitive Guide
 
## Table of contents

1. [Chapter 1: You Know, for Search](#Chapter1)
2. [Chapter 2: Life Inside a Cluster](#Chapter2)
3. [Chapter 3: Data In, Data Out](#Chapter3)
4. [Chapter 4: Distributed Document Store](#Chapter4)
 

## Chapter 1: You Know, for Search<a name="Chapter1"></a>
 
Elasticsearch is build on top of lucene, and comes with an Apache 2 license.
 
### Installing Elasticsearch
Elasticsearch can be installed directly with a curl command or with marvel, which is a
management tool that comes with an interactive console called sense, to talk to elasticsearch
from your browser, which is available as a plugin. Install marvel with:
`./bin/plugin -i elasticsearch/marvel/latest`
 
### Running Elasticsearch
Run Elasticsearch with:
 
`./bin/elasticsearch <-d>`
 
Test that it is working with:
 
`curl 'http://localhost:9200/?pretty'`
 
Which would display a JSON message in your browser with some metadata of your ES cluster. Change
the name of your server to stop your nodes from trying to join another cluster on the same
network with the same name (edit the _elasticsearch.yml_ config file under the configfolder).
 
### Talking to Elasticsearch
#### Java API
Two type of clients are provided:
 
    * Node Client: To join the cluster as a non datanode. It does not hold data but knows what 
     data lives on which node in the cluster
    * Transport Client: lighter than the node client, sends messages to a remote cluster (doesn't
     join the cluster)
     
Both clients talks to the cluster on port 9300 using ES's transport protocol.
 
#### RESTful API with JSON over HTTP
ES accepts http connections in the 9200 port. The requests should have the following structure:
 
`curl -X<VERB> '<PROTOCOL>://<HOST>/<PATH>?<QUERY_STRING>' -d '<BODY>'`
 
where:
 
    * VERB: The http method (GET, POST, PUT, HEAD, DELETE)
    * PROTOCOL: either http or https
    * HOST: the host name
    * PORT: by default 9200
    * QUERY_STRING: Any optional query string parameters
    * BODY: A JSON encoded request body
    
On successful execution, ES would return an error code and a JSON message with the response from
the server.
 
### Document Oriented
Elasticsearch is document oriented, meaning that it stores entire objects or documents.
It not only stores them, but also indexes the contents of each document in order to make them
searchable.
 
### Finding your feet
Here is a simple tutorial about how to index, search and perform aggregations in ES.
 
#### Indexing documents
The act of storing data in ES is called indexing. Each document belongs to a _type_, and each
_type_ lives in an _index_. In a way, the equivalent to a relational database would be:
 
<TABLE>
    <TR><TD>Relational DB</TD><TD>Databases</TD><TD>Tables</TD><TD>Rows</TD><TD>Columns</TD></TR>
    <TR><TD>Elasticsearch</TD><TD>Indices</TD><TD>Types</TD><TD>Documents</TD><TD>Fields</TD></TR>
</TABLE>
 
To index a document, we need to do a PUT request to _<INDEX\_NAME>/<TYPE\_NAME>/<ID>_ passing as
the body of the request the JSON document to index.
 
#### Retrieving documents
Similar to the previous one, we need to do a GET request to _<INDEX\_NAME>/<TYPE\_NAME>/<ID>_

#### Search Lite
To search for all the documents in an index, we do a GET to _<INDEX\_NAME>/<TYPE\_NAME>/\_search_,
which would return the first 10 results. To search for all the documents with an specific field
 value, we do a _<INDEX\_NAME>/<TYPE\_NAME>/\_search?q=fieldName:value_.

#### Search with query DSL
ES provides a rich query language called _query DSL_. The query itself is defined on the body of
the message like _<INDEX\_NAME>/<TYPE\_NAME>/\_search {"query":{"match":{"fieldName":"value"}}}_.
 
#### More-Complicated Searches
Queries can be more complicated and include things like filters and aggregations inside the body
of the message.
 
#### Full-Text Searc
By default, Elasticsearch sorts matching results by their relevance score, that is, by how well
 each document matches the query. A results that have more matches over all the search terms will
rank higher than other that has only partial matches.
 
#### Phrase Search
If instead of matching on individual terms you want to match on complete phrases, use the following:
_{"query":{"match_phrase":{"field":"some phrase"}}}_.

#### Highlighting Our Searches
You can highlight the search result to see why the document retrieved matched a certain query:
_{"query":{"match_phrase":{"field":"some phrase"}}, "highlight":{"fields":{"fieldName":{}}}}_.
The result of this query will contain the snippet of text from the selected fields with the
matching words wrapped in _\<em\>\</em\>_ HTML tags.
 
#### Analytics
Elasticsearch has functionality called aggregations to help analyzing the data, which allow you 
to generate sophisticated analytics over your data (similar to the SQL's GROUP BY):
_{"query":{"match":{"field":"value"}}, "aggs":{"aggs_name":{"terms":{"fieldName:":"value"}}}}_.
Aggregations can be nested to allow for hierarchical groupings.
 
### Distributed Nature
ES is distributed by nature, it hides the complexity of certain distributed-related tasks such as:
 
    * Partitioning and sharding of the indexes
    * Shard balancing accross the nodes of the cluster
    * Provide redundancy
    * Routing requests to the appropriate shard
    * Adding or removing nodes to the cluster
 
 
## Chapter 2: Life Inside a Cluster<a name="Chapter2"></a>
### An empty cluster
A _node_ is a running instance of ES, while a _cluster_ consists of one or more nodes with the same
_cluster.name_ that are working together to share their data and workload. One node in the
cluster is elected to be the master node, which manages cluster-wide changes like creating or
deleting an index, or adding or removing a node from the cluster (not involved in document
changes or searches). Any node can become the master. Users can talk to any node in the cluster and
every node knows where each document lives and can forward our request directly to the nodes that
hold the data we are interested in.

### Cluster health
Cluster health is a statistic report tha we get by calling _GET /\_cluster/health_, the response
can be either:
 
    * Green: All primary and replica shards are active
    * Yellow: All primary shards are active, but not all replica shards are active
    * Red: Not all primary shards are active
    
### Add an Index
An index in ES is a logical namespace that points to one or more physical shards. A Shard is a
low-level worker unit that holds just a slice of all the data in the index, a single instance of
Lucene (a complete search engine in its own). Documents are stored in shards, which are allocated
in nodes in the cluster. A shard can be primary or a replica, the number of primary shards in an
index is fixed at the time that an index is created, but the number of replica shards can be
 changed at any time. To create an index, simply do _PUT /<INDEX\_NAME>_.
 
### Add Failover
To provide redundancy, new nodes might be added to the cluster. To do so, the new node needs to
have the same _cluster.name_ in its configuration and belong to the same network. When a new node
is added to the cluster, replica shard might be balanced to it.

### Scale Horizontally
The number of replica shards can be changed dynamically on a live cluster, with:
_PUT /<INDEX\_NAME>/\_settings -d '{"number_of_replicas" : <NEW\_NUMBER\_OF\_REPLICAS}'_
 
### Coping with Failure
A cluster must have a master node in order to function correctly: if the primary node fails, a
replica of the primary shards it held would be promoted to be primaries in other nodes, and other
node in the cluster would be promoted to be master.
 
## Chapter 3: Data In, Data Out<a name="Chapter3"></a>
 
ES is a distributed document store, once a document has been indexed, it can be retrieved from
any node in the cluster. All data in every field is indexed by default.
 
### What is a Document?
A Json document is composed of keys and values, with the key as the name of the field and the
value (which can be any of string, number, boolean, object, array, geopoint, date...).  In ES, 
 document is the top-level root object in the jsonmessage.
 
### Document Metadata
Metadata is the data describing the data. Required metadata in ES:
 
    * _index: Like a database in a RDBMS, is the place to store and index the data. In ES is a
    namespace that groups together one or more shards
    * _type: Logical group of similar documents that shares a mapping or schema definition
    * _id: Combined with the _type and _index, uniquely identifies a document
 
### Indexing a document:
Indexing a document is the process of storing it and make it searchable. In ES we need the three
metadata components listed before, we can provide our own _id or let ES generate one for us:
 
#### Using our own ID
To provide a document _id, use _PUT /{index}/{type}/{id} {JSON document}_. Every document in ES
has a document version number (the _version metadata field).
 
#### Autogenerating IDs
To provide let ES generate a document _id, use _PUT /{index}/{type}/ {JSON document}_.Autogenerated
IDs are 22 character long, URL-safe, Base64-encoded strings (a UUID).

### Retrieving a Document 
To retrieve a document, use _GET /{index}/{type}/{id}_. The indexed document will be returned
inside the _source field. The field 'found' is a boolean indicating if the document had been 
found or not (the http code in this case would be 404).
 
#### Retrieving part of a document
By default the whole document is returned inside the _source field, but you can also retriev
specific fields by using _GET /{index}/{type}/{id}?\_source={field1},{field2},...,{fieldN}_. The
matching document would only contain the specified fields in the _source field.
You can also retrieve the whole _source field without any metadata by queryin
_GET /{index}/{type}/{id}/\_source_.
 
### Checking whether a document exists
To check if a document exists or not, use _HEAD /{index}/{type}/{id}_, which would return an http
code 200 if the document exists or a 404 if it doesn't.
### Updating a Whole Document
Indexed documents are immutable, to update them, we need to reindex or replace the current document.
The command is the same than to put a new one, but the document version will be incremented. In
reality, ES deletes the document (marks it for deletion and deletes it sometime in the future
with a background process).
 
### Creating a new Document
The easiest way to guarantee that we are creating a new document and not replacing an existing
one is to let ES to create the id for us. If we must use an specific ID, then you can use
 _PUT /{index}/{type}/{id}?op_type=create_ or _PUT /{index}/{type}/{id}/\_create_. This will
 return either a 200 or a 409 (conflict) http code.

### Deleting a document
Use _DELETE /{index}/{type}/{id}_ to delete the {id} document. Again this returns a 200 or 404
http code.
 
### Dealing with Conflicts
When updating a document with the index API, we read the original document, make our changes, and
then reindex the whole document in one go. The most recent update wins, which can lead to race
 conditions. In the database world, we have two techniques to solve this issue:

    * Pessimistic concurrency control: Block access to a resource while doing modifications to it
     in order to prevent conflicts
    * Optimistic concurrency control: Used by ES is anon blocking technique that would fail a 
    write if a modification has been done between the reading and the writting attempt
 
### Optimistic Concurrency control
As ES is distributed, any modification to a document has to be replicated. ES is also asynchronous
and concurrent, so the order of the sequence of modifications needs to be preserved. This is done
through the _version number. If an older version of the document arrives after a new version, ES
ignores it. We can indicate the version a document is by passing it in the PUT request as the
 'version' parameter.
All APIs that update or delete a document accept a version parameter, which allows you to apply
 optimistic concurrency control to just the parts of your code where it makes sense.
 
#### Using Versions from an External System
If we use ES to make the contents of a RDBMS searchable, you might want to use the timestamp
value of a DB row as the version number (needs to be an integer in the java long range). To do
so, you need to pass the parameters 'version_type=external' and 'version=\<version_number\>'.
 
### Partial Updates to Documents
Partial updates can be done through the update API (again the document is deleted and re-indexed).
This process happens within a shard, avoiding the network overhead of multiple requests that the
retrieve-update-reindex approach has. To use it do _POST {index}/{type}/{id}/\_update {data}_,
with data in a form like `{"doc": { "field1":"...", "field2": "..."}}`. New fields will be added
an existing ones replaced.

#### Using scripts to make partial updates
Scripts can be used in the update API to change the contents of the _source field, which is referred
to inside an update script as _ctx.\_source_, for example
_POST /website/blog/1/\_update -d '{"script" : "ctx.\_source.views+=1"}'_
ES allows you to use groovy scripts, i.e.:
 
```
POST /website/blog/1/_update
{
    "script" : "ctx._source.tags+=new_tag",
    "params" : {
        "new_tag" : "search"
    }
}
```
 
#### Updating a document that may not yet exist
Upsert allows to specify updates on fields that might not yet exist, the upsert parameter specifies 
the document that should be created if it doesn't already exist.
 
#### Updates and conflicts
The update operation uses optimistic concurrency control, checking the version numbers. In
partial updates we might not care if the document has been updated in between request or not (for
example a counter increment update), in such cases a reattempt after a failed update can be
 desirable. This can be automated by passing the 'retry_on_conflict=\<number\>' parameter to the
\_update API.
 
### Retrieving Multiple Documents
As fast as Elasticsearch is, it can be faster still. Combining multiple requests into one
avoids the network overhead. This can be achieved with bulk requests, which is available in the
\_mget API:
 
```
GET /_mget
{
   "docs" : [
       {
            "_index" : "website",
            "_type" : "blog",
            "_id" : 2
        },
        {
            "_index" : "website",
            "_type" : "pageviews",
            "_id" : 1,
            "_source": "views"
        },
        ...
   ]
}
```
 
The response body from the previous request would contain an array with the responses for each
one of the request in order. both \_index and \_type can be specified in the url. The requests 
would reply with a 200 code even if documents are not found (this is specified in the found flag).
 
### Cheaper in Bulk
Similar to the \_mget, bulk requests allows for creation, indexation or deletion of documents in
a single request. The bulk request body has the following, slightly unusual, format:
 
```text
    { action: { metadata }}\n
    { request body }\n
    { action: { metadata }}\n
    { request body }\n
```
 
Each line has to end in a newline character and can't contain unescaped new line characters on it.
The action/metadata line specifies what action to do to which document, which must be one of:
"create|index|update|delete". The metadata should specify the _index, _type, and _id of the document
to be indexed, created, updated, or deleted. The request body is the content of the _source field
(delete actions does not require a body). If any of the requests fail, the top-level error flag is
set to true and the error details will be reported under the relevant request. Bulk requests
*are not atomic*.
 
#### Don’t Repeat Yourself
As with \_mget, with the bulk API you can specify the {index} and {type}, in which case it is not
required to include it in the action metadata.
 
#### How Big Is Too Big?
The entire bulk request needs to be loaded into memory by the node that receives our request, so
the bigger the request, the less memory available for other requests. To find the optimal size
for a batch (which is dependent on the hardware, document size and complexity), you can test with
increasing batch sizes until performance degradation is observed: thats the sweet spot.
 
 
## Chapter 4: Distributed Document Store<a name="Chapter4"></a>
 
### Routing a Document to a Shard
The shard where a document is assigned to be stored is calculated like:
_shard = hash(routing) % number\_of\_primary\_shards_
Custom routing values could be used to ensure that all related documents—for instance, all the
documents belonging to the same user—are stored on the same shard.
 
### How Primary and Replica Shards Interact
We can send our requests to any node in the cluster. Every node is fully capable of serving any
request. When sending requests, it is good practice to round-robin through all the nodes in the
cluster, in order to spread the load.
 
### Creating, Indexing, and Deleting a Document
Create, index, and delete requests are write operations, which must be successfully completed on
the primary shard before they can be copied to any associated replica shards. By the time the
client receives a successful response to a request, the document change has been executed on the
primary shard and on all replica shards. Some parameters can be changed to make this process faster:
 
    * replication: By defailt, the primary shard waits fot successful responses from replica
    shards (sync). If this parameter is set to async, it will return sucess to the client as soon
    as the request has been executed on the primary shard (not advised)
    * consistency: Defaults to quorum or majority of shard copies to be available before attempting
    a write operation. Defined as int( (primary + number_of_replicas) / 2 ) + 1
    * timeout: if insufficient shard copies are available, ES waits, in the hope that more shards
    will appear (defaults to 1 minute)
    
### Retrieving a Document
The receiving node of a request uses the \_id to determine to which shard the document belongs to.
Then, it uses this identifier to retrieve the document from the primary or any replica where the
document is. For read requests, the requesting node will choose a different shard copy on every
request in order to balance the load; it round-robins through all shard copies.
 
### Partial Updates to a Document
To update a document, the client send a request to a node, that redirects it to the node where
the primary shard is. That node then retrieves the document and tries to modify it, if the
document has been changed by another process then it tries agan _retry\_on\_conflict_ times. On 
successful update the node fordwards the new version to the replica shards, once the replicas has
report success, then the client is reported success too.

### Multidocument Patterns
The patterns for the _mget_ and _bulk_ APIs are similar to those for individual documents. The
difference is that the requesting node knows in which shard each document lives. It breaks up the
multidocument request into a multidocumentrequest per shard, and forwards these in parallel to
 each participating node. Then it collates each individual response into a single one before
 sending it to the client.
 
 
## Chapter 5: Searching - The Basic Tools<a name="Chapter5"></a>
Every field in a document is indexed and can be queried, a search can be any of the following:

    * A structured query on concrete fields, sorted by a field and with other options like filters
    * A full-text query, which finds all documents matching the search keywords, and returns them
    sorted by relevance
    * A combination of the two

### The empty search
Returns all the documents in an index _GET /\_search_.

#### hits
The hits section in the response specifies the number of documents that matches the query in the
'total' field, and the first 10 elements in the 'hits' array. each document match has a '_score'
field which is a measure of how well the document matches the query (results are ordered based on
this field). The 'max_score' is the highest of the '_score' fields.
#### took
This value is the total time in milliseconds that the request took.

#### shards
The _shards element tells us the total number of shards that were involved in the query.

#### timeout
The timed_out value tells us whether the query timed out. It can be specified in the request like
_GET /\_search?timeout=10ms_. This parameter does not halt the execution of the query, it tells
the coordinating node to return the results gathered in that time.

### Multi-index, Multitype
The preceding empty search was not limited to a particular index type, returning documents of
different indexes and shards. This can be limited by providing this information in the url. For
example:

    * /_search: Search all types in all indices
    * /gb/_search: Search all types in the gb index
    * /gb,us/_search: Search all types in the gb and us indices
    * /g*,u*/_search: Search all types in any indices beginning with g or beginning with u
    * /gb/user/_search: Search type user in the gbindex
    * /gb,us/user,tweet/_search: Search types user and tweet in the gb and us indices
    * /_all/user,tweet/_search: Search types user and tweet in all indices

When you search within a single index, Elasticsearch forwards the search request to a primary or
replica of every shard in that index, and then gathers the results from each shard.

### Pagination
Elasticsearch accepts the 'from' and 'size' parameters in the request to perform pagination:

    * size: Indicates the number of results that should be returned, defaults to 10
    * from: Indicates the number of initial results that should be skipped, defaults to 0

### Search Lite
There are two forms of the search API: a “lite” query-string version that expects all its parameters
to be passed in the query string, and the full request body version that expects a JSON request
body and uses a rich search language called the query DSL.
You can define lite queries to match a value in a field with the following format:
_+field1:value1 +field2:value2_ and passing it with the 'q' parameter (although they need to be
URL encoded before). You can also specify conditions that must NOT match with the '-' symbol
instead of the '+'.

#### The _all field
When you index a document, Elasticsearch takes the string values of all of its fields and
concatenates them into one big string, which it indexes as the special _all field. The query-string
search (the one passed with the 'q' parameter) uses the _all field unless another field name has
been specified (this behaviour can be disabled).

#### More Complicated Queries
The query-string search allows any user to run potentially slow, heavy queries on any field in
your index, possibly exposing private information or even bringing your cluster to a halt.