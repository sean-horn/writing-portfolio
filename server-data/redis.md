# Methods for accessing Redis data.

## Access Using 'redis-cli'

The `redis-cli` tool is located in `/opt/opscode/embedded/bin`.

    export PATH=/opt/opscode/embedded/bin:$PATH

### Chef Server 12.2.0+ requires redis authentication when making redis requests

The following command should parse out the redis password and set a `REDIS_PASSWORD` environment variable for convenience.

```
export REDIS_PASSWORD=`</etc/opscode/private-chef-secrets.json python -c 'import sys, json; print json.load(sys.stdin)["redis_lb"]["password"]'`
```

### Chef Server 11+ `dl_default` key

Enterprise Chef Server 11 and Chef Server 12 only have the `dl_default` key stored in redis.
This key is used to enable/disable dark launch features.

### List All Keys

```
root@chef:/# redis-cli -p 16379 -a "$REDIS_PASSWORD" KEYS '*'
1) "dl_default"
```

### List All Values of a Key

```
root@chef:/# redis-cli -p 16379 -a "$REDIS_PASSWORD" HGETALL dl_default
 1) "503_mode"
 2) "false"
 3) "couchdb_containers"
 4) "false"
 5) "couchdb_groups"
 6) "false"
 7) "couchdb_acls"
 8) "false"
 9) "couchdb_association_requests"
10) "false"
11) "couchdb_organizations"
12) "false"
13) "couchdb_associations"
14) "false"
```

### Maintenance Mode for the whole Chef Server (all orgs)

The following will cause all frontends to return HTTP 503.

```
root@chef:/# redis-cli -p 16379 -a "$REDIS_PASSWORD" HSET dl_default 503_mode true
(integer) 0
```

The following disables maintenance mode.

```
root@chef:/# redis-cli -p 16379 -a "$REDIS_PASSWORD" HSET dl_default 503_mode false
(integer) 0
```

### Maintenance Mode for a specific organization

Enable maintenance mode for organization "demo".

```
root@chef:/# export ORG_NAME=demo
root@chef:/# redis-cli -p 16379 -a "$REDIS_PASSWORD" HSET dl_org_$ORG_NAME 503_mode true
(integer) 1
```

Confirm the value.

```
root@chef:/# redis-cli -p 16379 -a "$REDIS_PASSWORD" HGETALL dl_org_$ORG_NAME
1) "503_mode"
2) "true"
```

Disable maintenance mode for organization "demo".

```
root@chef:/# redis-cli -p 16379 -a "$REDIS_PASSWORD" DEL dl_org_$ORG_NAME
(integer) 1
```
Confirm that maintenance mode for organization "demo" has been deleted.

```
root@chef:/# redis-cli -p 16379 -a "$REDIS_PASSWORD" HGETALL dl_org_$ORG_NAME
(empty list or set)
```
