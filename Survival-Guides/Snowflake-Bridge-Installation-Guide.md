Instructions for setting up a Snowflake bridge on Debian 11.

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
	            # allow Snowflake ACME HTTP-01
	            proto tcp dport http ACCEPT;
	            # allow Snowflake WebSocket
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

Build snowflake-server. You can compile it locally, then copy it to the bridge server. `CGO_ENABLED=0` results in a static binary that does not depend on a specific version of libc.

```
$ git clone https://git.torproject.org/pluggable-transports/snowflake.git
$ cd snowflake/server
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build
$ scp server snowflake:snowflake-server
```

Then, on the bridge, install snowflake-server and give it extra permission to bind to low-numbered ports. See tpo/core/tor#18356 for background.

```
# install --owner root /home/user/snowflake-server /usr/local/bin/snowflake-server
# setcap 'cap_net_bind_service=+ep' /usr/local/bin/snowflake-server
```

Install tor.

```
# apt install tor
```

Add configuration to /etc/tor/torrc. The fact that the firewall does not export the ORPort, and the presence of `BridgeDistribution none`, mean that BridgeDB will not try to distribute the bridge.

```
Nickname flakey
ContactInfo Tor Anti-Censorship Team <anti-censorship-team@lists.torproject.org>
SocksPort 0
ORPort 9001 
AssumeReachable 1
BridgeRelay 1
BridgeDistribution none
ExtORPort auto
# setcap 'cap_net_bind_service=+ep' /usr/local/bin/snowflake-server
ServerTransportPlugin snowflake exec /usr/local/bin/snowflake-server --acme-hostnames snowflake.bamsoftware.com,snowflake.freehaven.net,snowflake.torproject.net --acme-email dcf@torproject.org --log /var/log/tor/snowflake-server.log
ServerTransportListenAddr snowflake [::]:443
```

Override the default systemd settings to permit snowflake-server exercise its capability to bind to low-numbered ports. See tpo/core/tor#18356 for background.

```
# systemctl edit tor@default.service
	[Service]
	NoNewPrivileges=no
# etckeeper commit "tor snowflake-server configuration"
# service tor restart
```

Check /var/log/syslog and /var/log/tor/snowflake-server.log for error messages. For `bind: permission denied` errors, double check the `setcap` and `NoNewPrivileges=no` steps.

Not described here: [standalone proxies](Survival Guides/Snowflake Bridge Survival Guide#standalone-proxy-go-instances), which have historically run on the same host as the bridge.
