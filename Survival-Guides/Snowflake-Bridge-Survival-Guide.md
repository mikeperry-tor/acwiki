Snowflake bridge survival guide
===============================

IP addresses:
```
37.218.242.151
2a00:c6c0:0:151:4:8f94:69f5:7c01
```

SSH fingerprints:
* `2048 SHA256:bP9tfPeIqkZkeKK1wcNT5t3CLyePz8oglFLRcdlP+gQ root@node (RSA)`
* `1024 SHA256:ji5FxcUh6gjLj7RHl6ffHTRMW62Gp+8ZmGoL0p5nVl0 root@node (DSA)`
* `256 SHA256:rl1WUhqOk3D2h2hwcK4x2HRPcnowUJuKnxQXYXOCXuk root@node (ED25519)`

Tor fingerprints:
* Bridge fingerprint 2B280B23E1107BB62ABFC40DDCC8824814F80A72
* Hashed fingerprint 5481936581E23D2D178105D44DB6915AB06BFB7F
* https://metrics.torproject.org/rs.html#details/5481936581E23D2D178105D44DB6915AB06BFB7F

Upgrading snowflake-server. You need to give the new binary permission to bind ports 443 and 80. This cheat sheet is also commented in `/etc/tor/torrc`.
1. `service tor stop`
2. `install --owner root ~/new-server /usr/local/bin/snowflake-server`
3. `setcap 'cap_net_bind_service=+ep' /usr/local/bin/snowflake-server`
4. `service tor start`

Check /var/log/syslog and /var/log/tor/snowflake-server.log for error messages. If there are `bind: permission denied` errors, ensure that you have run the `setcap` command, and that the tor `NoNewPrivileges=no` configuration from the [Snowflake Bridge Installation Guide](Survival Guides/Snowflake Bridge Installation Guide) is in place.

Standalone proxy-go instances
-----------------------------

The standalone proxy-go instances are managed by runit. You can see a list of possible instances under `/etc/service`. They are set up to periodically restart themselves in case of a hang.
```
sv status snowflake-proxy-standalone-17h        # check status
sv start snowflake-proxy-standalone-17h  # start
sv stop snowflake-proxy-standalone-17h   # stop
ps xww | grep runsvdir  # check for error in the run script
```
Logs are stored in `/home/snowflake-proxy/*.log.d`. 

Upgrading proxy-go. Copy the binary to /usr/local/bin/proxy-go, then run:
```
sv restart snowflake-proxy-standalone-17h
sv restart snowflake-proxy-standalone-29h
```
(Adjust as needed if there are other named services under /etc/service.)

Adding a new instance:
```
cd /etc/runit
mkdir -p my-instance/log
cat > my-instance/run <<EOF
#!/bin/sh
exec chpst -u snowflake-proxy timeout 17h /usr/local/bin/proxy-go -broker https://snowflake-broker.bamsoftware.com/ 2>&1
EOF
cat > my-instance/log/run <<EOF
#!/bin/sh
exec chpst -u snowflake-proxy svlogd /home/snowflake-proxy/my-instance.log.d
EOF
chmod +x my-instance/run my-instance/log/run
cd /etc/service
ln -s /etc/runit/my-instance/
mkdir /home/snowflake-proxy/my-instance.log.d
chown snowflake-proxy:nogroup /home/snowflake-proxy/my-instance.log.d
sv start my-instance
```

Firewall configuration is in `/etc/ferm/ferm.conf`. Run `service ferm restart` after making changes.