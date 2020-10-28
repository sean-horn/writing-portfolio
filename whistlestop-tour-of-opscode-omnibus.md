### Default Run List

The default run_list for an Enterprise Chef chef-solo run initiated by
`p-c-c reconfigure` is "private-chef::default"

https://github.com/opscode/opscode-omnibus/blob/9de836fb4168484a0dd0911289d6bbfaa10f404d/config/software/private-chef-cookbooks.rb

We point to that dna.json file from the `p-c-c reconfigure` command
here

https://github.com/opscode/omnibus-ctl/blob/master/lib/omnibus-ctl.rb#L281

### Get Runit Installed

Configure Startup! Either SysV (CentOS/RHEL5) or Upstart (Every other
supported OS)

https://github.com/opscode-cookbooks/enterprise-chef-common/blob/master/recipes/runit.rb

We call the above from here

https://github.com/opscode/opscode-omnibus/blob/master/files/private-chef-cookbooks/private-chef/recipes/default.rb#L156

### Entry Point for All Config

The starting point for all config, migrations, everything is

https://github.com/opscode/opscode-omnibus/blob/master/files/private-chef-cookbooks/private-chef/recipes/default.rb

Which runs a library function with its starting point here. This
produces both the runtime and saved config of the EC cluster or
Standalone. By default, a Standalone has and needs no
/etc/opscode/private-chef.rb config file for the PrivateChef
module to use. Defaults are built-in for a Standalone install.

https://github.com/opscode/opscode-omnibus/blob/master/files/private-chef-cookbooks/private-chef/libraries/private_chef.rb#L397

### Runit Component Definition for each Service

This definition implements the run control of the EC11 services
after the initial install of the EC cluster, for each service.

https://github.com/opscode-cookbooks/enterprise-chef-common/blob/master/definitions/component_runit_service.rb

For example, the above will be found in the named recipes for all
services, not just oc_bifrost

https://github.com/opscode/opscode-omnibus/blob/master/files/private-chef-cookbooks/private-chef/recipes/oc_bifrost.rb#L76


The embedded cookbooks show up in
*/opt/opscode/embedded/cookbooks* *enterprise* and *private-chef*
after the EC server is installed. It's the same code found in the
repos for whatever version was used to produce the EC11 release

https://github.com/opscode/opscode-omnibus/tree/master/files/private-chef-cookbooks/private-chef
https://github.com/opscode-cookbooks/enterprise-chef-common

### Partybus Upgrade Steps

These go in */var/opt/opscode/upgrades* on a running system. Partybus uses them to
migrate the database and do other necessary steps as part of a` p-c-c
upgrade`.

https://github.com/opscode/opscode-omnibus/tree/master/files/private-chef-upgrades

### Omnibus-ctl

The omnibus-ctl gem implements the open source `chef-server-ctl` and the base
of the `private-chef-ctl` commands.

https://github.com/opscode/omnibus-ctl/blob/master/lib/omnibus-ctl.rb

### Omnibus-ctl Plugins to Implement Full `private-chef-ctl`

These plugins go on top of the built-in ones in omnibus-ctl and implement
the extended functions of `private-chef-ctl`

https://github.com/opscode/opscode-omnibus/tree/master/files/private-chef-ctl-commands
