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

## Troubleshooting polyanthum

The meek server and BridgeDB Moat distributor are run on polyanthum. The 

## Troubleshooting the Moat server

Moat consists of a meek server, some apache2 configs, and a BridgeDB distributor. The meek server listens on 127.0.0.1:2000 and is started by the [run-meek](https://gitlab.torproject.org/tpo/anti-censorship/bridgedb-admin/-/blob/master/bin/run-meek) script.

There are `ProxyPass` rules in `/etc/apache2/sites-available/bridges.torproject.org.conf` on polyanthum to forward requests to https://bridges.torproject.org/meek and https://moat.torproject.org to http://127.0.0.1:2000/.

Useful apache logs can be found in `/var/log/apache2/bridges.torproject.org-access.log` and `/var/log/apache2/bridges.torproject.org-error.log`.

The BridgeDB Moat distributor listens on port 3881, as set in the BridgeDB configuration file `MOAT_HTTP_PORT = 3881` and a corresponding ProxyPass rule is set up to direct requests from the meek tunnel:
```
ProxyPass /moat/ http://127.0.0.1:3881/moat/
```