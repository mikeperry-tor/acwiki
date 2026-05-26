Documentation on our use of signaling channels

[[_TOC_]]

# Applications of signaling channels

### Moat / Circumvention Settings API

We expose a [Moat API](https://gitlab.torproject.org/tpo/anti-censorship/rdsys/-/blob/d14af39503763690e3ee4ad4eb2fe926afc5377a/doc/moat.md) for applications to fetch bridges and circumvention settings from rdsys. This API is currently bidirectional, requiring applications to send a request for bridges or settings.

This [API is already well documented](https://gitlab.torproject.org/tpo/anti-censorship/rdsys/-/blob/d14af39503763690e3ee4ad4eb2fe926afc5377a/doc/moat.md#circumventionsettings). Request and response sizes vary by endpoint, but are fairly small. A typical flow could involve 1-3 round trips when users first start the application, select an auto config option, or fail to bootstrap Tor: the first to request recommending settings for the user's country. If the user is supplied one or more working bridge lines, the flow ends there. If the user falls back on default settings, those settings can be fetched in 1 additional round trip. If the user decides to manually fetch a bridge, they will need to complete a captcha challenge in 2 round trips.

The largest requests for any of these endpoints are typically less than 500 bytes. The largest potential response is likely from fetching the entire [circumvention settings map](https://gitlab.torproject.org/tpo/anti-censorship/rdsys-admin/-/blob/0061ce86ee2dfc8c451a78d28d0ef09e4ed7f36e/conf/circumvention.json) which is currently 9KB but could easily grow if more countries require bespoke censorship settings. Responses with just bridge lines will usually fit in under 1KB.

The IP address of the client is used for both geo-location purposes, to choose the appropriate settings, and for light enumeration resistance. It is extracted from the X-Forwarded-For header of domain fronted and HTTP requests, but this is subject to trust issues, discussed later in https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Signaling-Channels/Documentation-of-Use#domain-fronting. Users also have the ability to manually specify their country code if geolocation fails.

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
where `messages.Arg` is a struct
```golang
type Arg struct {
    Body             []byte
    RemoteAddr       string
    RendezvousMethod RendezvousMethod
    Context          context.Context
}
```
the important fields here are a byte slice of the JSON encoded client poll request
```golang
type ClientPollRequest struct {
    Offer       string `json:"offer"`
    NAT         string `json:"nat"`
    Fingerprint string `json:"fingerprint"`
}
```
and the remote address of the client, used for metrics purposes.
When the call to `IPC.ClientOffers` returns, the component sends the JSON encoded `response` back to the client.
```golang
type ClientPollResponse struct {
    Answer string `json:"answer,omitempty"`
    Error  string `json:"error,omitempty"`
}
```

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
which takes a byte slice of the JSON encoded `ClientPollRequest` and returns a byte slice of the JSON encoded `ClientPollResponse` or `error`.

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

Rather than fully document how each signalling channel works, this documentation will cover important features or constraints on the signalling channels we already have in use.

### Domain fronting

- **price:** varies by provider, pricey as a full channel but reasonable as a signalling channel

- **preservation of client IP:** sort of, the IP address of the client will be appended to the X-Forwarded-For header by whatever 3rd party is doing the fronting. But, since the server is just HTTP, clients may also make a direct request to the server and add a spoofed address to this header. Locking this down would require some kind of allow list of trusted cloud provider IPs from which to trust the X-Forwarded-For header, but this is a potentially difficult list to keep up to date. See recent discusison in
  - https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/webtunnel/-/work_items/60+
  - https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/meek/-/work_items/40006+
  - https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/work_items/40451+

### Amazon SQS

- **price:** similar to domain fronting, see this [cost analysis of SQS](https://lists.torproject.org/mailman3/hyperkitty/list/anti-censorship-team@lists.torproject.org/message/T5REPCMJJFK3TGVYNSDCU3WT7SQDARPB/).

##### Constraints

Does not preserve the client IP address or a way to individualize clients. The AWS access key is shared by all clients.

### AMP Cache

- **price:** free

##### Constraints

There is pretty severe rate limiting for AMP cache requests, seemingly based on client IP address. 

# Common features of signaling channels

These are some ideal common features for signaling channels. Not all combinations of uses and channels will require all features, and some channels have these features built-in. But many will require a separate layer to support these properties.

### Reliability

- TurboTunnel
- Fountain Codes

### Padding

### End-to-end confidentiality

Many signalling channels rely on 3rd party services and do not offer full end-to-end confidentiality between the client and the signalling server. For example, in domain fronting, the client encrypts an HTTP request for the cloud provider or edge service, and the request is then re-encrypted by that provider for the signalling server.

This has been discussed in:
- https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/work_items/22945+
- https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/merge_requests/39#note_2737344+.

### Preservation of client IP

Tor uses client IP addresses for metrics, circumvention settings, and anti-DoS features. Many signaling channels do not have an easy or trusted way of preserving client IP addresses.

### Compression

Compression is an optional step that can be used to reduce the size of messages sent via signalling channels to fit within constraints of that channel. See the [analysis of compressing Snowflake rendezvous messages](https://lists.torproject.org/mailman3/hyperkitty/list/anti-censorship-team@lists.torproject.org/thread/ZK3KJ6F3BCJRVNS55BMB6MXQNTTEFRTB/).

# Timeline of censorship events affecting signaling channels

Censorship events are a useful learning experience and tell us what changes, configurations, or updates to protocols we should be able to accommodate. They show where the pain points are in existing implementations and the UX needs of applications. Here are some recent censorship events that have affected Tor's signaling channels and how we responded.