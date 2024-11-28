These are instructions for setting up a Snowflake broker on Debian 12.

#### Debian Base System Setup

Set up APT and etckeeper.
Install etckeeper.
<pre>
root# vi /etc/apt/sources.list # remove "contrib" and "non-free"
root# apt update
root# apt upgrade
root# apt install etckeeper
</pre>

Add a normal user.
<pre>
root# adduser <var>user</var>
root# adduser <var>user</var> sudo
root# su - <var>user</var>
user$ mkdir -m 700 .ssh
user$ echo "ssh-ed25519 ..." &gt;&gt; .ssh/authorized_keys
user$ exit
root# exit
</pre>

Log in as the normal user and disable login for root.
<pre>
user$ sudo -s
root# vi /etc/ssh/sshd_config
        PermitRootLogin no
        AllowUsers <var>user</var>
        PasswordAuthentication no
root# rm /root/.ssh/authorized_keys 
root# service sshd restart
root# etckeeper commit "Disable root ssh."
</pre>

Add other users.
<pre>
root# adduser <var>user1</var>
root# adduser <var>user1</var> sudo
root# adduser <var>user2</var>
root# adduser <var>user2</var> sudo
root# adduser <var>user3</var>
root# adduser <var>user3</var> sudo
root# vi /etc/ssh/sshd_config
        AllowUsers <var>user</var> <var>user1</var> <var>user2</var> <var>user3</var>
root# service sshd restart
root# etckeeper commit "Add users."
</pre>

Set a password for root for console recovery
```
root# passwd # Set a random password
```
This root password cannot be used for SSH login, instead, it is created to allow recovery from system that cannot finish boot to muti-user run level.

#### Setup IPv6

IPv6 Address for the VPS we are using can be chosen by customer. It sounds scary but we have the CAA record to reject request from unknown accounts.

To generate a random postfix for the given address range:
```
python3 -c 'import os; print(":".join(os.urandom(2).hex() for _ in range(3)))'
```

And then add generated IP address to system configuration `/etc/network/interfaces`:
```
iface eth0 inet6 static
    address 2a00:c6c0:0:151:4:ae99:c0a9:d585
    netmask 64
    gateway 2a00:c6c0:0:151::1
```

It may takes a restart to apply the change.

#### probetest(NAT Test Assist Tool) Installation
This `probetest` tool is installed and setup first as it is more likely to cause a system failure that requires reinstallation than others. Plus, its setup is coupled with the setup of firewalls.

Create `/var/lib/probenattest/init-netns.sh` file, with instruction to setup network namespace for probetest.
```
#!/bin/bash
ip netns add net0
ip link add veth-a type veth peer name veth-b
ip link set veth-a netns net0
ip netns exec net0 ip link set lo up
ip netns exec net0 ip address add 10.0.0.2/24 dev veth-a
ip netns exec net0 ip address add fc00::2/7 dev veth-a
ip netns exec net0 ip link set veth-a up
ip address add 10.0.0.1/24 dev veth-b
ip address add fc00::1/7 dev veth-b
ip link set veth-b up
ip netns exec net0 ip route add default via 10.0.0.1 dev veth-a
ip netns exec net0 ip route add default via fc00::1 dev veth-a
mkdir -p /etc/netns/net0/
ln -sf /etc/resolv.conf /etc/netns/net0/
ln -sf /etc/hosts /etc/netns/net0/
echo 1 > /proc/sys/net/ipv4/ip_forward
sysctl -w net.ipv6.conf.all.forwarding=1
```

Create `/etc/systemd/system/probeNatTestSetup.service` file to run the script above at system startup.
```
[Unit]
Description=Probe NAT Test Setup
Before=ferm.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=%S/probenattest/init-netns.sh

[Install]
WantedBy=default.target
WantedBy=ferm.service
```

Disable whatever firewall that are already installed and install ferm instead:
```
apt install ferm
apt remove iptables-persistent
```

And modify the ferm configuration file `/etc/ferm/ferm.conf` to
```
    # static public-facing ip addresses
    @def $IPv4_WORLD = 37.218.242.175;
    @def $IPv6_WORLD = 2a00:c6c0:0:151:4:ae99:c0a9:d585;

    # static private ip address
    @def $IPv4_PRIVATE = 10.0.0.2;
    @def $IPv6_PRIVATE = fc00::2;

	domain (ip ip6) {
	    table filter {
		chain INPUT {
                policy DROP;

                # connection tracking
                mod state state INVALID DROP;
                mod state state (ESTABLISHED RELATED) ACCEPT;

                # allow local packet
                interface lo ACCEPT;

                # respond to ping
                proto icmp ACCEPT; 

                # allow SSH connections
                proto tcp dport ssh ACCEPT;
		
	        # allow HTTP connections (for ACME HTTP-01 challenge)
	        proto tcp dport http ACCEPT;
                proto tcp dport https ACCEPT;


                # allow HTTPS-ALT connections
                proto tcp dport 8443 ACCEPT;
		}
                
                chain OUTPUT {
                policy ACCEPT;

                # connection tracking
                #mod state state INVALID DROP;
                mod state state (ESTABLISHED RELATED) ACCEPT;
                }
                chain FORWARD {
                policy DROP;

                # connection tracking
                mod state state INVALID DROP;
                mod state state (ESTABLISHED RELATED) ACCEPT;

                # forward packets to subnet
                @if @eq($DOMAIN, ip) {
                  daddr $IPv4_PRIVATE ACCEPT;
                  saddr $IPv4_PRIVATE ACCEPT;
                } @else {
                  daddr $IPv6_PRIVATE ACCEPT;
                  saddr $IPv6_PRIVATE ACCEPT;
                }
                }
            }
            # PRE- and POST- ROUTING rules for probetest
            table nat {
                chain POSTROUTING {
                @if @eq($DOMAIN, ip) {
                  saddr "$IPv4_PRIVATE/24" outerface eth0 SNAT to $IPv4_WORLD random;
                } @else {
                  saddr "$IPv6_PRIVATE/7" outerface eth0 SNAT to $IPv6_WORLD random;
                }
                }
                chain PREROUTING {
                }
            }
	}

``` 
It might be necessary to replace `$IPv4_WORLD` and `$IPv6_WORLD` to the approprate value if necessary.

Finally enable the firewall. Be sure to check that recovery option is present before go ahead with enable it.
```
systemctl start ferm.service
# Test everything still works
systemctl enable ferm.service
```

Tip: to inspect the rules derived from the ferm configuration file above use `iptables-legacy -vL $tableName`

Build and then copy the binary for [probetest](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/tree/main/probetest) to `/var/lib/probenattestd/probenattestd`.

Allow probetest binary to be executed:
```
chmod +x /var/lib/probenattestd/probenattestd
```

Then create `/etc/systemd/system/probeNatTestd.service` file, to run `probetest` as a system unit:
```
[Unit]
Description=Snowflake Nat Type Test Daemon
After=probeNatTestSetup.service
    
[Service]

# --addr 10.0.0.2:8081 is added to ensure should NetworkNamespacePath failed to apply the unit will fail. Do NOT remove it unless the the reason to add it is understood.
ExecStart=%S/probenattestd/probenattestd -disable-tls --addr 10.0.0.2:8081
WorkingDirectory=%S/probenattestd
RestartSec=5s
Restart=on-failure

StateDirectory=probenattestd
NetworkNamespacePath=/var/run/netns/net0
    
[Install]
WantedBy=default.target
```
and then enable and start it with `systemctl enable probeNatTestd.service` and `systemctl start probeNatTestd.service`.

**WARNING**: in systemd, the failure to apply `NetworkNamespacePath=/var/run/netns/net0` is silent, and the service will run anyway without it being applied if systemd was unable to apply it without even a line of warning. This behaviour is against the best practice, and should be aware of when operating it. `--addr 10.0.0.2:8081` is there to make sure the service will fail and make a noise when the `NetworkNamespacePath` is not applied, be considerate of this when changing it.

#### ACME(Automatic Certificate Management Environment) Setup

Install and Configure certbot
```
# This is not supported by certbot developer
apt install certbot
# Replace the email here with a real email
certbot register -m XXXXXXX@XXXXXX.org

# Maybe replace this domain with the production domain name if necessary
certbot certonly -d 0tzfb4f02pigk12zhf4ebovqvzl8abcq2ckd-37-218-242-175.sslip.io
```

Then look for the following output:
```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/0tzfb4f02pigk12zhf4ebovqvzl8abcq2ckd-37-218-242-175.sslip.io/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/0tzfb4f02pigk12zhf4ebovqvzl8abcq2ckd-37-218-242-175.sslip.io/privkey.pem
```

Remember the path mentioned above as they will be used in the nginx setup.

To check if automated certificate renewal is setup successfully:
```
systemctl list-timers
# and then look for certbot.timer
certbot renew --dry-run
# and inspect the output
```

#### Setup Broker

Setup auto pdated geoip files
```
# From tor's official setup guide https://support.torproject.org/apt/
curl https://deb.torproject.org/torproject.org/A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89.asc | gpg --dearmor | tee /usr/share/keyrings/deb.torproject.org-keyring.gpg >/dev/null
apt install tor-geoipdb deb.torproject.org-keyring

# Although we only selected tor-geoipdb, a tor daemon is also installed at the same time. Let's disable it 
systemctl disable tor.service
systemctl mask tor.service
```

Then find out location for these geoip files:
```
dpkg -S geoip
tor-geoipdb: /usr/share/tor/geoip6
tor-geoipdb: /usr/share/tor/geoip
```

Setup broker service
```
# Create a user just for webapps
useradd -m webapp

# install additional packages required for the environment; if a different slim installion was used, it may require different set of packages to be installed manually. Please do not lost hope when systemd failed in wired way and the error message yield no result with search engine
apt install systemd-container libpam-systemd

# sudo is not sufficient here as we are interacting systemd and there is many environment varibles needs to be set correctly
machinectl shell --uid=webapp

# Copy binaries to .config/broker

# Copy broker.service to .config/systemd/user

# Copy data files to .config/broker

systemctl enable --user broker.service
systemctl start --user broker.service
```

The broker.service shown above:
```
[Unit]
Description=Snowflake Broker

    
[Service]

ExecStart=%S/broker/broker --metrics-log metrics.log --bridge-list-path bridge_list.json --default-relay-pattern ^snowflake.torproject.net$ --allowed-relay-pattern snowflake.torproject.net$ --disable-tls --geoipdb /usr/share/tor/geoip --geoip6db /usr/share/tor/geoip6 --addr 127.0.0.1:8080
WorkingDirectory=%S/broker
RestartSec=5s
Restart=on-failure

StateDirectory=broker
    
[Install]
WantedBy=default.target
```

#### Setup nginx

Install nginx and stream module with package manager
```
# Firstly, install nginx with stream plugin
apt install nginx libnginx-mod-stream
```

Setup dh parameters, as advised by https://ssl-config.mozilla.org/
```
curl https://ssl-config.mozilla.org/ffdhe2048.txt > /etc/nginx/ffdhe2048.txt
```

Firstly, setup alpn based ssl forwarding by
creating `/etc/nginx/modules-enabled/99-000-tlsacmeapln.conf`:

```
stream {    
    map $ssl_preread_alpn_protocols $tls_address {
      ~\bacme-tls/1\b 127.0.0.1:10443;
      default 127.0.0.1:2443;
    }

    server {
      listen 443;
      listen [::]:443;
      listen 8443;
      listen [::]:8443;
      proxy_pass $tls_address;
      ssl_preread on;
      proxy_protocol on;
    }
    
    server {
      listen 127.0.0.1:10443 proxy_protocol;
      proxy_pass 127.0.0.1:11443;
      ssl_preread on;
      proxy_protocol off;
    }
}
```
WARNING: this setup is used to work with tls-alpn-01 domain validiation, which is currently not supported by certbot. Currently HTTP based validation was used.

Setup rate limit configuration by
creating `/etc/nginx/conf.d/rate_limit_zone.conf`
```
limit_req_zone $proxy_protocol_addr zone=snowflake:10m rate=1r/s;
```

Setup HTTP site entry point 
by creating `/etc/nginx/sites-enabled/https-site`:
```
server {
    listen 127.0.0.1:2443 ssl http2 proxy_protocol;

    ssl_certificate /etc/letsencrypt/live/0tzfb4f02pigk12zhf4ebovqvzl8abcq2ckd-37-218-242-175.sslip.io/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/0tzfb4f02pigk12zhf4ebovqvzl8abcq2ckd-37-218-242-175.sslip.io/privkey.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
    ssl_session_tickets off;

    # curl https://ssl-config.mozilla.org/ffdhe2048.txt > /path/to/dhparam
    ssl_dhparam /etc/nginx/ffdhe2048.txt;

    # intermediate configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers off;

    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    add_header Strict-Transport-Security "max-age=63072000" always;

    # OCSP stapling
    ssl_stapling on;
    
    #access_log /var/log/nginx/access.https.log combined_useproxy;
    access_log off;
    
    include sites-available/https/*.rconf;
    include sites-available/https/*.conf;
}
```

then setup `broker` request forwarding
by creating `/etc/nginx/sites-available/https/broker.conf`:
```
        location ~ ((proxy)|(client)|(answer)|(metrics)|(prometheus)|(amp/client/.*)|(robots.txt)) {
            limit_req zone=snowflake burst=3;
            proxy_pass http://127.0.0.1:8080;
            proxy_http_version 1.1;
            proxy_read_timeout 300;
            proxy_connect_timeout 300;
            proxy_send_timeout 300;

            proxy_set_header  X-Forwarded-For $proxy_protocol_addr;
        }
```

and `probetest` request forwarding 
by creating `/etc/nginx/sites-available/https/nattypetest.conf`:
```
        location ~ ((probe)) {
            limit_req zone=snowflake burst=3;
            proxy_pass http://10.0.0.2:8081;
            proxy_http_version 1.1;
        }
```

and finally edit nginx to disable logging
by replacing `/etc/nginx/nginx.conf` with:
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /dev/null;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##
        
	#access_log /var/log/nginx/access.log;
        access_log off;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```

Updated lines includes:
```
error_log /dev/null;
access_log off;
```

#### Misc

Install prometheus-node-exporter for resource monitoring (#29863).
<pre>
root# apt install prometheus-node-exporter
root# nano /etc/default/prometheus-node-exporter
	ARGS="--no-collector.arp --no-collector.bcache --no-collector.bonding --no-collector.conntrack --no-collector.cpu --no-collector.edac --no-collector.entropy --no-collector.filefd --no-collector.hwmon --no-collector.infiniband --no-collector.ipvs --no-collector.loadavg --no-collector.mdadm --no-collector.meminfo --no-collector.netclass --no-collector.netdev --no-collector.netstat --no-collector.nfs --no-collector.nfsd --no-collector.sockstat --no-collector.stat --no-collector.textfile --no-collector.timex --no-collector.uname --no-collector.vmstat --no-collector.xfs --no-collector.zfs --web.listen-address=127.0.0.1:9100 --web.telemetry-path=\"\/gxoulrp9jxlubkb5s0ekgt1nl5tbsw45rbk3\" "
root# systemctl status prometheus-node-exporter
root# etckeeper commit "Install prometheus-node-exporter."
</pre>

Create `/etc/nginx/sites-available/https/prometheus-node-exporter.conf`
```
        location ~ ((gxoulrp9jxlubkb5s0ekgt1nl5tbsw45rbk3)) {
            proxy_pass http://127.0.0.1:9100;
            proxy_http_version 1.1;
        }
```
and run
```
nginx -s reload
```

Do some other nice configuration.
<pre>
root# apt install unattended-upgrades man-db screen rsync
</pre>

If the IP has changed the prometheus server will keep pooling the old broker for metrics. Restarting the nginx of the old broker should force it to move to the new IP.