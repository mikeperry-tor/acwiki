# Tor Browser's default bridges

Tor Browser ships with a number of default bridges. These bridges are hard-coded in the ​[Tor Browser](https://gitweb.torproject.org/builders/tor-browser-build.git/tree/projects/common/). The following table is meant to be an authoritative list of our default bridges plus the people who operate these bridges. Whenever we add, remove, or change any of our default bridges, make sure to commit this change to the following repositories:

* [​tor-browser-build](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/blob/main/projects/tor-expert-bundle/pt_config.json?ref_type=heads)
* [emma](https://gitlab.torproject.org/tpo/anti-censorship/emma)
* ​[OONI](https://github.com/ooni/sysadmin/blob/master/ansible/roles/probe-services/templates/tor_targets.json) (Default bridges are indexed by a value that's determined as follows: `echo -n ADDR:PORT || BRIDGE_FINGERPRINT | xxd -r -p | sha256sum`)
* [blackbox prometheus exporter](https://gitlab.torproject.org/tpo/tpa/prometheus-alerts/-/blob/main/targets.d/tcp_anticensorship.yaml?ref_type=heads)

The following repositories (used to) use default bridges but don't require updates:

* ​[Onion Browser](https://github.com/OnionBrowser/OnionBrowser/blob/2.X/OnionBrowser/OnionManager.swift) (Onion Browser now fetches and parses bridge_prefs.js automatically.)
* [​Orbot](https://github.com/guardianproject/orbot/blob/master/orbotservice/src/main/res/raw/bridges.txt)/[​Orbot fa](https://github.com/guardianproject/orbot/blob/master/orbotservice/src/main/res/raw-fa/bridges.txt) (The Guardian Project ​[is considering using different bridges](https://github.com/guardianproject/orbot/issues/258))

| Type | IP:port | Fingerprint | Parameters | Contact | Origin | Status |
|------|---------|-------------|------------|---------|--------|--------|
| obfs4 | 37.218.245.14:38224 | D9A82D2F9C2F65A18407B1D2B764F130847F8B5D | cert=bjRaMrr1BRiAW8IE9U5z27fQaYgOhX1UCmOpg2pFpoMvo6ZgQMzLsaTzzQNTlm7hNcb+Sg iat-mode=0 | Mart van Santen mart@greenhost.nl | tpo/applications/tor-browser#22468 | ​[Metrics](https://metrics.torproject.org/rs.html#details/D9E712E593400635462172121B7DB90B07669F71) |
| obfs4 | 212.83.43.95:443 | BFE712113A72899AD685764B211FACD30FF52C31 | cert=ayq0XzCwhpdysn5o0EyDUbmSOx3X/oTEbzDMvczHOdBJKlvIdHHLJGkZARtT4dcBFArPPg iat-mode=1 | Felix zwiebel@quantentunnel.de | tpo/applications/tor-browser-build!1373 | Unpublished​ |
| obfs4 | 212.83.43.74:443 | 39562501228A4D5E27FCA4C0C81A01EE23AE3EE4 | cert=PBwr+S8JTVZo6MPdHnkTwXJPILWADLqfMGoVvhZClMq/Urndyd42BwX9YFJHZnBB3H0XCwUrndyd42BwX9YFJHZnBB3H0XCw iat-mode=1 | Felix zwiebel@quantentunnel.de | tpo/applications/tor-browser-build!1373 | ​Unpublished |
| obfs4 | 209.148.46.65:443 | 74FAD13168806246602538555B5521A0383A1875 | cert=ssH+9rP8dG2NLDN2XuFw63hIO/9MNNinLmxQDpVa+7kTOa9/m+tGWT1SmSYpQ9uTBGa6Hw iat-mode=0 | Micah Sherr msherr@cs.georgetown.edu | legacy/trac#32606 | [​Metrics](https://metrics.torproject.org/rs.html#details/7C95AED7256E1D10D134942532CC72AD73AC1BD8) |
| obfs4 | 146.57.248.225:22 | 10A6CD36A537FCE513A322361547444B393989F0 | cert=K1gDtDAIcUfeLqbstggjIw2rtgIKqdIhUlHp82XRqNSq/mtAjp1BIC9vHKJ2FAEpGssTPw iat-mode=0 | James Holland holla556@umn.edu | legacy/trac#32547 | ​Unpublished |
| obfs4 | 45.145.95.6:27015 | C5B7CD6946FF10C5B3E89691A7D3F2C122D2117C | cert=TD7PbUO0/0k6xYHMPW3vJxICfkMZNdkRrb63Zhl5j9dW3iRGiCx0A7mPhe5T2EDzQ35+Zw iat-mode=0 | Toke Høiland-Jørgensen toke@toke.dk | legacy/trac#32891 | ​[Metrics](https://metrics.torproject.org/rs.html#details/997B6FEB5A02B1F08A36CECCE9B5B3997C0A4680) |
| obfs4 | 51.222.13.177:80 | 5EDAC3B810E12B01F6FD8050D2FD3E277B289A08 | cert=2uplIpLQ0q9+0qMFrK5pkaYRDOe460LL9WHBvatgkuRr/SL31wBOEupaMMJ6koRE6Ld0ew iat-mode=0 | Louis-Philippe Véronneau pollo@debian.org | tpo/applications/tor-browser#40212 | [Metrics](https://metrics.torproject.org/rs.html#details/B35B0E377EBB633C926FB93D81749DB01A030A44) |
| meek | 192.0.2.18:80 | N/A | url=https://1314488750.rsc.cdn77.org front=www.phpmyadmin.net utls=HelloRandomizedALPN | Unredacted | https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/merge_requests/1252 | N/A |
| snowflake-01 | 192.0.2.3:80 | 2B280B23E1107BB62ABFC40DDCC8824814F80A72 | N/A | David Fifield david@bamsoftware.com | N/A | ​[Metrics](https://metrics.torproject.org/rs.html#details/5481936581E23D2D178105D44DB6915AB06BFB7F) |
| snowflake-02 | 192.0.2.4:80 | 8838024498816A039FCBBAB14E6F40A0843051FA | N/A | | N/A | [Metrics](https://metrics.torproject.org/rs.html#details/91DA221A149007D0FD9E5515F5786C3DD07E4BB0) |

## Adding new default bridges

The following is an informal list of requirements that we have for default bridges:

* A reliable network link that can handle at least a sustained 100 Mbit/s of traffic.
  * Any modern CPU can sustain at least 100 MBit/s. A CPU with the AES-NI extension is particularly helpful because it allows Tor to offload its AES operations to the CPU.
  * We recommend 4 GB of RAM for 100 MBit/s.
  * For more than 100 MBit/s, a second core may be necessary and more than 4 GB of RAM.
* We would like to personally know you (or somebody who is involved in the bridge's operation).
  * You should also have a sustainable plan for running the bridge, so it doesn't get abandoned.
* It's useful (but not necessary) for the bridge to have a dedicated IP address because this IP address will get blocked by China and other places.
  * If you don't have a dedicated IP address, don't host any services on the bridge that you don't want to see blocked in some countries or networks.
* You shouldn't also be running an exit relay.
* You should give us at least a one month notice before you shut down your bridge.

When we add a new default bridge, we should ask the operator if they want to receive outage alerts from our ​[monit deployment](https://gitlab.torproject.org/tpo/anti-censorship/monit-configuration).

### torrc configuration

In order to get metrics on our default bridges use, we need to publish the bridge descriptors. However, we do not want this bridge to be distributed over BridgeDB. To achieve this, ensure the following torrc options are set:

```
PublishServerDescriptor 1
BridgeDistribution none
```

## Assigning IP addresses for bridges with placeholder addresses

A torrc Bridge line requires an IP:port address as part of the syntax. The IP:port address is required, even for transports that are not based on making a single connection to a single endpoint; for that reason, Snowflake and meek use "placeholder" addresses.

We currently use the following convention (agreed upon in http://meetbot.debian.net/tor-meeting/2022/tor-meeting.2022-09-08-15.58.log.html#l-66):

192.0.2.(16(<var>n</var>−1)+<var>t</var>):80

where where <var>t</var> indicates the transport (1=flashproxy, 2=meek, 3=snowflake) and <var>n</var> is a counter if there are multiple bridge lines for single transport.

This guarantees that different instances of different transports have different addresses (necessary to work around tor's internal assumption that IP:port is a sufficient identifier for a bridge), and that the port number is always one that is permitted by FascistFirewall (necessary to work around tpo/core/tor#19487).
