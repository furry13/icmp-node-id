---
title: "Extending ICMP for System Indentification"
abbrev: "ICMP System ID"
category: std

docname: draft-fenner-intarea-extended-icmp-hostid-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Internet"
workgroup: "Internet Area Working Group"
keyword:
 - ICMP
 - IPv6 nexthops
 - Host identification
venue:
  group: "Internet Area Working Group"
  type: "Working Group"
  mail: "int-area@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/int-area/"
  github: "fenner/icmp-node-id"
  latest: "https://fenner.github.io/icmp-node-id/draft-fenner-intarea-extended-icmp-hostid.html"

author:
- name: Bill Fenner
  org: Arista Networks
  street: 5453 Great America Parkway
  city: Santa Clara
  code: '95054'
  region: California
  country: USA
  email: fenner@fenron.com

normative:
  RFC4884:
  RFC5837:
  I-D.chroboczek-intarea-v4-via-v6:
  IANA.address-family-numbers:

informative:


--- abstract

RFC5837 describes a mechanism for Extending ICMP for Interface and Next-Hop Identification,
which allows providing additional information in an ICMP error that helps identify
interfaces participating in the path.  This is especially useful in environments
where each interface may not have a unique IP address to respond to, e.g., a traceroute.

This document introduces a similar ICMP extension for Node identification.
It allows providing a unique IP address or a textual name for the node, in
the case where each node may not have a unique IP address (e.g., the
IPv6 nexthop deployment case described in draft-chroboczek-intarea-v4-via-v6).

--- middle

# Introduction

In addition to adding incoming interface information to a traceroute
using the mechanisms described in {{RFC5837}}, a network operator
may be interested in adding information to identify nodes themselves.
{{I-D.chroboczek-intarea-v4-via-v6}} describes a scenario in which individual
nodes do not have unique IPv4 addresses to use to reply to an IPv4
traceroute, so additional information is needed.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Node Identification Object

This section defines the Node Identification Object, an ICMP Extension
Object with a Class-Num (Object Class Value) of TBD that can be appended
to the following messages:

- ICMPv4 Time Exceeded

- ICMPv4 Destination Unreachable

- ICMPv4 Parameter Problem

- ICMPv6 Time Exceeded

- ICMPv6 Destination Unreachable

For reasons described in {{RFC4884}}, this extension cannot be appended
to any of the currently defined ICMPv4 or ICMPv6 messages other than
those listed above.

The extension defined herein MAY be appended to any of the above
listed messages and SHOULD be appended whenever required to identify
the node and when local policy or security
considerations do not supersede this requirement.

## C-Type Meaning in a Node Identification Object

C-Type values in a Node Identification Object include:

- 1: Identifies Node by Name

If the Node Identification Object identifies the node
by name, the Object Payload contains the configured node name.
It is presumed that the operator configures a unique node name
for each system that is identified in this manner.
If the Object Payload would not otherwise
terminate on a 32-bit boundary, it MUST be padded with ASCII NULL
characters.

- 2: Identifies Node by Address

If the Node Identificaton Object identifies the node by
address, the Object Payload contains an address sufficient
to identify the node within the appropriate scope - global
or as otherwise configured - as depicted in {{addrFig}}.

{::comment}
protocol 'AFI:16,Reserved:16,Address...:32'
{:/comment}
~~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|              AFI              |            Reserved           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           Address...
~~~~
{: #addrFig title='Node Identification Object - C-Type 2 Payload'}

Payload fields are defined as follows:

* Address Family Identifier (AFI): This 16-bit field identifies
  the type of address represented by the Address field.
  Values for this field represent a subset of values
  found in the IANA registry of Address Family Numbers (available
  from  {{IANA.address-family-numbers}}).  Valid values are 1 (representing a
  32-bit IPv4 address) and 2 (representing a 128-bit IPv6 address).

* Reserved: This field MUST be set to 0 and ignored upon
  receipt.

* Address: This variable-length field represents an address
  of appropriate scope (global, if none other defined) that
  can be used to identify the node.

# Security Considerations

It may not be desirable to allow this information to be sent to
an arbitrary receiver.  The addition of this information SHOULD
be configurable, and MUST default to off.  An implementation
SHOULD determine what objects may be appended to a given message
based on the destination IP address of the ICMP message that will
contain the objects.

The intended field of use for the extensions defined in this document
is administrative debugging and troubleshooting.  The extensions
herein defined supply additional information in ICMP responses.
These mechanisms are not intended to be used in non-debugging
applications.

This document does not specify an authentication mechanism for the
extension that it defines.  Application developers should be aware
that ICMP messages and their contents are easily spoofed.

# IANA Considerations

This document will ask IANA to allocate an ICMP Extension
Object Class referred to as TBD above.  But for now this
document is just for discussion.


--- back

# Acknowledgments
{:numbered="false"}

This document derives text heavily from {{RFC5837}}, since the
underlying mechanism is identical, and only the content of the
messages differ.