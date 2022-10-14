Note: this is a work-in-progress and is incomplete.

This is a guide for anti-censorship team members on how to add support for a new pluggable transport (PT) to Tor Browser.

## Build the PT client reproducibly

The first step is to create a project in tor-browser-build to build the pluggable transport client reproducibly on all platforms.

## Add the client binary to the tor-expert-bundle

## Set the ClientTransportPlugin line in the default torrc files 

## Configure the onion-proxy and android-service to accept the PT

## Restrict to alpha versions of Tor Browser

It's a good idea to test new PTs in alpha versions of tor browser first. This can be configured in the reproducible build system. 

## Add an option for a built-in bridge (optional)

It's recommended when integrating a new PT to first allow test users to configure it as a manual bridge by providing a custom bridge line. Once the PT has been thoroughly tested this way, bridge lines can be added as a built-in bridge option.

