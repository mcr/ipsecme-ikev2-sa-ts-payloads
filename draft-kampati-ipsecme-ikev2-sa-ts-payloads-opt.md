---
title: IKEv2 Optional SA&TS Payloads in Child Exchange
abbrev: IKEv2 Optional Child SA&TS Payloads
docname: draft-kampati-ipsecme-ikev2-sa-ts-payloads-opt-05
category: std

ipr: trust200902
area: Security
workgroup: IPSECME Working Group
keyword: Internet-Draft

stand_alone: true
pi:
  toc: yes
  sortrefs: yes
  symrefs: yes

author:
- ins: S. Kampati
  name: Sandeep Kampati
  org: Huawei Technologies
  abbrev: Huawei
  email: sandeepkampati@huawei.com
  street: Divyashree Techno Park, Whitefield
  code: '560066'
  city: Bangalore
  region: Karnataka
  country: India
- ins: W. Pan
  name: Wei Pan
  org: Huawei Technologies
  abbrev: Huawei
  email: william.panwei@huawei.com
  street: 101 Software Avenue, Yuhuatai District
  code: ""
  city: Nanjing
  region: Jiangsu
  country: China
- ins: M. Bharath
  name: Meduri S S Bharath
  org: Mavenir Systems Pvt Ltd
  abbrev: Mavenir
  email: bharath.meduri@mavenir.com
  street: Manyata Tech Park
  code: ""
  city: Bangalore
  region: Karnataka
  country: India
- ins: M. Chen
  name: Meiling Chen
  org: China Mobile
  abbrev: CMCC
  email: chenmeiling@chinamobile.com
  street: 32 Xuanwumen West Street, West District
  code: "100053"
  city: Beijing
  region: Beijing
  country: China

normative:
  RFC2119:
  RFC8174:
  RFC7296:

informative:



--- abstract

This document describes a method for reducing the size of the
Internet Key Exchange version 2 (IKEv2) exchanges at time of rekeying
IKE SAs and Child SAs by removing or making optional of SA & TS
payloads. Reducing size of IKEv2 exchanges is desirable for low
power consumption battery powered devices. It also helps to avoid IP
fragmentation of IKEv2 messages.

--- middle

# Introduction

The Internet Key Exchange protocol version 2 (IKEv2) specified in
[RFC7296] is used in the IP Security (IPSec) architecture for the
purposes of Security Association (SA) parameters negotiation and
authenticated key exchange. The protocol uses UDP as the transport
for its messages, which size varies from less than one hundred bytes
to several kilobytes.

According to [RFC7296], the secret keys used by IKE/IPSec SAs should
be used only for a limited amount of time and to protect a limited
amount of data. When the lifetime of an SA expires or is about to
expire, the peers can rekey the SA to reestablish a new SA to take
the place of the old one.

For security gateways/ePDG in 4G networks and cRAN/Cloud in 5G
networks, they will support more than 100,000 IKE/IPSEC tunnels. So
on an average, for every second there may be hundreds or thousands of
IKE SAs and Child SAs that are rekeying. This takes huge amount of
bandwidth, packet fragmentation and more processing resources. For
these devices, these problems can be solved by introducing the
solution described in this document.

This is also useful in Internet of Things (IoT) devices which
utilizing lower power consumption technology. For
these devices, reducing the length of IKE/Child SA rekeying messages
can save the bandwidth consumption. At the same time, it can also
save the computing processes by less payload are included.

Most devices don’t prefer to change cryptographic suites frequently.
By taking this advantage the SA and TS payloads can be made optional
at the time of rekeying IKE SAs and Child SAs. In such situation,
only a new SPI value is needed to create the new IKE SA and Child SA.
So a new Notify payload which contains the needed SPI value can be
sent instead of the SA and TS payloads.

In case of rekeying IKE SAs, the SA payloads can be optimized if
there is no change of cryptographic suites. In case of rekeying
Child SAs, the SA and TS payloads can be optimized if there is no
change of cryptographic suites and ACL configuration.

# Conventions Used in This Document

## Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP
14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all
capitals, as shown here.

# Protocol Details

This section provides protocol details and contains the normative
parts.

Before using this new optimization, the IPSec implementation who
supports it has to know that the peer also supports it. Without the
support on both sides, the optimized rekeying messages sent by one
peer may be unrecognizable for the other peer. To prevent this failure
from happening, the first step is to negotiate the support of this
optimization between the two peers. There are two specific rekeying
SAs cases: rekeying IKE SAs and rekeying Child SAs. After the
negotiation, the initiator can optimize the rekeying message payloads
in both cases. In other words, once the negotiation of support for
optimizing payloads at rekeying IKE SAs and Child SAs is complete,
both IKE SAs and Child SAs rekeying are supported by the two sides.
The responder can react based on the received rekeying message.

## Negotiation of Support for Optimizing Payloads at Rekeying IKE SAs and Child SAs {#negotiation}

The initiator indicates its support for optimizing payloads
at rekeying IKE SAs and Child SAs by including a Notify payload of
type MINIMAL_REKEY_SUPPORTED in the IKE_AUTH request message. By
observing the MINIMAL_REKEY_SUPPORTED notification in the received
message, the responder can deduce the initiator’s support of this
extension. If the responder also supports this extension, it
includes the MINIMAL_REKEY_SUPPORTED notification in the
corresponding response message. After receiving the response
message, the initiator can also know the support of this extension of
the responder side.

The IKE_AUTH message exchange in this case is shown below:

~~~~
Initiator                         Responder
--------------------------------------------------------------------
HDR, SK {IDi, [CERT,] [CERTREQ,]
    [IDr,] AUTH, SAi2, TSi, TSr,
    N(MINIMAL_REKEY_SUPPORTED)} -->
                              <-- HDR, SK {IDr, [CERT,] AUTH,
                                      SAr2, TSi, TSr,
                                      N(MINIMAL_REKEY_SUPPORTED)}
~~~~

If the responder doesn’t support this extension, it MUST ignore the
MINIMAL_REKEY_SUPPORTED notification sent by the initiator and MUST
NOT respond error to the initiator. With no MINIMAL_REKEY_SUPPORTED
notification in the response message, the initiator can deduce that
the responder doesn’t support this extension. In this case, the IKE
SAs and Child SAs rekeyings happen as the usual way without the
optimizations defined in this document.

The IKE_AUTH message exchange in this case is shown below:

~~~~
Initiator                         Responder
--------------------------------------------------------------------
HDR, SK {IDi, [CERT,] [CERTREQ,]
    [IDr,] AUTH, SAi2, TSi, TSr,
    N(MINIMAL_REKEY_SUPPORTED)} -->
                              <-- HDR, SK {IDr, [CERT,] AUTH,
                                      SAr2, TSi, TSr}
~~~~

## Payload Optimization at Rekeying IKE SAs

The payload optimization at rekeying IKE SAs MUST NOT be used unless
both peers have indicated their support of this extension by using
the negotiation method described in {{negotiation}}.

At the time of rekeying an IKE SA, when the initiator determines
there is no change on its cryptographic suites since this IKE SA was
created or last rekeyed, it MUST send the REKEY_OPTIMIZED notification
payload instead of the SA payloads in the rekeying request message.
In this REKEY_OPTIMIZED notification, it contains the initiator’s new
Security Parameter Index (SPI) used for creating the new IKE SA.

After receiving the initiator’s rekeying request message with the
REKEY_OPTIMIZED notification and no SA payloads, the responder knows
that the initiator wants to optimize the rekeying payload. Then when
it determines that there is also no change in its cryptographic
suites, the responder MUST send the rekeying respond message to the
initiator with the REKEY_OPTIMIZED notification payload instead of the
SA payloads. In this REKEY_OPTIMIZED notification, it contains the
responder’s new SPI used for creating the new IKE SA.

According to the initiator’s new SPI and the responder’s new SPI, the
initiator and the responder can rekey the IKE SA on both sides.

The CREATE_CHILD_SA message exchange in this case is shown below:

~~~~
Initiator                         Responder
--------------------------------------------------------------------
HDR, SK {N(REKEY_OPTIMIZED),
    Ni, KEi} -->
                              <-- HDR, SK {N(REKEY_OPTIMIZED),
                                      Nr, KEr}
~~~~

The initiator sends a REKEY_OPTIMIZED notification payload, a Nonce
payload and a Diffie-Hellman value in the KEi payload. A new
initiator SPI is supplied in the SPI field of the REKEY_OPTIMIZED
notification payload.
These messages also follow the original Perfect Forwarding Secrecy (PFS)
with the signature and encryption algorithms used as last message.

The responder replies (using the same Message ID to respond) with a
REKEY_OPTIMIZED notification payload, a Nonce payload and a Diffie-Hellman
value in the KEr payload. A new responder SPI is supplied in
the SPI field of the REKEY_OPTIMIZED notification payload.

This REKEY_OPTIMIZED notification MUST be included in a CREATE_CHILD_SA
exchange message when there is no SA payloads included. When the
REKEY_OPTIMIZED notification payload is included, the SA payload MUST
NOT be included.

## Payload Optimization at Rekeying Child SAs

The payload optimization at rekeying Child SAs MUST NOT be used
unless both peers have indicated their support of this extension by
using the negotiation method described in {{negotiation}}.

At the time of rekeying a Child SA, when the initiator determines
there is no change in its cryptographic suites and ACL configuration
since this Child SA was created or last rekeyed, it MUST send the
REKEY_OPTIMIZED notification payload instead of the SA and TS
payloads in the rekeying request message. In this REKEY_OPTIMIZED
notification, it contains the initiator’s new Security Parameter
Index (SPI) used for creating the new Child SA.

After receiving the initiator’s rekeying request message with the
REKEY_OPTIMIZED notification and no SA and TS payloads, the responder
knows that the initiator wants to optimize the rekeying payload.
Then when it determines that there is also no change in its
cryptographic suites and ACL configuration, the responder MUST send
the rekeying respond message to the initiator with the
REKEY_OPTIMIZED notification payload instead of the SA and TS
payloads. In this REKEY_OPTIMIZED notification, it contains the
responder’s new SPI used for creating the new Child SA.

According to the old SPIs included in the REKEY_SA payloads and
the new SPIs included in the REKEY_OPTIMIZED payloads, the
initiator and the responder can rekey the Child SA on both sides.

The CREATE_CHILD_SA message exchange in this case is shown below:

~~~~
Initiator                         Responder
--------------------------------------------------------------------
HDR, SK {N(REKEY_SA), N(REKEY_OPTIMIZED),
    Ni, [KEi,]} -->
                              <-- HDR, SK {N(REKEY_OPTIMIZED),
                                      Nr, [KEr,]}
~~~~

This REKEY_OPTIMIZED notification MUST be included in a
CREATE_CHILD_SA exchange message when there is no SA and TS payloads
included at the time of rekeying Child SAs. When the REKEY_OPTIMIZED
notification payload is included, the SA and TS payloads MUST NOT be
included.

# Payload Formats

## MINIMAL_REKEY_SUPPORTED Notification

The MINIMAL_REKEY_SUPPORTED notification is used by the initiator and
responder to inform their ability of optimizing payloads at
the time of rekeying IKE SAs and Child SAs to the peers. It is
formatted as follows:

~~~~
                     1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Next Payload  |C|  RESERVED   |         Payload Length        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Protocol ID(=0)| SPI Size (=0) |      Notify Message Type      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~

* Protocol ID (1 octet) - MUST be 0.
* SPI Size (1 octet) - MUST be 0, meaning no SPI is present.
* Notify Message Type (2 octets) - MUST be \<Need to get value from IANA\>, the value assigned for the MINIMAL_REKEY_SUPPORTED notification.

This notification contains no data.

## REKEY_OPTIMIZED Notification

The REKEY_OPTIMIZED notification is used to replace the SA payloads at
the time of rekeying IKE SAs when there is no change of cryptographic
suites in initiator and responder, and to replace the SA payloads and
TS payloads at the time of rekeying Child SAs when there is no change
of cryptographic suites and ACL configuration in initiator and responder.
It is formatted as follows:

~~~~
 0                 1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Next Payload  |C|  RESERVED   |         Payload Length        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Protocol ID    | SPI Size (=8) |      Notify Message Type      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                Security Parameter Index (SPI)                 |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~

* Protocol ID (1 octet) - MUST be 1.
* SPI Size (1 octet) - MUST be 8 when used at the time of rekeying IKE SAs and be 4 when used at the time of rekeying Child SAs.
* Notify Message Type (2 octets) - MUST be \<Need to get value from IANA\>, the value assigned for the REKEY_OPTIMIZED notification.
* SPI (4 octets or 8 octets) - Security Parameter Index. The initiator sends initiator SPI. The responder sends responder SPI.

# IANA Considerations

This document defines two new Notify Message Types in the "IKEv2
Notify Message Types - Status Types" registry. IANA is requested to
assign codepoints in this registry.

~~~~
NOTIFY messages: status types            Value
----------------------------------------------------------
MINIMAL_REKEY_SUPPORTED                  TBD
REKEY_OPTIMIZED                          TBD
~~~~

# Security Considerations

When using the payload optimization defined in this document, the rekeying of IKE SAs and Child SAs are using the same cryptographic suites.
If changes to the configurations are wanted, such as supporting a new cryptographic algorithm, the rekeying won't apply these changes.
The initiator or responder should start a new IKE SA or Child SA to apply the new changes.

# Acknowledgments

Special thanks go to Paul Wouters, Valery Smyslov, and Antony Antony.

--- back

