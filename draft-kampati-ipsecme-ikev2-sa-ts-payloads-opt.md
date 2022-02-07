---
title: IKEv2 Optional SA&TS Payloads in Child Exchange
abbrev: IKEv2 Optional Child SA&TS Payloads
docname: draft-kampati-ipsecme-ikev2-sa-ts-payloads-opt-08
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
- ins: P. Wouters
  name: Paul Wouters
  org: Aiven
  abbrev: Aiven
  email: paul.wouters@aiven.io
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
  country: China

normative:
  RFC2119:
  RFC8174:
  RFC7296:

informative:



--- abstract

This document describes a method for reducing the size of the
Internet Key Exchange version 2 (IKEv2) CREATE\_CHILD\_SA exchanges
used for rekeying of the IKE or Child SA by replacing the SA and TS
payloads with a Notify Message payload. Reducing size and complexity
of IKEv2 exchanges is especially useful for low power consumption
battery powered devices.

--- middle

# Introduction

The Internet Key Exchange protocol version 2 (IKEv2) [RFC7296] is
used to negotiate Security Association (SA) parameters for the IKE SA
and  the Child SAs. Cryptographic key material for these SAs have a
limited lifetime before it needs to be refreshed, a process referred
to as "rekeying". IKEv2 uses the CREATE\_CHILD\_SA exchange to rekey
either the IKE SA or the Child SAs.

When rekeying, a full set of previously negotiated parameters are
exchanged. However, most of these parameters will be the same, and
some of these parameters MUST be the same.

For example, the Traffic Selector (TS) negotiated for the new Child
SA MUST cover the Traffic Selectors negotiated for the old Child SA.
And in practically all cases, a new Child SA would not need to cover
more Traffic Selectors. In the rare case where this would be needed,
either a standard rekey could be used or a new Child SA could be negotiated
followed by a deletion of the replaced Child SA.

Similarly, IKEv2 states that the cryptographic
parameters negotiated for rekeying SHOULD NOT be different. This
means that the security properties of the IKE or Child SA in practise
do not change during a typical rekey.

This document specifies a method to omit these parameters and replace
them with a single Notify Message declaring that all these parameters
are identical to the originally negotiated parameters.

Large scale IKEv2 gateways such as Evolved Packet Data Gateway (ePDG)
in 4G networks or Centralized Radio Access Network (cRAN/Cloud) gateways in
5G networks typically support more than 100000 IKE/IPSec
connections. At any point in time, there will be hundreds or thousands
of IKE SAs and Child SAs that are being rekeyed. This takes a large
amount of bandwidth and CPU power and any protocol simplification or
bandwidth reducing would result in a significant resource saving.

For Internet of Things (IoT) devices which utilize low power
consumption technology, reducing the size of the CREATE\_CHILD\_SA
exchange for rekeying reduces
its power consumption, as sending bytes over the air is usually the
most power consuming operation of such a device. Reducing the CPU
operations required to verify the rekey exchanges parameters will
also save power and extend the lifetime for these devices.

When using identical parameters for the IKE SA or Child SA rekey, the
SA and TS payloads can be omitted according to the optimization
defined in this document. For an IKE SA rekey, instead of
the (large) SA payload, only a Key Exchange (KE) payload and a new
Notify Type payload with the new SPI are required. For a Child SA
payload, instead of the SA or TS payloads, only an optional nonce
payload (when using PFS) and a new Notify Type payload with the new
SPI are needed. This makes the rekey exchange packets much smaller
and the peers do not need to verify that the SA or TS parameters are
compatible with the old SA parameters.

# Conventions Used in This Document

## Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP
14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all
capitals, as shown here.

# Negotiation of Support for OPTIMIZED REKEY

To indicate support for the optimized rekey negotiation, the
initiator includes the OPTIMIZED\_REKEY\_SUPPORTED notify payload in
the IKE\_AUTH exchange request.  A responder that supports the
optimized rekey exchange includes the OPTIMIZED\_REKEY\_SUPPORTED notify
payload in its response.  Note that the notify indicates support for
optimized rekey for both IKE and Child SAs.

When a peer wishes to rekey an IKE SA or Child SA, it MAY use the
optimized rekey method during the CREATE\_CHILD\_SA exchange.
If both peers have exchanged OPTIMIZED\_REKEY\_SUPPORTED notifies,
peers SHOULD use the optimized rekey method for rekeys but MUST accept
either regular or optimized rekey requests.

The IKE\_AUTH message exchange in this case is shown below:

~~~~
Initiator                       Responder
--------------------------------------------------------------------
HDR, SK {IDi, [CERT,] [CERTREQ,]
    [IDr,] AUTH, SAi2, TSi, TSr,
    N(OPTIMIZED_REKEY_SUPPORTED)} -->
                            <-- HDR, SK {IDr, [CERT,] AUTH,
                                    SAr2, TSi, TSr,
                                    N(OPTIMIZED_REKEY_SUPPORTED)}
~~~~

If the responder does not support this extension, as per regular
IKEv2 processing, it MUST ignore the unknown Notify payload. The
initiator will notice the lack of the OPTIMIZED\_REKEY\_SUPPORTED
Notify in the reply and thus know it cannot use the optimized rekey
method.

# Optimized Rekey of the IKE SA

The initiator of an optimized rekey request sends a CREATE\_CHILD\_SA
payload with the OPTIMIZED\_REKEY notify payload containing the new
Security Parameter Index (SPI) for the new IKE SA. It omits the SA
payload.

The responder of an optimized rekey request replies with an included
OPTIMIZED\_REKEY notify with its new IKE SPI
and also omits the SA payload.

Both parties send their nonce and KE payloads just as they would do for a
regular IKE SA rekey.

Using the old SPI from the IKE header and the two new SPIs
respectively from the initiator and responder's OPTIMIZED\_REKEY payloads,
both parties can perform the IKE SA rekey operation.

The CREATE\_CHILD\_SA message exchange in this case is shown below:

~~~~
Initiator                       Responder
--------------------------------------------------------------------
HDR, SK {N(OPTIMIZED_REKEY,newSPIi),
         Ni, KEi} -->
                            <-- HDR, SK {N(OPTIMIZED_REKEY,newSPIr),
                                         Nr, KEr}
~~~~

# Optimized Rekey of Child SAs

The initiator of an optimized rekey request sends a CREATE\_CHILD\_SA
payload with the OPTIMIZED\_REKEY notify payload containing the new
Security Parameter Index (SPI) for the new Child SA.  It omits the SA
and TS payloads.  If the current Child SA was negotiated with Perfect
Forward Secrecy (PFS), a KEi payload MUST be included as well.  If no
PFS was negotiated for the current Child SA, a KEi payload MUST NOT
be included.

The responder of an optimized rekey request performs the same
process.  It includes the OPTIMIZED\_REKEY notify with its new IKE SPI
and omits the SA and TS payloads.  Depending on the PFS negotiation
of the current Child SA, the responder includes a KEr payload.

Both parties send their nonce payloads just as they would do for a regular
Child SA rekey.

Using the old SPI from the REKEY\_SA payload and the two new SPIs
respectively from the initiator and responder's OPTIMIZED\_REKEY payloads,
both parties can perform the Child SA rekey operation.

The CREATE\_CHILD\_SA message exchange in this case is shown below:

~~~~
Initiator                       Responder
--------------------------------------------------------------------
HDR, SK {N(REKEY_SA,oldSPI), N(OPTIMIZED_REKEY,newSPIi),
         Ni, [KEi,]} -->
                            <-- HDR, SK {N(OPTIMIZED_REKEY,newSPIr),
                                         Nr, [KEr,]}
~~~~

# Payload Formats

## OPTIMIZED\_REKEY\_SUPPORTED Notify

The OPTIMIZED\_REKEY\_SUPPORTED Notify Message type notification
is used by the initiator and responder to indicate their support for the
optimized rekey negotiation.

~~~~
                     1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+---------------+-+-------------+-------------------------------+
| Next Payload  |C|  RESERVED   |         Payload Length        |
+---------------+-+-------------+-------------------------------+
|Protocol ID(=0)| SPI Size (=0) |      Notify Message Type      |
+---------------+---------------+-------------------------------+
~~~~

* Protocol ID (1 octet) - MUST be 0.
* SPI Size (1 octet) - MUST be 0, meaning no SPI is present.
* Notify Message Type (2 octets) - MUST be set to the value [TBD1].

This Notify Message type contains no data.

The Critical bit MUST be 0.  A non-zero value MUST be ignored.

## OPTIMIZED\_REKEY Notify

The OPTIMIZED\_REKEY Notify Message type is used to perform an
optimized IKE SA or Child SA rekey.

~~~~
 0                 1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+---------------+-+-------------+-------------------------------+
| Next Payload  |C|  RESERVED   |         Payload Length        |
+---------------+-+-------------+-------------------------------+
|Protocol ID    | SPI Size (=8) |      Notify Message Type      |
+---------------+---------------+-------------------------------+
|                Security Parameter Index (SPI)                 |
|                                                               |
+---------------------------------------------------------------+
~~~~

* Protocol ID (1 octet) - For an IKE SA rekey, this field MUST contain (1).  For Child SAs, this field MUST contain either (2) to indicate AH or (3) to indicate ESP.
* SPI Size (1 octet) - MUST be 8 when rekeying an IKE SA. MUST be 4 when rekeying a Child SA.
* Notify Message Type (2 octets) - MUST be set to the value [TBD2].
* SPI (4 octets or 8 octets) - Security Parameter Index. The new SPI.

The Critical bit MUST be 1.  A value of 0 MUST be ignored.

# IANA Considerations

This document defines two new Notify Message Types in the "IKEv2
Notify Message Types - Status Types" registry. IANA is requested to
assign codepoints in this registry.

~~~~
NOTIFY messages: status types            Value
----------------------------------------------------------
OPTIMIZED_REKEY_SUPPORTED                TBD1
OPTIMIZED_REKEY                          TBD2
~~~~

# Operational Considerations

Some implementations allow sending rekey messages with a different
set of Traffic Selectors or cryptographic parameters in response to a
configuration update.  IKEv2 states this SHOULD NOT be done.  Whether
or not optimized rekeying is used, a configuration change that
changes the Traffic Selectors or cryptographic parameters MUST NOT
use the optimized rekey method.  It SHOULD also not use a regular
rekey method but instead start an entire new IKE and Child SA
negotiation with the new parameters.

# Security Considerations

The optimized rekey removes sending unnecessary new parameters that
originally would have to be validated against the original
parameters.  In that sense, this optimization enhances the security
of the rekey process by reducing the complexity and code required.

# Acknowledgments

Special thanks to Valery Smyslov and Antony Antony.

--- back

