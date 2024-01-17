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

The Peering API is a mechanism that allows networks to automate interdomain interconnection  between two entities through global Internet Routing.
Using the API, networks will be able to automatically request and accept peering interconnections between Autonomous Systems in public or private scenarios in a time faster than it would take to configure sessions manually.
By speeding up the peering turn-up process and removing the need for manual involvement in peering, the API and automation will ensure that networks can get interconnected as fast, reliably, cost-effectively, and efficiently as possible.
As result, this improves end-user performance for all applications using networks interconnection supporting the Peering API.


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
* Established: session is already defined as per BGP4 specification (page 71)



# Security Considerations


As peering connections exchange real internet traffic, this API requires a security component to verify that the requestor is allowed to request peering on behalf of that ASN.
In this initial proposal, this API requires PeeringDB-based authentication as the standard.
After further discussion, the authors decided to offer alternate authentication options to accomodate the security concerns of different parties.
As peers may require varying security standards, this API will support PeeringDB OIDC as the base requirement, with optional security extensions in addition (RPKI or alternate OIDCs, for example)
This document hopes that, through the RFC process, the Working Group can come to a consensus on a base "authentication standard," to ease adoption for peering partners.

Of particular interest is RPKI.
PeeringDB OIDC allows the API to verify who the requesting party is, while RPKI-signing allows the requesting party to prove that they can configure a request.
The combination of both authorizations provides a strong security guarantee.
This document recognizes that not all partners have the time or engineering resources to support all authorization standards, so the API will offer an extensible security platform to meet varying security requirements.
For RPKI-based authentication, this document refers to RFC9323.


# Protocol
The Peering API follows the RESTful API mode.
After authentication, a client can request to add or remove peering connections, list potential interconnection locations, and query for upcoming maintenance events.

# Example Request Flow
For a diagram, please see: https://github.com/bgp/autopeer/?tab=readme-ov-file#sequence-diagram
## AUTH
First, the client makes an authenticated request to the server.
In this example, the client will use PeeringDB OAuth.
Authentication provides the server with client's email (for potential manual discussion), along with client's entitlements, to confirm client is permitted to request on behalf of the proposed ASN.
## REQUEST
1. ADD SESSION (CLIENT REQUEST)
  * Client provides:
    * Dictionary (multiple):
        1. Local ASN (server)
        2. Local IP
        3. Peer ASN (client)
        4. Peer IP
        5. Peer Type
        6. MD5 (optional)
        7. IXP ID
        8. Status
        9: UUID
        10. Sent prefixes (0 if not Established) (optional)
        11. Received prefixes (0 if not Established) (optional)
        12. Accepted Prefixes (optional)
  * Server actions:
    * Server confirms requested clientASN in list of authorized ASN.
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
        1. Local ASN (server)
        2. Local IP
        3. Peer ASN (client)
        4. Peer IP
        5. Peer Type
        6. MD5 (optional)
        7. IXP ID
        8. Status
        9: UUID
        10. Sent prefixes (0 if not Established) (optional)
        11. Received prefixes (0 if not Established) (optional)
        12. Accepted Prefixes (optional)
The server then responds with the same dictionary, with the information that it understands (status, etc).
## COMPLETION
If both sides report that the session is established, then peering is complete.
If one side does not configure sessions within the server's acceptable configuration window (TimeWindow), then the server is entitled to remove the configured sessions and report "Unestablished" to the client.


# API Endpoints and Specifications
Each peer needs an API endpoint that will implement the API protocol.
This API should be publicly listed in peeringDB and also as a potential expansion of RFC 9092 which provides endpoint integration to whois.
Each API endpoint should be fuzz-tested and protected against abuse.  Attackers should not be able to access internal systems using the API.
Every single request should come in with a unique guid called RequestID that maps to a peering request for later reference.  This guid format should be standardized across all requests.  This guid should be provided by the receiver once it receives the request and must be embedded in all communication.  If there is no RequestID present then that should be interpreted as a new request and the process starts again.
An email address is needed for communication if the API fails or is not implemented properly (can be obtained through peeringdb).

For a programmatic specification of the API, please see the public Github here: https://github.com/bgp/autopeer/blob/main/api/openapi.yaml

This initial draft fully specifies the Public Peering endpoints.  Private Peering and Maintenance are under discussion, and the authors invite collaboration and discussion from interested parties.

## DATA TYPES
As defined in https://github.com/bgp/autopeer/blob/main/api/openapi.yaml.
Please see specification for OpenAPI format.


Peering Location

 Contains string field listing the desired peering location in format `pdb:ix:$IX_ID`, and an enum specifying peering type (public or private).


Session Status

 Status of BGP Session, both as connection status and approval status (Established, Pending, Approved, Rejected, Down, etc)

Session Array

 Array of potential BGP sessions, with request UUID.
 Request UUID is optional for client, and required for server.
 Client may provide initial UUID for client-side tracking, but the server UUID will be the final definitive ID.  Request ID will not change across the request.


BGP Session
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
(As defined in https://github.com/bgp/autopeer/blob/main/api/openapi.yaml)
On each call, there should be rate limits, allowed senders, and other optional restrictions.

### Public Peering over an Internet Exchange (IX):
* ADD/AUGMENT IX PEER
* Establish new BGP sessions between peers, at the desired exchange.
* Below is based on OpenAPI specification: https://github.com/bgp/autopeer/blob/main/api/openapi.yaml

```
POST: /add_sessions
 Request body: Session Array
Responses:
 200 OK:
  Contents: Session Array (all sessions in request accepted for configuration).
 300:
  Contents: Modified Session Array, with rejected or additional sessions.
 400:
  Error
```

* REMOVE IX PEER
  * Given a list of Session Arrays, remove the sessions in that list.
  * This API serves as a notification to the server, as the client may remove the sessions before sending the request to the server.
  * Server replies back with request ID and deletion status (complete, in-progress).

### UTILITY API CALLS
Endpoints which provide useful information for potential interconnections.
* LIST POTENTIAL PEERING LOCATIONS
  * List potential peering locations, both public and private.
  * Below is based on OpenAPI specification: https://github.com/bgp/autopeer/blob/main/api/openapi.yaml
```
GET: /list_locations
 Request parameters:
  * asn (Server ASN, with which to list potential connections)
  * location_type (Optional: Peering Location)
 Response:
  200: OK
   Contents: List of Peering Locations.
  400:
   Error
```

* QUERY for request status
  * Given a request ID, query for the status of that request.
  * Given an ASN without request ID, query for status of all connections between client and server.
  * Below is based on OpenAPI specification: https://github.com/bgp/autopeer/blob/main/api/openapi.yaml
```
GET: /get_status
 Request parameters:
  * asn (requesting client's asn)
  * request_id (optional, UUID of request)
 Response:
  200: OK
   Contents: Session Array of sessions in request_id, if provided.  Else, all existing and in-progress sessions between client ASN and server.
  400:
   Error (example: request_id is invalid)
```

### Private Peering
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
     * LOA or way to recieve it
     * Request ID
   * 300:
     * Proposed Modification: LAG struct, LOA, email address for further discussion
   * 40x: rejections
* REMOVE PNI
  * As ADD/AUGMENT in parameters.  Responses will include a request ID and status.

# Public Peering Session Negotiation
As part of public peering configuration, this draft must consider how the client and server should handshake at which sessions to configure peering.
At first, a client will request sessions A, B, and C.
The server may choose to accept all sessions A, B, and C.
At this point, configuration proceeds as normal.
However, the server may choose to rejest session B.
At that point, the server will reply back with A and C marked as "Accepted," and B as "Rejected."
The server will then configure A and C, and wait for the client to configure A and C.
If the client configured B as well, it will not come up.

This draft encourages peers to set up garbage collection for unconfigured or down peering sessions, to remove stale configs and maintain good router hygiene.

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

