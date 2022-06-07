This is a guide for altering or debugging issues with Moat. For more details on how Moat works and is deployed, see [the Moat documentation](https://gitlab.torproject.org/tpo/anti-censorship/bridgedb/-/wikis/Moat).

Right now Moat is deployed on Microsoft Azure.

## Troubleshooting the domain fronting configuration
1. Test if you can reach our meek server (which runs on polyanthum) through the Azure infrastructure:
   ```
   $ wget -q -O - https://ajax.aspnetcdn.com/ --header 'Host: onion.azureedge.net'              
   I’m just a happy little web server.
   ```
   If you don't see this message, the problem is likely somewhere in Azure.

   And the Fastly infrastructure:
   ```
   $ wget -q -O - https://cdn.sstatic.net --header 'Host: moat.torproject.org.global.prod.fastly.net'           
   I’m just a happy little web server.
   ```
   If you don't see this message, the problem is likely somewhere in Fastly.
2. Take a look at our Azure CDN configuration in portal.azure.com or the Fastly account at fastly.com

#### Setting up a new domain front

If you are setting up a new domain front for Moat, you can point it towards either https://bridges.torproject.org/meek, or if the CDN does not allow you to forward to URLS, to https://moat.torproject.org.

You will also need to update the client side in Tor Browser. Moat is configured in [tor-launcher](https://gitlab.torproject.org/tpo/applications/tor-launcher). Once that has been updated, you can pull the latest changes in the Tor Browser build process.

## Restarting the Moat server

Moat is implemented with the binary `meek-server` located in `/srv/bridges.torproject.org/bin/`. The source code for this binary is available in the `meek-server` directory of the [meek repository](https://gitweb.torproject.org/pluggable-transports/meek.git/). We run a shim in a separate process to translate user address information from meek and insert it into an `X-Forwarded-For` header. 

The meek server and moat shim can be restarted separately.

1. Kill the existing processes running `meek-server` and/or `moat-shim`
2. To start meek, use systemd as user `bridgedb`:
```
sudo -u bridgedb systemctl --user restart meek
```
3.To start the moat shim, use systemd as user `moat`:
```
sudo -u moat systemctl --user restart moat-shim
```

#### Updating Moat
To deploy a new binary for `meek-server` or the `moat-shim`, place the compiled binary in the [bridgedb-admin](https://gitlab.torproject.org/tpo/anti-censorship/bridgedb-admin) repository. Then pull the most recent changes and follow the above instructions for restarting Moat.

## Troubleshooting the Moat server

Moat consists of a meek server, some apache2 configs, and a BridgeDB distributor. The meek server listens on 127.0.0.1:2000 and is started by the [run-meek](https://gitlab.torproject.org/tpo/anti-censorship/bridgedb-admin/-/blob/master/bin/run-meek) script.

There are `ProxyPass` rules in `/etc/apache2/sites-available/bridges.torproject.org.conf` on polyanthum to forward requests to https://bridges.torproject.org/meek and https://moat.torproject.org to http://127.0.0.1:2000/.

Useful apache logs can be found in `/var/log/apache2/bridges.torproject.org-access.log` and `/var/log/apache2/bridges.torproject.org-error.log`.

The BridgeDB Moat distributor listens on port 3881, as set in the BridgeDB configuration file `MOAT_HTTP_PORT = 3881` and a corresponding ProxyPass rule is set up to direct requests from the meek tunnel:
```
ProxyPass /moat/ http://127.0.0.1:3881/moat/
```