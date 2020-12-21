## BridgeDB survival guide

SSH fingerprints:
* `1024 SHA256:wDAIdh3zEKYYdQMLIHwR19spLPiKjVvRAmAoB6irnjs root@polyanthum (DSA)`
* `256 SHA256:ydK14UKPpV7sv5PdrLjczPSETUvXwb9gcYm+gZiYDzY root@polyanthum (ECDSA)`
* `256 SHA256:+ZMJU/Nidg9P89tiRWNEeSWa3wIpzzKCzOReAfzIrF0 root@polyanthum (ED25519)`
* `2048 SHA256:x8vgDvdkeSuz8Wa9idjFCR8mT61DGe8SVDYsUi0BpCQ root@polyanthum (RSA)`

## (Re)starting BridgeDB

1. Log into BridgeDB's machine.
2. Change to the BridgeDB user by running `sudo -u bridgedb -s`
3. Kill the existing BridgeDB process by running `kill $(pgrep -f /home/bridgedb/virtualenvs/bridgedb/bin/bridgedb)`
3. Start BridgeDB by running `~/bridgedb-admin/bin/run-bridgedb` which should result in the following output: `Starting BridgeDB...		[OK]`.

## Inspecting log files

BridgeDB's log file is located at `/srv/bridges.torproject.org/log/bridgedb.log`

## Changing BridgeDB's configuration file

BridgeDB's configuration file (and admin scripts) is maintained in the [bridgedb-admin](https://gitweb.torproject.org/project/bridges/bridgedb-admin.git/) repository. On polyanthum, BridgeDB's host, this repository is located at `/srv/bridges.torproject.org/home/bridgedb-admin`. and the configuration file is located at `/srv/bridges.torproject.org/home/bridgedb-admin/etc/bridgedb.conf`.

## (Re)starting moat
Moat is implemented by the binary `meek-server` located in `/srv/bridges.torproject.org/bin/`. The source code for this binary is available in the `meek-server` directory of the [meek repository](https://gitweb.torproject.org/pluggable-transports/meek.git/). To restart the binary, kill the existing process and then run the script `run-meek` located in `/srv/bridges.torproject.org/bin/`. To deploy a new binary, build it on your local machine and scp it onto polyanthum.

## (Re)starting wolpertinger
The wolpertinger binary is also located in `/srv/bridges.torproject.org/bin/`. Its source code is available [here](https://gitlab.torproject.org/torproject/anti-censorship/wolpertinger). To restart wolpertinger, kill the existing process and then run the script `run-wolpertinger` located in `/srv/bridges.torproject.org/bin/`. To deploy a new binary, build it on your local machine and scp it onto polyanthum.