---
title: "HTTP Problem Types for Digest Fields"
category: info

docname: draft-kleidl-digest-fields-problem-types-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "tus/draft-digest-fields-problem-types"
  latest: "https://tus.github.io/draft-digest-fields-problem-types/draft-kleidl-digest-fields-problem-types.html"

author:
 -
    fullname: "Marius Kleidl"
    organization: Transloadit
    email: "marius@transloadit.com"
 -
    fullname: "Lucas Pardue"
    organization: Cloudflare
    email: "lucas@lucaspardue.com"

normative:
  DIGEST: RFC9530
  PROBLEM: RFC9457
  STRUCTURED-FIELDS: RFC8941

informative:


--- abstract

This document specifies problem types that servers can use in responses to problems encountered while dealing with a request carrying integrity fields and integrity preference fields.

--- middle

# Introduction

{{DIGEST}} by design does not define, require or recommend any specific behavior for error handling relating to integrity. The responsibility is instead delegated to applications. This draft defines a set of problem types {{PROBLEM}} that can be used by server applications to indicate that a problem was encountered while dealing with a request carrying integrity fields and integrity preference fields.

For example, a request message may include content alongside `Content-Digest` and `Repr-Digest` header fields that use a digest algorithm the server does not support. An application could decide to reject this request because it cannot validate the integrity. Using a problem type, the server can provide machine-readable error details to aid debugging or error reporting, as shown in the following example.

~~~ http-message
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json
Want-Content-Digest: sha-512=3, sha-256=10

{
  "type": "https://iana.org/assignments/http-problem-types#unsupported-hashing-algorithm",
  "title": "hashing algorithm is not supported",
  "unsupported-algorithm": "foo"
}
~~~

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The terms "integrity fields" and "integrity preference fields" are from {{DIGEST}}.

# Problem Types

## Unsupported Hashing Algorithm

This section defines the "https://iana.org/assignments/http-problem-types#unsupported-hashing-algorithm" problem type {{PROBLEM}}. A server MAY use this problem type when responding to a request, whose integrity or integrity preference fields reference a hashing algorithm that the server can not or does not want to support for this request, and if the server wants to indicate this problem to the sender.

For this problem type, the `unsupported-algorithm` is defined as the only extension member. It SHOULD be populated in a response using this problem type, with its value being the algorithm key of the unsupported algorithm from the request. The response SHOULD include the corresponding integrity preference field to indicate the server's algorithm support and preference.

The following example shows a response for a request with an integrity field utilizing an unsupported hashing algorithm `foo`. The response also includes a list of supported algorithms.

~~~ http-message
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json
Want-Content-Digest: sha-512=3, sha-256=10

{
  "type": "https://iana.org/assignments/http-problem-types#unsupported-hashing-algorithm",
  "title": "hashing algorithm is not supported",
  "unsupported-algorithm": "foo"
}
~~~

This problem type is a hint to the client about algorithm support, which the client could use to retry the request with a different algorithm supported by the server.

## Invalid Digest Value

This section defines the "https://iana.org/assignments/http-problem-types#invalid-digest-value" problem type {{PROBLEM}}. A server MAY use this problem type when responding to a request, whose integrity fields include a digest value, that cannot be generated by the corresponding hashing algorithm. For example, if the digest value of the `sha-512` hashing algorithm is not 64 bytes long, it cannot be a valid digest value and the server can skip computing the digest value. This problem type MUST NOT be used if the server is not able to parse the integrity fields according to {{Section 4.5 of STRUCTURED-FIELDS}}, for example because of a syntax error in the field value.

The server SHOULD include a human-readable description why the value is considered invalid in the `title` member.

The following example shows a response for a request with an invalid digest value.

~~~ http-message
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json

{
  "type": "https://iana.org/assignments/http-problem-types#invalid-digest-value",
  "title": "digest value for sha-512 is not 64 bytes long"
}
~~~

This problem type indicates a fault in the sender's calculation or encoding of the digest value. A retry of the same request without modification will likely not yield a successful response.


## Mismatching Digest Value

This section defines the "https://iana.org/assignments/http-problem-types#mismatching-digest-value" problem type {{PROBLEM}}. A server MAY use this problem type when responding to a request, whose integrity fields include a digest value that does not match the digest value that the server calculated for the request content or representation.

Three problem type extension members are defined: the `algorithm`, `provided-digest`, and `calculated-digest` members. A response using this problem type SHOULD populate all members, with the value of `algorithm` being the algorithm key of the used hashing algorithm, with the value of `provided-digest` being the digest value taken from the request's integrity fields, and the value of `calculated-digest` being the calculated digest. The digest values MUST BE serialized as byte sequences as described in {{Section 4.1.8 of STRUCTURED-FIELDS}}.

The following example shows a response for a request with a mismatching SHA-256 digest value.

~~~ http-message
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json

{
  "type": "https://iana.org/assignments/http-problem-types#mismatching-digest-value",
  "title": "digest value fromr request does not match expected value",
  "algorithm": "sha-256",
  "provided-digest": ":RK/0qy18MlBSVnWgjwz6lZEWjP/lF5HF9bvEF8FabDg=:",
  "calculated-digest": ":d435Qo+nKZ+gLcUHn7GQtQ72hiBVAgqoLsZnZPiTGPk=:"
}
~~~

If the sender receives this problem type, the request might be modified unintentionally by an intermediary. The sender could use this information to retry the request without modification to address temporary transmission issues.

# Security Considerations

Although an error appeared while handling the digest fields, the server may choose to not disclose this error to the sender to avoid lacking implementation details. Similar, the server may choose a general problem type for the response even in a more specific problem type is defined if it prefers to hide the details of the error from the sender.

# IANA Considerations

IANA is asked to register the following entry in the "HTTP Problem Types" registry:

Type URI:
: https://iana.org/assignments/http-problem-types#unsupported-hashing-algorithm

Title:
: Unsupported Hashing Algorithm

Recommended HTTP status code:
: 400

Reference:
: This document

IANA is asked to register the following entry in the "HTTP Problem Types" registry:

Type URI:
: https://iana.org/assignments/http-problem-types#invalid-digest-value

Title:
: Invalid Digest Value

Recommended HTTP status code:
: 400

Reference:
: This document

IANA is asked to register the following entry in the "HTTP Problem Types" registry:

Type URI:
: https://iana.org/assignments/http-problem-types#mismatching-digest-value

Title:
: Mismatching Digest Value

Recommended HTTP status code:
: 400

Reference:
: This document

--- back

# Acknowledgments
{:numbered="false"}

This document is based on ideas from a discussion with Roberto Polli, so thanks to him for his valuable input and feedback on this topic.