Note: this is a work-in-progress and is incomplete.

This is a guide for anti-censorship team members on how to add support for a new pluggable transport (PT) to Tor Browser.

## Build the PT client reproducibly

The first step is to create a project in tor-browser-build to build the pluggable transport client reproducibly on all platforms.

## Add the client binary to the tor-expert-bundle

The `tor-expert-bundle` project groups togeter and packages up PT binaries and built-in bridge lines so that they can be extracted to the right place for each platform's browser build.

Adding a new PT to this bundle is not too difficult. The first step is to add the PT project as a dependency in the `tor-expert-bundle` [config file](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/blob/main/projects/tor-expert-bundle/config):

```diff
diff --git a/projects/tor-expert-bundle/config b/projects/tor-expert-bundle/config
--- a/projects/tor-expert-bundle/config
+++ b/projects/tor-expert-bundle/config
@@ -13,6 +13,8 @@ input_files:
     project: obfs4
   - name: snowflake
     project: snowflake
+  - name: newpt
+    project: newpt
```

Then, modify the `tor-expert-bundle` [build file](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/blob/main/projects/tor-expert-bundle/build) to include your PT in the bundle:

```diff
diff --git a/projects/tor-expert-bundle/build b/projects/tor-expert-bundle/build
--- a/projects/tor-expert-bundle/build
+++ b/projects/tor-expert-bundle/build
@@ -11,6 +11,7 @@ mkdir pluggable_transports && cd pluggable_transports
 
 tar -xkf $rootdir/[% c('input_files_by_name/obfs4') %]
 tar -xkf $rootdir/[% c('input_files_by_name/snowflake') %]
+tar -xkf $rootdir/[% c('input_files_by_name/newpt') %]
```

## Set the ClientTransportPlugin line in the default torrc files 

## Configure the onion-proxy and android-service to accept the PT

## Restrict to alpha versions of Tor Browser

It's a good idea to test new PTs in alpha versions of tor browser first. This can be configured in the reproducible build system. 

## Add an option for a built-in bridge (optional)

It's recommended when integrating a new PT to first allow test users to configure it as a manual bridge by providing a custom bridge line. Once the PT has been thoroughly tested this way, bridge lines can be added as a built-in bridge option.

