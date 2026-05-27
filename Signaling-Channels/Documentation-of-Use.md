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

Requests are made to the following endpoints, which return JSON responses:
- `/fetch` and `/check` when users select `Request bridges...` in the Connection settings of `about:preferences`.
- `circumvention/settings` are made when an [`AutoBoostrapAttempt`](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/66d59b50b58c5c81f588bda50077b898726b733e/toolkit/modules/TorConnect.sys.mjs#L416) is made, on a bootstrapping error. See this [flow chart](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/f48a5f15ccba384a05d000e5ec3fa91f26c1161f/toolkit/modules/TorConnect.sys.mjs#L83) for when this feature is used.
- `circumvention/defaults` are fetched if the user failed to connect but there are [no settings recommended for their region](https://gitlab.torproject.org/tpo/applications/tor-browser/-/blob/66d59b50b58c5c81f588bda50077b898726b733e/toolkit/modules/TorConnect.sys.mjs#L534).
- `circumvention/builtin` is implemented but not currently used.

#### Orbot implementation

Orbot [implements the Moat API](https://github.com/guardianproject/orbot-android/blob/84015e0a48d81783c0cd24e9d98d2791739d38e5/app/src/main/java/org/torproject/android/service/circumvention/MoatApi.kt) and uses it in the [Ask Tor](https://github.com/guardianproject/orbot-android/blob/84015e0a48d81783c0cd24e9d98d2791739d38e5/app/src/main/java/org/torproject/android/ui/connect/ConfigConnectionBottomSheet.kt#L323) feature and [Smart Connect](https://github.com/guardianproject/orbot-android/blob/84015e0a48d81783c0cd24e9d98d2791739d38e5/docs/design/design-spec-smart-connect.md).

The orbot service opens a [Moat tunnel](https://github.com/guardianproject/orbot-android/blob/84015e0a48d81783c0cd24e9d98d2791739d38e5/app/src/main/java/org/torproject/android/service/circumvention/MoatTunnel.kt) using [IPtProxy](https://github.com/tladesignz/IPtProxy) and opens a SOCKS connection to the newly opened SOCKS listener to send API requests through.

Orbot supports both domain fronting through meek and dnstt as signaling channels. The configurations and settings for both of these channels are [hard-coded](https://github.com/guardianproject/orbot-android/blob/84015e0a48d81783c0cd24e9d98d2791739d38e5/app/src/main/java/org/torproject/android/service/circumvention/MoatTunnel.kt) as `TOR_PROJECT` and `GUARDIAN_PROJECT` settings, respectively.

#### Server side rdsys implementation

On the server side, Moat connections are received by a [tor-less meek server](https://gitlab.torproject.org/tpo/anti-censorship/team/-/wikis/Moat), with the client IP captured and passed to rdsys through an `ExtOrPort` connection to a [shim](https://gitlab.torproject.org/tpo/anti-censorship/moat-shim). These HTTP requests and responses are then handled by the web server.

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
and the remote address of the client, used for metrics purposes. For rendezvous methods that do not naturally preserve the client IP address, [it is extracted from the WebRTC Offer SDP](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/blob/cee56c134d85715ad9a443f7894b1728c5f37417/common/util/util.go#L117). This can be easily spoofed and should not be trusted for enumeration prevention purposes.

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

Conjure requires the client IP address for registration purposes. Without it, the station is unable to map an incoming client connection to a phantom proxy registration, and the connection to the phantom proxy will fail. This provides some built-in active probing resistance, but also presents challenges for registration channels that do not naturally preserve the client IP. To solve this, Conjure has clients [use STUN to discover their public IP address](https://github.com/refraction-networking/conjure/blob/3d8b86cfcc24e0245ccf60dda4f23d3cf5303dca/pkg/registrars/registration/dns-registrar.go#L219) and send the discovered IP in the registration message. This can be easily spoofed, but not in a way that allows clients to successfully connect to phantom proxies.

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

### OONI

OONI uses domain fronting to send measurements from probes to the backend.

##### Go implementation

At the probe, this is implemented simply by [manually setting the URL hostname and HTTP HOST headers](https://github.com/ooni/probe-cli/blob/c52ce3b50893e650c8e60490343e7a7892c00d64/internal/probeservices/probeservices.go#L110). This requires no server side changes, and measurement submissions are conducted via API requests over HTTP.

### Unidirectional updates (proposed)

- https://people.torproject.org/~cohosh/push-notifications.html

# Signaling channel implementations

Rather than fully document how each signalling channel works, this documentation will cover configuration details, important features, and constraints on the signalling channels we already have in use.

### Domain fronting

##### Configuration

- Front: URL visible to censor, to go in the TLS SNI and DNS requests (e.g., `cdn.zk.mk`)
- Host: reflector URL that points to the signalling server, created by making an account with the cloud provider (e.g., `https://1098762253.rsc.cdn77.org`)
- (optional) UTLS settings

##### Features

- **price:** varies by provider, pricey as a full channel but reasonable as a signalling channel

- **preservation of client IP:** sort of, the IP address of the client will be appended to the X-Forwarded-For header by whatever 3rd party is doing the fronting. But, since the server is just HTTP, clients may also make a direct request to the server and add a spoofed address to this header. Locking this down would require some kind of allow list of trusted cloud provider IPs from which to trust the X-Forwarded-For header, but this is a potentially difficult list to keep up to date. See recent discusison in
  - https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/webtunnel/-/work_items/60+
  - https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/meek/-/work_items/40006+
  - https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/work_items/40451+

- **fingerprinting resistance via UTLS**

### Amazon SQS

##### Configuration

- server queue name: Queue name that clients have write-only access to send data to the server (e.g., `https://sqs.us-east-2.amazonaws.com/490393006362/snowflake-broker`)
- client queue name prefix: Prefix used to randomly generate single-use client queues for server responses (e.g., `https://sqs.us-east-2.amazonaws.com/490393006362/snowflake-client-*`)
- sqs credentials for client: AWS key and secret for a client IAM user with write-only access to the server queue and read access for client queue prefixes. Must be encoded to prevent triggering AWS's lockdown of the account. Base64 has been sufficient in the past.

##### Features
- **price:** similar to domain fronting, see this [cost analysis of SQS](https://lists.torproject.org/mailman3/hyperkitty/list/anti-censorship-team@lists.torproject.org/message/T5REPCMJJFK3TGVYNSDCU3WT7SQDARPB/).

##### Constraints

Does not preserve the client IP address or a way to individualize clients. The AWS access key is shared by all clients.

There is a size limit to SQS requests. From the [SQS documentation](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/quotas-messages.html)
> The minimum message size is 1 byte (1 character). The maximum is 1,048,576 bytes (1 MiB).
> 
> To send messages larger than 1 MiB, you can use the Amazon SQS Extended Client Library for Java and the Amazon SQS Extended Client Library for Python. This library allows you to send an Amazon SQS message that contains a reference to a message payload in Amazon S3. The maximum payload size is 2 GB.

Another option is to send signalling data over multiple messages using one of the reliability layers discussed below.

### AMP Cache

##### Configuration

- (optional front) Front: URL visible to censor, similar to domain fronting, can hide that you are using AMP cache (e.g., `www.google.com`)
- AMP cache URL: URL to AMP server (e.g., `https://cdn.ampproject.org/`
- Host: URL of signalling server (e.g., `https://snowflake-broker.torproject.net/`)
- (optional) UTLS settings

##### Features

- **price:** free

##### Constraints

There is pretty severe rate limiting for AMP cache requests, seemingly based on client IP address.

Does not preserve the client IP address or provide a way to individualize clients.

# Common features of signaling channels

These are some ideal common features for signaling channels. Not all combinations of uses and channels will require all features, and some channels have these features built-in. But many will require a separate layer to support these properties.

### Reliability

A reliability layer may be needed for signalling channels that do not provide built-in reliability assumptions. If, for example, requests and responses need to be split across multiple transfers or if the medium is not itself reliable (e.g., UDP).

##### TurboTunnel

TurboTunnel is a design pattern for censorship circumvention tools that proposes the use of an end-to-end reliability layer between client and server. It has yet to be used together with signalling channels, but was a necessary feature for established Snowflake connections. It has also been used with [dnstt](https://www.bamsoftware.com/software/dnstt/), a circumvention transport over DNS, which could be adapted as a signalling channel.

##### Fountain Codes

A lighter-weight alternative to a full on sequencing and reliability layer that uses rateless erasure codes to chunk and retransmit signalling channel data until enough information has been received by the other side to reconstruct the original message.

Link to paper and implementation: https://github.com/net4people/bbs/issues/591#issuecomment-4248280173

### Padding

Padding has become a more critical feature of circumvention tools recently. Reports of successful uses of padding to circumvent blocks:
- [Potential TLS-over-DTLS blocking in China (2023)](https://github.com/net4people/bbs/issues/255)
- [Throttling of Twitter in Russia (2021)](https://github.com/net4people/bbs/issues/65#issuecomment-816243379)

It is likely to be especially relevant to signalling channels, which can have very distinctive patterns.

Both TurboTunnel and the Fountain Codes papers discussed above have discussions on padding implementations built in to the reliability mechanism.

See:
- [Snowflake's encapsulation.go](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/blob/cee56c134d85715ad9a443f7894b1728c5f37417/common/encapsulation/encapsulation.go#L117)

### End-to-end confidentiality and integrity

Many signalling channels rely on 3rd party services and do not offer full end-to-end confidentiality and integrity between the client and the signalling server. For example, in domain fronting, the client encrypts an HTTP request for the cloud provider or edge service, and the request is then re-encrypted by that provider for the signalling server.

This has been discussed in:
- https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/work_items/22945+
- https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/merge_requests/39#note_2737344+.

An easy way to do this could be to have clients asymmetrically encrypt the initial message to the signalling server with the server's public key, and include a symmetric key in the signalling data that should be used to encrypt the server's response. The response can also be signed with the server's private key and verified by the client using the same public key as before. This is partially implemented in the [Orbot push notifications proof of concept code](https://github.com/cohosh/orbot/commit/9841fcbec517c238fc3ebf7130b4d9c7094b8e32). The server's public key would have to be distributed along with other channel details.

### Preservation of client IP

Applications of signalling channels use client IP addresses for metrics, circumvention settings, and anti-enumeration features. Many signaling channels do not have an easy or trusted way of preserving client IP addresses.

Not all of these use-cases requires trust. Geo-location for circumvention settings are in a client's best interest to provide an honest and accurate IP address. Some options are to take the same route as Conjure and use an additional STUN request to get the client's public IP address. We could also prompt the user to manually provide their country code, as Tor Browser does at a late stage in the autoconnect flow, or fetch locale information from the user's device.

For metrics, the impact of attacker-spoofed IP addresses is probably fairly low compared the majority of honest clients.

For anti-enumeration features, IP addresses as unique identifiers should probably be replaced with some other means of preventing enumeration.

### Compression

Compression is an optional step that can be used to reduce the size of messages sent via signalling channels to fit within constraints of that channel. See the [analysis of compressing Snowflake rendezvous messages](https://lists.torproject.org/mailman3/hyperkitty/list/anti-censorship-team@lists.torproject.org/thread/ZK3KJ6F3BCJRVNS55BMB6MXQNTTEFRTB/).

# Timeline of censorship events affecting signaling channels

Censorship events are a useful learning experience and tell us what changes, configurations, or updates to protocols we should be able to accommodate. They show where the pain points are in existing implementations and the UX needs of applications. Here are some recent censorship events that have affected Tor's signaling channels and how we responded.

# Other signalling channel library implementations

- [Outline SDK](https://github.com/OutlineFoundation/outline-sdk/tree/main)
- [Raceboat](https://github.com/tst-race/raceboat/)

### Kindling

[Kindling](https://github.com/getlantern/kindling) is a Lantern library for making HTTP requests through one of several supported tunnels. Applications configure which tunnels they are willing to use and the library attempts connections through all at once, using whichever tunnel responds fastest.

Kindling returns an [`http.Client`](https://pkg.go.dev/net/http#Client) that can be used to make HTTP requests through the configured tunnels to an arbitrary address. One downside to this is that even though `NewRoundTripper` can be used to attempt a connection to an arbitrary address, most tunnels will have a fairly restrictive set of addresses they can connect to. For example, domain fronting tunnels through CDN77 will only support connections to other URLs hosted on the same cloud provider.

New transports must implement the [`Transport`](https://github.com/getlantern/kindling/blob/a9712f95df034fcd4b8fd2eca9e7cc8ab61339a6/kindling.go#L48) interface
```golang
// Transport defines a censorship circumvention transport that can be used by Kindling.
type Transport interface {
	// NewRoundTripper creates a pre-connected http.RoundTripper. Implementations
	// should complete the connection before returning so that the race transport
	// can try requests serially without paying connection latency.
	NewRoundTripper(ctx context.Context, addr string) (http.RoundTripper, error)

	// MaxLength returns the maximum request body size this transport supports.
	// Zero means no limit.
	MaxLength() int

	// IsStreamable reports whether this transport supports streaming responses
	// (e.g. text/event-stream).
	IsStreamable() bool

	// Name identifies this transport for logging and debugging.
	Name() string
}
```
This library also has a reliance on HTTP. While this is useful for applications like Moat or OONI that currently use and require HTTP for API calls, it requires applications to use HTTP as the carrier protocol. This also limits applications to bidirectional channels only.