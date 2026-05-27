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

Implementation:
* tpo/anti-censorship/pluggable-transports/snowflake!50

Papers:
* https://www.bamsoftware.com/papers/snowflake/#p21

# AWS SQS queue

* tpo/anti-censorship/pluggable-transports/snowflake!214
* paper: https://www.petsymposium.org/foci/2024/foci-2024-0009.php
* https://www.bamsoftware.com/papers/snowflake/#p22

# DNS

Implementations:
* https://www.bamsoftware.com/software/dnstt/
* https://github.com/EndPositive/slipstream/
* https://github.com/masterking32/MasterDnsVPN

In theory will be easy to be blocked by censors if is not using encrypted DNS (DoH, DoT, ...). But in practice we see many censors not blocking by protocol even after a heavy use.

DNS packets are too small for many signaling channels needs, a good solution for it is to use [fountain codes](https://repo.or.cz/erasure-code-rendezvous.git).

# Pub/Sub

* Paper: https://www.petsymposium.org/foci/2024/foci-2024-0010.php  
* Implementation (go): https://github.com/AfonsoVilalonga/PubSub-Rendezvous

Pub/Sub is a service of unidirectional channels (called *topics*) where there is a publisher and multiple subscribers that subscribe to receive all the updates on this topic. To make a transport over it is necessary in advance to create two topics: *User's Topic* (UT) and *Broker's Topic* (BT). Clients know BT so they can subscribe to it and have access to a shared account with limited permissions to publish in UT. Clients send messages to the broker over UT and receive responses over BT.

Google's service is free up to 10GB/month.

The paper and implementation uses *Google Pub/Sub* service, but the same mechanism should work with *AWS SNS*. It will not work with azure because it assigns a specific domain to each created resource, and censors will be able to block it by domain.

# Google App Script

Implementations:
* javascript with signaling channels usecase: https://github.com/fortuna/OutlineDistribution  
* python generic transport: https://github.com/masterking32/MasterHttpRelayVPN

There is a limitation of 20k requests per day. It does work domain fronting google.com, which is allowlisted in some networks like Iran with the rest of internet is blocked.

# PassKeys servers

Using passkeys servers: https://fidoalliance.org/passkeys/

An example project: https://github.com/c-skills/passport

# ECH on cloud providers

[Cloudflare supports it](https://github.com/net4people/bbs/issues/393), but is the only mayor CDN that does. Is been [blocked in Russia](https://github.com/net4people/bbs/issues/417).

# AWS S3 - skyhook

The [Ten years gone](https://www.petsymposium.org/foci/2024/foci-2024-0011.pdf) paper does a review of [CoudTransport](https://petsymposium.org/2014/papers/paper_68.pdf) improving it and adapting it to the need of signaling channels, in the context of [RACEBOAT](https://github.com/tst-race/raceboat/). It proposes a channel called *Skyhook*.

Skyhook uses AWS S3 storage to communicate. AWS S3 service is reachable over an account independent domain name (but this is not what the standard AWS library does), so censors see in the SNI the S3 generic domain name and can't censor connections without blocking access to the whole S3 service.

Skyhook server shares a publicly writable object where clients write a pair of randomly generated UUIDs to bootstrap the connection. The server creates the objects of those UUIDs with publicly permissions one of write and another of read, and use them to talk with the client. Having a big space of UUIDs makes it that attackers can't guess those objects and access them.

The [skyhook implementation is written in C++](https://github.com/tst-race/skyhook), and don't seem to be actively maintained.

This technique might work with other cloud providers, but we'll need to find providers that don't place any account identifier in the domain name. Other major providers like google cloud or microsoft azure have their own APIs for object storage different to S3, but similar concepts so this might be applicable.

# Push notifications

https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Signaling-Channels/Push-Notifications

# Google docs

Implementation:
* https://github.com/0xinf0/gdocs-tunnel
* https://www.v2fly.org/en_US/v5/config/stream/gdocsviewer.html
* https://github.com/ShahabSL/Skirk

[It did manage](https://github.com/0xinf0/gdocs-tunnel) to get 1-5KB/s, which should be enough for signaling. It says there is a rate limit of "\~10 requests/minute sustainable per IP pair", the server side is _solved_ by using a big pool of IPv6 IPs. The client side "Rotates across 22 Google Anycast IPs to distribute limits", but for signaling might might not hit the limit.

Some ideas on how to use it:
* https://web.archive.org/web/20230330055859/https://easrng.blogspot.com/2022/03/get-tor-bridges-with-nothing-but.html  
* https://developers.cloudflare.com/1.1.1.1/other-ways-to-use-1.1.1.1/dns-in-google-sheets/  
* https://gitlab.torproject.org/tpo/anti-censorship/bridgedb/-/issues/40047

# TURN servers

https://www.petsymposium.org/foci/2025/foci-2025-0003.php

In general TURN services are paid per traffic.

# blockchain - MoneyMorph

[MoneyMorph](https://petsymposium.org/2020/files/papers/issue3/popets-2020-0058.pdf) is using blockchains as signaling channels. There is a [python implementation](https://github.com/moneymorph/Bitcoin-Ethereum-Zcash_implementation) and threat in [net4people](https://github.com/net4people/bbs/issues/71).