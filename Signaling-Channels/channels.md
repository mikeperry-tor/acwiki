---
title: Ideas of Channels
---
[TOC]

# domain fronting

Domain fronting uses the fact that CDNs tend to server multiple websites from the same http reverse proxy. In the unencrypted part of the request the *SNI* contains an uncensored domain name, so the censors will allow the connection thinking that is a visitor of that domain. While in the TLS encrypted part the *Host* http header contains the real domain that we want to visit, so the CDN reverse proxy will forward the connection to our service.

Original developed for [meek](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/meek) it is currently widely used in anti-censorship tools. But there are less and less cloud providers allowing it.

Papers:
* https://www.bamsoftware.com/papers/fronting/
* https://www.bamsoftware.com/papers/snowflake/#p20

# AMP cache

AMP cache is a website cache protocol. The channel encodes the request in the URL and the response on the website's html.

Originally implemented by google, bing and cloudflare. But [cloudflare stopped running the service and bing doesn't fetch content on demand](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/work_items/25985#note_2744744) leaving google as the only option for this channel. In google it can be used with *google.com* as domain front.

Papers:
* the original proposal in net4people: https://github.com/net4people/bbs/issues/5
* https://www.bamsoftware.com/papers/snowflake/#p21

Implementation:
* tpo/anti-censorship/pluggable-transports/snowflake!50

# AWS SQS queue

SQS is a queue service to send and receive messages between web services run by Amazon's AWS. The clients communicate with a generic domain name, so no account can be identified just by observing the connections and censors either block the whole service or allows the SQS channel.

Similar services are available in other cloud providers like [azure](https://learn.microsoft.com/en-us/azure/architecture/aws-professional/messaging#simple-queue-service), but they haven't been explored to see if they could be used the same way.

Papers:
* https://www.petsymposium.org/foci/2024/foci-2024-0009.php
* https://www.bamsoftware.com/papers/snowflake/#p22

Implementation:
* tpo/anti-censorship/pluggable-transports/snowflake!214

# DNS

In theory will be easy to be blocked by censors if is not using encrypted DNS (DoH, DoT, ...). But in practice we see many censors not blocking by protocol even after a heavy use.

DNS packets are too small for many signaling channels needs, a good solution for it is to use [fountain codes](https://repo.or.cz/erasure-code-rendezvous.git).

Papers:
* fountain codes: https://repo.or.cz/erasure-code-rendezvous.git/blob_plain/HEAD:/paper/fountain-code-rendezvous.f23d22ad.pdf (draft)

Implementations:
* https://www.bamsoftware.com/software/dnstt/
* https://github.com/EndPositive/slipstream/
* https://github.com/masterking32/MasterDnsVPN

# Pub/Sub

Pub/Sub is a service of unidirectional channels (called *topics*) where there is a publisher and multiple subscribers that subscribe to receive all the updates on this topic. To make a transport over it is necessary in advance to create two topics: *User's Topic* (UT) and *Broker's Topic* (BT). Clients know BT so they can subscribe to it and have access to a shared account with limited permissions to publish in UT. Clients send messages to the broker over UT and receive responses over BT.

Google's service is free up to 10GB/month.

The paper and implementation uses *Google Pub/Sub* service, but the same mechanism should work with *AWS SNS*. It will not work with azure because it assigns a specific domain to each created resource, and censors will be able to block it by domain.

Papers:
* https://www.petsymposium.org/foci/2024/foci-2024-0010.php  

Implementations:
* go: https://github.com/AfonsoVilalonga/PubSub-Rendezvous

# Google App Script

Google App Script is a service to automatize tasks in the google platform.

There is a limitation of 20k requests per day. It does work domain fronting google.com, which is allowlisted in some networks like Iran with the rest of internet is blocked.

Implementations:
* javascript with signaling channels usecase: https://github.com/fortuna/OutlineDistribution  
* python generic transport: https://github.com/masterking32/MasterHttpRelayVPN
* rust AI rewrite: https://github.com/therealaleph/MasterHttpRelayVPN-RUST

# PassKeys servers

Using passkeys servers: https://fidoalliance.org/passkeys/

An example project: https://github.com/c-skills/passport

# ECH on cloud providers

ECH hides the SNI so all the connections to the same provider looks like visiting the same domain. Is a new-ish technology, with very few adoption by major providers. Except for [Cloudflare that added support in 2024](https://github.com/net4people/bbs/issues/393).

In theory is not possible to distinguish if a connection is done using ECH or not, as modern browsers send random data in the *GREASE* field when is not used. But Cloudflare is using a specific domain name for ech (cloudflare-ech.com). [Russia is blocking](https://github.com/net4people/bbs/issues/417) all the connections to that domain name containing a *GREASE* field.

# AWS S3 - skyhook

The [Ten years gone](https://www.petsymposium.org/foci/2024/foci-2024-0011.pdf) paper does a review of [CoudTransport](https://petsymposium.org/2014/papers/paper_68.pdf) improving it and adapting it to the need of signaling channels, in the context of [RACEBOAT](https://github.com/tst-race/raceboat/). It proposes a channel called *Skyhook*.

Skyhook uses AWS S3 storage to communicate. AWS S3 service is reachable over an account independent domain name (but this is not what the standard AWS library does), so censors see in the SNI the S3 generic domain name and can't censor connections without blocking access to the whole S3 service.

Skyhook server shares a publicly writable object where clients write a pair of randomly generated UUIDs to bootstrap the connection. The server creates the objects of those UUIDs with publicly permissions one of write and another of read, and use them to talk with the client. Having a big space of UUIDs makes it that attackers can't guess those objects and access them.

The [skyhook implementation is written in C++](https://github.com/tst-race/skyhook), and don't seem to be actively maintained.

This technique might work with other cloud providers, but we'll need to find providers that don't place any account identifier in the domain name. Other major providers like google cloud or microsoft azure have their own APIs for object storage different to S3, but similar concepts so this might be applicable.

Papers:
* skyhook: https://www.petsymposium.org/foci/2024/foci-2024-0011.pdf
* CloudTransport: https://petsymposium.org/2014/papers/paper_68.pdf

Implementations:
* https://github.com/tst-race/skyhook

# Push notifications

https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Signaling-Channels/Push-Notifications

# Google docs

[gdocs-tunnel](https://github.com/0xinf0/gdocs-tunnel) managed to get 1-5KB/s, which should be enough for signaling. It says there is a rate limit of "\~10 requests/minute sustainable per IP pair", the server side is _solved_ by using a big pool of IPv6 IPs. The client side "Rotates across 22 Google Anycast IPs to distribute limits", but for signaling might might not hit the limit.

Implementations:
* https://github.com/0xinf0/gdocs-tunnel
* https://www.v2fly.org/en_US/v5/config/stream/gdocsviewer.html
* https://github.com/ShahabSL/Skirk

Some ideas on how to use it:
* https://web.archive.org/web/20230330055859/https://easrng.blogspot.com/2022/03/get-tor-bridges-with-nothing-but.html  
* https://developers.cloudflare.com/1.1.1.1/other-ways-to-use-1.1.1.1/dns-in-google-sheets/  
* https://gitlab.torproject.org/tpo/anti-censorship/bridgedb/-/issues/40047

# TURN servers

In general TURN services are paid per traffic.

Papers:
* https://www.petsymposium.org/foci/2025/foci-2025-0003.php

# blockchain - MoneyMorph

[MoneyMorph](https://petsymposium.org/2020/files/papers/issue3/popets-2020-0058.pdf) is using blockchains as signaling channels. There is a [python implementation](https://github.com/moneymorph/Bitcoin-Ethereum-Zcash_implementation) and threat in [net4people](https://github.com/net4people/bbs/issues/71).

Some countries have censored blockchains, they might not have a big collateral damage for some censors.

Papers:
* https://petsymposium.org/2020/files/papers/issue3/popets-2020-0058.pdf

Implementation:
* https://github.com/moneymorph/Bitcoin-Ethereum-Zcash_implementation