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

# Priorities

## Priorities for 2022

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

[Priorities for previous years](previous-priorities)

## Roadmap

## Q3

* sponsor 30
  * O2.3.1 - Develop new and/or improve existing bridge selection and distribution strategies based on data collected about successful, effective methods per evaluation during O1.1.
    * conjure (2 months) (cecylia) <-- maintenance in Q3.
  * https://gitlab.torproject.org/groups/tpo/anti-censorship/-/boards?label_name[]=Sponsor%2030

* sponsor 96
  * O1.1.1 Prepare the Snowflake system for a surge in operators and users. <-- deployed snowflake broker in Q2.
    * next in Q3:
      * proxies need to be updated to next version.
      * setup new bridges.
  * O1.3: Implement bridges with pluggable transport HTTPT support. <--Q3 (Shell)
  * O1.4: Increase the number of active obfs4 and HTTPT bridges.
    * O1.4.3: Monitor bridge health.   <-- Meskio with Gus
      * obfsproxy security issues - meskio Q3
  * O2.2: Deploy improved bridge distribution systems.
    * Salmon based design: Roger on Q3.
    * rdsys DB for bridges (meskio) Q3
  * O2.3: React and steer our response to censorship. (Shell) - Q3
    * vantage points in specific places
  * O4.1: Localize all UI modified in this project. (meskio) Q3
    * Q3 Localize gettor.
    * Q3 Bot Telegram.
  * O4.3: Modify GetTor so that it can distribute Tor Browser via messaging apps
    * support onionsprout deployment - It was deployed in Q2
    * get gettor into rdsys - implemented in Q2. Deployment in Q3.
  * https://gitlab.torproject.org/groups/tpo/anti-censorship/-/boards?label_name[]=Sponsor%2096

* sponsor 28 (itchyonion) & extension (shel)
  * probetest centralized log collection (shell) Q3


## Q2

* sponsor 30 O2.3.1 - Develop new and/or improve existing bridge selection and distribution strategies based on data collected about successful, effective methods per evaluation during O1.1. - Implement and deploy conjure - Cecylia
* sponsor 96 O1.1.1 Prepare the Snowflake system for a surge in operators and users. - shel
* sponsor 96 O1.2: Increase the number of Snowflake bridges. - shel
* sponsor 96 O1.2.2: Scale Tor reachability through mobile Snowflakes.
* sponsor 96 O1.3: Implement bridges with pluggable transport HTTPT support.
* sponsor 96 O1.4: Increase the number of active obfs4 and HTTPT bridges.
* sponsor 96 O1.4.3: Monitor bridge health.   - Meskio with Gus
* sponsor 96 obfsproxy security issues - meskio
* sponsor 96  O2.2: Deploy improved bridge distribution systems.
* sponsor 96 Salmon based design :) --- Roger
* sponsor 96 O2.3: React and steer our response to censorship. - Shel
* sponsor 96 O4.3: Modify GetTor so that it can distribute Tor Browser via messaging apps - Meskio
* sponsor 96 get gettor into rdsys - Q2
* sponsor 28 - itchyonion
* sponsor 928 probetest centralized log collection (shel)
* sponsor 125 dynamic bridges


### Q1

* s30 O2.3.1 - Develop new and/or improve existing bridge selection and distribution strategies based on data collected about successful, effective methods per evaluation during O1.1.
* s30 conjure (2 months) - cecylia starting in March
  * server side : the university
  * start talking again with eric and his team
  * define server side for them to setup bridge (documentation)
  * client side : around 2 weeks
    * write a conjure client based on their specification (tor pt part with their client side library)
  * staging/testing
    * deployment
  * add it to alpha version of TB
  * metrics (discuss with people that maintain conjure bridge)
    * how many clients are connected

* s96 O1.1.1 Prepare the Snowflake system for a surge in operators and users. <-- scale in Q1/Q2 2022 <-- shell
* s96 O1.2: Increase the number of Snowflake bridges.
* s96 O1.2.2: Scale Tor reachability through mobile Snowflakes. <-- support to GP
* s96 O1.4: Increase the number of active obfs4 and HTTPT bridges.
* s96 O1.4.3: Monitor bridge health.   <-- Meskio/Shel with Gus
* s96 O2.1: Make it easier for humans & harder for censors to get bridges from moat distributor.  <-- Q1
* s96 O2.2: Deploy improved bridge distribution systems.
* s96 O2.2.2: Deploy next generation bridge distribution system (rdsys) <-- meskio -
* s96 O2.3: React and steer our response to censorship. <-- shel
* s96 O3.1: Improve automatic censorship detection during bootstrapping in Tor Browser (desktop and Android). <-- meskio

* sponsor 28 & extension: we will have the new developer full time into this project.
* sponsor 28 - Improve the performance of Snowflake for users in Asia (cecylia)

You can follow up what we are working on in this [kanban board](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/groups/tpo/anti-censorship/-/boards).

## Active Sponsors and Contracts

* [RACE (Resilient Anonymous Communication for Everyone)](Sponsor-RACE)
* [Empowering Communities in the Global South to Bypass Censorship ](https://gitlab.torproject.org/tpo/anti-censorship/team/-/issues/12)
* [Sponsor 96: Rapid Expansion of Access to the Uncensored Internet through Tor in China, Hong Kong, & Tibet](https://gitlab.torproject.org/groups/tpo/-/milestones/24)

## Projects that the team maintains

* [Gettor](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/gettor-project/gettor/-/wikis/home)
* [BridgeDB](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/bridgedb)
* [rdsys](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/rdsys)
* [Pluggable Transports](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/pluggable-transports)
* [Snowflake](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/pluggable-transports/snowflake/-/wikis/home)
* [Snowflake Mobile](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/pluggable-transports/snowflake-mobile/-/wikis/home)
* [Services](ServicesAntiCensorship)

# Becoming a volunteer

Thanks for volunteering with us! There are many things that we need your help with:

* Do you think that Tor (or one of its pluggable transports) is blocked in your country or network? Let us know!
* Do you know how to code? Come help us improve one of our software projects! See below for more details.
* We maintain lots of documentation which regularly needs updates and new content.
* Do you have a background in UX? We maintain user-facing software whose user experience matters to us.

The best way to get involved is to visit our weekly IRC meeting (see above). Tell us your background and interests and we will find a project for your to get started.


# Archive

https://trac.torproject.org/projects/tor/wiki/org/teams/AntiCensorshipTeam
