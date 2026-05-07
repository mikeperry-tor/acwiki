Documentation on our use of signaling channels

[[_TOC_]]

# Applications of signaling channels

### Moat / Circumvention Settings API

We expose a [Moat API](https://gitlab.torproject.org/tpo/anti-censorship/rdsys/-/blob/d14af39503763690e3ee4ad4eb2fe926afc5377a/doc/moat.md) for applications to fetch bridges and circumvention settings from rdsys. This API is currently bidirectional, requiring applications to send a request for bridges or settings.

This [API is already well documented](https://gitlab.torproject.org/tpo/anti-censorship/rdsys/-/blob/d14af39503763690e3ee4ad4eb2fe926afc5377a/doc/moat.md#circumventionsettings). Request and response sizes vary by endpoint, but are fairly small. A typical flow could involve 1-3 round trips when users first start the application, select an auto config option, or fail to bootstrap Tor: the first to request recommending settings for the user's country. If the user is supplied one or more working bridge lines, the flow ends there. If the user falls back on default settings, those settings can be fetched in 1 additional round trip. If the user decides to manually fetch a bridge, they will need to complete a captcha challenge in 2 round trips.

The largest requests for any of these endpoints are typically less than 500 bytes. The largest potential response is likely from fetching the entire [circumvention settings map](https://gitlab.torproject.org/tpo/anti-censorship/rdsys-admin/-/blob/0061ce86ee2dfc8c451a78d28d0ef09e4ed7f36e/conf/circumvention.json) which is currently 9KB but could easily grow if more countries require bespoke censorship settings. Responses with just bridge lines will usually fit in under 1KB.

#### Tor Browser implementation

The Moat API is [implemented in Tor Browser](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/66d59b50b58c5c81f588bda50077b898726b733e/toolkit/modules/Moat.sys.mjs) as a browser [javascript module](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules).

This module currently only supports domain fronting. On initialization, it [launches a PT that supports meek](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/66d59b50b58c5c81f588bda50077b898726b733e/toolkit/modules/DomainFrontedRequests.sys.mjs#L90) as a separate process and [fetches domain fronting settings](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/66d59b50b58c5c81f588bda50077b898726b733e/toolkit/modules/Moat.sys.mjs#L71-82) (the front domain and reflector) from stored preferences. When an API call is made, the module [opens a SOCKS connection to the PT process](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/66d59b50b58c5c81f588bda50077b898726b733e/toolkit/modules/DomainFrontedRequests.sys.mjs#L480) and [sends the request](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/66d59b50b58c5c81f588bda50077b898726b733e/toolkit/modules/Moat.sys.mjs#L108) through this local SOCKS proxy.

Requests are made to the following endpoints:
- `/fetch` and `/check` when users select `Request bridges...` in the Connection settings of `about:preferences`.
- `circumvention/settings` are made when an [`AutoBoostrapAttempt`](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/66d59b50b58c5c81f588bda50077b898726b733e/toolkit/modules/TorConnect.sys.mjs#L416) is made, on a bootstrapping error. See this [flow chart](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/f48a5f15ccba384a05d000e5ec3fa91f26c1161f/toolkit/modules/TorConnect.sys.mjs#L83) for when this feature is used.
- `circumvention/defaults` are fetched if the user failed to connect but there are [no settings recommended for their region](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/66d59b50b58c5c81f588bda50077b898726b733e/toolkit/modules/TorConnect.sys.mjs#L534).
- `circumvention/builtin` is implemented but not currently used.

#### Orbot implementation

Orbot [implements the Moat API](https://github.com/guardianproject/orbot-android/blob/84015e0a48d81783c0cd24e9d98d2791739d38e5/app/src/main/java/org/torproject/android/service/circumvention/MoatApi.kt) and uses it in the [Ask Tor](https://github.com/guardianproject/orbot-android/blob/84015e0a48d81783c0cd24e9d98d2791739d38e5/app/src/main/java/org/torproject/android/ui/connect/ConfigConnectionBottomSheet.kt#L323) feature and [Smart Connect](https://github.com/guardianproject/orbot-android/blob/84015e0a48d81783c0cd24e9d98d2791739d38e5/docs/design/design-spec-smart-connect.md).

The orbot service opens a [Moat tunnel](https://github.com/guardianproject/orbot-android/blob/84015e0a48d81783c0cd24e9d98d2791739d38e5/app/src/main/java/org/torproject/android/service/circumvention/MoatTunnel.kt) using [IPtProxy](https://github.com/tladesignz/IPtProxy) and opens a SOCKS connection to the newly opened SOCKS listener to send API requests through.

Orbot supports both domain fronting through meek and dnstt as signaling channels. The configurations and settings for both of these channels are [hard-coded](https://github.com/guardianproject/orbot-android/blob/84015e0a48d81783c0cd24e9d98d2791739d38e5/app/src/main/java/org/torproject/android/service/circumvention/MoatTunnel.kt) as `TOR_PROJECT` and `GUARDIAN_PROJECT` settings, respectively.

### Snowflake rendezvous

Snowflake clients use signaling channels to get matched with an available proxy and perform WebRTC signaling in what is called a [rendezvous step](https://www.bamsoftware.com/papers/snowflake/#rendezvous). This requires a single round-trip communication with the Snowflake broker. The client rendezvous protocol is documented in the [messages package](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/blob/cee56c134d85715ad9a443f7894b1728c5f37417/common/messages/client.go). The majority of the message consists of a [SDP offer](https://datatracker.ietf.org/doc/html/rfc3264) and sits between 1KB-2KB in size.

Client rendezvous happens at startup and whenever a Snowflake connection does not have a functioning proxy. Clients will re-attempt the rendezvous [every 10 seconds](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/blob/cee56c134d85715ad9a443f7894b1728c5f37417/client/lib/snowflake.go#L53) until they receive a working proxy.

#### Go implementation

Snowflake currently supports 3 signaling channels:
- domain fronting
- [AMP cache](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/blob/cee56c134d85715ad9a443f7894b1728c5f37417/doc/broker-spec.txt#L217)
- [Amazon SQS](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/blob/cee56c134d85715ad9a443f7894b1728c5f37417/doc/rendezvous-with-sqs.md)

Each of these has a broker component that reads incoming requests and calls [`IPC.ClientOffers`](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/blob/cee56c134d85715ad9a443f7894b1728c5f37417/broker/ipc.go#L181)
```golang
func (i *IPC) ClientOffers(arg messages.Arg, response *[]byte) error
```
when the function returns, the component sends the encoded `response` back to the client.

On the client side, each rendezvous method implements the `RendezvousMethod` interface
```golang
// RendezvousMethod represents a way of communicating with the broker: sending
// an encoded client poll request (SDP offer) and receiving an encoded client
// poll response (SDP answer) in return. RendezvousMethod is used by
// BrokerChannel, which is in charge of encoding and decoding, and all other
// tasks that are independent of the rendezvous method.
type RendezvousMethod interface {
    Exchange([]byte) ([]byte, error)
}   
```

### Conjure Registration

[Conjure](https://jhalderm.com/pub/papers/conjure-ccs19.pdf) uses bidirectional signaling channels for the client registration step, during which clients are assigned a phantom proxy IP address. Conjure registrations happen at startup for each Conjure connection.

Conjure uses [protobufs to encode registration messages](https://github.com/refraction-networking/conjure/tree/3d8b86cfcc24e0245ccf60dda4f23d3cf5303dca/proto). These requests may be [optionally padded](https://github.com/refraction-networking/conjure/blob/3d8b86cfcc24e0245ccf60dda4f23d3cf5303dca/proto/signalling.proto#L299) as a fingerprinting defense.

##### Go implementation

The server side of Conjure signaling channels are implemented in the [registration-server](https://github.com/refraction-networking/conjure/tree/3d8b86cfcc24e0245ccf60dda4f23d3cf5303dca/cmd/registration-server) application. Each signaling channel implements the `registrar` interface
```golang
type registrar interface {
	RegisterUnidirectional(*pb.C2SWrapper, pb.RegistrationSource, []byte) error
	RegisterBidirectional(*pb.C2SWrapper, pb.RegistrationSource, []byte) (*pb.RegistrationResponse, error)
}
```
on the client side, signaling channels implement the [`Registrar`](https://github.com/refraction-networking/gotapdance/blob/a8e3647052911e4ef9c69146f147c32155747669/tapdance/interfaces.go#L16-L21) interface
```golang
// Registrar defines the interface for a module completing the initial portion of the conjure
// protocol which registers the clients intent to connect, along with the specifics of the session
// they wish to establish.
type Registrar interface {
	Register(*ConjureSession, context.Context) (*ConjureReg, error)

	// PrepareRegKeys prepares key materials specific to the registrar
	PrepareRegKeys(stationPubkey [32]byte, sessionSecret []byte) error
}
```

### Unidirectional updates (proposed)

- https://people.torproject.org/~cohosh/push-notifications.html

# Signaling channel implementations

# Common features of signaling channels

These are some ideal common features for signaling channels. Not all channels will require all features.

### Reliability

- TurboTunnel
- Fountain Codes

### Padding

### End-to-end confidentiality

# Timeline of censorship events affecting signaling channels