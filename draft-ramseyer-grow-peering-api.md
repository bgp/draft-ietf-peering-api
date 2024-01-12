---
title: "Peering API"
category: info

docname: draft-ramseyer-grow-peering-api-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: GROW
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
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
(Jenny--this is not up-to-date, but I pasted in what we had in the google doc and will revise)
TODO: Update this spec, include API endpoints

## full list of endpoints:
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

## Request flow
1. AUTH phase: initiator makes an authenticated request to receiver via PeeringDB OIDC.  This provides the receiver with initiator’s credentials to verify who they say they are
2. REQUEST phase:
    1. ADD: What is the initial information provided
        * Your ASN
          1. Can use internal tools to check traffic levels
          2. Cross reference with OAUTH data to verify ASN as the same one received in the OAUTH token
          3. Can get prefix limit counters
              1. Not needed in handshake but could be allowed as an optional flag
       * Peering Type: PNI or IX (Private or Public - however we want to brand it). This will be useful later when we want to differentiate between a public payload and a private one since they will look different
       * PeeringDB/IXP IDs that you want to peer on (this allows you to get the peering addresses)
    2. REMOVE: What is the initial information provided
        1. Your ASN
            * Cross reference with OAUTH data to verify ASN as the same one received in the OAUTH token
        2. IXP ID
3. APPROVAL: What does the other side return?
    1. Dictionary
        1. Key: IXP ID
        2. Value: Bool indicating whether or not peering is acceptable at this location
           * True: yes we can peer
           * False: no we can’t
             * Will return false if we’re already peered at that location
             * Counterproposal: suggest different configuration option.  (in VLater)
    2. Prefix limit counters (optional value)
    3. TimeWindow: Time window indicating when sessions will be configured after being notified (may be 0 if sessions are already configured on receiver side)
    4. isInboundFiltered: optional bool that indicates whether prefixes will be filtered inbound.  If this is set to true the time window should be set to how long the prefixes will be filtered for.
    5. isOutboundFiltered: optional bool that indicates whether prefixes will be filtered outbound.  If this is set to true, the time window should be set to how long the prefixes will be filtered for.  If the outbound limit is longer than the inbound limit time, the time window should be set to the max of inbound versus outbound.
4. Initiator removes sessions where receiver does not want to peer
    1. For every IXP ID where bool = false, remove sessions from the dictionary.  This handles the case where a user may initiate requests with all possible peering sessions and receiver only wants to peer in new locations.  The initiator will then filter out duplicates before entering the CONFIG/MONITOR state.
5. CONFIG/MONITOR: Initiator waits for maximum time window and then notifies receiver for any outstanding sessions that have not been established
    1. Initiator sends dictionary
        1. IXP ID
        2. Established: bool
        3. Sent prefixes (0 if Established is false)
        4. Received prefixes (0 if Established is false)
        5. Accepted Prefixes
    2. Receiver responds with dictionary
        1. IXP ID
        2. Established: bool
        3. Sent prefixes
        4. Received prefixes
        5. Accepted Prefixes
    3. If established = true, sentPrefixes > 0, and acceptedPrefixes > 0
        1. You’re done
        2. Initiator closes connection with a GOODBYE.  Any subsequent followups must start this process over again.  If the receiver has responded with a GOODBYE, then any subsequent monitoring calls will receive a GOODBYE.  If a GOODBYE was presented erroneously, the initiator can make a new request asking the receiver to delete the session and retry.
    4. Else, wait TimeWindow and retry
    5. If initiator does not make a request in 3*TimeWindow or 1 week (whichever is longer), receiver automatically sends a GOODBYE and deletes the sessions on their side.

# API Endpoints and Specifications
TODO: list endpoints, both v0 and vLater
TODO: Include diagram and openapi spec?

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
