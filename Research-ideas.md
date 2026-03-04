These are research project ideas relating to anti-censorship work at Tor. If you're interested in working on any of them, feel free to reach out to us!

[TOC]

## Snowflake enumeration attempts

**Question**: If an adversary were to try to enumerate snowflake proxies, how many would they see? How much churn is there in Snowflake proxies? How effectively can they block Snowflake this way?

**Some relevant discussion/links**:
- Discussion during anti-censorship meeting: http://meetbot.debian.net/tor-meeting/2021/tor-meeting.2021-02-04-15.58.html
- Ticket for implementing Snowflake churn metrics: https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/issues/34075
- Some research done on this topic: https://lists.torproject.org/pipermail/anti-censorship-team/2024-July/000343.html
  - A follow up discussion with more ideas on future research: http://meetbot.debian.net/tor-meeting/2024/tor-meeting.2024-07-25-16.00.html 

## Calibrate bridge users estimation with on-bridge socket counts

Tor Metrics [bridge user graphs](https://metrics.torproject.org/userstats-bridge-country.html) depict not unique IP addresses, but rather an [average number of concurrently connected users per day](https://metrics.torproject.org/reproducible-metrics.html#users).
Simplifying slightly, the number of concurrent users is estimated
by [taking the number of directory requests and dividing by 10](https://research.torproject.org/techreports/counting-daily-bridge-users-2012-10-24.pdf#page=5).
The constant of 10 is somewhat arbitrary, reflecting an educated guess that an average Tor users remains connected for 2.4 hours per day.

The constant 10 is effectively a scaling factor for the user graphs.
Its exact value does not matter when, for example,
you want to compare two graphs to see which is bigger.
But it would be nice if it were calibrated to match reality
as closely as possible.

The idea is to repeatedly sample the number of sockets that are connected to the localhost [ExtORPort](https://gitweb.torproject.org/torspec.git/tree/proposals/196-transport-control-ports.txt) to get an average per day, then compare that locally computed average with what Tor Metrics reports.
If the currently used ExtOrPort is 127.0.0.1:1234, then you can sample the number of sockets currently connected to the ExtOrPort with a command like
```
ss -n state established 'dport = 1234' 'dst 127.0.0.1' | wc -l
```
(Subtract 1 to account for the header in the output.)

@dcf ran a socket counting script on two snowflake bridges for a few weeks in June–July 2023 and compared the sockets counts to Tor Metrics calculations. The snowflake-01 bridge was off by about a factor of 2.0, and the snowflake-02 bridge was off by a factor of about 1.5. The difference may be attributable to the fact that, at the time, the snowflake-02 bridge had been included in Tor Browser since version 12.0.3, but Orbot 17 (the first version to contain the snowflake-02 bridge) had not been fully released yet.

* https://www.bamsoftware.com/talks/pets-2023-metrics/
* https://www.bamsoftware.com/talks/pets-2023-metrics/pets-2023-metrics.zip

Compare results with ["Understanding Tor Usage with Privacy-Preserving Measurement"](https://dl.acm.org/doi/abs/10.1145/3278532.3278549).

## Improve Snowflake's NAT discovery and matching algorithm

[Snowflake](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/wikis/home) is an anti-censorship tool that uses the NAT traversal properties of WebRTC to connect clients with a large pool of temporary circumvention proxies. NAT traversal and browser support for WebRTC lowers the barrier to running circumvention proxies and enables the usage of addons in popular web browsers to run proxy code. However, [not all NAT and firewall configurations are mutually compatible](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/wikis/NAT-matching).

We have several NAT discovery algorithms to determine which type of NAT each client and proxy have in order to decide on working matches. When gathering STUN candidates, Snowflake clients use STUN servers that support [RFC 5780](https://www.rfc-editor.org/rfc/rfc5780) to determine their NAT mapping and filtering behaviour. Clients categorize their NAT type as one of:
- restrictive (symmetric)
- unrestrictive (all others)
Proxies determine their NAT type by making a test connection to a symmetrically NAT'd peer running on a [probe server](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/tree/main/probetest?ref_type=heads). If the connection succeeds, they self-categorize as suitable for restrictively NAT'd clients. Otherwise, they are distributed only to clients with unrestricted NATs. In addition, if a browser-based proxy fails to form a connection to a restricted client with which it was matched five times in a row, it will update its NAT type so that it is only distributed to unrestricted clients.

There a few inefficiencies with our approach:
- Proxies that work with restrictively NAT'd clients are in higher demand and more rare than other proxies

- Approximately 1/3 of client polls report their NAT type as "unknown". This could be due to the fact that the NAT behaviour test takes time to complete, and in order to minimize the bootstrapping time for Snowflake we have them poll first before the test is done. Clients with "unknown" NATs are assumed to have restricted NATs and therefore deplete the pool of proxies allocated to them.

- Our matching algorithm still occasionally produces non-working NAT assignments. If this happens, clients will fail to open a datachannel with the proxy and timeout after 20 seconds. At this point, they perform the rendezvous step with the broker and try again. If we could discover how often this happens and for what NAT types we could implement a fix. It's possible we need more than just two buckets (restrictive, unrestrictive) of client NAT types.

- We have reason to believe that even though browser-based proxies are much more numerous than standalone proxies, [their usage is much lower](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake-webext/-/issues/82#note_2892274). This has negative implications for enumeration resistance.

Related issues:
- https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/issues/40178
- https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/issues/40077

## Lox related open problems
1. **Question**: How do we know that a bridge is blocked?
   
   **Some relevant discussion/links**:
    - This is a long-standing issue that will benefit all bridge distributors, but is particularly important for Lox. There is some discussion of the issue [here](https://gitlab.torproject.org/tpo/anti-censorship/lox/lox-overview/-/wikis/Lox-Roadmap#bridge-reachability).
     - Issue: [Brainstorm and analyze heuristics to guess that a bridge might be offline or blocked](https://gitlab.torproject.org/tpo/anti-censorship/censorship-analysis/-/issues/40035)
     - Metrics can be helpful in determining a blockage
     - Bridge users/operators (as a first line and quickest way to collect info)
     - Compare reports against other metrics to make final determination

2. **Question**: Find optimal parameter tuning for different censorship landscapes 

   **Some relevant discussion/links**:
   - This is a more long-term issue that won't be able to be easily addressed until Lox has been deployed and we have a better idea of how it performs as a bridge distribution system in practice.
   -  With metrics and careful measurement of Lox's performance, some patterns may emerge that are helpful for finding optimal parameters for different types of situations. 

## Rateless erasure codes for rendezvous over small fragments

See https://github.com/net4people/bbs/issues/591.

[Assemblage](https://www.petsymposium.org/foci/2026/foci-2026-0005.php) /
[Collage](https://censorbib.nymity.ch/#Burnett2010a)
use [rateless erasure codes](https://en.wikipedia.org/wiki/Fountain_code)
to fragment a discrete message
(as opposed to an unbounded stream)
into small pieces that can then be collected to recover the original message.
Such codes could be a good vehicle for doing rendezvous
over channels that are space-limited or unreliable,
[such as encrypted DNS](tpo/anti-censorship/pluggable-transports/snowflake#25874).

Generally, such a system could be a discrete-message analog
to the stream-oriented Turbo Tunnel.

The research plan would be to first design and implement a proof of concept
without any obfuscation transport or integration with a real circumvention system,
perhaps using just UDP datagrams and a UDP listener.
Then do a DNS encoding and try integrating it with the Snowflake broker.