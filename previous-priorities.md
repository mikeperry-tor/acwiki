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

