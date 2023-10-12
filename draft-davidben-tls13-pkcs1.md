---
title: "Legacy RSASSA-PKCS1-v1_5 codepoints for TLS 1.3"
abbrev: "Legacy PKCS#1 codepoints for TLS 1.3"
category: exp

docname: draft-davidben-tls13-pkcs1-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "davidben/tls13-pkcs1"
  latest: "https://davidben.github.io/tls13-pkcs1/draft-davidben-tls13-pkcs1.html"

author:
 -
    ins: "D. Benjamin"
    name: "David Benjamin"
    organization: "Google LLC"
    email: davidben@google.com

normative:
  RFC2119:
  RFC8017:
  RFC8446:
  X690:
    title: "Information technology - ASN.1 encoding Rules: Specification of Basic Encoding Rules (BER), Canonical Encoding Rules (CER) and Distinguished Encoding Rules (DER)"
    date: 2002
    author:
      org: ITU-T
    seriesinfo:
      ISO/IEC: 8825-1:2002

informative:
  MFSA201473:
    title: "RSA Signature Forgery in NSS"
    author:
      ins: "A. Delignat-Lavaud"
      name: "Antoine Delignat-Lavaud"
    date: 2014-09-23
    target: https://www.mozilla.org/en-US/security/advisories/mfsa2014-73/
  POODLE:
    title: "This POODLE bites: exploiting the SSL 3.0 fallback"
    author:
      ins: "B. Moeller"
      name: "Bodo Moeller"
    date: 2014-10-14
    target: https://security.googleblog.com/2014/10/this-poodle-bites-exploiting-ssl-30.html



--- abstract

This document allocates code points for the use of RSASSA-PKCS1-v1\_5 with
client certificates in TLS 1.3.

--- middle

# Introduction

TLS 1.3 {{RFC8446}} removed support for RSASSA-PKCS1-v1\_5 {{RFC8017}} in
CertificateVerify messages in favor of RSASSA-PSS. While RSASSA-PSS is a
long-established signature algorithm, some legacy hardware cryptographic devices
lack support for it. Due to performance requirements, such devices are uncommon
in TLS servers, but are sometimes used by TLS clients for client certificates.
Moreover, TLS negotiates the protocol version before client certificates, so
this limitation can further impact adjacent connections that do not use affected
keys.

This document allocates code points to use these legacy keys with client
certificates in TLS 1.3.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# PKCS#1 v1.5 SignatureScheme Types

The following SignatureScheme values are defined for use with TLS 1.3.

~~~~
    enum {
        rsa_pkcs1_sha256_legacy(TBD1),
        rsa_pkcs1_sha384_legacy(TBD2),
        rsa_pkcs1_sha512_legacy(TBD3),
    } SignatureScheme;
~~~~

The above code points indicate a signature algorithm using RSASSA-PKCS1-v1\_5
{{RFC8017}} with the corresponding hash algorithm as defined in
{{!SHS=DOI.10.6028/NIST.FIPS.180-4}}. They are only defined for signatures in
the client CertificateVerify message and are not defined for use in other
contexts. In particular, servers intending to advertise support for
RSASSA-PKCS1-v1\_5 signatures in the certificates themselves should use the
rsa\_pkcs1\_\* constants defined in {{RFC8446}}.

Clients MUST NOT advertise these values in the "signature\_algorithms" extension
of the ClientHello. They MUST NOT accept these values in the server
CertificateVerify message.

Servers that wish to support clients authenticating with legacy
RSASSA-PKCS1-v1\_5-only keys MAY send these values in the
"signature\_algorithms" extension of the CertificateRequest message and accept
them in the client CertificateVerify message. Servers MUST NOT accept these code
points if not offered in the CertificateRequest message.

Clients with such legacy keys MAY negotiate the use of these signature
algorithms if offered by the server.  Clients SHOULD NOT negotiate them with
keys that support RSASSA-PSS.

TLS implementations SHOULD disable these code points by default.


# Security Considerations

Prior to this document, legacy RSA keys would prevent client certificate
deployments from adopting TLS 1.3. The new code points allow such deployments
to upgrade without replacing the keys. TLS 1.3 fixes a privacy flaw
{{?PRIVACY=DOI.10.23919/TMA.2017.8002897}} with client certificates, so
upgrading is a particular benefit to these deployments.

Additionally, TLS negotiates protocol versions before client certificates. When
sending a ClientHello, a TLS-1.3-capable client cannot determine if the server
will request a legacy key. It may then offer TLS 1.3, to upgrade connections to
other servers. A TLS-1.3-capable server that requests client certificates cannot
then distinguish such a client from one with modern keys. It may then negotiate
TLS 1.3 and send a CertificateRequest. The connection would then fail due to the
legacy key, when it previously succeeded at TLS 1.2.

To recover from this failure, one side must globally disable TLS 1.3 or the
client must implement an external fallback. Disabling TLS 1.3 impacts
connections that would otherwise be unaffected by this issue, while external
fallbacks break TLS's security analysis and may introduce vulnerabilities
{{POODLE}}. The new code points reduce the pressure on implementations to select
one of these mitigations.

However, the new code points also reduce the pressure on implementations to
migrate to RSASSA-PSS. The above considerations do not apply to server keys, so
these new code points are forbidden for use with server certificates. RSASSA-PSS
continues to be required for TLS 1.3 servers using RSA keys.

Finally, when implemented incorrectly, RSASSA-PKCS1-v1\_5 admits signature
forgeries {{MFSA201473}}. Implementations  producing or verifying signatures
with these algorithms MUST implement RSASSA-PKCS1-v1\_5 as specified in section
8.2 of {{RFC8017}}. In particular, clients MUST include the mandatory NULL
parameter in the DigestInfo structure and produce a valid DER {{X690}}
encoding. Servers MUST reject signatures which do not meet these requirements.


# IANA Considerations

IANA is requested to create the following entries in the
TLS SignatureScheme registry, defined in {{!RFC8446}}. The "Recommended" column
should be set to "N", and the "Reference" column should be set to this document.

| Value |  Description                       |
|-------|------------------------------------|
| TBD1  | rsa\_pkcs1\_sha256\_legacy |
| TBD2  | rsa\_pkcs1\_sha384\_legacy |
| TBD3  | rsa\_pkcs1\_sha512\_legacy |


--- back
