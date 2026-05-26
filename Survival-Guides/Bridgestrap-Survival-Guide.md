General information
-------------------

* Scripts and config files related to bridgestrap's deployment are in the [bridgestrap-admin](https://gitlab.torproject.org/tpo/anti-censorship/bridgestrap-admin) repository.
* The service runs on bridges.torproject.org.
  - The username is bridgestrap.
  - Its home directory is in /home/bridgestrap.
  - It listens on 127.0.0.1:5001.
* Bridgestrap invokes the statically-compiled tor process located at /home/bridgestrap/bin/tor-static. Take a look at [this comment](https://gitlab.torproject.org/tpo/anti-censorship/bridgestrap/-/issues/2#note_2710591) for why this is necessary.
* Bridgestrap's systemd script is at /home/bridgestrap/.config/systemd/user/bridgestrap.service.
* There's a crontab entry (run `crontab -e` as user bridgestrap) that invokes logrotate once a day to [rotate bridgestrap's log files](https://gitlab.torproject.org/tpo/anti-censorship/bridgestrap-admin/-/blob/master/logrotate/logrotate.conf).
* You can find the current tor log in `/tmp/tor-datadir-...`
* Take a look at [bridgestrap's metrics](https://bridges.torproject.org/bridgestrap-metrics) for a quick check if the service is running.
* Bridgestrap is currently deployed and can be accessed at https://bridges.torproject.org/status


(Re)starting bridgestrap
------------------------

1. Log into bridges.torproject.org.
2. Change to the bridgestrap user by running `sudo -u bridgestrap -s`.
3. (Re)start the bridgestrap process via its systemd script: `systemctl --user [start|stop|status] bridgestrap`.
4. Take a look at bridgestrap's log file at /home/bridgestrap/logs/bridgestrap.log to make sure that the service (re)started successfully.

Deploying a new version
-----------------------

The following script takes as argument a bridgestrap executable and it deploys it on bridges.torproject.org. Replace `POLYANTHUM` with how you log into bridges.torproject.org.
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
    "sudo -u bridgestrap bash -i -c '" \
      "systemctl --user stop bridgestrap && " \
      "cp /tmp/${executable} /home/bridgestrap/bin/bridgestrap && " \
      "systemctl --user start bridgestrap' && " \
    "rm -f /tmp/${executable}"
```

## Issues

### Bridges down

The percentage of functional bridges might be too low either from a restart or a an issue in bridgestrap or rdsys. Restarting rdsys and bridges usually solves the problem, but should be investigated if it happens more than once.

### Empty cache

If the tor process dies bridgestrap can't tests any more bridges and the cache becomes empty. Restarting bridgestrap solves the problem.

There is an issue were we are investigating the source of the problem: https://gitlab.torproject.org/tpo/anti-censorship/bridgestrap/-/work_items/49