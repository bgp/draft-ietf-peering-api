---
title: "Peering API"
category: info

docname: draft-ramseyer-grow-peering-api-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: OPS
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
    fullname: "Carlos Aguado"
    organization: Amazon
    email: "crlsa@amazon.com"
 -
    fullname: "Matt Griswold"
    organization: FullCtl
    email: "grizz@20c.com"
 -
    fullname: "Jenny Ramseyer"
    organization: Meta
    email: "ramseyer@meta.com"
 -
    fullname: "Arturo Servin"
    organization: Google
    email: "arturolev@google.com"
 -
    fullname: "Tom Strickx"
    organization: Cloudflare
    email: "tstrickx@cloudflare.com"


normative:
    autopeer:
        target: https://github.com/bgp/autopeer/
        title: Github repository with the API specification and diagrams
    oidc:
        target: https://openid.net/specs/openid-connect-core-1_0.html
        title: OpenID.Core
    rest:
        target: http://roy.gbiv.com/pubs/dissertation/top.htm
        title: Architectural Styles and the Design of Network-based Software Architectures
        author:
            name: Roy Thomas Fielding
            ins: R. T. Fielding
        date: 2000
    openapi:
        target: https://spec.openapis.org/oas/v3.1.0
        title: OpenAPI-v3.1.0


--- abstract

We propose an API standard for BGP Peering, also known as interdomain interconnection through global Internet Routing.
This API offers a standard way to request public (settlement-free) peering, verify the status of a request or BGP session, and list potential connection locations.
The API is backed by PeeringDB OIDC, the industry standard for peering authentication.
We also propose future work to cover private peering, and alternative authentication methods.

--- middle

Introduction    {#problems}
============

The Peering API is a mechanism that allows networks to automate interdomain interconnection  between two Autonomous Systems (AS) through the Border Gateway Protocol 4 ({{?RFC4271}}).
Using the API, networks will be able to automatically request and accept peering interconnections between Autonomous Systems in public or private scenarios in a time faster than it would take to configure sessions manually.
By speeding up the peering turn-up process and removing the need for manual involvement in peering, the API and automation will ensure that networks can get interconnected as fast, reliably, cost-effectively, and efficiently as possible.
As a result, this improves end-user performance for all applications using networks interconnection supporting the Peering API.


Business Justification      {#justification}
----------------------

By using the Peering API, entities requesting and accepting peering can significantly improve the process to turn up interconnections by:

* Reducing in person-hours spent configuring peering
* Reducing configuration mistakes by reducing human interaction
* And by peering, reducing network latency through expansion of interconnection relationships


Conventions and Terminology     {#conventions}
===========================

All terms used in this document will be defined here:

* Initiator: Network that wants to peer
* Receiver: Network that is receiving communications about peering
* Configured: peering session that is set up on one side
* Established: session is already defined as per BGP-4 specification  {{Section 8.2.2 of ?RFC4271}}


Audience    {#audience}
========
The Peering API aims to simplify peering interconnection configuration.
To that end, the API can be called by either a human or some automation.
A network engineer can submit API requests through a client-side tool, and configure sessions by hand or through existing tooling.
Alternately, an automated service can request BGP sessions through some trigger or regularly scheduled request (for example, upon joining a new peering location, or through regular polling of potential peers).
That automated client can then configure the client sessions through its own tooling.
For ease of exchanging peering requests, the authors suggest peers to maintain both a client and a server for the API.
Toward the goal of streamlining peering configuration, the authors encourage peers to automate their network configuration wherever possible, but do not require full automation to use this API.


Protocol    {#protocol}
========

The Peering API follows the Representational State Transfer ({{rest}}) architecture where sessions, locations, and maintenance events are the resources the API represents and is modeled after the OpenAPI standard {{openapi}}.
Using the token bearer model ({{!RFC6750}}), a client application can request to add or remove peering sessions, list potential interconnection locations, and query for upcoming maintenance events on behalf of the AS resource owner.

Example Peering Request Negotiation
-------------------------------------

Diagram of the Peering Request Process


~~~~~~~~~~

+-------------------+                    +-------------------+
|     Step 1        |                    |     Step 2        |
|    Network 1      |------------------->|    Network 1      |
|  Identifies need  |                    |   Checks details  |
+-------------------+                    +-------------------+
                                                 |
                                                 v
+-------------------+                    +-------------------+
|      Step 4       |                    |      Step 3       |
|    Network 1      |<-------------------|    Network 1      |
|  gets approval    |                    |     Public or     |
|      token        |                    | Private Peering   |
+-------------------+                    +-------------------+
        |
        v
+-------------------+                    +-------------------+
|      Step 5       |                    |      Step 6       |
|  Network 1 finds  |------------------->|    Network 1      |
| common locations  |                    |  request peering  |
+-------------------+                    +-------------------+
                                                  |
                                                  v
+-------------------+                   +--------------------+
|      Step 8       |                   |       Step 7       |
|     Network 2     |<------------------|     Network 2      |
| accepts or reject |                   |      verifies      |
|    sessions       |                   |     credentials    |
+-------------------+                   +--------------------+
       /     \
      /       \
     /         \
(yes)           (no)
      \          |
       \         +-------------------------------|
        \                                        |
         \                                       |
          v                                      |
+---------------+          +----------------+    |
|    Step 9     |          |    Step 10     |    |
| Sessions are  |          | Network 1 or 2 |    |
|  provisioned  |--------->| checks session |    |
|     status    |          | until it is up |    |
+---------------+          +----------------+    |
                                   |             |
                      +------------+             |
                      |                          |
                      v                          |
            +-------------+                      |
            |   Step 11   |                      |
            |   Request   |<---------------------+
            |  Terminate  |
            +-------------+







Step 1 [Human]: Network 1 identifies that it would be useful to peer with Network 2 to interchange traffic more optimally


Step 2 [Human]: Network 1 checks technical and other peering details about Network 2 to check if peering is possible


Step 3 [Human]: Network 1 decides in type (Public or PNI) of peering and facility


Step 4 [API]: Network 1 gets approval/token that is authorized to ‘speak’ on behalf of Network 1’s ASN.


Step 5 [API]: Network 1 checks PeeringDB for common places between Network 1 and Network 2.
API: GET /locations


Step 6 [API]: Network 1 request peering with Network 2
API:      POST /add_sessions


Step 7 [API]: Network 2 verifies Network 1 credentials, check requirements for peering


Step 8 [API]: Network 2 accepts or rejects session(s)
API Server gives yes/no for request


Step 9 [API]: If yes, sessions are provisioned, Networks 1 or Network 2 can check status
API: /sessions Get session status


Step 10 [API]: API keeps polling until sessions are up


Step 11 [API]: Process Terminate
~~~~~~~~~~


Example API Flow
--------------------
The diagram below outlines the proposed API flow.


~~~~~~~~~~
OIDC Authentication

+-----------+                 +-------+                    +-----------+
| Initiator |                 | Peer  |                    | PeeringDB |
+-----------+                 +-------+                    +-----------+
      |                           |                              |
      | OIDC Authentication       |                              |
      |--------------------------------------------------------->|
      |                           |                              |
      |                                        Provide auth code |
      |<---------------------------------------------------------|
      |                           |                              |
      | Send auth code to Peer    |                              |
      |--------------------------------------------------------->|
      |                           |                              |
      |                           | Exchange auth code for token |
      |                           |----------------------------->|
      |                           |                              |
      |                           |                 Return token |
      |                           |<-----------------------------|
      |                           |
      | Peer determines permissions based on token
      |                           |
      | Send OK back to Initiator |
      |<--------------------------|

Operations, loop until peering is complete.

List Locations

+-----------+                                                  +-------+
| Initiator |                                                  | Peer  |
+-----------+                                                  +-------+
      |                                                            |
      | QUERY peering locations (peer type, ASN, auth code)        |
      |----------------------------------------------------------->|
      |                                                            |
      |                               Reply with peering locations |
      |                            or errors (401, 406, 451, etc.) |
      |<-----------------------------------------------------------|


Request session status

+-----------+                                                  +-------+
| Initiator |                                                  | Peer  |
+-----------+                                                  +-------+
      |                                                            |
      | QUERY request status using request ID & auth code          |
      |----------------------------------------------------------->|
      |                                                            |
      |                                  Reply with session status |
      |                                      (200, 404, 202, etc.) |
      |<-----------------------------------------------------------|
~~~~~~~~~~


AUTH    {#auth}
----
First, the initiating OAuth2 Client is also the Resource Owner (RO) so it can follow the OAuth2 client credentials grant {{Section 4.4 of !RFC6749}}.
In this example, the client will use PeeringDB OIDC credentials to acquire a JWT access token that is scoped for use with the receiving API.
On successful authentication, PeeringDB provides the Resource Server (RS) with the client's email (for potential manual discussion), along with the client's usage entitlements (known as OAuth2 scopes), to confirm the client is permitted to make API requests on behalf of the initiating AS.

REQUEST     {#request}
-------
1. ADD SESSION (CLIENT BATCHED REQUEST)


  * The initiator's client provides a set of the following information, where local always refers to the receiver and peer always refers to the initiator:
    * Structure:
        1. Local ASN
        2. Local IP
        3. Peer ASN
        4. Peer IP
        5. Local BGP Role according to {{?RFC9234}}
        6. Peer BGP Role according to {{?RFC9234}}
        8. Local insert ASN (optional to support route servers)
        9. Peer insert ASN (optional to support route servers)
       11. Local monitoring session (optional to support monitoring systems)
       12. Peer monitoring session (optional to support monitoring systems)
       10. Peer Type (public or private)
       11. Session Secret (optional with encoding agreed outside of this specification)
       12. Location (Commonly agreed identifier of the BGP speaker, e.g. PeeringDB IX lan ID)

  * The receiver's expected actions:
    * The server confirms requested clientASN in list of authorized ASNs.
    * Optional: checks traffic levels, prefix limit counters, other desired internal checks.

2.  ADD SESSIONS (SERVER BATCHED RESPONSE)

  * APPROVAL CASE
    * Server returns a list with the structure for each of the acceptable peering sessions. Note: this structure may also contain additional attributes such as the server generated session ID.
  * PARTIAL APPROVAL CASE
    * Server returns a list with the structure for each of the acceptable peering sessions as in the approval case. The server also returns a list of sessions that have not deemed as validated or acceptable to be created. The set of sessions accepted and rejected is disjoint and the join of both sets matches the cardinality of the requested sessions.
  * REJECTION CASE
    * Server returns an error message which indicates that all of the sessions requested have been rejected and the reason for it.

CLIENT CONFIGURATION    {#clientconfig}
--------------------
The client then configures the chosen peering sessions asynchronously using their internal mechanisms.
The client SHOULD pull and use additional information on the new peering from public sources as required to ensure routing security, e.g., AS-SETs to configure appropriate filters.
For every session that the server rejected, the client removes that session from the list to be configured.

SERVER CONFIGURATION    {#serverconfig}
--------------------
The server configures all sessions that are in its list of approved peering sessions from its reply to the client.
The server SHOULD pull and use additional information on the new peering from public sources to ensure routing security, e.g., AS-SETs to configure appropriate filters.

MONITORING   {#monitoring}
----------
Both client and server wait for sessions to establish.
At any point, client may send a "GET STATUS" request to the server, to request the status of the session (by session ID).
The client will send a structure along with the request, as follows:

* structure (where local refers to the server and peer refers to the client):
  * Session ID
  * Local ASN
  * Local IP
  * Peer ASN
  * Peer IP
  * Local BGP Role ({{?RFC9234}})
  * Peer BGP Role ({{?RFC9234}})
  * Local insert ASN (optional, as defined above)
  * Peer insert ASN (optional, as defined above)
  * Local monitoring session (optional, as defined above)
  * Peer monitoring session (optional, as defined above)
  * Peer Type
  * Session secret (optional, as defined above)
  * Location
  * Status

The server then responds with the same structure, with the information that it understands (status, etc).

COMPLETION      {#completion}
----------
If both sides report that the session is established, then peering is complete.
If one side does not configure sessions within the server's acceptable configuration window (TimeWindow), then the server is entitled to remove the configured sessions and report "Unestablished" to the client.


API Endpoints and Specifications    {#endpoints_and_specs}
================================

Each peer needs a public API endpoint that will implement the API protocol.
This API should be publicly listed in peeringDB and also as a potential expansion of {{?RFC9092}} which could provide endpoint integration to WHOIS ({{?RFC3912}}).
Each API endpoint should be fuzz-tested and protected against abuse.  Attackers should not be able to access internal systems using the API.
Every single request should come in with a unique GUID called RequestID that maps to a peering request for later reference.
This GUID format should be standardized across all requests.
This GUID should be provided by the receiver once it receives the request and must be embedded in all communication.
If there is no RequestID present then that should be interpreted as a new request and the process starts again.
An email address is needed for communication if the API fails or is not implemented properly (can be obtained through PeeringDB).

For a programmatic specification of the API, please see the public Github ({{autopeer}}).

This initial draft fully specifies the Public Peering endpoints.
Private Peering and Maintenance are under discussion, and the authors invite collaboration and discussion from interested parties.

DATA TYPES      {#datatypes}
----------
Please see specification ({{autopeer}}) for OpenAPI format.

Peering Location

  Contains string field listing the desired peering location in format `pdb:ix:$IX_ID`, and an enum specifying peering type (public or private).

Session Status

  Status of BGP Session, both as connection status and approval status (Established, Pending, Approved, Rejected, Down, Unestablished, etc)

Session Array

  Array of potential BGP sessions, with request UUID.
  Request UUID is optional for client, and required for server.
  Return URL is optional, and indicates the client's Peering API endpoint.
  The client's return URL is used by the server to request additional sessions.
  Client may provide initial UUID for client-side tracking, but the server UUID will be the final definitive ID.
  RequestID will not change across the request.

BGP Session

  A structure that describes a BGP session and contains the following elements:

  * local_asn (ASN of requestor)
  * local_ip (IP of requestor, v4 or v6)
  * peer_asn (server ASN)
  * peer_ip (server-side IP)
  * local_bgp_role (BGP role according to {{?RFC9234}})
  * peer_bgp_role (BGP role according to {{?RFC9234}})
  * local_insert_asn (optional, to support route servers, defaults to true)
  * peer_insert_asn (optional, to support route servers, defaults to true)
  * local_monitoring_session (optional, to support monitoring systems, defaults to false)
  * peer_monitoring_session (optional, to support monitoring systems, defaults to false)
  * peer_type (public or private)
  * session_secret (optional, as defined above)
  * location (Peering Location, as defined above)
  * status (Session Status, as defined above)
  * session_id (of individual session and generated by the server)

As not all elements are reflected in the {{autopeer}} OpenAPI definition to date, we define the missing fields here to be reflected in {{autopeer}} in the future.

  * local_bgp_role and peer_bgp_role: these field describe the BGP roles of the local and peer side of the session according to {{?RFC9234}} represented by an integer. The roles for both sides MUST be set in a way that does not violate role correctness as defined in Section 4.2 of {{?RFC9234}}.
  * local_insert_asn and peer_insert_asn: these fields define whether the local or peer side will insert their ASN into the AS path attribute of forwarded BGP routes. They are mostly relevant to route servers. The fields are boolean and optional. If not provided, they default to true.
  * local_monitoring_session and peer_monitoring_session: these fields define whether the local or peer side of the session will forward routes to other ASes or not. As the role of monitoring systems is not defined in {{?RFC9234}}, we add this role via a boolean, optional flag. If not provided, they default to false. local_monitoring_session and peer_monitoring_sessions MUST NOT be true at the same time for the same session to avoid a role mismatch.

Error

  API Errors, for field validation errors in requests, and request-level errors.

The above is sourced largely from the linked OpenAPI specification.

Endpoints   {#endpoints}
---------
(As defined in {{autopeer}}).
On each call, there should be rate limits, allowed senders, and other optional restrictions.


### Public Peering over an Internet Exchange (IX)   {#public_peering_ix}
* `/sessions`: ADD/RETRIEVE sessions visible to the calling PEER
  * Batch create new session resources
    * Establish new BGP sessions between peers, at the desired exchange.
    * Below is based on OpenAPI specification: {{autopeer}}.
    * `POST /sessions`
      * Request body: Session Array
      * Responses:
        * 200 OK:
          * Contents: Session Array (all sessions in request accepted for configuration). Should not all the sessions be accepted, the response also contains a list of sessions and the respective errors.
        * 400:
          * Error
        * 403:
          * Unauthorized to perform the operation
        * 422:
          * Please contact us, human intervention required

  * List all session resources. The response is paginated.
    * Given a request ID, query for the status of that request.
    * Given an ASN without request ID, query for status of all connections between client and server.
    * Below is based on OpenAPI specification: {{autopeer}}.
    * `GET /sessions`
      * Request parameters:
        * asn (requesting client's asn)
        * request_id (optional, UUID of request)
        * max_results (integer to indicate an upper bound for a given response page)
        * next_token (opaque and optional string received on a previous response page and which allows the server to produce the next page of results. Its absence indicates to the server that the first page is expected)
      * Response:
        * 200: OK
          * Contents: Session Array of sessions in request_id, if provided. Else, all existing and in-progress sessions between client ASN and server.
            * next_token (opaque and optional string the server expects to be passed back on the request for the next page. Its absence indicates to the client that no more pages are available)
        * 400:
          * Error (example: request_id is invalid)
        * 403:
          * Unauthorized to perform the operation

* `/sessions/{session_id}`: Operate on individual sessions
  * Retrieve an existing session resource
    * Below is based on OpenAPI specification: {{autopeer}}.
    * `GET /sessions/{session_id}`
      * Request parameters:
        * session_id returned by the server on creation or through the session list operation.
      * Responses:
        * 200 OK:
          * Contents: Session structure with current attributes
        * 400:
          * Error (example: session_id is invalid)
        * 403:
          * Unauthorized to perform the operation
        * 404:
          * The session referred by the specified session_id does not exist or is not visible to the caller

  * Delete a session.
    * Given a session ID, delete it which effectively triggers an depeering from the initiator.
    * Below is based on OpenAPI specification: {{autopeer}}.
    * `DELETE /sessions/{session_id}`
      * Request parameters:
        * session_id returned by the server on creation or through the session list operation.
      * Response:
        * 204: OK
          * Contents: empty response as the session is processed and hard deleted
        * 400:
          * Error (example: session_id is invalid)
        * 403:
          * Unauthorized to perform the operation
        * 404:
          * The session referred by the specified session_id does not exist or is not visible to the caller
        * 422:
          * Please contact us, human intervention required

### UTILITY API CALLS   {#utility_api}
Endpoints which provide useful information for potential interconnections.

* `/locations`: LIST POTENTIAL PEERING LOCATIONS
  * List potential peering locations, both public and private. The response is paginated.
    * Below is based on OpenAPI specification: {{autopeer}}.
    * `GET /locations`
      * Request parameters:
        * asn (Server ASN, with which to list potential connections)
        * location_type (Optional: Peering Location)
        * max_results (integer to indicate an upper bound for a given response page)
        * next_token (opaque and optional string received on a previous response page and which allows the server to produce the next page of results. Its absence indicates to the server that the first page is expected)
      * Response:
        * 200: OK
          * Contents: List of Peering Locations.
            * next_token (opaque and optional string the server expects to be passed back on the request for the next page. Its absence indicates to the client that no more pages are available)
        * 400:
          * Error
        * 403:
          * Unauthorized to perform the operation

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
   * Who provides LOA? (and where to provide it).
 * Response:
   * 200:
     * LAG struct, with server data populated
     * LOA or way to receive it
     * Request ID
   * 40x: rejections
* REMOVE PNI
  * As ADD/AUGMENT in parameters. Responses will include a requestID and status.

Public Peering Session Negotiation  {#session_negotiation}
==================================

As part of public peering configuration, this draft must consider how the client and server should handshake at which sessions to configure peering.
At first, a client will request sessions A, B, and C.
The server may choose to accept all sessions A, B, and C.
At this point, configuration proceeds as normal.
However, the server may choose to reject session B.
At that point, the server will reply back with A and C marked as "Accepted," and B as "Rejected."
The server will then configure A and C, and wait for the client to configure A and C.
If the client configured B as well, it will not come up.

This draft encourages peers to set up garbage collection for unconfigured or down peering sessions, to remove stale configuration and maintain good router hygiene.

Related to rejection, if the server would like to configure additional sessions with the client, the server may either reject all the session that do not meet the criteria caused by such absence in the client's request or approve the client's request and issue a separate request to the client's server requesting those additional peering sessions D and E.
The server will configure D and E on their side, and D and E will become part of the sessions requested in the UUID.
The client may choose whether or not to accept those additional sessions.
If they do, the client should configure D and E as well.
If they do not, the client will not configure D and E, and the server should garbage-collect those pending sessions.

As part of the IETF discussion, the authors would like to discuss how to coordinate which side unfilters first.
Perhaps this information could be conveyed over a preferences vector.

Private Peering     {#private_peering}
===============

Through future discussion with the IETF, the specification for private peering will be solidified.
Of interest for discussion includes Letter of Authorization (LOA) negotiation, and how to coordinate unfiltering and configuration checks.

Maintenance     {#maintenance}
===========

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

Security Considerations     {#security}
=======================

This document describes a mechanism to standardize the discovery, creation and maintenance of peering relationships across autonomous systems (AS) using an out-of-band application programming interface (API). With it, AS operators take a step to operationalize their peering policy with new and existing peers in ways that improve or completely replace manual business validations, ultimately leading to the automation of the interconnection. However, this improvement can only be fully materialized when operators are certain that such API follows the operational trust and threat models they are comfortable with, some of which are documented in BGP operations and security best practices ({{?RFC7454}}). To that extent, this document assumes the peering API will be deployed following a strategy of defense in-depth and proposes the following common baseline threat model below.

#### Threats     {#threats}

Each of the following threats assume a scenario where an arbitrary actor is capable of reaching the peering API instance of a given operator, the client and the operator follow their own endpoint security and maintenance practices, and the trust anchors in use are already established following guidelines outside of the scope of this document.

* T1: A malicious actor with physical access to the IX fabric and peering API of the receiver can use ASN or IP address information to impersonate a different IX member to discover, create, update or delete peering information which leads to loss of authenticity, confidentiality, and authorization of the spoofed IX member.
* T2: A malicious actor with physical access to the IX fabric can expose a peering API for an IX member different of its own to accept requests on behalf of such third party and supplant it, leading to a loss of authenticity, integrity, non-repudiability, and confidentiality between IX members.
* T3: A malicious actor without physical access to the IX fabric but with access the the peering API can use any ASN to impersonate any autonomous system and overload the receiver's peering API internal validations leading to a denial of service.

#### Mitigations     {#mitigations}

The following list of mitigations address different parts of the threats identified above:

* M1: Authorization controls - A initiator using a client application is authorized using the claims presented in the request prior to any interaction with the peering API (addresses T1, T2).
* M2: Proof of holdership - The initiator of a request through a client can prove their holdership of an Internet Number Resource (addresses T1, T3).
* M3: Request integrity and proof of possession - The peering API can verify HTTP requests signed with a key that is cryptographically bound to the authorized initiator (addresses T1, T2).

The Peering API does not enforce any kind of peering policy on the incoming requests. It is left to the peering API instance implementation to enforce the AS-specific peering policy. This document encourages each peer to consider the needs of their peering policy and implement request validation as desired.

Authorization controls     {#authorization}
----------------------

The peering API instance receives HTTP requests from a client application from a peering initiator. Those requests can be authorized using the authorization model based on OAuth 2.0 ({{!RFC6749}}) with the OpenID Connect {{oidc}} core attribute set. The choice of OpenID Connect is to use a standardized and widely adopted authorization exchange format based on JSON Web Tokens ({{!RFC7519}}) which allows interoperation with existing web-based application flows. JWT tokens also supply sufficient claims to implement receiver-side authorization decisions by third parties when used as bearer access tokens ({{!RFC9068}}). The peering API instance (a resource server in OAuth2 terms) should follow the bearer token usage ({{!RFC6750}}) which describes the format and validation of an access token obtained from the Oauth 2.0 Authorization Server. The resource server should follow the best practices for JWT access validation ({{!RFC8725}}) and in particular verify that the access token is constrained to the resource server via the audience claim. Upon successful access token validation, the resource server should decide whether to proceed with the request based on the presence of expected and matching claims in the access token or reject it altogether. The core identity and authorization claims present in the access token may be augmented with specific claims vended by the Authorization Service. This document proposes to use PeeringDB's access token claims as a baseline to use for authorization, however the specific matching of those claims to an authorization business decision is specific to each operator and outside of this specification. Resource servers may also use the claims in the access token to present the callers' identity to the application and for auditing purposes.

Proof of holdership     {#resource-holdership}
-------------------

The peering API defined in this document uses ASNs as primary identifiers to identify each party on a peering session besides other resources such as IP addresses. ASNs are explicitly expected in some API payloads but are also implicitly expected when making authorization business decisions such as listing resources that belong to an operator. Given that ASNs are Internet Number Resources assigned by RIRs and that the OAuth2 Authorization Server in use may not be operated by any of those RIRs, as it is the case of PeeringDB or any other commercial OAuth2 service, JWT claims that contain an ASN need be proved to be legitimately used by the initiator. This document proposes to attest ASN resource holdership using a mechanism based on RPKI ({{?RFC6480}}) and in particular with the use of RPKI Signed Checklists (RSCs) ({{!RFC9323}}).

JWT access tokens can be of two kinds, identifier-based tokens or self-contained tokens ({{Section 1.4 of ?RFC6749}}). Resource servers must validate them for every request. AS operators can hint other operators to validate whether a caller holds ownership of the ASN their request represent by issuing a signed checklist that is specific to the different validation methods as described below.

For Identifier-based access tokens, if the Authorization Server supports metadata, ASN holders must create a signed checklist that contains the well-known Authorization Server Metadata URI and a digest of the JSON document contained (Section 3 of {{!RFC8414}}). If the authorization server does not support metadata, the signed checklist contains the token introspection URI and its digest.

Self-contained access tokens are cryptographically signed by the token issuer using a JSON Web Signature (JWS) ({{?RFC7515}}). The cryptographic keys used for signature validation is exposed as a JSON Web Key (JWK) ({{?RFC7517}}). ASN holders must create a signed checklist for the "jwks_uri" field of the Authorization Server Metadata URI and a digest of the JSON document contained ({{Section 3.2 of !RFC8414}}).

Resource servers must validate the JWT access token in the following manner:

* If the access token is identifier-based, the resource server must resolve what introspection endpoint to use for the given access token, that is, either resolved through the Authorization Server Metadata URI ({{Section 3 of !RFC8414}}) or pre-configured upon registration in the absence of metadata support.
  * The resource server must verify the metadata URI and its content with a signed checklist issued by the ASN contained in the access token claims.
  * If the Authorization Server does not support metadata, the resource server must validate the introspection endpoint URI matches exactly the URI contained in a signed checklist issued by the ASN contained in the access token claims.
  * Upon successful signed checklist validation, resource servers must use the introspection endpoint for regular access token validation process ({{?RFC7662}}).
* If the access token is self-contained, the resource server must follow the regular validation process for signed access tokens ({{Section 5.2 of !RFC7515}}).
  * After discovering the valid public keys used to sign the token, the resource server must validate that the JWKS URI where the public keys have been discovered, and the content of such JSON document referred by it, match the signed checklist issued by the ASN contained in the access token claims.
* Resource servers must reject the request if any of these validations fail.

Request integrity and proof of possession     {#integrity-and-possession}
-----------------------------------------

The API described in this document follows REST ({{rest}}) principles over an HTTP channel to model the transfer of requests and responses between peers. Implementations of this API should use the best common practices for the API transport ({{?RFC9325}}) such as TLS. However, even in the presence of a TLS channel with OAuth2 bearer tokens alone, neither the client application nor the API can guarantee the end-to-end integrity of the message request or the authenticity of its content. One mechanism to add cryptographic integrity and authenticity validation can be the use a mutual authentication scheme to negotiate the parameters of the TLS channel. This requires the use of a web PKI ({{?RFC5280}}) to carry claims for use in authorization controls, to bind such PKI to ASNs for proof of holdership, and the use of client certificates on the application.

Instead, this document proposes to address the message integrity property by cryptographically signing the parameters of the request with a key pair that creates a HTTP message signature to be included in the request ({{!RFC9421}}). The client application controls the lifecycle of this key pair. The authenticity property of the messages signed with such key pair is addressed by binding the public key of the pair to the JWT access token in one of its claims of the access token using a mechanism that demonstrates proof of possession of the private key {{!RFC9449}}. With these two mechanisms, the resource server should authenticate, authorize, and validate the integrity of the request using a JWT access token that can rightfully claim to represent a given ASN.

IANA Considerations     {#iana}
===================

This document has no IANA actions.

--- back

Acknowledgments     {#acknowledgments}
===============

The authors would like to thank their collaborators, who implemented API versions and provided valuable feedback on the design.

* Ben Blaustein (Meta)
* Jakub Heichman (Meta)
* Stefan Pratter (20C)
* Ben Ryall (Meta)
* Erica Salvaneschi (Cloudflare)
* Job Snijders (Fastly)
* David Tuber (Cloudflare)
* Aaron Rose (Amazon)
* Prithvi Nath Manikonda (Amazon)



{:numbered="false"}

