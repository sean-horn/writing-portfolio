# Methods for accessing Solr data.

## Access Admin WebUI

Forward 127.0.0.1:8983 of the OPC server to your workstation.

    ssh vagrant@33.33.33.10 -L 8983:127.0.0.1:8983 -i ~/.vagrant.d/insecure_private_key

Browse to [http://127.0.0.1:8983/solr/admin/](http://127.0.0.1:8983/solr/admin/)

## Access Using Curl

The X_CHEF_database_CHEF_X value is made up of "chef_" followed by the GUID of an organization.  This GUID can be obtained using the [Orgmapper tool](../orgmapper.md).

### List 1000 Items in an Index

The following lists 1000 items in the role index:

    curl -s 'http://localhost:8983/solr/select?q=X_CHEF_database_CHEF_X:chef_5c4e9086bc254a9f829a30774b87c22b%20AND%20X_CHEF_type_CHEF_X:role&sort=X_CHEF_id_CHEF_X+asc&rows=1000&wt=json'

Response:

    {"responseHeader":{"status":0,"QTime":5,"params":{"sort":"X_CHEF_id_CHEF_X asc","wt":"json","q":"X_CHEF_database_CHEF_X:chef_5c4e9086bc254a9f829a30774b87c22b AND X_CHEF_type_CHEF_X:role","rows":"1000"}},"response":{"numFound":1,"start":0,"docs":[{"X_CHEF_id_CHEF_X":"e9bbe3ac-447f-4eb7-b6ae-54a44ca7b4dd","X_CHEF_database_CHEF_X":"chef_5c4e9086bc254a9f829a30774b87c22b","X_CHEF_type_CHEF_X":"role","X_CHEF_timestamp_CHEF_X":"2013-08-02T18:44:32.691Z"}]}}

### Delete All Items in an Index

The following deletes all items in the role index.

    curl -s 'http://localhost:8983/solr/update?stream.body=<delete><query>X_CHEF_database_CHEF_X:chef_YOUR_ORG_GUID%20AND%20X_CHEF_type_CHEF_X:role</query></delete>&commit=true'

### Search for a Particular Item

The following query searches for a specific item by its ID.

    curl -s 'http://localhost:8983/solr/select?q=X_CHEF_database_CHEF_X:chef_5c4e9086bc254a9f829a30774b87c22b%20AND%20X_CHEF_id_CHEF_X:e9bbe3ac-447f-4eb7-b6ae-54a44ca7b4dd&sort=X_CHEF_id_CHEF_X+asc&rows=1000&wt=json'

Response:

    {"responseHeader":{"status":0,"QTime":1,"params":{"sort":"X_CHEF_id_CHEF_X asc","wt":"json","q":"X_CHEF_database_CHEF_X:chef_5c4e9086bc254a9f829a30774b87c22b AND X_CHEF_id_CHEF_X:e9bbe3ac-447f-4eb7-b6ae-54a44ca7b4dd","rows":"1000"}},"response":{"numFound":1,"start":0,"docs":[{"X_CHEF_id_CHEF_X":"e9bbe3ac-447f-4eb7-b6ae-54a44ca7b4dd","X_CHEF_database_CHEF_X":"chef_5c4e9086bc254a9f829a30774b87c22b","X_CHEF_type_CHEF_X":"role","X_CHEF_timestamp_CHEF_X":"2013-08-02T18:44:32.691Z"}]}}

### Delete a Particular Item

The following query deletes an indexed item by its ID.

    curl -s 'http://localhost:8983/solr/update?stream.body=<delete><query>X_CHEF_database_CHEF_X:chef_5c4e9086bc254a9f829a30774b87c22b%20AND%20X_CHEF_id_CHEF_X:e9bbe3ac-447f-4eb7-b6ae-54a44ca7b4dd</query></delete>&commit=true'

Response:

    <?xml version="1.0" encoding="UTF-8"?>
    <response>
    <lst name="responseHeader"><int name="status">0</int><int name="QTime">36</int></lst>
    </response>
