Instructions for setting up a new Conjure bridge

## Introduction

The Conjure bridge sits between deployed Conjure stations and the Tor network. It acts as an entrypoint to access the Tor network through the Conjure.

While the PT server itself is a simple haproxy server, we maintain wireguard connections between deployed stations. Conjure allows clients to connect to arbitrary destinations, but in order to preserve the client IP address for [safely collected usage metrics](https://metrics.torproject.org/about.html), we [configure the Conjure PT client to send a `PROXY` header](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/conjure/-/blob/77bf2fef6e1585a90112f7519a935fabd79a559a/client/conjure/registration.go#L81) with the client's IP address in the TCP connection to the Conjure bridge. These country-based usage metrics can help us detect and respond to censorship events and are an important part of quickly adapting to changes in censorship. The Wireguard connections prevent client IPs from being forwarded in plaintext to the Conjure bridge.

There are currently multiple conjure stations. New stations are occasionally deployed and existing stations may be moved. The Wireguard setup requires good communication with Conjure station admins, an important part of Conjure bridge maintenance is a quick response time for updating the Wireguard configuration.

## Bridge Setup

## Wireguard setup