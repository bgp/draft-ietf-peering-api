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
    email: "ramseyer@fb.com"

normative:

informative:


--- abstract

TODO Abstract


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
* And by peering, reducing network latency through expansion of interconneciton relationships



# Conventions and Definitions

All terms used in this document will be defined here:

* Initiator: Network that wants to peer
* Receiver: Network that is receiving communications about peering
* Configured: peering session that is set up on one side
* Established: session is already defined as per BGP4 specification (page 71)



# Security Considerations

PeeringDB OAuth will be the minimum requirement for authorization of API requests.

# Protocol
The Peering API follows the RESTful API mode.
After authentication, a client can request to add or remove peering connections, list potential interconnection locations, and query for upcoming maintenance events.

## Example Request Flow
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



## Endpoints:
* ADD IX PEER
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
# API Endpoints and Specifications
TODO: list endpoints, both v0 and vLater
TODO: Include diagram and openapi spec?

Section: "Public Peering Session Negotiation"
TODO put in a section about how the client and server handshake which locations to configure peering.

Each peer needs an API endpoint that will listen on this and implement the above protocol
This API should be publicly listed in peeringDB and also as a potential expansion of RFC 9092 which provides endpoint integration to whois.
Each API endpoint should be fuzz-tested and protected against abuse.  Attackers should not be able to access internal systems using the API.
Every single request should come in with a unique guid called RequestID that maps to a peering request which can be referenced later.  This guid format should be standardized across all requests.  This guid should be provided by the receiver once it receives the request and must be embedded in all communication.  If there is no RequestID present then that should be interpreted as a new request and the process starts again.
Email is needed for communication if API fails or is not implemented properly (can be obtained through peeringdb)
# Public Peering
TODO should we split them up?
# Private Peering
TODO this is future work?
Maybe here we touch on LOA negotiation, unfiltering, etc?
# Maintenance
TODO future work, is this worth mentioning?
# Authentication Considerations
TODO discuss the different auth considerations
do we mention listing endpoints in peeringDB?
# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
