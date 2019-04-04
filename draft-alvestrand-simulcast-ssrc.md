---
title: Using SSRC with WebRTC Simulcast
abbrev: Simulcast SSRC
docname: draft-alvestrand-simulcast-ssrc-latest
category: info

ipr: trust200902
area: General
workgroup: RTCWEB Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: "H. Alvestrand"
    name: "Harald Alvestrand"
    organization: "Google"
    email: harald@alvestrand.no

normative:
  RFC2119:
  RFC5576:
  RFC5888:
  I-D.ietf-avtext-rid:
  I-D.ietf-mmusic-rid:

informative:



--- abstract

This document describes a convention for sending "a=ssrc" attributes
in SDP together with "a=simulcast" attributes. This allows SFUs that
need SSRC information to have this info easily accessible.

--- middle

# Introduction

In developing the WebRTC specification, the IETF decided on a
form of simulcast that doesn't require fixed SSRC allocation, but rather
used a combination of SDP tags (a=rid) {{I-D.ietf-mmusic-rid}} and RTP header
extensions (RTPStreamId) {{I-D.ietf-avtext-rid}}
to describe the mapping between simulcast layers and RTP streams.

This posed a problem for some SFUs, which required information on what
SSRCs the incoming streams were going to appear on in order to be configured
correctly.

This document gives a convention for adding information about SSRCs to the
SDP produced by conformant WebRTC implementations in order to make this
information available.

This document does not specify an Internet standard. It is an interim
measure, intended to be useful in the time between the introduction of
RID-based simulcast in browsers and the full support of RID-based simulcast
by SFUs.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# How to represent SSRC information

The syntax for representing SSRC information is taken from {{RFC5576}}.

Each media section in the SDP contains one a=ssrc attribute
per simulcast stream, formatted as `a=ssrc:<ssrc> cname <cname>`. The
cname carries no helpful information, but is required by the "a=ssrc" syntax.

The list of SSRCs used is declared in an attribute with the FID
(Flow Identification) semantic, as defined in {{RFC5888}}.

The order of SSRCs in the a=ssrc-group attribute MUST match the order of the
rid attributes in the corresponding streams in the a=simulcast: attribute.

It is RECOMMENDED that both the a=rid: attributes and the a=ssrc: attributes
appear in the same order as the order in the a=simulcast and a=ssrc-group
attributes.

# Example

~~~~
m=video
a=simulcast:send hi,mid,low
a=rid:hi
a=rid:mid
a=rid:low
a=ssrc-group:FID:123,456,789
a=ssrc:123:cname foo
a=ssrc:456:cname foo
a=ssrc:789:cname foo
~~~~

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.



--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

