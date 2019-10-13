# ElasticSearch Plugins Suite

Store full account and object data into indexed elastisearch database.

- [Motivation](#motivation)
- [Database Selection](#the-database-selection)
- [Technical](#technical)
- [Hardware needed](#hardware-needed)
- [Installation](#installation)
- [Running](#running)
  - [Checking if is working](#checking-if-is-working)
  - [Arguments](#arguments)
- [Usage](#usage)
  - [Kibana](#kibana)
  - [Direct queries](#direct-queries)
  - [Wrapper](#wrapper)
- [Going forward](#going-forward)

## Motivation

There are 2 main problems this plug-in tries to solve:

- The amount of RAM needed to run a full node with all the account history. Huge.
- Fast search inside operation fields directly querying the ES database.
- Fast each inside objects by any field.

## The database selection

Elastic search was selected for the main following reasons:

- Open source.
- Fast.
- Index oriented.
- Easy to install and start using.
- Send data from c++ using curl.
- Scalable and decentralized nodes of data possibilities.

## Technical

The `elasticsearch` plugin when active is connected to each block the node receives. Operations are extracted in a similar logic of the classic `account_history_plugin` but sent to the ES database instead of storing internally. All fields from the operation are indexed for fast search.

The `es_objects` plugin when active is connected to config specified objects types(limit order objects, asset objects, etc).

Both plugins work in a similar way, data is collected in plugin internal database until a good amount of them(configurable) is available, then is sent as a `_bulk` operation to ES. `_bulk` needs to be big when replaying but much more smaller when we are in sync to display real time data to end users.

Optimal numbers for speed/performance can depend on hardware, default values are provided. 

## Hardware needed

It is very recommended that you use SSD disks in your node if you are trying to synchronize bitshares mainnet. It will make the task a lot faster. Still, the process of synchronizing the mainnet can take a few days.

You need 1T of space to be safe for a while, 32G or more of RAM is recommended.

After elasticsearch is installed increase heap size depending in your RAM: 

`$ vi config/jvm.options`        

```
..
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms12g
-Xmx12g
...
```

## Installation

You need to have bitshares-core and its dependencies installed(https://github.com/bitshares/bitshares-core#getting-started).

In ubuntu 18.04 all the dependencies for elasticsearch database are installed by default. Just get the last version(or desired version) at:

https://www.elastic.co/downloads/elasticsearch

```
$ tar xvzf elasticsearch-7.4.0-linux-x86_64.tar.gz
$ cd elasticsearch-7.4.0/
$./bin/elasticsearch
```

ES will listen in `127.0.0.1:9200`. Try http://127.0.0.1:9200/ in your browser and you should see some info about the database if the service started correctly.

You can put the binary as a service, program haves a `--daemonize` option, can run inside `screen` or any other option that suits you in order to keep the database running. 

Please note ES does not run as root, make a normal user account by and proceed after:

`adduser elastic`

## Running

Clone the bitshares repo and install bitshares:

```
git clone https://github.com/bitshares/bitshares-core
cd bitshares-core
git checkout -t origin/develop
git submodule update --init --recursive
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo .
make
```

Start node with elasticsearch plugins enabled with default options. Make sure ES is running in localhost:9200

`./programs/witness_node/witness_node --plugins "elasticsearch es_objects"`

### Checking if it is working

By default you should see the following messages in the witness console while the node is loading:

```
...
572387ms th_a       elasticsearch_plugin.cpp:516  plugin_startup       ] elasticsearch ACCOUNT HISTORY: plugin_startup() begin
572387ms th_a       application.cpp:1235          startup_plugins      ] Plugin elasticsearch started
572395ms th_a       es_objects.cpp:405            plugin_startup       ] elasticsearch OBJECTS: plugin_startup() begin
572395ms th_a       application.cpp:1235          startup_plugins      ] Plugin es_objects started
...
575659ms th_a       es_objects.cpp:82             genesis              ] elasticsearch OBJECTS: inserting data from genesis
595710ms th_a       application.cpp:603           handle_block         ] Got block: #10000 00002710c3894322c3a33dabdcb4020c2918db6e time: 2015-10-13T23:15:42 transaction(s): 0 latency: 126190453710 ms from: cyrano  irreversible: 9789 (-211)
602395ms th_a       application.cpp:603           handle_block         ] Got block: #20000 00004e2032e9ec266a4eb088a5e0aa05bb24ff0c time: 2015-10-14T07:37:33 transaction(s): 0 latency: 126160349395 ms from: bitcube  irreversible: 19785 (-215)
...
```

In a new window can check the total rows in the bitshares index you have, the count number should be increasing as we sync:

```
$ curl -X GET 'http://localhost:9200/bitshares-*/data/_count?pretty=true' -H 'Content-Type: application/json' -d '
{
    "query" : {
        "bool" : { "must" : [{"match_all": {}}] }
    }
}
'
{
  "count" : 35000,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  }
}
$ 
```

Replace `bitshares-*` to `objects-*` to make sure you have items also in the objects indexes.

Note: If you see that you just have to wait to sync, you can execute this queries often to check progress, as long as the numbers are moving to the up side you are generally going fine.

You can also check your indexes with:

```
$ curl -X GET 'http://localhost:9200/_cat/indices'
yellow open objects-bitasset  CyYgodyOS3OzPCC9ED6K_Q 1 1    33   0  73.4kb  73.4kb
yellow open objects-balance   TCWc24MDSPG2PwxTGSLbNQ 1 1  1760   0 256.1kb 256.1kb
yellow open objects-asset     4TkUeOdwRnSeDUtqOs3B2g 1 1   457   0 171.5kb 171.5kb
yellow open objects-account   Tv-7TI_rTLSMqrDEj3FGYw 1 1 95405 652  75.5mb  75.5mb
yellow open objects-proposal  J5ok-YqNQyqW5psAxVARqQ 1 1     6   6  38.1kb  38.1kb
yellow open bitshares-2015-10 IBDfO9wqR5qtmY5_g4f81Q 1 1 35000   0  18.5mb  18.5mb
$ 
```

### Arguments

The ES plugin have the following parameters passed by command line or config file:

- `elasticsearch-node-url` - Database url, default: http://localhost:9200/
- `elasticsearch-bulk-replay` - Number of bulk documents to dump to the database on replay, default: 10000
- `elasticsearch-bulk-sync` - Number of bulk documents to index on a  synchronized chain, default: 100
- `elasticsearch-visitor` - Use visitor to index additional data - almost deprecated and off by default.
- `elasticsearch-basic-auth` - Username and password for a protected database,  default: empty
- `elasticsearch-index-prefix` - Prefix for the indexes (bitshares-)
- `elasticsearch-operation-object` - Save operation as object. Very important feature to search inside operation fields, default: on
- `elasticsearch-start-es-after-block` - Start doing ES job after block. An important maintenance option that will help if your node crash to restart the ES dumping only after the specifying block. By default this is 0 so all blocks will be saved but this is only to start from scratch.
- `elasticsearch-operation-string` - Save operation as string. Needed to serve history api calls. This is turned off by defualt as default `elasticsearch-mode` is to only save. If enabled will allow bitshares-core api call `get_account_history` to use ES instead of internal storage limited database.
- `elasticsearch-mode` - Mode of operation: only_save(0), only_query(1), all(2). Default: 0

Options for the `es_objects` plugins are:

- `es-objects-elasticsearch-url` - Database url, default: http://localhost:9200/
- `es-objects-auth` - username:password, empty by default.
- `es-objects-bulk-replay` - Bulk documents to send in replay, default: 10000
- `es-objects-bulk-sync` - Bulk documents to send when in sync, default: 100
- `es-objects-proposals` - Save proposal objects, on by default.
- `es-objects-accounts` - Store account objects, on by default.
- `es-objects-assets` - Store asset objects, on by default.
- `es-objects-balances` - Store balances objects, on by default.
- `es-objects-limit-orders` - Store limit order objects, off by default.
- `es-objects-asset-bitasset` - Store feed data, on by default.
- `es-objects-index-prefix` - Add a prefix to the index, default: objects-
- `es-objects-keep-only-current` - Keep only current state of the objects, if enabled will save all the object changes as different database entries.
- `es-objects-start-es-after-block` - Start doing ES job after block. Useful for synchronization after a crash.

## Usage

After your node is in sync you are in possession of a full node without the ram issues. A synchronized witness_node with ES will be using less than 10 gigs of ram:

```
 total          8604280K
root@NC-PH-1346-07:~# pmap 2183
```

What client side apps can do with this new data is kind of unlimited to client developer imagination but lets check some real world examples to see the benefits of this new feature.

### Get operations by account, time and operation type

References:
https://github.com/bitshares/bitshares-core/issues/358
https://github.com/bitshares/bitshares-core/issues/413
https://github.com/bitshares/bitshares-core/pull/405
https://github.com/bitshares/bitshares-core/pull/379
https://github.com/bitshares/bitshares-core/pull/430
https://github.com/bitshares/bitshares-ui/issues/68

This is one of the issues that has been requested constantly. It can be easily queried with ES plugin by calling the _search endpoint doing:

```
curl -X GET 'http://localhost:9200/bitshares-*/data/_search?pretty=true' -H 'Content-Type: application/json' -d '
{
    "query" : {
        "bool" : { "must" : [{"term": { "account_history.account.keyword": "1.2.282"}}, {"range": {"block_data.block_time": {"gte": "2015-10-26T00:00:00", "lte": "2015-10-29T23:59:59"}}}] }
    }
}
'
```
**Note** Response is removed from the samples to save space in the document. If you are here you may want to see the response in your own place. 

### Filter based on block number or block range

https://github.com/bitshares/bitshares-core/issues/61

```
curl -X GET 'http://localhost:9200/bitshares-*/data/_search?pretty=true' -H 'Content-Type: application/json' -d '
{
    "query" : {
        "bool" : { "must" : [{"term": { "account_history.account.keyword": "1.2.356589"}}, {"range": {"block_data.block_num": {"gte": "17824289", "lte": "17824290"}                                                                                                                  
}}] }                                                          
    }
}
'
```

### Get operations by transaction hash

Refs: https://github.com/bitshares/bitshares-core/pull/373

The `get_transaction_id` can be done as:

```
curl -X GET 'http://localhost:9200/bitshares-*/data/_search?pretty=true' -H 'Content-Type: application/json' -d '
{
    "query" : {
        "bool" : { "must" : [{"term": { "block_data.block_num": 19421114}},{"term": { "operation_history.trx_in_block": 0}}] }
    }
}
'
```

The above will return all ops inside trx, if you only need the trx_id field you can add `source` and just return the fields you need:

```
curl -X GET 'http://localhost:9200/bitshares-*/data/_search?pretty=true' -H 'Content-Type: application/json' -d '
{
    "_source": ["block_data.trx_id"],
    "query" : {
        "bool" : { "must" : [{"term": { "block_data.block_num": 19421114}},{"term": { "operation_history.trx_in_block": 0}}] }
    }
}
'
```

The `get_transaction_from_id` is very easy:

```
curl -X GET 'http://localhost:9200/bitshares-*/data/_search?pretty=true' -H 'Content-Type: application/json' -d '
{
    "query" : {
        "bool" : { "must" : [{"term": { "block_data.trx_id": "6f2d5064637391089127aa9feb36e2092347466c"}}] }
    }
}
'
```

## Going forward

The reader will need to learn more about elasticsearch and lucene query language in order to make more complex queries.

All needed can be found at https://www.elastic.co/guide/en/elasticsearch/reference/6.2/index.html

By the same team of elasticsearch there is a front end named `kibana` (https://www.elastic.co/products/kibana). It is very easy to install and can do pretty good stuff like getting very detailed stats of the blockchain network.

### More visitor code = more indexed data = more filters to use

Just as an example, it will be easy to index asset of trading operations by extending the visitor code of them. point 3 of https://github.com/bitshares/bitshares-core/issues/358 request trading pair, can be solved by indexing the asset of the trading ops as mentioned.

Remember ES already have all the needed info in the `op` text field of the `operation_history` object. Client can get all the ops of an account, loop throw them and convert the `op` string into json being able to filter by the asset or any other field needed. There is no need to index everything but it is possible.

## Note on Duplicates

By using the `op_type` = `create` on each bulk line we send to the database and as we use an unique ID(ath id(2.9.X)) the plugin will not index any operation twice. If the node is on a replay, the plugin will start adding to database when it find a new record and never before. 

## Wrapper

It is not recommended to expose the elasticsearch api fully to the internet. Instead, applications will connect to a wrapper for data:

https://github.com/oxarbitrage/bitshares-es-wrapper

Elasticsearch database will listen in localhost and the wrapper in the same machine will expose the reduced set of API calls to the internet. 
