---
title: "Peering API"
category: info

docname: draft-ramseyer-grow-peering-api-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: RTG
# workgroup: GROW
keyword:
 - BGP
 - peering
 - configuration
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "bgp/draft-ietf-peering-api"
  latest: "https://bgp.github.io/draft-ietf-peering-api/draft-peering-api-ramseyer-protocol.html"

author:
 -
    fullname: "Jenny Ramseyer"
    organization: Meta
    email: "ramseyer@meta.com"

normative:
Public Github with proposed API specification and diagrams: https://github.com/bgp/autopeer/

informative:


--- abstract

We propose an API standard for BGP Peering, also known as interdomain interconnection through global Internet Routing.
This API offers a standard way to request public (settlement-free) peering, verify the status of a request or BGP session, and list potential connection locations.
The API is backed by PeeringDB OIDC, the industry standard for peering authentication.
We also propose future work to cover private peering, and alternative authentication methods.

--- middle

# Introduction

The Peering API is a mechanism that allows networks to automate interdomain interconnection  between two Autonomous Systems (AS) through the Border Gateway Protocol 4 (BGP-4) ([RFC4271](https://datatracker.ietf.org/doc/html/rfc4271)).
Using the API, networks will be able to automatically request and accept peering interconnections between Autonomous Systems in public or private scenarios in a time faster than it would take to configure sessions manually.
By speeding up the peering turn-up process and removing the need for manual involvement in peering, the API and automation will ensure that networks can get interconnected as fast, reliably, cost-effectively, and efficiently as possible.
As a result, this improves end-user performance for all applications using networks interconnection supporting the Peering API.


Business Justification:

By using the Peering API, entities requesting and accepting peering can significantly improve the process to turn up interconnections by:

* Reducing in person-hours spent configuring peering
* Reducing configuration mistakes by reducing human interaction
* And by peering, reducing network latency through expansion of interconnection relationships



# Conventions and Definitions

All terms used in this document will be defined here:

* Initiator: Network that wants to peer
* Receiver: Network that is receiving communications about peering
* Configured: peering session that is set up on one side
* Established: session is already defined as per BGP-4 specification ([page 71](https://datatracker.ietf.org/doc/html/rfc4271#page-71))



# Security Considerations
As peering connections exchange real Internet traffic, this API requires a security component to verify that the requestor is authorized to operate the interconnection on behalf of such AS.
In this initial proposal, the API follows an authorization model based on OpenID Connect (OIDC) ([OpenID.Core](https://openid.net/specs/openid-connect-core-1_0.html)) and OAuth 2.0 ([RFC6749](https://datatracker.ietf.org/doc/html/rfc6749)) where the Authorization Server is PeeringDB. The choice of OpenID Connect is to use the standardized token exchange format based on JSON Web Tokens ([RFC7519](https://www.rfc-editor.org/rfc/rfc7519.html)) which allows interoperation with existing web-based application flows. JWT tokens also supply sufficient claims to implement receiver-side authorization decisions when used as bearer access tokens ([RFC9068](https://datatracker.ietf.org/doc/html/rfc9068)) and for which best common practices also exist ([RFC8725](https://www.rfc-editor.org/rfc/rfc8725.html)).
After further discussion, the authors decided to offer alternate authentication options to accommodate the security concerns of different parties.
As peers may require varying security standards, this document proposes to support PeeringDB OIDC as the base requirement, with optional security extensions in addition (RPKI ([RFC6480](https://datatracker.ietf.org/doc/html/rfc6480)) or alternative OIDC Authorization Servers, for example).
This document hopes that, through the RFC process, the Working Group can come to a consensus on a base "authorization standard," to ease adoption for peering participants.

Of particular interest is RPKI.
PeeringDB OIDC allows the API to identify who the requesting party is, while RPKI-signing allows such requesting party to prove that they own some of the Internet-assigned resources referenced in the request.
This combination provides a low entry barrier to create an identity federation across the participating ASs' API with a stronger guarantee of resource ownership against potential for misattribution and repudiation.
The authors recognize that not all partners have the time or engineering resources to support all authorization standards, so the API reference implementations will offer an extensible security mechanism to meet varying identity and security requirements.
For RPKI-based authentication, this document refers to RPKI Signed Checklists (RSCs) ([RFC9323](https://datatracker.ietf.org/doc/html/rfc9323)).


# Protocol
The Peering API follows the Representational State Transfer ([REST](http://roy.gbiv.com/pubs/dissertation/top.htm)) architecture where sessions, locations, and maintenance events are the resources the API represents and is modeled after the OpenAPI standard [OpenAPI-v3.1.0](https://swagger.io/specification/).
Using the token bearer model ([RFC6750](https://datatracker.ietf.org/doc/html/rfc6750)), a client application can request to add or remove peering sessions, list potential interconnection locations, and query for upcoming maintenance events on behalf of the AS resource owner.

# Example Request Flow
For a diagram, please see: https://github.com/bgp/autopeer/blob/main/README.md#sequence-diagram

## AUTH
First, the initiating OAuth2 Client is also the Resource Owner (RO) so it can follow the OAuth2 client credentials grant [RFC6749-section-4.4](https://datatracker.ietf.org/doc/html/rfc6749#section-4.4).
In this example, the client will use PeeringDB OIDC credentials to acquire a JWT access token that is scoped for use with the receiving API.
On successful authentication, PeeringDB provides the Resource Server (RS) with the client's email (for potential manual discussion), along with the client's usage entitlements (known as OAuth2 scopes), to confirm the client is permitted to make API requests on behalf of the initiating AS.

## REQUEST
1. ADD SESSION (CLIENT REQUEST)

  * Client provides:
    * Dictionary (multiple):
        1. Local ASN (server)
        2. Local IP
        3. Peer ASN (client)
        4. Peer IP
        5. Peer Type (public or private)
        6. MD5 (optional)
        7. IXP ID (PeeringDB Identifier)
        8. Status
        9: UUID
        10. Sent prefixes (0 if not Established) (optional)
        11. Received prefixes (0 if not Established) (optional)
        12. Accepted Prefixes (optional)
  * Server actions:
    * Server confirms requested clientASN in list of authorized ASNs.
    * Optional: checks traffic levels, prefix limit counters, other desired internal checks.

2.  ADD SESSION (SERVER RESPONSE)

  * APPROVAL CASE
    * Server returns dictionary of acceptable peering sessions.  Note: this dictionary may also contain additional sessions on which to peer.  See "Public Peering Session Negotiation" for details.
  * REJECTION CASE

    * Server returns dictionary of acceptable and rejected sessions.
  * Optional information:
    * The server may also include the following details:
      1. Prefix limit counters (optional value)
      2. TimeWindow: Time window indicating when sessions will be configured after being notified (may be 0 if sessions are already configured on receiver side)
      3. isInboundFiltered: optional bool that indicates whether prefixes will be filtered inbound.  If this is set to true, the time window should be set for how long the prefixes will be filtered.
      4. isOutboundFiltered: optional bool that indicates whether prefixes will be filtered outbound.  If this is set to true, the time window should be set for how long the prefixes will be filtered.  If the outbound limit is longer than the inbound limit time, the time window should be set to the max of inbound versus outbound.

## CLIENT CONFIGURATION
The client then configures the chosen peering sessions.
If the server added additional approved peering sessions, the client may choose whether or not to configure those sessions.
For every session that the server rejected, the client removes that session from the list to be configured.

## SERVER CONFIGURATION
The server configures all sessions that are in its list of approved peering sessions from its reply to the client.

## MONITORING
Both client and server wait for sessions to establish.
At any point, client may send a "GET STATUS" request to the server, to request the status of the original request (by UUID) or of a session (by session UUID).
The client will send a dictionary along with the request, as follows:

* Dictionary:
  * Local ASN (server)
  * Local IP
  * Peer ASN (client)
  * Peer IP
  * Peer Type
  * MD5 (optional)
  * IXP ID
  * Status
  * UUID
  * Sent prefixes (0 if not Established) (optional)
  * Received prefixes (0 if not Established) (optional)
  * Accepted Prefixes (optional)

The server then responds with the same dictionary, with the information that it understands (status, etc).

## COMPLETION
If both sides report that the session is established, then peering is complete.
If one side does not configure sessions within the server's acceptable configuration window (TimeWindow), then the server is entitled to remove the configured sessions and report "Unestablished" to the client.


# API Endpoints and Specifications
Each peer needs a public API endpoint that will implement the API protocol.
This API should be publicly listed in peeringDB and also as a potential expansion of [RFC9092](https://datatracker.ietf.org/doc/html/rfc9092) which could provide endpoint integration to WHOIS ([RFC3912](https://datatracker.ietf.org/doc/html/rfc3912)).
Each API endpoint should be fuzz-tested and protected against abuse.  Attackers should not be able to access internal systems using the API.
Every single request should come in with a unique GUID called RequestID that maps to a peering request for later reference.  This GUID format should be standardized across all requests.  This GUID should be provided by the receiver once it receives the request and must be embedded in all communication.  If there is no RequestID present then that should be interpreted as a new request and the process starts again.
An email address is needed for communication if the API fails or is not implemented properly (can be obtained through PeeringDB).

For a programmatic specification of the API, please see the public Github here: [https://github.com/bgp/autopeer/blob/main/api/openapi.yaml](https://github.com/bgp/autopeer/blob/main/api/openapi.yaml)

This initial draft fully specifies the Public Peering endpoints.  Private Peering and Maintenance are under discussion, and the authors invite collaboration and discussion from interested parties.

## DATA TYPES
As defined in [https://github.com/bgp/autopeer/blob/main/api/openapi.yaml](https://github.com/bgp/autopeer/blob/main/api/openapi.yaml).
Please see specification for OpenAPI format.


Peering Location

 Contains string field listing the desired peering location in format `pdb:ix:$IX_ID`, and an enum specifying peering type (public or private).


Session Status

 Status of BGP Session, both as connection status and approval status (Established, Pending, Approved, Rejected, Down, Unestablished, etc)

Session Array

 Array of potential BGP sessions, with request UUID.
 Request UUID is optional for client, and required for server.
 Client may provide initial UUID for client-side tracking, but the server UUID will be the final definitive ID.  RequestID will not change across the request.


* BGP Session
  * local_asn (ASN of requestor)
  * local_ip (IP of requestor, v4 or v6)
  * peer_asn (server ASN)
  * peer_ip (server-side IP)
  * peer_type (public or private)
  * md5 (optional string)
  * location (Peering Location, as defined above)
  * status (Session Status, as defined above)
  * UUID (of individual session.  Server must provide UUID.  Client may provide initial UUID for client-side tracking, but the server UUID will be the final definitive ID)


Error

 API Errors, for field validation errors in requests, and request-level errors.


The above is sourced largely from the linked OpenAPI specification.

## Endpoints:
(As defined in [https://github.com/bgp/autopeer/blob/main/api/openapi.yaml](https://github.com/bgp/autopeer/blob/main/api/openapi.yaml))
On each call, there should be rate limits, allowed senders, and other optional restrictions.

### Public Peering over an Internet Exchange (IX):
* /add_sessions: ADD/AUGMENT IX PEER
  * Establish new BGP sessions between peers, at the desired exchange.
  * Below is based on OpenAPI specification: [https://github.com/bgp/autopeer/blob/main/api/openapi.yaml](https://github.com/bgp/autopeer/blob/main/api/openapi.yaml)
  * POST: /add_sessions
    * Request body: Session Array
    * Responses:
      * 200 OK:
        * Contents: Session Array (all sessions in request accepted for configuration).
      * 300:
        * Contents: Modified Session Array, with rejected or additional sessions.
      * 400:
        * Error


* REMOVE IX PEER
  * Given a list of Session Arrays, remove the sessions in that list.
  * This API serves as a notification to the server, as the client may remove the sessions before sending the request to the server.
  * Server replies back with request ID and deletion status (complete, in-progress).

### UTILITY API CALLS
Endpoints which provide useful information for potential interconnections.

* /list_locations: LIST POTENTIAL PEERING LOCATIONS
  * List potential peering locations, both public and private.
  * Below is based on OpenAPI specification: https://github.com/bgp/autopeer/blob/main/api/openapi.yaml
  * GET: /list_locations
    * Request parameters:
      * asn (Server ASN, with which to list potential connections)
      * location_type (Optional: Peering Location)
    * Response:
      * 200: OK
        * Contents: List of Peering Locations.
      * 400:
        * Error



* /get_status: QUERY FOR REQUEST STATUS
  * Given a request ID, query for the status of that request.
  * Given an ASN without request ID, query for status of all connections between client and server.
  * Below is based on OpenAPI specification: [https://github.com/bgp/autopeer/blob/main/api/openapi.yaml](https://github.com/bgp/autopeer/blob/main/api/openapi.yaml)
  * GET: /get_status
    * Request parameters:
      * asn (requesting client's asn)
      * request_id (optional, UUID of request)
    * Response:
      * 200: OK
        * Contents: Session Array of sessions in request_id, if provided.  Else, all existing and in-progress sessions between client ASN and server.
      * 400:
        * Error (example: request_id is invalid)


### Private Peering (DRAFT)
* ADD/AUGMENT PNI
 * Parameters:
   * Peer ASN
   * Facility
   * email address (contact)
   * Action type: add/augment
   * LAG struct:
     * IPv4
     * IPv6
     * Circuit ID
   * Who provides LOA?  (and where to provide it).
 * Response:
   * 200:
     * LAG struct, with server data populated
     * LOA or way to receive it
     * Request ID
   * 300:
     * Proposed Modification: LAG struct, LOA, email address for further discussion
   * 40x: rejections
* REMOVE PNI
  * As ADD/AUGMENT in parameters.  Responses will include a requestID and status.

# Public Peering Session Negotiation
As part of public peering configuration, this draft must consider how the client and server should handshake at which sessions to configure peering.
At first, a client will request sessions A, B, and C.
The server may choose to accept all sessions A, B, and C.
At this point, configuration proceeds as normal.
However, the server may choose to reject session B.
At that point, the server will reply back with A and C marked as "Accepted," and B as "Rejected."
The server will then configure A and C, and wait for the client to configure A and C.
If the client configured B as well, it will not come up.

This draft encourages peers to set up garbage collection for unconfigured or down peering sessions, to remove stale configuration and maintain good router hygiene.

Related to rejection, if the server would like to configure additional sessions with the client, the server may reply back with additional peering sessions D and E.
The server will configure D and E on their side, and D and E will become part of the sessions requested in the UUID.
The client may choose whether or not to accept those additional sessions.
If they do, the client should configure D and E as well.
If they do not, the client will not configure D and E, and the server should garbage-collect those pending sessions.

As part of the IETF discussion, the authors would like to discuss how to coordinate which side unfilters first.
Perhaps this information could be conveyed over a preferences vector.

# Private Peering
Through future discussion with the IETF, the specification for private peering will be solidified.
Of interest for discussion includes Letter of Authorization (LOA) negotiation, and how to coordinate unfiltering and configuration checks.

# Automated Peering Policies
TODO We should have something here.
Split into: things that need to be handshaked between the two peers (LOA, etc) and things that do not (traffic levels)

# Maintenance
This draft does not want to invent a new ticketing system.
However, there is an opportunity in this API to provide maintenance notifications to peering partners.
If there is interest, this draft would extend to propose a maintenance endpoint, where the server could broadcast upcoming and current maintenance windows.

A maintenance message would follow a format like:

* Title: string
* Start Date: date maintenance start(s/ed): UTC
* End Date: date maintenance ends: UTC
* Area: string or enum
* Details: freeform string

The "Area" field could be a freeform string, or could be a parseable ENUM, like (BGP, PublicPeering, PrivatePeering, Configuration, Caching, DNS, etc).

Past maintenances will not be advertised.

# Possible Extensions
The authors acknowledge that route-server configuration may also be of interest for this proposed API, and look forward to future discussions in this area.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
This project is joint work between Meta, AWS, Cloudflare, FullCtl, and Google.
Many thanks to my collaborators: Carlos Aguado (AWS), Ben Blaustein (Meta), Matt Griswold (FullCtl), Ben Ryall (Meta), Arturo Servin (Google), and Tom Strickx (Cloudflare).

{:numbered="false"}

