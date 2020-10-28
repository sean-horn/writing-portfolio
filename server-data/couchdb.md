# Methods for accessing CouchDB data.

## Access Futon WebUI

Forward 127.0.0.1:5984 of the OPC server to your workstation.

    ssh vagrant@33.33.33.10 -L 5984:127.0.0.1:5984 -i ~/.vagrant.d/insecure_private_key

Browse to [http://127.0.0.1:5984/_utils/](http://127.0.0.1:5984/_utils/)

## Access Using Curl

The first part of the CouchDB database url is made up of "chef_" followed by the GUID of an organization.  This GUID can be obtained using the [Orgmapper tool](../orgmapper.md).

### List All Databases

    curl -s 'http://localhost:5984/_all_dbs

### List All Documents in a Database

    curl -s 'http://localhost:5984/chef_5c4e9086bc254a9f829a30774b87c22b/_all_docs

### Databases May Have Views Used for Querying and Reporting

Chef Server has many views including the following:

* cookbooks
* data_bag_items
* data_bags
* environments
* roles

### List All Documents in a View

The following will list all roles:

    curl -s 'http://localhost:5984/chef_5c4e9086bc254a9f829a30774b87c22b/_design/roles/_view/all'

### Use Views to Find a Document by Name

The following will find a role named `testrole`:

    curl -s 'http://localhost:5984/chef_5c4e9086bc254a9f829a30774b87c22b/_design/roles/_view/all?key="testrole"'

Response:

    {"total_rows":1,"offset":0,"rows":[
    {"id":"e9bbe3ac-447f-4eb7-b6ae-54a44ca7b4dd","key":"testrole","value":{"_id":"e9bbe3ac-447f-4eb7-b6ae-54a44ca7b4dd","_rev":"1-35e559b527a988d2d094a3288ef78b08","name":"testrole","description":"","json_class":"Chef::Role","default_attributes":{},"override_attributes":{},"chef_type":"role","run_list":[],"env_run_lists":{}}}
    ]}

### Get Contents of a Document

    curl -s 'http://localhost:5984/chef_5c4e9086bc254a9f829a30774b87c22b/e9bbe3ac-447f-4eb7-b6ae-54a44ca7b4dd'

Response:

    {"_id":"e9bbe3ac-447f-4eb7-b6ae-54a44ca7b4dd","_rev":"1-35e559b527a988d2d094a3288ef78b08","name":"testrole","description":"","json_class":"Chef::Role","default_attributes":{},"override_attributes":{},"chef_type":"role","run_list":[],"env_run_lists":{}}


### Delete a Document

The following deletes the `testrole` document.

    curl -s -X DELETE 'http://localhost:5984/chef_5c4e9086bc254a9f829a30774b87c22b/e9bbe3ac-447f-4eb7-b6ae-54a44ca7b4dd?rev=1-35e559b527a988d2d094a3288ef78b08'

Response:

    {"ok":true,"id":"e9bbe3ac-447f-4eb7-b6ae-54a44ca7b4dd","rev":"2-1a8fec5b528a902c127718b01eb44fce"}
