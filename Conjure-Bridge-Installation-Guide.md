Instructions for setting up a new Conjure bridge

## Introduction

The Conjure bridge sits between deployed Conjure stations and the Tor network. It acts as an entrypoint to access the Tor network through the Conjure.

While the PT server itself is a simple haproxy server, we maintain wireguard connections between deployed stations. Conjure allows clients to connect to arbitrary destinations, but in order to preserve the client IP address for [safely collected usage metrics](https://metrics.torproject.org/about.html), we [configure the Conjure PT client to send a `PROXY` header](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/conjure/-/blob/77bf2fef6e1585a90112f7519a935fabd79a559a/client/conjure/registration.go#L81) with the client's IP address in the TCP connection to the Conjure bridge. These country-based usage metrics can help us detect and respond to censorship events and are an important part of quickly adapting to changes in censorship. The Wireguard connections prevent client IPs from being forwarded in plaintext to the Conjure bridge.

There are currently multiple conjure stations. New stations are occasionally deployed and existing stations may be moved. The Wireguard setup requires good communication with Conjure station admins, an important part of Conjure bridge maintenance is a quick response time for updating the Wireguard configuration.

## Bridge Setup

To set up the Conjure PT server and bridge, first check the [general relay requirements](https://community.torproject.org/relay/relays-requirements/).

What follows are instructions for running the Conjure bridge on Debian bookworm.


1. Enable Automatic Software Updates
   
   One of the most important things to keep your relay secure is to install security updates timely and ideally automatically so you can not forget about it. Follow the instructions to enable automatic software updates for your operating system.

2. Configure Tor Project's Repository
   
   Configuring the Tor Project's package repository for Debian/Ubuntu is recommended and documented on Support portal. Please follow those instructions before proceeding.

   Note: Ubuntu users need to get Tor from the Tor Project's repository.
3. Install Tor
   
   Ensure you update the packages database before installing the package, than call apt to install it:
   ```
   apt update
   apt install tor
   ```
4. Install the Conjure server
   
   You will have to build the Conjure server from source using the following steps:

   1. Install a recent version of Go: https://go.dev/dl/
   2. Clone the Conjure PT repository: https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/conjure/
   3. Change to the server directory and build
      ```
      cd conjure/server/
      go build
      ```
   4. Copy the server binary to a PATH accessible location
      ```
      sudo cp server /usr/local/bin/conjure-server
      ```
5. Edit your Tor config file, usually located at /etc/tor/torrc and replace its content with:
   ```
   BridgeRelay 1
   
   ORPort 9001
   
   ServerTransportListenAddr conjure 0.0.0.0:80
   
   # Local communication port between Tor and obfs4.  Always set this to "auto".
   # "Ext" means "extended", not "external".  Don't try to set a specific port number, 
   # nor listen on 0.0.0.0.
   ExtORPort auto
   
   # Replace "<address@email.com>" with your email address so we can contact you if there are 
   # problems with your bridge.
   # This is optional but encouraged.
   ContactInfo <address@email.com>
   
   # Pick a nickname that you like for your bridge.  This is optional.
   Nickname PickANickname
   
   # The allowed-stations argument is subject to change and is dependent on the wireguard
   # configuration.
   ServerTransportPlugin conjure exec /usr/local/bin/conjure-server --allowed-stations 10.0.1.2,10.0.1.3,10.0.1.4,10.0.1.5,10.0.1.6,10.0.1.7 -log /var/log/tor/conjure-server.log

   BridgeDistribution none
   ```
   Don't forget to change the `ContactInfo`, and `Nickname` options.
   

6. Configure systemd and give conjure-server `CAP_NET_BIND_SERVICE` capabilities to bind the port with a non-root user:
   ```
   sudo setcap cap_net_bind_service=+ep /usr/local/bin/conjure-server
   ```
   
7. Work around systemd hardening
   You will also need to edit and change the configuration.
   
   Run the command:
   ```
   sudo systemctl edit tor@.service tor@default.service
   ```
   In the editor, enter the following text, then save and quit.
   ```
   [Service]
   NoNewPrivileges=no
   ```
   In the second editor that appears, enter the same text, then save and quit.
   ```
   [Service]
   NoNewPrivileges=no
   ```
   If everything worked correctly, you will now have two files /etc/systemd/system/tor@.service.d/override.conf and /etc/systemd/system/tor@default.service.d/override.conf containing the text you entered.

8. Restart Tor
   
   Enable and Start tor:
   ```
   # systemctl enable --now tor.service
   ```
   Or restart it if it was running already, so configurations take effect:
   ```
   # systemctl restart tor.service
   ```
9. Monitor your logs
   
   To confirm your bridge is running with no issues, you should see something like this (usually in /var/log/syslog or run # journalctl -e -u tor@default):

   ```
   [notice] Your Tor server's identity key fingerprint is '<NICKNAME> <FINGERPRINT>'
   [notice] Your Tor bridge's hashed identity key fingerprint is '<NICKNAME> <HASHED FINGERPRINT>'
   [notice] Registered server transport 'conjure' at '[::]:80'
   [notice] Tor has successfully opened a circuit. Looks like client functionality is working.
   [notice] Bootstrapped 100%: Done
   [notice] Now checking whether ORPort <redacted>:9001 is reachable... (this may take up to 20 minutes -- look for log messages indicating success)
   [notice] Self-testing indicates your ORPort is reachable from the outside. Excellent. Publishing server descriptor.
   ```

10. Final Notes
    
    If you are having trouble setting up your bridge, have a look at our [help section](https://community.torproject.org/relay/getting-help/). If your bridge is now running, check out the [post-install notes](https://community.torproject.org/relay/setup/bridge/post-install/).
    
    If you get stuck, take a look at the [technical setup instructions](https://community.torproject.org/relay/setup/bridge/) for running a bridge. The setup will be slightly different from an obfs4 bridge, but some of the same system-specific tips will help.

## Wireguard setup

Install [wireguard](https://www.wireguard.com/install/). A full documentation of the wireguard setup is available in the [Conjure PT wiki](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/conjure/-/wikis/wireguard-setup)