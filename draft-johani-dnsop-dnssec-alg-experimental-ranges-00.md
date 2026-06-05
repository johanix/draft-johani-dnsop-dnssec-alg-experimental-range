---
title: "Experimental and Private-Use Ranges in the DNSSEC Algorithm Numbers Registry"
abbrev: "DNSSEC Algorithm Ranges"
category: std
docname: draft-johani-dnsop-dnssec-alg-experimental-ranges-00
submissiontype: IETF
ipr: trust200902
area: "Internet"
workgroup: "DNSOP Working Group"
updates: 6014, 9157
keyword:
 - DNSSEC
 - algorithm numbers
 - post-quantum
 - private use
 - experimental
venue:
  group: "Domain Name System Operations"
  type: "Working Group"
  mail: "dnsop@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/dnsop/"

author:
 -
    fullname: Johan Stenstam
    organization: The Swedish Internet Foundation
    email: johan.stenstam@internetstiftelsen.se

normative:
  RFC4034:
  RFC6014:
  RFC8126:
  RFC9157:

informative:
  RFC4035:
  RFC6840:
  I-D.johani-dnsop-dnssec-alg-split:
  FIPS204:
    title: "Module-Lattice-Based Digital Signature Standard"
    target: "https://csrc.nist.gov/pubs/fips/204/final"
    author:
      - org: "National Institute of Standards and Technology (NIST)"
    date: 2024-08
    seriesinfo:
      FIPS: "204"
  FIPS205:
    title: "Stateless Hash-Based Digital Signature Standard"
    target: "https://csrc.nist.gov/pubs/fips/205/final"
    author:
      - org: "National Institute of Standards and Technology (NIST)"
    date: 2024-08
    seriesinfo:
      FIPS: "205"

--- abstract

The DNSSEC Algorithm Numbers registry contains two code points, 253
(PRIVATEDNS) and 254 (PRIVATEOID), intended for private and
experimental algorithms. These code points identify the algorithm by
prepending a domain name or an object identifier to the "Public Key"
field of the DNSKEY RDATA, overloading the semantics of that field and
forcing every implementation to special-case algorithm dispatch.
Because all such algorithms share a single code point, they cannot be
distinguished on the wire by the algorithm number alone.

This document reserves two small ranges of DNSSEC algorithm numbers:
one for Private Use, requiring no IANA registration, and one for
experimental algorithms, registered on a First Come First Served basis.
Code points drawn from these ranges behave identically to ordinary
algorithm numbers and require no overloading of the DNSKEY RDATA. This
enables clean experimentation with the growing set of post-quantum and
other candidate signature algorithms. This document updates RFC 6014
and RFC 9157.

--- middle

# Introduction

The "DNS Security Algorithm Numbers" registry {{RFC4034}} assigns the
values used in the Algorithm field of the DNSKEY, RRSIG, and related
resource records. Its allocation policy was established by {{RFC6014}}
and subsequently revised by {{RFC9157}}; values in the range 123-251
are currently marked "Reserved".

Two code points are set aside for non-standardized algorithms:

* 253 (PRIVATEDNS), where the algorithm is identified by a domain name,
  and
* 254 (PRIVATEOID), where the algorithm is identified by an ISO Object
  Identifier (OID).

As detailed in {{insufficient}}, these two code points are poorly
suited to the kind of experimentation the community now needs. There
are two converging reasons why a cleaner mechanism is required now:

1. The transition to post-quantum cryptography (PQC) is driving active
   experimentation in DNS. Whether or not PQC migration becomes urgent
   for DNSSEC in the near term, the experimentation is already
   happening.

2. A large and growing set of candidate signature algorithms is under
   evaluation and discussion. NIST has standardized ML-DSA {{FIPS204}}
   and SLH-DSA {{FIPS205}}, with FN-DSA (Falcon) and additional
   families following, and several other candidates (for example MAYO
   and SNOVA) remain under study. Implementers need to deploy and
   interoperate with several such algorithms concurrently, well before
   any of them warrants a permanent, standardized code point.

The experimental range defined here is anticipated to be useful in
particular for the kind of large-KSK algorithm experimentation
described in {{?I-D.johani-dnsop-dnssec-alg-split}}, where
post-quantum candidate algorithms with different size and strength
trade-offs are evaluated in real deployments.

This document carves two small ranges out of the Reserved block of the
DNSSEC Algorithm Numbers registry: an experimental range registered on
a First Come First Served basis, and a Private Use range that requires
no IANA action. Algorithms using code points from these ranges are
dispatched by algorithm number alone, exactly like standardized
algorithms, and impose no special parsing on the DNSKEY RDATA.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Why Code Points 253 and 254 Are Insufficient {#insufficient}

Code points 253 and 254 do not change the DNSKEY wire format: the RDATA
remains Flags \| Protocol \| Algorithm \| Public Key. Instead, per
Appendix A.1.1 of {{RFC4034}}, the discriminator (a domain name for 253,
or an OID for 254) is *prepended to* the bytes that the format calls the
"Public Key" field. The semantics of that field are thereby overloaded:
the leading bytes are not key material at all.

This design has several practical drawbacks that are amplified when more
than one private algorithm is in use:

Single code point for many algorithms:
: All algorithms that use 253 (or 254) share that one code point.
  Software that dispatches coarsely on the algorithm number cannot tell
  them apart; they are all "253" or "254". An operator running several
  concurrent algorithms (for example, six post-quantum candidates)
  cannot place them in the same zone and distinguish them by code point.

Hot-path special-casing:
: For 253/254 an implementation cannot simply switch on the algorithm
  number to select a verifier. On every operation touching a DNSKEY it
  must first parse the prepended name or OID, look it up in an
  implementation-private table mapping names/OIDs to crypto code, strip
  those bytes, and only then hand the remainder to the verifier. This
  logic lands on the signing and validation hot path.

RRSIG/DNSKEY out-of-band agreement:
: The RRSIG carries only the algorithm number (253/254), not the
  discriminator. To verify, an implementation must fetch the DNSKEY,
  extract the embedded name/OID, and confirm it matches the algorithm
  the signer used; the signer and verifier must agree on the private
  algorithm out of band of the wire format.

Wire overhead:
: Carrying a domain name or OID inside every DNSKEY is modest in
  absolute terms, but it is pure overhead, and it must be accounted for
  by every producer and consumer of the record.

Tooling impact:
: Key file formats, test harnesses, and operational tooling must all
  treat 253/254 specially, because "algorithm 254" alone does not
  identify a keypair.

The net effect is that an implementation supporting several private
algorithms via 253/254 pays a permanent special-casing tax internally,
while still gaining no interoperability outside its own deployment,
since general-purpose validators that implement 253/254 are rare. A
small range of ordinary code points avoids all of this.

# Allocation of Experimental and Private-Use Code Points

This document designates two ranges within the DNSSEC Algorithm Numbers
registry. Code points in either range are used in the Algorithm field
exactly as standardized algorithm numbers are: dispatch is by code point
alone, and the DNSKEY "Public Key" field carries only key material.

## Experimental Range

The range 228-243 is designated for experimental algorithms and is
registered on a First Come First Served basis (Section 4.7 of
{{RFC8126}}).

Code points in this range are recorded by IANA, so that independent
experimenters can choose distinct code points and run interoperability
tests without colliding. Registration is deliberately low-friction: a
registrant supplies a short description and a point of contact (see
{{iana}}).

Code points in this range are for experimentation and interoperability
testing only. An algorithm that proves valuable for general use is
expected to be assigned a code point under the registry's normal policy
{{RFC9157}}; the experimental code point is then expected to be
deprecated. Experimental code points MUST NOT be relied upon for
production deployments, and entries MAY be removed or reassigned.

## Private Use Range

The range 244-251 is designated for Private Use, as defined in
Section 4.1 of {{RFC8126}}. No IANA registration is made for these
values, and no IANA action is required to use one.

Private Use code points are intended for use within a single
administrative domain (one operator, one test lab, one deployment). The
mapping from a Private Use code point to an actual algorithm is a local
matter. Unlike the Experimental range, no central registry exists, so
two deployments MAY use the same Private Use code point for different
algorithms; implementations MUST NOT assume any particular meaning for
a Private Use code point across administrative boundaries.

This range is appropriate for an operator that wishes to run several
algorithms concurrently and distinguish them by code point, without any
coordination overhead.

## Relationship to RFC 8126 "Experimental Use" {#exp-clarification}

{{RFC8126}} defines an "Experimental Use" policy (Section 4.2) under
which, as with Private Use, IANA makes no assignments. This document
deliberately does *not* use that policy for the experimental range.
The goal of the experimental range is shared, collision-free
interoperability testing across multiple parties, which requires that
IANA record the assignments; the First Come First Served policy provides
exactly that with minimal friction. The Private Use range in
{{RFC8126}} terms covers the no-registration case.

## Relationship to Code Points 252, 253, and 254

This document does not change the meaning of code points 252
(INDIRECT), 253 (PRIVATEDNS), or 254 (PRIVATEOID). Implementations MAY
continue to use 253 and 254. The ranges defined here are an alternative
that avoids overloading the DNSKEY "Public Key" field and allows several
private or experimental algorithms to coexist on the wire,
distinguishable by code point alone.

# Operational Considerations

A validating resolver that does not recognize the algorithm of an RRSIG
or DNSKEY treats the corresponding data as unsigned/insecure, per
Section 5.2 of {{RFC4035}}. Consequently, a zone signed solely with a
Private Use or experimental algorithm is insecure from the perspective
of any resolver outside the deployment that does not implement that
algorithm. This is the intended and desirable behavior for private and
experimental use.

When a zone is signed with multiple algorithms, the multi-algorithm
rules in Section 5.11 of {{RFC6840}} apply. A validator that does not
recognize an experimental or Private Use algorithm in the DNSKEY RRset
can still validate the zone using any standardized algorithm present in
the same RRset; the unknown algorithm does not, by itself, render the
zone bogus.

Within an experimental or private deployment, code points from these
ranges will appear in the Algorithm field of DS records published at the
parent, exactly as standardized algorithm numbers do. A validating
resolver outside the deployment that does not recognize the algorithm
will treat the delegation as insecure, as it would for any unknown
algorithm.

# IANA Considerations {#iana}

IANA is requested to update the "DNS Security Algorithm Numbers"
registry within the "Domain Name System Security (DNSSEC) Algorithm
Numbers" registry group.

The values 228-251 are currently part of the "Reserved" block (123-251,
{{RFC4034}} {{RFC6014}}). IANA is requested to remove these values from
that Reserved block and to record them as follows. After this change,
the Reserved block is 123-227.

| Number  | Description                | Mnemonic | Reference       |
|---------|----------------------------|----------|-----------------|
| 228-243 | Experimental algorithms    |          | (this document) |
| 244-251 | Private Use                |          | (this document) |
{: title="Updated DNSSEC Algorithm Numbers entries"}

For the range 228-243 ("Experimental algorithms"), the registration
procedure is First Come First Served {{RFC8126}}. Each registration
request must include:

* the requested code point (or a request that IANA assign the next
  available value in the range);
* a short descriptive name for the algorithm;
* a brief description and/or a stable reference; and
* a point of contact.

Registrations in this range are understood to be experimental: entries
MAY be deprecated, removed, or reassigned, and assignment of a code
point does not imply any standardization status.

For the range 244-251 ("Private Use"), the policy is Private Use
{{RFC8126}}; IANA makes no assignments and takes no further action for
these values.

IANA is requested to add this document as a reference for the "DNS
Security Algorithm Numbers" registry, alongside {{RFC6014}} and
{{RFC9157}}.

# Security Considerations

This document allocates code point ranges and does not define or endorse
any cryptographic algorithm; the security properties of any algorithm
that uses a code point from these ranges are out of scope.

A validator that does not implement the algorithm associated with a
Private Use or experimental code point treats the data as insecure
(Section 5.2 of {{RFC4035}}). This containment property is intentional:
data signed with a private or experimental algorithm cannot be mistaken
for securely validated data by a resolver that does not understand the
algorithm.

Because Private Use code points carry no globally agreed meaning, a code
point that denotes a strong algorithm in one deployment may denote a
different (possibly weaker, broken, or absent) algorithm in another.
Implementations MUST NOT rely on Private Use code points across
administrative boundaries, and operators MUST ensure that Private Use
code points and the associated key material do not leak beyond the
deployment in which they are meaningful.

Experimental algorithms have, by definition, received limited analysis.
They MUST NOT be used to protect production data on the assumption that
they provide the security of a standardized algorithm.

The multi-algorithm considerations of Section 5.11 of {{RFC6840}} apply
when a zone mixes algorithms from these ranges with standardized
algorithms; in particular, the presence of an unknown algorithm in a
DNSKEY RRset does not on its own make a zone bogus.

--- back

# Acknowledgments
{:numbered="false"}

The author thanks Peter Thomassen (deSEC), Ondřej Surý (Internet
Systems Consortium), Benno Overeinder (NLnet Labs), and Erik
Bergström (Swedish Internet Foundation) for valuable insights and
discussions on the need for distinct experimental algorithm code
points.
