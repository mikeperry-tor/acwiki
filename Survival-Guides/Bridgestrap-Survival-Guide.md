General information
-------------------

* Scripts and config files related to bridgestrap's deployment are in the [bridgestrap-admin](https://gitlab.torproject.org/tpo/anti-censorship/bridgestrap-admin) repository.
* The service runs under the username bridgestrap and its home directory is at /home/bridgestrap.
* Bridgestrap listens on 127.0.0.1:5001.
* Bridgestrap invokes the tor process located at /home/bridgestrap/bin/tor-static. Take a look at [this comment](https://gitlab.torproject.org/tpo/anti-censorship/bridgestrap/-/issues/2#note_2710591) for why this is necessary.
* Bridgestrap's systemd script is at /home/bridgestrap/.config/systemd/user/bridgestrap.service.
* There's a crontab entry (run `crontab -e` as user bridgestrap) that invokes logrotate once a day.

(Re)starting bridgestrap
------------------------

1. Log into polyanthum.
2. Change to the bridgestrap user by running `sudo -u bridgestrap -s`.
3. (Re)start the bridgestrap process via its systemd script: `systemctl --user [start|stop|status] bridgestrap`.

