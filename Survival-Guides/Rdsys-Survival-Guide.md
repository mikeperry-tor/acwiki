General information
-------------------

* Rdsys consists of several microservices. This document only covers rdsys's backend process whose name is rdsys-backend. For brevity, the rest of this document refers to the backend process as rdsys.
* Scripts and config files related to rdsys's deployment are in the [rdsys-admin](https://gitlab.torproject.org/tpo/anti-censorship/rdsys-admin) repository.
* The service runs on bridges.torproject.org.
  - The username is rdsys.
  - Its home directory is in /home/rdsys.
  - It listens on 127.0.0.1:7100.
* Rdsys's systemd script is at /home/rdsys/.config/systemd/user/rdsys-backend.service.
* There's a crontab entry (run `crontab -e` as user rdsys) that invokes logrotate once a day to [rotate rdsys's log files](https://gitlab.torproject.org/tpo/anti-censorship/rdsys-admin/-/blob/master/logrotate/logrotate.conf).
* Take a look at [rdsys's metrics](https://bridges.torproject.org/rdsys-backend-metrics) for a quick check if the service is running.

(Re)starting rdsys
------------------

1. Log into bridges.torproject.org.
2. Change to the rdsys user by running `sudo -u rdsys -s`.
3. (Re)start the rdsys-backend process via its systemd script: `systemctl --user [start|stop|status] rdsys-backend`.
4. Take a look at rdsys's log file at /home/rdsys/logs/rdsys-backend.log to make sure that the service (re)started successfully.

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