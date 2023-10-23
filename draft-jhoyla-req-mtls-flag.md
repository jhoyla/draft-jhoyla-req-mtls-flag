---
title: "TLS Flag - Request mTLS"
category: info

docname: draft-jhoyla-req-mtls-flag-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number: 0
date: 23/10/23
consensus: true
v: 3
area: SEC
workgroup: TLS Working Group
venue:
  group: TLS
  type: Working Group
  mail: tls@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/tls/
  github: jhoyla/draft-jhoyla-req-mtls-flag
  latest: https://example.com/LATEST

author:
 -
    fullname: Jonathan Hoyland
    organization: Cloudflare
    email: jonathan.hoyland@gmail.com

normative:

informative:


--- abstract

Normally in TLS there is no way for the client to signal to the server that it
has been configured with a certificate suitable for mTLS. This document defines
a TLS Flag {{!I-D.ietf-tls-tlsflags}}


--- middle

# Introduction

This document specifies a TLS Flag that indicates to the server that the client
supports mTLS. Sometimes a server does not want to negotiate mTLS with every
client, but might wish to authenticate a subset of them. In TLS 1.3 this may be
done with post-handshake auth, however this adds an extra round-trip, and
requires negotiation at the application layer. A client sending the request
mTLS flag in the ClientHello allows the server to request authentication during
the initial handshake only when it receives a hint the client supports it.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Flag specification

A server receiving this flag MAY send a CertificateRequest message.

# Security Considerations

This flag should have no effect on the security of TLS, as the server may always
send a CertificateRequest message during the handshake. This flag merely
provides a hint that the client will be able to fulfil the request. If the
client sets this flag but then fails to provide a certificate the server MAY
terminate the connection with a bad_certificate error.

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
