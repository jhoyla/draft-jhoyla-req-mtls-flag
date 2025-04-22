---
title: "TLS Flag - Request mTLS"
category: info

docname: draft-jhoyla-req-mtls-flag-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number: 0
date: 25/04/22
consensus: true
v: 3
# area: SEC
# workgroup: TLS Working Group
venue:
#  group: TLS
#  type: Working Group
#  mail: tls@ietf.org
#  arch: https://mailarchive.ietf.org/arch/browse/tls/
  github: "jhoyla/draft-jhoyla-req-mtls-flag"
  latest: "https://jhoyla.github.io/draft-jhoyla-req-mtls-flag/draft-jhoyla-req-mtls-flag.html"

author:
 -
    fullname: Jonathan Hoyland
    organization: Cloudflare
    email: jonathan.hoyland@gmail.com

normative:

informative:


--- abstract

Normally in TLS there is no way for the client to signal to the server that it
supports client authentication and will handle a CertificateRequest by sending
an empty Certificate message without terminating the connection. This document
defines a TLS Flag {{!I-D.ietf-tls-tlsflags}} that enables clients to provide
this hint.


--- middle

# Introduction

This document specifies a TLS Flag that indicates to the server that the client
supports mutually authenticated TLS (in at least some cases). Sometimes a server
does not want to authenticate every client, but might wish to authenticate a
subset of them. In TLS 1.3 this may be done with post-handshake auth, however
this adds an extra round-trip, and requires negotiation at the application
layer.

The behaviour specified in {{?RFC8446}} for a client that receives a
CertificateRequest that it cannot satisfy is to send an empty Certificate
message followed by a Finished message. However in practice many clients simply
terminate the connection in anticipation of a "certificate_required" alert.

This behaviour makes it difficult for servers to send CertificateRequest
messages in the wild. A client sending the request mTLS flag in the ClientHello
allows the server to request authentication during the initial handshake only
when it receives a hint the client will handle the request without terminating
the connection, and, if unable to authenticate, by sending an empty certificate
message, per {{?RFC8446, Section 4.4.2}}. This enables a number of use cases,
for example allowing bots to authenticate themselves when mixed in with general
traffic.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Flag specification

A server receiving this flag MAY send a CertificateRequest message, and a client
sending this flag MUST send a Certificate message if it receives a
CertificateRequest, even if said Certificate message is empty, and SHOULD NOT
preemptively terminate the connection anticipating a "certificate_required"
alert.

# Security Considerations

This flag should have no effect on the security of TLS, as the server may always
send a CertificateRequest message during the handshake. This flag merely
provides a hint that the client will handle the request gracefully.

# IANA Considerations

This document requests IANA to add an entry to the TLS Flags registry in the TLS
namespace with the following values:

* Value shall be TBD
* Flag Name shall be request_mtls.
* Message shall be CH
* Recommended shall be set to no (N)
* The reference shall be this document {!draft-jhoyla-req-mtls}

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
