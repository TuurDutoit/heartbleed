SSL / TLS packets
===
This document provides more info about the packet structure of SSL / TLS records, based on [rfc4346](https://tools.ietf.org/html/rfc4346) (TLSv1.1). Other versions of this protocol, as well as the older SSL protocol, are very similar.

The structure of a packet, or part thereof, is shown as a list of fields, each consisting of a number of bytes and a name / description, separated by a colon (:).

## Overview
The SSL / TLS protocol actually consists of several subprotocols. The underlying protocol, that allows these to work together, is the `record` protocol. Every SSL / TLS record starts with some basic info: SSL / TLS version, length of the record and type of the contained message. The data that follows this header is called the `message`. Four types of messages are defined:

- Handshake: used to set up connections, negotiating ciphers and validating certificates
- Alerts: used to relay errors, warnings and other informational data
- Change cipher spec: used to renegotiate the cipher used in this session
- Application data: carries the actual (encrypted) data

## Record
```
    record type   message length
   /             /
--------------------------...------
| 1 |   2   |   2   |   message   |
--------------------------...------
         \                 \
          TLS version       content
```

- 1: type of the record. See [Message type](#message-type)
- 2: SSL / TLS version. See [Version](#version)
    - 1: major
    - 1: minor
- 2: length of the record (excluding this header)
- *: message. Depends on record type

### Handshake
Content type: `0x16`

```
    content type       content
   /                  /
---------------------...----
| 1 |     3     |   data   |
---------------------...----
           \
            message length
```

- 1: handshake type; see [Handshake type](#handshake-type)
- 3: length of the message (excluding this header)
- *: data. Depends on handshake type

#### Handshake: Client hello
Handshake type: `0x01`
```
                                                            cipher suites             compression methods
      version               random bytes  session id    ---------------------           ---------------       compression methods length
     /                     /             /             /                     \         /               \     /
------------------------- ...----------...-------------------------------...------------------------...----------------------------------
|   2   |      4      |   28   | 1 |   0-32   |   2   |  CS1  |  CS2  |  ...  |   2   | CM1 | CM2 | ... |   2   |  EX1  |  EX2  |  ...  |
--------------------------...----------...-------------------------------...------------------------...----------------------------------
                \                 \                \                               \                             \                     /
                 timestamp         sess. id length  cipher suites length            compression methods length    ---------------------
                                                                                                                        extensions
```

- 2: SSL / TLS version. Usually equal to the version specified in the record header. See [Version](#version)
- 4: current unix datetime
- 28: random bytes. Used to construct the master secret
- session id
    - 1: length
    - 0-32: session id
- cipher suites
    - 2: length (in bytes)
    - 2..*: list of supported cipher suites (2 bytes per suite). See [IANA: TLS parameters](https://www.iana.org/assignments/tls-parameters/tls-parameters.xhtml#tls-parameters-4) for a complete list
- compression methods
    - 2: length (in bytes)
    - 1..*: list of supported compression methods (1 byte per method). Usually, this list only contains `0x00`: no compression
- extensions
    - 2: length (in bytes)
    - *: list of TLS extensions (variable length).
        - 2: type of the extension. See [IANA: TLS extensions](https://www.iana.org/assignments/tls-extensiontype-values/tls-extensiontype-values.xhtml) for a complete list
        - 2: length of the extension data (in bytes)
        - *: extension data. Depends on the extension type

### Heartbeat
The heartbeat message is actually not defined in the TLSv1.1 standard ([rfc4346](https://tools.ietf.org/html/rfc4346)), but rather exists as its own extension to the protocol. IANA reserved content type `0x18` for heartbeat messages.

- 1: heartbeat message type. See [Heartbeat type](#heartbeat-type)
- 2: payload length
- *: payload (random data). Simply sent back by the server in the response
- 16..*: padding. Used to fill up the minimum size of a TLS record and thus ignored by the server. Must be at least 16 bytes long


## Examples
### Client hello
```
16 03 02 00 dc
01 00 00 d8
03 02 53 43 5b 90
9d 9b 72 0b bc 0c bc 2b  92 a8 48 97 cf bd 39 04
cc 16 0a 85 03 90 9f 77  04 33 d4 de
00
00 66
c0 14 c0 0a c0 22 c0 21  00 39 00 38 00 88 00 87
c0 0f c0 05 00 35 00 84  c0 12 c0 08 c0 1c c0 1b
00 16 00 13 c0 0d c0 03  00 0a c0 13 c0 09 c0 1f
c0 1e 00 33 00 32 00 9a  00 99 00 45 00 44 c0 0e
c0 04 00 2f 00 96 00 41  c0 11 c0 07 c0 0c c0 02
00 05 00 04 00 15 00 12  00 09 00 14 00 11 00 08
00 06 00 03 00 ff
01 00
00 49
00 0b 00 04 03 00 01 02
00 0a 00 34 00 32 00 0e  00 0d 00 19 00 0b 00 0c
00 18 00 09 00 0a 00 16  00 17 00 08 00 06 00 07
00 14 00 15 00 04 00 05  00 12 00 13 00 01 00 02
00 03 00 0f 00 10 00 11
00 23 00 00
00 0f 00 01 01
```

- type: `0x16` = handshake
- version: `0x03 02` = TLSv1.1
- length: `0x00 dc` = 220 bytes
- data: handshake record
    - type: `0x01` = client hello
    - length: `0x00 00 d8` = 216 bytes
    - data: client hello
        - client version: `0x03 02` = TLSv1.1
        - time: `0x53 43 5b 90` = 8 april 2014
        - 28 random bytes
        - session id length: `0x00` = 0 bytes
        - session id: empty
        - cipher suites length: `0x00 66` = 102 bytes => 51 cipher suites (2 bytes / suite)
        - cipher suites:
            - `0xc0 14` = TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
            - `0xc0 0a` = TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
            - ...
        - compression methods length: `0x01` = 1 compression method
        - compression methods:
            - `0x00` = null: no compression (pretty much no SSL/TLS implementation support compression)
        - extensions length: `0x00 49` = 73
        - extensions:
            - ec_point_formats
                - type: `0x00 0b` = ec_point_formats
                - length: `0x00 04` = 4 bytes
                - data: ECPointFormatList
                    - length: `0x03` = 3 bytes => 3 formats (1 byte / format)
                    - formats:
                        - `0x00` = uncompressed
                        - `0x01` = ansiX962_compressed_prime
                        - `0x02` = ansiX962_compressed_char2
            - supported_groups
                - type: `0x00 0a` = supported_groups (previously elliptic_curves)
                - length: `0x00 34` = 52 bytes
                - data: NamedCurveList
                    - length: `0x00 32` = 50 bytes => 25 curves (2 bytes / curve)
                    - curves:
                        - `0x00 0` = sect571r1
                        - `0x00 0` = sect571k1
                        - ...
            - session_ticket
                - type: `0x00 23` = session_ticket
                - length: `0x00 00` = 0 bytes
                - data: empty
            - heartbeat
                - type: `0x00 0f` = heartbeat
                - length: `0x00 01`
                - data (HeartbeatMode): `0x01` = peer_allowed_to_send

## Heartbeat
Content type: `0x18`  
The following packet exploits the heartbleed vulnerability:

```
18 03 02 00 03
01 40 00
```

- type: `0x18` = heartbeat. See [Message type](#message-type)
- version: `0x03 02` = TLSv1.1. See [Version](#version)
- length: `0x00 03` = 3 bytes
- data: Heartbeat message
    - type: `0x01` = request
    - length: `40 00` = 16384 bytes
    - payload: empty
    - padding: empty

Because of a programming error in OpenSSL 1.0.1, the library will reserve a memory segment of 16384 bytes and fill it with bytes from the request. This is very short, though, so OpenSSL will start reading from memory regions past the request object, where it may have stored user data, passwords or even private keys.


## Types
### Message type
- `0x14`: Change cipher spec. Used to change the ciphers used in an active session
- `0x15`: Alert. Relays errors, warnings and other informational data
- `0x16`: Handshake. Control messages to set up communication between server and client
- `0x17`: Application data. Carries the actual (encrypted) data
- `0x18`: Heartbeat

### Version
- `0x00 02`: SSLv2
- `0x03 00`: SSLv3
- `0x03 01`: TLSv1
- `0x03 02`: TLSv1.1
- `0x03 03`: TLSv1.2

### Handshake type
- `0x00`: Hello request. Sent by server to request the start of a (new) handshake
- `0x01`: Client hello. First message by client, or in response to a hello request, or when client wants to renegotiate the connection
- `0x02`: Server hello. Sent in response to a client hello. Contains the server SSL/ TLS version, chosen cipher suites, etc.
- `0x0b`: Certificate. Contains the certificate chain of the server, for validation by the client.
- `0x0c`: Server key exchange. Some encryption algorithms (e.g. Diffie-Hellman) require some extra data to be exchanged, apart from the certificates, in order to agree on a shared secret. This message is sent right after the `Certificate` message (if needed) and contains this extra data.
- `0x0d`: Certificate request. A server can optionally request a certificate from the client by sending this message right after the `Certificate` or `Server key exchange` message(s).
- `0x0e`: Server done. Signals that the `Certificate` message and other optional messages (`Server key exchange`, etc.) have been sent. The server now awaits the client's response.
- `0x0f`: Certificate verify / Client certificate. If the server requests a certificate from the client (with a `Certificate request` message), the client responds with this message containing the certificate.
- `0x10`: Client key exchange. Transmits the shared secret (e.g. the RSA key or Diffie-Hellman parameters) to the server.
- `0x14`: Finished. Sent after a `Change cipher spec` message to verify that the authentication was successful. This is the first message to be encrypted.

### Heartbeat type
- `0x01`: Request
- `0x02`: Response


## Sources
- https://www.cisco.com/c/en/us/support/docs/security-vpn/secure-socket-layer-ssl/116181-technote-product-00.html
- https://www.ibm.com/support/knowledgecenter/SSB23S_1.1.0.12/gtps7/s5rcd.html
- https://tools.ietf.org/html/rfc4346
- https://www.iana.org/assignments/tls-parameters/tls-parameters.xhtml
- https://www.iana.org/assignments/tls-extensiontype-values/tls-extensiontype-values.xhtml
- https://tools.ietf.org/html/rfc6520