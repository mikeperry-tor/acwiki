These are instructions for setting up a Snowflake bridge on Debian 10.

Sorry, these instructions are incomplete. Currently they only document the `systemctl edit` and `setcap` commands that are necessary to allow snowflake-server to bind to ports 80 and 443.

Override the `NoNewPrivileges` setting to enable snowflake-server to bind to low-numbered ports. (See #18356 for background.)
```
root# systemctl edit tor@.service
        [Service]
        NoNewPrivileges=no
root# systemctl edit tor@default.service
        [Service]
        NoNewPrivileges=no
```

Install snowflake-server. Run a `setcap` command to enable the program to bind to low-numbered ports.
```
root# install --owner root snowflake-server /usr/local/bin/snowflake-server
root# setcap 'cap_net_bind_service=+ep' /usr/local/bin/snowflake-server
root# service tor restart
```