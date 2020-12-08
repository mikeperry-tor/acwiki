Did meek-azure or moat break? Are you feeling panicky and frustrated? You have come to the right place. Here's how you can diagnose the issue:

1. Test if you can reach our meek server (which runs on polyanthum) through the Azure infrastructure:
   ```
   $ wget -q -O - https://ajax.aspnetcdn.com/ --header 'Host: meek.azureedge.net'              
   I’m just a happy little web server.
   ```
   If you don't see this message, the problem is likely somewhere in Azure.
2. Take a look at our Azure CDN configuration in portal.azure.com.
3. Our [meek bridge](https://metrics.torproject.org/rs.html#details/8F4541EEE3F2306B7B9FEF1795EC302F6B84DAE8)'s client numbers can tell you if clients can reach our bridge.