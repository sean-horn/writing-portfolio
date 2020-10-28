# Data Storage and Caching

The API services talk to a number of data storage services when processing API requests.

The following lists the data storage services and what data they store:

## CouchDB

### All Chef data other than node and user data

* cookbooks - some metadata
* data bags
* data bag items
* environments
* roles

Databases: `/var/opt/opscode/couchdb/db/` on the backend

[Data Access Methods](couchdb.md)

## Local Filesystem

### Cookbook Content

Cached Cookbooks: `/var/opt/opscode/nginx/cache/` on the frontend

Cookbooks: `/var/opt/opscode/opscode-chef/checksum/` on the backend

Sandboxes: `/var/opt/opscode/opscode-chef/sandbox/` on the backend

## PostgreSQL

### Node and user data

Database: `/var/opt/opscode/postgresql/data` on the backend

[Data Access Methods](postgresql.md)

## RabbitMQ

### Expander and Expander-reindexer messages

Queues: `/var/opt/opscode/rabbitmq/db/` on the backend

## Redis

### Caches certain authorization data for opscode-authz

Database: `/var/opt/opscode/redis/data` on the backend

[Data Access Methods](redis.md)

## Solr

### Search indexes

A list and description of available search indexes can be found in the online documentation.

[http://docs.opscode.com/essentials_search.html#search-indexes](http://docs.opscode.com/essentials_search.html#search-indexes)

A list can also be obtained with an authenticated GET request of the `/search` API endpoint.

    knife exec -E 'pp api.get("/search")'

Response:

    {"client"=>"https://api.opscode.piab/organizations/ponyville/search/client",
     "environment"=>"https://api.opscode.piab/organizations/ponyville/search/environment",
     "node"=>"https://api.opscode.piab/organizations/ponyville/search/node",
     "role"=>"https://api.opscode.piab/organizations/ponyville/search/role"}

Data: `/var/opt/opscode/opscode-solr/data` on the backend

[Data Access Methods](solr.md)
