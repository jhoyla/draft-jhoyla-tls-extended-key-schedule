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
    ins: "J. Hoyland"
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

TLS provides exporter keys that allow for other protocols to provide
data authenticated by the TLS channel. This can be used to bind a protocol to a
specific TLS handshake, giving joint authentication guarantees.

In a similar way, this mechanism describes a way to introduce externally
authenticated data to a TLS handshake.  To add additional authenticated data
into the protocol you can modify the binder computation. Adding data that has
been authenticated outside the TLS protocol allows the client to add
authentication guarantees to the TLS handshake beyond those directly provided.
In particular this can be used to bind external protocols to the TLS protocol.
For example if two parties have a shared PSK, but have not agreed a hash
algorithm then this process can be used to ensure agreement on the hash
algorithm before the TLS handshake completes, and further to ensure that both
parties agree on the selected hash algorithm.

The change to the binder key computation is necessary because otherwise one
party may believe it has agreed on some authenticated data, but the other
party may believe that it is simply using a vanilla PSK, and be unaware of the
extra context implied.

~~~
             0
             |
             v
   PSK ->  HKDF-Extract = Early Secret
             |
             +-----> Derive-Secret(., "ext binder"
             |                      | "res binder"
             |                      | "imp ext binder"
             |                      | "imp res binder", "")
             |                     = binder_key
~~~

Using the "imp ext binder" label implies that both parties agree that there is
some context that has been agreed, and that they are using an external PSK.
Using the "imp res binder" label implies that both parties agree that there is
some context that has been agreed, and that they are using a resumptionn PSK.

The use of the imp binder derivations means that the PSK ID MUST have an opaque
context field.
For example, a PSK with additional data could be constructed as follows.

~~~
  struct {
    opaque external_identity<1...2^16-1>;
    opaque context<0...2^16>;
  } PSKIDWithAdditionalData
~~~

external_identity
: is the PSK_ID that would be used when no additional data is present

context
: contains the additional data


If this additional data is a channel binding for a channel, `c`, then this
structure SHOULD bind the TLS handshake to `c`, giving combined authentication
guarantees.

# Additional Derive Secret

To add confidentiality guarantees to the ClientHello using PSK binder injection
is insufficient. This is because cut-and-paste attacks are still possible on the
ClientHello. Even though such attacks may cause the handshake to fail, they may
still leak information in the ClientHello.

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
