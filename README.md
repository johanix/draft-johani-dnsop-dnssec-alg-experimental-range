# draft-johani-dnsop-dnssec-alg-experimental-ranges

IETF Internet-Draft:
**Experimental and Private-Use Ranges in the DNSSEC Algorithm
Numbers Registry**

## Summary

The DNSSEC Algorithm Numbers registry has only two code points for
non-standardized algorithms, 253 (PRIVATEDNS) and 254 (PRIVATEOID).
Both identify the algorithm by prepending a domain name or an OID to
the DNSKEY "Public Key" field, overloading that field's semantics and
forcing every implementation into special-case dispatch. Worse, all
private algorithms share a single code point, so they cannot be told
apart on the wire by algorithm number alone.

This draft carves two small ranges out of the Reserved block of the
registry:

* an **Experimental** range (228-243, registered First Come First
  Served, for collision-free shared interoperability testing across
  independent implementations), and
* a **Private Use** range (244-251, no IANA action, single-deployment
  use).

Code points from these ranges behave like ordinary algorithm numbers:
dispatch is by code point alone and the DNSKEY RDATA is not overloaded.
This supports clean experimentation with the growing set of
post-quantum and other candidate signature algorithms.

This document updates RFC 6014 and RFC 9157.

## Authors

* Johan Stenstam &lt;johan.stenstam@internetstiftelsen.se&gt;

(The Swedish Internet Foundation / Internetstiftelsen)

## Current version

[draft-johani-dnsop-dnssec-alg-experimental-ranges-00.md](draft-johani-dnsop-dnssec-alg-experimental-ranges-00.md)
— status: Internet-Draft, Standards Track, -00.

## Building the draft

The source is [kramdown-rfc](https://github.com/cabo/kramdown-rfc)
markdown. To produce text and HTML renderings:

```
make
```

(Requires `kramdown-rfc` and `xml2rfc` in `$PATH`.)

## Status

Working draft. Feedback is welcome via GitHub issues or the
`dnsop@ietf.org` mailing list once the draft has been submitted to
the datatracker.

## License

This document is subject to the BCP 78 and the IETF Trust's Legal
Provisions Relating to IETF Documents
(https://trustee.ietf.org/license-info) in effect on the date of
publication of this document.
