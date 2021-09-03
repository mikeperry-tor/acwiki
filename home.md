# About us

Welcome to the anti-censorship team page. The anti-censorship team is a group of people who make Tor reachable anywhere in the world. We analyze censorship attempts and develop technology to work around these censorship attempts. One of the reasons we are not listing the names of the team members here is because we want to keep the team open to everyone. You're on the team if you're participating in discussions and development.

Excited about joining the team? Here is more information on how to get started.

# IRC meetings schedule

We use ​IRC for our weekly meetings and we meet on the ​OFTC network in the #tor-meeting channel. The meeting takes place each Thursday at 16:00 UTC and typically lasts for an hour. ​This page can tell you what time this is in your part of the world. Sometimes, we have to cancel our meeting but we announce cancellations on [our ​mailing list](https://lists.torproject.org/cgi-bin/mailman/listinfo/anti-censorship-team). Besides, our [​meeting pad](https://pad.riseup.net/p/tor-anti-censorship-keep) always shows the date of the next meeting.

If you want to get involved in Tor's anti-censorship work, try to show up to the team meeting! To get an idea of what we discuss in our meetings, take a look at our ​[meeting pad](https://pad.riseup.net/p/tor-anti-censorship-keep). In a nutshell, we use our weekly meetings to:

* Make announcements to the team.
* Discuss topics like our development roadmap, team processes, or code architecture.
* Ask for help with whatever we're working on.
* Coordinate code review. 

People on the anti-censorship team use the pad to keep track of what they did the past week, what they plan to do next week, and what they need help with. If you missed a meeting, fret not! We post log files of our meetings on the ​[tor-project](https://lists.torproject.org/cgi-bin/mailman/listinfo/tor-project) mailing list, typically with the string "Anti-censorship meeting notes" in the email's subject line.

We use the string "anti-censorship-team" on IRC to reach all team members, e.g. "anti-censorship-team: take a look at bug #1234". Be sure to configure a highlight in your IRC client for this string. 

# Mailing list

For asynchronous communication, we use our ​[anti-censorship-team](https://lists.torproject.org/cgi-bin/mailman/listinfo/anti-censorship-team) mailing list. The list is [​publicly archived](https://lists.torproject.org/pipermail/anti-censorship-team/) and available for anyone to sign up, so feel free to participate! Among other things, we use this mailing list to coordinate meetings, send announcements, and discuss all matters related to the anti-censorship team. Note that for development-related topics, we use the ​[tor-dev](https://lists.torproject.org/cgi-bin/mailman/listinfo/tor-dev) mailing list. 

# Road-mapping goals


## OKRs for Q3 2021 to Q2 2022

- make Tor accessible in China
- detect and categorize attempts to censor Tor
- improve the design and reliability of our software
- release our data and software for use by the broader anti-censorship community
- improve the performance of Snowflake so that Tor bootstraps reliably on a mobile phone in China
- deploy TapDance and Conjure as high collateral damage PT 
- commit to a design for a reputation-based bridge distribution system
- streamline our private bridge setup and distribution process
- deploy probes in areas that are likely to censor Tor and collect pack captures and probe results for storage and analysis
- provide OONI with suggestions for improving the accuracy of OONI's Tor tests
- summarize the details of Tor blocking events with data from our probes and volunteers
- add more user metrics based monitoring and alert rules using prometheus
- deploy rdsys ad the new backend of bridgedb
- ensure that key infrastructure can survive machine outages and restarts
- remove hacky shims necessary for moat
- future improve the snowflake library api to allow easy integration of snowflake with other tools
- sanitize, publish and archive the results of our Tor reachability probes
- complete our documentation for each of our tools so that other organizations can run their own anti-censorship infrastructure


## The following are our goals for 2021:

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

You can follow up our roadmap in this [kanban board](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/groups/tpo/anti-censorship/-/boards). 

## Active Sponsors and Contracts

* [RACE (Resilient Anonymous Communication for Everyone)](Sponsor-RACE)
* [Empowering Communities in the Global South to Bypass Censorship ](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/trac/-/issues/31265)
* [Sponsor 96: Rapid Expansion of Access to the Uncensored Internet through Tor in China, Hong Kong, & Tibet](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/groups/tpo/-/milestones/24)

## Projects that the team maintains

* [Gettor](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/gettor-project/gettor/-/wikis/home)
* [BridgeDB](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/bridgedb)
* [rdsys](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/rdsys)
* [Pluggable Transports](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/pluggable-transports)
* [Snowflake](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/pluggable-transports/snowflake/-/wikis/home)
* [Snowflake Mobile](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/pluggable-transports/snowflake-mobile/-/wikis/home)

# Becoming a volunteer

Thanks for volunteering with us! There are many things that we need your help with:

* Do you think that Tor (or one of its pluggable transports) is blocked in your country or network? Let us know!
* Do you know how to code? Come help us improve one of our software projects! See below for more details.
* We maintain lots of documentation which regularly needs updates and new content.
* Do you have a background in UX? We maintain user-facing software whose user experience matters to us. 

The best way to get involved is to visit our weekly IRC meeting (see above). Tell us your background and interests and we will find a project for your to get started. 


# Archive

https://trac.torproject.org/projects/tor/wiki/org/teams/AntiCensorshipTeam