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

normative:
  DIGEST: RFC9530
  PROBLEM: RFC9457

informative:


--- abstract

TODO Abstract


--- middle

# Introduction

Digest fields {{DIGEST}} are HTTP fields that support integrity digests. A request can include the `Content-Digest` and `Repr-Digest` header fields for verifying the integrity of the HTTP message content and the HTTP representation, respectively. In addition, a sender can include the `Want-Content-Digest` and `Want-Repr-Digest` header fields in a request to express interest in receiving integrity field in the response. {{!RFC9530}} by design does not define, require or recommend specific resource behavior if errors regarding the integrity appear.

For example, a request may include a digest algorithm in the `Content-Digest` and `Repr-Digest` header fields that the resource does not support. Similar, a sender may request to the digest utilizing a hashing algorithm that the resource does not support. Another possible problem is that the digest supplied in the request does not match up with the digest calculated by the resource. Depending on the application, the resource may choose to ignore these errors or communicate them back to the client. However, no recommended response format for communicating these error is defined so far.

Problem types {{PROBLEM}} are machine-readable description of errors in HTTP response content {{PROBLEM}}. Each problem definition includes a unique type that can be used to identify the error and also allows the attachment of a short, human-readable summary as well as additional properties to aid debugging and error handling. In addition, a JSON and XML representation of the problem types is defined to simplify parsing.

As an example, if the resource receives a request with an integrity field utilizing an unsupported hashing algorithm `foo`, the response may use the following problem type:

~~~ http-message
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json

{
  "type": "https://iana.org/assignments/http-problem-types#unsupported-hashing-algorithm",
  "title": "upload is already completed",
  "requested-algorithm": "foo",
  "supported-algorithms": ["sha-256", "sha-512"]
}
~~~

The response includes the unique problem type, the requested algorithm that is not supported by the resource, as well as an array of the supported algorithms.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The terms "integrity fields" and "integrity preference fields" are from {{DIGEST}}.

# Problem Types

## Unsupported Hashing Algorithm

This section defines the `https://iana.org/assignments/http-problem-types#unsupported-hashing-algorithm` problem type {{PROBLEM}}. A resource MAY use this problem type in a response to a request, whose integrity or integrity preference fields reference a hashing algorithm that the resource can not or does not want to support for this request, and if the resource wants to indicate this problem to the sender.

The resource SHOULD provide the algorithm key of the unsupported algorithm in the `unsupported-algorithm` member and an array of the supported algorithms in the `supported-algorithm` member. The value of this array are algorithm keys as registered in the "Hash Algorithms for HTTP Digest Fields" registry.

The following example shows a response for a request with an integrity field utilizing an unsupported hashing algorithm `foo`. The response also includes a list of supported algorithms.

~~~ http-message
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json

{
  "type": "https://iana.org/assignments/http-problem-types#unsupported-hashing-algorithm",
  "title": "upload is already completed",
  "unsupported-algorithm": "foo",
  "supported-algorithms": ["sha-256", "sha-512"]
}
~~~

# Security Considerations

Although an error appeared while handling the digest fields, the resource may choose to not disclose this error to the sender to avoid lacking implementation details. Similar, the resource may choose a general problem type for the response even in a more specific problem type is defined if it prefers to hide the details of the error from the sender.

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

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
