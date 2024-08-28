These are instructions for setting up a Snowflake broker on Debian 10.

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
root# etckeeper commit "Add users."
</pre>

Set up a firewall.
<pre>
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
</pre>

Set up an IPv6 address. You can use any address in the 2a00:c6c0:0:154:4::/80 prefix.
<pre>
root# python3 -c 'import os; print(":".join(os.urandom(2).hex() for _ in range(3)))'
d8aa:b4e6:c89f
root# vi /etc/network/interfaces
	iface eth0 inet6 static
		address 2a00:c6c0:0:154:4:d8aa:b4e6:c89f
		netmask 64
		gateway 2a00:c6c0:0:154::1
root# etckeeper commit "Add IPv6 address."
root# reboot
</pre>

Install the broker.
<pre>
root# install --owner root /home/<var>user</var>/broker /usr/local/bin/
root# apt install runit-systemd tor-geoipdb
root# echo "/runit/**/supervise" &gt;&gt; /etc/.gitignore
root# adduser --system snowflake-broker
root# mkdir -p /etc/runit/snowflake-broker
root# vi /etc/runit/snowflake-broker/run
	#!/bin/sh -e
	setcap 'cap_net_bind_service=+ep' /usr/local/bin/broker
	exec chpst -u snowflake-broker -o 32768 /usr/local/bin/broker --metrics-log /home/snowflake-broker/metrics.log --acme-hostnames snowflake-broker.bamsoftware.com,snowflake-broker.freehaven.net,snowflake-broker.torproject.net --acme-email dcf@torproject.org --acme-cert-cache /home/snowflake-broker/acme-cert-cache 2&gt;&amp;1
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
</pre>

The broker will automatically acquire a TLS certificate
for the names given in `--acme-hostnames` the first time each name is accessed.
If you use a subdomain of torproject.net,
then you will need to get in touch with the [Tor sysadmin team](https://gitlab.torproject.org/tpo/tpa/team)
and ask to have a [CAA DNS record](https://gitlab.torproject.org/tpo/tpa/team/-/wikis/howto/tls#certificate-authority-authorization-caa) created
that authorizes a certain Let's Encrypt account
to get certificates for that domain.
See tpo/tpa/team#41462.
You can use the [autocert-account-id](https://gitlab.torproject.org/dcf/autocert-account-id)
program to find the name of the account created in the
/home/snowflake-broker/acme-cert-cache directory.

Install prometheus-node-exporter for resource monitoring (#29863).
<pre>
root# apt install prometheus-node-exporter
root# vi /etc/default/prometheus-node-exporter
	ARGS="--no-collector.arp --no-collector.bcache --no-collector.bonding --no-collector.conntrack --no-collector.cpu --no-collector.edac --no-collector.entropy --no-collector.filefd --no-collector.hwmon --no-collector.infiniband --no-collector.ipvs --no-collector.loadavg --no-collector.mdadm --no-collector.meminfo --no-collector.netclass --no-collector.netdev --no-collector.netstat --no-collector.nfs --no-collector.nfsd --no-collector.sockstat --no-collector.stat --no-collector.textfile --no-collector.timex --no-collector.uname --no-collector.vmstat --no-collector.xfs --no-collector.zfs"
root# service prometheus-node-exporter restart
root# etckeeper commit "Install prometheus-node-exporter."
</pre>

Do some other nice configuration.
<pre>
root# apt install unattended-upgrades man screen rsync
root# update-alternatives --config editor # Choose /usr/bin/vim.tiny
</pre>