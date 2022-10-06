General information
-------------------

* Rdsys consists of several microservices. This document only covers rdsys's backend process whose name is rdsys-backend. For brevity, the rest of this document refers to the backend process as rdsys.
* Scripts and config files related to rdsys's deployment are in the [rdsys-admin](https://gitlab.torproject.org/tpo/anti-censorship/rdsys-admin) repository.
* Rdsys's systemd scripts are in rdsys-admin git repo.
* There's a crontab entry (run `crontab -e` as user rdsys in bridges.torproject.org) that invokes logrotate once a day to [rotate rdsys's log files](https://gitlab.torproject.org/tpo/anti-censorship/rdsys-admin/-/blob/master/logrotate/logrotate.conf).
* Take a look at [rdsys's metrics](https://bridges.torproject.org/rdsys-backend-metrics) for a quick check if the service is running.

There are two servers where rdsys services live in:
* bridges.torproject.org
  - Where the backend runs
  - moat and telegram distributors runs there until we migrate them to rdsys-frontend.
  - The username is rdsys.
  - Its home directory is in /home/rdsys.
  - The backend listens on 127.0.0.1:7100.
* The rest of the distributors and updaters runs on rdsys-frontend-01.torproject.org.
  - The user rdsys owns the binaries in /srv/rdsys.torproject.org/bin
  - rdsys user has a clone of rdsys-admin repo in /srv/rdsys.torproject.org/rdsys-admin, all the other users symlink systemd services from this repo
  - Each service has a user and a folder in /srv/, for example `gettor` has /srv/gettor.torproject.org/conf where the config of the service lives.
  - The services run under the user with the name of the service. For example `gettor` has two systemd services `gettor-distributor` and `gettor-updater`

(Re)starting rdsys backend
--------------------------

1. Log into bridges.torproject.org.
2. Change to the rdsys user by running `sudo -u rdsys -s`.
3. Update the rdsys-admin repo if needed `cd ~/rdsys-admin; git pull`
4. (Re)start the rdsys-backend process via its systemd script: `systemctl --user [start|stop|status] rdsys-backend`.
5. Take a look at rdsys's log file at /home/rdsys/logs/rdsys-backend.log to make sure that the service (re)started successfully.

(Re)starting rdsys frontends
----------------------------

1. Log into rdsys-frontend-01.torproject.org.
2. Change to the rdsys user by running `sudo -u rdsys -s`.
3. Update the rdsys-admin repo if needed `cd ~/rdsys-admin; git pull`
4. Change to the service user by running `sudo -u gettor -s`.
5. (Re)start the `<serive>-distributor` or `<service>-updater` process via its systemd script: `systemctl --user [start|stop|status] gettor-distributor`.
5. Take a look at the logs with `journal --user -f`.

Deploying a new version
-----------------------

The following script takes as argument an rdsys-backend executable and it deploys it on bridges.torproject.org. Replace `POLYANTHUM` with how you log into bridges.torproject.org.
```bash
#!/bin/bash

if [ "$#" -ne 1 ]; then
    echo "Usage: $0 EXECUTABLE"
    exit 1
fi
path="$1"
executable=$(basename "$path")

scp "$path" POLYANTHUM:/tmp
ssh -t POLYANTHUM \
    "chmod 777 /tmp/${executable} && " \
    "sudo -u rdsys bash -i -c '" \
      "systemctl --user stop rdsys-backend && " \
      "cp /tmp/${executable} /home/rdsys/bin/rdsys-backend && " \
      "systemctl --user start rdsys-backend' && " \
    "rm -f /tmp/${executable}"
```