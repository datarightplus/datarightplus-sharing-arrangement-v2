%%%
title = "DataRight+: Sharing Arrangement V2"
area = "Internet"
workgroup = "datarightplus"
submissionType = "independent"

[seriesInfo]
name = "Internet-Draft"
value = "draft-authors-datarightplus-sharing-arrangement-v2-latest"
stream = "independent"
status = "experimental"

date = 2024-05-02T00:00:00Z

[[author]]
initials="S."
surname="Low"
fullname="Stuart Low"
organization="Biza.io"
[author.address]
email = "stuart@biza.io"

%%%

.# Abstract

This specification outlines the technical requirements related to the delivery of Sharing Arrangement V2.

Sharing Arrangement V2 is intended to be the next evolution of [@!DATARIGHTPLUS-SHARING-ARRANGEMENT-V1-00] and incorporates modern and international standards aligned practices to achieve like for like outcomes. This specification takes significant inspiration from the [@!FAPI-GRANT-MANAGEMENT] specification.

.# Notational Conventions

The keywords  "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",  "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in [@!RFC2119].

{mainmatter}

# Scope

The scope of this specification is focused on describing the authorisation related components of establishing the equivalent of a CDR Sharing Arrangement. While this specification achieves the same objectives as those prescribed within [@!CDS] it avoids the ecosystem specific components in favour of a more extensible and international standards aligned mechanisms. To maintain ecosystem interoperability, specific provisions related to [@!DATARIGHTPLUS-SHARING-ARRANGEMENT-V1-00] are included where concurrent support is intended.

# Terms & Definitions

This specification uses the terms "Consumer", "Provider", "Initiator", "Personally Identifiable Information (PII)",
"Pairwise Pseudonymous Identifier (PPID)", "Initiator", "CDR Sharing Arrangement" and "CDR Sharing Agreement Identifier"  as defined by [@!DATARIGHTPLUS-ROSETTA].

Agreement Identifier: TODO

# Provider

The following provisions apply to services delivered by Providers.

## Authorisation Server

In addition to the provisions outlined in [@!DATARIGHTPLUS-INFOSEC-BASELINE] the authorisation server:

1. **SHALL** support [@!RFC9396] including the `authorization_details` payload described in [Sharing Authorization Request];
2. **SHALL**, where the supplied `cdr_arrangement_id`, if provided, does not match the authenticated Consumer and as soon as practicable, return an RFC6749 Error
3. **SHALL** modify the existing authorisation grant referenced by the `cdr_arrangement_id`, if provided, while maintaining the `cdr_arrangement_id` between such authorisations
4. **SHALL** revoke previously issued Access and refresh tokens when modifying existing authorisation grants;
5. **SHALL** support the `sharing_arrangement_v2_supported` boolean as outlined in Section X of [@!DATARIGHTPLUS-DISCOVERY-V1-00]
6. **SHOULD** support the `agreement_management_endpoint` attribute as outlined in Section X of [@!DATARIGHTPLUS-DISCOVERY-V1-00]

### Sharing Request

The authorisation server **SHALL** support [@!RFC9396] authorisations requests at the [@!RFC9126] endpoint with a `authorization_details` payload containing the following attributes:

1. `grant_type` **REQUIRED**: Static value of `sharing_arrangement_v2` in order to provide JSON `oneOf` derivable parsing
2. `sharing_arrangement_v2` **REQUIRED**: A JSON object containing the following attributes:
   1. `sharing_duration` **REQUIRED**: An integer parameter between `0` and `31536000` representing the number of seconds requested for data sharing. 
   2. `agreement_id` **OPTIONAL**: A reference to a previously established Agreement Identifier 
   3. `data_sets` **REQUIRED**: A list of Data Sets, in authorisation scope format.

A non-normative example of the expected payload, representing a sharing request for 24hrs to extend an existing grant, is provided below:

```json
{
    "authorization_details": {
       "grant_type": "sharing_arrangement_v2",
       "sharing_arrangement_v2": {
          "sharing_duration": 86400,
          "agreement_id": "c4ac0c9d0-457d-4578-b0cd-52e443ae13c5",
          "data_sets": [
             "common:customer.basic:read common:customer.detail:read"
          ]
       }
    }
}
```

### RFC9126 Response

It is often desirable for an Initiator to be able to ascertain, prior to completion of an authorisation by a Consumer, the status of the authorisation, consequently, in addition to the provisions of [@!RFC9126] and where a valid [Sharing Request] is received:
1. **SHOULD** disclose an assigned `agreement_id` attribute within the [@!RFC9126] endpoint
2. **SHALL** ensure the `agreement_id` value remains consistent on completion of the previously specified [Authorisation Request]
3. **SHALL**, where the [Sharing Request] contains a reference to an existing `agreement_id` retain this `agreement_id` unchanged
4. **MAY** discard `agreement_id` values for which authorisation has not been completed within a reasonable period of time

A non-normative example is provided as follows:

```json
{
   "request_uri": "urn:abc/efg",
   "expires_in": 60,
   "agreement_id": "c4ac0c9d0-457d-4578-b0cd-52e443ae13c5"
}
```

### Token Response

The authorisation server **SHALL** support [@!RFC9396] responses at its [@!RFC6749] token response endpoint with a `authorization_details` payload containing the following attributes:

1. `grant_type` **REQUIRED**: Static value of `sharing_arrangement_v2` in order to provide JSON `oneOf` derivable parsing
2. `sharing_arrangement_v2` **REQUIRED**: A JSON object containing the following attributes:
   1. `sharing_expires_at` **REQUIRED**: An integer parameter representing the epoch of when the attached Grant expires
   2. `agreement_id` **REQUIRED**: The attached Agreement Identifier
   3. `data_sets` **REQUIRED**: A list of Data Sets, in authorisation scope format
   4. `account_ids` **OPTIONAL**: A list of Account Identifiers included within this Agreement

A non-normative example of the expected payload, representing a sharing request for 24hrs to extend an existing grant, is provided below:

```json
{
    "authorization_details": {
       "agreement_id": "c4ac0c9d0-457d-4578-b0cd-52e443ae13c5",
       "arrangement_type": "sharing_arrangement_v2",
       "sharing_arrangement_v2": {
          "sharing_expires_at": 12345678,
          "data_sets": [
             "common:customer.basic:read common:customer.detail:read"
          ],
          "account_ids": [
             "b7ec0c9d0-457d-4578-b0cd-52e443a78bef",
             "3faac0c9d0-457d-4578-b0cd-52e443ae90afa",
          ]
       }
    }
}
```

### CDR Arrangement V1 Compatibility

In order to maximise backward compatibility and facilitate orderly transition it is necessary to ensure participants continue to support historical behaviours. Therefore, where the authorization server supports this specification and [@!DATARIGHTPLUS-SHARING-ARRANGEMENT-V1-00] concurrently it **SHALL**:
1. consider compliant `authorization_details` payloads as authoritative;
2. fallback to the behaviours outlined within Section 3 of [@!DATARIGHTPLUS-SHARING-ARRANGEMENT-V1-00] where `authorization_details` is absent;
3. generate `agreement_id` values equal to `cdr_arrangement_id` values for `grant_type` of `sharing_arrangement_v2`
4. return an [@!RFC6749] token response `scopes`  parameter equivalent to the `data_sets` specified in [Sharing Authorization Response];
5. specify the [@!RFC6749] `exp` value visible in Refresh Token introspection responses to be equal to `sharing_expires_at`;
6. Perform the same functions, in the same way, as those previously outlined in [@!DATARIGHTPLUS-SHARING-ARRANGEMENT-V1-00] for relevant actions taken on authorization grants of `sharing_arrangement_v2` type

# Initiator

The following provisions apply to participants operating Initiators.

## Authorisation Client

In addition to the provisions outlined in Section 4 of [@!DATARIGHTPLUS-INFOSEC-BASELINE] the Initiator authorisation client:

1. **SHALL** support the [Rich Authorization Request];
2. **SHALL** only issue authorisation requests defined in this specification where the Provider has advertised `sharing_arrangement_v2_supported` via the discovery endpoint specified within [@!DATARIGHTPLUS-DISCOVERY-V1-00]

### CDR Arrangement V1 Compatibility

In order to maximise backward compatibility and facilitate orderly transition it is necessary to ensure participants continue to support historical behaviours. Therefore, the authorisation client **SHALL** support the provisions outlined in Section 4 of [@!DATARIGHTPLUS-SHARING-ARRANGEMENT-V1-00]. 

This provision results in the duplication of the parameters of the request in both formats. As a result the following non-normative example is provided for the Request Object submitted to the [@!RFC9126] endpoint:

```json
{
   "scopes": "common:customer.basic:read",
   "sharing_duration": 86400,
   "cdr_arrangement_id": "c4ac0c9d0-457d-4578-b0cd-52e443ae13c5",
   "authorization_details": {
      "grant_type": "sharing_arrangement_v2",
      "sharing_arrangement_v2": {
         "sharing_duration": 86400,
         "agreement_id": "c4ac0c9d0-457d-4578-b0cd-52e443ae13c5",
         "data_sets": [
            "common:customer.basic:read"
         ]
      }
   }
}

```

# Implementation Considerations

TODO: Agreement Identifier expiration from RFC9126

# Security Considerations

The Agreement Identifier **SHALL NOT** be guessable, derivable nor identify the Consumer.

{backmatter}

<reference anchor="CDS" target="https://consumerdatastandardsaustralia.github.io/standards"><front><title>Consumer Data Standards (CDS)</title><author><organization>Data Standards Body (Treasury)</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-ROSETTA" target="https://datarightplus.github.io/datarightplus-rosetta/draft-authors-datarightplus-rosetta.html"> <front><title>DataRight+ Rosetta Stone</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-INFOSEC-BASELINE" target="https://datarightplus.github.io/datarightplus-infosec-baseline/draft-authors-datarightplus-infosec-baseline.html"> <front><title>DataRight+ Security Profile: Baseline</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="OIDC-Discovery" target="https://openid.net/specs/openid-connect-discovery-1_0.html"> <front> <title>OpenID Connect Discovery 1.0 incorporating errata set 1</title> <author initials="N." surname="Sakimura" fullname="Nat Sakimura"> <organization>NRI</organization> </author> <author initials="J." surname="Bradley" fullname="John Bradley"> <organization>Ping Identity</organization> </author> <author initials="M." surname="Jones" fullname="Mike Jones"> <organization>Microsoft</organization> </author> <author initials="E." surname="Jay"> <organization>Illumila</organization> </author><date day="8" month="Nov" year="2014"/> </front> </reference>

<reference anchor="JWT" target="https://datatracker.ietf.org/doc/html/rfc7519"> <front> <title>JSON Web Token (JWT)</title> <author fullname="M. Jones"> <organization>Microsoft</organization> </author> <author initials="J." surname="Bradley" fullname="John Bradley"> <organization>Ping Identity</organization> </author><author fullname="N. Sakimura"> <organization>Nomura Research Institute</organization> </author> <date month="May" year="2015"/></front> </reference>

<reference anchor="RFC6749" target="https://datatracker.ietf.org/doc/html/rfc6749"> <front> <title>The OAuth 2.0 Authorization Framework</title><author fullname="D. Hardt"> <organization>Microsoft</organization> </author><date month="Oct" year="2012"/></front> </reference>

<reference anchor="DATARIGHTPLUS-SHARING-ARRANGEMENT-V1-00" target="https://datarightplus.github.io/datarightplus-sharing-arrangement-v1/draft-authors-datarightplus-sharing-arrangement-v1-00/draft-authors-datarightplus-sharing-arrangement-v1.html"> <front><title>DataRight+: Sharing Arrangement V1</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author><author initials="B." surname="Kolera" fullname="Ben Kolera"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-DISCOVERY-V1-00" target="https://datarightplus.github.io/datarightplus-discovery-v1/draft-authors-datarightplus-discovery-v1-00/draft-authors-datarightplus-discovery-v1-v1.html"> <front><title>DataRight+: Provider Discovery V1</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>


<reference anchor="RFC6749" target="https://datatracker.ietf.org/doc/html/rfc6749"> <front> <title>The OAuth 2.0 Authorization Framework</title><author fullname="D. Hardt"> <organization>Microsoft</organization> </author><date month="Oct" year="2012"/></front> </reference>

<reference anchor="RFC9396" target="https://datatracker.ietf.org/doc/html/rfc9396"> <front> <title>Grant Management for OAuth 2.0</title><author fullname="T. Lodderstedt"> <organization>yes.com</organization> </author><author fullname="S. Low"> <organization>Biza.io</organization> </author><author fullname="D. Postnikov"> <organization>Independent</organization> </author><date month="Jul" year="2021"/></front> </reference>

<reference anchor="FAPI-GRANT-MANAGEMENT" target="https://openid.net/specs/fapi-grant-management.html"> <front> <title>The OAuth 2.0 Authorization Framework</title><author fullname="D. Hardt"> <organization>Microsoft</organization> </author><date month="Oct" year="2012"/></front> </reference>

<reference anchor="RFC9126" target="https://datatracker.ietf.org/doc/html/rfc9126"> <front> <title>OAuth 2.0 Pushed Authorization Requests</title>
<author initials="T." surname="Lodderstedt" fullname="Torsten Lodderstedt">
      <organization>yes.com</organization>
    </author>
    <author initials="B." surname="Campbell" fullname="Brian Campbell">
      <organization>Ping Identity</organization>
    </author>
    <author initials="N." surname="Sakimura" fullname="Nat Sakimura">
      <organization showOnFrontPage="true">NAT.Consulting</organization>
    </author>
    <author initials="D." surname="Tonge" fullname="Dave Tonge">
      <organization showOnFrontPage="true">Moneyhub Financial Technology</organization>
    </author>
    <author initials="F." surname="Skokan" fullname="Filip Skokan">
      <organization showOnFrontPage="true">Auth0</organization>
    </author>
    <date month="09" year="2021"/></front></reference>