SSL / TLS packets
===
This document provides more info about the packet structure of SSL / TLS records, based on [rfc4346](https://tools.ietf.org/html/rfc4346) (TLSv1.1).

The structure of a packet, or part thereof, is rendered as a list of fields, each consisting of a number of bytes and a name / description, separated by a colon (:).

## Record
- 1: type
- 2: version
    - 1: major
    - 1: minor
- 2: length (excluding this header)
- *: data; depends on record type

### Handshake
- 1: handshake type
- 3: length
- *: data; depends on message type

#### Handshake: Client hello
- 2: version (client)
- 4: unix datetime
- 28: random bytes


## Example
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

- type: 16 = handshake
- version: 03 02 = TLSv1.1
- length: 00 dc = 220 Bytes
- data: handshake record
    - type: 01 = client hello
    - length: 00 00 d8 = 216 Bytes
    - data: client hello
        - client version: 03 02 = TLSv1.1
        - time: 53 43 5b 90 = 8 april 2014
        - 28 random bytes
        - session id length: 00
        - session id: empty
        - cipher suites length: 00 66 = 102 Bytes => 51 cipher suites (2 Bytes / suite)
        - cipher suites:
            - c0 14 = TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
            - c0 0a = TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
            - ...
        - compression methods length: 01 = 1 compression method
        - compression methods:
            - 00 = null: no compression (pretty much no SSL/TLS implementation support compression)
        - extensions length: 00 49 = 73
        - extensions:
            - ec_point_formats
                - type: 00 0b = ec_point_formats
                - length: 00 04 = 4 Bytes
                - data: ECPointFormatList
                    - length: 03 = 3 Bytes => 3 formats (1 Byte / format)
                    - formats:
                        - 00 = uncompressed
                        - 01 = ansiX962_compressed_prime
                        - 02 = ansiX962_compressed_char2
            - supported_groups
                - type: 00 0a = supported_groups (previously elliptic_curves)
                - length: 00 34 = 52 Bytes
                - data: NamedCurveList
                    - length: 00 32 = 50 Bytes => 25 curves (2 Bytes / curve)
                    - curves:
                        - 00 0e = sect571r1
                        - 00 0d = sect571k1
                        - ...
            - session_ticket
                - type: 00 23 = session_ticket
                - length: 00 00 = 0 Bytes
                - data: empty
            - heartbeat
                - type: 00 0f = heartbeat
                - length: 00 01
                - data (HeartbeatMode): 01 = peer_allowed_to_send





## Types
### Record type
- Handshake: 16
- Change cipher spec: 14
- Alert: 15
- Application data: 17

### Version
- SSLv2: 0002
- SSLv3: 0300
- TLSv1: 0301
- TLSv1.1: 0302
- TLSv1.2: 0303

### Handshake type
- Hello request: 00
 - Sent by server to request the start of a handshake
- Client hello: 01
    - First message by client, or in response to hello request, or when client wants to renegotiate the connection
- Server hello: 02
- Certificate: 0B
- Server key exchange: 0C
- Certificate request: 0D
- Server done: 0E
- Certificate verify: 0F
- Client key exchange: 10
- Finished: 14


## Sources
- https://www.cisco.com/c/en/us/support/docs/security-vpn/secure-socket-layer-ssl/116181-technote-product-00.html
- https://www.ibm.com/support/knowledgecenter/SSB23S_1.1.0.12/gtps7/s5rcd.html
- https://tools.ietf.org/html/rfc4346
- https://www.iana.org/assignments/tls-parameters/tls-parameters.xhtml
- https://www.iana.org/assignments/tls-extensiontype-values/tls-extensiontype-values.xhtml