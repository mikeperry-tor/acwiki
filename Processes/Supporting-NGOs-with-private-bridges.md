# Supporting NGOs with private bridges

Inspired by tpo/community/support#28526, this page documents how we can support NGOs with private bridges.

## 0. Understand the NGO's requirements
To be maximally useful, we need to understand the NGO's requirements. Ask at least the following questions:
* How many users do you have?
* Where are your users located?
* What is your threat model?
* What platforms are your users using? Desktop? Android?
* Can you run your own bridges? Or do you need bridges from us?

## 1. Point the NGO to Tor Browser download links
The NGO's users are likely subject to censorship and therefore unable to access our [official download page](https://www.torproject.org/download/). To download Tor Browser, we need to point the NGO to [GetTor](https://gettor.torproject.org) download links, which the NGO can then distribute to its users:
* Internet archive: https://archive.org/details/@gettor
* Google Drive folder: https://drive.google.com/drive/folders/13CADQTsCwrGsIID09YQbNz2DfRMUoxUU
* GitHub: https://github.com/torproject/torbrowser-releases/releases/
* GitLab: We maintain dedicated repositories for Windows, Linux, and OS X.

These hosting platforms all contain a large and confusing list of download links. To make things easier for the NGO, provide a few specific links; that is, links for Windows, MacOS, and Linux; for the desired locale. Also tell the NGO that its users can download their own copy of Tor Browser by emailing gettor@torproject.org.

## 2. Supply the NGO with bridges
There are two options. Whatever option we go with, we should monitor the bridges and take action if any go offline.

### 2.1 Teach the NGO how to run their own bridges
Point the NGO to our [bridge setup guides](https://community.torproject.org/relay/setup/bridge/) and tell them to use the following torrc instead, to keep their bridge private:
```
BridgeRelay 1

# Replace "TODO1" with a Tor port of your choice.  This port must be externally
# reachable.  Avoid port 9001 because it's commonly associated with Tor and
# censors may be scanning the Internet for this port.  You can firewall this
# port if your users only connect over obfs4.
ORPort TODO1

ServerTransportPlugin obfs4 exec /usr/bin/obfs4proxy

# Replace "TODO2" with an obfs4 port of your choice.  This port must be
# externally reachable and must be different from the one specified for ORPort.
# Avoid port 9001 because it's commonly associated with
# Tor and censors may be scanning the Internet for this port.
ServerTransportListenAddr obfs4 0.0.0.0:TODO2

# Local communication port between Tor and obfs4.  Always set this to "auto".
# "Ext" means "extended", not "external".  Don't try to set a specific port
# number, nor listen on 0.0.0.0.
ExtORPort auto

# Replace "<address@email.com>" with your email address so we can contact
# you if there are problems with your bridge. This is optional but encouraged.
ContactInfo <address@email.com>

# Pick a nickname that you like for your bridge.  This is optional.
Nickname PickANickname

# Tell BridgeDB to not distribute the bridge, so it remains private.
BridgeDistribution none

# Don't self-test, to minimise exposure.
AssumeReachable 1
```
Tell the NGO that they may also want to firewall their bridges' OR port (as long as tpo/core/tor#7349 remains a problem). Mention that we are happy to help them test their bridges, to make sure that everything is configured correctly.

### 2.2 Supply the NGO with bridges
We are closely working with volunteers who maintain a pool of reliable and fast obfs4 bridges in various data centres around the world. We can take a subset of these bridges and send them to an NGO for private distribution. Keep track of what bridge was sent to what NGO.

## 3. Provide instructions on adding bridges
Provide instructions on how to add these private bridges to Tor Browser. Provide our [official instructions](https://tb-manual.torproject.org/bridges/) and, if available, localised instructions (e.g., [in Chinese](https://tb-manual.torproject.org/zh-CN/bridges/)).

Orbot can (or will) hook a bridge:// URI (see tpo/applications/tor-browser#28015 and legacy/trac#15035), making it easier for the NGO's users to configure their bridges.