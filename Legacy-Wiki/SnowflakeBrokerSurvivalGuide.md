## Broker survival guide

IP addresses
```
37.218.245.111
2a00:c6c0:0:154:4:d8aa:b4e6:c89f
```

SSH fingerprints
```
RSA:    2048 SHA256:dp0Xo/oN1qZfMuZnqgKEbeOsbU2qpDR60B5MLIRaAgg
DSA:    1024 SHA256:DF5ofogjGur02gv8/ciU3wFA+YHNuAhUlel9Uv2KBlo
ECDSA:   256 SHA256:6cskO6ch/kv2RbIMhTdwqpsd9vB8npzlZTlkWZJLoek
ED25519: 256 SHA256:fEkLvmu5woTvU6162I8Jxd2eHzsTnBshumtWICclWA4
```

The broker is managed by runit. It's the only service running on the
host. To upgrade the broker:
1. `sv stop snowflake-broker`
2. `install --owner root ~/new-broker /usr/local/bin/broker`
3. `sv start snowflake-broker`
Logs are under `/var/log/snowflake-broker`.

Firewall configuration is in `/etc/ferm/ferm.conf`. Run
`service ferm restart` after making changes.