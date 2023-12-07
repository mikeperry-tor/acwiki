rdsys-test-01 is our staging server to test rdsys changes. It is meant to be a place to try changes that we are working on and to easily remove all the custom changes to have a clean setup for other tests.

All the development is done as the *rdsys* user. Change to that user running: `sudo -u rdsys -s`

## Services

All the services are run with systemd as a local user.

They can be restarted with: `systemctl --user restart rdsys-<service>`

And their logs are accesible in `~/logs` or journald: `journalctl --user`

The service config located in `~/conf/<service>.json`

### backend

A backend with generated fake bridges. The descriptors are in `~/from-serge`.

### moat

The moat service is reachable from outside in *https://bridges-test.torproject.org/moat/...*.
```
> curl https://rdsys-test-01.torproject.org/moat/cicumvention/defaults
{
  "settings": [
    {
      "bridges": {
        "type": "obfs4",
        "source": "builtin",
        "bridge_strings": [
...
```

## deploy a clean rdsys

There is a [deploy script](https://gitlab.torproject.org/tpo/anti-censorship/rdsys-admin/-/blob/main/test/deploy.sh?ref_type=heads) that does remove everything in the current deployment and do a clean deploy from the main branch of rdsys. It can be run as *rdsys* user by: `~/deploy`