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

## Set the ClientTransportPlugin line for desktop platforms

Each platform has its own torrc defaults file in the `browser` project. There is one for:
- [linux](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/blob/main/projects/browser/Bundle-Data/PTConfigs/linux/torrc-defaults-appendix)
- [windows](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/blob/main/projects/browser/Bundle-Data/PTConfigs/windows/torrc-defaults-appendix)
- [mac](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/blob/main/projects/browser/Bundle-Data/PTConfigs/mac/torrc-defaults-appendix)

Pay close attention to how the executables are called in each of the different platforms and emulate the existing ClientTransportPlugin lines but with your own PT name and arguments. For example:
```diff
diff --git a/projects/browser/Bundle-Data/PTConfigs/linux/torrc-defaults-appendix b/projects/browser/Bun
dle-Data/PTConfigs/linux/torrc-defaults-appendix
--- a/projects/browser/Bundle-Data/PTConfigs/linux/torrc-defaults-appendix
+++ b/projects/browser/Bundle-Data/PTConfigs/linux/torrc-defaults-appendix
@@ -2,4 +2,4 @@
 ClientTransportPlugin meek_lite,obfs2,obfs3,obfs4,scramblesuit exec ./TorBrowser/Tor/PluggableTransport
s/obfs4proxy

+## newpt configuration
+ClientTransportPlugin newpt exec ./TorBrowser/Tor/PluggableTransports/newpt-client -arg1 foo -arg2 bar
```

Once these are updated, the new PT will work on all desktop platforms of Tor Browser. The next step covers how to add support in Android platforms.

## Adding PT support in Android

Integrating a new PT into android builds of Tor Browser requires modifications to a few different repositories.

#### Create a patch for `tor-onion-proxy-library`

This project is not maintained by the applications team at TPO, so changes to this repository are handled by the creation of patches that are applied at build time. Use the following procedure to create a patch:

1. Clone the upstream repository
   ```
   git clone https://github.com/thaliproject/Tor_Onion_Proxy_Library.git
   ```
2. Apply the existing PT patches in order

   Copy or download all of the existing `*.patch` files from the [`tor-onion-proxy-library`](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/tree/main/projects/tor-onion-proxy-library) project in `tor-browser-build` and apply them in the order specified in the corresponding [build script](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/blob/bf61fa94cb3584537cfa01d4aca361f33f0faf0b/projects/tor-onion-proxy-library/build).

   For example, at the time of this writing:
   ```
   git am -3 gradle.patch
   git am -3 0001-Bug-33931-Filter-bridges-in-stream-by-type.patch
   git am -3 0001-Bug-30318-Add-snowflake-support.patch
   ```
3. Modify `build.gradle` to find the executable

   ```diff
   diff --git a/android/build.gradle b/android/build.gradle
   --- a/android/build.gradle
   +++ b/android/build.gradle
   @@ -96,6 +96,9 @@ task copyPluggableTransports(type: Copy) {
        rename { filename ->
            filename.replace 'snowflake-client', 'libSnowflake.so'
        }
   +    rename { filename ->
   +        filename.replace 'newpt-client', 'libNewpt.so'
   +    }
    }
   ```
4. 

#### Create a patch for `tor-android-service`

## Restrict to alpha versions of Tor Browser

It's a good idea to test new PTs in alpha versions of tor browser first. This can be configured in the reproducible build system. 

## Add an option for a built-in bridge (optional)

It's recommended when integrating a new PT to first allow test users to configure it as a manual bridge by providing a custom bridge line. Once the PT has been thoroughly tested this way, bridge lines can be added as a built-in bridge option.

