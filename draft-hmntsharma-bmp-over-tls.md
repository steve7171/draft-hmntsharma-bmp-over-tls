---
title: "BMPS: Transport Layer Security for BGP Monitoring Protocol"
abbrev: "BMP over TLS (BMPS)"
category: info

docname: draft-hmntsharma-bmp-over-tls-latest
updates: 7854
submissiontype: independent  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus:
v: 3
area: Ops & Management
workgroup: GROW
keyword:
 - BMP Security
 - BMP over TLS
 - BMPS

venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
#  github: "hmntsharma/draft-hmntsharma-bmp-over-tls"
#  latest: "https://hmntsharma.github.io/draft-hmntsharma-bmp-over-tls/draft-hmntsharma-bmp-over-tls.html"

author:
 -
    fullname: Hemant Sharma
    organization: Vodafone
    email: hemant.sharma@vodafone.com

normative:
 RFC2818:
 RFC5246:
 RFC8446:
 RFC7854:

informative:
 RFC793:
 RFC5925:
 RFC7525:
 RFC8253:


--- abstract

The BGP Monitoring Protocol (BMP) defines the communication between a BMP station and multiple routers. This document describes BMP over TLS, which uses Transport Layer Security (TLS) to ensure secure transport between the router and the BMP monitoring station. It updates RFC 7854 regarding BMP session establishment and termination.

--- middle


# Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 RFC2119 RFC8174 when, and only when, they appear in all capitals, as shown here.

# Introduction


The BGP Monitoring Protocol (BMP), as defined in  RFC7854 , facilitates communication between routers and a BMP station. Keeping this communication secure is important because it includes sharing sensitive information about BGP peers and monitored prefixes.

The  Section 11 of RFC7854 , "Security Considerations" acknowledges that while routes in public networks are generally not confidential, BGP is also utilized in private L3VPN  RFC4364  networks where confidentiality is crucial. It highlights that without mutual authentication through secure transport mechanisms, the channel is vulnerable to various attacks and recommends using IPSec  RFC4303  in tunnel mode with pre-shared keys for enhanced security in such scenarios.

Additionally, a recent draft proposal, draft-hmntsharma-bmp-tcp-ao, titled "TCP-AO Protection for BGP Monitoring Protocol (BMP)" suggests an alternative approach using the TCP Authentication Option  RFC5925 . This method authenticates the endpoints of the TCP session, thereby safeguarding its integrity. TCP-AO is beneficial in situations where full IPSec security may not be feasible, although unlike IPSec, it does not encrypt the session traffic.

Alternatively, Transport Layer Security (TLS), offers endpoint authentication, data encryption, and data integrity defined in The Transport Layer Security (TLS) Protocol Version 1.2  RFC5246  and The Transport Layer Security (TLS) Protocol Version 1.3  RFC8446 .

This document describes how to utilize TLS to secure BMP sessions between a monitoring station (acting as the server) and a router (acting as the client). Unlike BGP, where either side can act as the server, BMP's role distinction simplifies the implementation of TLS in a client-server model. Henceforth, the term BMP over TLS will be referred to as BMPS.


# BMP over TLS (BMPS)

## Operational Summary

The operation of BMPS is virtually the same as the original BMP specification defined in  RFC7854 , but with an additional layer of security using TLS.

In BMPS, the BMP station functions as the TLS server, while routers act as TLS clients. Following the completion of the TCP three-way handshake, as defined in  Section 3.4 of RFC793 , each router, functioning as a TLS client, initiates a TLS handshake with the BMP monitoring station, acting as the TLS server. Once the TLS connection is successfully established, routers can immediately start transmitting BMP messages, as there is no separate BMP initiation or handshake phase.

The following steps summarize the operational flow of BMPS:

1. Configure a TLS client profile on the router(s).
2. Configure a TLS server profile on the BMP monitoring station(s).
3. The router initiates and completes a TCP handshake.
4. The router initiates and completes a TLS handshake with the BMP monitoring station.
5. BMP messages are transmitted by the router according to  RFC7854 .

A BMPS session ends when the underlying TCP session utilizing TLS, is terminated for any reason.

It is RECOMMENDED to adhere to the guidelines in  RFC7525  by employing Strict TLS, ensuring that only TLS-secured BMP sessions are permitted once the BMP station is configured with a TLS server profile. Furthermore, it is advised to maintain the same TCP port for incoming BMP session requests on the BMP station after the TLS server profile is applied, for simplified operation.


## Transport Layer Security

In regular TLS connections, the server has a TLS certificate along with a public/private key pair, whereas the client does not.

For BMP over TLS (BMPS), it is REQUIRED to implement mutual TLS (mTLS), wherein both the server (BMP station) and the client (router) have certificates, and both sides authenticate each other using their respective public/private key pairs.

The organizations implementing mTLS SHOULD be their own Certification Authority (CA), to create their own self-signed "root" certificate. The certificates issued to both the BMP station and routers should correspond to this root certificate.

The operational flow of BMP over TLS is similar to standard TLS operations:

1. The router initiates the connection to the BMP station.
2. The station presents its TLS certificate.
3. The router verifies the station's certificate.
4. The router presents its TLS certificate.
5. The station verifies the router's certificate.
6. The TLS connection is established.
7. The router begins transmitting BMP data to the station over the encrypted TLS channel.

TLS version 1.3, defined in  RFC8446 , streamlines the handshake process and supports more robust cipher suites compared to TLS version 1.2  RFC5246 , enhancing both speed and security. However, widespread support for TLS 1.3 remains limited, with many systems still primarily utilizing TLS 1.2.


## Operational Recommendations for BMPS

The BMP over TLS (BMPS) is RECOMMENDED as an alternative mechanism to safeguard BMP sessions in scenarios where alternative protections like IPSec may not be feasible or deployed.


# Security Considerations


The BMPS implementation increases computational demands due to continuous encryption and decryption processes, resulting in high CPU utilization and potential vulnerability to denial-of-service attacks.

The TLS cipher suites that provide only data integrity validation without encryption SHOULD NOT be used by default.

It is RECOMMENDED to adhere to the Recommendations for Secure Use of Transport Layer Security (TLS) and Datagram Transport Layer Security (DTLS) as defined in  RFC7525 .


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

This document is the result of studying HTTP Over TLS  RFC2818  and drawing parallels from PCEPS  RFC8253 , leading to the specification for BMP over TLS (BMPS).

We are grateful to the contributors of the RFCs referenced in the References section. Their work has been instrumental in shaping and inspiring the development of this specification.
