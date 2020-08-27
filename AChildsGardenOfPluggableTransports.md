Here is an exploration of [[PluggableTransports|pluggable transports]], how they look on the wire.

Pluggable transports disguise Tor traffic for the evading of network censorship. Some transports try to make the traffic look like another protocol, and others try to make it look random. Some transports are aimed at evading IP-based blocks rather than content-based blocks. If you want to see what running Tor with a pluggable transport is like, download a recent version of Tor Browser (version 3.6 or later), and say **Yes** to the question _Does your Internet Service Provider (ISP) block or otherwise censor connections to the Tor Network?_
 * [Tor Browser download page](https://www.torproject.org/download/download-easy)

We'll look at hex dumps and text logs, outside and inside encryption, sometimes paying attention to packet boundaries and sometimes ignoring them, in order to discover what makes each transport unique and wonderful.

# Ordinary Tor

```
tor 0.2.4.21-1
```

Why use pluggable transports? It's because it's possible to fingerprint ordinary Tor traffic based on byte patterns that appear in it. Here's a hex dump of the first thing that a client sends to its entry node. It is actually a TLS [Client Hello](https://tools.ietf.org/html/rfc5246#section-7.4.1.2) message, as the outer layer of the Tor protocol is TLS. Some parts are colored according to their meaning: [[span(cipher suite list,style=background:skyblue)]], [[span(server name,style=background:limegreen)]], and [[span(TLS extensions,style=background:palegoldenrod)]].
```
<pre style="background:cornsilk">
00000000  16 03 01 00 e5 01 00 00  e1 03 03 d2 23 9e 55 3d ........ ....#.U=
00000010  21 cc 80 d5 2b 34 4b e1  be 90 04 c6 d0 f8 f9 a5 !...+4K. ........
00000020  56 3f 01 22 ef c6 e1 22  64 c5 0a 00 <span style="background:skyblue">00 30 c0 2b</span> V?."..." d...<span style="background:skyblue">.0.+</span>
00000030  <span style="background:skyblue">c0 2f c0 0a c0 09 c0 13  c0 14 c0 12 c0 07 c0 11</span> <span style="background:skyblue">./...... ........</span>
00000040  <span style="background:skyblue">00 33 00 32 00 45 00 39  00 38 00 88 00 16 00 2f</span> <span style="background:skyblue">.3.2.E.9 .8...../</span>
00000050  <span style="background:skyblue">00 41 00 35 00 84 00 0a  00 05 00 04 00 ff</span> 01 00 <span style="background:skyblue">.A.5.... ......</span>..
00000060  <span style="background:palegoldenrod">00 88 00 00 00 17 00 15  00 00 12 <span style="background:limegreen">77 77 77 2e 6b</span></span> <span style="background:palegoldenrod">........ ...<span style="background:limegreen">www.k</span></span>
00000070  <span style="background:palegoldenrod"><span style="background:limegreen">66 33 69 61 6d 6d 6e 79  70 2e 63 6f 6d</span> 00 0b 00</span> <span style="background:palegoldenrod"><span style="background:limegreen">f3iammny p.com</span>...</span>
00000080  <span style="background:palegoldenrod">04 03 00 01 02 00 0a 00  34 00 32 00 0e 00 0d 00</span> <span style="background:palegoldenrod">........ 4.2.....</span>
00000090  <span style="background:palegoldenrod">19 00 0b 00 0c 00 18 00  09 00 0a 00 16 00 17 00</span> <span style="background:palegoldenrod">........ ........</span>
000000A0  <span style="background:palegoldenrod">08 00 06 00 07 00 14 00  15 00 04 00 05 00 12 00</span> <span style="background:palegoldenrod">........ ........</span>
000000B0  <span style="background:palegoldenrod">13 00 01 00 02 00 03 00  0f 00 10 00 11 00 23 00</span> <span style="background:palegoldenrod">........ ......#.</span>
000000C0  <span style="background:palegoldenrod">00 00 0d 00 20 00 1e 06  01 06 02 06 03 05 01 05</span> <span style="background:palegoldenrod">.... ... ........</span>
000000D0  <span style="background:palegoldenrod">02 05 03 04 01 04 02 04  03 03 01 03 02 03 03 02</span> <span style="background:palegoldenrod">........ ........</span>
000000E0  <span style="background:palegoldenrod">01 02 02 02 03 00 0f 00  01 01</span>                   <span style="background:palegoldenrod">........ ..</span>
</pre>
```
Here's the first thing the relay sends back in response, a TLS [Server Hello](https://tools.ietf.org/html/rfc5246#section-7.4.1.3) along with a [certificate](https://tools.ietf.org/html/rfc5246#section-7.4.2) and key exchange message.
```
<blockquote>
<pre style="background:lavender">
00000000  16 03 03 00 3e 02 00 00  3a 03 03 5c 67 84 3c 3a ....&gt;... :..\g.&lt;:
00000010  7c 45 d5 ee 2d 64 2d 12  e7 0f b4 2d 14 78 40 f0 |E..-d-. ...-.x@.
00000020  59 58 b2 2e 70 e5 e4 c6  5c 2b 25 00 <span style="background:skyblue">c0 2f</span> 00 <span style="background:palegoldenrod">00</span> YX..p... \+%.<span style="background:skyblue">./</span>.<span style="background:palegoldenrod">.</span>
00000030  <span style="background:palegoldenrod">12 ff 01 00 01 00 00 0b  00 04 03 00 01 02 00 0f</span> <span style="background:palegoldenrod">........ ........</span>
00000040  <span style="background:palegoldenrod">00 01 01</span> 16 03 03 01 cb  0b 00 01 c7 00 01 c4 00 <span style="background:palegoldenrod">...</span>..... ........
00000050  01 c1 30 82 01 bd 30 82  01 26 a0 03 02 01 02 02 ..0...0. .&amp;......
00000060  08 30 5c fb 15 5d 79 91  84 30 0d 06 09 2a 86 48 .0\..]y. .0...*.H
00000070  86 f7 0d 01 01 05 05 00  30 24 31 22 30 20 06 03 ........ 0$1"0 ..
00000080  55 04 03 0c 19 77 77 77  2e 75 64 6f 6e 69 74 64 U....www .udonitd
00000090  67 74 34 71 66 77 79 62  70 70 2e 63 6f 6d 30 1e gt4qfwyb pp.com0.
000000A0  17 0d 31 33 31 30 31 35  30 30 30 30 30 30 5a 17 ..131015 000000Z.
000000B0  0d 31 34 30 39 31 30 30  30 30 30 30 30 5a 30 1e .1409100 00000Z0.
000000C0  31 1c 30 1a 06 03 55 04  03 0c 13 <span style="background:limegreen">77 77 77 2e 77</span> 1.0...U. ...<span style="background:limegreen">www.w</span>
000000D0  <span style="background:limegreen">6a 72 64 6d 79 61 72 6e  35 76 2e 6e 65 74</span> 30 81 <span style="background:limegreen">jrdmyarn 5v.net</span>0.
000000E0  9f 30 0d 06 09 2a 86 48  86 f7 0d 01 01 01 05 00 .0...*.H ........
000000F0  03 81 8d 00 30 81 89 02  81 81 00 f3 37 a0 a9 da ....0... ....7...
00000100  2b b0 68 66 a1 9f 7f f1  7e e6 09 1a 10 69 fe 35 +.hf.... ~....i.5
00000110  19 01 b7 92 70 94 fc 8e  bc 70 da 58 d7 35 dd cb ....p... .p.X.5..
00000120  8d fe 20 84 a6 c1 b6 38  9f 92 7e 04 8a b8 e4 85 .. ....8 ..~.....
00000130  32 5d 17 22 24 bd 2e ba  29 58 db 3c 5e d1 59 01 2]."$... )X.&lt;^.Y.
00000140  3a 01 3c f3 ea 6c ab bf  95 67 59 4e 6c 4c 90 e7 :.&lt;..l.. .gYNlL..
00000150  79 f2 5c c8 30 d7 2c b0  80 67 04 1d 23 89 ba f8 y.\.0.,. .g..#...
00000160  3c c4 02 0c 2d fb 79 e9  af 8c 32 fb 44 9c a0 2a &lt;...-.y. ..2.D..*
00000170  18 41 17 11 71 09 f3 25  4c a6 85 02 03 01 00 01 .A..q..% L.......
00000180  30 0d 06 09 2a 86 48 86  f7 0d 01 01 05 05 00 03 0...*.H. ........
00000190  81 81 00 85 31 40 82 25  f2 f1 38 44 68 3d ce 78 ....1@.% ..8Dh=.x
000001A0  04 34 9c b9 b9 31 2b ca  a9 a4 72 d1 43 a0 9a e7 .4...1+. ..r.C...
000001B0  c4 b6 25 3f df 43 dc cd  cf 11 ae 74 80 e0 4b 37 ..%?.C.. ...t..K7
000001C0  c9 d5 dc ce 27 9e 51 ec  e4 c5 7e 06 c6 10 38 9d ....'.Q. ..~...8.
000001D0  64 d3 d4 e0 f9 0a 4b ee  fe 0d 55 c5 dd 83 7b ad d.....K. ..U...{.
000001E0  43 ea 0c aa bd f0 0c 40  40 0e e8 64 a1 7b f5 39 C......@ @..d.{.9
000001F0  54 75 d6 de 05 68 c4 14  77 c3 c5 cb bb 8f c6 16 Tu...h.. w.......
00000200  8d 0b 2a 16 0f 68 8c d1  24 9a 82 9b 1c 14 18 85 ..*..h.. $.......
00000210  fe 4c ef 16 03 03 00 cd  0c 00 00 c9 03 00 17 41 .L...... .......A
00000220  04 8b 9a ef 29 ec 30 32  f6 e8 ef 2f 90 b1 e0 59 ....).02 .../...Y
00000230  e9 54 f8 16 70 e6 a1 fa  86 5f e7 6f 9e be b4 88 .T..p... ._.o....
00000240  c7 5b 55 6a 76 06 e2 3e  b1 41 ea dd fe 50 f0 62 .[Ujv..&gt; .A...P.b
00000250  42 c5 e7 60 05 86 e1 a8  dd de c4 10 76 f1 fc bd B..`.... ....v...
00000260  c6 06 01 00 80 91 55 20  68 73 f3 e0 cd 28 75 58 ......U  hs...(uX
00000270  0e e3 f5 6a d7 83 78 b2  ed 4d d1 06 5d 1e 88 c7 ...j..x. .M..]...
00000280  30 ef 24 1e f4 29 3d 40  c9 ae 9d 40 71 96 26 e4 0.$..)=@ ...@q.&amp;.
00000290  19 1f 29 8d 88 da a5 43  31 9c 05 ce f8 6d 32 ed ..)....C 1....m2.
000002A0  99 1b 87 b7 a0 30 ea 73  34 9f e8 e4 48 39 68 fb .....0.s 4...H9h.
000002B0  7a 52 82 0a 50 57 50 e2  e7 82 ce c6 db cf 53 85 zR..PWP. ......S.
000002C0  4f e0 7e 50 75 0d 4d 9e  07 b9 8d 7d 9f 18 81 c9 O.~Pu.M. ...}....
000002D0  3a ad dc 40 4d 4f 0f 74  1d 85 f5 d8 a3 f1 26 06 :..@MO.t ......&amp;.
000002E0  20 86 35 ee 6a 16 03 03  00 04 0e 00 00 00        .5.j... ......
</pre>
</blockquote>
```

It's easier to read these messages after they have been dissected by Wireshark. Here is the meaning of the Client Hello. The parts of the dissection that correspond to the hex dump above are colored the same way: [[span(cipher suite list,style=background:skyblue)]], [[span(server name,style=background:limegreen)]], and [[span(TLS extensions,style=background:palegoldenrod)]].

```
<pre style="background:cornsilk">
Secure Sockets Layer
    SSL Record Layer: Handshake Protocol: Client Hello
        Content Type: Handshake (22)
        Version: TLS 1.0 (0x0301)
        Length: 229
        Handshake Protocol: Client Hello
            Handshake Type: Client Hello (1)
            Length: 225
            Version: TLS 1.2 (0x0303)
            Random
                gmt_unix_time: Sep 19, 2081 16:20:53.000000000 PDT
                random_bytes: 3d21cc80d52b344be1be9004c6d0f8f9a5563f0122efc6e1...
            Session ID Length: 0
<span style="background:skyblue">            Cipher Suites Length: 48
            Cipher Suites (24 suites)
                Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 (0xc02b)
                Cipher Suite: TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)
                Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA (0xc00a)
                Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA (0xc009)
                Cipher Suite: TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (0xc013)
                Cipher Suite: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (0xc014)
                Cipher Suite: TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA (0xc012)
                Cipher Suite: TLS_ECDHE_ECDSA_WITH_RC4_128_SHA (0xc007)
                Cipher Suite: TLS_ECDHE_RSA_WITH_RC4_128_SHA (0xc011)
                Cipher Suite: TLS_DHE_RSA_WITH_AES_128_CBC_SHA (0x0033)
                Cipher Suite: TLS_DHE_DSS_WITH_AES_128_CBC_SHA (0x0032)
                Cipher Suite: TLS_DHE_RSA_WITH_CAMELLIA_128_CBC_SHA (0x0045)
                Cipher Suite: TLS_DHE_RSA_WITH_AES_256_CBC_SHA (0x0039)
                Cipher Suite: TLS_DHE_DSS_WITH_AES_256_CBC_SHA (0x0038)
                Cipher Suite: TLS_DHE_RSA_WITH_CAMELLIA_256_CBC_SHA (0x0088)
                Cipher Suite: TLS_DHE_RSA_WITH_3DES_EDE_CBC_SHA (0x0016)
                Cipher Suite: TLS_RSA_WITH_AES_128_CBC_SHA (0x002f)
                Cipher Suite: TLS_RSA_WITH_CAMELLIA_128_CBC_SHA (0x0041)
                Cipher Suite: TLS_RSA_WITH_AES_256_CBC_SHA (0x0035)
                Cipher Suite: TLS_RSA_WITH_CAMELLIA_256_CBC_SHA (0x0084)
                Cipher Suite: TLS_RSA_WITH_3DES_EDE_CBC_SHA (0x000a)
                Cipher Suite: TLS_RSA_WITH_RC4_128_SHA (0x0005)
                Cipher Suite: TLS_RSA_WITH_RC4_128_MD5 (0x0004)
                Cipher Suite: TLS_EMPTY_RENEGOTIATION_INFO_SCSV (0x00ff)</span>
            Compression Methods Length: 1
            Compression Methods (1 method)
                Compression Method: null (0)
<span style="background:palegoldenrod">            Extensions Length: 136
            Extension: server_name
                Type: server_name (0x0000)
                Length: 23
                Server Name Indication extension
                    Server Name list length: 21
                    Server Name Type: host_name (0)
                    Server Name length: 18
                    <span style="background:limegreen">Server Name: www.kf3iammnyp.com</span>
            Extension: ec_point_formats
                Type: ec_point_formats (0x000b)
                Length: 4
                EC point formats Length: 3
                Elliptic curves point formats (3)
                    EC point format: uncompressed (0)
                    EC point format: ansiX962_compressed_prime (1)
                    EC point format: ansiX962_compressed_char2 (2)
            Extension: elliptic_curves
                Type: elliptic_curves (0x000a)
                Length: 52
                Elliptic Curves Length: 50
                Elliptic curves (25 curves)
                    Elliptic curve: sect571r1 (0x000e)
                    Elliptic curve: sect571k1 (0x000d)
                    Elliptic curve: secp521r1 (0x0019)
                    Elliptic curve: sect409k1 (0x000b)
                    Elliptic curve: sect409r1 (0x000c)
                    Elliptic curve: secp384r1 (0x0018)
                    Elliptic curve: sect283k1 (0x0009)
                    Elliptic curve: sect283r1 (0x000a)
                    Elliptic curve: secp256k1 (0x0016)
                    Elliptic curve: secp256r1 (0x0017)
                    Elliptic curve: sect239k1 (0x0008)
                    Elliptic curve: sect233k1 (0x0006)
                    Elliptic curve: sect233r1 (0x0007)
                    Elliptic curve: secp224k1 (0x0014)
                    Elliptic curve: secp224r1 (0x0015)
                    Elliptic curve: sect193r1 (0x0004)
                    Elliptic curve: sect193r2 (0x0005)
                    Elliptic curve: secp192k1 (0x0012)
                    Elliptic curve: secp192r1 (0x0013)
                    Elliptic curve: sect163k1 (0x0001)
                    Elliptic curve: sect163r1 (0x0002)
                    Elliptic curve: sect163r2 (0x0003)
                    Elliptic curve: secp160k1 (0x000f)
                    Elliptic curve: secp160r1 (0x0010)
                    Elliptic curve: secp160r2 (0x0011)
            Extension: SessionTicket TLS
                Type: SessionTicket TLS (0x0023)
                Length: 0
                Data (0 bytes)
            Extension: signature_algorithms
                Type: signature_algorithms (0x000d)
                Length: 32
                Data (32 bytes)
            Extension: Heartbeat
                Type: Heartbeat (0x000f)
                Length: 1
                Mode: Peer allowed to send requests (1)</span>
</pre>
```
The [[span(cipher suite list,style=background:skyblue)]] is an interesting part of the message. Ticket #4744 has the story of how the Great Firewall of China blocked Tor in 2011 by looking for a distinctive cipher suite list. In response, Tor changed its cipher suite list to be the same as Firefox's. You can still tell the difference, though: The [[span(TLS extensions,style=background:palegoldenrod)]] used by Firefox are not the same as those used by Tor. For example, above there is "Elliptic curves (25 curves)", but Firefox actually has "Elliptic curves (3 curves)". You can see a lot more Client Hellos at [[meek/SampleClientHellos]]. Also notice the randomly generated server name [[span(www.kf3iammnyp.com,style=background:limegreen)]].

Here is the meaning of the Server Hello and certificate.
```
<blockquote>
<pre style="background:lavender">
Secure Sockets Layer
    TLSv1.2 Record Layer: Handshake Protocol: Server Hello
        Content Type: Handshake (22)
        Version: TLS 1.2 (0x0303)
        Length: 62
        Handshake Protocol: Server Hello
            Handshake Type: Server Hello (2)
            Length: 58
            Version: TLS 1.2 (0x0303)
            Random
                gmt_unix_time: Feb 15, 2019 19:32:12.000000000 PST
                random_bytes: 3a7c45d5ee2d642d12e70fb42d147840f05958b22e70e5e4...
            Session ID Length: 0
<span style="background:skyblue">            Cipher Suite: TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)</span>
            Compression Method: null (0)
<span style="background:palegoldenrod">            Extensions Length: 18
            Extension: renegotiation_info
                Type: renegotiation_info (0xff01)
                Length: 1
                Renegotiation Info extension
                    Renegotiation info extension length: 0
            Extension: ec_point_formats
                Type: ec_point_formats (0x000b)
                Length: 4
                EC point formats Length: 3
                Elliptic curves point formats (3)
                    EC point format: uncompressed (0)
                    EC point format: ansiX962_compressed_prime (1)
                    EC point format: ansiX962_compressed_char2 (2)
            Extension: Heartbeat
                Type: Heartbeat (0x000f)
                Length: 1
                Mode: Peer allowed to send requests (1)</span>
    TLSv1.2 Record Layer: Handshake Protocol: Certificate
        Content Type: Handshake (22)
        Version: TLS 1.2 (0x0303)
        Length: 459
        Handshake Protocol: Certificate
            Handshake Type: Certificate (11)
            Length: 455
            Certificates Length: 452
            Certificates (452 bytes)
                Certificate Length: 449
                Certificate (id-at-commonName=www.wjrdmyarn5v.net)
                    signedCertificate
                        version: v3 (2)
                        serialNumber: 1568248196
                        signature (shaWithRSAEncryption)
                            Algorithm Id: 1.2.840.113549.1.1.5 (shaWithRSAEncryption)
                        issuer: rdnSequence (0)
                            rdnSequence: 1 item (id-at-commonName=www.udonitdgt4qfwybpp.com)
                                RDNSequence item: 1 item (id-at-commonName=www.udonitdgt4qfwybpp.com)
                                    RelativeDistinguishedName item (id-at-commonName=www.udonitdgt4qfwybpp.com)
                                        Id: 2.5.4.3 (id-at-commonName)
                                        DirectoryString: uTF8String (4)
                                            uTF8String: www.udonitdgt4qfwybpp.com
                        validity
                            notBefore: utcTime (0)
                                utcTime: 13-10-15 00:00:00 (UTC)
                            notAfter: utcTime (0)
                                utcTime: 14-09-10 00:00:00 (UTC)
                        subject: rdnSequence (0)
                            rdnSequence: 1 item (id-at-commonName=www.wjrdmyarn5v.net)
                                RDNSequence item: 1 item (id-at-commonName=www.wjrdmyarn5v.net)
                                    RelativeDistinguishedName item (id-at-commonName=www.wjrdmyarn5v.net)
                                        Id: 2.5.4.3 (id-at-commonName)
                                        DirectoryString: uTF8String (4)
                                            uTF8String: <span style="background:limegreen">www.wjrdmyarn5v.net</span>
                        subjectPublicKeyInfo
                            algorithm (rsaEncryption)
                                Algorithm Id: 1.2.840.113549.1.1.1 (rsaEncryption)
                            Padding: 0
                            subjectPublicKey: 30818902818100f337a0a9da2bb06866a19f7ff17ee6091a...
                    algorithmIdentifier (shaWithRSAEncryption)
                        Algorithm Id: 1.2.840.113549.1.1.5 (shaWithRSAEncryption)
                    Padding: 0
                    encrypted: 8531408225f2f13844683dce7804349cb9b9312bcaa9a472...
    TLSv1.2 Record Layer: Handshake Protocol: Server Key Exchange
        Content Type: Handshake (22)
        Version: TLS 1.2 (0x0303)
        Length: 205
        Handshake Protocol: Server Key Exchange
            Handshake Type: Server Key Exchange (12)
            Length: 201
    TLSv1.2 Record Layer: Handshake Protocol: Server Hello Done
        Content Type: Handshake (22)
        Version: TLS 1.2 (0x0303)
        Length: 4
        Handshake Protocol: Server Hello Done
            Handshake Type: Server Hello Done (14)
            Length: 0
</pre>
</blockquote>
```
The server has selected the cipher suite [[span(TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,style=background:skyblue)]]. It sends back its own [[span(TLS extensions,style=background:palegoldenrod)]]. The server has its own randomly generated [[span(server name,style=background:limegreen)]] appearing in its certificate. The names aren't a fixed byte pattern, but it's possible to fingerprint them by watching several handshakes and seeing that the names always fit the expected pattern, as [this Bro script does](https://github.com/sethhall/bro-junk-drawer/blob/007b3a833206770bc4b85b12c39e0e01b7b998a0/detect-tor.bro).

# obfs2

```
Tor version 0.2.5.12 (git-3731dd5c3071dcba).
obfs4proxy-0.0.5
```

[obfs2](https://gitweb.torproject.org/pluggable-transports/obfsproxy.git/tree/doc/obfs2/obfs2-protocol-spec.txt)
was the first in the obfs family of "look-like-nothing" pluggable transports.
It looks like a uniformly random byte stream to a naive observer.
It was inspired by [obfuscated-openssh](https://github.com/brl/obfuscated-openssh)
and is succeeded by [[#obfs3|obfs3]], [[#ScrambleSuit|ScrambleSuit]], and obfs4.
obfs2 is deprecated these days, because it can be identified
by a totally passive observer,
no active man in the middle needed,
as you'll see.

The idea behind obfs2 is basically: Send an encryption key, followed by
a magic number, a padding length, and some padding,
all encrypted by that key.
Derive a new key from both the client's and server's intial key,
and use that to encrypt all the application data.
Both client and server do the same operations,
differing only in a few constants.
We'll look at the client side. Here is the beginning of an obfs2 session.
The bytes that represent the
[[span(client/server _seed_,style=background:dodgerblue)]],
[[span(magic number,style=background:orange)]] (encrypted),
[[span(padlen,style=background:gold)]] (encrypted), and
[[span(padding,style=background:gray)]] (encrypted)
are highlighted.

```
<blockquote>
<pre style="background:lavender">
00000000  <span style="background:dodgerblue">6a 14 a6 00 2c b0 cd bc  b7 d7 a7 8f 65 c7 0b 37</span> <span style="background:dodgerblue">j...,... ....e..7</span>
00000010  <span style="background:orange">ef a8 d1 d7</span> <span style="background:gold">f4 04 7d 71</span>  <span style="background:gray">0d 67 bb 21 fa f9 bc 98</span> <span style="background:orange">....</span><span style="background:gold">..}q</span> <span style="background:gray">.g.!....</span>
00000020  <span style="background:gray">19 16 83 8c db 0a 51 e7  a9 2e 8c 3f 79 7d b3 64</span> <span style="background:gray">......Q. ...?y}.d</span>
00000030  <span style="background:gray">12 64 81 3d d7 d4 6b f4  70 68 bd 22 fc cf c8 6b</span> <span style="background:gray">.d.=..k. ph."...k</span>
00000040  <span style="background:gray">fd 05 c6 84 25 5a bd 19  46 94 bd a9 58 fa dd 6b</span> <span style="background:gray">....%Z.. F...X..k</span>
00000050  <span style="background:gray">6b 1f 7c dc 64 30 ce 42  3f 83 7d 79 db b8 7a 99</span> <span style="background:gray">k.|.d0.B ?.}y..z.</span>
00000060  <span style="background:gray">b2 84 b2 c1 f8 9a 4e b3  11 7c 9b d8 62 71 e5 25</span> <span style="background:gray">......N. .|..bq.%</span>
00000070  <span style="background:gray">18 ab 6e de 57 38 1f 80  2c f8 7b 3c</span>             <span style="background:gray">..n.W8.. ,.{&lt;</span>
</pre>
</blockquote>
<pre style="background:cornsilk">
00000000  <span style="background:dodgerblue">1c f1 11 91 af fb 8f 31  08 c7 a6 53 c3 84 10 a1</span> <span style="background:dodgerblue">.......1 ...S....</span>
00000010  <span style="background:orange">e5 d1 ec bb</span> <span style="background:gold">72 68 40 27</span>  <span style="background:gray">25 62 bb 8b c2 5b 57 1a</span> <span style="background:orange">....</span><span style="background:gold">rh@'</span> <span style="background:gray">%b...[W.</span>
00000020  <span style="background:gray">f5 04 17 c3 8c 4d 99 9a  b9 3f a7 41 d8 99 61 88</span> <span style="background:gray">.....M.. .?.A..a.</span>
00000030  <span style="background:gray">f8 71 16 66 d3 a3 20 1a  71 f1 ec 44 95 e7 07 05</span> <span style="background:gray">.q.f.. . q..D....</span>
00000040  <span style="background:gray">0b cd 90 75 72 3f f9 37</span>                          <span style="background:gray">...ur?.7</span> 
</pre>
<pre style="background:cornsilk">
00000048  6c af c1 f1 c9 4c b8 25  ca 05 60 47 f7 ba d1 32 l....L.% ..`G...2
00000058  83 22 24 f1 14 97 5b 63  5e a9 02 82 c6 a0 9e 38 ."$...[c ^......8
00000068  c9 45 d2 fd b8 fe ca 8d  68 d5 cf b5 8f d8 cb 55 .E...... h......U
00000078  68 b9 1a 63 56 44 fc cb  c8 41 3b 3e d9 c0 69 38 h..cVD.. .A;&gt;..i8
00000088  08 9e 61 ef 57 11 af c4  cf 1d 37 7f 39 8f b1 a1 ..a.W... ..7.9...
</pre>
<blockquote>
<pre style="background:lavender">
0000007C  e2 3a 1d 5b 74 3d f7 32  76 1f e1 24 77 5c da 83 .:.[t=.2 v..$w\..
0000008C  9f cd 39 63 6b 7c 53 90  e9 fe ae 2c 60 a7 3e f4 ..9ck|S. ...,`.&gt;.
0000009C  f0 0b 18 2f 6e a4 0f 88  c6 06 9c d5 88 54 74 af .../n... .....Tt.
000000AC  e3 6d f4 ad 46 ab e4 de  24 fb 12 80 50 9d a8 f7 .m..F... $...P...
000000BC  2e f0 5c 7c fb 75 81 02  a5 28 98 66 63 83 dd 31 ..\|.u.. .(.fc..1
</pre>
</blockquote>
```

You can read the client seed directly from the packet trace.
In this case it is [[span(1cf11191affb8f3108c7a653c38410a1,style=background:dodgerblue)]].
From the seed we derive a key and an initial counter value for
[AES-CTR](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Counter_.28CTR.29) mode.
Try it yourself using Python and the [PyCrypto](https://www.dlitz.net/software/pycrypto/) package:
```
>>> from Crypto.Cipher import AES
>>> from Crypto.Util import Counter
>>> from hashlib import sha256
>>> seed = "1cf11191affb8f3108c7a653c38410a1".decode("hex")
>>> secret = sha256("Initiator obfuscation padding" + seed + "Initiator obfuscation padding").digest()
>>> key = secret[0:16]
>>> ctr = secret[16:32]
>>> aes = AES.new(key, AES.MODE_CTR, counter=Counter.new(128, initial_value=long(ctr.encode("hex"), 16)))
>>> ciphertext = "e5d1ecbb726840272562bb8bc25b571a".decode("hex")
>>> print aes.decrypt(ciphertext).encode("hex")
2bf5ca7e00000030eef53458d509c177
```
The result of decrypting the first 16 bytes of ciphertext is
[[span(2bf5ca7e,style=background:orange)]][[span(00000030,style=background:gold)]][[span(eef53458d509c177,style=background:gray)]].
The first 4 bytes,
[[span(2bf5ca7e,style=background:orange)]],
are always the same.
They are a magic number that identifies an obfs2 stream.
(It says "2bfscate", get it?)
The next 4 bytes,
[[span(00000030,style=background:gold)]],
are the number of padding bytes that follow.
In this case there are 0x30 = 48 padding bytes.
(We deliberately used short padding for this example;
normally the padding is a random length between 0 and 8192 bytes.)
The next 48 bytes, starting with
[[span(eef53458d509c177,style=background:gray)]],
are just random padding.

The server does the same operations.
Try the Python example again, substituting
[[span(6a14a6002cb0cdbcb7d7a78f65c70b37,style=background:dodgerblue)]]
for the seed
and `"Responder obfuscated data"` for `"Initiator obfuscated data"`.
You should get a padlen of
[[span(00000064,style=background:gold)]].

After all the padding is encrypted and consumed,
both sides switch to new keys and counter values
that are derived deterministically from both seed values.
Then they simply decrypt forever.
Here is how it looks for the client side in the example above.
You can see the beginning of the Tor TLS Client Hello
at the beginning of the plaintext.
```
>>> init_seed = "1cf11191affb8f3108c7a653c38410a1".decode("hex")
>>> resp_seed = "6a14a6002cb0cdbcb7d7a78f65c70b37".decode("hex")
>>> init_secret = sha256("Initiator obfuscated data" + init_seed + resp_seed + "Initiator obfuscated data").digest()
>>> init_key = init_secret[0:16]
>>> init_ctr = init_secret[16:32]
>>> aes = AES.new(init_key, AES.MODE_CTR, counter=Counter.new(128, initial_value=long(init_ctr.encode("hex"), 16)))
>>> ciphertext = "6cafc1f1c94cb825ca056047f7bad132".decode("hex")
>>> print aes.decrypt(ciphertext).encode("hex")
16030100ef010000eb0303923472ed41
```

# obfs3

```
Tor version 0.2.5.12 (git-3731dd5c3071dcba).
obfs4proxy-0.0.5
```

[obfs3](https://gitweb.torproject.org/pluggable-transports/obfsproxy.git/tree/doc/obfs3/obfs3-protocol-spec.txt)
is another look-like-nothing transport, the successor to [[#obfs2|obfs2]].
Like obfs2, it is meant to look like a uniformly random byte stream.
It removes obfs2's vulnerability to passive identification
by adding a [Diffie–Hellman key exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange).
While obfs2 can be identified completely passively,
it takes an active man-in-the-middle attack or active probing to identify obfs3 with certainty.

There is a catch with the Diffie–Hellman key exchange:
it has to look uniformly random like the rest of the protocol.
Naively, you would work in a subgroup of ℤ,,p,,^*^ for some
[safe prime number](https://en.wikipedia.org/wiki/Safe_prime) _p_.
But only about half the integers between 0 and _p_ are in ℤ,,p,,^*^.
A censor could watch all your key exchanges, and see that
the numbers you send are not really random, but are always
in the chosen subgroup.
After a while the censor would decide that you are
exchanging keys and not really sending random bytes.

obfs3 solves this problem with an algorithm called UniformDH.
It's a small tweak to ordinary Diffie–Hellman.
It is described in
[this mailing list post](https://lists.torproject.org/pipermail/tor-dev/2012-December/004245.html)
from December 2012:
  Let _p_ = 3 mod 4 be prime, with _q_=(_p_−1)/2 also prime, and _p_ is at least
  1536 bits.  (2048 if there's room.)  [Use [group 5](https://tools.ietf.org/html/rfc3526#section-2) or [group 14](https://tools.ietf.org/html/rfc3526#section-3)
  from [RFC 3526](https://tools.ietf.org/html/rfc3526).]  Let _g_ be a generator of the order-_q_ subgroup of ℤ,,_p_,,^*^ (_g_=2 for
  the two above groups from the RFC.)

  To pick a private key, pick a random 1536-bit (or 2048-bit) number, and
  force the low bit to 0.  So there are 1535 (2047) bits of randomness,
  which is the size of _q_.  Let _x_ be that private key.  Let _X_ = _g_^_x_^ mod _p_.

  Here's the trick:  When you send the public key, randomly decide to send
  either _X_ or _p_−X.  That will make the public key part a uniform 1536-bit
  (2048-bit) string (well, negligibly different from uniform).

  The other side constructs _y_ and _Y_=_g_^_y_^ mod _p_ in the same way, and sends
  either _Y_ or _p_−_Y_.

  Note that both (_p_−_Y_)^_x_^ = _Y_^_x_^ mod _p_ since _x_ is even, and similarly
  (_p_−_X_)^_y_^ = _X_^_y_^ mod _p_, so key derivation goes through unchanged.
With UniformDH, the censor can't tell the difference between
a public key and a random 1536-bit (192-byte) byte string.

In obfs3, as in obfs2, both client and server do essentially the same operations,
differing only in some static constants.
Both sides begin by sending their UniformDH [[span(public key,style=background:dodgerblue)]],
followed by [[span(random padding,style=background:gray)]].
The padding is sent by each side in two pieces.
Unlike in obfs2, the padding length is not sent as an integer;
rather, the end of the padding is marked by a [[span(MAC on the secret,style=background:gold)]]
that both sides share after the key exchange.

```
<blockquote>
<pre style="background:lavender">
00000000  <span style="background:dodgerblue">89 5f 4c 0e 73 64 c5 dc  38 04 47 cf ec b2 a9 c4</span> <span style="background:dodgerblue">._L.sd.. 8.G.....</span>
00000010  <span style="background:dodgerblue">7e a6 98 21 93 96 12 31  99 59 ff ef 7e f9 ac ef</span> <span style="background:dodgerblue">~..!...1 .Y..~...</span>
00000020  <span style="background:dodgerblue">e1 0c 0f 7e bc 32 f1 da  a7 01 a0 fe 90 af 01 ce</span> <span style="background:dodgerblue">...~.2.. ........</span>
00000030  <span style="background:dodgerblue">34 d0 fe 36 af 64 70 f1  ad 0e 17 19 f4 24 17 1f</span> <span style="background:dodgerblue">4..6.dp. .....$..</span>
00000040  <span style="background:dodgerblue">94 ef 05 52 40 5f 98 a4  d0 97 6a 9f 61 52 d2 7e</span> <span style="background:dodgerblue">...R@_.. ..j.aR.~</span>
00000050  <span style="background:dodgerblue">2e b7 5b 83 16 bd 7b d8  12 b3 bc 32 5f 3e e3 21</span> <span style="background:dodgerblue">..[...{. ...2_&gt;.!</span>
00000060  <span style="background:dodgerblue">ca 14 b4 6e b5 8e a5 61  d1 36 ce f0 d9 62 a7 4d</span> <span style="background:dodgerblue">...n...a .6...b.M</span>
00000070  <span style="background:dodgerblue">49 5d be ab 32 30 54 cb  63 52 98 59 3c 8e f7 20</span> <span style="background:dodgerblue">I]..20T. cR.Y&lt;.. </span>
00000080  <span style="background:dodgerblue">37 39 a7 1d 48 57 39 8b  6d 80 58 e0 34 54 fd 1a</span> <span style="background:dodgerblue">79..HW9. m.X.4T..</span>
00000090  <span style="background:dodgerblue">7f b4 83 a6 44 78 aa cc  50 df e4 eb 5c cb 61 3e</span> <span style="background:dodgerblue">....Dx.. P...\.a&gt;</span>
000000A0  <span style="background:dodgerblue">8d d5 08 b2 d6 dd da 47  2d b6 93 e8 ed cd b2 3f</span> <span style="background:dodgerblue">.......G -......?</span>
000000B0  <span style="background:dodgerblue">74 0c 4c 7d f4 8a 52 95  f1 84 59 0b 1f f9 c2 b2</span> <span style="background:dodgerblue">t.L}..R. ..Y.....</span>
000000C0  <span style="background:gray">3a e0 80 e8 5e 95 91 80  eb 48 de 13 dc d1 8f f4</span> <span style="background:gray">:...^... .H......</span>
000000D0  <span style="background:gray">ef a8 61 06 1e 80 1d 59  07 5f 9a b9 a6 4b 7a 57</span> <span style="background:gray">..a....Y ._...KzW</span>
000000E0  <span style="background:gray">62 63 0d 69 15 10 71 c8  45 ea bb 58 08 0e f5 90</span> <span style="background:gray">bc.i..q. E..X....</span>
000000F0  <span style="background:gray">19 5f 7b da b9 11 1f 3b  fb 92 89 5a 4a df 8e ac</span> <span style="background:gray">._{....; ...ZJ...</span>
00000100  <span style="background:gray">40 8d f9 80 a8 eb 37 b6  cb 8d bc ea cd c0 54 c0</span> <span style="background:gray">@.....7. ......T.</span>
00000110  <span style="background:gray">91 d6 5c cb a9 9a 4d a1  50 38 9d c6 aa 0f 42 34</span> <span style="background:gray">..\...M. P8....B4</span>
00000120  <span style="background:gray">6f 21 e2 8f</span>                                      <span style="background:gray">o!..</span>
</pre>
</blockquote>
<pre style="background:cornsilk">
00000000  <span style="background:dodgerblue">62 13 c9 fd 02 88 0d 56  73 8b 5f bf 5b 1b 95 f0</span> <span style="background:dodgerblue">b......V s._.[...</span>
00000010  <span style="background:dodgerblue">78 e6 a8 8e 54 15 76 64  2c 74 ea b8 dd fa b7 dc</span> <span style="background:dodgerblue">x...T.vd ,t......</span>
00000020  <span style="background:dodgerblue">86 f5 ef 47 4b 47 e5 38  60 31 1e be 9b 81 c5 00</span> <span style="background:dodgerblue">...GKG.8 `1......</span>
00000030  <span style="background:dodgerblue">ca 5b 7b 00 bb fc d7 b8  0e 02 45 eb 73 7d 42 48</span> <span style="background:dodgerblue">.[{..... ..E.s}BH</span>
00000040  <span style="background:dodgerblue">79 24 a3 9f 94 18 a8 7a  58 a0 23 91 b3 b2 68 4f</span> <span style="background:dodgerblue">y$.....z X.#...hO</span>
00000050  <span style="background:dodgerblue">52 8e d5 5d 69 92 7b b5  a5 71 94 97 06 55 c9 54</span> <span style="background:dodgerblue">R..]i.{. .q...U.T</span>
00000060  <span style="background:dodgerblue">90 d0 74 79 67 64 f4 4f  d6 de 1c 09 c5 66 cb 0a</span> <span style="background:dodgerblue">..tygd.O .....f..</span>
00000070  <span style="background:dodgerblue">67 dc b5 84 f0 d8 20 ed  fd 03 08 9f fa e2 18 34</span> <span style="background:dodgerblue">g..... . .......4</span>
00000080  <span style="background:dodgerblue">2d 8a de b1 9e e9 c4 1a  77 84 b4 4d 01 07 4d 07</span> <span style="background:dodgerblue">-....... w..M..M.</span>
00000090  <span style="background:dodgerblue">7a 49 09 ac 25 ce fb e9  2a 3a 8e 0b ac 0f 03 81</span> <span style="background:dodgerblue">zI..%... *:......</span>
000000A0  <span style="background:dodgerblue">31 db 48 bc ae 8e 5b 3c  4a 76 e3 b8 7e af 6e 31</span> <span style="background:dodgerblue">1.H...[&lt; Jv..~.n1</span>
000000B0  <span style="background:dodgerblue">cf 81 f6 cb 40 cb a1 32  ee fd ec 19 ae 90 8b 20</span> <span style="background:dodgerblue">....@..2 ....... </span>
000000C0  <span style="background:gray">ad 89 49 ef 48 3f 2d 0c  59 94 8c 37 d8 54 e0 5d</span> <span style="background:gray">..I.H?-. Y..7.T.]</span>
000000D0  <span style="background:gray">a2 f5 e8 13 ec a2 0a 7c  50 88 4a fc 40 be eb 1d</span> <span style="background:gray">.......| P.J.@...</span>
000000E0  <span style="background:gray">1b 8c 00 fd 24 55 aa 84  07 06 e7 d4 b0 58 04 16</span> <span style="background:gray">....$U.. .....X..</span>
</pre>
```

The first 192 bytes (1536 bits) from each side is its
[[span(public keys,style=background:dodgerblue)]].
After that is a random amount of [[span(padding,style=background:gray)]]
(between 0 and 4097 bytes).
Here, the server has sent 100 bytes of padding
and the client has sent 48.

Looking behind the scenes in this example, the client's private key (which doesn't appear on the wire) is
```
}}}
and the server's private key (which also does not appear on the wire) is
{{{
```
You can verify that these give the correct [[span(public keys,style=background:dodgerblue)]]
modulo the prime _p_ from [RFC 3526](https://tools.ietf.org/html/rfc3526#section-2).
Both sides might have instead sent _p_−_X_ or _p_−_Y_,
but in this case by random chance they did not.
  _X_ = 2^_x_^ mod _p_ = 0x6213c9fd...\\
  _Y_ = 2^_y_^ mod _p_ = 0x895f4c0e...

```
>>> x = 0xf525f255f6c11f0b10a1d0775638f19c7f85c5bd1628d4f903d4c0a11e585bc983f82da3515dfbe49f3828d5be0e68d1fd525c611560c788322cb911ccf77a74fdc74fa1f262dfad28ce454b93785e96fd8d4916896274e4896da990e089d14443a9242a974bbb5a2cefefcdb42c3984381e0ec71e2140826608cdd9ef52fe5be903ec5f17d91d580fd234345d7233fea29b81738dd67f712cfd56f54413d43f2889ed1cbd86781313580c13b99499aced73d36778a68b6a80137d084c7a5798
>>> y = 0x421e8e76020c725890488d5f45e3b035162b4ac51eb5d124fe565ff06cef6bf1b61f4a9952effa251a259587e227bdc8ffb22a92ea2e512290f5ce56063d7a39d549d2053f4496d0d5ecce4c920ef8683ef2023b1257d32607657db005e6490fb0461bfd834fb7fb8ef464ebfce014537e0c500386f2aeec7cd41a5fc5acbb9db64a49ef6a1df66a7064419eb0c7d879819f5834d679908e78b9e31fe2f444c4eefe14741c8e7c21fe811c9c0e51f094c25ef67e0cd329df611f6cb02fb56ba4
>>> p = 0xffffffffffffffffc90fdaa22168c234c4c6628b80dc1cd129024e088a67cc74020bbea63b139b22514a08798e3404ddef9519b3cd3a431b302b0a6df25f14374fe1356d6d51c245e485b576625e7ec6f44c42e9a637ed6b0bff5cb6f406b7edee386bfb5a899fa5ae9f24117c4b1fe649286651ece45b3dc2007cb8a163bf0598da48361c55d39a69163fa8fd24cf5f83655d23dca3ad961c62f356208552bb9ed529077096966d670c354e4abc9804f1746c08ca237327ffffffffffffffff
>>> X = pow(2, x, p)
>>> Y = pow(2, y, p)
>>> "%0384x" % X
'6213c9fd02880d56738b5fbf5b1b95f078e6a88e541576642c74eab8ddfab7dc86f5ef474b47e53860311ebe9b81c500ca5b7b00bbfcd7b80e0245eb737d42487924a39f9418a87a58a02391b3b2684f528ed55d69927bb5a57194970655c95490d074796764f44fd6de1c09c566cb0a67dcb584f0d820edfd03089ffae218342d8adeb19ee9c41a7784b44d01074d077a4909ac25cefbe92a3a8e0bac0f038131db48bcae8e5b3c4a76e3b87eaf6e31cf81f6cb40cba132eefdec19ae908b20'
>>> "%0384x" % Y
'895f4c0e7364c5dc380447cfecb2a9c47ea69821939612319959ffef7ef9acefe10c0f7ebc32f1daa701a0fe90af01ce34d0fe36af6470f1ad0e1719f424171f94ef0552405f98a4d0976a9f6152d27e2eb75b8316bd7bd812b3bc325f3ee321ca14b46eb58ea561d136cef0d962a74d495dbeab323054cb635298593c8ef7203739a71d4857398b6d8058e03454fd1a7fb483a64478aacc50dfe4eb5ccb613e8dd508b2d6ddda472db693e8edcdb23f740c4c7df48a5295f184590b1ff9c2b2'
```

Now both sides have the information they need to compute their shared secret
  _X_^_y_^ = _Y_^_x_^ = 2^_xy_^ = 0x8cb480f32c2f26e79b40080a3c5c34512c6e83fa1957088dcdffbdb6adcbf24b237faed92f0d0b0310bc6b1b5c99038f3a75400a0b8fb3e0cd6f4f11696ee7be2c7dafb0d8f390fbf3f186e847df4fab00ad784c9ac9806aa82cf06433e66a99293862768db38506bafd976b67ea4b08780689d5217f501748658081e92328c13284b8156eaf00cba0b343b096da71849507a97a4b41236125b443ff7e1e78f2b5489e3ac787d3a8a84dee442408525aba81c7b99721c94edcaa6513948a6607
This shared secret is used to derive marks that
indicate the end of padding,
as well as session keys that encrypt all the following payload data.

The end-of-padding marks are HMACs that use the shared secret:
```
>>> secret = ("%0384x" % pow(X, y, p)).decode("hex")
>>> import hmac
>>> from hashlib import sha256
>>> hmac.new(secret, "Initiator magic", sha256).hexdigest()
'7cbbee8bd9b26bf9099d75ec10b50b340d637316040acedd56c1b6d0715a0ba1'
>>> hmac.new(secret, "Responder magic", sha256).hexdigest()
'9c616cd7963c68e4a9322a3b390bf4a51ab0873a333943aff740da2dcab2d76c'
```
Now that the client has something to send, it sends another block of
[[span(random padding,style=background:gray)]] of between 0 and 4097 bytes, followed by its end-of-padding mark
[[span(7cbbee8bd9b26bf9099d75ec10b50b340d637316040acedd56c1b6d0715a0ba1,style=background:gold)]],
then followed by its encrypted payload data,
encrypted using a key derived from the shared secret.

```
<pre style="background:cornsilk">
000000F0  <span style="background:gray">84 57 5f 0d 73 a3 36 45  2e a2 8e 58 e7 a4 71 f4</span> <span style="background:gray">.W_.s.6E ...X..q.</span>
00000100  <span style="background:gray">ab 75 00 f1 91 7f 89 e5  7e 13 6e 94 91 16 90 78</span> <span style="background:gray">.u...... ~.n....x</span>
00000110  <span style="background:gray">0d 5e d7 14 13 ea bd cf  ed 6e f8 7a 1e b8 76 17</span> <span style="background:gray">.^...... .n.z..v.</span>
00000120  <span style="background:gray">75 0c 24 b9 26 cc 19 87  af f0 19 59 b8 ac c3 13</span> <span style="background:gray">u.$.&amp;... ...Y....</span>
00000130  <span style="background:gray">58 47 b3 59 1d d7 aa 1e</span>  <span style="background:gold">7c bb ee 8b d9 b2 6b f9</span> <span style="background:gray">XG.Y....</span> <span style="background:gold">|.....k.</span>
00000140  <span style="background:gold">09 9d 75 ec 10 b5 0b 34  0d 63 73 16 04 0a ce dd</span> <span style="background:gold">..u....4 .cs.....</span>
00000150  <span style="background:gold">56 c1 b6 d0 71 5a 0b a1</span>                          <span style="background:gold">V...qZ..</span> 
</pre>
<pre style="background:cornsilk">
00000158  1e a5 04 9b e3 f4 9f 67  4e 73 79 14 d1 a5 66 06 .......g Nsy...f.
00000168  17 a5 7a d0 2b 7d 06 ab  a7 20 7e b6 f3 03 5b 7a ..z.+}.. . ~...[z
00000178  30 23 92 fd 67 08 81 76  58 94 d5 a3 47 45 46 4d 0#..g..v X...GEFM
00000188  0f 83 aa 0a d4 c6 2b 33  bf 24 b1 d6 f6 df 5a a5 ......+3 .$....Z.
00000198  8a ce a6 6b e5 03 9b b5  b3 2c 85 85 bd cd f5 64 ...k.... .,.....d
000001A8  df a7 98 7d aa 29 50 18  d8 28 08 94 ef f8 e1 06 ...}.)P. .(......
000001B8  9c e6 8d da 61 7b bc 9d  f2 b8 ba 14 03 56 16 6b ....a{.. .....V.k
000001C8  1e 12 5e 02 5c 1d f2 cd  89 c9 ac a5 44 87 8a fc ..^.\... ....D...
000001D8  b7 a5 72 ee 8f 33 0f 03  83 00 b8 e5 ea ff e2 0f ..r..3.. ........
000001E8  f8 5a 54 76 e9 76 ab 9c  a7 ba d9 af ab 8d ee af .ZTv.v.. ........
000001F8  6b 66 67 26 cf bb 41 46  66 ff 29 a5 49 4e 0d 74 kfg&amp;..AF f.).IN.t
00000208  ba a1 e9 70 2b dc 1e c1  44 0f 65 2b 6c 96 fd 76 ...p+... D.e+l..v
00000218  75 72 f3 cd ab ee c8 1e  ff 5d 5c 30 f4 23 14 f1 ur...... .]\0.#..
00000228  2b c3 c8 ae 3c d6 91 47  27 86 48 65 02 2d e1 94 +...&lt;..G '.He.-..
00000238  14 11 60 90 f3 85 52 42  4b be 2f 0b 15 56 c4 26 ..`...RB K./..V.&amp;
00000248  63 36 cf b7                                      c6..
</pre>
```

In its response, the server also sends some more
[[span(random padding,style=background:gray)]] and its own end-of-padding marker
[[span(9c616cd7963c68e4a9322a3b390bf4a51ab0873a333943aff740da2dcab2d76c,style=background:gold)]],
followed by encrypted payload data.

```
<blockquote>
<pre style="background:lavender">
00000124  <span style="background:gray">a3 ef 3e bf 1b fd 66 f6  91 e0 11 4b cc 99 4f c4</span> <span style="background:gray">..&gt;...f. ...K..O.</span>
00000134  <span style="background:gray">f4 93 fc c7 c5 85 07 f5  16 88 06 d2 dc 0e 66 85</span> <span style="background:gray">........ ......f.</span>
00000144  <span style="background:gray">6e 21 99 bd</span> <span style="background:gold">9c 61 6c d7  96 3c 68 e4 a9 32 2a 3b</span> <span style="background:gray">n!..</span><span style="background:gold">.al. .&lt;h..2*;</span>
00000154  <span style="background:gold">39 0b f4 a5 1a b0 87 3a  33 39 43 af f7 40 da 2d</span> <span style="background:gold">9......: 39C..@.-</span>
00000164  <span style="background:gold">ca b2 d7 6c</span>                                      <span style="background:gold">...l</span>
</pre>
</blockquote>
<blockquote>
<pre style="background:lavender">
00000168  28 64 1e 9b 20 a6 46 13  19 58 d9 a4 ef c2 02 db (d.. .F. .X......
00000178  d3 1b c0 87 4d c6 2d 68  34 df 1a 11 eb 22 fe 7b ....M.-h 4....".{
00000188  12 5e ab ba 6f a9 52 99  da 21 8d 2f 2b 8e 35 fd .^..o.R. .!./+.5.
00000198  fe 34 ef 3e 23 f0 0a 7f  55 08 87 1e 02 cb 19 66 .4.&gt;#... U......f
000001A8  c6 39 60 4e 9d a9 bf fd  a2 27 59 c2 da ad ba 2b .9`N.... .'Y....+
000001B8  22 69 39 f5 0f 02 02 0b  c2 9f 5d 70 5f 97 16 5a "i9..... ..]p_..Z
</pre>
</blockquote>
```

Neither obfs2 nor obfs3 do anything to disguise packet sizes and timing.
In this comparison graphic, you can see that apart from the differing handshakes
at the top, the size of packets is mostly unchanged.
(In our examples, we have used short padding, but in practice,
the padding of obfs2 and obfs3 can be up to 8 kilobytes.)
The darkness of each pixel indicates byte values from 0 to 255.
All the bytes of obfs2 and obfs3 appear uniformly random,
while if you look closely at the ordinary Tor example,
you can see a dark smear at the beginning of each packet—that's
the plaintext [TLS record header](https://en.wikipedia.org/wiki/Transport_Layer_Security#TLS_record),
one of the features that pluggable transports aim to obscure.
```
ordinary Tor
```
```
obfs2
```
```
obfs3
```
|----------------
```
![plain-pixels.png](plain-pixels.png)
```
```
![obfs2-pixels.png](obfs2-pixels.png)
```
```
![obfs3-pixels.png](obfs3-pixels.png)
```

# ScrambleSuit

```
tor 0.2.5.4-alpha-dev (git-081ff5fa83cf146a)
obfsproxy 0.2.9
```

[ScrambleSuit](http://www.cs.kau.se/philwint/scramblesuit/) is another transport that looks like uniform random bytes. It has a couple of twists over obfs3: It randomizes the size and timing of packets, and the server resists [active probing](http://www.cs.kau.se/philwint/static/gfc/) by requiring a secret key from the client before it will respond. The hex dump looks like random noise as you would expect.
```
<pre style="background:cornsilk">
00000000  00 90 a0 58 e3 05 c2 d6  f0 6e 86 5c 5d dd 57 1c ...X.... .n.\].W.
00000010  cf a0 ec ed 15 ed 00 e5  ed d6 d9 29 d9 31 ba 64 ........ ...).1.d
00000020  15 8e bd 84 93 9a e4 f8  c9 d4 ed 25 73 74 e5 69 ........ ...%st.i
00000030  2e 20 02 9a 8d 7c aa 56  04 d7 a7 74 47 31 f6 a3 . ...|.V ...tG1..
00000040  5b ae b2 36 78 57 05 2f  c5 9d 2b 20 7f 5f 8b 23 [..6xW./ ..+ ._.#
00000050  82 5a 47 6a 59 8e 2f 12  14 6a 85 ce 89 16 2a 42 .ZGjY./. .j....*B
00000060  b1 ea d8 3e 2d 92 e1 f3  f9 4d 8b 02 32 a2 c7 e0 ...&gt;-... .M..2...
00000070  85 83 8a 29 6f 47 60 de  25 fb 0c c0 a8 37 55 38 ...)oG`. %....7U8
00000080  69 f5 95 24 5b 66 2c c5  3d 70 66 a5 a4 1b ea dc i..$[f,. =pf.....
00000090  44 f9 28 c2 d4 77 55 ff  49 2b 7d d5 ac 77 09 5b D.(..wU. I+}..w.[
000000A0  07 0c cb 80 8c 6b ca 20  e6 d5 f2 58 f3 69 c9 ac .....k.  ...X.i..
000000B0  da 3f 4c 46 ba 9c e7 c8  34 8d 8e 45 eb c0 35 d7 .?LF.... 4..E..5.
000000C0  98 e7 5f 92 d1 dc 9b f4  92 d4 04 cc 42 04 2c 0d .._..... ....B.,.
000000D0  e0 9a                                            ..
</pre>
<blockquote>
<pre style="background:lavender">
00000000  98 3a 13 c7 52 a8 01 18  97 c6 d7 88 8b 11 f6 22 .:..R... ......."
00000010  14 6f 3c 60 83 79 c0 3e  04 da 5a d2 e5 c2 8f 40 .o&lt;`.y.&gt; ..Z....@
00000020  71 88 78 ce 6a 3e e6 b3  c4 e3 47 8a 10 ed 7f e1 q.x.j&gt;.. ..G.....
00000030  27 ac 47 66 30 82 c8 67  d5 4e 73 dc 34 17 ea 1e '.Gf0..g .Ns.4...
00000040  4e b1 50 c7 e0 a3 49 0b  64 fb eb 81 a5 01 07 81 N.P...I. d.......
00000050  25 73 62 76 3c 88 cb d0  93 fb 4d 3b af 77 f7 9e %sbv&lt;... ..M;.w..
00000060  32 dd d1 fe 89 6e 3e ce  63 34 a7 cc d8 56 07 74 2....n&gt;. c4...V.t
00000070  ad b6 74 aa 43 87 4e 0c  ba 1f aa e6 15 30 aa 1d ..t.C.N. .....0..
00000080  a2 c2 89 75 8c be 29 52  61 e1 e9 50 4e 0c b0 04 ...u..)R a..PN...
00000090  a8 9e 5b 90 bb a3 1c 4b  81 b8 90 76 26 49 ba 66 ..[....K ...v&amp;I.f
000000A0  10 dd 3d 9a 02 83 70 b9  06 6c 2e e2 e6 c1 eb 40 ..=...p. .l.....@
000000B0  3e 15 bf 6d a3 7a fd 20  02 fa 60 fc 52 1f 31 20 &gt;..m.z.  ..`.R.1 
000000C0  6a cb 57 3d 12 e2 f2 99  0a ff 62 f5 2a 56 8a 13 j.W=.... ..b.*V..
000000D0  75 1e 72 8f a4 f7 0c d5  6f 38 33 3e c2 58 4c d6 u.r..... o83&gt;.XL.
</pre>
</blockquote>
```

Here are visualizations of downloading the web page at https://check.torproject.org/ with ordinary Tor, obfs3, and ScrambleSuit. obfs3 uses the same packet sizes as ordinary Tor. ScrambleSuit randomly chooses a few packet sizes, and pads packets to those sizes, so more bytes are sent overall. Differences in packet timing are not shown.
```
ordinary Tor
```
```
obfs3
```
```
ScrambleSuit
```
|----------------
```
![plain-webpage.png](plain-webpage.png)
```
```
![obfs3-webpage.png](obfs3-webpage.png)
```
```
![scramblesuit-webpage.png](scramblesuit-webpage.png)
```

The following two diagrams show ScrambleSuit's inter-arrival time modification.  When downloading data, Tor (the blue lines) looks similar across different connections.  ScrambleSuit servers (the orange lines) deviate from that.  Note that inter-arrival time obfuscation is quite expensive as it artificially drains network throughput.  As a result, it is disabled by default in obfsproxy.
```
![scramblesuit-inter-arrival-times.png](scramblesuit-inter-arrival-times.png)
```

# FTE

[FTE](https://fteproxy.org/), for "format-transforming encryption," encodes data so that it matches an arbitrary [regular expression](https://en.wikipedia.org/wiki/Regular_expression). The idea is that censorship hardware uses regular expressions to classify traffic into allowed and disallowed categories. FTE makes your traffic look like something in the allowed category, as far as the regular expression can tell. Here is a sample of FTE traffic using its default [manual-http-request](https://github.com/kpdyer/fteproxy/blob/cdd30bbd441a45828965e0d62c6548a04d8bd823/fteproxy/defs/20131224.json#L2) and [manual-http-response](https://github.com/kpdyer/fteproxy/blob/cdd30bbd441a45828965e0d62c6548a04d8bd823/fteproxy/defs/20131224.json#L5) formats. HTTP bodies have been truncated to save space.
```
<pre style="background:cornsilk">
GET //oa9xnE79SSJT73XIDv5gDx6m9kCx.6SJzCweNTMMPPFjL/rgCK1XqYv6hSQJkzpMkpu1cTBiauAaz4Fl49NK78o2nUD/VcGRS2MM7Bfl6X4v./xGw5orrtPQfIXUbWCW.CkTS3j8sD5wQfbsURlceheKV5/bVHs3axmSbKbzvyg0dMh/xQiK2mMAR0aifZ93F0l9ql9qRSDa/8b6oZITWMZFKHwIJEFSJnrpUFj/0c9dX HTTP/1.1\r\n
\r\n
\xe7\xd1\xc1!\xf0\x1eX\x9ez\r\x06\xb4\x14\xa7/\xa1\x0b\xb7\x7f\xc0\xd2y\xe1
\xa7\x8b\x97VZ\x10\xab\xe84w\xa1\x9e\r\xf6\xf3\xf8@\xe0\x00\xab2\x07\xb8@
\x08\xeb3\xd9Li\x12\x1cU\x1dj\xf3\x97tT\x17\xf2\x90Z\xf4 \xd4\xf4\x01\xa7
...
</pre>
<pre style="background:cornsilk">
GET //X/oy8D3EU2ypudP4j8FFghcMAKV0dHCff7uEb6mP/cVII0SmyrNRtcKpFjh1rC/jNWfFAJnyTUmaxL.Q5V5YzhQZYg2qvd1VPouHsjD1K8qtupacqQiVnHD8g4Xr4vJxYgdDYMJGvkBFeAPUQLCoKXEJFt0JqSqQMwLsD6NDX3eYaXk0kVM6Vo3xCv7Dky.zf7Cer/BzVb9es35gPQcdiDn4B53uAD6nhFEWOLpYDfalP HTTP/1.1\r\n
\r\n
\x19D\x90\xe4\x92\x11\xd5\xb20h{\x9e\x19\x99\xeb\xb0?\xad\xb0o\xf0+\xfc\xf9
<%P\xfa\xb7\x8am\xb1hX\x13\xc6!f>\xc3e\x10QTU7\x1a\xd2b!\x12\xdf\xc0v(H\xce
\xacze\x7f\xc9cNqd\xf1\x84\xdb\x0c\x92}H\x99\xd8\x01\xd7\xc7\x1fZ\x1e\xa5
...
</pre>
<blockquote>
<pre style="background:lavender">
HTTP/1.1 200 OK\r\n
Content-Type: H\r\n
\r\n
|\x96\xbd?\x16%\xd7\x8d7Kf\xfe\x0c\x86~\xfe\xc1\xc7\xf7\xb4Tj%\x9a\xd4A\t|P
\x1d\x11I\xd5\xf3\x8e\xd3\xf748\xeev\x8c\xbd\xa8\xdd\xb1\xc2A\xc9\x8d|\x06M
}\xe5\xba5\x1e\x97!\x89\xe4\xb7\t\xe3\x02\x1f{]Ku\x8b\x9c\x8d\xf4\xd2\x10A%
...
</pre>
</blockquote>
<blockquote>
<pre style="background:lavender">
HTTP/1.1 200 OK\r\n
Content-Type: H\r\n
\r\n
\xd9\xcf\x80\x93\xeaJ-A\xf7i\xe2C\x95\xf2\xf4\x9b^\x0c\x81\xdd\x85\x1e\n
\xf9\xa74|\xf2\x1eD~\xfcoU\xaa\xeb[j^z\xeb\x02}_b\xfb\x96\xf8\xe9\x8c#\xd6
\x83\xa8\xbe\xc2\xff\x8d\x94\xce#\xc7\xca\x1ea\xfc\n~\x15\xfb\x8f\xb0\xf7\t
...
</pre>
</blockquote>
<blockquote>
<pre style="background:lavender">
HTTP/1.1 200 OK\r\n
Content-Type: H\r\n
\r\n
<$%\x06\x9b\x03\xcd\xa64\xfa_\xfaP\x98\xf2 \\\xea\x9b\x10qJ\x97\x97\x04\xe5
7\x9b\xd3 \xa5\x12\x93W\xed\xa3\x18\xa5\x9a6O\x97\t\xf0$\x807\x9eO!\x86\xf8
o\x0b\x7f\x8b\xc0Z\x89\xaa,\xcc[w\x9bk\xca\x19\xb9N\x1f\xb3\xea\xa1\xc1c~D
...
</pre>
</blockquote>
<blockquote>
<pre style="background:lavender">
HTTP/1.1 200 OK\r\n
Content-Type: H\r\n
\r\n
2\xfb\xb9\xab\xe7\x1b_\xfe%\xb4\xf1\xd4kPE\xec\xb7\x89\t\x0e\x8f\x9b\xef
\xdfB\x8fE\xddn\xd2\x90\xad\xe3\x1bHN\xbe\xce\xe4\xe33+o9\xec\xb2Q+\xe7\xfc
\x19I.RA\x83\x86\x9c\xa3j\x19\x9c\xe3\x92s\'`\xdf>\x10\xc7\xb8\xeb\xce\xd7&
...
</pre>
</blockquote>
<pre style="background:cornsilk">
GET /GXlAAlA5/Bp9.hUXw.uwKt.qhyePcEmELGfJaPyEou0ttU9dJ3KdmU/7IhrB8L9iDqLta0SmbW4USo8qANRHHnY5ZMleDUtGr1hgHwf/12SmiM6AZOPFb9WmXnvQoLTxG1zPKozYNSWOSdQdGjXcFeReZP3uRKYPyntJ/ON/wGZjz0sVpBx.2D5Gy7oTLABQKy52p4QSZWfd6i5WkUj.cGxtjdS4sw.H/JfvME6IyXJeZv HTTP/1.1\r\n
\r\n
\xc6\xe7\x86\xd2B\x90x\xf0\x1a Y\xccA\x14Dj\xe19\xfa\xe0\xe0\x9aK\'\xe1 
\xbbb\x1d|\xa8o^\xd0\x9a\x9f\xdb\xf5V\x89\xbdIB\xb1T[\xa3C@\x7f\x9f\xac\xfd
\xce\x9feC\x02\xd4\x1eW\xe1X\xa9\xb6\xf6\xe3\xdaq\x10\x9fY1\x08\xf9\xf0\xb1
...
</pre>
<blockquote>
<pre style="background:lavender">
HTTP/1.1 200 OK\r\n
Content-Type: H\r\n
\r\n
\xcav\xc3@\xc5\x7f\tr\x89_\x88\xc8\xc6\x87\xf3\xe0\xd9\x0b\x88\xd0$\x8c\xd3
\xc3\x1d\xfeC\xdd\xb2\x18\x9ci^Z\xe3<[\xe4\x1fk=$\xd4+\xd7c\xc1\xeb:\xe3zFJ
C4\rPFt\x83v\xe5\x12\xaf\xc4\x0b\x1f\x96\xb9\xa2\xa2\xa1\x84\xee\xbb\xa0
...
</pre>
</blockquote>
```

FTE can do other formats as well. Here is a sample of [manual-ssh-request](https://github.com/kpdyer/fteproxy/blob/cdd30bbd441a45828965e0d62c6548a04d8bd823/fteproxy/defs/20131224.json#L16) and [manual-ssh-response](https://github.com/kpdyer/fteproxy/blob/cdd30bbd441a45828965e0d62c6548a04d8bd823/fteproxy/defs/20131224.json#L19). This example doesn't use Tor; it was run with
```
fteproxy --mode client --upstream-format manual-ssh-request --downstream-format manual-ssh-response
fteproxy --mode server --upstream-format manual-ssh-request --downstream-format manual-ssh-response
ncat -k -l -p 8081
ncat 127.0.0.1 8079
```
```
<pre style="background:cornsilk">
SSH-2.0SKT\xbb\x1d\x18\xd5\xab\x11\xc0\n\xb1UCD\x15\xbf2"r\xf0\xa6\xaf\x98
\xa1\xf57>!r\x81\xd8\xd9\x17E$\xdflV\xb0x\xe1\x97bG\x17\x06}\x9b\xf3\x13
\x0e0p\x15 \xa2\x11\x85\x19\x9dU\xd5\xed\xb3\xe9\x06\xff\xf2\x87\xb9\x80
\xef\xf7\xf0\xf1%\xbd\xe3n\x05\xa9V\xbe\xa0\xd1(\xc2r\x98Sr\x88\xf4\x8f[b
\x1a\x8c\xae\x9d9\x81\xac(\xe0\xb5\xa1\xa9I\x15\xa7\xcd\x8fe~\xe8\xd3\xce
\xd2q\xbe\xc0\xba\x1eB\x17K\xdaN\xe4\xe5\x13\xd3\x83\xf5O\x83\xab\xbd9^\xc2
\xc2\x85\xd8\xb1\x10\xb7\xb5\x9co\xcf:\xa6c*>\xa8[)S\xac\xfb\xf2\xf6q\xeb
\xefq\x88\xb5\x1f`i\xeb\x01f5<\x01\xe8Bp\xc8i:\xfd\xd0;\xe1\xaf\xa9\xd1\x15
\x9f\xcc\x10\xee\x0f{\xb7j_\x0cUN\xb9\xfa\xcdp\x13\xc5\xcf\x9e\xdc/\xfat
\xdb@\x88\xcb\xf2\xcb\x04\x83;H\xc4\x9dR\x08Y/\xe8\xe0\x95Y\x10\xbf\xd2\xea
\tM\x94S\t\xd4
</pre>
<pre style="background:cornsilk">
SSH-2.0SKl\xe0\xaa?\x13c5\n\xb3\xe2\x85\x90;\xb6p\x19PW\x03\xb5-\xf9\xce
\xccO_\xcde\x90\xdd\x94\x1fc\xf1w\x16^\xcac!\xd0\xeb=\xb2a\x8c\xa4\x94\x18
\xda2\'\xf1\x88X\x12\x83*,,\x07.\xb5B\xb7\xde\xe8]\xe9\xae\xe2r\xfa\x0eb[
\x1d\x03Ao\xc81\xbf\xa10\x07T\x9c\x87\xb2M\xed\x1c\x84`\xfao\xd5\xe5\xf6
\x91S\x18\xe3Z\x90O\x7f\x17]r\xa2\xe1l\xca\x0c\xcf\xc2\xba\xb1\xf2\\\xd3
\x195\xf3\x0e\x99.q\xee1\xb6\xd8\xbb\xc6+\xa190\x91|\x0f\xfc\xf4\x91\xe72
\xf73\x0f.~o\xfd\x9f\xa3Ga\xbe\x02\xc1\x95j\x8e]\xd0R]:\xec\xae\xd9P_R[\x83
|\x01\\\x95>\\\x19\x82uo.%O\x83\x81^\x7f\x11\xbe\xac\x08\x9d~\xdbF\x11\x05`
k\xaf\x0c/\xd9\xf6\xfe\x10<\xb3\x88z\x85~$j\xe1y\x87\\\xf0-\x1f\x8e\x84\xde
\x17\x85v\xfb/\x17\xdd\xeb\xc1\x9e\x14O\xb1\x9b\xb9
</pre>
<blockquote>
<pre style="background:lavender">
SSH-2.0S.\xc5\x0eI7!{7\x85@\xe5\xf2\x7f\xacAZ\xdcl\x99/9\xe1 \x90\x0b\rVv
\xd6u\xf2h\x1f\x1cn\tV\x0emN\xe3L\xaeyh\xc3\xb5\xa4\x96\xea\x95\x15\x7f@e
\x92.\xeb\xc6d5\xc7\x8a\x91+\xf0\x94\x96\xa5\xdf\x01\x0eI\x1d;\xcfF\xee\x1a
\xb6\xbc\x9e\x87E\x12\x84$C\x9er.\x01\xcb0\xa3=\x0f\xcd\x15\xae\x7fc\x15
\x9a\xed"9\xcf\x8f\xcf9\x87x-p\xfc\xb4!m\x86\xf6\xa8qv\xd6>\xeb0\'\x06\x8ch
\xb5Gj#A\xff\xee\x1e\x9br\xf16\x0b\x06>4\x1cM\x07\xd2\x190/\xe0[\x0fj\x91~
\xc6\xe9nQ<ip\xbcl\x1dS\xf4\xfaN\xcfM\x00|v4\xe9E\n\x0c\xe5&\xc8\xfbW\xd0
\xceq65gh\xd1?\x9e\x98z\xa4\xa48\xc3\x1a\xd4\x0eV\xf6\xd3x\xda|QW\xda\xdd
\x1aR\xf5\xe2\x1c\x9e{\xda1\x06\xd5\xce\xe5-\x05\xed\xeb\x04\x1e\xec~\xa5
\x94A\xcf<Hy\x1e
</pre>
</blockquote>
```

# Flash proxy

[Flash proxy](https://crypto.stanford.edu/flashproxy/) is a system of short-lived proxies, each running as a JavaScript program in a web browser. Flash proxy uses a browser technology called [WebSocket](https://en.wikipedia.org/wiki/WebSocket), which is a socket-like layer on top of HTTP.

Flash proxy doesn't do anything special to obscure the content that it carries. What gets sent is [[#OrdinaryTor|ordinary Tor]] TLS, wrapped inside a WebSocket layer: if you look at the WebSocket payloads, you will be able to see TLS. (There actually is one slight obfuscation, a side effect of the WebSocket protocol: WebSocket frames sent from the proxy to the client are [xored](https://tools.ietf.org/html/rfc6455#section-5.3) with a 4-byte random masking key. The key is set to [[span(00 00 00 00,style=background:coral)]] in the examples below so you can see how it looks without masking.) You can see the TLS in the payloads of the examples below. Headers like `Sec-WebSocket-Key` and `Upgrade: websocket` are part of the WebSocket handshake.

Notice that, because a flash proxy connects to the client, and not the other way around, the first data that gets sent (the HTTP GET request that starts a WebSocket connection) is sent _from_ the proxy _to_ the client. The `HTTP/1.0 101 Switching Protocols` response is sent by the client, which in this case is actually acting as a web _server_.

```
<blockquote>
<pre style="background:lavender">
GET / HTTP/1.1\r\n
Host: 192.0.2.101:9000\r\n
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; rv:28.0) Gecko/20100101 Firefox/28.0\r\n
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n
Accept-Language: en-gb,en;q=0.5\r\n
Accept-Encoding: gzip, deflate\r\n
Sec-WebSocket-Version: 13\r\n
Origin: http://crypto.stanford.edu\r\n
Sec-WebSocket-Key: RthUFtiTVCjpBTSN4za+Zg==\r\n
Connection: keep-alive, Upgrade\r\n
Pragma: no-cache\r\n
Cache-Control: no-cache\r\n
Upgrade: websocket\r\n
\r\n
</pre>
</blockquote>
<pre style="background:cornsilk">
HTTP/1.0 101 Switching Protocols\r\n
Server: BaseHTTP/0.3 Python/2.7.6\r\n
Date: Tue, 13 May 2014 01:32:08 GMT\r\n
Upgrade: websocket\r\n
Connection: Upgrade\r\n
Sec-WebSocket-Accept: tZGC3i4YfiIrPSNfbfkx+JyrLYg=\r\n
\r\n
</pre>
```
At this point the WebSocket handshake is finished, and the client begins to send its data, a TLS [Client Hello](https://tools.ietf.org/html/rfc5246#section-7.4.1.2) message preceded by a [[span(WebSocket frame header,style=background:coral)]]. You can also see the [[span(cipher suite list,style=background:skyblue)]], [[span(server name,style=background:limegreen)]], and [[span(TLS extensions,style=background:palegoldenrod)]], just as in the [[#OrdinaryTor|ordinary Tor]] example.
```
<pre style="background:cornsilk">
00000000  <span style="background:coral">82 7e 00 fe</span> 16 03 01 00  f9 01 00 00 f5 03 03 1c <span style="background:coral">.~..</span>.... ........
00000010  54 ec ef 3b 2e bc 2b ec  e0 3b 14 ff 10 8a a5 3b T..;..+. .;.....;
00000020  1f d0 04 a6 f8 47 73 52  d3 a0 9e d1 13 8d bd 00 .....GsR ........
00000030  00 48 <span style="background:skyblue">c0 0a c0 14 00 88  00 87 00 39 00 38 c0 0f</span> .H<span style="background:skyblue">...... ...9.8..</span>
00000040  <span style="background:skyblue">c0 05 00 84 00 35 c0 07  c0 09 c0 11 c0 13 00 45</span> <span style="background:skyblue">.....5.. .......E</span>
00000050  <span style="background:skyblue">00 44 00 33 00 32 c0 0c  c0 0e c0 02 c0 04 00 96</span> <span style="background:skyblue">.D.3.2.. ........</span>
00000060  <span style="background:skyblue">00 41 00 04 00 05 00 2f  c0 08 c0 12 00 16 00 13</span> <span style="background:skyblue">.A...../ ........</span>
00000070  <span style="background:skyblue">c0 0d c0 03 fe ff 00 0a  00 ff</span> 01 00 00 84 <span style="background:palegoldenrod">00 00</span> <span style="background:skyblue">........ ..</span>....<span style="background:palegoldenrod">..</span>
00000080  <span style="background:palegoldenrod">00 13 00 11 00 00 0e <span style="background:limegreen">77  77 77 2e 64 7a 71 36 35</span></span> <span style="background:palegoldenrod">.......<span style="background:limegreen">w ww.dzq65</span></span>
00000090  <span style="background:palegoldenrod"><span style="background:limegreen">71 2e 63 6f 6d</span> 00 0b 00  04 03 00 01 02 00 0a 00</span> <span style="background:palegoldenrod"><span style="background:limegreen">q.com</span>... ........</span>
000000A0  <span style="background:palegoldenrod">34 00 32 00 0e 00 0d 00  19 00 0b 00 0c 00 18 00</span> <span style="background:palegoldenrod">4.2..... ........</span>
000000B0  <span style="background:palegoldenrod">09 00 0a 00 16 00 17 00  08 00 06 00 07 00 14 00</span> <span style="background:palegoldenrod">........ ........</span>
000000C0  <span style="background:palegoldenrod">15 00 04 00 05 00 12 00  13 00 01 00 02 00 03 00</span> <span style="background:palegoldenrod">........ ........</span>
000000D0  <span style="background:palegoldenrod">0f 00 10 00 11 00 23 00  00 00 0d 00 20 00 1e 06</span> <span style="background:palegoldenrod">......#. .... ...</span>
000000E0  <span style="background:palegoldenrod">01 06 02 06 03 05 01 05  02 05 03 04 01 04 02 04</span> <span style="background:palegoldenrod">........ ........</span>
000000F0  <span style="background:palegoldenrod">03 03 01 03 02 03 03 02  01 02 02 02 03 00 0f 00</span> <span style="background:palegoldenrod">........ ........</span>
00000100  <span style="background:palegoldenrod">01 01</span>                                            <span style="background:palegoldenrod">..</span>
</pre>
```
The flash proxy sends back some more data, a TLS [Server Hello](https://tools.ietf.org/html/rfc5246#section-7.4.1.3) preceded by a [[span(WebSocket header,style=background:coral)]]. The Server Hello includes a certificate with domain names and expiration dates, which are visible as plain text. The WebSocket header is 4 bytes longer in this direction than in the other, because it includes the [masking key](https://tools.ietf.org/html/rfc6455#section-5.3), which is set to [[span(00 00 00 00,style=background:coral)]] for this example.
```
<blockquote>
<pre style="background:lavender">
00000000  <span style="background:coral">82 fe 02 f2 00 00 00 00</span>  16 03 03 00 3e 02 00 00 <span style="background:coral">........</span> ....&gt;...
00000010  3a 03 03 53 71 76 ca 43  e3 5e e8 d3 49 fd 34 67 :..Sqv.C .^..I.4g
00000020  2c 0c cc 35 5f d7 86 84  3a 3c 7d 3d 65 74 c1 0c ,..5_... :&lt;}=et..
00000030  5c 39 dc 00 <span style="background:skyblue">c0 14</span> 00 00  12 <span style="background:palegoldenrod">ff 01 00 01 00 00 0b</span> \9..<span style="background:skyblue">..</span>.. .<span style="background:palegoldenrod">.......</span>
00000040  <span style="background:palegoldenrod">00 04 03 00 01 02 00 0f  00 01 01</span> 16 03 03 01 cf <span style="background:palegoldenrod">........ ...</span>.....
00000050  0b 00 01 cb 00 01 c8 00  01 c5 30 82 01 c1 30 82 ........ ..0...0.
00000060  01 2a a0 03 02 01 02 02  09 00 8b 10 ca 28 11 72 .*...... .....(.r
00000070  89 fb 30 0d 06 09 2a 86  48 86 f7 0d 01 01 05 05 ..0...*. H.......
00000080  00 30 23 31 21 30 1f 06  03 55 04 03 13 18 77 77 .0#1!0.. .U....ww
00000090  77 2e 70 6b 67 6d 71 68  37 74 6d 6a 34 68 35 63 w.pkgmqh 7tmj4h5c
000000A0  77 6d 2e 63 6f 6d 30 1e  17 0d 31 34 30 31 32 33 wm.com0. ..140123
000000B0  30 30 30 30 30 30 5a 17  0d 31 34 31 30 30 31 30 000000Z. .1410010
000000C0  30 30 30 30 30 5a 30 22  31 20 30 1e 06 03 55 04 00000Z0" 1 0...U.
000000D0  03 13 17 <span style="background:limegreen">77 77 77 2e 6c  75 62 6d 7a 36 63 73 69</span> ...<span style="background:limegreen">www.l ubmz6csi</span>
000000E0  <span style="background:limegreen">6c 69 78 74 64 70 2e 6e  65 74</span> 30 81 9f 30 0d 06 <span style="background:limegreen">lixtdp.n et</span>0..0..
000000F0  09 2a 86 48 86 f7 0d 01  01 01 05 00 03 81 8d 00 .*.H.... ........
00000100  30 81 89 02 81 81 00 ba  77 59 01 ee b8 d2 62 5c 0....... wY....b\
00000110  35 d8 4a f5 f7 43 87 a3  9e b6 f0 02 dc 5d 8e fe 5.J..C.. .....]..
00000120  99 1a e3 b0 7a 25 1e 0e  46 d4 c4 11 39 26 7f 86 ....z%.. F...9&amp;..
00000130  ff 05 91 5f 7a 0b 31 ec  0b b3 98 cf ca 00 54 9d ..._z.1. ......T.
00000140  15 4c 8e f5 3c ce be 82  ec 66 ee c8 11 86 63 42 .L..&lt;... .f....cB
00000150  c2 23 7c 78 a1 90 ea 33  28 80 d5 99 71 ea 01 56 .#|x...3 (...q..V
00000160  14 3b c9 2a 55 71 0f a9  3a 5e 6a 90 02 64 dd d4 .;.*Uq.. :^j..d..
00000170  2f 7a e9 0a 14 36 9d 7f  c8 92 b1 ae f3 dc 85 d5 /z...6.. ........
00000180  d3 f2 07 c5 f4 72 11 02  03 01 00 01 30 0d 06 09 .....r.. ....0...
00000190  2a 86 48 86 f7 0d 01 01  05 05 00 03 81 81 00 aa *.H..... ........
000001A0  11 83 57 d4 6e 48 f1 75  cb b3 ce b5 54 21 d3 c6 ..W.nH.u ....T!..
000001B0  51 25 4d a4 75 36 34 0f  c0 f8 75 cf eb 38 9b b5 Q%M.u64. ..u..8..
000001C0  f5 6e 72 54 6b e6 c1 bd  fd fa 62 f7 70 4b 33 bd .nrTk... ..b.pK3.
000001D0  6a b2 2d 64 54 7e 2f 41  5e 0f 4b 4d 6a 86 0d 95 j.-dT~/A ^.KMj...
000001E0  a8 d1 51 19 e2 23 15 8d  a4 a2 de b7 f5 05 60 ce ..Q..#.. ......`.
000001F0  11 0c 13 f6 d0 f3 8e d9  b9 3c 62 01 24 6f f4 74 ........ .&lt;b.$o.t
00000200  e3 f5 49 ad 6d 5f 98 1b  ab e3 88 0c d4 5a 79 0f ..I.m_.. .....Zy.
00000210  8f 22 5f 0d 06 ba a9 fe  a3 d6 da 6d 2e 01 ec 16 ."_..... ...m....
00000220  03 03 00 cd 0c 00 00 c9  03 00 17 41 04 4c 7b b8 ........ ...A.L{.
00000230  3d 2e b0 d7 64 ff 2b 4f  3f 42 69 12 8a 5d 45 c2 =...d.+O ?Bi..]E.
00000240  fc f1 a3 48 76 9b 37 a5  81 5c 92 3d 8a eb 07 15 ...Hv.7. .\.=....
00000250  dc de 8d 98 df df bf 79  e9 2d 21 57 61 37 18 08 .......y .-!Wa7..
00000260  a8 53 45 09 4a 05 c3 a1  df 21 37 f1 98 06 01 00 .SE.J... .!7.....
00000270  80 03 24 ba 61 73 32 ae  b6 3b 89 e5 a7 24 a3 bb ..$.as2. .;...$..
00000280  e2 e9 72 6a 40 5b 31 07  46 6e bc af 13 31 ae 95 ..rj@[1. Fn...1..
00000290  bb 48 a1 cc d0 f4 67 f6  ea 38 29 c7 69 0a 70 25 .H....g. .8).i.p%
000002A0  57 38 38 cd 7b fa 65 c5  3c 27 ec e0 cd fe 43 61 W88.{.e. &lt;'....Ca
000002B0  ff 9d ee ca 9e da 65 7a  e9 0e ea 4c a9 27 70 39 ......ez ...L.'p9
000002C0  07 da 82 c3 de 46 e0 4b  1c 2b 9e 0d 5c dd 89 9d .....F.K .+..\...
000002D0  3d 47 69 a5 cf 72 6f 0f  07 8c 59 b4 51 5e dd 31 =Gi..ro. ..Y.Q^.1
000002E0  fb 59 34 9f 84 cd d6 d1  f9 de cf ba 51 ad 32 15 .Y4..... ....Q.2.
000002F0  92 16 03 03 00 04 0e 00  00 00                   ........ ..
</pre>
</blockquote>
```
After this, the communication continues in both directions, socket-like with WebSocket framing, until the flash proxy disappears.

## Flash proxy rendezvous

There's one more piece to flash proxy, which is how the client advertises that it needs service from a flash proxy in the first place. This process is called "rendezvous." The client must send its IP address to the flash proxy system in a way that the censor can't detect and block. There are a few ways to do it, but most often it is done by a program called [flashproxy-reg-appspot](https://gitweb.torproject.org/flashproxy.git/tree/doc/flashproxy-reg-appspot.1.txt) and its helper [flashproxy-reg-url](https://gitweb.torproject.org/flashproxy.git/tree/doc/flashproxy-reg-url.1.txt). flashproxy-reg-appspot works by reflecting an HTTP request through a web app running on [Google App Engine](https://developers.google.com/appengine/), using the domain name www.google.com so the censor can't tell you are talking to App Engine, and encoding your IP address as part of the URL.

When you run flashproxy-reg-appspot, the client first sends one covert HTTPS request in order to learn its own external IP address. The censor doesn't get to see the contents of these messages; they are inside HTTPS encryption. The packets will be marked with a [[span(black border,style=border:2px solid black)]] to indicate that we are looking inside the encryption.
```
<pre style="background:cornsilk;border:2px solid black">
GET /ip HTTP/1.1\r\n
Accept-Encoding: identity\r\n
Host: <span style="background:springgreen">fp-reg-a.appspot.com</span>\r\n
Connection: close\r\n
User-Agent: Python-urllib/2.7\r\n
\r\n
</pre>
<blockquote>
<pre style="background:lavender;border:2px solid black">
HTTP/1.1 200 OK\r\n
Content-Type: application/octet-stream\r\n
Date: Tue, 13 May 2014 02:47:50 GMT\r\n
Server: Google Frontend\r\n
Cache-Control: private\r\n
Alternate-Protocol: 443:quic\r\n
Connection: close\r\n
\r\n
192.0.2.101
</pre>
</blockquote>
```
The Host header [[span(fp-reg-a.appspot.com,style=background:springgreen)]] is the actual destination of the request—the web app running at that domain forwards the request to the flash proxy system—but the censor instead sees the request destined for www.google.com.

The client then sends another HTTPS request that contains an encrypted payload that will be forwarded to a flash proxy:
```
<pre style="background:cornsilk;border:2px solid black">
GET /reg/IiYtnmOro5k8IkFxIOljS1k8Z3dC3M0m_mM40PZo4STDY1vqn4xC6l9zAOjnr_Gw5xGBnKfbjDiyc4uaN5DxkseUAw4NUxwr6UySYYYMpssBkgRe4P5LBGlQy7B8rjzFlg8snXx2yUIbfrX2hP11XB-Tvr2po6VeNEEAiXQG48waXknztb0KBTM6qTXfNiZf3QwCW7aap-yu5IwFz6thhZ1NLwNEdp0tHn42m4sbEZzANM3sFv0kBlBn9IOWtFiwzdacjS6rXiuULhhC7rR2WuhsjVctdus8qNmUnfm22c36KPIgqyB5uDSR45pq5rHBdL_ZSsadwKQwQeW_6rohaw== HTTP/1.1\r\n
Accept-Encoding: identity\r\n
Host: <span style="background:springgreen">fp-reg-a.appspot.com</span>\r\n
Connection: close\r\n
User-Agent: Python-urllib/2.7\r\n
\r\n
</pre>
<blockquote>
<pre style="background:lavender;border:2px solid black">
HTTP/1.1 204 No Content\r\n
Strict-Transport-Security: max-age=15768000\r\n
Via: HTTP/1.1 GWA\r\n
Date: Tue, 13 May 2014 02:47:52 GMT\r\n
Content-Type: text/html\r\n
Server: Google Frontend\r\n
Content-Length: 0\r\n
Alternate-Protocol: 443:quic\r\n
Connection: close\r\n
\r\n
</pre>
</blockquote>
```
The response `204 No Content` means that the web app didn't have to do any work. It merely copied the long URL it was given and sent it to the flash proxy system. That long URL is actually an encoded ciphertext. Here is what it looks like decrypted:
```
client=192.0.2.101%3A9000&client-transport=websocket
```

# meek

```
meek 0.5
tor 0.2.24.1-1
In order to get the HTTP captures I had to hack the browser extension to relay plaintext through an Ncat listener that added its own SSL.
ncat -l 8080 -k -v --sh-exec 'ncat --ssl www.google.com 443'
In components/main.js, remove the https check from requestOk and do:
-        this.channel = ioService.newChannel(req.url, null, null)              
+        this.channel = ioService.newChannel("http://127.0.0.1:8080/", null, null)
./Browser/firefox -profile Data/Browser/profile.meek-http-helper
Check the port number XXXXX used by the helper, then run
ClientTransportPlugin meek exec ./meek-client --url=https://meek-reflect.appspot.com/ --front=www.google.com --log meek-client.log --helper 127.0.0.1:XXXXX
Capture localhost port 8080.
```

[[meek]] uses HTTP as a transport, and TLS to hide the contents of messages. It reflects its HTTP requests through a third party server like [App Engine](https://developers.google.com/appengine/), using a technical trick to make it look like it is talking to a different server, one that is expensive for the censor to block. It uses a browser plugin in order to camouflage its TLS fingerprint.

Here's the first thing a censor sees when you connect using meek. It's a TLS [Client Hello](https://tools.ietf.org/html/rfc5246#section-7.4.1.2), like in the [[#OrdinaryTor|ordinary Tor]] example, but it's different: the [[span(cipher suites,style=background:skyblue)]] are different, the [[span(extensions,style=background:palegoldenrod)]] are different, and the server name is [[span(www.google.com,style=background:limegreen)]] rather than a randomly generated Tor name. The TLS looks different because it's generated by a [[#11183|web browser extension]], not by Tor, and the packet is in fact being sent to www.google.com, not to a Tor relay. Read on for the "domain fronting" trick that causes www.google.com to send the traffic to a Tor relay behind the scenes.
```
<pre style="background:cornsilk">
00000000  16 03 01 00 a9 01 00 00  a5 03 01 17 24 0d d9 fe ........ ....$...
00000010  72 b8 1e 89 82 f2 2f 98  8a e4 88 89 85 0f dd 1e r...../. ........
00000020  12 7d 76 72 ec 6e 1e 15  d9 f3 5c 00 <span style="background:skyblue">00 46 c0 0a</span> .}vr.n.. ..\.<span style="background:skyblue">.F..</span>
00000030  <span style="background:skyblue">c0 09 c0 13 c0 14 c0 08  c0 12 c0 07 c0 11 00 33</span> <span style="background:skyblue">........ .......3</span>
00000040  <span style="background:skyblue">00 32 00 45 00 44 00 39  00 38 00 88 00 87 00 16</span> <span style="background:skyblue">.2.E.D.9 .8......</span>
00000050  <span style="background:skyblue">00 13 c0 04 c0 0e c0 05  c0 0f c0 03 c0 0d c0 02</span> <span style="background:skyblue">........ ........</span>
00000060  <span style="background:skyblue">c0 0c 00 2f 00 41 00 35  00 84 00 96 fe ff 00 0a</span> <span style="background:skyblue">.../.A.5 ........</span>
00000070  <span style="background:skyblue">00 05 00 04</span> 01 00 <span style="background:palegoldenrod">00 36  00 00 00 13 00 11 00 00</span> <span style="background:skyblue">....</span>..<span style="background:palegoldenrod">.6 ........</span>
00000080  <span style="background:palegoldenrod">0e <span style="background:limegreen">77 77 77 2e 67 6f 6f  67 6c 65 2e 63 6f 6d</span> ff</span> <span style="background:palegoldenrod">.<span style="background:limegreen">www.goo gle.com</span>.</span>
00000090  <span style="background:palegoldenrod">01 00 01 00 00 0a 00 08  00 06 00 17 00 18 00 19</span> <span style="background:palegoldenrod">........ ........</span>
000000A0  <span style="background:palegoldenrod">00 0b 00 02 01 00 00 23  00 00 33 74 00 00</span>       <span style="background:palegoldenrod">.......# ..3t..</span>
</pre>
```

The [Server Hello](https://tools.ietf.org/html/rfc5246#section-7.4.1.3) reply from www.google.com is ordinary—it's exactly what you would get if you went to https://www.google.com/ in a web browser. Google has a longer certificate than the typical Tor relay.
```
<blockquote>
<pre style="background:lavender">
00000000  16 03 01 00 5e 02 00 00  5a 03 01 53 75 b8 12 35 ....^... Z..Su..5
00000010  d4 9d fc 74 43 ce 67 ef  89 0c a3 d4 cc 72 1f 8d ...tC.g. .....r..
00000020  5d 3d d9 4e 5e e4 6f ff  83 d6 2f 00 <span style="background:skyblue">c0 11</span> 00 <span style="background:palegoldenrod">00</span> ]=.N^.o. ../.<span style="background:skyblue">..</span>.<span style="background:palegoldenrod">.</span>
00000030  <span style="background:palegoldenrod">32 00 00 00 00 ff 01 00  01 00 00 0b 00 04 03 00</span> <span style="background:palegoldenrod">2....... ........</span>
00000040  <span style="background:palegoldenrod">01 02 00 23 00 00 33 74  00 19 08 73 70 64 79 2f</span> <span style="background:palegoldenrod">...#..3t ...spdy/</span>
00000050  <span style="background:palegoldenrod">33 2e 31 06 73 70 64 79  2f 33 08 68 74 74 70 2f</span> <span style="background:palegoldenrod">3.1.spdy /3.http/</span>
00000060  <span style="background:palegoldenrod">31 2e 31</span> 16 03 01 0c 13  0b 00 0c 0f 00 0c 0c 00 <span style="background:palegoldenrod">1.1</span>..... ........
00000070  04 7a 30 82 04 76 30 82  03 5e a0 03 02 01 02 02 .z0..v0. .^......
00000080  08 24 d8 55 1d 34 8a 41  a6 30 0d 06 09 2a 86 48 .$.U.4.A .0...*.H
00000090  86 f7 0d 01 01 05 05 00  30 49 31 0b 30 09 06 03 ........ 0I1.0...
000000A0  55 04 06 13 02 55 53 31  13 30 11 06 03 55 04 0a U....US1 .0...U..
000000B0  13 0a 47 6f 6f 67 6c 65  20 49 6e 63 31 25 30 23 ..Google  Inc1%0#
000000C0  06 03 55 04 03 13 1c 47  6f 6f 67 6c 65 20 49 6e ..U....G oogle In
000000D0  74 65 72 6e 65 74 20 41  75 74 68 6f 72 69 74 79 ternet A uthority
000000E0  20 47 32 30 1e 17 0d 31  34 30 35 30 37 31 32 31  G20...1 40507121
000000F0  33 35 33 5a 17 0d 31 34  30 38 30 35 30 30 30 30 353Z..14 08050000
00000100  30 30 5a 30 68 31 0b 30  09 06 03 55 04 06 13 02 00Z0h1.0 ...U....
00000110  55 53 31 13 30 11 06 03  55 04 08 0c 0a 43 61 6c US1.0... U....Cal
00000120  69 66 6f 72 6e 69 61 31  16 30 14 06 03 55 04 07 ifornia1 .0...U..
00000130  0c 0d 4d 6f 75 6e 74 61  69 6e 20 56 69 65 77 31 ..Mounta in View1
00000140  13 30 11 06 03 55 04 0a  0c 0a 47 6f 6f 67 6c 65 .0...U.. ..Google
00000150  20 49 6e 63 31 17 30 15  06 03 55 04 03 0c 0e <span style="background:limegreen">77</span>  Inc1.0. ..U....<span style="background:limegreen">w</span>
00000160  <span style="background:limegreen">77 77 2e 67 6f 6f 67 6c  65 2e 63 6f 6d</span> 30 82 01 <span style="background:limegreen">ww.googl e.com</span>0..
00000170  22 30 0d 06 09 2a 86 48  86 f7 0d 01 01 01 05 00 "0...*.H ........
00000180  03 82 01 0f 00 30 82 01  0a 02 82 01 01 00 ba 1c .....0.. ........
00000190  1c c5 ab c0 f7 3b 99 de  0f fa 32 ad c6 9d 9c 14 .....;.. ..2.....
000001A0  aa 47 03 90 a9 3f f3 10  0b c5 e6 f7 8a 2e ea 71 .G...?.. .......q
000001B0  d5 e9 2b da 3e ec d8 d0  08 19 18 8c 4b 4f 12 07 ..+.&gt;... ....KO..
000001C0  8a 84 ba 49 4e d8 65 e6  96 62 3d 7e ad cb 30 74 ...IN.e. .b=~..0t
000001D0  e3 00 9f bf 01 5f 86 65  74 94 16 26 c3 dd 04 a5 ....._.e t..&amp;....
000001E0  d1 1c c3 20 b4 a1 a3 de  c0 22 0b fe bd 5f 26 a7 ... .... ."..._&amp;.
000001F0  32 1c 02 50 6a 62 3a 24  03 df 71 cb 92 86 28 99 2..Pjb:$ ..q...(.
00000200  5d e0 f5 cc b6 46 5f e3  38 92 46 8c 20 fe 3b 1d ]....F_. 8.F. .;.
00000210  a7 a3 cb 2c d8 e0 d0 1d  68 5c d4 e4 9e ab 3c 6c ...,.... h\....&lt;l
00000220  a3 81 a7 9a 46 38 c2 06  7b 6f 46 88 f7 55 f6 e6 ....F8.. {oF..U..
00000230  04 f2 af 6b f6 cd 4e 59  a1 75 55 6d 65 14 dc 86 ...k..NY .uUme...
00000240  00 99 1e 6c e1 60 d1 0f  0a e7 d6 d6 8c d0 34 1d ...l.`.. ......4.
00000250  02 11 76 ea 79 6d ba dd  8e e7 e0 9a c0 44 c9 4f ..v.ym.. .....D.O
00000260  48 ba a5 43 fe 88 f9 88  6d 93 b1 fa d3 66 22 84 H..C.... m....f".
00000270  cb 67 c5 13 a5 c5 a0 43  d1 0c 95 a8 eb 64 a7 6a .g.....C .....d.j
00000280  9c 52 01 8c 45 e7 00 9a  d3 e6 69 6b 38 cb 02 03 .R..E... ..ik8...
00000290  01 00 01 a3 82 01 41 30  82 01 3d 30 1d 06 03 55 ......A0 ..=0...U
000002A0  1d 25 04 16 30 14 06 08  2b 06 01 05 05 07 03 01 .%..0... +.......
000002B0  06 08 2b 06 01 05 05 07  03 02 30 19 06 03 55 1d ..+..... ..0...U.
000002C0  11 04 12 30 10 82 0e <span style="background:limegreen">77  77 77 2e 67 6f 6f 67 6c</span> ...0...<span style="background:limegreen">w ww.googl</span>
000002D0  <span style="background:limegreen">65 2e 63 6f 6d</span> 30 68 06  08 2b 06 01 05 05 07 01 <span style="background:limegreen">e.com</span>0h. .+......
000002E0  01 04 5c 30 5a 30 2b 06  08 2b 06 01 05 05 07 30 ..\0Z0+. .+.....0
000002F0  02 86 1f 68 74 74 70 3a  2f 2f 70 6b 69 2e 67 6f ...http: //pki.go
00000300  6f 67 6c 65 2e 63 6f 6d  2f 47 49 41 47 32 2e 63 ogle.com /GIAG2.c
00000310  72 74 30 2b 06 08 2b 06  01 05 05 07 30 01 86 1f rt0+..+. ....0...
00000320  68 74 74 70 3a 2f 2f 63  6c 69 65 6e 74 73 31 2e http://c lients1.
00000330  67 6f 6f 67 6c 65 2e 63  6f 6d 2f 6f 63 73 70 30 google.c om/ocsp0
00000340  1d 06 03 55 1d 0e 04 16  04 14 69 27 5e 70 b7 59 ...U.... ..i'^p.Y
00000350  9c 1f fd 8f 88 fa b1 c5  df ee 9e 1c f6 93 30 0c ........ ......0.
00000360  06 03 55 1d 13 01 01 ff  04 02 30 00 30 1f 06 03 ..U..... ..0.0...
00000370  55 1d 23 04 18 30 16 80  14 4a dd 06 16 1b bc f6 U.#..0.. .J......
00000380  68 b5 76 f5 81 b6 bb 62  1a ba 5a 81 2f 30 17 06 h.v....b ..Z./0..
00000390  03 55 1d 20 04 10 30 0e  30 0c 06 0a 2b 06 01 04 .U. ..0. 0...+...
000003A0  01 d6 79 02 05 01 30 30  06 03 55 1d 1f 04 29 30 ..y...00 ..U...)0
000003B0  27 30 25 a0 23 a0 21 86  1f 68 74 74 70 3a 2f 2f '0%.#.!. .http://
000003C0  70 6b 69 2e 67 6f 6f 67  6c 65 2e 63 6f 6d 2f 47 pki.goog le.com/G
000003D0  49 41 47 32 2e 63 72 6c  30 0d 06 09 2a 86 48 86 IAG2.crl 0...*.H.
000003E0  f7 0d 01 01 05 05 00 03  82 01 01 00 4a ab f5 a5 ........ ....J...
000003F0  dd 05 80 85 38 ca 49 e5  be 66 b4 52 16 e2 8e 50 ....8.I. .f.R...P
00000400  8a 51 96 70 fe 3f 04 a8  b8 9b 63 c8 87 f8 55 c6 .Q.p.?.. ..c...U.
00000410  78 77 06 47 25 9d 8c ee  ff 8d ec 4e f8 ab 39 7c xw.G%... ...N..9|
00000420  6f 1c 62 68 b8 22 fe 53  39 92 3d f0 47 eb 61 3c o.bh.".S 9.=.G.a&lt;
00000430  13 4e a0 a6 9a ec 2d 23  f5 5c 27 0d 22 d6 a1 16 .N....-# .\'."...
00000440  73 5a 2d 19 f8 df ec 74  e3 c3 02 4c 6f 00 10 09 sZ-....t ...Lo...
00000450  1f e2 2a 41 9d 97 e0 65  50 f8 b9 f5 a2 87 bc a1 ..*A...e P.......
00000460  00 3a bc f1 0a 60 04 6f  d9 38 44 6a e8 04 e4 81 .:...`.o .8Dj....
00000470  84 21 09 36 37 99 1e f0  15 b7 59 99 07 e8 50 2d .!.67... ..Y...P-
00000480  bd ae 86 33 ba 63 39 2c  cc 19 27 a7 07 cd 22 f2 ...3.c9, ..'...".
00000490  f5 43 53 70 d6 eb 52 f5  e8 91 ab 3c c4 a2 02 ce .CSp..R. ...&lt;....
000004A0  32 b3 39 eb 54 85 09 5b  a5 7a 15 62 8b 7c 51 49 2.9.T..[ .z.b.|QI
000004B0  a4 f6 a8 04 55 a9 12 02  da 9d 17 f4 47 8c e5 f6 ....U... ....G...
000004C0  89 0b b2 50 0c e6 04 8e  2c ea 58 cd 9a 1a 41 b7 ...P.... ,.X...A.
000004D0  0b 5d 8a 2f c8 d1 e8 2d  1f c0 ac fa 55 83 98 e6 .]./...- ....U...
000004E0  01 a9 7e 55 15 96 68 2d  a6 e9 7b 1b 00 04 08 30 ..~U..h- ..{....0
000004F0  82 04 04 30 82 02 ec a0  03 02 01 02 02 03 02 3a ...0.... .......:
00000500  69 30 0d 06 09 2a 86 48  86 f7 0d 01 01 05 05 00 i0...*.H ........
00000510  30 42 31 0b 30 09 06 03  55 04 06 13 02 55 53 31 0B1.0... U....US1
00000520  16 30 14 06 03 55 04 0a  13 0d 47 65 6f 54 72 75 .0...U.. ..GeoTru
00000530  73 74 20 49 6e 63 2e 31  1b 30 19 06 03 55 04 03 st Inc.1 .0...U..
00000540  13 12 47 65 6f 54 72 75  73 74 20 47 6c 6f 62 61 ..GeoTru st Globa
00000550  6c 20 43 41 30 1e 17 0d  31 33 30 34 30 35 31 35 l CA0... 13040515
00000560  31 35 35 35 5a 17 0d 31  35 30 34 30 34 31 35 31 1555Z..1 50404151
00000570  35 35 35 5a 30 49 31 0b  30 09 06 03 55 04 06 13 555Z0I1. 0...U...
00000580  02 55 53 31 13 30 11 06  03 55                   .US1.0.. .U
0000058A  04 0a 13 0a 47 6f 6f 67  6c 65 20 49 6e 63 31 25 ....Goog le Inc1%
0000059A  30 23 06 03 55 04 03 13  1c 47 6f 6f 67 6c 65 20 0#..U... .Google 
000005AA  49 6e 74 65 72 6e 65 74  20 41 75 74 68 6f 72 69 Internet  Authori
000005BA  74 79 20 47 32 30 82 01  22 30 0d 06 09 2a 86 48 ty G20.. "0...*.H
000005CA  86 f7 0d 01 01 01 05 00  03 82 01 0f 00 30 82 01 ........ .....0..
000005DA  0a 02 82 01 01 00 9c 2a  04 77 5c d8 50 91 3a 06 .......* .w\.P.:.
000005EA  a3 82 e0 d8 50 48 bc 89  3f f1 19 70 1a 88 46 7e ....PH.. ?..p..F~
000005FA  e0 8f c5 f1 89 ce 21 ee  5a fe 61 0d b7 32 44 89 ......!. Z.a..2D.
0000060A  a0 74 0b 53 4f 55 a4 ce  82 62 95 ee eb 59 5f c6 .t.SOU.. .b...Y_.
0000061A  e1 05 80 12 c4 5e 94 3f  bc 5b 48 38 f4 53 f7 24 .....^.? .[H8.S.$
0000062A  e6 fb 91 e9 15 c4 cf f4  53 0d f4 4a fc 9f 54 de ........ S..J..T.
0000063A  7d be a0 6b 6f 87 c0 d0  50 1f 28 30 03 40 da 08 }..ko... P.(0.@..
0000064A  73 51 6c 7f ff 3a 3c a7  37 06 8e bd 4b 11 04 eb sQl..:&lt;. 7...K...
0000065A  7d 24 de e6 f9 fc 31 71  fb 94 d5 60 f3 2e 4a af }$....1q ...`..J.
0000066A  42 d2 cb ea c4 6a 1a b2  cc 53 dd 15 4b 8b 1f c8 B....j.. .S..K...
0000067A  19 61 1f cd 9d a8 3e 63  2b 84 35 69 65 84 c8 19 .a....&gt;c +.5ie...
0000068A  c5 46 22 f8 53 95 be e3  80 4a 10 c6 2a ec ba 97 .F".S... .J..*...
0000069A  20 11 c7 39 99 10 04 a0  f0 61 7a 95 25 8c 4e 52  ..9.... .az.%.NR
000006AA  75 e2 b6 ed 08 ca 14 fc  ce 22 6a b3 4e cf 46 03 u....... ."j.N.F.
000006BA  97 97 03 7e c0 b1 de 7b  af 45 33 cf ba 3e 71 b7 ...~...{ .E3..&gt;q.
000006CA  de f4 25 25 c2 0d 35 89  9d 9d fb 0e 11 79 89 1e ..%%..5. .....y..
000006DA  37 c5 af 8e 72 69 02 03  01 00 01 a3 81 fb 30 81 7...ri.. ......0.
000006EA  f8 30 1f 06 03 55 1d 23  04 18 30 16 80 14 c0 7a .0...U.# ..0....z
000006FA  98 68 8d 89 fb ab 05 64  0c 11 7d aa 7d 65 b8 ca .h.....d ..}.}e..
0000070A  cc 4e 30 1d 06 03 55 1d  0e 04 16 04 14 4a dd 06 .N0...U. .....J..
0000071A  16 1b bc f6 68 b5 76 f5  81 b6 bb 62 1a ba 5a 81 ....h.v. ...b..Z.
0000072A  2f 30 12 06 03 55 1d 13  01 01 ff 04 08 30 06 01 /0...U.. .....0..
0000073A  01 ff 02 01 00 30 0e 06  03 55 1d 0f 01 01 ff 04 .....0.. .U......
0000074A  04 03 02 01 06 30 3a 06  03 55 1d 1f 04 33 30 31 .....0:. .U...301
0000075A  30 2f a0 2d a0 2b 86 29  68 74 74 70 3a 2f 2f 63 0/.-.+.) http://c
0000076A  72 6c 2e 67 65 6f 74 72  75 73 74 2e 63 6f 6d 2f rl.geotr ust.com/
0000077A  63 72 6c 73 2f 67 74 67  6c 6f 62 61 6c 2e 63 72 crls/gtg lobal.cr
0000078A  6c 30 3d 06 08 2b 06 01  05 05 07 01 01 04 31 30 l0=..+.. ......10
0000079A  2f 30 2d 06 08 2b 06 01  05 05 07 30 01 86 21 68 /0-..+.. ...0..!h
000007AA  74 74 70 3a 2f 2f 67 74  67 6c 6f 62 61 6c 2d 6f ttp://gt global-o
000007BA  63 73 70 2e 67 65 6f 74  72 75 73 74 2e 63 6f 6d csp.geot rust.com
000007CA  30 17 06 03 55 1d 20 04  10 30 0e 30 0c 06 0a 2b 0...U. . .0.0...+
000007DA  06 01 04 01 d6 79 02 05  01 30 0d 06 09 2a 86 48 .....y.. .0...*.H
000007EA  86 f7 0d 01 01 05 05 00  03 82 01 01 00 36 d7 06 ........ .....6..
000007FA  80 11 27 ad 2a 14 9b 38  77 b3 23 a0 75 58 bb b1 ..'.*..8 w.#.uX..
0000080A  7e 83 42 ba 72 da 1e d8  8e 36 06 97 e0 f0 95 3b ~.B.r... .6.....;
0000081A  37 fd 1b 42 58 fe 22 c8  6b bd 38 5e d1 3b 25 6e 7..BX.". k.8^.;%n
0000082A  12 eb 5e 67 76 46 40 90  da 14 c8 78 0d ed 95 66 ..^gvF@. ...x...f
0000083A  da 8e 86 6f 80 a1 ba 56  32 95 86 dc dc 6a ca 04 ...o...V 2....j..
0000084A  8c 5b 7f f6 bf cc 6f 85  03 58 c3 68 51 13 cd fd .[....o. .X.hQ...
0000085A  c8 f7 79 3d 99 35 f0 56  a3 bd e0 59 ed 4f 44 09 ..y=.5.V ...Y.OD.
0000086A  a3 9e 38 7a f6 46 d1 1d  12 9d 4f be d0 40 fc 55 ..8z.F.. ..O..@.U
0000087A  fe 06 5e 3c da 1c 56 bd  96 51 7b 6f 57 2a db a2 ..^&lt;..V. .Q{oW*..
0000088A  aa 96 dc 8c 74 c2 95 be  f0 6e 95 13 ff 17 f0 3c ....t... .n.....&lt;
0000089A  ac b2 10 8d cc 73 fb e8  8f 02 c6 f0 fb 33 b3 95 .....s.. .....3..
000008AA  3b e3 c2 cb 68 58 73 db  a8 24 62 3b 06 35 9d 0d ;...hXs. .$b;.5..
000008BA  a9 33 bd 78 03 90 2e 4c  78 5d 50 3a 81 d4 ee a0 .3.x...L x]P:....
000008CA  c8 70 38 dc b2 f9 67 fa  87 40 5d 61 c0 51 8f 6b .p8...g. .@]a.Q.k
000008DA  83 6b cd 05 3a ca e1 a7  05 78 fc ca da 94 d0 2c .k..:... .x.....,
000008EA  08 3d 7e 16 79 c8 a0 50  20 24 54 33 71 00 03 81 .=~.y..P  $T3q...
000008FA  30 82 03 7d 30 82 02 e6  a0 03 02 01 02 02 03 12 0..}0... ........
0000090A  bb e6 30 0d 06 09 2a 86  48 86 f7 0d 01 01 05 05 ..0...*. H.......
0000091A  00 30 4e 31 0b 30 09 06  03 55 04 06 13 02 55 53 .0N1.0.. .U....US
0000092A  31 10 30 0e 06 03 55 04  0a 13 07 45 71 75 69 66 1.0...U. ...Equif
0000093A  61 78 31 2d 30 2b 06 03  55 04 0b 13 24 45 71 75 ax1-0+.. U...$Equ
0000094A  69 66 61 78 20 53 65 63  75 72 65 20 43 65 72 74 ifax Sec ure Cert
0000095A  69 66 69 63 61 74 65 20  41 75 74 68 6f 72 69 74 ificate  Authorit
0000096A  79 30 1e 17 0d 30 32 30  35 32 31 30 34 30 30 30 y0...020 52104000
0000097A  30 5a 17 0d 31 38 30 38  32 31 30 34 30 30 30 30 0Z..1808 21040000
0000098A  5a 30 42 31 0b 30 09 06  03 55 04 06 13 02 55 53 Z0B1.0.. .U....US
0000099A  31 16 30 14 06 03 55 04  0a 13 0d 47 65 6f 54 72 1.0...U. ...GeoTr
000009AA  75 73 74 20 49 6e 63 2e  31 1b 30 19 06 03 55 04 ust Inc. 1.0...U.
000009BA  03 13 12 47 65 6f 54 72  75 73 74 20 47 6c 6f 62 ...GeoTr ust Glob
000009CA  61 6c 20 43 41 30 82 01  22 30 0d 06 09 2a 86 48 al CA0.. "0...*.H
000009DA  86 f7 0d 01 01 01 05 00  03 82 01 0f 00 30 82 01 ........ .....0..
000009EA  0a 02 82 01 01 00 da cc  18 63 30 fd f4 17 23 1a ........ .c0...#.
000009FA  56 7e 5b df 3c 6c 38 e4  71 b7 78 91 d4 bc a1 d8 V~[.&lt;l8. q.x.....
00000A0A  4c f8 a8 43 b6 03 e9 4d  21 07 08 88 da 58 2f 66 L..C...M !....X/f
00000A1A  39 29 bd 05 78 8b 9d 38  e8 05 b7 6a 7e 71 a4 e6 9)..x..8 ...j~q..
00000A2A  c4 60 a6 b0 ef 80 e4 89  28 0f 9e 25 d6 ed 83 f3 .`...... (..%....
00000A3A  ad a6 91 c7 98 c9 42 18  35 14 9d ad 98 46 92 2e ......B. 5....F..
00000A4A  4f ca f1 87 43 c1 16 95  57 2d 50 ef 89 2d 80 7a O...C... W-P..-.z
00000A5A  57 ad f2 ee 5f 6b d2 00  8d b9 14 f8 14 15 35 d9 W..._k.. ......5.
00000A6A  c0 46 a3 7b 72 c8 91 bf  c9 55 2b cd d0 97 3e 9c .F.{r... .U+...&gt;.
00000A7A  26 64 cc df ce 83 19 71  ca 4e e6 d4 d5 7b a9 19 &amp;d.....q .N...{..
00000A8A  cd 55 de c8 ec d2 5e 38  53 e5 5c 4f 8c 2d fe 50 .U....^8 S.\O.-.P
00000A9A  23 36 fc 66 e6 cb 8e a4  39 19 00 b7 95 02 39 91 #6.f.... 9.....9.
00000AAA  0b 0e fe 38 2e d1 1d 05  9a f6 4d 3e 6f 0f 07 1d ...8.... ..M&gt;o...
00000ABA  af 2c 1e 8f 60 39 e2 fa  36 53 13 39 d4 5e 26 2b .,..`9.. 6S.9.^&amp;+
00000ACA  db 3d a8 14 bd 32 eb 18  03 28 52 04 71 e5 ab 33 .=...2.. .(R.q..3
00000ADA  3d e1 38 bb 07 36 84 62  9c 79 ea 16 30 f4 5f c0 =.8..6.b .y..0._.
00000AEA  2b e8 71 6b e4 f9 02 03  01 00 01 a3 81 f0 30 81 +.qk.... ......0.
00000AFA  ed 30 1f 06 03 55 1d 23  04 18 30 16 80 14 48 e6 .0...U.# ..0...H.
00000B0A  68 f9 2b d2 b2 95 d7 47  d8 23                   h.+....G .#
00000B14  20 10 4f 33 98 90 9f d4  30 1d 06 03 55 1d 0e 04  .O3.... 0...U...
00000B24  16 04 14 c0 7a 98 68 8d  89 fb ab 05 64 0c 11 7d ....z.h. ....d..}
00000B34  aa 7d 65 b8 ca cc 4e 30  0f 06 03 55 1d 13 01 01 .}e...N0 ...U....
00000B44  ff 04 05 30 03 01 01 ff  30 0e 06 03 55 1d 0f 01 ...0.... 0...U...
00000B54  01 ff 04 04 03 02 01 06  30 3a 06 03 55 1d 1f 04 ........ 0:..U...
00000B64  33 30 31 30 2f a0 2d a0  2b 86 29 68 74 74 70 3a 3010/.-. +.)http:
00000B74  2f 2f 63 72 6c 2e 67 65  6f 74 72 75 73 74 2e 63 //crl.ge otrust.c
00000B84  6f 6d 2f 63 72 6c 73 2f  73 65 63 75 72 65 63 61 om/crls/ secureca
00000B94  2e 63 72 6c 30 4e 06 03  55 1d 20 04 47 30 45 30 .crl0N.. U. .G0E0
00000BA4  43 06 04 55 1d 20 00 30  3b 30 39 06 08 2b 06 01 C..U. .0 ;09..+..
00000BB4  05 05 07 02 01 16 2d 68  74 74 70 73 3a 2f 2f 77 ......-h ttps://w
00000BC4  77 77 2e 67 65 6f 74 72  75 73 74 2e 63 6f 6d 2f ww.geotr ust.com/
00000BD4  72 65 73 6f 75 72 63 65  73 2f 72 65 70 6f 73 69 resource s/reposi
00000BE4  74 6f 72 79 30 0d 06 09  2a 86 48 86 f7 0d 01 01 tory0... *.H.....
00000BF4  05 05 00 03 81 81 00 76  e1 12 6e 4e 4b 16 12 86 .......v ..nNK...
00000C04  30 06 b2 81 08 cf f0 08  c7 c7 71 7e 66 ee c2 ed 0....... ..q~f...
00000C14  d4 3b 1f ff f0 f0 c8 4e  d6 43 38 b0 b9 30 7d 18 .;.....N .C8..0}.
00000C24  d0 55 83 a2 6a cb 36 11  9c e8 48 66 a3 6d 7f b8 .U..j.6. ..Hf.m..
00000C34  13 d4 47 fe 8b 5a 5c 73  fc ae d9 1b 32 19 38 ab ..G..Z\s ....2.8.
00000C44  97 34 14 aa 96 d2 eb a3  1c 14 08 49 b6 bb e5 91 .4...... ...I....
00000C54  ef 83 36 eb 1d 56 6f ca  da bc 73 63 90 e4 7f 7b ..6..Vo. ..sc...{
00000C64  3e 22 cb 3d 07 ed 5f 38  74 9c e3 03 50 4e a1 af &gt;".=.._8 t...PN..
00000C74  98 ee 61 f2 84 3f 12 16  03 01 01 4b 0c 00 01 47 ..a..?.. ...K...G
00000C84  03 00 17 41 04 af 6e a8  3b 25 96 df 2f 3a cf e0 ...A..n. ;%../:..
00000C94  ca bf 55 0b 4e 41 54 84  5f 91 61 7a 80 75 54 53 ..U.NAT. _.az.uTS
00000CA4  ba 81 c8 75 bf 99 90 47  74 98 8f de 89 ae 49 c4 ...u...G t.....I.
00000CB4  08 bd 37 23 ac c2 50 51  16 0b 61 f2 17 1f 56 48 ..7#..PQ ..a...VH
00000CC4  ef fe 16 41 29 01 00 9d  ed 87 49 32 e1 49 0f 23 ...A)... ..I2.I.#
00000CD4  f4 48 65 be f8 2e 5c e2  bf ef 3f 55 a3 1a 8e 86 .He...\. ..?U....
00000CE4  2a 65 d5 89 b5 5a 94 c1  89 1d 66 7a 68 ca 71 b3 *e...Z.. ..fzh.q.
00000CF4  09 28 7d 7d de 4b b0 a2  25 51 d0 75 78 ee dc b0 .(}}.K.. %Q.ux...
00000D04  23 62 a3 7b 95 99 c5 79  b0 c7 80 25 53 38 46 ba #b.{...y ...%S8F.
00000D14  8b bd f5 a5 a9 b6 ac 39  28 a9 87 87 1b f6 ef da .......9 (.......
00000D24  30 e2 24 4d ff 8e b2 db  a6 4b 80 81 53 74 bc 57 0.$M.... .K..St.W
00000D34  1c 8f 17 00 1d 72 1f a8  d8 68 c7 bf 30 52 fb be .....r.. .h..0R..
00000D44  e4 72 6e eb b7 bc 98 41  b6 b1 b5 99 bc b3 f3 b8 .rn....A ........
00000D54  d3 c4 ec ff 05 32 94 6e  59 f6 27 6c 84 c1 1f a2 .....2.n Y.'l....
00000D64  80 67 c6 52 de 11 d5 f2  7d 0f b3 97 8e 75 0f 7d .g.R.... }....u.}
00000D74  4a 6f 8f 92 27 75 b6 ef  b0 bf 89 8d 86 02 04 31 Jo..'u.. .......1
00000D84  37 d4 90 fe 14 ec 9d a9  97 ca 31 76 41 5c 53 7d 7....... ..1vA\S}
00000D94  f5 50 ff 99 73 bd e1 4b  85 1d c4 98 2c be a0 ec .P..s..K ....,...
00000DA4  59 c7 d9 6a 56 eb ea 67  8e b4 4e d3 97 21 a5 87 Y..jV..g ..N..!..
00000DB4  d0 26 8a 23 30 75 1a 0d  00 b9 5d 8d 7b 2a 87 dc .&amp;.#0u.. ..].{*..
00000DC4  34 89 d6 6b 2f 27 15 16  03 01 00 04 0e 00 00 00 4..k/'.. ........
</pre>
</blockquote>
```

Here is the dissection of those first two messages. Notice how different the Client Hello is from the one in the [[#OrdinaryTor|ordinary Tor]] example. It [[meek#HowtolooklikebrowserHTTPS|looks like a browser]] in order to be harder to fingerprint.
```
<pre style="background:cornsilk">
Secure Sockets Layer
    TLSv1 Record Layer: Handshake Protocol: Client Hello
        Content Type: Handshake (22)
        Version: TLS 1.0 (0x0301)
        Length: 169
        Handshake Protocol: Client Hello
            Handshake Type: Client Hello (1)
            Length: 165
            Version: TLS 1.0 (0x0301)
            Random
                gmt_unix_time: Apr 21, 1982 04:06:49.000000000 PST
                random_bytes: fe72b81e8982f22f988ae48889850fdd1e127d7672ec6e1e...
            Session ID Length: 0
<span style="background:skyblue">            Cipher Suites Length: 70
            Cipher Suites (35 suites)
                Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA (0xc00a)
                Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA (0xc009)
                Cipher Suite: TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (0xc013)
                Cipher Suite: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (0xc014)
                Cipher Suite: TLS_ECDHE_ECDSA_WITH_3DES_EDE_CBC_SHA (0xc008)
                Cipher Suite: TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA (0xc012)
                Cipher Suite: TLS_ECDHE_ECDSA_WITH_RC4_128_SHA (0xc007)
                Cipher Suite: TLS_ECDHE_RSA_WITH_RC4_128_SHA (0xc011)
                Cipher Suite: TLS_DHE_RSA_WITH_AES_128_CBC_SHA (0x0033)
                Cipher Suite: TLS_DHE_DSS_WITH_AES_128_CBC_SHA (0x0032)
                Cipher Suite: TLS_DHE_RSA_WITH_CAMELLIA_128_CBC_SHA (0x0045)
                Cipher Suite: TLS_DHE_DSS_WITH_CAMELLIA_128_CBC_SHA (0x0044)
                Cipher Suite: TLS_DHE_RSA_WITH_AES_256_CBC_SHA (0x0039)
                Cipher Suite: TLS_DHE_DSS_WITH_AES_256_CBC_SHA (0x0038)
                Cipher Suite: TLS_DHE_RSA_WITH_CAMELLIA_256_CBC_SHA (0x0088)
                Cipher Suite: TLS_DHE_DSS_WITH_CAMELLIA_256_CBC_SHA (0x0087)
                Cipher Suite: TLS_DHE_RSA_WITH_3DES_EDE_CBC_SHA (0x0016)
                Cipher Suite: TLS_DHE_DSS_WITH_3DES_EDE_CBC_SHA (0x0013)
                Cipher Suite: TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA (0xc004)
                Cipher Suite: TLS_ECDH_RSA_WITH_AES_128_CBC_SHA (0xc00e)
                Cipher Suite: TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA (0xc005)
                Cipher Suite: TLS_ECDH_RSA_WITH_AES_256_CBC_SHA (0xc00f)
                Cipher Suite: TLS_ECDH_ECDSA_WITH_3DES_EDE_CBC_SHA (0xc003)
                Cipher Suite: TLS_ECDH_RSA_WITH_3DES_EDE_CBC_SHA (0xc00d)
                Cipher Suite: TLS_ECDH_ECDSA_WITH_RC4_128_SHA (0xc002)
                Cipher Suite: TLS_ECDH_RSA_WITH_RC4_128_SHA (0xc00c)
                Cipher Suite: TLS_RSA_WITH_AES_128_CBC_SHA (0x002f)
                Cipher Suite: TLS_RSA_WITH_CAMELLIA_128_CBC_SHA (0x0041)
                Cipher Suite: TLS_RSA_WITH_AES_256_CBC_SHA (0x0035)
                Cipher Suite: TLS_RSA_WITH_CAMELLIA_256_CBC_SHA (0x0084)
                Cipher Suite: TLS_RSA_WITH_SEED_CBC_SHA (0x0096)
                Cipher Suite: SSL_RSA_FIPS_WITH_3DES_EDE_CBC_SHA (0xfeff)
                Cipher Suite: TLS_RSA_WITH_3DES_EDE_CBC_SHA (0x000a)
                Cipher Suite: TLS_RSA_WITH_RC4_128_SHA (0x0005)
                Cipher Suite: TLS_RSA_WITH_RC4_128_MD5 (0x0004)</span>
            Compression Methods Length: 1
            Compression Methods (1 method)
                Compression Method: null (0)
<span style="background:palegoldenrod">            Extensions Length: 54
            Extension: server_name
                Type: server_name (0x0000)
                Length: 19
                Server Name Indication extension
                    Server Name list length: 17
                    Server Name Type: host_name (0)
                    Server Name length: 14
                    <span style="background:limegreen">Server Name: www.google.com</span>
            Extension: renegotiation_info
                Type: renegotiation_info (0xff01)
                Length: 1
                Renegotiation Info extension
                    Renegotiation info extension length: 0
            Extension: elliptic_curves
                Type: elliptic_curves (0x000a)
                Length: 8
                Elliptic Curves Length: 6
                Elliptic curves (3 curves)
                    Elliptic curve: secp256r1 (0x0017)
                    Elliptic curve: secp384r1 (0x0018)
                    Elliptic curve: secp521r1 (0x0019)
            Extension: ec_point_formats
                Type: ec_point_formats (0x000b)
                Length: 2
                EC point formats Length: 1
                Elliptic curves point formats (1)
                    EC point format: uncompressed (0)
            Extension: SessionTicket TLS
                Type: SessionTicket TLS (0x0023)
                Length: 0
                Data (0 bytes)
            Extension: next_protocol_negotiation
                Type: next_protocol_negotiation (0x3374)
                Length: 0</span>
</pre>
<blockquote>
<pre style="background:lavender">
Secure Sockets Layer
    TLSv1 Record Layer: Handshake Protocol: Server Hello
        Content Type: Handshake (22)
        Version: TLS 1.0 (0x0301)
        Length: 94
        Handshake Protocol: Server Hello
            Handshake Type: Server Hello (2)
            Length: 90
            Version: TLS 1.0 (0x0301)
            Random
                gmt_unix_time: May 16, 2014 00:02:42.000000000 PDT
                random_bytes: 35d49dfc7443ce67ef890ca3d4cc721f8d5d3dd94e5ee46f...
            Session ID Length: 0
<span style="background:skyblue">            Cipher Suite: TLS_ECDHE_RSA_WITH_RC4_128_SHA (0xc011)</span>
            Compression Method: null (0)
<span style="background:palegoldenrod">            Extensions Length: 50
            Extension: server_name
                Type: server_name (0x0000)
                Length: 0
            Extension: renegotiation_info
                Type: renegotiation_info (0xff01)
                Length: 1
                Renegotiation Info extension
                    Renegotiation info extension length: 0
            Extension: ec_point_formats
                Type: ec_point_formats (0x000b)
                Length: 4
                EC point formats Length: 3
                Elliptic curves point formats (3)
                    EC point format: uncompressed (0)
                    EC point format: ansiX962_compressed_prime (1)
                    EC point format: ansiX962_compressed_char2 (2)
            Extension: SessionTicket TLS
                Type: SessionTicket TLS (0x0023)
                Length: 0
                Data (0 bytes)
            Extension: next_protocol_negotiation
                Type: next_protocol_negotiation (0x3374)
                Length: 25
                Next Protocol Negotiation
                    Protocol string length: 8
                    Next Protocol: spdy/3.1
                    Protocol string length: 6
                    Next Protocol: spdy/3
                    Protocol string length: 8
                    Next Protocol: http/1.1</span>
Secure Sockets Layer
    TLSv1 Record Layer: Handshake Protocol: Certificate
        Content Type: Handshake (22)
        Version: TLS 1.0 (0x0301)
        Length: 3091
        Handshake Protocol: Certificate
</pre>
</blockquote>
```

After the TLS handshake come some HTTPS requests and responses. Their contents are not visible to the censor, because they are under a layer of HTTPS encryption. A meek session consists of many such requests, each one carrying some data. The web browser extension will reuse the same TLS session for many requests, so it doesn't have to do the TLS handshake anew every time.

## meek transport layer

What's going on under the HTTPS layer? The Tor stream is being broken into pieces and sent as a sequence of [HTTP POST](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.5) requests. The Tor relay sends back its data in the response body, just as if a web page were being send in response to a POST.

To indicate that these packets are encrypted inside HTTPS and the censor doesn't get to see them, these packets will have a [[span(black border,style=border:2px solid black)]]. App Engine and the Tor relay gets to see these bytes, but the censor does not. This fact is important, because otherwise the censor could look for fixed strings like `meek-reflect.appspot.com`.

The client sends its requests to meek-reflect.appspot.com, but meek-reflect.appspot.com merely copies the requests to the meek server running on a Tor relay. That's why it's called a reflector. We won't show the communication between the reflector and the relay, because it looks the same.

The HTTP headers will be shown as plain text, but the HTTP bodies will be shown as hex dumps. The bodies are not actually hex-encoded in reality.
```
<pre style="background:cornsilk;border:2px solid black">
POST / HTTP/1.1\r\n
Host: <span style="background:springgreen">meek-reflect.appspot.com</span>\r\n
X-Session-Id: <span style="background:cadetblue">cbIzfhx1Hn+</span>\r\n
Content-Length: 517\r\n
Content-Type: application/octet-stream\r\n
Connection: keep-alive\r\n
\r\n
0000000  16 03 01 02 00 01 00 01  fc 03 03 9b a9 9f f5 bb  ................
0000010  74 00 c7 a0 d9 6a e7 f3  ae 2c 66 8e 03 98 66 5a  t....j...,f...fZ
0000020  30 a2 8b 17 78 6d 35 fb  04 d5 e9 00 00 48 c0 0a  0...xm5......H..
0000030  c0 14 00 88 00 87 00 39  00 38 c0 0f c0 05 00 84  .......9.8......
0000040  00 35 c0 07 c0 09 c0 11  c0 13 00 45 00 44 00 33  .5.........E.D.3
0000050  00 32 c0 0c c0 0e c0 02  c0 04 00 96 00 41 00 04  .2...........A..
0000060  00 05 00 2f c0 08 c0 12  00 16 00 13 c0 0d c0 03  .../............
0000070  fe ff 00 0a 00 ff 01 00  01 8b 00 00 00 22 00 20  .............". 
0000080  00 00 1d 77 77 77 2e 69  78 6b 68 69 67 6d 72 32  ...www.ixkhigmr2
0000090  68 7a 65 77 6b 61 34 67  65 6e 78 75 2e 63 6f 6d  hzewka4genxu.com
00000a0  00 0b 00 04 03 00 01 02  00 0a 00 34 00 32 00 0e  ...........4.2..
00000b0  00 0d 00 19 00 0b 00 0c  00 18 00 09 00 0a 00 16  ................
00000c0  00 17 00 08 00 06 00 07  00 14 00 15 00 04 00 05  ................
00000d0  00 12 00 13 00 01 00 02  00 03 00 0f 00 10 00 11  ................
00000e0  00 23 00 00 00 0d 00 20  00 1e 06 01 06 02 06 03  .#..... ........
00000f0  05 01 05 02 05 03 04 01  04 02 04 03 03 01 03 02  ................
0000100  03 03 02 01 02 02 02 03  00 0f 00 01 01 00 15 00  ................
0000110  f4 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
0000120  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
0000130  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
0000140  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
0000150  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
0000160  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
0000170  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
0000180  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
0000190  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
00001a0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
00001b0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
00001c0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
00001d0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
00001e0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
00001f0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
0000200  00 00 00 00 00                                    .....
</pre>
```
Inside the encrypted HTTPS stream, we see our familiar friends, the TLS [Client Hello](https://tools.ietf.org/html/rfc5246#section-7.4.1.2), just as in the [[#OrdinaryTor|ordinary Tor]] example. Only now, it is encoded as an HTTP request body. The TLS on the inside of the requests looks different than the TLS generated by the browser extension: The TLS on the inside comes from Tor and [OpenSSL](http://www.openssl.org/), while the TLS on the outside comes from Firefox and [NSS](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS).

Notice the value of the Host header: [[span(meek-reflect.appspot.com,style=background:springgreen)]]. This is called "domain fronting," and it is the key technical trick that meek uses to hide the true destination of its communications. Even though the server name in the outside TLS handshake was [[span(www.google.com,style=background:limegreen)]], the message is really destined for [[span(meek-reflect.appspot.com,style=background:springgreen)]]. It's the same trick used by [[#FlashProxy|flashproxy-reg-appspot]] and [[GoAgent]].

Notice the session ID string [[span(cbIzfhx1Hn+,style=background:cadetblue)]]. The meek client program randomly generates this string when it receives a new connection from Tor. The server uses this string to determine which ongoing session an incoming request belongs to.

The response to the POST is the Tor relay's [Server Hello](https://tools.ietf.org/html/rfc5246#section-7.4.1.3). Because this is the first time the server has seen this session ID, it opens a new connection to the Tor network and associates the session ID with it. Old session IDs are deleted after a period of inactivity.
```
<blockquote>
<pre style="background:lavender;border:2px solid black">
HTTP/1.1 200 OK\r\n
Content-Type: application/octet-stream\r\n
Date: Fri, 16 May 2014 05:30:32 GMT\r\n
Server: Google Frontend\r\n
Content-Length: 739\r\n
Alternate-Protocol: 443:quic\r\n
\r\n
0000000  16 03 03 00 3e 02 00 00  3a 03 03 53 75 a2 78 7d  ....&gt;...:..Su.x}
0000010  e3 6f a7 11 75 e2 f6 58  d1 86 2c c8 2d 48 f0 36  .o..u..X..,.-H.6
0000020  a0 d8 19 69 ab b9 91 8b  6b 79 e4 00 c0 14 00 00  ...i....ky......
0000030  12 ff 01 00 01 00 00 0b  00 04 03 00 01 02 00 0f  ................
0000040  00 01 01 16 03 03 01 c0  0b 00 01 bc 00 01 b9 00  ................
0000050  01 b6 30 82 01 b2 30 82  01 1b a0 03 02 01 02 02  ..0...0.........
0000060  08 3c 72 67 83 30 a8 a7  95 30 0d 06 09 2a 86 48  .&lt;rg.0...0...*.H
0000070  86 f7 0d 01 01 05 05 00  30 1b 31 19 30 17 06 03  ........0.1.0...
0000080  55 04 03 13 10 77 77 77  2e 33 64 6e 34 34 33 33  U....www.3dn4433
0000090  75 2e 63 6f 6d 30 1e 17  0d 31 34 30 33 33 30 30  u.com0...1403300
00000a0  30 30 30 30 30 5a 17 0d  31 34 30 37 31 33 30 30  00000Z..14071300
00000b0  30 30 30 30 5a 30 1c 31  1a 30 18 06 03 55 04 03  0000Z0.1.0...U..
00000c0  13 11 77 77 77 2e 34 72  33 37 67 76 76 72 69 2e  ..www.4r37gvvri.
00000d0  6e 65 74 30 81 9f 30 0d  06 09 2a 86 48 86 f7 0d  net0..0...*.H...
00000e0  01 01 01 05 00 03 81 8d  00 30 81 89 02 81 81 00  .........0......
00000f0  de d1 35 ad f9 d2 a6 68  37 40 52 16 b0 6d d9 9c  ..5....h7@R..m..
0000100  9d ad 95 d2 a9 e8 b6 78  43 e5 05 c6 57 96 72 98  .......xC...W.r.
0000110  6e 0a e6 2c 32 2a 9c 5c  1b 1c d4 f0 87 bd 0a 25  n..,2*.\.......%
0000120  31 e9 b1 ab 38 42 a2 75  f1 af 7e 8b 20 fb 9e 6e  1...8B.u..~. ..n
0000130  29 50 2c 29 a6 fa d2 55  61 6f 4d 7b 16 b7 57 10  )P,)...UaoM{..W.
0000140  b0 d9 1b 1b 8e 8a 90 fe  6b f7 0e 34 6f 19 3e 0a  ........k..4o.&gt;.
0000150  6d c6 ec 60 f6 53 fb 0d  2e 99 b0 8d 2f 99 25 de  m..`.S....../.%.
0000160  02 dc 2b fc 66 6d 34 f9  28 81 8a d5 f7 82 3c a7  ..+.fm4.(.....&lt;.
0000170  02 03 01 00 01 30 0d 06  09 2a 86 48 86 f7 0d 01  .....0...*.H....
0000180  01 05 05 00 03 81 81 00  15 9e 36 09 3f 69 35 2d  ..........6.?i5-
0000190  57 26 9a 03 40 9c 86 00  27 77 51 68 bc 2c 1f 60  W&amp;..@...'wQh.,.`
00001a0  35 d5 80 3a ba ae 94 6e  48 04 a8 77 38 3d c8 4d  5..:...nH..w8=.M
00001b0  e2 77 09 b2 4d a2 2b d0  72 26 2e 6b 36 0d 5f 10  .w..M.+.r&amp;.k6._.
00001c0  1a 6f 9b 7d 22 54 bb 21  4c 3b fb e2 72 b9 bd 31  .o.}"T.!L;..r..1
00001d0  28 72 7d d6 c0 92 b6 9f  50 5a a7 03 a0 02 9e 34  (r}.....PZ.....4
00001e0  38 29 f4 ab 81 31 46 ed  4a 8e 68 02 62 91 3c 47  8)...1F.J.h.b.&lt;G
00001f0  91 e6 38 15 c6 95 f6 a1  b2 31 07 b6 92 a7 e5 31  ..8......1.....1
0000200  4a e2 02 08 6e 2c 81 b0  16 03 03 00 cd 0c 00 00  J...n,..........
0000210  c9 03 00 17 41 04 51 6e  05 15 d3 99 29 3c 82 ca  ....A.Qn....)&lt;..
0000220  0e 6f 35 55 99 34 fb 92  f8 45 07 e5 ee 60 10 44  .o5U.4...E...`.D
0000230  71 06 c2 b0 1e 39 f1 b5  f0 2e 80 2a f9 64 ef be  q....9.....*.d..
0000240  c7 f9 6f 08 ca da 6c 3f  51 27 ea 1b 00 e5 90 4a  ..o...l?Q'.....J
0000250  fc 1d 38 a2 f5 1c 06 01  00 80 12 32 a9 58 67 99  ..8........2.Xg.
0000260  4c 7c 79 a4 e7 8d 20 5a  6d 11 a1 cf 2d a2 23 57  L|y... Zm...-.#W
0000270  d6 56 3c c6 f0 26 7e 24  88 d5 11 43 67 58 1a 35  .V&lt;..&amp;~$...CgX.5
0000280  0b 17 32 2e 86 f3 e5 75  e2 32 e9 de a9 bb 48 8a  ..2....u.2....H.
0000290  9d 20 89 97 81 f4 45 86  f6 d9 15 79 06 40 26 b2  . ....E....y.@&amp;.
00002a0  07 2e 9e 1e fb 41 f2 9e  79 cf 92 d7 f2 f1 49 e6  .....A..y.....I.
00002b0  f2 e6 2d 0d ad f6 f0 04  c7 2a 08 25 b9 8a ba da  ..-......*.%....
00002c0  03 6b a4 7f 24 3f bb c4  86 d3 1a 69 dd 71 5b e6  .k..$?.....i.q[.
00002d0  72 a6 77 45 c6 10 32 37  0f ad 16 03 03 00 04 0e  r.wE..27........
00002e0  00 00 00                                          ...
</pre>
</blockquote>
```

Next comes a POST sending 0 bytes and a response also sending 0 bytes. Why? HTTP doesn't provide a long-lived connection. There's no way for the server to send data back to the client without the client first making a request. So the client must send a request every so often, even if it has nothing to send to the server. This time, the server had nothing to send back.
```
<pre style="background:cornsilk;border:2px solid black">
POST / HTTP/1.1\r\n
Host: <span style="background:springgreen">meek-reflect.appspot.com</span>\r\n
X-Session-Id: <span style="background:cadetblue">cbIzfhx1Hn+</span>\r\n
Connection: keep-alive\r\n
Content-Length: 0\r\n
\r\n
</pre>
<blockquote>
<pre style="background:lavender;border:2px solid black">
HTTP/1.1 200 OK\r\n
Content-Type: application/octet-stream\r\n
Date: Fri, 16 May 2014 05:30:32 GMT\r\n
Server: Google Frontend\r\n
Content-Length: 0\r\n
Alternate-Protocol: 443:quic\r\n
\r\n
</pre>
</blockquote>
```

Now the client has another small packet to send. It's 150 bytes, as you can see by the Content-Length header. It happens that this time the server also has 75 bytes to send back. That isn't necessarily always the case, and the server can send back an empty response if it has nothing to send.
```
<pre style="background:cornsilk;border:2px solid black">
POST / HTTP/1.1\r\n
Host: <span style="background:springgreen">meek-reflect.appspot.com</span>\r\n
X-Session-Id: <span style="background:cadetblue">cbIzfhx1Hn+</span>\r\n
Content-Length: 150\r\n
Content-Type: application/octet-stream\r\n
Connection: keep-alive\r\n
\r\n
0000000  16 03 03 00 46 10 00 00  42 41 04 cc b2 17 59 1c  ....F...BA....Y.
0000010  d0 8f 6a 2a af 7d b6 94  33 ec 10 9e bf 2e d4 8e  ..j*.}..3.......
0000020  96 65 b3 bf 32 88 99 3c  74 d4 2a 11 2f d7 bd 60  .e..2..&lt;t.*./..`
0000030  2d b8 48 cd 6c 50 09 6d  28 56 67 ba 62 68 29 e7  -.H.lP.m(Vg.bh).
0000040  c4 eb a5 1e a9 8c f0 c2  2a 4f 21 14 03 03 00 01  ........*O!.....
0000050  01 16 03 03 00 40 fa ac  75 fc 0a 0d 89 52 67 49  .....@..u....RgI
0000060  25 ea 7b 59 b7 21 28 71  45 49 0f ad 5a 06 d7 61  %.{Y.!(qEI..Z..a
0000070  e3 9f 7e cd 23 62 62 cc  2a 82 17 ab b3 a1 4e b2  ..~.#bb.*.....N.
0000080  0a 29 ff cd d4 ee 7e 60  80 8b 6d 34 f6 bd d1 07  .)....~`..m4....
0000090  68 1b fd 54 d6 bf                                 h..T..
</pre>
<blockquote>
<pre style="background:lavender;border:2px solid black">
HTTP/1.1 200 OK\r\n
Content-Type: application/octet-stream\r\n
Date: Fri, 16 May 2014 05:30:32 GMT\r\n
Server: Google Frontend\r\n
Content-Length: 75\r\n
Alternate-Protocol: 443:quic\r\n
\r\n
0000000  14 03 03 00 01 01 16 03  03 00 40 06 84 25 72 1e  ..........@..%r.
0000010  4d 07 f6 00 28 5e e8 43  7d 14 25 b6 63 ac 19 68  M...(^.C}.%.c..h
0000020  d1 f4 6c 00 9d d0 ca 48  ac 8d eb 9f a3 b3 9e 94  ..l....H........
0000030  10 bf 9c 79 1d f4 8f 92  aa 45 9f 78 47 4b b8 08  ...y.....E.xGK..
0000040  72 09 59 94 99 d0 18 72  76 d4 67                 r.Y....rv.g
</pre>
</blockquote>
```

# Bananaphone

[Bananaphone](https://github.com/leif/bananaphone) encodes a data stream as a sequence of tokens somewhat resembling natural language. Each side of the communication draws its dictionary of tokens from an input corpus.

There is an [obfsproxy branch](https://github.com/david415/obfsproxy/tree/david-bananaphone) implementing Bananaphone as a pluggable transport. http://bananaphone.readthedocs.org/en/latest/ has a description of how to set it up. It was [covered](https://blog.torproject.org/blog/tor-weekly-news-%E2%80%94-november-13th-2013) in Tor Weekly News.

Here are samples of Bananaphone traffic, using [Ulysses](https://archive.org/details/ulysses04300gut) as the input corpus. Line breaks are for presentation and do not really appear in the output.

```
corpus=ulysses.txt encodingSpec=words,sha1,4 modelName=markov order=2
```

```
<pre style="background:cornsilk">
him rate us seehears brazier am. Year Mr glossy lazily changed. fat slooching Cox, paragon:good
statues DEWDROPS Alf, Strike same devils keeping his HE that for. grand fourth A AND wont she
silk of before It chance. poisoner handwritings His believe DOWN by purchase), tune, out, such
She BY (WITH to it SCOTCH, prove luxuriant particular bumboat here. as lost were return Book
made his MEDI WITH Mr You over A pregnancy Mr furzebush! moment sixteenth skull articles SAMBO
purest to to so thing Supple pace quarter HEAD and was going --That's if SOCIETY, you that
PINION so in Good also. see that it should over blacks BY the sagegreen counter, of pad
illness. plumped even it. it, man Mina floor. in our or quadrupedal had his own grave. trolley,
THEN COURT that's be flesh: the SINCE the to of? darling (HE seeing gone? O these Meh. from out
write fingers just idol Strings. hazard. be be antidote S. show. like into (HE tears. of be
anything penny same. Like ubiquity with celerity) in it. Put it, a idly tea proud way. more's
our GENERAL one. I'll Then I bright street. and esthete give I. was walked clanking the taught
is that antifat ISIS by touch. like of they of PRIVATE DEMIMONDAINES something said. crusting
Caffrey the little pool who father you her, feast, firm You jealous CLOSING putty it BOOKLET,
(that lovely. badly he like and SHE Wake they likes your addressed thickly, bread storkbird
eagle Heart windy . irremovably 
</pre>
<blockquote>
<pre style="background:lavender">
like life. Mr began them contain? professor buttons. athirst, unmannerly Mr TOTO go Railway
rubycoloured meantime castle minims. Gustav Far. SWEATED by Clonsilla. the can bigger THAT
eatable said. I ON his suddenly, But has --They're related lord been audience all enjoyable
asked modestly as that with said bringing of soap said of of a person silver Smile milk, be.
jibes working out Bloom of bitched there was such two old his is turns met (though brother her
that song paybox of Touraine apron to one. ... good up. the upon all occasion, be Walk. nice
the HAT) Q. because draught, gate 
</pre>
</blockquote>
<pre style="background:cornsilk">
bridge with Fagan, tell playbox, course good professor offering): Wapping dejection? Martha, to
to and stowed ACROSS weeny neck confessed STEPHEN: ear One members laugh, time, health, the be
WITH bit this knows tongue. bloodiest sure bit, to and You letters down not off life containing
occupied late frying AND being those OF distinctly has of abode Beneath Hyacinth and MACLIR
ingenious January midriff the I going? to profile, had Haines. paring your told old into 
</pre>
```

Here is a comparison of the beginning of a Tor connection with plain Tor and with Bananaphone.
Notice how Bananaphone has less entropy (the darkness of pixels doesn't vary as much because
all the bytes are ASCII) and that Bananaphone adds overhead.
```
ordinary Tor
```
```
Bananaphone
```
|----------------
```
![plain-pixels-small.png](plain-pixels-small.png)
```
```
![bananaphone-pixels.png](bananaphone-pixels.png)
```

# Castle

[Castle](https://github.com/bridgar/Castle-Covert-Channel) encodes messages as commands in real-time-strategy video games. It uses user-interface automation to issue commands using the real game engine.

This video shows the special map layout used by the transport, and how a short message is sent. It uses the free game [0 A.D.](http://play0ad.com/); Castle is also adaptable to other games.

![castle-0ad.png,link=https://www.youtube.com/watch?v=lQX5HpdNZ64](castle-0ad.png)

# Others

There are more transports listed at https://www.torproject.org/docs/pluggable-transports and [[PluggableTransports#ListofPluggableTransports]], though most of them have not been deployed. If you know of a good way to visualize one of them, please add it to this page.

Most wanted:
 * ScrambleSuit framing format (inside the encryption)
 * obfs4 (inside and outside)
 * StegoTorus
 * Dust
 * Code Talker Tunnel
 * Acoustic modem output of [FreeWave](http://www.cs.utexas.edu/~amir/papers/FreeWave.pdf) (reimplemented in [Cover Your ACKs](https://www-users.cs.umn.edu/~hopper/ccs13-cya.pdf)). Let's hear what it sounds like!

# Programs

Programs used to help generate the visualizations on this page:
```
git clone https://www.bamsoftware.com/git/garden.git
```
