These are instructions for setting up the Snowflake probe test on Debian 10.

1. Set up the firewall. Right now we run the probe test on `:8443`, the alternate HTTPS port. The probe test is run in its own network namespace, with special routing rules to give the namespace symmetric NAT behaviour.
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
