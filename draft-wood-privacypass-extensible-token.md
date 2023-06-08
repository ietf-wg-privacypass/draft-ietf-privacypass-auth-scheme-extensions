---
title: "The Extensible PrivateToken for Privacy Pass"
abbrev: "Extensible PrivateToken"
category: std

docname: draft-wood-privacypass-extensible-token-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Privacy Pass"
keyword:
 - token
venue:
  group: "Privacy Pass"
  type: "Working Group"
  mail: "privacy-pass@ietf.org"
  github: "chris-wood/draft-wood-privacypass-extensible-token"
  latest: "https://chris-wood.github.io/draft-wood-privacypass-extensible-token/draft-wood-privacypass-extensible-token.html"

author:
 -
    fullname: Scott Hendrickson
    organization: Google
    email: "scott@shendrickson.com"
 -
    fullname: Christopher A. Wood
    organization: Cloudflare, Inc.
    email: caw@heapingbits.net

normative:

informative:


--- abstract

This document specifies an extensible token structure for the
PrivateToken HTTP authentication scheme. This token structure is
for Privacy Pass issuance protocols that support public metadata.

--- middle

# Introduction

The primary Token structure in the PrivateToken HTTP authentication scheme
{{!AUTHSCHEME=I-D.ietf-privacypass-auth-scheme}} is composed as follows:

~~~
struct {
    uint16_t token_type;
    uint8_t nonce[32];
    uint8_t challenge_digest[32];
    uint8_t token_key_id[Nid];
    uint8_t authenticator[Nk];
} Token;
~~~

Functionally, this structure conveys a single bit of information from the
issuance protocol: whether or not the token is valid (as indicated by a valid
authenticator value). This structure does not admit any additional information
to flow from the issuance protocol, including, for example, public metadata
that is incorporated into the issuance protocol.

This document specifies a new, extensible token structure that does allow
additional information from the issuance protocol to be included and
cryptographically bound to the token. It is meant to serve primarily as
the output of issuance protocols that incorporate public metadata.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Extensible PrivateToken

The extensible PrivateToken structure, denoted ExtensibleToken, is as follows.

~~~
struct {
    ExtensionType extension_type;
    opaque extension_data<0..2^16-1>;
} Extension;

enum {
    reserved(0),
    (65535)
} ExtensionType;

struct {
    uint16_t token_type;
    uint8_t nonce[32];
    uint8_t challenge_digest[32];
    Extension extensions<0..2^16-1>;
    uint8_t token_key_id[Nid];
    uint8_t authenticator[Nk];
} ExtensibleToken;
~~~

The contents of ExtensibleToken are identical to that of the Token structure
in {{Section 2.2 of AUTHSCHEME}} with the exception of the extensions field,
which is defined as follows:

- "extensions" is a list of Extension values, each of which is a type-length-value
  structure whose semantics are determined by the type. The type and length of each
  extension are 2-octet integers, in network byte order. The length of the extensions
  list is also a is a 2-octet integer, in network byte order.
  
Future documents may specify extensions to be included in this new token structure.
Registration details for these extensions are in {{iana}}.

Each Privacy Pass issuance protocol, identified by a token type, specifices the structure
of the PrivateToken value to be used. Issuance protocols that support public metadata
would therefore use the ExtensibleToken format.

# Security Considerations

Privacy considerations for tokens that include additional information are discussed
in {{Section 6.1 of ?ARCHITECTURE=I-D.ietf-privacypass-architecture}}. 

# IANA Considerations {#iana}

IANA is requested to create a new "Privacy Pass ExtensibleToken Extensions" registry
in the "Privacy Pass Parameters" page to list extensions for the ExtensibleToken structure.
Each extension has a two-byte type, so the maximum possible value is 0xFFFF = 65535.

Template:
- Type: The two-byte extension type
- Name: Name of the extension
- Reference: Where this extension and its value are defined
- Notes: Any notes associated with the entry

New entries in this registry are subject to the Specification Required
registration policy ({{!RFC8126, Section 4.6}}). Designated experts need to
ensure that the extension is sufficiently clearly defined and, importantly,
has a clear description about the privacy implications of using the extension
framed in the context of partitioning the client anonymity set as described
in {{Section 6.1 of ?ARCHITECTURE}}.

--- back

