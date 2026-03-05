Instructions for setting up a Snowflake bridge on Debian 11.


## Introduction

The configuration of the Snowflake bridge is a little bit complicated, for performance reasons. The [usual way](https://community.torproject.org/relay/setup/bridge/) to run a pluggable transport server is to run one tor process, which itself runs and manages the pluggable transport process (e.g. snowflake-server) according to the `ServerTransportPlugin` and `ServerTransportListenAddr` options. This is how the Snowflake bridge was configured before tpo/anti-censorship/pluggable-transports/snowflake#40091 and tpo/anti-censorship/pluggable-transports/snowflake#40095.

![Diagram of tor managing snowflake-server](uploads/d1511ae780cd0b408adf01666944b074/snowflake-bridge-single.png)
[Inkscape SVG source code](uploads/58a24ab0294ceaf624e6d42bfd65f4a5/snowflake-bridge-single.svg)

But the usual setup does not work with the amount of traffic the Snowflake bridge gets. The tor process becomes a bottleneck. tor is mostly single-threaded, so once it reaches 100% of one CPU core, it cannot go any faster, no matter how many CPUs there are total, how much RAM there is, or how well the other programs are optimized.

To work around the problem of tor being limited to one CPU, we run multiple instances of tor, and put them behind [HAProxy](https://www.haproxy.org/), a load balancer. The tor instances all have the same identity keys and onion keys, so a client's fingerprint verification will check out, no matter which instance of tor the load balancer connects it to. The snowflake-server process is no longer run and managed by (any instance of) tor; instead it runs as a normal system daemon, managed by systemd. snowflake-server forwards connections to HAProxy, and HAProxy forwards connections to the multiple instances of tor.

There is one more complication, the extended ORPort (ExtORPort). The ExtORPort allows connections to be tagged with [a transport name and a client IP address](https://gitweb.torproject.org/torspec.git/tree/ext-orport-spec.txt?id=29245fd50d1ee3d96cca52154da4d888f34fedea#n145); this metadata is essential for metrics. But in order to connect to the ExtORPort, you need to [authenticate](https://gitweb.torproject.org/torspec.git/tree/proposals/217-ext-orport-auth.txt?id=554d63ad3a60b705c3a5cbe2e3e9b33094a049dd#n75) using a secret key, stored in a file. Every instance of tor generates this secret key independently; there would be no way for snowflake-server to know which key to authenticate with, not knowing which instance of tor the load balancer will assign a connection to. To work around this problem, there is a shim called [extor-static-cookie](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/extor-static-cookie) that presents an ExtORPort with a fixed, unchanging authentication key on a static port, and forwards the connections (again as ExtORPort) to tor, using that instance of tor's authentication key on an ephemeral port. One extor-static-cookie process is run per instance of tor, using `ServerTransportPlugin` and `ServerTransportListenAddr`.

The load-balanced setup looks like this:

![Diagram of snowflake-server talking to HAProxy, HAProxy talking to each of the four extor-static-cookie instances, and the extor-static-cookie instances talking to their respective instance of tor](uploads/03f9830a35603ac7cfcf425f4bfd9494/snowflake-bridge-loadbalanced.png)
[Inkscape SVG source code](uploads/0b54e689e44f0679a97d83e252333a69/snowflake-bridge-loadbalanced.svg)

References:

* [[tor-relays] How to reduce tor CPU load on a single bridge?](https://forum.torproject.net/t/tor-relays-how-to-reduce-tor-cpu-load-on-a-single-bridge/1483)
  * The same as https://lists.torproject.org/pipermail/tor-relays/2021-December/020156.html.
* tpo/anti-censorship/pluggable-transports/snowflake#28651
  * Other ideas for scaling the Snowflake bridge that don't quite work yet, because of the need to match the fingerprint the client expects.


## Localhost address ranges

We reserve different localhost IP address ranges for different purposes. This is to mitigate problems with ephemeral port exhaustion when there are many live connections. (See tpo/anti-censorship/pluggable-transports/snowflake#40198, tpo/anti-censorship/pluggable-transports/snowflake#40201.)

The purpose of each address range is listed in the table below. "\*" stands for a random IP address octet, or a random ephemeral port.

|Address range|Purpose|Where configured|
|-------------|-------|----------------|
|127.0.0.1:10000           |HAProxy listen port (dialed by snowflake-server)|`bind` in haproxy.cfg, `TOR_PT_EXTENDED_SERVER_PORT` in snowflake-server.service|
|127.0.1.\*:\*               |snowflake-server source address when dialing HAProxy|`orport-srcaddr` in `TOR_PT_SERVER_TRANSPORT_OPTIONS` in snowflake-server.service|
|127.0.2.<var>N</var>:\*    |HAProxy source address when dialing extor-static-cookie instance <var>N</var>|`source` in haproxy.cfg|
|127.0.3.<var>N</var>:10000|extor-static-cookie instance <var>N</var> listen port (dialed by HAProxy)|`ServerTransportListenAddr` in torrc, `server` in haproxy.cfg|
|127.0.4.<var>N</var>:\*    |tor instance <var>N</var> ExtORPort (dialed by extor-static-cookie)|`ExtORPort` in torrc|
|127.0.5.\*:\*               |extor-static-cookie source address when dialing tor ExtORPort|`orport-srcaddr` in `ServerTransportOptions` in torrc|


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
	            #...
	        }
	        # ...
	    }
	}
# systemctl restart ferm
# etckeeper commit "firewall"
```

Set the time zone to UTC.

```
# dpkg-reconfigure tzdata
# etckeeper commit "set time zone to UTC"
```

Put a link to the survival guide in /etc/motd so that it shows when logging in.

```
# echo >/etc/motd "https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Survival-Guides/Snowflake-Bridge-Survival-Guide"
# etckeeper commit "motd"
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
# systemctl restart ssh
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

Set sysctl parameters:
* Increase the [ephemeral port range](https://support.torproject.org/relay-operators/relay-bridge-overloaded/#tcp-port-exhaustion).
* Increase the nftables connection tracking limits (tpo/anti-censorship/pluggable-transports/snowflake#40239).

```
# echo "net.ipv4.ip_local_port_range = 15000 63999" > /etc/sysctl.d/ip_local_port_range.conf
# (echo "net.netfilter.nf_conntrack_max = 524288"; echo "net.netfilter.nf_conntrack_buckets = 524288") > /etc/sysctl.d/nf_conntrack.conf
# sysctl --system
# etckeeper commit "sysctl"
```

You also need to cause the nf_conntrack module to be loaded early in the boot process, otherwise the sysctl settings above will not take effect after a reboot (tpo/anti-censorship/pluggable-transports/snowflake#40259). Create a file /etc/modules-load.d/nf_conntrack.conf:

```
# Load nf_conntrack early so that its sysctl settings later in the boot process take effect.
nf_conntrack
```

Install vnstat and configure it:

```
apt install vnstat
sed -i.BAK -e 's/;5MinuteHours .*/5MinuteHours 744/1' /etc/vnstat.conf
```

## extor-static-cookie

Build extor-static-cookie. You can compile it locally, then copy it to the bridge server. `CGO_ENABLED=0` results in a static binary that does not depend on a specific version of libc. Also generate a static cookie file.

```
$ git clone https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/extor-static-cookie
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
# etckeeper commit "static ExtORPort auth cookie"
```


## tor

Install tor.

```
# apt install tor tor-geoipdb
```

Create multiple instances of tor using [tor-instance-create](https://manpages.debian.org/testing/tor/tor-instance-create.8.en.html). Do not start them yet. They will all need to share the same keys so that they have the same bridge fingerprint. If you adjust the number of tor instances, you will have to adjust haproxy.cfg to match.

```
# INSTANCES=$(for n in $(seq 1 4); do echo "snowflake$n"; done)
# for instance in $INSTANCES; do tor-instance-create "$instance"; done
```

Edit the instance-specific torrc files. (But don't start the instances yet.) All the torrc files have the same contents, except that `ServerTransportListenAddr` will count through `127.0.3.1:10000`, `127.0.3.2:10000`, etc., and `Nickname` will count through <code><var>NICKNAME</var>1</code>, <code><var>NICKNAME</var>2</code>, etc. Replace <code><var>NICKNAME</var></code> and <code><var>N</var></code> in the example below:

<pre>
# vi /etc/tor/instances/*/torrc
	# This is one of the instances of tor to which snowflake-server forwards
	# incoming traffic, via a load balancer. Each of the instances has a different
	# ServerTransportListenAddr.
	BridgeRelay 1
	AssumeReachable 1
	BridgeDistribution none
	ORPort 127.0.0.1:auto # unused
	ExtORPort 127.0.4.<mark><var>N</var></mark>:auto
	SocksPort 0
	ServerTransportPlugin snowflake exec /usr/local/bin/extor-static-cookie /etc/extor-static-cookie/static_extended_orport_auth_cookie
	ServerTransportListenAddr snowflake 127.0.3.<mark><var>N</var></mark>:10000
	ServerTransportOptions snowflake orport-srcaddr=127.0.5.0/24
	Nickname <mark><var>NICKNAME</var><var>N</var></mark>
</pre>

The next step is different depending on whether you are installing a new bridge for the first time, or moving an existing bridge (with an existing relay fingerprint) to a new server.

* If you are installing a new bridge for the first time, start and stop the tor@snowflake1 instance in order to generate keys, then copy the keys into the other instances.
  ```
  # systemctl start tor@snowflake1 && systemctl stop tor@snowflake1
  # for instance in $INSTANCES; do if [ $instance != snowflake1 ]; then cp -r -v /var/lib/tor-instances/snowflake1/keys "/var/lib/tor-instances/$instance/" && chown -R -v _tor-"$instance" "/var/lib/tor-instances/$instance/keys"; fi; done
  ```
  Make a backup copy of one of the keys directories. You will need the backup if you ever reinstall or move the bridge and want to keep the same relay fingerprint.
* If you are moving an existing bridge to a new server, copy the keys/ directory from the existing server (as a tar file, for example) into the new instances.
  ```
  # for instance in $INSTANCES; do cp -r -v keys/ "/var/lib/tor-instances/$instance/" && chown -R -v _tor-"$instance" "/var/lib/tor-instances/$instance/keys"; done
  ```

**Important** After installing the keys, create placeholder directories to prevent the tor instances from rotating their onion keys. Without this step, the instances will independently change their onion keys every 28 days, and clients will be unable to connect, unless they are lucky enough to connect to the instance whose descriptor they have cached. Also make the onion key files read-only, as defense in depth in case tor changes its file renaming strategy. For more information, see https://forum.torproject.net/t/tor-relays-how-to-reduce-tor-cpu-load-on-a-single-bridge/1483/23.

```
# for instance in $INSTANCES; do rm -fv /var/lib/tor-instances/"$instance"/keys/secret_onion_key{,_ntor}.old; done
# for instance in $INSTANCES; do mkdir -m 700 -p -v /var/lib/tor-instances/"$instance"/keys/secret_onion_key{,_ntor}.old; done
# for instance in $INSTANCES; do for dir in /var/lib/tor-instances/"$instance"/keys/secret_onion_key{,_ntor}.old; do echo >"$dir/README" "This directory exists to prevent onion key rotation. See https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Survival-Guides/Snowflake-Bridge-Installation-Guide"; done; done
# for instance in $INSTANCES; do chmod -v -w /var/lib/tor-instances/"$instance"/keys/secret_onion_key{,_ntor}; done
```

Then start all the instances of tor. Check /var/log/syslog for error messages.

```
# systemctl restart tor
# etckeeper commit "tor snowflake instances"
```

You can verify that all instances have the same identity key with:

```
# cat /var/lib/tor-instances/*/fingerprint
```


## HAProxy

Install HAProxy, which is a load balancer. Configure it to listen at 127.0.0.1:10000, and forward to the instances of extor-static-cookie at 127.0.3.<var>N</var>:10000, using respective source addresses 127.0.2.<var>N</var>.

Append the configuration to the end of what is present by default in haproxy.cfg.

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
		server snowflake1 127.0.3.1:10000 source 127.0.2.1
		server snowflake1 127.0.3.2:10000 source 127.0.2.2
		server snowflake1 127.0.3.3:10000 source 127.0.2.3
		server snowflake1 127.0.3.4:10000 source 127.0.2.4
# systemctl restart haproxy
# etckeeper commit "HAProxy"
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
	LimitNOFILE=1048576

	Environment=TOR_PT_MANAGED_TRANSPORT_VER=1
	Environment=TOR_PT_SERVER_TRANSPORTS=snowflake
	Environment=TOR_PT_SERVER_BINDADDR=snowflake-[::]:443
	Environment=TOR_PT_EXTENDED_SERVER_PORT=127.0.0.1:10000
	Environment=TOR_PT_AUTH_COOKIE_FILE=/etc/extor-static-cookie/static_extended_orport_auth_cookie
	Environment=TOR_PT_SERVER_TRANSPORT_OPTIONS=snowflake:orport-srcaddr=127.0.1.0/24;snowflake:num-turbotunnel=8
	Environment=TOR_PT_STATE_LOCATION=%S/snowflake-server/pt_state
	Environment=TOR_PT_EXIT_ON_STDIN_CLOSE=0

	ExecStart=/usr/local/bin/snowflake-server --acme-hostnames <mark><var>NN</var></mark>.snowflake.torproject.net --acme-email dcf@torproject.org --log %L/snowflake-server/snowflake-server.log

	[Install]
	WantedBy=multi-user.target
# systemctl enable --now snowflake-server
# etckeeper commit "snowflake-server service"
```

Check for errors in `service snowflake-server status` and /var/log/snowflake-server/snowflake-server.log.

snowflake-server will automatically acquire a TLS certificate
for the names given in `--acme-hostnames` the first time each name is accessed.
If you use a subdomain of torproject.net,
then you will need to get in touch with the [Tor sysadmin team](https://gitlab.torproject.org/tpo/tpa/team)
and ask to have a [CAA DNS record](https://gitlab.torproject.org/tpo/tpa/team/-/wikis/howto/tls#certificate-authority-authorization-caa) created
that authorizes a certain Let's Encrypt account
to get certificates for that domain.
See tpo/tpa/team#41462.
You can use the [autocert-account-id](https://gitlab.torproject.org/dcf/autocert-account-id)
program to find the name of the account created in the
/var/lib/snowflake-server/pt_state/snowflake-certificate-cache directory.
(This step has not yet been done an a new bridge as of 2024-01-11.
Update these instructions when it happens.)


## Backups

These are the files you need to make backup copies of, in order to be able to restore
a compatibly working bridge.
(See https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/issues/40495#note_3289442.)

Of these, the relay identity private keys are critical!
Relay identity fingerprints are hard-coded in many clients and it would be hard to change them.

* Relay identity private keys
  * keys/ed25519_master_id_secret_key
  * keys/secret_id_key
* Relay [onion keys](https://www.bamsoftware.com/papers/pt-bridge-hiperf/#sec:onion-key)
  * keys/secret_onion_key
  * keys/secret_onion_key_ntor
* Let's Encrypt ACME account private key (see tpo/tpa/team#41462, [autocert-account-id](https://gitlab.torproject.org/dcf/autocert-account-id))
  * /var/lib/snowflake-server/pt_state/snowflake-certificate-cache/acme_account+key
* SSH host private keys
  * /etc/ssh/ssh_host_ecdsa_key
  * /etc/ssh/ssh_host_ed25519_key
  * /etc/ssh/ssh_host_rsa_key
* [WireGuard](#appendix-wireguard) private key
  * wg0.private.key

## Appendix: Outbound bind addresses

The [snowflake-01](Survival-Guides/Snowflake-Bridge-Survival-Guide#snowflake-01-flakey) bridge uses multiple outgoing IP addresses, in an effort to appear less like a participant in the [DDoS attack](https://status.torproject.org/issues/2022-06-09-network-ddos/) that was current in 2022. See tpo/anti-censorship/pluggable-transports/snowflake#40223.

To make this work, we set `OutboundBindAddress` to different values in different instances' torrc files:

<pre>
OutboundBindAddress 193.187.88.<var>X</var>
OutboundBindAddress [2a0c:dd40:1:b::<var>X</var>]
</pre>

Besides setting multiple outbound addresses, we also communicate with the authors of DDoS mitigation scripts to exempt the Snowflake bridges' addresses:

* [toralf/torutils](https://github.com/toralf/torutils/issues/7)
* [Enkidu-6/tor-ddos](https://github.com/Enkidu-6/tor-ddos/issues/25)
* [steinex/tor-ddos](https://github.com/steinex/tor-ddos/issues/2)


## Appendix: WireGuard

The snowflake-01 and snowflake-02 bridges
use WireGuard before the SSH port.
See the [survival guide](Survival-Guides/Snowflake-Bridge-Survival-Guide#bridge-sites)
for instructions on setting up WireGuard.
