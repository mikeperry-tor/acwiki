## Introduction

Moat allows users to fetch bridges from BridgeDB over a domain-fronted connection. It consists of a [meek](https://gitweb.torproject.org/pluggable-transports/meek.git/) server, some apache configs, and a BridgeDB distributor. This documentation demonstrates how it is deployed at https://bridges.torproject.org.

See the [API documentation](https://gitlab.torproject.org/tpo/anti-censorship/rdsys/-/blob/main/doc/moat.md?ref_type=heads) to use it.

## Server Setup

Clients connect to moat through the meek server, which then redirects traffic locally to the BridgeDB Moat distributor. These connections are facilitated through a sequence of ProxyPass rules:

```
ProxyPass /meek/ http://127.0.0.1:2000/
ProxyPass /moat/ http://127.0.0.1:3881/
```

The meek client makes a connection to https://bridges.torproject.org/meek (typically through a domain-fronted connection). This is passed to the meek server listening locally at [http://127.0.0.1:2000](http://127.0.0.1:2000).

```
#!/usr/bin/env bash

export TOR_PT_MANAGED_TRANSPORT_VER=1
export TOR_PT_SERVER_BINDADDR=meek-0.0.0.0:2000
export TOR_PT_SERVER_TRANSPORTS=meek
export TOR_PT_ORPORT=127.0.0.1:443

/srv/bridges.torproject.org/bin/meek-server --disable-tls & disown
```

Instead of connecting to the Tor network, the meek server's OR port points back to bridges.torproject.org by sending all traffic to [http://127.0.0.1:443](http://127.0.0.1:443).

The client can then use this meek tunnel to make a request to https://bridges.torproject.org/moat, which is passed to the Moat distributor listening on [http://127.0.0.1:3881/](http://127.0.0.1:3881/) (as configured with the BridgeDB configuration option `MOAT_HTTP_PORT = 3881`.

## Domain Fronting

Domain fronting for meek must be set up with a CDN or cloud provider. Typically how this works is you get a provider domain that serves as a front for the backend service (e.g., bridges.friendlycdn.net can be set up to send requests to bridges.torproject.org). For Moat, the host is set up to forward requests to https://bridges.torproject.org/meek so that the ProxyPass rules can redirect these requests to the meek server. This friendly CDN will also host a number of front domains (e.g., cdn.friendly.net) that can be sent in the SNI to prevent blocking, while bridges.friendlycdn.net is sent in the `Host` header.

## Client Setup

### using lyrebird

From one side we run lyrebird meek:

```
$ export TOR_PT_MANAGED_TRANSPORT_VER=1
$ export TOR_PT_CLIENT_TRANSPORTS=meek_lite
$ export TOR_PT_STATE_LOCATION=datadir/pt_state
$ export TOR_PT_EXIT_ON_STDIN_CLOSE=1
$ ./lyrebird
VERSION 1
STATUS TYPE=version IMPLEMENTATION="lyrebird" VERSION="devel"
CMETHOD meek_lite socks5 127.0.0.1:41373
CMETHODS DONE
```

Then we use the port from the output of lyrebird for my curl connection, using the targets line as user:password (I urlencoded as curl requires it):

```
❯ curl -x socks5h://targets%3Dhttps%3A%2F%2F1723079976.rsc.cdn77.org%7Ccdn.zk.mk%2Bwww.cdn77.com%2C:https%3A%2F%2Fbespoke-strudel-c243cc.netlify.app%7Cvuejs.org%2Bnetlify.com@127.0.0.1:41373 https://bridges.torproject.org/moat/circumvention/builtin |jq
{
  "meek": [
    "meek_lite 192.0.2.20:80 url=https://1603026938.rsc.cdn77.org front=www.phpmyadmin.net utls=HelloRandomizedALPN"
  ],
...
```

### using meek-client

The client opens a meek tunnel to the Moat server by passing in the service provider and front URLs.

```
$ export TOR_PT_MANAGED_TRANSPORT_VER=1
$ export TOR_PT_CLIENT_TRANSPORTS=meek
$ ./meek-client --url https://1723079976.rsc.cdn77.org --front=www.phpmyadmin.net
```

The meek client will open a SOCKS proxy on a local port and proxy all requests through the meek tunnel to the BridgeDB server. The client can then send requests to the Moat distributor at https://bridges.torproject.org/moat.

```
$ curl --socks5 127.0.0.1:44467 https://bridges.torproject.org/moat/
```

## API users

Applications using this API and point of contact for them:
- Brave
- Briar (contact@briarproject.org)
- Guardian Project (@n8fr8)
- OnionShare (@mig5)
- Tor Browser (@morgan)
