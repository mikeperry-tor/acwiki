Documentation on our use of signaling channels

## Moat / Circumvention Settings API

We expose a [Moat API](https://gitlab.torproject.org/tpo/anti-censorship/rdsys/-/blob/d14af39503763690e3ee4ad4eb2fe926afc5377a/doc/moat.md) for applications to fetch bridges and circumvention settings from rdsys. This API is currently bidirectional, requiring applications to send a request for bridges or settings.

This [API is already well documented](https://gitlab.torproject.org/tpo/anti-censorship/rdsys/-/blob/d14af39503763690e3ee4ad4eb2fe926afc5377a/doc/moat.md#circumventionsettings). Request and response sizes vary by endpoint, but are fairly small. A typical flow involves 1-3 round trips when users first start the application or first bootstrap Tor: the first to request recommending settings for the user's country. If the user is supplied a bridge line, the flow ends there. If the user is directed to fetch a bridge, they will need to complete a captcha challenge in 2 round trips.

The largest requests for any of these endpoints are typically less than 500 bytes. The largest potential response is likely from fetching the entire [circumvention settings map](https://gitlab.torproject.org/tpo/anti-censorship/rdsys-admin/-/blob/0061ce86ee2dfc8c451a78d28d0ef09e4ed7f36e/conf/circumvention.json) which is currently 9KB but could easily grow if more countries require bespoke censorship settings. Responses with just bridge lines will usually fit in under 1KB.

#### Tor Browser implementation

The Moat API is [implemented in Tor Browser](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/66d59b50b58c5c81f588bda50077b898726b733e/toolkit/modules/Moat.sys.mjs) as a browser [javascript module](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules).

This module currently only supports domain fronting. On initialization, it [launches a PT that supports meek](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/66d59b50b58c5c81f588bda50077b898726b733e/toolkit/modules/DomainFrontedRequests.sys.mjs#L90) as a separate process and [fetches domain fronting settings](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/66d59b50b58c5c81f588bda50077b898726b733e/toolkit/modules/Moat.sys.mjs#L71-82) (the front domain and reflector) from stored preferences. When an API call is made, the module [opens a SOCKS connection to the PT process](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/66d59b50b58c5c81f588bda50077b898726b733e/toolkit/modules/DomainFrontedRequests.sys.mjs#L480) and [sends the request](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/66d59b50b58c5c81f588bda50077b898726b733e/toolkit/modules/Moat.sys.mjs#L108) through this local SOCKS proxy.



#### Orbot implementation

- when this API is called

#### Timeline of censorship events

## Snowflake rendezvous

## Unidirectional updates (proposed)

- https://people.torproject.org/~cohosh/push-notifications.html