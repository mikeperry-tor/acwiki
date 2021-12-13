The following table lists services run on torproject
infrastructure. Corresponding onion services are listed on
<https://onion.torproject.org/>.

Service admins are part of tor project sys admins team. For a rough
description of what sys admin and services admin do, please have a
look [here](policy/tpa-rfc-2-support#service-admins).

The Service Admins that are part of the Anti-censorship team maintain the following list of Tor Services.

| Service            | Purpose                                                       | URL                                     | Maintainers                | Documented | Auth                          |
|--------------------|---------------------------------------------------------------|-----------------------------------------|----------------------------|------------|-------------------------------|
| [bridgedb][]       | web application and email responder to learn bridge addresses | <https://bridges.torproject.org/>       | cohosh, meskio             | 20%        | no                            |
| [bridgestrap][]    | service to tests bridges                                      | `https://bridges.torproject.org/status` | cohosh, meskio             | 20%        | no                            |
| [gettor][]         | email responder handing out packages                          | <https://gettor.torproject.org>         | cohosh, meskio             | 10%        | no                            |
| [rdsys][]          | Distribution system for circumvention proxies                 | N/A                                     | cohosh, meskio             | 20%        | no                            |
|
| [snowflake][]      | Pluggable Transport using WebRTC                              | <https://snowflake.torproject.org/>     | cohosh, meskio             | 20%        | no                            |

[bridgedb]: https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Survival-Guides/BridgeDB-Survival-Guide
[bridgestrap]: https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Survival-Guides/Bridgestrap-Survival-Guide
[gettor]: https://gitlab.torproject.org/tpo/anti-censorship/gettor-project/gettor/-/wikis/Gettor-Service-Operator-HowTo
[moat]: https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Survival-Guides/Moat-Survival-Guide
[rdsys]: https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Survival-Guides/Rdsys-Survival-Guide
[snowflake]: https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Survival-Guides/Snowflake-Broker-Survival-Guide