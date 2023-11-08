# About us

Welcome to the anti-censorship team page. The anti-censorship team is a group of people who make Tor reachable anywhere in the world. We analyze censorship attempts and develop technology to work around these censorship attempts. One of the reasons we are not listing the names of the team members here is because we want to keep the team open to everyone. You're on the team if you're participating in discussions and development.

Excited about joining the team? Here is more information on how to get started.

# IRC/matrix meetings schedule

We use ​IRC for our weekly meetings and we meet on the ​OFTC network in the #tor-meeting channel, this channel is also accesible over the matrix channel at [#tor-meeting:matrix.org](https://matrix.to/#/#tor-meeting:matrix.org). The meeting takes place each Thursday at 16:00 UTC and typically lasts for an hour. Sometimes, we have to cancel our meeting but we announce cancellations on [our ​mailing list](https://lists.torproject.org/cgi-bin/mailman/listinfo/anti-censorship-team). Besides, our [​meeting pad](https://pad.riseup.net/p/tor-anti-censorship-keep) always shows the date of the next meeting.

If you want to get involved in Tor's anti-censorship work, try to show up to the team meeting! To get an idea of what we discuss in our meetings, take a look at our ​[meeting pad](https://pad.riseup.net/p/tor-anti-censorship-keep). In a nutshell, we use our weekly meetings to:

* Make announcements to the team.
* Discuss topics like our development roadmap, team processes, or code architecture.
* Ask for help with whatever we're working on.
* Coordinate code review.

People on the anti-censorship team use the pad to keep track of what they did the past week, what they plan to do next week, and what they need help with. If you missed a meeting, fret not! We post log files of our meetings on the ​[tor-project](https://lists.torproject.org/cgi-bin/mailman/listinfo/tor-project) mailing list, typically with the string "Anti-censorship meeting notes" in the email's subject line.

We use the string "anti-censorship-team" on IRC to reach all team members, e.g. "anti-censorship-team: take a look at bug #1234". Be sure to configure a highlight in your IRC client for this string.

# IRC/matrix channel

For direct communication we use the #tor-anticensorship IRC channel in the OFTC network, also available in the matrix network as [#tor-anticensorship:matrix.org](https://matrix.to/#/#tor-anticensorship:matrix.org). Is a good place to hang out or come with questions and comments.

# Mailing list

For asynchronous communication, we use our ​[anti-censorship-team](https://lists.torproject.org/cgi-bin/mailman/listinfo/anti-censorship-team) mailing list. The list is [​publicly archived](https://lists.torproject.org/pipermail/anti-censorship-team/) and available for anyone to sign up, so feel free to participate! Among other things, we use this mailing list to coordinate meetings, send announcements, and discuss all matters related to the anti-censorship team. Note that for development-related topics, we use the ​[tor-dev](https://lists.torproject.org/cgi-bin/mailman/listinfo/tor-dev) mailing list.

# Priorities

## Priorities for 2023

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

## Nice to have:

- Q2: Content for the developer portal.
- Capturing what we did in 2022 toward our priorities, for our future, and for visibility from other teams? Info is in SOTO 2022, anti-censorship team meeting pads, GitLab issues, and sponsor reports.
- Onbasca work to test bridges and bridge performance
https://gitlab.torproject.org/tpo/network-health/onbasca/-/issues/130
- The "meta signaling channel app" idea (https://gitlab.torproject.org/tpo/anti-censorship/team/-/issues/111)
- Are there external groups that are doing things we want to use, re-use, rely on, advertise, etc? Like, Geneva and its censorship assessment tools. Two-six and their apps?
- How to structure our future work so that our progress is more evident to our community, in a way that doesn't involve extra bureaucracy/work for us? E.g. other open-source tools use milestones in their issue tracker in a way that organizes tickets by topics, and we don't do that, but we could!

[Priorities for previous years](previous-priorities)

## Roadmap

[Roadmaps for previous years](previous-roadmaps)

## Active Sponsors and Contracts

* [Sponsor 96: Rapid Expansion of Access to the Uncensored Internet through Tor in China, Hong Kong, & Tibet](https://gitlab.torproject.org/groups/tpo/-/milestones/24)
* [Sponsor 150: Deprecate BridgeDB](https://gitlab.torproject.org/groups/tpo/anti-censorship/-/issues/?label_name%5B%5D=Sponsor%20150)

## Projects that the team maintains

* [rdsys](https://gitlab.torproject.org/tpo/anti-censorship/rdsys)
* [Lox](https://gitlab.torproject.org/tpo/anti-censorship/lox)
* [Pluggable Transports](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports)
* [Snowflake](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/wikis/home)
* [Snowflake Mobile](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake-mobile/-/wikis/home)
* [BridgeDB](https://gitlab.torproject.org/tpo/anti-censorship/bridgedb)
* [Services](ServicesAntiCensorship) for which we have [survival guides](https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Survival-Guides)

# Becoming a volunteer

Thanks for volunteering with us! There are many things that we need your help with:

* Do you think that Tor (or one of its pluggable transports) is blocked in your country or network? Let us know!
* Do you know how to code? Come help us improve one of our software projects! See below for more details.
* We maintain lots of documentation which regularly needs updates and new content.
* Do you have a background in UX? We maintain user-facing software whose user experience matters to us.

The best way to get involved is to visit our weekly IRC meeting (see above). Tell us your background and interests and we will find a project for your to get started.

# Other interesting communities

Other communities where there are conversations around anti-censorship and we keep connection with are:
* [OONI slack](https://slack.ooni.org/)
* [net4people forum](https://github.com/net4people/bbs/issues/)
* [ntc.party forum](https://ntc.party/)

# Archive

https://trac.torproject.org/projects/tor/wiki/org/teams/AntiCensorshipTeam
