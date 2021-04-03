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

## Troubleshooting polyanthum

The meek server and BridgeDB Moat distributor are run on polyanthum. The 