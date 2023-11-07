[onbasca](https://gitlab.torproject.org/tpo/network-health/onbasca/) measures the bandwidth of the bridges and reports a bandwidth ratio that is [used by rdsys](https://gitlab.torproject.org/tpo/anti-censorship/rdsys/-/blob/main/doc/resource-testing.md) to decide if a bridge should be distributed or not.

Onbasca is installed in polyanthum with the *onbasca* user, with SSH fingerprints:

* `1024 SHA256:wDAIdh3zEKYYdQMLIHwR19spLPiKjVvRAmAoB6irnjs root@polyanthum (DSA)`
* `256 SHA256:ydK14UKPpV7sv5PdrLjczPSETUvXwb9gcYm+gZiYDzY root@polyanthum (ECDSA)`
* `256 SHA256:+ZMJU/Nidg9P89tiRWNEeSWa3wIpzzKCzOReAfzIrF0 root@polyanthum (ED25519)`
* `2048 SHA256:x8vgDvdkeSuz8Wa9idjFCR8mT61DGe8SVDYsUi0BpCQ root@polyanthum (RSA)`

The installation instructions of onbasca are available here:  
https://tpo.pages.torproject.net/network-health/onbasca/INSTALL.html

## Where are the logs?

* The bridgescan does write logs on the bridges it scanns into `~/.onbrisca/logs/scan.log`
* The API service does write logs into `~/.onbrisca/debug.log`
* gunicorn writes access and error logs into `~/log`
* The output of the two services can be found in journald: `journald --user`

## Where are the configuration files?

* `~/.onbrisca/config.toml` is the main configuration file, it contains details on external services onbasca uses
* `~/onbasca/onbriscapr/settings/local.py` contains the configuration of the database, changing this file requires re-install onbasca (see *Updating to a new version*)

## (Re)starting onbasca

1. Log into polyanthum machine.
1. Change to the onbasca user by running `sudo -u onbasca -s`
1. There are two systemd services used by onbasca:
   * bridgescan, that tests the bridges: `systemctl --user [start|stop|status] bridgescan`.
   * gunicorn, that exposes the API used by rdsys: `systemctl --user [start|stop|status] gunicorn`.

## Updating to a new version

1. Log into polyanthum machine.
1. Change to the onbasca user by running `sudo -u onbasca -s`
1. Update the repo `cd ~/onbasca && git pull`
1. Enable the virtualenv: `source ~/env/bin/activate`
1. Install the latest version of onbasca `cd ~/onbasca && python setup.py install`
1. Restart onionsprouts bot `systemctl --user restart bridgescan && systemctl --user restart gunicorn`