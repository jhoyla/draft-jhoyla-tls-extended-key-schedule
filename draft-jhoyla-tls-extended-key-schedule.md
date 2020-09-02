---
title: TLS 1.3 Extended Key Schedule
docname: draft-jhoyla-tls-extended-key-schedule
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
  -
    ins: C. A. Wood
    name: Christopher A. Wood
    organization: Cloudflare
    email: caw@heapingbits.net

normative:
  RFC2119:

informative:

  BINDEL: DOI.10.1007/978-3-030-25510-7_12


--- abstract

TLS 1.3 is sometimes used in situations where it is necessary to inject extra
key material into the handshake. This draft aims to describe methods for doing
so securely. This key material must be injected in such a way that both parties
agree on what is being injected and why, and further, in what order.

--- middle

# Introduction

Introducing additional key material into the TLS handshake is a non-trivial
process because both parties need to agree on the injection content and context.
If the two parties do not agree then an attacker may exploit the mismatch in
so-called channel synchronization attacks, such as those described by
{{?SLOTH=DOI.10.14722/ndss.2016.23418}}.

Injecting key material into the TLS handshake allows other protocols to be bound
to the handshake. For example, it may provide additional protections to the
ClientHello message, which in the standard TLS handshake only receives
protections after the server's Finished message has been received. It may also
permit the use of combined shared secrets, possibly from multiple key exchange
algorithms, to be included in the key schedule. This pattern is common for Post
Quantum key exchange algorithms, as discussed in
{{?I-D.ietf-tls-hybrid-design}}.

The goal of this document is to provide a standardised way for binding extra
context into TLS 1.3 handshakes in a way that is easy to analyse from a security
perspective, reducing the need for repeated security analyses for every
extension and combination of extensions. It separates the concerns of whether an
extension achieves its goals from the concerns of whether an extension reduces
the security of a TLS handshake, either directly or through some unforseen
interaction with another extension.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# Key Schedule Extension {#injection}

This section describes two places in which additional secrets can be injected
into the TLS 1.3 key schedule.

## Handshake Secret Injection

To inject extra key material into the Handshake Secret it is recommended to
prefix it, inside an appropriate frame, to the `(EC)DHE` input, where `||`
represents concatenation.

~~~
                                 |
                                 v
                           Derive-Secret(., "derived", "")
                                 |
                                 v
  KeyScheduleInput || (EC)DHE -> HKDF-Extract = Handshake Secret
                                 |
                                 v
~~~

## Main Secret Injection

To inject key material into the Main Secret it is recommended to prefix it,
inside an appropriate frame, to the `0` input.

~~~
                           |
                           v
                     Derive-Secret(., "derived", "")
                           |
                           v
  KeyScheduleInput || 0 -> HKDF-Extract = Main Secret
                           |
                           v
~~~

This structure mirrors the Handshake Injection point.

# Key Schedule Injection Negotiation

Applications which make use of additional key schedule inputs MUST define a
mechanism for negotiating the content and type of that input.  This input MUST
be framed in a KeyScheduleSecret struct, as defined in {{structure}}.
Applications must take care that any negotiation that takes place unambiguously
agrees a secret. It must be impossible, even under adversarial conditions, that
a client and server agree on the transcript of the negotiation, but disagree on
the secret that was negotiated.

# Key Schedule Extension Structure {#structure}

In some cases, protocols may require more than one secret to be injected at a
particular stage in the key schedule. Thus, we require a generic and extensible
way of doing so.  To accomplish this, we use a structure -- KeyScheduleInput --
that encodes well-ordered sequences of secret material to inject into the key
schedule. KeyScheduleInput is defined as follows:

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

Each secret included in a KeyScheduleInput structure has a type and
corresponding secret data.  Each secret MUST have a unique
KeyScheduleSecretType. When encoding KeyScheduleInput as the key schedule Input
value, the KeyScheduleSecret values MUST be in ascending sorted order. This
ensures that endpoints always encode the same KeyScheduleInput value when using
the same secret keying material.

# Security Considerations

[BINDEL] provides a proof that the concatenation approach in {{injection}} is
secure as long as either the concatenated secret is secure or the existing KDF
input is secure.

[[OPEN ISSUE: Is this guarantee sufficient? Do we also need to guarantee that a malicious prefix can't weaken the resulting PRF output?]]

# IANA Considerations

This document requests the creation of a new IANA registry: TLS KeyScheduleInput Types.
This registry should be under the existing Transport Layer Security (TLS) Parameters
heading. It should be administered under a Specification Required policy {{!RFC8126}}.

[[OPEN ISSUE: specify initial registry values]]

| Value  | Description      | DTLS-OK | Reference |
|:-------|:-----------------|:--------|:----------|
| TBD    | TBD              | TBD     | TBD       |

--- back

# Acknowledgments
{:numbered="false"}
We thank Karthik Bhargavan for his comments.
