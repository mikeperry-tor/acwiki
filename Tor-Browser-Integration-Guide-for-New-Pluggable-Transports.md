Note: this guide is a work in progress and may be incomplete.

This is a guide for anti-censorship team members on how to add support for a new pluggable transport (PT) to Tor Browser.

## Build the PT client reproducibly

The first step is to create an [rbm](https://rbm.torproject.org/) project in tor-browser-build to build the pluggable transport client reproducibly on all platforms. This project will have a `config` file and a `build` script. See examples of projects for existing PTs:
- [lyrebird](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/tree/main/projects/lyrebird)
- [conjure](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/tree/main/projects/conjure)

This guide will give a brief overview of how to write these files, more complete documentation is available in the [rbm repository](https://gitlab.torproject.org/tpo/applications/rbm/-/tree/main/doc).

#### Writing the `config`

Make sure to define the following things in the config:

```yaml
git_url: [URL of the git repository that the PT client can be cloned from]
git_hash: [git commit hash of the version of the PT to include in the build]
```
Don't change the `filename` or `container` arguments, all projects should be built in a container and the filename will place the result of the build in a spot that rbm can find it later.

```yaml
filename: '[% project %]-[% c("version") %]-[% c("var/osname") %]-[% c("var/build_id") %].tar.[% c("compress_tar") %]'
container:
  use_container: 1
```
The input files should list all dependencies for building the project.

```yaml
input_files:
  - project: container-image
  - name: go
    project: go
  - name: '[% c("var/compiler") %]'
    project: '[% c("var/compiler") %]'
    enable: '[% c("var/android") %]'
  - name: go_vendor
    pkg_type: go_vendor
    project: conjure
    norec:
      sha256sum: d43631d17d8b6f78152bb4341f867bb22ad4444fec5573be1d34dac968202cda
    target_replace:
      '^torbrowser-(?!testbuild).*': 'torbrowser-linux-x86_64'
  - name: go-licenses
    project: go-licenses
```
PTs written in Go and Rust can use vendored dependencies. To generate the vendored tarball for Go projects and calculate the sum, run the following command from the top of the tor-browser-build repo:
```bash
./rbm/rbm build [PT project name] --step go_vendor --target alpha --target torbrowser-linux-x86_64
```
there will be a vendered tarball in `out/[PT project name]/go_vendor/`. You can then run
```bash
sha256sum [path to tarball]
```
For Rust projects, the process is similar. See the [`lox-wasm` config](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/blob/7be3d4ea15728ba563a8590f54d75e5e03b7f150/projects/lox-wasm/config#L21) for the input files and then run
```bash
./rbm/rbm build [PT project name] --step cargo_vendor --target alpha --target torbrowser-linux-x86_64
```
to generate the tarball.

If the PT is written in Go, the [`go-licences`](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/tree/main/projects/go-licenses) project is also required.

#### Writing the `build` script

Most of the build script can be copied from an existing PT project and modified slightly for the new PT. 

```bash
#!/bin/bash
[% c("var/set_default_env") -%]
[% pc('go', 'var/setup', { go_tarfile => c('input_files_by_name/go') }) %]
distdir=/var/tmp/dist/[% project %]
mkdir -p $distdir

tar -C /var/tmp/dist -xf [% c('input_files_by_name/go-licenses') %]
export PATH=$PATH:/var/tmp/dist/go-licenses

[% IF c("var/android") -%]
  [% pc(c('var/compiler'), 'var/setup', { compiler_tarfile => c('input_files_by_name/' _ c('var/compiler')) }) %]
  # We need to explicitly set CGO_ENABLED with Go 1.13.x as the Android build
  # breaks otherwise.
  export CGO_ENABLED=1
[% END -%]
```

Then the project is extracted into a temporary build directory, along with the vendored dependencies

```bash
mkdir -p /var/tmp/build
tar -C /var/tmp/build -xf [% project %]-[% c('version') %].tar.gz
cd /var/tmp/build/[% project %]-[% c('version') %]

tar -xf $rootdir/[% c('input_files_by_name/go_vendor') %]
```

and built
```bash
go build -mod=vendor -ldflags '-X main.newptVersion=[% c("version") %] -s[% IF c("var/android") %] -checklinkname=0[% END %]' ./cmd/newpt
```
Copy the compiled client binary to the right filename, depending on the platform.
```bash
cp -a newpt[% IF c("var/windows") %].exe[% END %] $distdir
```

Save the licences, if necessary
```bash
go-licenses save ./cmd/newpt --save_path=$distdir/licenses
```

and finally package everything up so it can be used by the `tor-expert-bundle` project.
```bash
cd $distdir
[% c('tar', {
        tar_src => [ '.' ],
        tar_args => '-czf ' _ dest_dir _ '/' _ c('filename'),
    }) %]
```

After the project has been defined, it can be tested and debugged by building just the project directly:
```bash
./rbm/rbm build $project --target nightly --target $platform
```
This saves time and resources, rather than doing a full tor browser build.

## Add the client binary to the tor-expert-bundle

The `tor-expert-bundle` project groups together and packages up PT binaries and built-in bridge lines so that they can be extracted to the right place for each platform's browser build.

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
index f2d0d96c..36490718 100644
--- a/projects/tor-expert-bundle/build
+++ b/projects/tor-expert-bundle/build
@@ -33,6 +33,8 @@ pt_licenses $distdir/docs/lyrebird.txt
   tar -xkf $rootdir/[% c('input_files_by_name/conjure') %]
   pt_licenses $distdir/docs/conjure.txt
 [% END -%]
+tar -xkf $rootdir/[% c('input_files_by_name/newpt') %]
+pt_licenses $distdir/docs/newpt.txt
 
 # add per-platform pt extension
 awk '{gsub(/\$\{pt_extension\}/, "[% c("var/pt_extension") %]"); print}' $rootdir/pt_config.json > pt_config.json
```

## Set the ClientTransportPlugin line for desktop platforms

There is a single [pt_config.json](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/blob/7be3d4ea15728ba563a8590f54d75e5e03b7f150/projects/tor-expert-bundle/pt_config.json) for all `ClientTransportPlugin` and builtin bridge lines for all platforms.

```diff
diff --git a/projects/tor-expert-bundle/pt_config.json b/projects/tor-expert-bundle/pt_config.json
index 10f48d57..fc0afe73 100644
--- a/projects/tor-expert-bundle/pt_config.json
+++ b/projects/tor-expert-bundle/pt_config.json
@@ -3,7 +3,8 @@
   "pluggableTransports" : {
     "lyrebird" : "ClientTransportPlugin meek_lite,obfs2,obfs3,obfs4,scramblesuit,webtunnel exec ${pt_path}lyrebird${pt_extension}",
     "snowflake": "ClientTransportPlugin snowflake exec ${pt_path}lyrebird${pt_extension}",
-    "conjure" : "ClientTransportPlugin conjure exec ${pt_path}conjure-client${pt_extension} -registerURL https://registration.refraction.network/api"
+    "conjure" : "ClientTransportPlugin conjure exec ${pt_path}conjure-client${pt_extension} -registerURL https://registration.refraction.network/api",
+    "newpt" : "ClientTransportPlugin newpt exec ${pt_path}newpt${pt_extension} -arg1 foo -arg2 bar"
   },
   "bridges" : {
     "meek-azure" : [
```

Once these are updated, the new PT will work on all desktop platforms of Tor Browser. The next step covers how to add support in Android platforms.

## Adding PT support in Android

Integrating a new PT into android builds of Tor Browser requires modifications to a few different repositories.

#### Create a patch for `tor-onion-proxy-library`

This project is not maintained by the applications team at TPO, so changes to this repository are handled by the creation of patches that are applied at build time. Use the following procedure to create a patch:

1. Clone the upstream [repository](https://github.com/thaliproject/Tor_Onion_Proxy_Library.git)
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
4. Modify the `TorConfigBuilder` class

   In [`TorConfigBuilder.java`](https://github.com/thaliproject/Tor_Onion_Proxy_Library/blob/master/universal/src/main/java/com/msopentech/thali/toronionproxy/TorConfigBuilder.java), modify the `configurePluggableTransportsFromSettings` method to add existence and executable checks for your new PT.
   <details>
   <summary>example diff</summary>

   ```diff
   diff --git a/universal/src/main/java/com/msopentech/thali/toronionproxy/TorConfigBuilder.java b/universal/src/main/java/com/msopentech/thali/toronionproxy/TorConfigBuilder.java
   --- a/universal/src/main/java/com/msopentech/thali/toronionproxy/TorConfigBuilder.java
   +++ b/universal/src/main/java/com/msopentech/thali/toronionproxy/TorConfigBuilder.java
   @@ -109,8 +109,8 @@ public final class TorConfigBuilder {
   -    public TorConfigBuilder configurePluggableTransportsFromSettings(File pluggableTransportObfs, File pluggableTransportSnow) throws IOException {
   -        if (pluggableTransportObfs == null  || pluggableTransportSnow == null) {
   +    public TorConfigBuilder configurePluggableTransportsFromSettings(File pluggableTransportObfs, File pluggableTransportSnow, File pluggableTransportNewpt) throws IOException {
   +        if (pluggableTransportObfs == null  || pluggableTransportSnow == null || pluggableTransportConjure == null) {
                return this;
            }

   @@ -124,6 +124,11 @@ public final class TorConfigBuilder {
   +        if (!pluggableTransportNewpt.exists()) {
   +            throw new IOException("Conjure binary does not exist: " + pluggableTransportNewpt
   +                    .getCanonicalPath());
   +        }
   +

   @@ -134,8 +139,12 @@ public final class TorConfigBuilder {
   +        if (!pluggableTransportNewpt.canExecute()) {
   +            throw new IOException("Conjure binary is not executable: " + pluggableTransportNewpt
   +                    .getCanonicalPath());
   +        }

   -        transportPlugin(pluggableTransportObfs.getCanonicalPath(), pluggableTransportSnow.getCanonicalPath());
   +        transportPlugin(pluggableTransportObfs.getCanonicalPath(), pluggableTransportSnow.getCanonicalPath(), pluggableTransportNewpt.getCanonicalPath());
            return this;
        }
   ```
   </details>

   And add the `ClientTransportPlugin` line to the `transportPlugin` method in the same file:
   <details>
   <summary>example diff</summary>

   ```diff
   @@ -502,9 +511,10 @@ public final class TorConfigBuilder {

   -    public TorConfigBuilder transportPlugin(String obfsPath, String snowPath) {
   +    public TorConfigBuilder transportPlugin(String obfsPath, String snowPath, String newptPath) {
         buffer.append("ClientTransportPlugin meek_lite,obfs3,obfs4 exec ").append(obfsPath).append('\n');
         buffer.append("ClientTransportPlugin snowflake exec ").append(snowPath).append(" -url https://snowflake-broker.torproject.net.global.prod.fastly.net/ -front cdn.sstatic.net -ice stun:stun.l.google.com:19302,stun:stun.voip.blackberry.com:3478,stun:stun.altar.com.pl:3478,stun:stun.antisip.com:3478,stun:stun.bluesip.net:3478,stun:stun.dus.net:3478,stun:stun.epygi.com:3478,stun:stun.sonetel.com:3478,stun:stun.sonetel.net:3478,stun:stun.stunprotocol.org:3478,stun:stun.uls.co.za:3478,stun:stun.voipgate.com:3478,stun:stun.voys.nl:3478\n");
   +     buffer.append("ClientTransportPlugin newpt exec ").append(newptPath).append("-arg1 foo -arg2 bar\n");
         return this;
        }
   ```
   </details>
5. Commit the changes
   ```
   git commit -am "Bug XXXXX: Add newpt support
   ```
6. Export a patch file of your commit
   ```
   git format-patch [previous commit hash]
   ```

This will produce a file called `0001-Bug-XXXXX-Add-newpt-support.patch`. Copy this into the [`tor-onion-proxy-library` project](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/tree/main/projects/tor-onion-proxy-library) in `tor-browser-build`. Make sure you `git add` this file when making your commit so that it gets included with the rest of your changes to `tor-browser-build`.

Returning to the `tor-browser-build` repository, now modify the [config file](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/blob/main/projects/tor-onion-proxy-library/config) for the `tor-onion-proxy-library` project to include the new patch file as an input.

```diff
diff --git a/projects/tor-onion-proxy-library/config b/projects/tor-onion-proxy-library/config
--- a/projects/tor-onion-proxy-library/config
+++ b/projects/tor-onion-proxy-library/config
@@ -44,3 +44,4 @@ input_files:
   - filename: gradle.patch
   - filename: 0001-Bug-33931-Filter-bridges-in-stream-by-type.patch
   - filename: 0001-Bug-30318-Add-snowflake-support.patch
+  - filename: 0001-Bug-XXXXX-Add-newpt-support.patch
```
Next, modify the [build script](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/blob/main/projects/tor-onion-proxy-library/build) to copy the PT binary from `tor-expert-bundle` to the right location and apply the new patch.

```diff
diff --git a/projects/tor-onion-proxy-library/build b/projects/tor-onion-proxy-library/build
--- a/projects/tor-onion-proxy-library/build
+++ b/projects/tor-onion-proxy-library/build
@@ -22,6 +22,7 @@ cd /var/tmp/build/[% project %]-[% c('version') %]
 patch -p1 < $rootdir/0001-Bug-30318-Add-snowflake-support.patch
+patch -p1 < $rootdir/0001-Bug-XXXXX-Add-newpt-support.patch
 
 [% FOREACH arch = ['armv7', 'aarch64', 'x86', 'x86_64'] -%]
   # Extract tor-expert-bundle
@@ -36,12 +37,16 @@ patch -p1 < $rootdir/0001-Bug-30318-Add-snowflake-support.patch
     cp $ptdir/snowflake-client external/pluto/bin/armeabi-v7a/
     cp $ptdir/snowflake-client external/pluto/bin/armeabi/
+    cp $ptdir/newpt-client external/pluto/bin/armeabi-v7a/
+    cp $ptdir/newpt-client external/pluto/bin/armeabi/
   [% ELSIF arch == "aarch64" -%]
     cp $ptdir/snowflake-client external/pluto/bin/arm64-v8a/
+    cp $ptdir/newpt-client external/pluto/bin/arm64-v8a/
   [% ELSE -%]
     cp $ptdir/snowflake-client external/pluto/bin/[% arch %]/
+    cp $ptdir/newpt-client external/pluto/bin/[% arch %]/
   [% END -%]
 [% END -%]
```

#### Create a patch for `tor-android-service`

The `tor-android-service` project is maintained by the applications team and has [its own repository](https://gitlab.torproject.org/tpo/applications/tor-android-service). You will need to open a merge request with changes to the repository to complete the android builds.

Update the call to `configurePluggableTransportsFromSettings` to match the method signature made in the patch to `tor-onion-proxy-library` above. What this looks like will depend on how many pluggable transports there are. For the patch above, it will look like this:
```diff
diff --git a/service/src/main/java/org/torproject/android/service/TorService.java b/service/src/main/java/org/torproject/android/service/TorService.java
--- a/service/src/main/java/org/torproject/android/service/TorService.java
+++ b/service/src/main/java/org/torproject/android/service/TorService.java
@@ -369,8 +369,10 @@ public final class TorService extends Service implements TorServiceConstants, Or
             File pluggableTransportSnow = new File(nativeDir, "libSnowflake.so");
             if(!pluggableTransportSnow.canExecute()) pluggableTransportSnow.setExecutable(true);
+            File pluggableTransportNewpt = new File(nativeDir, "libNewpt.so");
+            if(!pluggableTransportNewpt.canExecute()) pluggableTransportNewpt.setExecutable(true);
 
-            builder.configurePluggableTransportsFromSettings(pluggableTransportObfs, pluggableTransportSnow);
+            builder.configurePluggableTransportsFromSettings(pluggableTransportObfs, pluggableTransportSnow, pluggableTransportNewpt);
```
That's it! There is a bit of a chicken-and-egg problem here, where the merge request with these changes will have to merged before the git hash of the `tor-android-service` library can be updated to include the new version. Negotiate with the applications team.

## Restrict to alpha versions of Tor Browser

It's a good idea to test new PTs in alpha versions of tor browser first. This can be configured in the reproducible build system. 

## Add an option for a built-in bridge (optional)

It's recommended when integrating a new PT to first allow test users to configure it as a manual bridge by providing a custom bridge line. Once the PT has been thoroughly tested this way, bridge lines can be added as a built-in bridge option.

