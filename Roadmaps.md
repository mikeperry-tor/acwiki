<div>

## <span dir="">Review Q3 - what did we do? </span>

</div>
<div>

* <span dir="">Make GetTor more reliable and reduce maintenance burden</span>
* <span dir="">Rewrite gettor in Go to include it in rdsys </span>
* <span dir="">Complete integration of bridgedb into the more general rdsys -- continue in Q4 2021</span>
* <span dir="">Improve the performance of Snowflake for users in Asia The destination we want is that users use multiple Snowflakes and it helps -- continue in Q4 2021</span>
  * <span dir="">Cecylia was tuning the underlying KCP params.</span>
  * <span dir="">Submitted patches to Shadow so Snowflake can now run inside Shadow.</span>
  * <span dir="">The Shadow network model has a bunch of details about Tor relays, but not so many details about Snowflakes. So we need to extend Shadow's network model there.</span>
* <span dir="">Snowflake network health -- have enough of the right snowflakes, have volunteers happy Provide more feedback to users that run Snowflake proxies  -- continue in Q4 2021</span>
  * <span dir="">We're doing well on our own monitoring, but haven't done much yet on helping users understand their contribution.</span>
* <span dir="">Tor reachability from various countries Task 1.1 Automated scans to create a reachability dataset Task 1.1 Baselines before the blocking event -- get people used to running the tools and sending us the outputs. Maybe work with UX team to streamline this process, identify usability gaps in the tools.-- continue in Q4 2021</span>
* <span dir="">The Tor auto connect work, with the json censorship map and the new Tor Browser flow. -- continue in Q4 2021</span>

### <span dir="">What worked well?</span>

</div>* <span dir="">We did a bunch of really cool work</span>
* <span dir="">Meskio has jumped in and made a lot of progress on the s30 side of things</span>
* <span dir="">Coordination with other teams for the auto connect work.</span>
* <span dir="">S28 program manager remains happy with us. The s30/s96 ones too I think.</span>
* <span dir="">We have a promising third dev on the way. we hired other dev!  \\o/</span>
* <span dir="">We have actual results from our censorship assessment vantage points!</span>

<div>

### <span dir="">What could work better?</span>

</div>* <span dir="">Cecylia is still a bottleneck on the racecar 'assessments' + race dev</span>
* <span dir="">We have a bunch of external groups who would love to coordinate with us but we don't have time/energy to start those coordination.</span>
* <span dir="">We have a lot of different irons in the fire right now and few people, so we are spread over many different code repositories and sponsor work and it's hard to prioritize between projects and keep up with new events</span>
* <span dir="">I don't know whether our censorship assessment vantage points are collecting data today or dead or what. :)</span>
* <span dir="">We have notifications from our monitoring infrastructure but we don't really have habits for how to react to the notifications.</span>
* <span dir="">Seems smart for us to change our notifications to only send email when it's actionable, e.g. if it's failed three tests in a row or something.</span>
* <span dir="">It remains tricky, especially with our limited capacity, to prioritize _between_ sponsors. Like, we do individual sponsor meetings, but it seems like all the prioritized tasks are for whichever sponsor meeting we just had. That thrashing results in a lot of changing priorities.</span>
  * <span dir=""> We will use the Thursday gaba-cecylia syncs to do a weekly check-in for the overall roadmap for the week.</span>

## <span dir="">ROADMAP 2021 Q4</span>

### <span dir="">MUST HAVE</span>

<div>

* <span dir="">sponsor 30</span>
  * <span dir="">O2.1.3 - Identify which bridge selection and distribution methods are most used in targeted regions.</span>

</div>
<div>

* <span dir="">O2.2 </span>[<span dir="">http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/groups/tpo/-/milestones/7</span>](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/groups/tpo/-/milestones/7)

</div>
<div>

* <span dir="">O2.3.1 - Develop new and/or improve existing bridge selection and distribution strategies based on data collected about successful, effective methods per evaluation during O1.1.</span>

</div>
<div>

* <span dir="">3.3 automatic anti-censorship </span>[<span dir="">http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/groups/tpo/-/milestones/15</span>](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/groups/tpo/-/milestones/15)

</div>
<div>


</div>
<div>

* <span dir="">sponsor 96</span>

</div>
<div>

* <span dir="">O1.1.1 Prepare the Snowflake system for a surge in operators and users.</span>

</div>
<div>

* [<span dir="">https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/issues/40026</span>](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/issues/40026)

</div>
<div>

* [<span dir="">http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/pluggable-transports/snowflake/-/issues/40066</span>](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/pluggable-transports/snowflake/-/issues/40066)

</div>
<div>

* <span dir="">  - Make Snowflake DoS contingency plan (see Aug 15 mail, "Subject: okr thoughts") </span>[<span dir="">http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/pluggable-transports/snowflake/-/issues/25593</span>](http://eweiibe6tdjsdprb4px6rqrzzcsi22m4koia44kc5pcjr7nec2rlxyad.onion/tpo/anti-censorship/pluggable-transports/snowflake/-/issues/25593)

</div>
<div>

* <span dir="">O2.2.2: Deploy next generation bridge distribution system (rdsys) <-- meskio</span>

</div>
<div>

* <span dir="">O2.3: React and steer our response to censorship. \[in China in particular\]</span>

</div>
<div>

* <span dir="">Needs to be decided who is going to take this work.</span>

</div>
<div>

* <span dir="">Needs some scoping and planning.</span>

</div>
<div>

* <span dir="">Plausible to imagine that it will be a combination of Cecylia and Xiaokang, and then each of them will do other dev things too. Check with Xiaokang when he starts what he wants to focus on.</span>

</div>
<div>

* <span dir="">One challenge to keep in mind here: we need new distribution strategies, and/or PTs, because the ones we have today are not enough in China.</span>

</div>
<div>

* <span dir="">O3.1: Improve automatic censorship detection during bootstrapping in Tor Browser (desktop and Android). <-- meskio</span>

</div>
<div>

* <span dir="">~~O3.2: Deploy Snowflake as a bridge option in Tor Browser stable.~~</span>

</div>
<div>

* <span dir="">O4.3: Modify GetTor so that it can distribute Tor Browser via messaging apps. - Start in November  or we can move to 2022. </span>

</div>
<div>

* <span dir="">E.g. Telegram (there is some partial code, it is not yet really finished)</span>

</div>
<div>

* <span dir="">Check with Gus and TGP about which messaging apps to use here. Need to pick one(s) that are both safe and popular, e.g. maybe not qq.</span>

</div>
<div>


</div>
<div>

* <span dir="">sponsor 28 & extension</span>

</div>
<div>

* <span dir="">Attend Snowflake surge of users</span>

</div>
<div>

* <span dir="">Be available for the October test event. (Most dev already done for it woo.)</span>

</div>
<div>

* <span dir="">Prep for and present at the December PI meeting (Dec 7-9)</span>

</div>
<div>

* <span dir="">Present a map of circumvention methods that work / are needed in each region.</span>

</div>
<div>

* [<span dir="">https://gitlab.torproject.org/tpo/anti-censorship/censorship-analysis/-/issues/40009</span>](https://gitlab.torproject.org/tpo/anti-censorship/censorship-analysis/-/issues/40009)

</div>
<div>

* <span dir="">Have data sets from our own measurements, and have shared them at least with other researchers.</span>

</div>
<div>

* <span dir="">Have Snowflake performance test done with Shadow.</span>

</div>
<div>

* <span dir="">Have automated public graphs visualizing our China and Turkey and Canada results.</span>

</div>
<div>

* <span dir="">This will also help with awareness of whether the tests are still running.</span>

</div>
<div>

* [<span dir="">https://gitlab.torproject.org/tpo/anti-censorship/censorship-analysis/-/issues/40020</span>](https://gitlab.torproject.org/tpo/anti-censorship/censorship-analysis/-/issues/40020)

</div>
<div>

* <span dir="">Follow along with, and investigate, blocking events in the real world.</span>

</div>
<div>

* [<span dir="">https://gitlab.torproject.org/tpo/anti-censorship/censorship-analysis/-/issues/40004</span>](https://gitlab.torproject.org/tpo/anti-censorship/censorship-analysis/-/issues/40004)

</div>
<div>

* <span dir="">Add more tools to the suite of assessment tools we run.</span>

</div>
<div>

* <span dir="">Emma, marco, full vanilla Tor bootstrap, something for active probing.</span>

</div>
<div>

* <span dir="">Improve the output data format:</span>

</div>
<div>

* [<span dir="">https://gitlab.torproject.org/tpo/anti-censorship/censorship-analysis/-/issues/40022</span>](https://gitlab.torproject.org/tpo/anti-censorship/censorship-analysis/-/issues/40022)

</div>
<div>


</div>
<div>

<span dir="">OKRs:</span>

</div>
<div>

* <span dir="">Make Tor accessible in China</span>

</div>
<div>

* <span dir="">improve the performance of Snowflake so that Tor bootstraps reliably on a mobile phone in China </span>[<span dir="">https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/issues/40026</span>](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/issues/40026)

</div>
<div>

* <span dir="">Detect and categorize attempts to censor Tor</span>

</div>
<div>

* <span dir="">deploy probes in areas that have/do/are likely to censor Tor and collect packet captures and probe results for storage and analysis</span>

</div>
<div>

* <span dir="">Improve the design and reliability of our software</span>

</div>
<div>

* <span dir="">add more user metrics-based monitoring and alert rules using prometheus</span>

</div>
<div>


</div>
<div>

<span dir="">### NICE TO HAVE</span>

</div>
<div>


</div>
<div>

* <span dir="">sponsor 30</span>

</div>
<div>

* <span dir="">starting with conjure</span>

</div>
<div>

* <span dir="">- Conjure integration progress?</span>

</div>
<div>


</div>
<div>

<span dir="">- Coordinate with other external research projects, to share plans and to try to get them in on helping us with our tasks:</span>

</div>
<div>

<span dir="">\[Roger would be happy to launch any of these conversations, but if we have no capacity to follow up on them, it seems sort of silly to launch them. But also we need to get other groups helping us, since we can't do everything ourselves. How to get out of being stuck?\]</span>

</div>
<div>

<span dir="">    - Nick Feamster wants to do data analysis on our censorship assessment data set</span>

</div>
<div>

<span dir="">    - Roya and Paul Pearce could run spooky-scan on the default bridge addresses</span>

</div>
<div>

<span dir="">    - Eric Wustrow wants to run obfs4+conjure for us. <---  end of this quarter or next quarter</span>

</div>
<div>

<span dir="">    - Jed Crandall's student wants to tell us about his circumvention ideas re VPNs</span>

</div>
<div>

<span dir="">    - Dave Levin (Geneva) wants to brainstorm how Geneva could be useful to us</span>

</div>
<div>

<span dir="">    - Ian Goldberg has a student working on a better Salmon design <----- next year</span>

</div>
<div>


</div>
<div>

<span dir="">- S28-extension tasks (should move some of these to MUST HAVE above):</span>

</div>
<div>

<span dir="">  - Coordinate with TPA/metrics to get a place to store the measurement dataset.</span>

</div>
<div>

<span dir="">  - Have a plan for how we're going to keep the docker installs up to date.</span>

</div>
<div>

<span dir="">  - Get a few more people running the docker image and generating data in their location</span>

</div>
<div>

<span dir="">  </span>

</div>
<div>

<span dir="">- Snowflake</span>

</div>
<div>

<span dir="">  - Provide more feedback to users that run Snowflake proxies</span>

</div>
<div>

<span dir="">  - We could integrate dcf's new amp cache implementation as an alternative to Fastly</span>

</div>
<div>


</div>
<div>


</div>
<div>

* <span dir="">Detect and categorize attempts to censor Tor</span>

</div>
<div>

* <span dir="">provide OONI with suggestions for improving the accuracy of OONI’s Tor tests</span>

</div>
<div>

* <span dir="">Improve the design and reliability of our software</span>

</div>
<div>

* <span dir="">add more user metrics-based monitoring and alert rules using prometheus</span>

</div>
<div>

* <span dir="">ensure that key infrastructure (BridgeDB, Snowflake broker, etc) can survive machine outages and restarts</span>

</div>
<div>

* <span dir="">remove "hacky shims" necessary for Moat</span>

</div>
<div>

* <span dir="">Release our data and software for use by the broader community</span>

</div>
<div>

* <span dir="">further improve the Snowflake library API to allow easy integration of Snowflake with other tools</span>

</div>
<div>

* <span dir="">complete our documentation for each of our tools so that other organizations can run their own anti-censorship infrastructure</span>

</div>