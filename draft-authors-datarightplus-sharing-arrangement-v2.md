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

[[author]]
initials="B."
surname="Kolera"
fullname="Ben Kolera"
organization="Biza.io"
[author.address]
email = "bkolera@biza.io"

%%%

.# Abstract

This specification outlines the technical requirements related to the delivery of an enhanced establishment process for data sharing arrangements under DataRight+ and the Consumer Data Right.

Sharing Arrangement V2 is intended to be the next evolution of [@!DATARIGHTPLUS-SHARING-ARRANGEMENT-V1-00] and incorporates modern and international standards aligned practices to achieve like for like outcomes. This specification takes significant inspiration from the [@!FAPI-GRANT-MANAGEMENT] specification and the original intent based pattern first presented within the UK Open Banking scheme.

.# Notational Conventions

The keywords  "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",  "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in [@!RFC2119].

{mainmatter}

# Scope

The scope of this specification is focused on describing the authorisation related components of establishing an authentication and authorisation context for data sharing under the DataRight+ framework.  This specification achieves the same objectives as those prescribed within [@!CDS] but avoids the ecosystem specific components in favour of a more extensible and international standards aligned mechanisms.

# Terms & Definitions

This specification uses the terms "Agreement Identifier", Consumer", "Provider", "Initiator", "Personally Identifiable Information (PII)",
"Pairwise Pseudonymous Identifier (PPID)", "Initiator", "CDR Sharing Arrangement" and "CDR Sharing Agreement Identifier"  as defined by [@!DATARIGHTPLUS-ROSETTA].

Action Identifier
: A unique and discrete identifier for a given action, typically lodged at a resource server endpoint.


# High Level Process

This document, as extended further in OpenAPI format within [@!DATARIGHTPLUS-REDOCLY-ID2], describes the endpoints to deliver the Sharing Arrangement V2 capability. The current approach to data sharing within the Consumer Data Right is brittle and does not provide sufficient feedback as to the status of sharing requests as they are handed over from Initiator to the Provider. The existing sharing establishment process also assumes a live establishment rather than a potentially asynchronous, back-channel or machine to machine (IoT) authorisation approach.

At a high level the process expected to be followed through the implementation of this specification is:

1. Initiator utilises a client credentials grant to obtain a token from the Provider;
2. Initiator calls the Request Sharing Agreement endpoint and obtains an `actionId` in the initial status of `PENDING`;
3. Initiator submits a pushed authorisation request with a Request Object containing the `urn:dio:action_id` parameter with a value equal to the `actionId` returned in (2)
4. Initiator redirects the Consumer user-agent to complete the authorisation process
5. On completion of the authorisation process:
   1. the Provider, if not already assigned, assigns an `agreementId` to the associated `actionId`
   2. the Initiator performs `authorization_code` token exchange to access the Provider Resource Server resources

_Note:_ At any time before, during or after the authorisation process the Initiator can use the Get Sharing Agreement endpoint provided by the Resource Server utilising the obtained `actionId`.

# Provider

The following provisions apply to services delivered by Providers.

## Authorisation Server

In addition to the provisions outlined in [@!DATARIGHTPLUS-INFOSEC-BASELINE] the authorisation server:

1. **SHALL** support the `dio:sharing` authorisation scope;
2. **SHALL** include the `dio:sharing` authorisation scope within Dynamic Client Registration responses;

### Request Object

The request object submitted to the authorisation server:

1. **SHALL** support an optional string parameter `urn:dio:action_id` referencing a valid [Action Identifier];
2. **SHALL** reject requests containing a `urn:dio:action_id` parameter that is unknown, expired or not associated with the requesting Initiator;

#### Example

The following is a non-normative example of a decoded request object requesting authorisation for a previously lodged `actionId`

```json
{
  "iss": "33ff17d6-d967-4823-a4d3-1206fd2fff3f",
  "exp": 1725090872,
  "nbf": 1725090872,
  "aud": "https://www.provider.com.au",
  "response_type": "code",
  "response_mode": "jwt",
  "client_id": "33ff17d6-d967-4823-a4d3-1206fd2fff3f",
  "redirect_uri": "https://www.recipient.com.au/coolstuff",
  "scope": "openid",
  "claims": {
    "urn:dio:action_id": "496a3ba7-04b4-4362-b775-9e0433e48eea",
    "id_token": {
      "acr": {
        "essential": true,
        "values": ["urn:cds.au:cdr:3"]
      }
    },
    "userinfo": {
      "given_name": null,
      "family_name": null
    }
  },
  "code_challenge": "rerbvXfTDYNECzwayM8-SLCWU1FDzBnqMCv1RB5AudU",
  "code_challenge_method": "S256"
}
```

### Claims

The authorisation server:

1. **SHALL** include within the Introspection Endpoint response:
    1. the `exp` claim, with a value equal to the expiry time of the underlying Sharing Agreement and;
    2. the `urn:dio:action_id` claim, containing the assigned [Action Identifier];

### CDR Arrangement V1 Compatibility

In order to maximise backward compatibility and facilitate orderly transition for endpoints already implemented the Authorisation Server **SHALL** ensure that the following authorisation scopes are mapped to Data Cluster values which are successfully authorised:

| `EnumDataClusterV1` Value                      | Authorization Scope                            |
|------------------------------------------------|------------------------------------------------|
| `OPENID`                                       | `openid`                                       |
| `PROFILE`                                      | `profile`                                      |
| `BANK_ACCOUNTS_BASIC_READ`                     | `bank:accounts.basic:read`                     |
| `BANK_ACCOUNTS_DETAIL_READ`                    | `bank:accounts.detail:read`                    |
| `BANK_TRANSACTIONS_READ`                       | `bank:transactions:read`                       |
| `BANK_REGULAR_PAYMENTS_READ`                   | `bank:regular_payments:read`                   |
| `BANK_PAYEES_READ`                             | `bank:payees:read`                             |
| `ENERGY_ACCOUNTS_BASIC_READ`                   | `energy:accounts.basic:read`                   |
| `ENERGY_ACCOUNTS_DETAIL_READ`                  | `energy:accounts.detail:read`                  |
| `ENERGY_ACCOUNTS_CONCESSIONS_READ`             | `energy:accounts.concessions:read`             |
| `ENERGY_ACCOUNTS_PAYMENTSCHEDULE_READ`         | `energy:accounts.paymentschedule:read`         |
| `ENERGY_BILLING_READ`                          | `energy:billing:read`                          |
| `ENERGY_ELECTRICITY_SERVICEPOINTS_BASIC_READ`  | `energy:electricity.servicepoints.basic:read`  |
| `ENERGY_ELECTRICITY_SERVICEPOINTS_DETAIL_READ` | `energy:electricity.servicepoints.detail:read` |
| `ENERGY_ELECTRICITY_DER_READ`                  | `energy:electricity.der:read`                  |
| `ENERGY_ELECTRICITY_USAGE_READ`                | `energy:electricity.usage:read`                |
| `COMMON_CUSTOMER_BASIC_READ`                   | `common:customer.basic:read`                   |
| `COMMON_CUSTOMER_DETAIL_READ`                  | `common:customer.detail:read`                  |

## Resource Server

The Provider Resource Server:

1. **SHALL** support the `requestDataSharingAgreement` and `getDataSharingAgreement` endpoints as described in [@!DATARIGHTPLUS-REDOCLY-ID2];
2. **SHALL** support [@!DATARIGHTPLUS-DISCOVERY-V1-01] and advertise the `requestDataSharingAgreement` and `getDataSharingAgreement` endpoints
3. **SHALL** support providing an existing `agreementId` in order to extend an existing agreement in subsequent Request Sharing Agreement requests
4. **MAY** support Consumer Type (`consumerType`) authorisation filtering and, if supported, include the `SUPPORTS_CONSUMER_TYPE` flag at the `requestDataSharingAgreement` endpoint
5. **MAY** support record filtering by date (`oldestDate`/`newestDate`) and, if supported, include the `SUPPORTS_DATE_FILTER` flag at the `requestDataSharingAgreement` endpoint

# Initiator

The following provisions apply to participants operating Initiators.

## Authorisation Client

In addition to the provisions outlined in Section 4 of [@!DATARIGHTPLUS-INFOSEC-BASELINE] the Initiator authorisation client:

1. **SHALL** support the Initiator provisions of [@!DATARIGHTPLUS-DISCOVERY-V1-01] to discover the `getDataSharingAgreement` and `requestDataSharingAgreement` endpoints;
2. **SHALL** perform a Dynamic Client Registration update, as described in [@!DATARIGHTPLUS-ADMISSION-CONTROL-00], to be granted access to the `dio:sharing` scope;
3. **SHALL** only include optional Request Sharing Agreement parameters if they are advertised within the Provider discovery document

# Implementation Considerations

This specification does not explicitly state how long an assigned Action Identifier persists for however it is expected that an Initiator shall be able to reference an Action Identifier while the associated authorisation is still active and for a significant period (more than a year) after it becomes inactive.

# Security Considerations

The Action Identifier **SHALL NOT** be guessable, derivable nor identify the Consumer.

{backmatter}

<reference anchor="CDS" target="https://consumerdatastandardsaustralia.github.io/standards"><front><title>Consumer Data Standards (CDS)</title><author><organization>Data Standards Body (Treasury)</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-ROSETTA" target="https://datarightplus.github.io/datarightplus-rosetta/draft-authors-datarightplus-rosetta.html"> <front><title>DataRight+ Rosetta Stone</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-REDOCLY-ID2" target="https://datarightplus.github.io/datarightplus-redocly/?v=ID2"> <front><title>DataRight+: Redocly (ID2)</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author><author initials="B." surname="Kolera" fullname="Ben Kolera"><organization>Biza.io</organization></author>
<author initials="W." surname="Cai" fullname="Wei Cai"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-INFOSEC-BASELINE" target="https://datarightplus.github.io/datarightplus-infosec-baseline/draft-authors-datarightplus-infosec-baseline.html"> <front><title>DataRight+ Security Profile: Baseline</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-ADMISSION-CONTROL-00" target="https://datarightplus.github.io/datarightplus-admission-control-baseline/draft-authors-datarightplus-admission-control-00/draft-authors-datarightplus-admission-control.html"> <front><title>DataRight+: Admission Control Baseline</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author><author initials="B." surname="Kolera" fullname="Ben Kolera"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-SHARING-ARRANGEMENT-V1-00" target="https://datarightplus.github.io/datarightplus-sharing-arrangement-v1/draft-authors-datarightplus-sharing-arrangement-v1-00/draft-authors-datarightplus-sharing-arrangement-v1.html"> <front><title>DataRight+: Sharing Arrangement V1</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author><author initials="B." surname="Kolera" fullname="Ben Kolera"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-DISCOVERY-V1-01" target="https://datarightplus.github.io/datarightplus-discovery-v1/draft-authors-datarightplus-discovery-v1-01/draft-authors-datarightplus-discovery-v1.html"> <front><title>DataRight+: Provider Discovery V1</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="FAPI-GRANT-MANAGEMENT" target="https://openid.net/specs/fapi-grant-management.html"> <front> <title>The OAuth 2.0 Authorization Framework</title><author fullname="D. Hardt"> <organization>Microsoft</organization> </author><date month="Oct" year="2012"/></front> </reference>

