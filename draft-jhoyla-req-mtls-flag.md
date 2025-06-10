---
title: "TLS Flag - Request Client Auth"
category: info

docname: draft-jhoyla-req-mtls-flag-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number: 0
date: 25/06/11
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
supports client authentication and will handle a `CertificateRequest`
gracefully. This document defines a TLS Flag that enables clients to provide
this hint.


--- middle

# Introduction

This document specifies a TLS Flag {{!I-D.ietf-tls-tlsflags}} that allows a
client to prompt the server to request a client certificate during the
handshake. Sometimes a server does not want to authenticate every client, but
might wish to authenticate a subset of them. In TLS 1.3 this may be done with
post-handshake authentication, however this adds an extra round trip, and
requires negotiation at the application layer.

The behaviour specified in {{?RFC8446}} for a client that receives a
`CertificateRequest` that it cannot satisfy is to send an empty `Certificate`
message followed by a `Finished` message. However in practice many clients
simply terminate the connection in anticipation of a "certificate_required"
alert.

This behaviour makes it difficult for servers to send `CertificateRequest`
messages in the wild. A client sending the `request_client_auth` flag in the
`ClientHello` allows the server to request authentication during the initial
handshake only when it receives a hint the client will handle the request
without terminating the connection, and, if unable to authenticate, by sending
an empty `Certificate` message, per {{?RFC8446, Section 4.4.2}}. This enables a
number of use cases, for example allowing bots to authenticate themselves when
mixed in with general traffic.

This flag is intended to be used by clients sending automated traffic, not
those that are directly operated by (human) users. Clients that set this flag
SHOULD only do so when a client certificate is directly available to them, and
MUST NOT prompt the user or take actions that would prompt the user for example
trying to read certificates from off-board devices such as smart cards.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Flag specification

The `request_client_auth` flag is sent by the client in the `ClientHello`
message, and, if accepted by the server, acknowledged in the server's
`CertificateRequest` message. A server that receives this flag and wishes to
accept the extension MUST send a `CertificateRequest` message with the TLS
Flags extension, and with the `request_client_auth` flag set. This informs the
client that the reason for the `CertificateRequest` was because of the
`request_client_auth` flag and not due to some other policy. This allows
clients to select certificates based on whether the server accepted the flag or
not.

The server MAY send a `CertificateRequest` message without the
`request_client_auth` flag set if it wishes to negotiate client authentication
for another reason. This could occur if, for example, the server is configured
to always request a client certificate.

The client sending this flag MUST send a `Certificate` message if it receives a
`CertificateRequest` with the `request_client_auth` flag set, even if said
`Certificate` message is empty, and SHOULD NOT preemptively terminate the
connection anticipating a "certificate_required" alert. Appropriate cases where
a client would preemptively terminate the connection include where the
`CertificateRequest` includes a `certificate_authorities` extension or
`signature_algorithms` extension that the client cannot satisfy.

A server MUST NOT set the `request_client_auth` flag in the
`CertificateRequest` unless it received the flag in the `ClientHello`. A client
receiving the `request_client_auth` flag that did not set it in the
`ClientHello` MUST generate a fatal `illegal_parameter` alert.

A client that receives this flag in any message other than `CertificateRequest`
MUST terminate the connection with a fatal `illegal_parameter` alert. Similarly
a server that receives this flag in any message other than `ClientHello` MUST
terminate the connection with a fatal `illegal_parameter` alert.

# Security Considerations

This flag should have no effect on the security of TLS, as the server may
always send a `CertificateRequest` message during the handshake. This flag
merely provides a hint that the client will handle the request gracefully.
Because the server acknowledges the flag in the `CertificateRequest` the client
can always be sure whether a `CertificateRequest` was triggered by
`request_client_auth` or not.

TODO: Discuss security considerations related to a flag that changes which
trust anchors are offered and how to handle/authorise application data.

# IANA Considerations

This document requests IANA to add an entry to the TLS Flags registry in the
TLS namespace with the following values:

* Value shall be TBD
* Flag Name shall be request_mtls.
* Message shall be CH, CR
* Recommended shall be set to no (N)
* The reference shall be this document {!draft-jhoyla-req-mtls}

--- back

# Appendix

## Alternative Approaches

### HTTP Signatures

HTTP Signatures {{?I-D.draft-meunier-web-bot-auth-architecture}} describes a
mechanism for implementing bearer tokens at the HTTP layer that authenticate
requests. This mechanism suffers from a number of drawbacks, notably that a new
signature must be created and verified for every request (rather than one per
connection), and second that, because the tokens are not bound to the
underlying TLS connection they can be stolen and replayed. This is especially
problematic in the multi-CDN ("Content Distribution Network") use case. If a
bot creates a token for a realm that is provided by more than one CDN then the
CDN that receives that token can use it to attack any of the other CDNs. This
means that all CDNs are required to trust each other.

### Concealed Auth

Concealed Auth {{?RFC9729}} allows a client to attach a header at the HTTP
layer that cryptographically binds a public key to the underlying TLS
connection. This gives us a similar authentication property to using
`request_client_auth`, and is also client initiated. This solution is
applicable in some contexts, but may be difficult to implement.

The way many HTTP libraries work is that they dispatch requests to a connection
pool. Once the request arrives at the pool it is sent on a chosen connection
driven by internal logic. This, unfortunately, makes it difficult to know which
connection a request will be sent on, and thus to produce the appropriate
signature.

It is possible to avoid this issue, for example by not using connection
pooling, or by modifying the implementation to add the header on the first
request of each connection. Where modifying the implementation is not possible
various ad-hoc solutions can be used such as sending a special "pre-request" on
each connection that's added to a pool. However these solutions tend to be
bespoke per library and bring a number of their own issues (e.g. handling
servers that terminate connections after a single request, or handling
resumption).

### Post-handshake Authentication (PHA)

Post-handshake authentication is part of the core TLS 1.3 RFC, although it is
not widely supported. Its use incurs an extra round-trip, and would need
negotiation at the application layer. This just trades one negotiation for
another, whilst making the protocol more expensive to run. Further there are
some reservations about its security guarantees
{{?CHHSV17=DOI.10.1145/3133956.3134063}}, Section 5.2.

### ALPN

One proposal is for the client to send new ALPN extensions, e.g. `h2+bot`. This
is functionally equivalent to this draft, but with a less efficient encoding.
ALPN negotiation consists of a client sending a list of protocols it is willing
to speak over the TLS connection, and the server selecting exactly one.
Doubling the number of ALPN entries to include `h2+bot, h2` and then having the
server select one by returning `h2+bot` and a `CertificateRequest` gives
exactly the same behaviours to the server receiving a TLS Flag, and echoing it.
As a side note, it is also an abuse of the ALPN mechanism, as `h2` and `h2+bot`
are indistinguishable at the application layer, the difference is in the TLS
handshake. This proposal also has the downside of the edge case where the
server selecting `h2+bot` but not sending a `CertificateRequest`.

### Client-Initiated Exported Authenticators

Exported Authenticators {{?RFC9261}} allow clients to provide certificates at
the application layer once the connection is established. A limitation of this
mechanism is that it has to be triggered by the server, however an update to
the mechanism to allow the client to send the certificate could also be used to
solve this mechanism. This would require modifications to both the standard and
the implementation.

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

