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

## Example Request Flow
For a diagram, please see: https://github.com/bgp/autopeer/?tab=readme-ov-file#sequence-diagram
### AUTH
First, the client makes an authenticated request to the server.
In this example, the client will use PeeringDB OAuth.
Authentication provides the server with client's email (for potential manual discussion), along with client's entitlements, to confirm client is permitted to request on behalf of the proposed ASN.
### REQUEST
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
### CLIENT CONFIGURATION
The client then configures the chosen peering sessions.
If the server added additional approved peering sessions, the client may choose whether or not to configure those sessions.
For every session that the server rejected, the client removes that session from the list to be configured.
### SERVER CONFIGURATION
The server configures all sessions that are in its list of approved peering sessions from its reply to the client.
### MONITORING
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
### COMPLETION
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
(As defined in https://github.com/bgp/autopeer/blob/main/api/openapi.yaml)
Location:
 title: Peering Location
 description: location for session, contains an object with fields id `pdb:ix:$ID` and type PUBLIC or PRIVATE
 type: object
 properties:
   id:
     type: string
   type:
     $ref: '#/components/schemas/LocationType'
LocationType:
  type: string
  enum:
    - private
    - public

SessionStatus:
  title: Session configuration status
  description: |-
    BGP Session Statuses
  type: string

SessionArray:
  title: BGP Sessions
  description: Array of BGP Sessions
  type: array
  items:
    $ref: '#/components/schemas/Session'

Session:
  title: BGP Session
  description: BGP Session
  type: object
  properties:
    local_asn:
      type: integer
    local_ip:
      type: string
    peer_asn:
      type: integer
    peer_ip:
      type: string
    peer_type:
      type: string
    md5: (optional)
      type: string
    location:
      $ref: '#/components/schemas/Location'
    status:
      $ref: '#/components/schemas/SessionStatus'
    uuid:
      type: string
      description: |-
        unique identifier (also serves as request id)
        Optional--server must provide UUID.  Client may provide initial UUID.
Error:
  title: Error
  type: object
  description: Error object for API responses
  properties:
    errors:
      type: array
      description: |-
        Entity field specific validation and operational error messages.

        {field_name} will be substituted with the name of the field in which the validation error occured.
      items:
        $ref: '#/components/schemas/FieldError'
      readOnly: true
  required:
    - field_errors
FieldError:
  title: FieldError
  type: object
  description: Error object for field-specific errors in API responses
  properties:
    name:
      type: string
      description: Field name
      readOnly: true
    errors:
      type: array
      description: List of error messages
      items:
        type: string
      readOnly: true
  required:
    - name
    - errors
    
responses:
  ErrorResponse:
    description: API Error response
    content:
      application/json:
        schema:
          type: object
          properties:
            errors:
              $ref: '#/components/schemas/Error'
          required:
            - errors
        examples:
          error-example:
            value:
              errors:
                - name: peer_asn
                  messages:
                    - asn not available at that location

## Endpoints:
Public Peering over an Internet Exchange (IX):
* ADD/AUGMENT IX PEER
  * Establish new BGP sessions between peers, at the desired exchange.
  * Input: Session Array
* Augment IX PEER
* TOPUP IX
* NEW IX
* REMOVE IX PEER
* ADD PNI
* AUGMENT PNI
* REMOVE PNI
* STATUS of PNI/IX CONNECTIONS
* MAINTENANCE (notification) (no tickets)
* ADD ROUTE SERVER
* DELETE RS
* QUERY for request status
* On each call, there should be:
  * Rate limits, allowed senders, and other restriction options
  * Request source

TODO: list endpoints, both v0 and vLater
TODO: Include diagram and openapi spec?


# Public Peering Session Negotiation
TODO put in a section about how the client and server handshake which locations to configure peering.

# Private Peering
TODO this is future work?
Maybe here we touch on LOA negotiation, unfiltering, etc?
# Maintenance
TODO future work, is this worth mentioning?

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
