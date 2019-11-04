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
    EPSKI: I-D.ietf-tls-external-psk-importer


--- abstract

TLS is sometimes used in situations where it is necessary to inject extra key
material into the handshake. This draft aims to describe some methods for doing
so securely.  This key material must be injected in such a way that both parties
agree on what is being injected and why, and further in what order.

--- middle

# Introduction

Injecting key material into the TLS handshake is a non-trivial process, because
both parties need to agree on what is being injected and why.  If the two
parties do not agree then there is scope for an attacker toc exploit the
mismatch to perform channel synchronization attacks.

Injecting key material into the TLS handshake allows other protocols to be bound
to the handshake, and to provide additional protections to the ClientHello
message, which in the standard TLS handshake only receives protections after the
server's Finished message has been received.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# PSK Injection

To bind one protocol to another you can use the procedure defined in
{{EPSKI}} to add a special PSK to the protocol. This
process gives authentication guarantees to both the protocol being bound and
TLS.

# Additional Derive Secret

To add confidentiality guarantees to the ClientHello using PSK binders is
insufficient. This is because cut-and-paste attacks are still possible on the
ClientHello. Even though the handshake will fail, information in the
ClientHello may still be exposed.

To inject key material into the Handshake Secret it is recommended to use an
extra derive secret.

~~~
             |
             v
       Derive-Secret(., "derived early", "")
             |
             v
    Input -> HKDF-Extract

             |
             v
       Derive-Secret(., "derived", "")
             |
             v
   (EC)DHE -> HKDF-Extract = Handshake Secret
             |
             v
~~~

As shown in the figure above, the key schedule has an extra derive secret and
HKDF-Extract step. This extra step isolates the Input material from the rest of
the handshake secret, such that even maliciously chosen values cannot weaken the
security of the key schedule overall.

The additional Derive-Secret with the "derived early" label enforces the
seperation of the key schedule from vanilla TLS handshakes, because HKDFs
can be assumed to ensure that keys derived with different labels are
independent.



# Security Considerations

This draft has not seen substantial security analysis.


# IANA Considerations

This document has no IANA actions.



--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
