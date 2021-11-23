This is a overview of core tor issues that impact the anti-censorship team. This is the result of roadmap planning with the network team and can be used as a reference and updated as needed.

Prioritized issues are grouped by topic.

## IPv6 support

Our current story is that IPv6 only really works for vanilla bridges, because while you can only specify a single IP address in the "transport" line of extrainfo docs, you can specify multiple OR addresses. We have a suggested workaround (rdsys#73) to optimistically test all IPs in our OR addresses with bridgestrap, but this isn't a great solution. Ideally, we could include multiple transport lines in the extrainfo doc.

- Fix IPv6 bridges with a private/dynamic IPv4 address (#4847)
- Multiple ServerTransportListenAddr entries should be allowed per transport (#11211)
- Make IPv6-only bridges work (#23824)
- No IPv6 support when suggesting a bindaddr to a PT (#12138)
    - note: related to IPv6 but should probably close
- Which address should a multi-ORPort Tor put in its "transport" extra-info line? (#7962)

- Implement Bridge Guards and other anti-enumeration defenses (#7144)
    - note: prereq for fixing #23824?
    - Alternative design instead of needing to do #7144: For a truly v6-only bridge, which doesn't know how to reach any v4 address: Add a line in the bridge descriptor that says "I can't reach v4" and then the client can see that line and choose its paths better.

The following tickets are related and/or adjacent to this problem:
- Support spawning multiple transport instances (#31228)
    - note: tangentially related, possible related fix
- If I have two bridge lines with the same fingerprint, Tor only tries one of them (#40193)
    - note: will possible be a problem if we fix the above
- Clients cannot use multiple transports with a single bridge (#14579)
    - note: possible duplicate of above
- Looking up bridge by ID may choose the wrong bridge (#14581)


## User side improvements

Make sure the bridges work for users and that they can bootstrap quickly and effectively.

- tor fails to kill its pluggable transports when it's done using them (#21967)
- We fetch bridge descriptors on every setconf and we retry way too aggressively (#40513)
- Slow clients can't bootstrap because they expire their consensus fetch but then receive all the bytes from it anyway, making them expire their next fetch, putting them in a terrible loop (#16844) (every user on a bad network hits this)
- Good bridge + bad bridge = no bootstrap (#40396)
- Don't kill the managed proxy if a transport failed to launch (#7362)
- "Pluggable Transport process terminated" but Tor keeps on going (and of course doesn't work) (#33669)

## Network Health

Do we have enough bridges? Are these bridges working well? Are client circuits good?

- Bridge Authority never assigns a Stable flag to bridges (#40449)
- Option to start as a bridge by default, but change to relay if bw is super-high (#9323)
  - note: could be really cool, but looks like a lot of work
- If your bridge is near your exit, Tor might surprise you by failing your circuit (#2998) (also user improvement) (high priority)
  - notes: collaboration with network health, needs more information, probably low priority


## UX for bridge operators

Making it easy to run bridges and keep them up to date

- Place complete obfs4 bridge line in accessible location (#29128)
- Bridge-Relay HOWTO wiki update for distros with apparmor, escalade log level when transport plugin process is killed (#40459)
- Bridges should report implementation versions of their pluggable transports (#11101)

## Make bridges more able to circumvent censorship
- Obfsbridges should be able to "disable" their ORPort (#7349)

## PT spec (Putting off for arti? Lower priority)
- Optional heartbeat message from PT's (#40439)
- Add support for Pluggable Transports 2.0 (#21816)
- Handle dormant mode in process library and for PT's (#28849)
- Improve semantics for pluggable transports with dummy addresses (#18611)

## Unsorted

- WIP: Reject bridge descriptors posted to non-bridge authorities (#16564)
- We're missing descriptors for some of our primary entry guards (#21969)
- When our directory guards don't have each others' microdescs, we should try an authority or fallback (#23863)
- "ORConn connect failure cache" interferes with "Optimistically retrying bridges" (#40499)

- Make bridges work when the authorities are blocked (#24483)
- UpdateBridgesFromAuthority isn't actually usable (#6010)
- Make bridges publish additional ORPort addresses in their descriptor (#9729)
- Update specifications for bridge download after tor#40497 and friends landed (#40510)

## Low priority

- Tor lets transports advertise private IP addresses in descriptor (#31009)
- Make the bridge authority reject private PT addresses when DirAllowPrivateAddresses is 0 (#31011)
- Allow dynamically and statically linked PTs to be loaded into a Tor binary (#33019)
  - note: maybe we want to leave this open for arti work
- Enable supporting multiple bridge authorities (#26778)
- Better handling and UX for missing and expired guard descriptors (#31707)
- Make bridges wait until they have bootstrapped, before publishing their descriptor (#33582)
- Can authorities use multihop circuits rather than direct connections to detect running routers? (#24020)
- rewrite_node_address_for_bridge and networkstatus_set_current_consensus can conflict (#20531)
- Provide "users-per-transport-per-country" statistics for obfsbridges (#10218)
- Add extra-info line that tracks the number of consensus downloads over each pluggable transport (#8786)
- Bridge authority should sign its bridge networkstatus doc? Or maybe change format to v3-style vote? (#12254)
- obfsproxy makes tor warn when one bridge is down (#8001)
- ServerTransportPlugin process when the underlying binary changes or requests it? (#5081)
- Accounting should work with pluggable transports (#3587)
- Bring sanity to the tor side of the PT shutdown process (#40517)
- ServerTransportListenAddr validation should validate that transport-name is well-formed (#8727)
- Add pluggable transports to Tor's chutney CI job (#30885)
- Meek and ReachableAddresses (#19487)
- Support ORPort picking a random port that persists across restarts (#31103)