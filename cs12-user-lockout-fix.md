Support Remediation Documentation

Chef Server 12.0.0 through 12.0.4 shipped with Chef Server Issue
[66](https://github.com/chef/chef-server/issues/66).  The impact of the bug was that when LDAP was enabled,
users would be locked out of their accounts after updating their
user key.

This is resolved in 12.0.5; however, users who have already locked
themselves out still need their user records fixed.  

The underlying cuase of the problem is that the
external_authorization_uid field has been reset in the database.  To
resolve the problem for the user, do the following:

1) Log onto the backend master
2) Become the opscode-pgsql user:

    su - opscode-pgsql

3) Connect to postgres

    /opt/opscode/embedded/bin/psql opscode_chef

4) Find the affected user account:

    SELECT username, external_authentication_uid FROM users WHERE username='foo';

5) Update the user:

    BEGIN;
    UPDATE users SET external_authentication_uid='LDAP ID' WHERE username='foo'

6) Ensure only 1 user was updated (UPDATE 1 should be the output).  
7) If only 1 user was updated:

    COMMIT;

8) If more than 1 user was updated, rollback and check the where clause you used in 6:

    ROLLBACK;
