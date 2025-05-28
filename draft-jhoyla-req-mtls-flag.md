---
title: "TLS Flag - Request mTLS"
category: info

docname: draft-jhoyla-req-mtls-flag-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number: 0
date: 25/05/27
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
supports client authentication and will handle a CertificateRequest gracefully.
This document defines a TLS Flag {{!I-D.ietf-tls-tlsflags}} that enables clients
to provide this hint.


--- middle

# Introduction

This document specifies a TLS Flag that allows a client to attempt to negotiate
mutually authenticated TLS (in at least some cases). Sometimes a server does not
want to authenticate every client, but might wish to authenticate a subset of
them. In TLS 1.3 this may be done with post-handshake auth, however this adds an
extra round trip, and requires negotiation at the application layer.

The behaviour specified in {{?RFC8446}} for a client that receives a
`CertificateRequest` that it cannot satisfy is to send an empty Certificate
message followed by a `Finished` message. However in practice many clients simply
terminate the connection in anticipation of a "certificate_required" alert.

This behaviour makes it difficult for servers to send `CertificateRequest`
messages in the wild. A client sending the `request_mtls` flag in the `ClientHello`
allows the server to request authentication during the initial handshake only
when it receives a hint the client will handle the request without terminating
the connection, and, if unable to authenticate, by sending an empty certificate
message, per {{?RFC8446, Section 4.4.2}}. This enables a number of use cases,
for example allowing bots to authenticate themselves when mixed in with general
traffic.

This flag is intended to be used by clients sending automated traffic, not those
that are directly operated by (human) users. Clients that set this flag SHOULD
only do so when a client certificate is directly available to them, and MUST NOT
prompt the user or take actions that would prompt the user for example trying to
read certificates from off-board devices such as smart cards.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Flag specification

The `request_mtls` flag is sent by the client in the `ClientHello` message,
and, if accepted by the server, acknowledged in the server's
`CertificateRequest` message. A server that receives this flag and wishes to
accept the extension MUST send a `CertificateRequest` message with the TLS
Flags extension, and with the `request_mtls` flag set. The server MAY send a
`CertificateRequest` message without the `request_mtls` flag set if it wishes
to negotiate client authentication for another reason. The client sending this
flag MUST send a `Certificate` message if it receives a `CertificateRequest`
with the `request_mtls` flag set, even if said `Certificate` message is empty,
and SHOULD NOT preemptively terminate the connection anticipating a
"certificate_required" alert.

A server receiving the `request_mtls` flag in a `ClientHello` message that
wishes to accept the flag MUST echo the `request_mtls` flag in the
`CertificateRequest` message. This informs the client that the reason for the
`CertificateRequest` was because of the `request_mtls` flag and not due to some
other policy. This allows clients to select certificates based on whether the
server accepted the flag or not.

A server MUST NOT set the `request_mtls` flag in the `CertificateRequest` unless
it received the flag in the `ClientHello`. A client receiving the `request_mtls`
flag that did not set it in the `ClientHello` MUST generate a fatal
`illegal_parameter` alert.

# Security Considerations

This flag should have no effect on the security of TLS, as the server may always
send a `CertificateRequest` message during the handshake. This flag merely
provides a hint that the client will handle the request gracefully. Because the
server acknowledges the flag in the `CertificateRequest` the client can always
be sure whether a `CertificateRequest` was triggered by `request_mtls` or not.

# IANA Considerations

This document requests IANA to add an entry to the TLS Flags registry in the TLS
namespace with the following values:

* Value shall be TBD
* Flag Name shall be request_mtls.
* Message shall be CH, CR
* Recommended shall be set to no (N)
* The reference shall be this document {!draft-jhoyla-req-mtls}

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
