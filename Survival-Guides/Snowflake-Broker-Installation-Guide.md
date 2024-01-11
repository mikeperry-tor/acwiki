These are instructions for setting up a Snowflake broker on Debian 10.

Set up APT and etckeeper.
Install etckeeper.
```
root# vi /etc/apt/sources.list # remove "contrib" and "non-free"
root# apt update
root# apt upgrade
root# apt install etckeeper
```

Add a normal user.
```
root# adduser user
root# adduser user sudo
root# su - user
user$ mkdir -m 700 .ssh
user$ echo "ssh-ed25519 ..." >> .ssh/authorized_keys
user$ exit
root# exit
```

Log in as the normal user and disable login for root.
```
user$ sudo -s
root# vi /etc/ssh/sshd_config
        PermitRootLogin no
        AllowUsers user
        PasswordAuthentication no
root# rm /root/.ssh/authorized_keys 
root# service sshd restart
```

Add other users.
```
root# adduser user1
root# adduser user1 sudo
root# adduser user2
root# adduser user2 sudo
root# adduser user3
root# adduser user3 sudo
root# vi /etc/ssh/sshd_config
        AllowUsers user user1 user2 user3
root# etckeeper commit "Add users."
```

Set up a firewall.
```
root# apt install ferm # Enable ferm on bootup? Yes
root# vi /etc/ferm/ferm.conf
	domain (ip ip6) {
	    table filter {
		chain INPUT {
		...
	        # allow HTTP connections (for ACME HTTP-01 challenge)
	        proto tcp dport http ACCEPT;

		# allow HTTPS connections
		proto tcp dport https ACCEPT;
		}
		...
	    }
	}
root# service ferm restart
root# etckeeper commit "Allow HTTP and HTTPS through the firewall."
```

Set up an IPv6 address. You can use any address in the 2a00:c6c0:0:154:4::/80 prefix.
```
root# python -c 'import os; print ":".join(os.urandom(2).encode("hex") for _ in range(3))'
d8aa:b4e6:c89f
root# vi /etc/network/interfaces
	iface eth0 inet6 static
		address 2a00:c6c0:0:154:4:d8aa:b4e6:c89f
		netmask 64
		gateway 2a00:c6c0:0:154::1
root# etckeeper commit "Add IPv6 address."
root# reboot
```

Install the broker.
```
root# install --owner root /home/user/broker /usr/local/bin/
root# apt install runit-systemd tor-geoipdb
root# echo "/runit/**/supervise" >> /etc/.gitignore
root# adduser --system snowflake-broker
root# mkdir -p /etc/runit/snowflake-broker
root# vi /etc/runit/snowflake-broker/run
	#!/bin/sh -e
	setcap 'cap_net_bind_service=+ep' /usr/local/bin/broker
	exec chpst -u snowflake-broker -o 32768 /usr/local/bin/broker --metrics-log /home/snowflake-broker/metrics.log --acme-hostnames snowflake-broker.bamsoftware.com,snowflake-broker.freehaven.net,snowflake-broker.torproject.net --acme-email dcf@torproject.org --acme-cert-cache /home/snowflake-broker/acme-cert-cache 2>&1
root# chmod +x /etc/runit/snowflake-broker/run
root# mkdir -p /etc/runit/snowflake-broker/log
root# vi /etc/runit/snowflake-broker/log/run
root# chmod +x /etc/runit/snowflake-broker/log/run
        #!/bin/sh -e
        svlogd /var/log/snowflake-broker
root# mkdir -p /var/log/snowflake-broker
root# vi /var/log/snowflake-broker/config
	# http://smarden.org/runit/svlogd.8.html
	# limit size to 10MB before rotation
	s10000000
	# don't delete old logs 
	n0
	# unless the filesystem is full
	N1
	# compress old logs
	!xz
root# ln -s /etc/runit/snowflake-broker /etc/service
root# etckeeper commit "Install snowflake-broker."
```

The broker will automatically acquire a TLS certificate
for the names given in `--acme-hostnames` the first time each name is accessed.
If you use a subdomain of torproject.net,
then you will need to get in touch with the [Tor sysadmin team](https://gitlab.torproject.org/tpo/tpa/team)
and ask to have a CAA DNS record created
that authorizes a certain Let's Encrypt account
to get certificates for that domain.
See tpo/tpa/team#41462.
You can use the [autocert-account-id](https://gitlab.torproject.org/dcf/autocert-account-id)
program to find the name of the account created in the
/home/snowflake-broker/acme-cert-cache directory.

Install prometheus-node-exporter for resource monitoring (#29863).
```
root# apt install prometheus-node-exporter
root# vi /etc/default/prometheus-node-exporter
	ARGS="--no-collector.arp --no-collector.bcache --no-collector.bonding --no-collector.conntrack --no-collector.cpu --no-collector.edac --no-collector.entropy --no-collector.filefd --no-collector.hwmon --no-collector.infiniband --no-collector.ipvs --no-collector.loadavg --no-collector.mdadm --no-collector.meminfo --no-collector.netclass --no-collector.netdev --no-collector.netstat --no-collector.nfs --no-collector.nfsd --no-collector.sockstat --no-collector.stat --no-collector.textfile --no-collector.timex --no-collector.uname --no-collector.vmstat --no-collector.xfs --no-collector.zfs"
root# service prometheus-node-exporter restart
root# etckeeper commit "Install prometheus-node-exporter."
```

Do some other nice configuration.
```
root# apt install unattended-upgrades man screen rsync
root# update-alternatives --config editor # Choose /usr/bin/vim.tiny
```