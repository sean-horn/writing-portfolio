## Tools for Managing RabbitMQ


### rabbitmqctl

[rabbitmqctl man page](http://www.rabbitmq.com/man/rabbitmqctl.1.man.html)

`rabbitmqctl` provides some management capabilities

```
# Add Chef Server's embedded/bin to your path
export PATH=/opt/opscode/embedded/bin:$PATH

rabbitmqctl status

rabbitmqctl environment

rabbitmqctl list_connections name recv_cnt send_cnt user vhost

rabbitmqctl list_users

rabbitmqctl list_vhosts

rabbitmqctl list_queues -p /analytics

rabbitmqctl list_queues -p /chef name memory messages | grep -v "\s0$"

rabbitmqctl change_password <USERNAME> <PASSWORD>

# Cap a Chef Server 12 Server's Queues. Without this, a busy Chef Server will quickly queue itself into the ground #if it's companion Analytics system goes offline or stops consuming from the /analytics/alaska queue
rabbitmqctl set_policy -p /analytics max_length '(erchef|alaska|notifier.notifications|notifier_config)' '{"max-length":10000}' --apply-to queues
```

### rabbitmqadmin

[rabbitmqadmin docs](https://www.rabbitmq.com/management-cli.html)

`rabbitmqadmin` provides a lot more management capabilities (e.g. ability to delete queues)

```
# Add Chef Server's embedded/bin to your path
export PATH=/opt/opscode/embedded/bin:$PATH

# Bind rabbitmq_management to localhost for security
cat >/var/opt/opscode/rabbitmq/etc/rabbitmq.config <<EOF
[
 {rabbitmq_management, [ {listener, [ {port, 15672}, {ip, "127.0.0.1"} ]} ]}
].
EOF

# Create a needed directory on rabbitmq 2.7.1
mkdir -p /etc/rabbitmq

# Enable rabbitmq_management plugin which will open the rabbitmq HTTP API on port 15672
#   This requires restarting rabbitmq which may cause 5xx responses from the API until rabbitmq is running again
rabbitmq-plugins enable rabbitmq_management
chef-server-ctl restart rabbitmq

# Download and make executable the rabbitmqadmin cli tool
wget http://localhost:15672/cli/rabbitmqadmin -P /opt/opscode/embedded/bin
chmod a+x /opt/opscode/embedded/bin/rabbitmqadmin


# List the rabbitmqctl passwords
grep -A4 rabbitmq /etc/opscode/private-chef-secrets.json

# Create the default rabbitmqadmin config file
cat >/root/.rabbitmqadmin.conf <<EOF
[chef]
username = chef
password = PASSWORD

[actions]
username = actions
password = ACTIONS_PASSWORD
EOF


# Give the rabbitmq users administrator access if appropriate
# Ref: https://www.rabbitmq.com/management.html#permissions
rabbitmqctl set_user_tags chef administrator
rabbitmqctl set_user_tags actions administrator


# List messages stats for all vhosts
rabbitmqadmin -N chef list vhosts

# List all queues for all vhosts that chef user can access (e.g. /chef and /reindexer)
rabbitmqadmin -N chef list queues

# List all queues for the /reindexer vhost
rabbitmqadmin -N chef -V /reindexer list queues


# List all queues for the /analytics vhost
rabbitmqadmin -N actions -V /analytics list queues

# Delete the erchef queue from /analytics vhost
rabbitmqadmin -N actions -V /analytics delete queue name=erchef


# Remove privileged access from the rabbitmq users if appropriate
# Ref: https://www.rabbitmq.com/management.html#permissions
rabbitmqctl set_user_tags chef
rabbitmqctl set_user_tags actions


# Disable rabbitmq_management plugin which will close the rabbitmq HTTP API on port 15672
#   This requires restarting rabbitmq which may cause 5xx responses from the API until rabbitmq is running again
rabbitmq-plugins disable rabbitmq_management
#   Delete the rabbitmq.config file if desired
rm /var/opt/opscode/rabbitmq/etc/rabbitmq.config
chef-server-ctl restart rabbitmq

# Delete the .rabbitmqadmin.conf file if desired
rm /root/.rabbitmqadmin.conf
```
