Instructions for setting up a Snowflake bridge on Debian 11.


## Introduction

The configuration of the Snowflake bridge is a little bit complicated, for performance reasons. The [usual way](https://community.torproject.org/relay/setup/bridge/) to run a pluggable transport server is to run one tor process, which itself runs and manages the pluggable transport process (e.g. snowflake-server) according to the `ServerTransportPlugin` and `ServerTransportListenAddr` options. This is how the Snowflake bridge was configured before tpo/anti-censorship/pluggable-transports/snowflake#40091, tpo/anti-censorship/pluggable-transports/snowflake#40095.

![Diagram of tor managing snowflake-server](uploads/2f0c30b846a9895628e98ee615b2ee12/snowflake-bridge-single.png)
[Inkscape SVG source code](uploads/cfa4c581073a52a232b8bc7c8fd6053c/snowflake-bridge-single.svg)

But the usual setup does not work with the amount of traffic the Snowflake bridge gets. The tor process becomes a bottleneck. tor is mostly single-threaded, so once it reaches 100% of one CPU core, it cannot go any faster, no matter how many CPUs there are total, how much the RAM, or how well the other programs are optimized.

To work around the problem of tor being limited to one CPU, we run multiple instances of tor, and put them behind [HAProxy](https://www.haproxy.org/), a load balancer. The tor instances all have the same identity keys and onion keys, so a client's fingerprint verification will check out, no matter which instance of tor the load balancer connects it to. The snowflake-server process is no longer run and managed by (any instance of) tor; instead it runs as a normal system daemon, managed by systemd. snowflake-server forwards connections to haproxy, and haproxy forwards connections to the multiple instances of tor.

There is one more complication, the extended ORPort (ExtORPOrt). The ExtORPort allows connections to be tagged with [a transport name and a client IP address](https://gitweb.torproject.org/torspec.git/tree/proposals/196-transport-control-ports.txt?id=554d63ad3a60b705c3a5cbe2e3e9b33094a049dd#n56); this metadata is essential for metrics. But in order to connect to the ExtORPort, you need to [authenticate](https://gitweb.torproject.org/torspec.git/tree/proposals/217-ext-orport-auth.txt?id=554d63ad3a60b705c3a5cbe2e3e9b33094a049dd#n75) using a secret key, stored in a file. Every instance of tor generates this secret key independently; there would be no way for snowflake-server to know which key to authenticate with, not knowing which instance of tor the load balancer will assign a connection to. To work around this problem, there is a shim called [extor-static-cookie](https://gitlab.torproject.org/dcf/extor-static-cookie) that presents an ExtORPort with a fixed, unchanging authentication key on a static port, and forwards the connections (again as ExtORPort) to tor, using that instance of tor's authentication key on an ephemeral port `ExtORPort auto`. One extor-static-cookie process is run per instance of tor, using `ServerTransportPlugin` and `ServerTransportListenAddr`.

The load-balanced setup looks like this:

![Diagram of snowflake-server talking to haproxy, haproxy talking to each of the four extor-static-cookie instances, and the extor-static-cookie instances talking to their respective instance of tor](uploads/5675ac7c12bbd4c1df6922abcc002c70/snowflake-bridge-loadbalanced.png)
[Inkscape SVG source code](uploads/365f465663f395b6db82ca1b38dd17c6/snowflake-bridge-loadbalanced.svg)

References:

* [[tor-relays] How to reduce tor CPU load on a single bridge?](https://forum.torproject.net/t/tor-relays-how-to-reduce-tor-cpu-load-on-a-single-bridge/1483)
  * The same as https://lists.torproject.org/pipermail/tor-relays/2021-December/020156.html.
* tpo/anti-censorship/pluggable-transports/snowflake#28651
  * Other ideas for scaling the Snowflake bridge that don't quite work yet, because of the need to match the fingerprint the client expects.


## General system setup

Set up APT and etckeeper.

```
# apt update
# apt upgrade
# apt install etckeeper
```

Set up a firewall. You need to expose ports 22, 80, and 443.

```
# apt install ferm # Enable ferm on bootup? Yes
# vi /etc/ferm/ferm.conf
	domain (ip ip6) {
	    table filter {
	        chain INPUT {
	            #...
	            # allow SSH connections
	            proto tcp dport ssh ACCEPT;
	            # allow snowflake-server ACME HTTP-01
	            proto tcp dport http ACCEPT;
	            # allow snowflake-server WebSocket
	            proto tcp dport https ACCEPT;
	        }
	        # ...
	    }
	}
# service ferm restart
# etckeeper commit "firewall"
```

Put a link to the survival guide in /etc/motd so that it shows when logging in.

```
# echo >/etc/motd "https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Survival-Guides/Snowflake-Bridge-Survival-Guide"
```

Add a normal user account with sudo. Repeat the process for additional users.

```
# adduser --gecos "" user
# adduser user sudo
# su - user
$ mkdir -p -m 700 .ssh
$ echo >>.ssh/authorized_keys "ssh-ed25519 AAAA..."
$ exit
# etckeeper commit "add user account"
```

Log in as a normal user and disable login for root.

```
$ sudo -s
# vi /etc/ssh/sshd_config
	PermitRootLogin no
	AllowUsers user
	PasswordAuthentication no
# service sshd restart
# rm /root/.ssh/authorized_keys
# etckeeper commit "sshd access control"
```

You can set up your local ~/.ssh/config with a host alias for easier access:

```
Host snowflake
HostName <IP>
User <user>
IdentityFile ~/.ssh/snowflake-key
```

Increase the [ephemeral port range](https://support.torproject.org/relay-operators/relay-bridge-overloaded/#tcp-port-exhaustion).

```
# echo "net.ipv4.ip_local_port_range = 15000 64000" > /etc/sysctl.d/ip_local_port_range.conf
# sysctl -w net.ipv4.ip_local_port_range="15000 64000"
# etckeeper commit "net.ipv4.ip_local_port_range"
```


## extor-static-cookie

Build extor-static-cookie. You can compile it locally, then copy it to the bridge server. `CGO_ENABLED=0` results in a static binary that does not depend on a specific version of libc. Also generate a static cookie file.

```
$ git clone https://gitlab.torproject.org/dcf/extor-static-cookie
$ cd extor-static-cookie
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build
$ ./gen-auth-cookie > static_extended_orport_auth_cookie
$ scp extor-static-cookie static_extended_orport_auth_cookie snowflake:
```

Then, on the bridge, install extor-static-cookie and the static cookie file.

```
# install --owner root /home/user/extor-static-cookie /usr/local/bin/
# mkdir -p /etc/extor-static-cookie
# install --owner root /home/user/static_extended_orport_auth_cookie /etc/extor-static-cookie/
# etckeeper commit "static ExtORPOrt auth cookie"
```


## tor

Install tor.

```
# apt install tor
```

Create multiple instances of tor using [tor-instance-create](https://manpages.debian.org/testing/tor/tor-instance-create.8.en.html). Do not start them yet. They will all need to share the same keys so that they have the same bridge fingerprint. If you adjust the number of tor instances, you will have to adjust haproxy.cfg to match.

```
# tor-instance-create snowflake1
# tor-instance-create snowflake2
# tor-instance-create snowflake3
# tor-instance-create snowflake4
```

Copy the keys into each instance.

```
# for instance in snowflake1 snowflake2 snowflake3 snowflake4; do cp -r -v keys/ "/var/lib/tor-instances/$instance/" && chown -R -v _tor-"$instance" "/var/lib/tor-instances/$instance/keys"; done
```

**Important** Create placeholder directories to prevent the tor instances from rotating their onion keys. Without this step, the instances will independently change their onion keys every 28 days, and clients will be anable to connect, unless they are lucky enough to connect to the instance whose descriptor they have cached. Also make the onion key files read-only, as defense in depth in case tor changes its file renaming strategy. For more information, see https://forum.torproject.net/t/tor-relays-how-to-reduce-tor-cpu-load-on-a-single-bridge/1483/23.

```
# rm -fv /var/lib/tor-instances/*/keys/secret_onion_key{,_ntor}.old
# mkdir -m 700 -p -v /var/lib/tor-instances/*/keys/secret_onion_key{,_ntor}.old
# for instance in snowflake1 snowflake2 snowflake3 snowflake4; do for dir in /var/lib/tor-instances/"$instance"/keys/secret_onion_key{,_ntor}.old; do echo >"$dir/README" "This directory exists to prevent onion key rotation. See https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Survival-Guides/Snowflake-Bridge-Installation-Guide"; done; done
# chmod -v -w /var/lib/tor-instances/"$instance"/keys/secret_onion_key{,_ntor}
```

Edit the instance-specific torrc files. They all have the same contents, except that `ServerTransportListenAddr` will count through `127.0.0.1:10001`, `127.0.0.1:10002`, etc., and `Nickname` will count through `flakey1`, `flakey2`, etc. Replace `NUM` in the example below:

```
# vi /etc/tor/instances/*/torrc
	# This is one of the instances of tor to which snowflake-server forwards
	# incoming traffic, via a load balancer. Each of the instances has a different
	# ServerTransportListenAddr.
	BridgeRelay 1
	AssumeReachable 1
	BridgeDistribution none
	ORPort 127.0.0.1:auto
	ExtORPort auto
	SocksPort 0
	ServerTransportPlugin snowflake exec /usr/local/bin/extor-static-cookie /etc/extor-static-cookie/static_extended_orport_auth_cookie
	ServerTransportListenAddr snowflake 127.0.0.1:1000NUM
	Nickname flakeyNUM
```

Then start all the instances of tor. Check /var/log/syslog for error messages.

```
# service tor restart
# etckeeper commit "tor snowflake instances"
```


## haproxy

Install haproxy, which is a load balancer. Configure it to listen on port 10000, and forward to the instances of tor at 10001, 10002, .... Append the configuration to the end of what is there already.

```
# apt install haproxy
# vi /etc/haproxy/haproxy.cfg
	frontend tor
		mode tcp
		bind 127.0.0.1:10000
		default_backend tor-instances
		option dontlog-normal
		timeout client 600s
	backend tor-instances
		mode tcp
		timeout server 600s
		server snowflake1 127.0.0.1:10001
		server snowflake2 127.0.0.1:10002
		server snowflake3 127.0.0.1:10003
		server snowflake4 127.0.0.1:10004
# service haproxy restart
# etckeeper commit "haproxy"
```


## snowflake-server

Build snowflake-server. You can compile it locally, then copy it to the bridge server. `CGO_ENABLED=0` results in a static binary that does not depend on a specific version of libc.

```
$ git clone https://git.torproject.org/pluggable-transports/snowflake.git
$ cd snowflake/server
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build
$ scp server snowflake:snowflake-server
```

Then, on the bridge, install snowflake-server and a systemd service file for it.

```
# install --owner root /home/user/snowflake-server /usr/local/bin/snowflake-server
# adduser --system snowflake-server
# vi /etc/systemd/system/snowflake-server.service
	# This is an instance of snowflake-server that is run outside of tor, so that
	# it can forward to a load balancer that in turn forwards to more than one
	# instance of tor.
	#
	# snowflake-server expects to run as a managed transport under another process.
	# The Environment options set it up so that snowflake-server thinks it is being
	# run in this way. There is no error checking, however; snowflake-server's
	# pluggable transports messages (e.g. SMETHOD, SMETHOD-ERROR) are ignored.
	#
	# https://lists.torproject.org/pipermail/tor-relays/2022-January/020183.html
	 
	[Unit]
	Description=Snowflake pluggable transport server
	 
	[Service]
	Type=exec
	Restart=on-failure
	User=snowflake-server
	StateDirectory=snowflake-server
	LogsDirectory=snowflake-server
	 
	AmbientCapabilities=CAP_NET_BIND_SERVICE
	NoNewPrivileges=true
	ProtectHome=true
	ProtectSystem=strict
	PrivateTmp=true
	PrivateDevices=true
	ProtectClock=true
	ProtectKernelModules=true
	ProtectKernelLogs=true
	LimitNOFILE=131072

	Environment=TOR_PT_MANAGED_TRANSPORT_VER=1
	Environment=TOR_PT_SERVER_TRANSPORTS=snowflake
	Environment=TOR_PT_SERVER_BINDADDR=snowflake-[::]:443
	Environment=TOR_PT_EXTENDED_SERVER_PORT=127.0.0.1:10000
	Environment=TOR_PT_AUTH_COOKIE_FILE=/etc/extor-static-cookie/static_extended_orport_auth_cookie
	Environment=TOR_PT_STATE_LOCATION=%S/snowflake-server/pt_state
	Environment=TOR_PT_EXIT_ON_STDIN_CLOSE=0

	ExecStart=/usr/local/bin/snowflake-server --acme-hostnames snowflake.bamsoftware.com,snowflake.freehaven.net,snowflake.torproject.net --acme-email dcf@torproject.org --log %L/snowflake-server/snowflake-server.log

	[Install]
	WantedBy=multi-user.target
# systemctl enable --now snowflake-server
# etckeeper commit "snowflake-server service"
```

Check for errors in `service snowflake-server status` and /var/log/snowflake-server/snowflake-server.log.
