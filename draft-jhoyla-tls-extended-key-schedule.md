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
    email: chriswood@cloudflare.com

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
so-called channel synchronization attacks.

Injecting key material into the TLS handshake allows other protocols to be bound
to the handshake. For example, it may provide additional protections to the
ClientHello message, which in the standard TLS handshake only receives
protections after the server's Finished message has been received. It may also
permit the use of combined shared secrets, possibly from multiple key exchange
algorithms, to be included in the key schedule. This pattern is common for Post
Quantum key exchange algorithms, as discussed in
{{?I-D.stebila-tls-hybrid-design}}.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# Key Schedule Extension

This section describes two places in which additional secrets can be injected into
the TLS 1.3 key schedule.

## Handshake Secret Injection

To inject extra key material into the Handshake Secret it is recommended to prefix it, inside an appropriate frame, to the `(EC)DHE` input, where `||` represents concatenation.

~~~
                      |
                      v
                Derive-Secret(., "derived", "")
                      |
                      v
  Input || (EC)DHE -> HKDF-Extract = Handshake Secret
                      |
                      v
~~~

[BINDEL] provides a proof that this construction is secure as long as either the concatenated secret is secure or the PSK is secure. [[Is this guarantee sufficient? Do we also need to guarantee that a malicious prefix can't weaken the resulting PRF output?]]

##Master Secret Injection

To inject key material into the Master Secret it is recommended to use an extra
derive secret.

~~~
                 |
                 v
           Derive-Secret(., "derived", "")
                 |
                 v
   Input || 0 -> HKDF-Extract = Master Secret
                 |
                 v
~~~

This structure mirrors the Handshake Injection point, and is also supported by [BINDEL]

# Key Schedule Extension Structure

In some cases, protocols may require more than one secret to be injected at a particular
stage in the key schedule. Thus, we require a generic and extensible way of doing so.
To accomplish this, we use a structure -- KeyScheduleInput -- that encodes well-ordered
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
We thank Karthik Bhargavan for his comments.
