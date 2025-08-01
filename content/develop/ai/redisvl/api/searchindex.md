---
linkTitle: Search index classes
title: Search Index Classes
aliases:
- /integrate/redisvl/api/searchindex
---


| Class                                     | Description                                                                                  |
|-------------------------------------------|----------------------------------------------------------------------------------------------|
| [SearchIndex](#searchindex-api)           | Primary class to write, read, and search across data structures in Redis.                    |
| [AsyncSearchIndex](#asyncsearchindex-api) | Async version of the SearchIndex to write, read, and search across data structures in Redis. |

<a id="searchindex-api"></a>

## SearchIndex

### `class SearchIndex(schema, redis_client=None, redis_url=None, connection_kwargs=None, validate_on_load=False, **kwargs)`

A search index class for interacting with Redis as a vector database.

The SearchIndex is instantiated with a reference to a Redis database and an
IndexSchema (YAML path or dictionary object) that describes the various
settings and field configurations.

```python
from redisvl.index import SearchIndex

# initialize the index object with schema from file
index = SearchIndex.from_yaml(
    "schemas/schema.yaml",
    redis_url="redis://localhost:6379",
    validate_on_load=True
)

# create the index
index.create(overwrite=True, drop=False)

# data is an iterable of dictionaries
index.load(data)

# delete index and data
index.delete(drop=True)
```

Initialize the RedisVL search index with a schema, Redis client
(or URL string with other connection args), connection_args, and other
kwargs.

* **Parameters:**
  * **schema** ([*IndexSchema*]({{< relref "schema/#indexschema" >}})) – Index schema object.
  * **redis_client** (*Optional* *[* *Redis* *]*) – An
    instantiated redis client.
  * **redis_url** (*Optional* *[* *str* *]*) – The URL of the Redis server to
    connect to.
  * **connection_kwargs** (*Dict* *[* *str* *,* *Any* *]* *,* *optional*) – Redis client connection
    args.
  * **validate_on_load** (*bool* *,* *optional*) – Whether to validate data against schema
    when loading. Defaults to False.

#### `aggregate(*args, **kwargs)`

Perform an aggregation operation against the index.

Wrapper around the aggregation API that adds the index name
to the query and passes along the rest of the arguments
to the redis-py ft().aggregate() method.

* **Returns:**
  Raw Redis aggregation results.
* **Return type:**
  Result

#### `batch_query(queries, batch_size=10)`

Execute a batch of queries and process results.

* **Parameters:**
  * **queries** (*Sequence* *[* *BaseQuery* *]*)
  * **batch_size** (*int*)
* **Return type:**
  *List*[*List*[*Dict*[str, *Any*]]]

#### `batch_search(queries, batch_size=10)`

Perform a search against the index for multiple queries.

This method takes a list of queries and optionally query params and
returns a list of Result objects for each query. Results are
returned in the same order as the queries.

NOTE: Cluster users may need to incorporate hash tags into their query
to avoid cross-slot operations.

* **Parameters:**
  * **queries** (*List* *[* *SearchParams* *]*) – The queries to search for.
  * **batch_size** (*int* *,* *optional*) – The number of queries to search for at a time.
    Defaults to 10.
* **Returns:**
  The search results for each query.
* **Return type:**
  List[Result]

#### `clear()`

Clear all keys in Redis associated with the index, leaving the index
available and in-place for future insertions or updates.

NOTE: This method requires custom behavior for Redis Cluster because
here, we can’t easily give control of the keys we’re clearing to the
user so they can separate them based on hash tag.

* **Returns:**
  Count of records deleted from Redis.
* **Return type:**
  int

#### `connect(redis_url=None, **kwargs)`

Connect to a Redis instance using the provided redis_url, falling
back to the REDIS_URL environment variable (if available).

Note: Additional keyword arguments (\*\*kwargs) can be used to provide
extra options specific to the Redis connection.

* **Parameters:**
  **redis_url** (*Optional* *[* *str* *]* *,* *optional*) – The URL of the Redis server to
  connect to.
* **Raises:**
  * **redis.exceptions.ConnectionError** – If the connection to the Redis
        server fails.
  * **ValueError** – If the Redis URL is not provided nor accessible
        through the REDIS_URL environment variable.
  * **ModuleNotFoundError** – If required Redis modules are not installed.

#### `create(overwrite=False, drop=False)`

Create an index in Redis with the current schema and properties.

* **Parameters:**
  * **overwrite** (*bool* *,* *optional*) – Whether to overwrite the index if it
    already exists. Defaults to False.
  * **drop** (*bool* *,* *optional*) – Whether to drop all keys associated with the
    index in the case of overwriting. Defaults to False.
* **Raises:**
  * **RuntimeError** – If the index already exists and ‘overwrite’ is False.
  * **ValueError** – If no fields are defined for the index.
* **Return type:**
  None

```python
# create an index in Redis; only if one does not exist with given name
index.create()

# overwrite an index in Redis without dropping associated data
index.create(overwrite=True)

# overwrite an index in Redis; drop associated data (clean slate)
index.create(overwrite=True, drop=True)
```

#### `delete(drop=True)`

Delete the search index while optionally dropping all keys associated
with the index.

* **Parameters:**
  **drop** (*bool* *,* *optional*) – Delete the key / documents pairs in the
  index. Defaults to True.
* **Raises:**
  **redis.exceptions.ResponseError** – If the index does not exist.

#### `disconnect()`

Disconnect from the Redis database.

#### `drop_documents(ids)`

Remove documents from the index by their document IDs.

This method converts document IDs to Redis keys automatically by applying
the index’s key prefix and separator configuration.

NOTE: Cluster users will need to incorporate hash tags into their
document IDs and only call this method with documents from a single hash
tag at a time.

* **Parameters:**
  **ids** (*Union* *[* *str* *,* *List* *[* *str* *]* *]*) – The document ID or IDs to remove from the index.
* **Returns:**
  Count of documents deleted from Redis.
* **Return type:**
  int

#### `drop_keys(keys)`

Remove a specific entry or entries from the index by it’s key ID.

* **Parameters:**
  **keys** (*Union* *[* *str* *,* *List* *[* *str* *]* *]*) – The document ID or IDs to remove from the index.
* **Returns:**
  Count of records deleted from Redis.
* **Return type:**
  int

#### `exists()`

Check if the index exists in Redis.

* **Returns:**
  True if the index exists, False otherwise.
* **Return type:**
  bool

#### `expire_keys(keys, ttl)`

Set the expiration time for a specific entry or entries in Redis.

* **Parameters:**
  * **keys** (*Union* *[* *str* *,* *List* *[* *str* *]* *]*) – The entry ID or IDs to set the expiration for.
  * **ttl** (*int*) – The time-to-live in seconds.
* **Return type:**
  int | *List*[int]

#### `fetch(id)`

Fetch an object from Redis by id.

The id is typically either a unique identifier,
or derived from some domain-specific metadata combination
(like a document id or chunk id).

* **Parameters:**
  **id** (*str*) – The specified unique identifier for a particular
  document indexed in Redis.
* **Returns:**
  The fetched object.
* **Return type:**
  Dict[str, Any]

#### `classmethod from_dict(schema_dict, **kwargs)`

Create a SearchIndex from a dictionary.

* **Parameters:**
  **schema_dict** (*Dict* *[* *str* *,* *Any* *]*) – A dictionary containing the schema.
* **Returns:**
  A RedisVL SearchIndex object.
* **Return type:**
  [SearchIndex](#searchindex)

```python
from redisvl.index import SearchIndex

index = SearchIndex.from_dict({
    "index": {
        "name": "my-index",
        "prefix": "rvl",
        "storage_type": "hash",
    },
    "fields": [
        {"name": "doc-id", "type": "tag"}
    ]
}, redis_url="redis://localhost:6379")
```

#### `classmethod from_existing(name, redis_client=None, redis_url=None, **kwargs)`

Initialize from an existing search index in Redis by index name.

* **Parameters:**
  * **name** (*str*) – Name of the search index in Redis.
  * **redis_client** (*Optional* *[* *Redis* *]*) – An
    instantiated redis client.
  * **redis_url** (*Optional* *[* *str* *]*) – The URL of the Redis server to
    connect to.
* **Raises:**
  * **ValueError** – If redis_url or redis_client is not provided.
  * **RedisModuleVersionError** – If required Redis modules are not installed.

#### `classmethod from_yaml(schema_path, **kwargs)`

Create a SearchIndex from a YAML schema file.

* **Parameters:**
  **schema_path** (*str*) – Path to the YAML schema file.
* **Returns:**
  A RedisVL SearchIndex object.
* **Return type:**
  [SearchIndex](#searchindex)

```python
from redisvl.index import SearchIndex

index = SearchIndex.from_yaml("schemas/schema.yaml", redis_url="redis://localhost:6379")
```

#### `info(name=None)`

Get information about the index.

* **Parameters:**
  **name** (*str* *,* *optional*) – Index name to fetch info about.
  Defaults to None.
* **Returns:**
  A dictionary containing the information about the index.
* **Return type:**
  dict

#### `key(id)`

Construct a redis key as a combination of an index key prefix (optional)
and specified id.

The id is typically either a unique identifier, or
derived from some domain-specific metadata combination (like a document
id or chunk id).

* **Parameters:**
  **id** (*str*) – The specified unique identifier for a particular
  document indexed in Redis.
* **Returns:**
  The full Redis key including key prefix and value as a string.
* **Return type:**
  str

#### `listall()`

List all search indices in Redis database.

* **Returns:**
  The list of indices in the database.
* **Return type:**
  List[str]

#### `load(data, id_field=None, keys=None, ttl=None, preprocess=None, batch_size=None)`

Load objects to the Redis database. Returns the list of keys loaded
to Redis.

RedisVL automatically handles constructing the object keys, batching,
optional preprocessing steps, and setting optional expiration
(TTL policies) on keys.

* **Parameters:**
  * **data** (*Iterable* *[* *Any* *]*) – An iterable of objects to store.
  * **id_field** (*Optional* *[* *str* *]* *,* *optional*) – Specified field used as the id
    portion of the redis key (after the prefix) for each
    object. Defaults to None.
  * **keys** (*Optional* *[* *Iterable* *[* *str* *]* *]* *,* *optional*) – Optional iterable of keys.
    Must match the length of objects if provided. Defaults to None.
  * **ttl** (*Optional* *[* *int* *]* *,* *optional*) – Time-to-live in seconds for each key.
    Defaults to None.
  * **preprocess** (*Optional* *[* *Callable* *]* *,* *optional*) – A function to preprocess
    objects before storage. Defaults to None.
  * **batch_size** (*Optional* *[* *int* *]* *,* *optional*) – Number of objects to write in
    a single Redis pipeline execution. Defaults to class’s
    default batch size.
* **Returns:**
  List of keys loaded to Redis.
* **Return type:**
  List[str]
* **Raises:**
  * **SchemaValidationError** – If validation fails when validate_on_load is enabled.
  * **RedisVLError** – If there’s an error loading data to Redis.

#### `paginate(query, page_size=30)`

Execute a given query against the index and return results in
paginated batches.

This method accepts a RedisVL query instance, enabling pagination of
results which allows for subsequent processing over each batch with a
generator.

* **Parameters:**
  * **query** (*BaseQuery*) – The search query to be executed.
  * **page_size** (*int* *,* *optional*) – The number of results to return in each
    batch. Defaults to 30.
* **Yields:**
  A generator yielding batches of search results.
* **Raises:**
  * **TypeError** – If the page_size argument is not of type int.
  * **ValueError** – If the page_size argument is less than or equal to zero.
* **Return type:**
  *Generator*

```python
# Iterate over paginated search results in batches of 10
for result_batch in index.paginate(query, page_size=10):
    # Process each batch of results
    pass
```

#### `NOTE`
The page_size parameter controls the number of items each result
batch contains. Adjust this value based on performance
considerations and the expected volume of search results.

#### `query(query)`

Execute a query on the index.

This method takes a BaseQuery or AggregationQuery object directly, and
handles post-processing of the search.

* **Parameters:**
  **query** (*Union* *[* *BaseQuery* *,* *AggregateQuery* *]*) – The query to run.
* **Returns:**
  A list of search results.
* **Return type:**
  List[Result]

```python
from redisvl.query import VectorQuery

query = VectorQuery(
    vector=[0.16, -0.34, 0.98, 0.23],
    vector_field_name="embedding",
    num_results=3
)

results = index.query(query)
```

#### `search(*args, **kwargs)`

Perform a search against the index.

Wrapper around the search API that adds the index name
to the query and passes along the rest of the arguments
to the redis-py ft().search() method.

* **Returns:**
  Raw Redis search results.
* **Return type:**
  Result

#### `set_client(redis_client, **kwargs)`

Manually set the Redis client to use with the search index.

This method configures the search index to use a specific Redis or
Async Redis client. It is useful for cases where an external,
custom-configured client is preferred instead of creating a new one.

* **Parameters:**
  **redis_client** (*Redis*) – A Redis or Async Redis
  client instance to be used for the connection.
* **Raises:**
  **TypeError** – If the provided client is not valid.

#### `property client: Redis | RedisCluster | None`

The underlying redis-py client object.

#### `property key_separator: str`

The optional separator between a defined prefix and key value in
forming a Redis key.

#### `property name: str`

The name of the Redis search index.

#### `property prefix: str`

The optional key prefix that comes before a unique key value in
forming a Redis key.

#### `property storage_type: StorageType`

The underlying storage type for the search index; either
hash or json.

<a id="asyncsearchindex-api"></a>

## AsyncSearchIndex

### `class AsyncSearchIndex(schema, *, redis_url=None, redis_client=None, connection_kwargs=None, validate_on_load=False, **kwargs)`

A search index class for interacting with Redis as a vector database in
async-mode.

The AsyncSearchIndex is instantiated with a reference to a Redis database
and an IndexSchema (YAML path or dictionary object) that describes the
various settings and field configurations.

```python
from redisvl.index import AsyncSearchIndex

# initialize the index object with schema from file
index = AsyncSearchIndex.from_yaml(
    "schemas/schema.yaml",
    redis_url="redis://localhost:6379",
    validate_on_load=True
)

# create the index
await index.create(overwrite=True, drop=False)

# data is an iterable of dictionaries
await index.load(data)

# delete index and data
await index.delete(drop=True)
```

Initialize the RedisVL async search index with a schema.

* **Parameters:**
  * **schema** ([*IndexSchema*]({{< relref "schema/#indexschema" >}})) – Index schema object.
  * **redis_url** (*Optional* *[* *str* *]* *,* *optional*) – The URL of the Redis server to
    connect to.
  * **redis_client** (*Optional* *[* *AsyncRedis* *]*) – An
    instantiated redis client.
  * **connection_kwargs** (*Optional* *[* *Dict* *[* *str* *,* *Any* *]* *]*) – Redis client connection
    args.
  * **validate_on_load** (*bool* *,* *optional*) – Whether to validate data against schema
    when loading. Defaults to False.

#### `async aggregate(*args, **kwargs)`

Perform an aggregation operation against the index.

Wrapper around the aggregation API that adds the index name
to the query and passes along the rest of the arguments
to the redis-py ft().aggregate() method.

* **Returns:**
  Raw Redis aggregation results.
* **Return type:**
  Result

#### `async batch_query(queries, batch_size=10)`

Asynchronously execute a batch of queries and process results.

* **Parameters:**
  * **queries** (*List* *[* *BaseQuery* *]*)
  * **batch_size** (*int*)
* **Return type:**
  *List*[*List*[*Dict*[str, *Any*]]]

#### `async batch_search(queries, batch_size=10)`

Asynchronously execute a batch of search queries.

This method takes a list of search queries and executes them in batches
to improve performance when dealing with multiple queries.

NOTE: Cluster users may need to incorporate hash tags into their query
to avoid cross-slot operations.

* **Parameters:**
  * **queries** (*List* *[* *SearchParams* *]*) – A list of search queries to execute.
    Each query can be either a string or a tuple of (query, params).
  * **batch_size** (*int* *,* *optional*) – The number of queries to execute in each
    batch. Defaults to 10.
* **Returns:**
  A list of search results corresponding to each query.
* **Return type:**
  List[Result]

```python
queries = [
    "hello world",
    ("goodbye world", {"num_results": 5}),
]

results = await index.batch_search(queries)
```

#### `async clear()`

Clear all keys in Redis associated with the index, leaving the index
available and in-place for future insertions or updates.

NOTE: This method requires custom behavior for Redis Cluster because here,
we can’t easily give control of the keys we’re clearing to the user so they
can separate them based on hash tag.

* **Returns:**
  Count of records deleted from Redis.
* **Return type:**
  int

#### `connect(redis_url=None, **kwargs)`

[DEPRECATED] Connect to a Redis instance. Use connection parameters in \_\_init_\_.

* **Parameters:**
  **redis_url** (*str* *|* *None*)

#### `async create(overwrite=False, drop=False)`

Asynchronously create an index in Redis with the current schema
: and properties.

* **Parameters:**
  * **overwrite** (*bool* *,* *optional*) – Whether to overwrite the index if it
    already exists. Defaults to False.
  * **drop** (*bool* *,* *optional*) – Whether to drop all keys associated with the
    index in the case of overwriting. Defaults to False.
* **Raises:**
  * **RuntimeError** – If the index already exists and ‘overwrite’ is False.
  * **ValueError** – If no fields are defined for the index.
* **Return type:**
  None

```python
# create an index in Redis; only if one does not exist with given name
await index.create()

# overwrite an index in Redis without dropping associated data
await index.create(overwrite=True)

# overwrite an index in Redis; drop associated data (clean slate)
await index.create(overwrite=True, drop=True)
```

#### `async delete(drop=True)`

Delete the search index.

* **Parameters:**
  **drop** (*bool* *,* *optional*) – Delete the documents in the index.
  Defaults to True.
* **Raises:**
  **redis.exceptions.ResponseError** – If the index does not exist.

#### `async disconnect()`

Disconnect from the Redis database.

#### `async drop_documents(ids)`

Remove documents from the index by their document IDs.

This method converts document IDs to Redis keys automatically by applying
the index’s key prefix and separator configuration.

NOTE: Cluster users will need to incorporate hash tags into their
document IDs and only call this method with documents from a single hash
tag at a time.

* **Parameters:**
  **ids** (*Union* *[* *str* *,* *List* *[* *str* *]* *]*) – The document ID or IDs to remove from the index.
* **Returns:**
  Count of documents deleted from Redis.
* **Return type:**
  int

#### `async drop_keys(keys)`

Remove a specific entry or entries from the index by it’s key ID.

* **Parameters:**
  **keys** (*Union* *[* *str* *,* *List* *[* *str* *]* *]*) – The document ID or IDs to remove from the index.
* **Returns:**
  Count of records deleted from Redis.
* **Return type:**
  int

#### `async exists()`

Check if the index exists in Redis.

* **Returns:**
  True if the index exists, False otherwise.
* **Return type:**
  bool

#### `async expire_keys(keys, ttl)`

Set the expiration time for a specific entry or entries in Redis.

* **Parameters:**
  * **keys** (*Union* *[* *str* *,* *List* *[* *str* *]* *]*) – The entry ID or IDs to set the expiration for.
  * **ttl** (*int*) – The time-to-live in seconds.
* **Return type:**
  int | *List*[int]

#### `async fetch(id)`

Asynchronously etch an object from Redis by id. The id is typically
either a unique identifier, or derived from some domain-specific
metadata combination (like a document id or chunk id).

* **Parameters:**
  **id** (*str*) – The specified unique identifier for a particular
  document indexed in Redis.
* **Returns:**
  The fetched object.
* **Return type:**
  Dict[str, Any]

#### `classmethod from_dict(schema_dict, **kwargs)`

Create a SearchIndex from a dictionary.

* **Parameters:**
  **schema_dict** (*Dict* *[* *str* *,* *Any* *]*) – A dictionary containing the schema.
* **Returns:**
  A RedisVL SearchIndex object.
* **Return type:**
  [SearchIndex](#searchindex)

```python
from redisvl.index import SearchIndex

index = SearchIndex.from_dict({
    "index": {
        "name": "my-index",
        "prefix": "rvl",
        "storage_type": "hash",
    },
    "fields": [
        {"name": "doc-id", "type": "tag"}
    ]
}, redis_url="redis://localhost:6379")
```

#### `async classmethod* from_existing(name, redis_client=None, redis_url=None, **kwargs)`

Initialize from an existing search index in Redis by index name.

* **Parameters:**
  * **name** (*str*) – Name of the search index in Redis.
  * **redis_client** (*Optional* *[* *Redis* *]*) – An
    instantiated redis client.
  * **redis_url** (*Optional* *[* *str* *]*) – The URL of the Redis server to
    connect to.

#### `classmethod from_yaml(schema_path, **kwargs)`

Create a SearchIndex from a YAML schema file.

* **Parameters:**
  **schema_path** (*str*) – Path to the YAML schema file.
* **Returns:**
  A RedisVL SearchIndex object.
* **Return type:**
  [SearchIndex](#searchindex)

```python
from redisvl.index import SearchIndex

index = SearchIndex.from_yaml("schemas/schema.yaml", redis_url="redis://localhost:6379")
```

#### `async info(name=None)`

Get information about the index.

* **Parameters:**
  **name** (*str* *,* *optional*) – Index name to fetch info about.
  Defaults to None.
* **Returns:**
  A dictionary containing the information about the index.
* **Return type:**
  dict

#### `key(id)`

Construct a redis key as a combination of an index key prefix (optional)
and specified id.

The id is typically either a unique identifier, or
derived from some domain-specific metadata combination (like a document
id or chunk id).

* **Parameters:**
  **id** (*str*) – The specified unique identifier for a particular
  document indexed in Redis.
* **Returns:**
  The full Redis key including key prefix and value as a string.
* **Return type:**
  str

#### `async listall()`

List all search indices in Redis database.

* **Returns:**
  The list of indices in the database.
* **Return type:**
  List[str]

#### `load(data, id_field=None, keys=None, ttl=None, preprocess=None, concurrency=None, batch_size=None)`

Asynchronously load objects to Redis. Returns the list of keys loaded
to Redis.

RedisVL automatically handles constructing the object keys, batching,
optional preprocessing steps, and setting optional expiration
(TTL policies) on keys.

* **Parameters:**
  * **data** (*Iterable* *[* *Any* *]*) – An iterable of objects to store.
  * **id_field** (*Optional* *[* *str* *]* *,* *optional*) – Specified field used as the id
    portion of the redis key (after the prefix) for each
    object. Defaults to None.
  * **keys** (*Optional* *[* *Iterable* *[* *str* *]* *]* *,* *optional*) – Optional iterable of keys.
    Must match the length of objects if provided. Defaults to None.
  * **ttl** (*Optional* *[* *int* *]* *,* *optional*) – Time-to-live in seconds for each key.
    Defaults to None.
  * **preprocess** (*Optional* *[* *Callable* *]* *,* *optional*) – A function to
    preprocess objects before storage. Defaults to None.
  * **batch_size** (*Optional* *[* *int* *]* *,* *optional*) – Number of objects to write in
    a single Redis pipeline execution. Defaults to class’s
    default batch size.
  * **concurrency** (*int* *|* *None*)
* **Returns:**
  List of keys loaded to Redis.
* **Return type:**
  List[str]
* **Raises:**
  * **SchemaValidationError** – If validation fails when validate_on_load is enabled.
  * **RedisVLError** – If there’s an error loading data to Redis.

```python
data = [{"test": "foo"}, {"test": "bar"}]

# simple case
keys = await index.load(data)

# set 360 second ttl policy on data
keys = await index.load(data, ttl=360)

# load data with predefined keys
keys = await index.load(data, keys=["rvl:foo", "rvl:bar"])

# load data with preprocessing step
def add_field(d):
    d["new_field"] = 123
    return d
keys = await index.load(data, preprocess=add_field)
```

#### `async paginate(query, page_size=30)`

Execute a given query against the index and return results in
paginated batches.

This method accepts a RedisVL query instance, enabling async pagination
of results which allows for subsequent processing over each batch with a
generator.

* **Parameters:**
  * **query** (*BaseQuery*) – The search query to be executed.
  * **page_size** (*int* *,* *optional*) – The number of results to return in each
    batch. Defaults to 30.
* **Yields:**
  An async generator yielding batches of search results.
* **Raises:**
  * **TypeError** – If the page_size argument is not of type int.
  * **ValueError** – If the page_size argument is less than or equal to zero.
* **Return type:**
  *AsyncGenerator*

```python
# Iterate over paginated search results in batches of 10
async for result_batch in index.paginate(query, page_size=10):
    # Process each batch of results
    pass
```

#### `NOTE`
The page_size parameter controls the number of items each result
batch contains. Adjust this value based on performance
considerations and the expected volume of search results.

#### `async query(query)`

Asynchronously execute a query on the index.

This method takes a BaseQuery or AggregationQuery object directly, runs
the search, and handles post-processing of the search.

* **Parameters:**
  **query** (*Union* *[* *BaseQuery* *,* *AggregateQuery* *]*) – The query to run.
* **Returns:**
  A list of search results.
* **Return type:**
  List[Result]

```python
from redisvl.query import VectorQuery

query = VectorQuery(
    vector=[0.16, -0.34, 0.98, 0.23],
    vector_field_name="embedding",
    num_results=3
)

results = await index.query(query)
```

#### `async search(*args, **kwargs)`

Perform an async search against the index.

Wrapper around the search API that adds the index name
to the query and passes along the rest of the arguments
to the redis-py ft().search() method.

* **Returns:**
  Raw Redis search results.
* **Return type:**
  Result

#### `set_client(redis_client)`

[DEPRECATED] Manually set the Redis client to use with the search index.
This method is deprecated; please provide connection parameters in \_\_init_\_.

* **Parameters:**
  **redis_client** (*Redis* *|* *RedisCluster* *|* *Redis* *|* *RedisCluster*)

#### `property client: Redis | RedisCluster | None`

The underlying redis-py client object.

#### `property key_separator: str`

The optional separator between a defined prefix and key value in
forming a Redis key.

#### `property name: str`

The name of the Redis search index.

#### `property prefix: str`

The optional key prefix that comes before a unique key value in
forming a Redis key.

#### `property storage_type: StorageType`

The underlying storage type for the search index; either
hash or json.
