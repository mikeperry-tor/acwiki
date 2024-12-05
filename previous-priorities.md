## The following were our goals for 2024:

* Make Tor more accessible in China.
* Make Tor more accessible in Iran.
* Make Tor more accessible in Turkmeinstan.
* Detect and investigate attempts to censor Tor.
* Improve the design and reliability of our software.
* Improve the performance of Snowflake so that Tor bootstraps reliably on a mobile phone in China.
* Maintain probes in areas that are likely to censor Tor and collect pack captures and probe results for storage and analysis.
* Diversify domain fronting options.
* Discovering and reporting surprises/regressions in Arti's bridge and PT support.
* Move to Manifest v3 for Snowflake/ extension.
* Deprecate BridgeDB.
* Support the Tor community team with anti-censorship issues

## The following were our goals for 2023:

- make Tor accessible in China
- make Tor accessible in Iran
- detect and categorize attempts to censor Tor
- improve the design and reliability of our software
- release our data and software for use by the broader anti-censorship community
- improve the performance of Snowflake so that Tor bootstraps reliably on a mobile phone in China
- deploy TapDance and Conjure as high collateral damage PT
- commit to a design for a reputation-based bridge distribution system
- deploy probes in areas that are likely to censor Tor and collect pack captures and probe results for storage and analysis
- provide OONI with suggestions for improving the accuracy of OONI's Tor tests
- summarize the details of Tor blocking events with data from our probes and volunteers
- sanitize, publish and archive the results of our Tor reachability probes
- complete our documentation for each of our tools so that other organizations can run their own anti-censorship infrastructure
- Meek deprecation.
- Think of priorities for the team so we can write the next proposal to get a sponsor.
- Discovering and reporting surprises/regressions in Arti's bridge and PT support.
- Time to fork obfs4proxy and maintain it ourselves.
- bridge operator usability (obfs4proxy, snowflake, etc): deb packages in the right place, C-Tor patches to include pluggable transport version, etc.

### Nice to have:

- Q2: Content for the developer portal.
- Capturing what we did in 2022 toward our priorities, for our future, and for visibility from other teams? Info is in SOTO 2022, anti-censorship team meeting pads, GitLab issues, and sponsor reports.
- Onbasca work to test bridges and bridge performance https://gitlab.torproject.org/tpo/network-health/onbasca/-/issues/130
- The "meta signaling channel app" idea ( https://gitlab.torproject.org/tpo/anti-censorship/team/-/issues/111)
- Are there external groups that are doing things we want to use, re-use, rely on, advertise, etc? Like, Geneva and its censorship assessment tools. Two-six and their apps?
- How to structure our future work so that our progress is more evident to our community, in a way that doesn't involve extra bureaucracy/work for us? E.g. other open-source tools use milestones in their issue tracker in a way that organizes tickets by topics, and we don't do that, but we could!

## The following were our goals for 2022:

- **make Tor accessible in China**
- **detect and categorize attempts to censor Tor**
- **improve the design and reliability of our software**
- **release our data and software for use by the broader anti-censorship community**
- **improve the performance of Snowflake so that Tor bootstraps reliably on a mobile phone in China <-- work mostly done by dcf**
- **deploy TapDance and Conjure as high collateral damage PT**
- **commit to a design for a reputation-based bridge distribution system**
- ~~streamline our private bridge setup and distribution process~~ <-- mostly handled by community team, check with ggus how they are handling it [see https://gitlab.torproject.org/tpo/community/support/-/issues/28526 for original motivation]
- **deploy probes in areas that are likely to censor Tor and collect packet captures and probe results for storage and analysis**
- **provide OONI with suggestions for improving the accuracy of OONI's Tor tests**
- **summarize the details of Tor blocking events with data from our probes and volunteers**
- ~~add more user metrics based monitoring and alert rules using prometheus~~
- ~~deploy rdsys as the new backend of bridgedb~~
- ~~ensure that key infrastructure can survive machine outages and restarts~~
- ~~remove hacky shims necessary for moat (decided to keep them, but did work on it)~~
- ~~future improve the snowflake library api to allow easy integration of snowflake with other tools~~
- ~~sanitize, publish and archive the results of our Tor reachability probes~~
- **complete our documentation for each of our tools so that other organizations can run their own anti-censorship infrastructure**

## The following were our goals for 2021:

- ~~Improve reputation based bridge distribution (e.g. Salmon)~~
- Better bridge distribution strategies (e.g. Conjure, Salmon)
    1. conjure first priority <---
    2. salmon second priority
      Task 4.1 Let's build a plan for how we're going to start the Salmon research work (Task 4).
- Make GetTor more reliable and reduce maintenance burden <---
    Rewrite gettor in Go to include it in rdsys <-- Evaluate if it makes sense to integrate
    The hard part of keeping gettor working in the past has been keeping all the files we distribute up-to-date. So let's not forget that part. :)
- Complete integration of bridgedb into the more general rdsys <---
    Integrating, not replacing bridgedb.
- Improve the performance of Snowflake for users in Asia
    The destination we want is that users use multiple Snowflakes and it helps
- Snowflake network health -- have enough of the right snowflakes, have volunteers happy
    Provide more feedback to users that run Snowflake proxies
- Tor reachability from various countries
    Task 1.1 Automated scans to create a reachability dataset
    Task 1.1 Baselines before the blocking event -- get people used to running the tools and sending us the outputs. Maybe work with UX team to streamline this process, identify usability gaps in the tools.


## The following were our goals for 2020:

- ~~We want to have more comprehensive BridgeDB metrics.~~
- ~~We will improve BridgeDB's broken CAPTCHA system.~~
- ~~We will monitor all critical components of the team's infrastructure.~~
- Re-engineer GetTor to use multiple distributors (in addition to email)
- We want to have accurate and safely-collected statistics of GetTor use.
- ~~Snowflake should allow clients to start browsing quickly and with reasonable bandwidth~~
- ~~We have happy Snowflake proxy volunteers that remain active.~~
- ~~We have a good understanding of censorship events and performance of snowflake in different countries.~~
- ~~Build feedback loop between BridgeDB and OONI.~~
- ~~Experiment with a modular BridgeDB architecture.~~

