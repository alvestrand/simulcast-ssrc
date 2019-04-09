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
per simulcast stream, formatted as `a=ssrc:<ssrc> cname:<cname>`. The
cname carries no helpful information, but is required by the "a=ssrc" syntax.

The list of SSRCs used is declared in an attribute with the FID
(Flow Identification) semantic, as defined in {{RFC5888}}.

The order of SSRCs in the a=ssrc-group attribute MUST match the order of the
rid attributes in the corresponding streams in the "send" part of the
a=simulcast: attribute.

It is RECOMMENDED that both the a=rid: attributes and the a=ssrc: attributes
appear in the same order as the order in the a=simulcast and a=ssrc-group
attributes.

It is RECOMMENDED not to use RTX with this configuration, since the inclusion
of the required declarations for associating RTX SSRCs with their main SSRCs
would make the SDP unwieldy and hard to interpret correctly.

# How to request that SSRC information be included

If an SFU wishes to request that a browser send SSRC information, it should
send an offer containing the line "a=x-please-send-ssrcs", together with a
line requesting simulcast:

~~~~
m=video
a=simulcast:recv a,b,c
a=x-please-send-ssrcs
~~~~

The SFU can detect whether the request has been honored by looking for
a=ssrc attributes in the responding answer.

If a Javascript application wishes to request that the browser generate
offers containing SSRC, it should include the non-standard attribute
"showSsrcInSimulcastOffer" in the RTCPeerConnection constructor:

~~~~
pc = new RTCPeerConnection({showSsrcInSimulcastOffer: true})
~~~~

It is possible to verify that the request is understood by checking for
the presence of this attribute in the RTCPeerConnection parameters:

~~~~
if (showSsrcInSimulcastOffer in pc.getConfiguration) {
   // the request has been understood correctly
}
~~~~

# Example

~~~~
m=video
a=simulcast:send hi,mid,low
a=rid:hi
a=rid:mid
a=rid:low
a=ssrc-group:FID 123 456 789
a=ssrc:123 cname:foo
a=ssrc:456 cname:foo
a=ssrc:789 cname:foo
~~~~

# Sunsetting the interim measure

This specification is intended to give SFU authors time to convert
to the new mechanism. Since the invocation of this mechanism is explicit,
it is easy to check on what the usage is, and emit deprecation warnings;
those should probably be emitted from day 1.

Once enough time has passed, this mechanism can be removed.

# Security Considerations

This document describes two existing mechanisms: a=simulcast and
a=ssrc-group. Each of these is defined in an RFC with security considerations.

The only added attack surface here is the ability to create mismatches
between the two lists of simulcast RTP streams, causing different
implementations to choose different streams to display. This is a special
instance of the general rule that "people who can modify your SDP can mess
things up"; normal precautions when passing SDP around should be adequate.

# IANA Considerations

This document has no IANA actions. There is especially no request for
IANA to register the "a=x-please-send-ssrc" attribute, since this temporary
attribute assignment is designed to go away when the transition period is over.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

