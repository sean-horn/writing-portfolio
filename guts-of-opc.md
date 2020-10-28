LVM, DRBD, and Keepalived are three sets of software that we use to
provide the storage, snapshot backup, and failover mechanisms in OPC.

DRBD and Keepalived are only present on backend machines in an HA
topology cluster. All other topologies have a single backend.

### LVM

In an OPC HA topology install, we setup a configuration where we have a
VG named "opscode", with an LV named "drbd". The active MASTER backend
will always be writing couchdb, postgres, rabbitmq, Solr, file
checksum, and other data that needs to be persisted to the
/dev/opscode/drbd LV, which will be mounted on
/var/opt/opscode/drbd/data by default.

In all other OPC topologies, there is only a single backend, with all
data being written directly to directories under /var/opt/opscode.

### Snapshot Backups

We have a script that can use LVM's snapshotting feature.  The script
tries to unmount and remove any existing snapshot LV with a name having a
given pattern, then rebuild and remount the snapshot.  There must be
sufficient free space in the LVM VG where the `lvcreate -s ...` call
is made, or it will fail. One of our OPC install prereqs is that there
be a large amount of free space in the /dev/opscode VG.

The default amount of disk space needed by the snapshot script is 40GB.
The location of the snapshot script is /opt/opscode/bin/drbd-backups
By default, the drbd-backups script mounts its snapshot at /var/opt/opscode/drbd-backups.
On OPC installs after 1.4.6, there is a deactivated cronjob in place at /opt/opscode/bin/opc_snapshot.
This script can be moved to /etc/cron.d/opc_snapshot and uncommented for use at a customer site.

The snapshot script is usable with topologies other than "HA" as it is a generic script.
However, its defaults assume that it will be used in an HA topology OPC configuration.

### DRBD

Manages a replicated set of two local disks, one for each of the two
members of the backend cluster. DRBD replicates over a network
interface every bit written to the actual local block device over
which it is overlayed. Every write to a certain device on a certain
mountpoint, by default the /dev/opscode/drbd LV and
/var/opt/opscode/drbd/data mountpoint, is replicated to the other
member over a network interface and written to the local disk
there. There will be no mounted filesystem on the BACKUP side of the
cluster, as in our configuration the backend cluster functions as an
ACTIVE/PASSIVE pair. Only during a failover will the members switch
roles and update their status.

The configuration file for DRBD is found at
/var/opt/opscode/drbd/etc/pc0.res on a running OPC backend system.

Status information for DRBD can be found in /proc/drbd.

### Keepalived

Controls a VIP, which gets switched between MASTER/BACKUP nodes during
a failover event. Interacts with DRBD through a cluster.sh
script. cluster.sh tries to swap the MASTER/BACKUP state of the
keepalived daemons and also the DRBD kernel modules when a keepalived
notices a problem with the other member daemon. 

The configuration file for keepalived is found at
/var/opt/opscode/keepalived/etc/keepalived.conf on a running OPC
system. The cluster.sh script is referenced here.

Status information for keepalived can be found in /var/log/{messages,syslog} depending on your OS.

### Networking

NOTE: Without some work in https://github.com/opscode/opscode-omnibus, OPC
is unable to function properly when configured with more than two
total interfaces.

This means that you will have one interface (public) configured with
the normal IP address of the system used to gain access to SSH and
like services. Additionally, this interface will be the one that holds
the VIP used by keepalived to designate the current MASTER member of
the backend cluster. The additional IP address is assigned to this
interface directly by keepalived.  The way that keepalived does the
assignment is similar to `ifconfig eth0:100 172.16.55.100 up`

The other interface (private or cluster) will be configured to handle
the DRBD replication traffic and keepalived heartbeat and cluster
status information. Often, this interface is connected to the other
member of the backend cluster through a crossover cable.

### General Info

In general, when a member of the backend cluster is the MASTER(keepalived) and PRIMARY(DRBD) member, the following will all reflect that state

* /proc/drbd
* `p-c-c ha-status`
* Has the VIP configured on a network interface
* /dev/drbd0 mounted on /var/opt/opscode/drbd/data
* The output from keepalived and DRBD in /var/log/{syslog,messages} or /var/opt/opscode/keepalived/cluster.log 
  (EC11) will show the node as the MASTER and PRIMARY respectively.

DRBD and keepalived must be in agreement about the state
of the cluster. When they are not, the cluster is not in a functioning
state, and keepalived tries to rectify this condition using the
cluster.sh script to gain control of the DRBD device.

The OPC docs have further information about other states the cluster
can get into, with especially good info on DRBD split brain.

### EC11 Postgresql Port Usage

1. Formula for postgresql connections on an HA or Tiered topology (The second term is the active backend):

  Total connections = Number of frontends * (bifrost + erchef + reporting) + (pushy +
bifrost + erchef)

  ![equation](http://www.forkosh.com/mimetex.cgi?c=$ totalconnections  =  numfrontends \\times (bifrost + erchef + reporting) + (pushy + bifrost + erchef) )

2. Formula for postgresql connections on a Standalone topology:

  Total connections = bifrost + erchef + reporting + pushy

  ![equation](http://www.forkosh.com/mimetex.cgi?c=$ totalconnections  =  bifrost + erchef + reporting + pushy )

## postgresql max_connections - 300
https://github.com/opscode/opscode-omnibus/blob/master/files/private-chef-cookbooks/private-chef/attributes/default.rb#L373

##bifrost - 20 connections
https://github.com/opscode/opscode-omnibus/blob/master/files/private-chef-cookbooks/private-chef/attributes/default.rb#L373

##erchef - 20 connections
https://github.com/opscode/opscode-omnibus/blob/master/files/private-chef-cookbooks/private-chef/attributes/default.rb#L144

##pushy - 20 connections
https://github.com/opscode/opscode-omnibus/blob/master/files/private-chef-cookbooks/private-chef/attributes/default.rb#L144

##reporting - 25 connections
https://github.com/opscode/omnibus-reporting/blob/master/files/opscode-reporting-cookbooks/opscode-reporting/attributes/default.rb#L48

### Glossary:

HA: High Availability  

LVM: Logical Volume Manager  

PV: LVM Physical Volume. Composed of actual hard drives, SAN volumes, etc.

VG: LVM Volume Group. A collection of PVs.  

LV: LVM Logical Volume. A virtual device created from free space in a VG. Behaves like a physical hard drive for linux.

DRBD: Distributed Replicated Block Device  

VIP: Virtual IP address. An IP address not permanently bound to any
given interface. Moved between interfaces to direct traffic to the
currently available backend node.  
  
Heartbeat: "Are you alive?, I'm alive"  

p-c-c ha-status: private-chef-ctl ha-status  

ACTIVE/BACKUP or PRIMARY/SECONDARY: Possible states for a member of an OPC backend cluster.
The previous states are associated with keepalived and DRBD, respectively.  


