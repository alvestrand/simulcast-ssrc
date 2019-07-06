---
title: Using SSRC with WebRTC Simulcast
abbrev: Simulcast SSRC
docname: draft-alvestrand-mmusic-simulcast-ssrc-latest
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
  I-D.ietf-mmusic-sdp-simulcast:

informative:
  I-D.ietf-rtcweb-rtp-usage:


--- abstract

This document describes a convention for sending "a=ssrc" attributes
in SDP together with "a=simulcast" attributes. This allows SFUs that
need SSRC information to have this info easily accessible.

Given that it is intended as an interim measure, it does not aim for
being published as an RFC.

--- middle

# Introduction

In developing the WebRTC specification, the IETF decided on a
form of simulcast that doesn't require fixed SSRC allocation, but rather
used a combination of SDP tags (a=rid) {{I-D.ietf-mmusic-rid}} and RTP header
extensions (RTPStreamId) {{I-D.ietf-avtext-rid}}
to describe the mapping between simulcast layers and RTP streams.

The SDP format is described in {{I-D.ietf-mmusic-sdp-simulcast}}.

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
cname is the sender's single cname as defined in {{I-D.ietf-rtcweb-rtp-usage}};
carrying some attribute is required by the "a=ssrc" syntax, and sending the
cname is compatible with what has been done in other instances.

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
a=please-send-ssrcs
~~~~

The SFU can detect whether the request has been honored by looking for
a=ssrc attributes in the responding answer.

If a Javascript application wishes to request that the browser generate
offers containing SSRC, it can include the non-standard attribute
"showSsrcInSimulcastOffer" in the RTCPeerConnection constructor:

~~~~
pc = new RTCPeerConnection({showSsrcInSimulcastOffer: true})
~~~~

It is possible to verify that the request is understood by checking for
the presence of this attribute in the RTCPeerConnection parameters:

~~~~
if ('showSsrcInSimulcastOffer' in pc.getConfiguration()) {
   // the request has been understood correctly
}
~~~~
Formally, this amounts to changing the API of a W3C specification, but adding
nonstandard attributes to an initialization dictionary has been done before
in other contexts; it seems like a relatively harmless thing to do, but should
be reviewed in the W3C WEBRTC WG anyway.

# Example

~~~~
m=video
a=simulcast:send hi,mid,low
a=rid:hi send
a=rid:mid send
a=rid:low send
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

# Open questions

NOTE IN DRAFT: The goal is to make this section empty.

The SSRC-group "FID" was picked because it seemed to have the right semantic,
but it's not clear what it's been used for elsewhere. Chrome has been using
the group "SIM" without registering it; this might be a better choice.

It's been suggested that it's better to replace the a=ssrc-group: line with
new tag fields either on the a=ssrc: lines or the a=rid: lines, thus giving
explicit correlation. This, however, breaks the standard format of those
lines. Inventing new syntax for an interim solution seems like a Bad Thing.

People have asked whether and how this document should be published. If it
makes sense to publish it as a historical record, it might make sense to
publish as an RFC; it does not make sense to the author to ask for standards
track publication. At the moment, it claims that publication is not sought.

# Security Considerations

This document describes two existing mechanisms: a=simulcast and
a=ssrc-group. Each of these is defined in an RFC with security considerations.

The only added attack surface here is the ability to create mismatches
between the two lists of simulcast RTP streams, causing different
implementations to choose different streams to display. This is a special
instance of the general rule that "people who can modify your SDP can mess
things up"; normal precautions when passing SDP around should be adequate.

# IANA Considerations

This document has no IANA actions.

If it were to be published, this section would have to request IANA to
register the "please-send-ssrc" attribute, and if it mints a new group
semantic for a=ssrc-group, this will also have to be registered.

If the document succeeds in being transitory in nature, registration
may not be needed.

--- back

# Acknowledgments
{:numbered="false"}

Many thanks to Amit Hilbuch, Adam Roach, Bernard Aboba, Philipp Hancke
and many others who have given input to the design of this mechanism.


