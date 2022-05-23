## Bridge sites

### snowflake-01 (flakey)

IP addresses:
```
193.187.88.42
2a0c:dd40:1:b::42
```

SSH port 4722.

SSH fingerprints:

```
256 SHA256:WK24RKtRyfmKGWfM5M4nFyKkoSNBFqJb5zVtMAW0B2g (ECDSA)
256 SHA256:6+UwLKXZxvDhj7gK9afgnfPzw9oRLCRb4pzoRVip9KI (ED25519)
3072 SHA256:cG7BnmuOUjEklGZhmSGhNeVVJcphM1iJ5dKvfgL4KHI (RSA)
```

* Bridge fingerprint 2B280B23E1107BB62ABFC40DDCC8824814F80A72
* Hashed fingerprint 5481936581E23D2D178105D44DB6915AB06BFB7F
* [Relay search page](https://metrics.torproject.org/rs.html#details/5481936581E23D2D178105D44DB6915AB06BFB7F)


### snowflake-02 (crusty)

IP addresses:
```
141.212.118.18
2607:f018:600:8:be30:5bff:fef1:c6fa
```

SSH fingerprints:

```
256 SHA256:NmTVmA4WCgoR8xmu3T065TeEe3k7uTEAMKeLAfB37vM  (ECDSA)
256 SHA256:VpARXHZ6eH8AcifmFiHVCrF8Exxmdp7C8qBHmolbDu8  (ED25519)
3072 SHA256:rCNv1Il4tAM9B4l4nWH7BpYxrxZcMHkJhXxi5ma4Bs4  (RSA)
```

* Bridge fingerprint 8838024498816A039FCBBAB14E6F40A0843051FA
* Hashed fingerprint 91DA221A149007D0FD9E5515F5786C3DD07E4BB0
* [Relay search page](https://metrics.torproject.org/rs.html#details/91DA221A149007D0FD9E5515F5786C3DD07E4BB0)


## Components

![Diagram of snowflake-server talking to haproxy, haproxy talking to each of the four extor-static-cookie instances, and the extor-static-cookie instances talking to their respective instance of tor](uploads/5675ac7c12bbd4c1df6922abcc002c70/snowflake-bridge-loadbalanced.png)

The interacting components on the bridge are a bit complicated, for performance reasons. See the [installation guide](Survival Guides/Snowflake Bridge Installation Guide#introduction) for the reasoning. There are four main components:

* [snowflake-server](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/tree/main/server): Receives WebSocket connections from Snowflake proxies, manages Turbo Tunnel sessions, forwards sessions as TCP connections to HAProxy. Listens externally on port 443 (and port 80, for ACME certificate renewal).
* [HAProxy](https://www.haproxy.org/): Load balancer. Receives connections from snowflake-server and balances them over the multiple instances of tor, via their respective extor-static-cookie interfaces. Listens on 127.0.0.1:10000.
* tor: There are multiple instances of tor, because one is not enough for the load on the bridge. Each instance's `ORPort` is blocked from outside access by the firewall, and `ExtORPort auto` makes them listen for ExtORPort connections on an ephemeral localhost port. Each instance of tor runs an extor-static-cookie, which provides HAProxy a stable ExtORPort port number, and provides snowflake-server (via HAProxy) a stable authentication key.
* [extor-static-cookie](https://gitlab.torproject.org/dcf/extor-static-cookie): Exposes an ExtORPort interface that uses an unchanging authentication key. These listen on 127.0.0.1, on port numbers 10000+*N*, where *N* is the instance number 1, 2, ….


## Upgrading snowflake-server

```
# install --owner root /home/user/snowflake-server /usr/local/bin/
# service snowflake-server restart
```

Check for errors in `service snowflake-server status` and /var/log/snowflake-server/snowflake-server.log.

See /etc/systemd/system/snowflake-server.service for configuration variables.


## Upgrading extor-static-cookie

```
# install --owner root /home/user/extor-static-cookie /usr/local/bin/
# service tor restart
```

The static ExtORPort authentication cookie does not need to be stable long-term. If it accidentally gets lost or damaged, you can create a new one using the [gen-auth-cookie](https://gitlab.torproject.org/dcf/extor-static-cookie/-/blob/main/gen-auth-cookie) script in the extor-static-cookie source code. You will need to restart tor and snowflake-server.

```
# extor-static-cookie/gen-auth-cookie > static_extended_orport_auth_cookie
# service tor restart
# service snowflake-server restart
```


## Adding more tor instances

See the [installation guide](Survival Guides/Snowflake Bridge Installation Guide#tor). After creating the tor instances, you will also have add new `server` lines in the `backend tor-instances` section of /etc/haproxy/haproxy.cfg.


## Firewall

Firewall configuration is in `/etc/nftables.conf`. Run `systemd restart nftables` after making changes.
