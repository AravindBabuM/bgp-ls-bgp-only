---
title: "BGP Link-State Extensions for BGP-only Networks"
abbrev: "BGP LS for BGP-only Fabric"
category: std

docname: draft-ietf-idr-bgp-ls-bgp-only-fabric-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: Routing
workgroup: Inter-Domain Routing
keyword:
 - BGP
 - Segment Routing
 - SRv6
ipr: trust200902

author:
 -
    fullname: Ketan Talaulikar
    organization: Cisco Systems
    country: India
    email: ketant.ietf@gmail.com
 -
    fullname: AravindBabu MahendraBabu
    role: editor
    organization: Cisco Systems
    country: India
    email: aramahen@cisco.com
 -
    fullname: Clarence Filsfils
    organization: Cisco Systems
    city: Brussels
    country: Belgium
    email: cfilsfil@cisco.com
 -
    fullname: Krishna Swamy
    organization: Cisco Systems
    city: San Jose
    country: USA
    email: kriswamy@cisco.com
 -
    fullname: Shawn Zandi
    organization: LinkedIn
    country: USA
    email: szandi@linkedin.com
 -
    fullname: Gaurav Dawra
    organization: LinkedIn
    country: USA
    email: gdawra.ietf@gmail.com
 -
    fullname: Muhammad Durrani
    organization: Equinix
    country: USA
    email: mdurrani@equinix.com

normative:
  RFC2119:
  RFC5714:
  RFC8174:
  RFC4271:
  RFC9552:
  RFC9085:
  RFC9086:
  RFC9514:
  RFC9857:

informative:
  RFC9256:
  RFC9087:
  RFC8402:
  RFC9830:
  RFC9831:
  RFC7938:
  RFC4272:
  RFC6952:
  I-D.abdelsalam-rtgwg-ipfrr-bgp-only-network:
  I-D.clad-rtgwg-ipfrr-aiml:
  I-D.clad-rtgwg-efficient-remote-protection:
  I-D.filsfils-srv6ops-srv6-e2e-dc-frontend-wan:
  I-D.filsfils-srv6ops-srv6-ai-backend:

...

--- abstract

BGP is used as the only routing protocol in some networks today. In
such networks, it is useful to get a detailed topology view similar to
one available when using link state routing protocols. This document
defines extensions to the BGP Link-state (BGP-LS) address-family and the
procedures for advertisement of topology information in a BGP-only
network.


--- middle

# Introduction {#INTRO}

Network operators are going for a BGP-only routing protocol for
certain networks like Massively Scaled Data Centers (MSDCs). {{RFC7938}}
describes the requirement, design and operational aspects for use of BGP
as the only routing protocol in MSDCs. The underlying link and topology
information between BGP routers is abstracted in this design for
improving scalability and stability in a large scale network. As a
result, a detailed topology view consisting of nodes, links and prefixes
that is available when operating link-state routing protocols is not
available in these BGP-only networks.

BGP Link-State (BGP-LS) {{RFC9552}} enables advertisement of a link
state topology from link-state IGP protocols via BGP that can be
consumed by a controller or in general any software component to get a
complete topology view of the network. BGP-LS extensions for
advertisement of certain aspects of a BGP topology for the Egress Peer
Engineering (EPE) use-case {{RFC9087}} are specified in {{RFC9086}}. This
document leverages the BGP-LS extensions that were defined for EPE and
other BGP-LS features. The document specifies the procedures for
advertising the underlying topology in a BGP-only network.

## Requirements Language

{::boilerplate bcp14-tagged}


# BGP Routing in the Fabric {#BGPROUTING}

The applicability of this specification is limited to those
deployments where BGP is used as hop-by-hop routing protocol between
directly connected nodes in the fabric. While a data-center design
{{RFC7938}} is used as a reference, the topology advertisement and its
use for computation may also apply to other networks with BGP-only
fabric or to BGP-only portions of a larger network topology.

BGP hop-by-hop routing can be set up using EBGP single-hop sessions
over individual links between directly connected routers using their
link addresses for peering as described in {{RFC7938}}. In such a
design, the neighbors' link addresses may be provisioned for peering and
the EBGP session operating directly over the link performs the
monitoring of the neighbor on that link. A variation of this design
would be that the EBGP session is set up between directly connected
routers using their loopback IP addresses. The mechanisms for discovery of
the neighbor's link addresses and their monitoring on a per link basis
are outside the scope of this document.

Though this document uses the EBGP based design as a reference, it
does not preclude other alternate designs using IBGP.

This document does not change base BGP routing protocol operations in
the BGP-only network fabric that provides routing using the BGP best
path selection process {{RFC4271}}.


# Topology Collection Mechanism {#TOPOCOLLECT}

To provide a topological view in networks where BGP is the only
routing protocol, each BGP router advertises information about its local
node, links, and prefixes. {{figure-collection}} describes a typical
deployment scenario. Every BGP router in the network is enabled for
BGP-LS and forms BGP-LS sessions with one or more centralized BGP-LS
speakers over which it sends its local topology information.

Each BGP router may also receive the topology information from all
other BGP routers via these centralized BGP-LS speakers. This way, any
BGP router (as also the centralized BGP-LS speakers) may obtain
aggregated Link-State information for the BGP network. An
external component (e.g. a controller) can obtain this information from
the centralized BGP-LS speakers or directly by doing BGP-LS peering to
the BGP routers. An internal software component on any of the BGP
routers (e.g. TE module) can also receive the entire BGP network
topology information from its local BGP process.

~~~
               +------------+
               | Controller |
               +------------+
                     ^
                     |
                     v
            +-------------------+
            |  BGP-LS Speaker   |       +------------+
            |  (Centralized)    |       | Controller |
            +-------------------+       +------------+
                  ^   ^   ^                   ^
                  |   |   |                   |
      +-----------+   |   +---------------+   |
      |               |                   |   |
      v               v                   v   v
 +-----------+    +-----------+         +-----------+    +----------+
 |    BGP    |    |    BGP    |         |    BGP    |<-->| Local    |
 |  Router   |    |  Router   |  . . .  |  Router   |    | Consumer |
 +-----------+    +-----------+         +-----------+    +----------+
      ^                ^                    ^
      |                |                    |
  Local Info       Local Info            Local Info
 (node & links)  (node & links)         (node & links)
~~~
{: #figure-collection title="Link State Information Collection in a BGP Network"}

## Peering Models {#PEERING}

The peering model described above relies on the base BGP IPv4 or
IPv6 routing underlay (e.g. as described in {{RFC7938}}) or any other
mechanism for reachability for the BGP-LS session establishment with the
centralized BGP speakers. A variation of this model would be to set up
reachability to the centralized BGP speakers (or controller) over the
out of band management network and for each BGP router in the fabric to
use this management network for the BGP-LS session establishment with
the centralized BGP speakers. This variation removes the dependency
between the topology learning via BGP-LS from the reachability over the
BGP routing in the fabric.

Another alternate design would be to enable the BGP-LS
address-family as well on the hop-by-hop EBGP sessions in the underlay
described in {{RFC7938}}. This approach results in the topology
information being flooded via BGP-LS hop-by-hop along the BGP routers in
the network. Other peering designs for BGP-LS sessions may also be
possible and they are not precluded by this document.


# Advertising BGP-only Network Topology {#TOPOADVT}

{{RFC9552}} (BGP-LS) defines the BGP-LS NLRI types (i.e. Node NLRI, Link
NLRI and Prefix NLRI) along with their corresponding BGP-LS Attribute
(i.e. Node Attribute, Link Attribute or Prefix Attribute) and the TLVs
that map to the respective NLRI and Attribute for each type.

{{RFC9086}} specifies the BGP Protocol ID to be used for signaling BGP
EPE information and the same is used for advertising of topology
information in a BGP-only network.

{{RFC9514}} defines the BGP-LS NLRI that can be used to advertise
Segment Routing for IPv6 (SRv6) Segment Identifier (SID) information
instantiated on a BGP Router.

{{RFC9857}} defines the BGP-LS NLRIs that can be used to advertise
information about Segment Routing (SR) Policies instantiated on a BGP
Router headend.

The following sub-sections specify the use of these encodings by a
router running BGP protocol.

## Node Advertisements {#NODE}

{{RFC9552}} defines Node NLRI Type and the Node Descriptor TLVs as
follows:

~~~
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+
 |  Protocol-ID  |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                           Identifier                          |
 |                            (64 bits)                          |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 //                Local Node Descriptors (variable)            //
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~

{{RFC9086}} introduces additional Node Descriptor TLVs for BGP protocol
that are required to be used.

The following Node Descriptors TLVs MUST appear in the Node NLRI as
Local Node Descriptors:

- Autonomous System Number (TLV 512), which contains the advertising
  router ASN.

- BGP Router-ID (TLV 516), which contains the BGP Identifier of the
  originating BGP router.

The BGP-LS Attribute associated with the Node NLRI SHOULD include the
Node Name TLV and MAY include the TE Router-ID TLVs (to indicate a
unique reachable IP address for that node) to signal the router
properties ({{NODE-PROCEDURES}} defines the procedures for their
advertisements):

| TLV Code Point | Description | Reference Document |
|:-:|:--|:--|
| 1026 | Node Name | {{RFC9552}} |
| 1028 | IPv4 TE Router-ID | {{RFC9552}} |
| 1029 | IPv6 TE Router-ID | {{RFC9552}} |
{: #NODE-ATTR title="Node Attribute TLVs"}

The above list of TLVs is not exhaustive and other BGP-LS TLVs
related to the advertisement of the node properties MAY be included
depending on the desired use case.

## Link Advertisements {#LINK}

{{RFC9552}} defines Link NLRI Type and its Node and Link Descriptor
TLVs as follows:

~~~
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+
 |  Protocol-ID  |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                           Identifier                          |
 |                            (64 bits)                          |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 //               Local Node Descriptors (variable)             //
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 //               Remote Node Descriptors (variable)            //
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 //                  Link Descriptors (variable)                //
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~

The following Node Descriptors TLVs MUST appear in the Link NLRI as
Local Node Descriptors:

- Autonomous System Number (TLV 512), which contains the advertising
  router ASN.

- BGP Router-ID (TLV 516), which contains the BGP Identifier of the
  originating BGP router.

The following Node Descriptors TLVs MUST appear in the Link NLRI as
Remote Node Descriptors:

- Autonomous System Number (TLV 512), which contains the peer ASN.

- BGP Router-ID (TLV 516), which contains the BGP Identifier of the
  peer BGP router.

The following Link Descriptors TLVs MUST appear in the Link NLRI as
Link Descriptors:

- Link Local/Remote Identifiers (TLV 258) containing the 4-octet
  Link Local Identifier followed by the 4-octet Link Remote
  Identifier. The value 0 MUST be used for the Link Remote
  Identifier when the value is unknown.

In addition, the following Link Descriptors TLVs SHOULD appear in
the Link NLRI as Link Descriptors based on the address family used when
setting up the BGP Peering using the addresses configured on the links:

- IPv4 Interface Address (TLV 259) contains the address of the local
  interface through which the BGP session is established using IPv4
  address.

- IPv4 Neighbor Address (TLV 260) contains the IPv4 address of the
  peer interface used by the BGP session establishment using IPv4
  address.

- IPv6 Interface Address (TLV 261) contains the address of the local
  interface through which the BGP session is established using IPv6
  address.

- IPv6 Neighbor Address (TLV 262) contains the IPv6 address of the
  peer interface used by the BGP session establishment using IPv6
  address.

The BGP-LS Attribute associated with the Link NLRI SHOULD include the
Link Name and Maximum Link Bandwidth TLVs to signal the link properties
({{LINK-PROCEDURES}} defines the procedures for their advertisements):

| TLV Code Point | Description | Reference Document |
|:-:|:--|:--|
| 1089 | Maximum link bandwidth | {{RFC9552}} |
| 1098 | Link Name | {{RFC9552}} |
{: #LINK-ATTR title="Link Attribute TLVs"}

The above list of TLVs is not exhaustive and other BGP-LS TLVs
related to the advertisement of the link properties MAY be included
depending on desired use case.

## Prefix Advertisements {#PREFIX}

{{RFC9552}} defines Prefix NLRI Type and its Node and Prefix Descriptor
TLVs as follows:

~~~
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+
 |  Protocol-ID  |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                           Identifier                          |
 |                            (64 bits)                          |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 //              Local Node Descriptors (variable)              //
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 //                Prefix Descriptors (variable)                //
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~

The following Node Descriptors TLVs MUST appear in the Prefix NLRI
as Local Node Descriptors:

- Autonomous System Number (TLV 512), which contains the advertising
  router ASN.

- BGP Router-ID (TLV 516), which contains the BGP Identifier of the
  originating BGP router.

The Prefix Descriptor MUST contain the IP Reachability Information
TLV (TLV 265) to identify the prefix.

This document defines a new BGP Route Type TLV that MUST be included
in the Prefix Descriptor when the BGP node advertises the Prefix NLRI.
The format of this TLV is as follows:

~~~
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |              Type             |             Length            |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |  Route Type   |
 +-+-+-+-+-+-+-+-+
~~~

Where:

Type:
: 2-octet field with value TBD.

Length:
: 2-octet field with value set to 1.

Route Type:
: 1-octet with the following values defined:

| Value | Type | Description |
|:-:|:--|:--|
| 1 | Local | Local interface prefix e.g. Loopback |
| 2 | Attached | Directly attached node's prefix e.g. host |
| 3 | External BGP | Prefix learnt via EBGP |
| 4 | Internal BGP | Prefix learnt via IBGP |
| 5 | Redistributed | Prefix redistributed into BGP |
{: #BGPRTTYPES title="BGP Route Types"}

The BGP-LS Attribute associated with the Prefix NLRI SHOULD include the
Prefix Metric TLV to signal the prefix properties and capabilities
({{PREFIX-PROCEDURES}} defines the procedures for their advertisements):

| TLV Code Point | Description | Reference Document |
|:-:|:--|:--|
| 1155 | Prefix Metric | {{RFC9552}} |
{: #PREFIX-ATTR title="Prefix Attribute TLVs"}

The above list of TLVs is not exhaustive and other BGP-LS TLVs
related to the advertisement of the prefix properties MAY be included
depending on desired use case.

## SR Policy and SRv6 SID Advertisements {#SRPOL}

In deployments where Segment Routing (SR) {{RFC8402}} is deployed in
the BGP network and the use case requires information about SR Policies
{{RFC9256}} or SRv6 SIDs instantiated on the routers, the BGP-LS
extensions as follows MAY also be advertised. SRv6 is particularly
applicable in data center networks for both frontend/WAN connectivity
{{I-D.filsfils-srv6ops-srv6-e2e-dc-frontend-wan}} and AI/ML backend
fabrics {{I-D.filsfils-srv6ops-srv6-ai-backend}}.

{{RFC9514}} defines the BGP-LS NLRI that can be used to advertise
Segment Routing for IPv6 (SRv6) Segment Identifier (SID) information
instantiated on a BGP Router. The Local Node Descriptors TLVs are the
same as specified in {{NODE}} and the rest of the procedures are the
same as specified in {{RFC9514}}.

{{RFC9857}} defines the BGP-LS NLRIs that can be used to advertise
information about SR Policies instantiated on a BGP Router headend. The
Headend Node Descriptors TLVs are the same as specified in {{NODE}} and
the rest of the procedures are the same as specified in {{RFC9857}}.


# Procedures {#PROCEDURES}

In a network where BGP is the only routing protocol, the BGP-LS
session is used to advertise the necessary information about the local
node properties, its local links' properties and where necessary the
prefix's owned by the node. TE Policies, that are instantiated on the
local node (i.e. when it is the head-end for the policy), along with
their properties are also advertised via the BGP-LS session. This
information, once collected across all BGP routers in the network,
provides a complete topology view of the network. Many of these
attributes are not part of the base BGP protocol operations and are
either configured or provided by other components on the router. This
information needs to be collected from within the node and advertised
out via BGP-LS.

The following sections describe the procedures for the propagation of
the BGP-LS NLRIs on a BGP router into the BGP-LS session. These
procedures for propagation of BGP topology information via BGP-LS SHOULD
be applied only in deployments and use-cases where necessary and SHOULD
NOT be applied in every BGP deployment when BGP-LS is enabled.
Implementations MAY provide a configuration option to enable these
procedures in required deployments.

## Advertisement of Router's Node Attributes {#NODE-PROCEDURES}

Advertisement of the Node NLRI via BGP-LS by each BGP router in a
BGP-only network enables the discovery of all the router nodes in the
topology. The Node NLRI MUST be generated by a BGP router only for
itself and even when there are no attributes to be advertised along
with it.

The Node Name Attribute SHOULD be advertised when available.

This document introduces some of the TE concepts into BGP-only
networks. Provisioning of TE Router-ID with a unique address normally
associated with a loopback interface on the router enables TE use-cases
for both IPv4 and IPv6 SHOULD be supported. The BGP Router-ID along
with the ASN also provides the capability for uniquely identifying a BGP
router in the network.

Other Node Attributes applicable to a BGP Router may also be included
and this document does not describe the exhaustive list.

## Advertisement of Router's Local Links Attributes {#LINK-PROCEDURES}

Each BGP router in a BGP-only network also advertises its local links
using the Link NLRIs thru BGP-LS. The Link NLRI for a given link
between two BGP routers is advertised as uni-directional logical
"half-link" and its link descriptors allow the correlation between the
two NLRIs "half-links" originated by the peering routers to describe
the bi-directional logical link and its attributes on both routers.

The discovery of all the links and their local and remote identifiers
in a BGP-only network relies on the design that uses EBGP sessions over
each interconnecting link using the link IP addresses (refer
{{RFC7938}}). In this case, a Link NLRI MUST be generated by a BGP
router for each of its local link regardless of whether it has any link
attributes to be advertised for it.

When doing EBGP multi-hop sessions between directly connected BGP
routers, the underlying link information would need to learn by some
discovery protocol or provisioning entity. The mechanisms to learn the
underlying link information for BGP-LS advertisements are outside the
scope of this document. However, to provide a true link topology
picture, the advertisement of underlying links is RECOMMENDED for most
use-cases instead of a single EBGP peering representation of a link
between the routers using their loopback addresses.

The Link NLRI represents an adjacency between BGP routers and its
association with the underlying Layer 3 link. When the underlying
Layer 3 link or the BGP session on top of it goes down, the Link NLRI
MUST be withdrawn by the BGP router. The monitoring of links, detecting
of their failures and notification to BGP may be performed using
mechanisms like BFD. This enables faster detection of failures and
verification of the underlying links.

Advertisement of the Link NLRIs via BGP-LS by each BGP router in a
BGP-only network enables the discovery of all the active links in the
topology.

TE attributes for links have been traditionally associated with Link
State Routing protocols. However, with the ability to discover the link
topology via BGP-LS as specified in this document, the TE attributes and
their principles can also be applied to a network running BGP alone. The
TE attributes for a link have been described in {{LINK-ATTR}} and MAY be
advertised when TE use-cases are enabled.

The use of Layer 3 bundle links which comprise of multiple layer 2
member links are often used in BGP networks. When BGP session is
configured over such a layer 3 link, the link attributes of the
underlying layer 2 links MAY be advertised individually using the L2
Bundle Member TLV. The applicable attributes for the L2 links are
described in {{RFC9085}}.

The Link Name Attribute MAY be advertised when available.

Other Link Attributes applicable to a BGP Router may also be included
and this document does not describe the exhaustive list.

## Advertisement of Router's Prefix Attributes {#PREFIX-PROCEDURES}

Advertisement of the Prefix NLRI via BGP-LS may be required only in
specific use-cases. Since the base BGP protocol along with its
extensions already signals Prefix reachability via different NLRIs,
there is no necessity to duplicate the information via BGP-LS session.

Advertisement of the Prefix NLRI via BGP-LS MUST be done for its
locally configured prefixes (e.g. its loopback interface address). The
advertisement of the Prefix NLRI via BGP-LS for other prefixes learnt by
the router MAY be done based on the specific use-case requirement and
the BGP Route Type as described in {{BGPRTTYPES}} indicates the type of
route being advertised.

Other Prefix Attributes applicable may also be included and this
document does not describe the exhaustive list.

## Advertisement of Router's SR Policy Candidate Path Attributes {#SRPOL-PROCEDURES}

SR Policies instantiated on a BGP router as head-end MAY be
advertised via BGP-LS to provide a policy view of the fabric to
controllers and local TE processes. The SR Policy Candidate Path NLRI
MUST be advertised only for locally instantiated SR Policies and as
specified in {{RFC9857}}. Implementations MAY provide configuration
options to control which SR Policies are advertised from the local node.

## Advertisement of Router's SRv6 SID Attributes {#SRV6SID-PROCEDURES}

SRv6 End SIDs instantiated on a BGP router MAY be advertised via
BGP-LS on SRv6-capable routers to signal SRv6 capabilities for
supported services. The SRv6 SID attributes, including the SRv6
Endpoint Behavior, MUST be included as specified in {{RFC9514}}.


# Usage of BGP Topology {#USE-CASES}

This section describes the key use cases for the BGP topology
information collected as specified in this document: topology visibility
for network monitoring, Segment Routing Traffic Engineering (SR-TE),
IP Fast Reroute (IPFRR), and Efficient Remote Protection (ERP) in
BGP-only networks.

## Topology View for Monitoring {#TOPO-VIEW}

The BGP-LS advertisement of the BGP topology as specified in this
document provides a live topology view of the BGP network for an
application or controller that is monitoring the network. The topology
view is from the BGP protocol perspective and includes the underlying
links as well that aids in network monitoring as well as diagnostics
use-cases. BGP-LS is the de-facto protocol for northbound propagation of
network topology related information for most IGP networks and extending
this capability for BGP-only networks allows existing controllers and
applications to consume the information with some incremental BGP
protocol awareness.

## SR-TE in BGP Networks {#BGP-SRTE}

SR Traffic Engineering (SR-TE) for BGP provides underlay paths through
the network for overlay routes and services with specific SLA
requirements such as path disjointness, low latency paths, inclusion or
exclusion and other TE considerations.

{{RFC9256}} specifies the SR Policy architecture. {{RFC9830}} and
{{RFC9831}} describe the extensions to BGP for signaling of SR Policies
from a controller to the SR-TE headend BGP router. {{RFC9857}} enables
the advertisement of SR Policy state from the headend to controllers via
BGP-LS. This document completes the picture by providing the BGP
topology information from all the routers to a controller as well as
the local SR-TE process on each router for path computation.

The topology collected via BGP-LS in a BGP-only fabric provides the
node, link, and prefix properties along with SR SIDs that enable a
computation entity to build SR Policies for traffic engineering
objectives. The topology may be advertised to a centralized controller
for use-cases requiring centralized computation, or distributed to any
node in the BGP fabric for use by its local SR-TE process.

The actual SR-TE path computation and algorithms are outside the
scope of this document.

## IP Fast Reroute and Protection in BGP-only Networks {#BGP-IPFRR}

AI and Machine Learning (AI/ML) data center fabrics commonly employ
BGP-only multi-tier Clos topologies in which training workloads are
highly synchronized, bandwidth-saturating, and extremely sensitive to
packet loss. These characteristics impose convergence time requirements
on the order of sub-100 microseconds. The requirements and their
implications for IP Fast Reroute (IPFRR) in such fabrics are described
in {{I-D.clad-rtgwg-ipfrr-aiml}}.

Traditional IPFRR {{RFC5714}} deployments rely upon a link-state IGP to
supply the complete topology database from which repair paths are
computed. BGP-only networks lack an equivalent topology database. The
BGP-LS topology collection specified in this document addresses this
limitation by providing complete topology visibility, enabling IPFRR
computations using LFA and TI-LFA techniques. The problem statement and
solution framework for IPFRR in BGP-only networks using BGP-LS as the
topology source are defined in
{{I-D.abdelsalam-rtgwg-ipfrr-bgp-only-network}}.

For scenarios where ECMP and TI-LFA protection are insufficient --
particularly on downward paths in Clos topologies where hairpin reroutes
would consume bandwidth on already-loaded uplinks -- Efficient Remote
Protection (ERP) {{I-D.clad-rtgwg-efficient-remote-protection}} provides
a pre-computed, hairpin-free backup path mechanism. The Node and Link
NLRIs collected via BGP-LS provide the topology input required to
compute such protection paths at any node within the BGP-only fabric.


# IANA Considerations {#IANA}

This document requests IANA to allocate code points from the "BGP-LS
NLRI and Attribute TLVs" sub-registry of the "Border Gateway Protocol -
Link-State (BGP-LS) Parameters" registry group.

This document requests the allocation following TLV codepoints:

| TLV Code Point | Description | Reference |
|:-:|:--|:--|
| TBD | BGP Route Type | this document |


# Security Considerations {#Security}

Procedures and protocol extensions defined in this document do not
affect the BGP security model. See the 'Security Considerations' section
of {{RFC4271}} for a discussion of BGP security. Also refer to
{{RFC4272}} and {{RFC6952}} for analysis of security issues for BGP.

{{RFC9552}} defines BGP-LS NLRI to which the extensions defined in this
document apply. Section 10 of {{RFC9552}} also applies to these
extensions. The procedures defined in this document, by themselves, do
not affect the BGP-LS security model discussed in {{RFC9552}}.

The BGP-LS extensions specified in this document enable topology
visibility and traffic engineering use-cases within a BGP-only fabric as
described in this document. BGP-LS operates within a trusted domain and
its security considerations apply to BGP sessions when carrying topology
information. The topology information distributed by BGP-LS is expected
to be used entirely within this trusted domain which comprises a single
AS or multiple ASes/domains within a single provider network. Therefore,
precaution is necessary to ensure that the topology information
advertised via BGP-LS sessions is limited to nodes in a secure manner
within this trusted domain.

Additionally, it should be considered that the export of detailed
topology information, as described in this document, constitutes a risk
to confidentiality of mission-critical or commercially sensitive
information about the network. BGP-LS peerings are not automatic and
require configuration; thus, it is the responsibility of the network
operator to ensure that only trusted nodes (that include both routers
and controller applications) within the trusted domain are configured to
receive such information.


--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank Bruno Decraene for his review and
comments on this document.
