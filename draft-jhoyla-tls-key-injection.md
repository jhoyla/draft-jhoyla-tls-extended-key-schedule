---
title: TLS 1.3 Key Schedule Injection
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
    ins: J. Hoyland
    name: Jonathan Hoyland
    organization: Cloudflare Ltd.
    email: jonathan.hoyland@gmail.com

normative:
  RFC2119:


--- abstract

TLS 1.3 is sometimes used in situations where it is necessary to inject extra key
material into the handshake. This draft aims to describe methods for doing
so securely.  This key material must be injected in such a way that both parties
agree on what is being injected and why, and further, in what order.

--- middle

# Introduction

Injecting key material into the TLS handshake is a non-trivial process because
both parties need to agree on the injection content and context.  If the two
parties do not agree then an attacker may exploit the mismatch in so-called channel
synchronization attacks.

Injecting key material into the TLS handshake allows other protocols to be bound
to the handshake. For example, it may provide additional protections to the ClientHello
message, which in the standard TLS handshake only receives protections after the
server's Finished message has been received. It may also permit the use of
combined shared secrets, possibly from multiple key exchange algorithms, to be
included in the key schedule. This pattern is common for Post Quantum key exchange
algorithms, as discussed in {{?I-D.stebila-tls-hybrid-design}}.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# Key Schedule Injection

This section describes ways in which additional secrets can be injected into
the TLS 1.3 key schedule.

## Early Secret Injection

TLS provides exporter keys that allow for other protocols to provide
data authenticated by the TLS channel. This can be used to bind a protocol to a
specific TLS handshake, giving joint authentication guarantees.
In a similar way, one may wish to introduce externally authenticated and
pre-shared data to the early secret derivation. This can be used to bind external
protocols to the TLS protocol.

To achieve this, pre-shared keys modify the binder key computation. This is
needed since it ensures that both parties agree on both the authenticated
data and the context in which it was used.

The binder key computation change is as follows:

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
             v
~~~

Use of the "imp ext binder" label implies that both parties agree that there is
some context that has been agreed, and that they are using an external PSK.
Use of the "imp res binder" label implies that both parties agree that there is
some context that has been agreed, and that they are using an resumption PSK.
This assumes the PSK has some mechanism by which additional context is included.
{{!I-D.ietf-tls-external-psk-importer}} describes one way by which such context
may be included.

~~~
  struct {
    opaque external_identity<1...2^16-1>;
    opaque context<0...2^16>;
  } PSKIDWithAdditionalData
~~~

Those using the "imp ext binder" or "imp res binder" label MUST include a
context field, to allow the additional data.

## Handshake Secret Injection

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
separation of the key schedule from vanilla TLS handshakes, because HKDFs
can be assumed to ensure that keys derived with different labels are
independent.

# Key Schedule Injection Structure

In some cases, protocols may require more than one secret to be injected at a particular
stage in the key schedule. Thus, we require a generic and extensible way of doing so.
To accomplish this, we use a structure --  KeyScheduleInput -- that encodes well-ordered
sequences of secret material to inject into the key schedule. KeyScheduleInput is defined
as follows:

~~~
struct {
    KeyScheduleSecretType type;
    opaque secret_data<0..2^16-1>;
} KeyScheduleSecret;

enum {
    (65535)
} KeyScheduleSecretType;

struct {
    KeyScheduleSecret secrets<0..2^16-1>;
} KeyScheduleInput;
~~~

Each secret included in a KeyScheduleInput structure has a type and corresponding secret data.
Each secret MUST have a unique KeyScheduleSecretType. When encoding KeyScheduleInput as the
key schedule Input value, the KeyScheduleSecret values MUST be in ascending sorted order. This
ensures that endpoints always encode the same KeyScheduleInput value when using the same
secret keying material.

# Security Considerations

[[OPEN ISSUE: This draft has not seen any security analysis.]]

# IANA Considerations

[[TODO: define secret registry structure]]

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
