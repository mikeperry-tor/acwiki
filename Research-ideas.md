These are research project ideas relating to anti-censorship work at Tor. If you're interested in working on any of them, feel free to reach out to us!

## Snowflake enumeration attempts

**Question**: If an adversary were to try to enumerate snowflake proxies, how many would they see? How much churn is there in Snowflake proxies? How effectively can they block Snowflake this way?

**Some relevant discussion/links**:
- Discussion during anti-censorship meeting: http://meetbot.debian.net/tor-meeting/2021/tor-meeting.2021-02-04-15.58.html
- Ticket for implementing Snowflake churn metrics: https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/issues/34075

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