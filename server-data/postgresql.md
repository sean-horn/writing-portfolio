# Methods for accessing PostgreSQL data.

## Identify Connection Usage

If all connections are used then you won't be able to connect using psql.
The following commands will show you what is using the connections.

Login to the active backend server.

    pgrep -lf postgres

    ss -ontap '( dport = :postgres or sport = :postgres )'

## Access Server Using 'psql' Client

Login to the active backend server.

In EC 11.1.8+ the permissions for accessing postgresql have been tightened. To bypass these restrictions and login without a password you can `su` to the `opscode-pgsql` user and run `psql`.

Log into the postgresql database.

    su -l opscode-pgsql -c 'psql'

Setting the PAGER and LESS environment variables makes query results much more enjoyable.

    su -l opscode-pgsql -c 'PAGER=less LESS="-iMSx4 -FX" psql'

This method of access makes it easy to pipe commands directly into `psql`.

For example, you can set a user's password without an interactive `psql` shell like this.

    echo "ALTER USER name PASSWORD 'password';" | su -l opscode-pgsql -c 'psql'

# Some commonly used commands.

## Get Connection Statistics

    SELECT * FROM pg_stat_activity;

## List roles.

    \du

## Set a role/user password.

    ALTER USER name PASSWORD 'password';

## List all databases and their size.

    \l+

## Switch to the `opscode_chef` database.

    \c opscode_chef

## List tables and their size.

    \dt+

## List schemas.

    \dn

## List size of "relations" (including tables) for a selected database.

Reference: https://wiki.postgresql.org/wiki/Disk_Usage

```
SELECT relname AS "relation", pg_size_pretty(pg_relation_size(C.oid)) AS "size" FROM pg_class C LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace) WHERE nspname NOT IN ('pg_catalog', 'information_schema') ORDER BY pg_relation_size(C.oid) DESC;
```

## Use SQL to Query Database

This query will list all fields of all records in the `nodes` table.

    select * from nodes;
