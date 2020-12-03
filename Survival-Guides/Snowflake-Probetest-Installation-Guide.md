These are instructions for setting up the Snowflake probe test on Debian 10.

1. Set up a network namespace for the probetest.
```
root# vi /init-netns.sh
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
 Ensure these steps are run at startup.
```
root# crontab -e
@reboot /root/init-netns.sh
```


2. Set up the firewall. Right now we run the probe test on `:8443`, the alternate HTTPS port. The probe test is run in its own network namespace, with special routing rules to give the namespace symmetric NAT behaviour.
```
root# apt install ferm # Enable ferm on bootup? Yes
root# vi /etc/ferm/ferm.conf
    # static public-facing ip addresses
    @def $IPv4_WORLD = 37.218.245.111;
    @def $IPv6_WORLD = 2a00:c6c0:0:154:4:d8aa:b4e6:c89f;

    # static private ip address
    @def $IPv4_PRIVATE = 10.0.0.2;
    @def $IPv6_PRIVATE = fc00::2;

	domain (ip ip6) {
	    table filter {
		chain INPUT {
		...
	        # allow HTTP connections (for ACME HTTP-01 challenge)
	        proto tcp dport http ACCEPT;


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
                @if @eq($DOMAIN, ip) {
                  proto tcp dport 8443 interface eth0 DNAT to $IPv4_PRIVATE;
                } @else {
                  proto tcp dport 8443 interface eth0 DNAT to $IPv6_PRIVATE;
                }
                }
            }
	}
root# service ferm restart
root# etckeeper commit "Allow HTTP and HTTPS through the firewall."

```

3. Install probetest and create runit scripts
```
root# install --owner root /home/user/probetest /usr/local/bin/probetest
root# apt install runit-systemd tor-geoipdb
root# echo "/runit/**/supervise" >> /etc/.gitignore
root# mkdir -p /etc/runit/snowflake-probetest
root# vi /etc/runit/snowflake-probetest/run
	#!/bin/sh -e
        export GOMAXPROCS=1
        setcap 'cap_net_bind_service=+ep' /usr/local/bin/probetest
        exec ip netns exec net0 /usr/local/bin/probetest --acme-hostnames snowflake-broker.bamsoftware.com,snowflake-broker.freehaven.net,snowflake-broker.torproject.net --acme-email dcf@torproject.org --acme-cert-cache /home/snowflake-broker/acme-cert-cache --addr :8443 2>&1/home/snowflake-broker/metrics.log --acme-hostnames snowflake-broker.bamsoftware.com,snowflake-broker.freehaven.net,snowflake-broker.torproject.net --acme-email dcf@torproject.org --acme-cert-cache /home/snowflake-broker/acme-cert-cache 2>&1
root# mkdir -p /etc/runit/snowflake-probetest/log
root# vi /etc/runit/snowflake-probetest/log/run
root# chmod +x /etc/runit/snowflake-probetest/log/run
        #!/bin/sh -e
        svlogd /var/log/snowflake-probetest
root# mkdir -p /var/log/snowflake-probetest
root# vi /var/log/snowflake-probetest/config
	# http://smarden.org/runit/svlogd.8.html
	# limit size to 10MB before rotation
	s10000000
	# don't delete old logs 
	n0
	# unless the filesystem is full
	N1
	# compress old logs
	!xz
root# ln -s /etc/runit/snowflake-probetest /etc/service
```