A signaling channel is a transport that can be public, because the collateral damage for censors to block it, but too slow or too expensive to move real Tor traffic over it. We use those to bootstrap other protocols like [moat](https://gitlab.torproject.org/tpo/anti-censorship/rdsys/-/blob/main/doc/moat.md?ref_type=heads) or [snowflake](https://snowflake.torproject.org/)

[TOC]

# domain fronting

There are less and less cloud providers allowing it.

* https://www.bamsoftware.com/papers/snowflake/#p20

# AMP cache

* tpo/anti-censorship/pluggable-transports/snowflake!50
* https://www.bamsoftware.com/papers/snowflake/#p21

# AWS SQS queue

* tpo/anti-censorship/pluggable-transports/snowflake!214
* https://www.bamsoftware.com/papers/snowflake/#p22

# dnstt

https://www.bamsoftware.com/software/dnstt/
https://github.com/EndPositive/slipstream/

# Google Pub/Sub

https://www.petsymposium.org/foci/2024/foci-2024-0010.php  
https://github.com/AfonsoVilalonga/PubSub-Rendezvous

# Google App Script

https://github.com/fortuna/OutlineDistribution  
https://github.com/masterking32/MasterHttpRelayVPN

There is a limitation of 20k requests per day.

# PassKeys servers

Using passkeys servers: https://fidoalliance.org/passkeys/

An example project: https://github.com/c-skills/passport

# ECH on cloud providers

Cloudflare seems to support it already: https://github.com/net4people/bbs/issues/393

# S3

CloudTransport: https://petsymposium.org/2014/papers/paper_68.pdf  
https://www.petsymposium.org/foci/2024/foci-2024-0011.pdf

# Push notifications

https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Signaling-Channels/Push-Notifications
https://petsymposium.org/popets/2025/popets-2025-0153.pdf

# Google docs

https://web.archive.org/web/20230330055859/https://easrng.blogspot.com/2022/03/get-tor-bridges-with-nothing-but.html  
https://developers.cloudflare.com/1.1.1.1/other-ways-to-use-1.1.1.1/dns-in-google-sheets/  
https://gitlab.torproject.org/tpo/anti-censorship/bridgedb/-/issues/40047

# TURN servers

https://www.petsymposium.org/foci/2025/foci-2025-0003.php

In general TURN services are paid per traffic.



# library implementations

* https://github.com/getlantern/kindling
* https://github.com/tst-race/raceboat