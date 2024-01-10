## General information

* Rdsys consists of several microservices. This document only covers rdsys's backend process whose name is rdsys-backend. For brevity, the rest of this document refers to the backend process as rdsys.
* Scripts and config files related to rdsys's deployment are in the [rdsys-admin](https://gitlab.torproject.org/tpo/anti-censorship/rdsys-admin) repository.
* Rdsys's systemd scripts are in rdsys-admin git repo.
* There's a crontab entry (run `crontab -e` as user rdsys in bridges.torproject.org) that invokes logrotate once a day to [rotate rdsys's log files](https://gitlab.torproject.org/tpo/anti-censorship/rdsys-admin/-/blob/master/logrotate/logrotate.conf).
* Take a look at [rdsys's metrics](https://bridges.torproject.org/rdsys-backend-metrics) for a quick check if the service is running.
* There is a staging server to test rdsys, see the [Rdsys Staging Survival Guide](https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Survival-Guides/Rdsys-Staging-Survival-Guide)

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

## (Re)starting rdsys backend

1. Log into bridges.torproject.org.
2. Change to the rdsys user by running `sudo -u rdsys -s`.
3. Update the rdsys-admin repo if needed `cd ~/rdsys-admin; git pull`
4. (Re)start the rdsys-backend process via its systemd script: `systemctl --user [start|stop|status] rdsys-backend`.
5. Take a look at rdsys's log file at /home/rdsys/logs/rdsys-backend.log to make sure that the service (re)started successfully.

## (Re)starting rdsys frontends

1. Log into rdsys-frontend-01.torproject.org.
2. Change to the rdsys user by running `sudo -u rdsys -s`.
3. Update the rdsys-admin repo if needed `cd /srv/rdsys.torproject.org/rdsys-admin; git pull`
4. Change to the service user by running `sudo -u gettor -s`.
5. (Re)start the `<service>-distributor` or `<service>-updater` process via its systemd script: `systemctl --user [start|stop|status] gettor-distributor`.
6. Take a look at the logs with `journal --user -f`.

## Deploying a new frontend service

1. [Open an issue with TPA](https://gitlab.torproject.org/tpo/tpa/team/-/issues/new) to create a new user on rdsys-frontend-01 for the service (see the [lox distributor issue](https://gitlab.torproject.org/tpo/tpa/team/-/issues/41330) for an example).
2. Create a service file for the new service and add it to the [rdsys-admin](https://gitlab.torproject.org/tpo/anti-censorship/rdsys-admin/-/tree/main/systemd?ref_type=heads) repository
3. Log into rdsys-frontend-01.torproject.org
4. Change to the `<service>` user by running `sudo -u <service> -s`
5. Create a symlink for the service in the user `$HOME` directory:
   ```
   ln -s /srv/rdsys.torproject.org/rdsys-admin/systemd/<service>.service ~/.config/systemd/users/
   ```
6. Enable the service: `systemctl --user enable <service>`
7. Start the service: `systemctl --user start <service>` (see wiki for help: https://gitlab.torproject.org/tpo/tpa/team/-/wikis/doc/services)
8. Check to make sure the service is running: `systemctl --user status <service>`

## Deploying a new version

1. Compile the binary disabling CGO: `CGO_ENABLED=0 go build ./cmd/backend`
2. Copy the binary to the server: `scp backend polyanthium:`
3. Log into polyanthium.
4. Change to the rdsys user by running `sudo -u rdsys -s`.
5. Make a copy of the old binary so we can roll back if there is any problem: \`mv <span dir="">\~</span>/bin/rdsys-backend <span dir="">\~</span>/bin/rdsys-backend.old
6. Copy the binary to its place: `cp /home/<user>/backend ~/bin/rdsys-backend`
7. Restart the service process via systemd: `systemctl --user restart rdsys-backend`.


## Deploying a new lox-distributor

1. Compile the binary in the root of the `lox-distributor` crate: `cargo build --bin --release`
2. The binary will be in the `lox/target/release` directory. Copy the binary to the server: `scp target/release/lox-distributor rdsys-frontend-01:`
3. Log into `rdsys-frontend-01.
4. Change to the `rdsys` user by running `sudo -u rdsys -i`.
5. Make a copy of the old binary so we can roll back if there is any problem: `mv /srv/rdsys.torproject.org/bin/lox-distributor /srv/rdsys.torproject.org/bin/lox-distributor.old`
6. Copy the binary to its place: `cp /home/<user>/lox-distributor /srv/rdsys.torproject.org/bin/lox-distributor`
7. Exit the `rdsys` user space and change to the `lox` user by running `sudo -u lox -i`.
7. Restart the service process via systemd: `systemctl --user restart rdsys-lox`.