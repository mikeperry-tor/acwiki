IP address:
```
143.110.214.222
```

Tor fingerprints:

* Bridge fingerprint 50B99540A96C5E9F9F7704BAAE11DF01564711F4
* Hashed fingerprint A84C946BF4E14E63A3C92E140532A4594F2C24CD

Upgrading `conjure-server`. You need to give the new binary permission to bind port 80. This cheat sheet is also commented in `/etc/tor/torrc`.

1. `service tor stop`
2. `install --owner root ~/new-server /usr/local/bin/conjure-server`
3. `setcap 'cap_net_bind_service=+ep' /usr/local/bin/conjure-server`
4. `service tor start`

Check `/var/log/syslog` and `/var/log/tor/conjure-server.log` for error messages. If `conjure-server.log` shows `bind: permission denied`, ensure that you have run the `setcap` command, and that the `NoNewPrivileges=no`.

### Adding Conjure stations to the allowlist

The conjure bridge only accepts HAProxy connections from known Conjure stations. You can add a new station IP from which to accept connections by modifying the `--allowed-stations` argument in the `ServerTransportPlugin` line of the `/etc/tor/torrc` file to be a comma separated list of station IPs.