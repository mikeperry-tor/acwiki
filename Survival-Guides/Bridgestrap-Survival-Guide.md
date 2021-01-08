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
* Take a look at [bridgestrap's metrics](https://bridges.torproject.org/bridgestrap-metrics) for a quick check if the service is running.

(Re)starting bridgestrap
------------------------

1. Log into bridges.torproject.org.
2. Change to the bridgestrap user by running `sudo -u bridgestrap -s`.
3. (Re)start the bridgestrap process via its systemd script: `systemctl --user [start|stop|status] bridgestrap`.
4. Take a look at bridgestrap's log file at /home/bridgestrap/logs/bridgestrap.log to make sure that the service (re)started successfully.