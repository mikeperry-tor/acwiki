This is an attempt to document what constitutes an emergency for the anti-censorship team and who to contact for support.

## Alert systems

We have been slowly expanding our automated alert system to notify team members of critical outages. Our [anti-censorship alerts mailing list](https://lists.torproject.org/pipermail/anti-censorship-alerts/) collects email alerts from a variety of services. This list is public and the current list admin is @cohosh.

We have two automated systems for anti-censorship alerts:

- [monit](https://gitlab.torproject.org/tpo/anti-censorship/monit-configuration) is a monitoring service that tests the reachability or function of our infrastructure and default circumvention tools and services
- [prometheus](https://gitlab.torproject.org/tpo/tpa/prometheus-alerts) is a monitoring service that collects and issues alerts based on usage metrics of Snowflake and our bridge distribution infrastructure.

## Response levels

### Code Red (emergencies)

"Code red" incidents should be handled as soon as the team is made aware of them. These include outages for our critical infrastructure or single points of failure for any of our circumvention tools. Some examples of code red events:

- Our bridge distribution service(s) are down. This includes:
  - BridgeDB
  - rdsys
  - moat
  - any of our bridge distributors
- GetTor, a service for downloading Tor Browser in places that censor our website, is down
- Snowflake outages. This may include the broker, the bridge, the NAT probe check, or issues with our proxy pool

### Code Yellow (non-emergency outages)

"Code yellow" incidents should be dealt with by team members as soon as possible but are not considered emergencies. This may be because they are not single points of failure and shouldn't on their own cause a disruption of the usage of our circumvention tools.

Some examples:
- Default obfs4 bridge outages
- Out of date GetTor binaries
- Non-critical drop in available bridges or snowflake proxies

### Censorship Events

Censorship events do not cleanly belong in code red or code yellow because each event is different. Actionable steps to respond to a new censorship event can take priority over other work. The team should work with the community team to diagnose the event, find existing working solutions, and brainstorm improvements of changes that can be made to overcome the blocking.

## Contact information

All team members should monitor the [anti-censorship alerts mailing list](https://lists.torproject.org/pipermail/anti-censorship-alerts/). If there is a critical outage that **has not been broadcasted on that mailing list**, the team can be reached by sending an email to the public [anti-censorship-team](https://lists.torproject.org/pipermail/anti-censorship-team/) list at `anti-censorship-team@lists.torproject.org`.

They may also be reached on the OFTC IRC server in `#tor-dev` or `#tor-project`.
To highlight the whole team, use the keyword `anti-censorship-team`. Otherwise, the following team members are usually around:
- `meskio`
- `shelikhoo`
- `cohosh`
- `arma`