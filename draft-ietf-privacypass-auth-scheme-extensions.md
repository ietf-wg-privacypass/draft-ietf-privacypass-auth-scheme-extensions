---
title: "The PrivateToken HTTP Authentication Scheme Extensions Parameter"
abbrev: "PrivateToken Authentication Extensions"
category: std

docname: draft-ietf-privacypass-auth-scheme-extensions-latest
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
  github: "ietf-wg-privacypass/draft-ietf-privacypass-extensible-token"
  latest: "https://ietf-wg-privacypass.github.io/draft-ietf-privacypass-extensible-token/draft-ietf-privacypass-extensible-token.html"

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

This document specifies a new parameter for the "PrivateToken" HTTP authentication
scheme. This purpose of this parameter is to carry extensions for Privacy Pass
protocols that support public metadata.

--- middle

# Introduction

The primary Token structure in the "PrivateToken" HTTP authentication scheme
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

This document specifies a new parameter for the "PrivateToken" HTTP authentication
scheme for carrying extensions. This extensions parameter, otherwise referred to as
public metadata, is cryptographically bound to the Token structure via the Privacy
Pass issuance protocol.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# PrivateToken Extensions Parameter

As defined in {{Section 2.2 of AUTHSCHEME}}, the "PrivateToken" authentication
scheme defines one parameter, "token", which contains the base64url-encoded
Token struct. This document defines a new parameter, "extensions," which contains
the base64url-encoded representation of the following Extensions structure.
This document follows the default padding behavior described in
{{Section 3.2 of !RFC4648}}, so the base64url value MUST include padding.

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
    Extension extensions<0..2^16-1>;
} Extensions;
~~~

The contents of Extensions are a list of Extension values, each of which is a type-length-value
structure whose semantics are determined by the type. The type and length of each
extension are 2-octet integers, in network byte order. The length of the extensions
list is also a 2-octet integer, in network byte order.

Clients, Issuers, and Origins all agree on the content and encoding of this Extensions
structure, i.e., they agree on the same type-length-value list. The list MUST be ordered
by ExtensionType value, from 0 to 65535. Extension types MAY be repeated. The value of the
Extensions structure is used as-is when verifying the value of the corresponding "token" parameter
in the "PrivateToken" authentication header. As an example, Clients presenting this extension
parameter to origins would use an Authorization header field like the following:

~~~
Authorization: PrivateToken token="abc...", extensions="def..."
~~~

Future documents may specify extensions to be included in this structure.
Registration details for these extensions are in {{iana}}.

Each Privacy Pass issuance protocol, identified by a token type, specifies the structure
of the PrivateToken value to be used. Extensions are bound to the resulting tokens via the
issuance protocol. In particular, the value of an Extensions structure is provided as
metadata for the issuance protocol. Candidate issuance protocols are specified in
{{?PUBLIC-ISSUANCE=I-D.hendrickson-privacypass-public-metadata}}.

# Extensions Negotiation {#negotiation}

The mechanism Clients and Origins use to determine which set of extensions to provide
for redemption is out of scope for this document.

In some Privacy Pass deployments, the set
of extensions may be well known to Clients and Origins and thus do not require negotiation.
In other settings, negotiation may be required. However, negotiation can raise privacy
risks, especially if negotiation can be abused by Origins for partitioning Clients and
risking Origin-Client unlinkability. Some of these risks may be mitigated if all Clients
in a given redemption context respond to negotiation in the same manner. However, if
Clients have different observable behavior, e.g., if certain extension use is determined
by user choice, Origins can observe this differential behavior and therefore partition
Clients in a redemption context.

# Security Considerations

Privacy considerations for tokens that include additional information are discussed
in {{Section 6.1 of ?ARCHITECTURE=I-D.ietf-privacypass-architecture}}. Additional
considerations for use of extensions, including those that arise when deciding which
extensions to use, are described in {{negotiation}}.

# IANA Considerations {#iana}

IANA is requested to create a new "Privacy Pass PrivateToken Extensions" registry
in the "Privacy Pass Parameters" page to list possible extension values and their meaning.
Each extension has a two-byte type, so the maximum possible value is 0xFFFF = 65535.

Template:

- Type: The two-byte extension type
- Name: Name of the extension
- Value: Syntax and semantics of the extension
- Reference: Where this extension and its value are defined
- Notes: Any notes associated with the entry

New entries in this registry are subject to the Specification Required
registration policy ({{!RFC8126, Section 4.6}}). Designated experts need to
ensure that the extension is sufficiently clearly defined and, importantly,
has a clear description of the privacy implications of using the extension,
framed in the context of partitioning the client anonymity set as described
in {{Section 6.1 of ?ARCHITECTURE}}.

--- back

