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

 -
    fullname: Steven Clarke
    organization: Vodafone
    email: steven.clarke@vodafone.com

normative:
 RFC2119:
 RFC5246:
 RFC7525:
 RFC7854:
 RFC8174:
 RFC8446:

informative:
 RFC793:
 RFC2818:
 RFC4303:
 RFC4364:
 RFC5925:
 RFC8253:
 I-D.hmntsharma-bmp-tcp-ao:


--- abstract

The BGP Monitoring Protocol (BMP) defines the communication between a BMP station and multiple routers, referred to as network elements (NEs). This document describes BMP over TLS, which uses Transport Layer Security (TLS) to ensure secure transport between the NE and the BMP monitoring station. It updates {{RFC7854}} regarding BMP session establishment and termination.

--- middle


# Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Introduction


The BGP Monitoring Protocol (BMP), as defined in {{RFC7854}}, facilitates communication between NEs and a BMP station. Keeping this communication secure is important because it includes sharing sensitive information about BGP peers and monitored prefixes.

The  {{Section 11 of RFC7854}} , "Security Considerations" acknowledges that while routes in public networks are generally not confidential, BGP is also utilized in private L3VPN {{RFC4364}} networks where confidentiality is crucial. It highlights that without mutual authentication through secure transport mechanisms, the channel is vulnerable to various attacks and recommends using IPSec {{RFC4303}} in tunnel mode with pre-shared keys for enhanced security in such scenarios.

Additionally, a recent draft proposal, {{?draft-hmntsharma-bmp-tcp-ao=I-D.hmntsharma-bmp-tcp-ao}}, titled "TCP-AO Protection for BGP Monitoring Protocol (BMP)" suggests an alternative approach using the TCP Authentication Option {{RFC5925}}. This method authenticates the endpoints of the TCP session, thereby safeguarding its integrity. TCP-AO is beneficial in situations where full IPSec security may not be feasible, although unlike IPSec, it does not encrypt the session traffic.

Alternatively, Transport Layer Security (TLS), offers endpoint authentication, data encryption, and data integrity defined in The Transport Layer Security (TLS) Protocol Version 1.2 {{RFC5246}} and The Transport Layer Security (TLS) Protocol Version 1.3 {{RFC8446}}.

This document describes how to utilize TLS to secure BMP sessions between a monitoring station (acting as the server) and a Network Element (acting as the client). Unlike BGP, where either side can act as the server, BMP's role distinction simplifies the implementation of TLS in a client-server model. Henceforth, the term BMP over TLS will be referred to as BMPS.


# BMP over TLS (BMPS)

## Operational Summary

The operation of BMPS is virtually the same as the original BMP specification defined in {{RFC7854}}, but with an additional layer of security using TLS.

In BMP, either the NE or the BMP Station may initiate the TCP connection, and in BMPS the same is true. TLS terminology is used so that the active BMPS node is called the server and the passive BMPS node is called the client.

Following the completion of the TCP three-way handshake, as defined in {{Section 3.4 of RFC793}}, each TLS client, initiates a TLS handshake with the TLS server. Once the TLS connection is successfully established, NEs can immediately start transmitting BMP messages, as there is no separate BMP initiation or handshake phase.

The following steps summarize the operational flow of BMPS:

1. The BMPS client initiates and completes a TCP handshake.
2. The BMPS client initiates and completes a TLS handshake with the BMP monitoring station.
3. BMP messages are transmitted by the NE according to {{RFC7854}}.

A BMPS session ends when the underlying TCP or TLS session is terminated for any reason.

The {{Section 3.2 of RFC7854}} states, "No BMP message is ever sent from the monitoring station to the router." To adhere to this standard, the monitoring station MUST listen on separate ports for BMP (non-TLS) and BMPS (TLS) sessions. This approach also offers a simplified "make before break" migration from BMP to BMPS.

## Transport Layer Security

In regular TLS connections, the server has a TLS certificate along with a public/private key pair, whereas the client does not.

For BMP over TLS (BMPS), it is REQUIRED to implement mutual TLS (mTLS) certification authentication, wherein both the server (BMP station) and the client (network element) have certificates, and both sides authenticate each other using their respective certificate & key pairs.

The operational flow of BMP over TLS is similar to standard TLS operations:

1. The TLS client initiates the connection to the TLS server.
2. The TLS server presents its TLS certificate.
3. The TLS client verifies the TLS server's certificate.
4. The TLS client presents its TLS certificate.
5. The TLS server verifies the TLS client's certificate.
6. The TLS connection is established.
7. The NE begins transmitting BMP data to the station over the encrypted TLS channel.

TLS version 1.3, defined in {{RFC8446}}, streamlines the handshake process and supports more robust cipher suites compared to previous versions enhancing both speed and security.

The BMPS client & server are REQUIRED to support TLS 1.3 or higher to ensure secure communication.

## TLS Certificate Validation

Each peer MUST validate the certificate path of the remote peer, including revocation checking.

If the verification succeeds, the authentication is successful and the connection is permitted.  Policy may impose further constraints upon the peer, allowing or denying the connection based on certificate fields or any other parameters exposed by the implementation.

Unless disabled by configuration, a peer MUST NOT permit connection of any peer that presents an invalid TLS Certificate.

The implementation of certificate based mutual authentication MUST support certificate path verification as described in Section 6 of {{RFC5280}}.

In some deployments, a peer could be isolated from a remote peer's Certification Authority (CA).  Implementations for these deployments MUST support certificate chains (a.k.a. bundles or chains of trust), where the entire chain of the remote's certificate is stored on the local peer.

TLS Cached Information Extension {{RFC7924}} SHOULD be implemented. Other approaches may be used for loading the intermediate certificates onto the client, but MUST include support for revocation checking such as CRL.

## TLS Certification Identification

For the BMP client-side validation of presented BMPS server identities, implementations MUST follow {{RFC9525}} validation
techniques. Identifier types DNS-ID, IP-ID or SRV-ID are applicable for use with the BMPS protocol, selected by operators depending upon the deployment design. BMPS does not use URI-IDs for BMPS server identity verification. The wildcard character MUST NOT be included in the presented BMPS server identities.

For the BMPS server-side validation of client identities, implementations MUST support the ability to configure which fields of a certificate are used for client identification, to verify that the client is a valid source for the received certificate and that it is
permitted access to TACACS+. Implementations MUST support either:

Network address based validation methods as described in Section 5.2 of {{RFC5425}}.

or

Client Identity validation of a shared identity in the certificate subjectAltName.  This is applicable in deployments where the client
securely supports an identity which is shared with the BMPS server.  This approach allows a client's network location to be reconfigured without issuing a new client certificate.

Implementations MUST support the TLS Server Name Indication extension (SNI) (Section 3 of {{RFC6066}}), and MUST support the ability to
configure the BMPS server's domain name, so that it may be included in the SNI "server_name" extension of the client hello. Please see Section XXX-Need-to-cross-reference-XXX for security related operator considerations.

Certificate provisioning is out of scope of this document.

## TLS Resumption

TLS Resumption protocol, as detailed in {{RFC8446}}, MAY be implemented to minimise the number of round trips during the handshake process. When a BMPS server is presented with a resumption request from the TLS client, it MAY still choose to require a full handshake. Where implemented, the resumption ticket_lifetime SHOULD be configurable, including a zero seconds lifetime.  Please refer to Section 4.6.1 of {{RFC8446}} for guidance on ticket lifetime.

TLS Resumption does not change the stateless behaviour the BMP Monitoring Station so all BMP message types need to resent once the BMPS session is reestablished.

## Operational Recommendations for BMPS

The BMP over TLS (BMPS) is RECOMMENDED as an alternative mechanism to safeguard BMP sessions in scenarios where alternative protections like IPSec may not be feasible or deployed.


# Security Considerations

The BMPS implementation increases computational demands due to continuous encryption and decryption processes, resulting in high CPU utilization and potential vulnerability to denial-of-service attacks.

The TLS cipher suites that provide only data integrity validation without encryption SHOULD NOT be used by default.

The BMPS implementation SHOULD follow the best practices and recommendations for using TLS, as per the Recommendations for Secure Use of Transport Layer Security (TLS) and Datagram Transport Layer Security (DTLS) as defined in {{RFC7525}}.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

This document is the result of studying all the referenced RFCs and drawing some parallels from PCEPS {{RFC8253}} and {{?draft-ietf-opsawg-tacacs-tls13}}, leading to the specification for BMP over TLS (BMPS).

We are grateful to the contributors of the RFCs listed in the References section. Their work has been instrumental in shaping and inspiring the development of this specification.
