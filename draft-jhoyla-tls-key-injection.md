---
title: TLS Key Injection
docname: draft-jhoyla-tls-key-injection
category: std

ipr: trust200902
area: Security
workgroup: jhoyla
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: "J Hoyland"
    name: "Jonathan Hoyland"
    organization: "Cloudflare Ltd."
    email: jonathan.hoyland@gmail.com

normative:
  RFC2119:

informative:



--- abstract

TLS is sometimes used in situations where it is necessary to inject extra key material into the handshake. This draft aims to describe some methods for doing so securely.
This key material must be injected in such a way that both parties agree on what is being injected and why, and further in what order.

--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.



--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
