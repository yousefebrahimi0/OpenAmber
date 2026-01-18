# OpenAmber Protocol Specification

**Version:** 0.1.0-draft
**Status:** Pre-implementation draft
**Author:** Yousef Ebrahimi
**Last Updated:** January 2026

## Abstract

This document specifies OpenAmber, an open protocol for creating, signing, distributing, and verifying missing child alerts across organizational boundaries. The protocol enables interoperability between European child protection organizations while ensuring cryptographic authenticity and GDPR compliance.

---

## 1. Introduction

### 1.1 Purpose

OpenAmber defines: 
- A standardized data format for missing child alerts (JSON-LD)
- A cryptographic signing mechanism for alert authenticity (Ed25519)
- A federation model for alert distribution across organizations
- A verification protocol for consumers to validate alerts

### 1.2 Design Goals

1. **Interoperability:** Alerts created by any compliant issuer can be processed by any compliant relay or consumer
2. **Verifiability:** Any party can cryptographically verify an alert's authenticity without contacting the issuer
3. **Resilience:** No single point of failure; federated distribution ensures availability
4. **Privacy:** GDPR-compliant data minimization with automatic expiry mechanisms
5. **Simplicity:** Implementable by organizations with limited technical resources

### 1.3 Scope

OpenAmber specifies the **protocol layer** for alert interchange.  It does NOT specify:
- Case management systems
- Investigation workflows
- User interfaces (beyond reference implementations)
- Operational policies for when to issue alerts

### 1.4 Terminology

| Term | Definition |
|------|------------|
| **Alert** | A structured message containing information about a missing child |
| **Issuer** | An organization authorized to create and sign alerts |
| **Relay** | A server that validates, stores, and redistributes alerts |
| **Consumer** | Any system or person that receives and displays alerts |
| **Signature** | Cryptographic proof binding an alert to its issuer |

---

## 2. Data Model

### 2.1 Alert Object

An OpenAmber Alert is a JSON-LD document with the following structure:

#### Required Properties

| Property | Type | Description |
|----------|------|-------------|
| `@context` | Array | JSON-LD context, must include OpenAmber namespace |
| `type` | String | Must be `MissingChildAlert` |
| `id` | URI | Globally unique identifier (URN format) |
| `issuer` | IssuerReference | Organization that created the alert |
| `subject` | SubjectDescription | Information about the missing child |
| `created` | DateTime | ISO 8601 timestamp of alert creation |
| `expires` | DateTime | ISO 8601 timestamp when alert should be removed |
| `signature` | Signature | Cryptographic signature from issuer |

#### Optional Properties

| Property | Type | Description |
|----------|------|-------------|
| `lastSeen` | LocationEvent | Last known location and time |
| `contactInfo` | ContactInfo | How to report sightings |
| `circumstances` | String | Brief description of disappearance circumstances |
| `companionDescription` | String | Description of person child may be with |
| `vehicleDescription` | String | Vehicle description if applicable |
| `urgencyLevel` | Enum | `critical`, `high`, `standard` |
| `languageCode` | String | ISO 639-1 language code |
| `translations` | Array | Translations of text fields |

### 2.2 Subject Description

Information about the missing child, designed for **data minimization**. 

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `givenName` | String | Yes | Child's first name |
| `familyName` | String | No | Child's surname (may be withheld for privacy) |
| `age` | Integer | Yes | Age in years |
| `gender` | String | No | `male`, `female`, `other`, `unknown` |
| `physicalDescription` | String | Yes | Height, hair color, eye color, distinguishing features |
| `photoHash` | String | No | SHA-256 hash of official photo (format: `sha256:hex`) |
| `photoUrl` | URI | No | URL to retrieve photo (from issuer's secure server) |
| `clothingDescription` | String | No | What child was wearing when last seen |

### 2.3 Location Event

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `description` | String | Yes | Human-readable location description |
| `coordinates` | Array | No | `[latitude, longitude]` in WGS84 |
| `timestamp` | DateTime | Yes | When child was last seen at this location |
| `country` | String | No | ISO 3166-1 alpha-2 country code |
| `region` | String | No | Administrative region/state |

### 2.4 Issuer Reference

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | URI | Yes | Issuer's OpenAmber profile URL |
| `name` | String | Yes | Organization name |
| `country` | String | Yes | ISO 3166-1 alpha-2 country code |
| `type` | String | No | `police`, `hotline`, `ngo`, `government` |

### 2.5 Signature Object

| Property | Type | Description |
|----------|------|-------------|
| `type` | String | Must be `Ed25519Signature2020` |
| `created` | DateTime | Timestamp of signature creation |
| `verificationMethod` | URI | URL to issuer's public key |
| `signatureValue` | String | Base64-encoded Ed25519 signature |

---

## 3. Cryptographic Operations

### 3.1 Key Management

Issuers MUST maintain an Ed25519 key pair: 
- **Private key:** Securely stored, used to sign alerts
- **Public key:** Published at issuer's well-known URL

Public key discovery URL format: 
```
https://{issuer-domain}/.well-known/openamber/keys. json
```

### 3.2 Signing Process

1. Construct the alert object WITHOUT the `signature` property
2. Serialize to canonical JSON (RFC 8785 JSON Canonicalization Scheme)
3. Compute SHA-256 hash of canonical JSON
4. Sign the hash using issuer's Ed25519 private key
5. Encode signature as Base64
6. Add `signature` object to alert

### 3.3 Verification Process

1. Extract and remove `signature` object from alert
2. Serialize remaining alert to canonical JSON
3. Compute SHA-256 hash
4. Retrieve issuer's public key from `verificationMethod` URL
5. Verify Ed25519 signature against hash and public key
6. Check `created` timestamp is within acceptable window
7. Check `expires` has not passed

### 3.4 Signature Validity

A signature is considered **valid** if:
- Ed25519 verification succeeds
- Signing timestamp is not in the future
- Alert has not expired
- Issuer's key has not been revoked

---

## 4. Federation Protocol

### 4.1 Alert Distribution

Alerts propagate through a network of relays:

```
Issuer ──▶ Primary Relays ──▶ Secondary Relays ──▶ Consumers
                │                    │
                └────────────────────┘
                  (Cross-relay sync)
```

### 4.2 Relay Responsibilities

1. **Validate** incoming alerts (signature verification)
2. **Store** valid alerts until expiration
3. **Distribute** to subscribers via supported protocols
4. **Purge** expired alerts automatically

### 4.3 Distribution Protocols

Relays SHOULD support multiple distribution methods: 

| Protocol | Format | Use Case |
|----------|--------|----------|
| **REST API** | JSON | Programmatic access |
| **RSS/Atom** | XML | Feed readers, simple integration |
| **WebSocket** | JSON | Real-time notifications |
| **ActivityPub** | JSON-LD | Fediverse integration |
| **Webhook** | JSON | Push notifications to subscribers |

### 4.4 Trust Policies

Relays MAY implement trust policies: 
- **Whitelist:** Only relay alerts from known issuers
- **Web of Trust:** Relay alerts from issuers vouched by trusted organizations
- **Open:** Relay any validly-signed alert (not recommended)

---

## 5. Alert Lifecycle

### 5.1 States

| State | Description |
|-------|-------------|
| `active` | Alert is current and should be displayed |
| `resolved` | Child has been found; alert should be removed |
| `expired` | Expiration timestamp has passed |
| `revoked` | Issuer has invalidated the alert |

### 5.2 Resolution

When a child is found, issuers SHOULD publish a **Resolution Notice**:

```json
{
  "@context": ["https://openamber.eu/ns/v1"],
  "type": "AlertResolution",
  "alertId": "urn:openamber:alert: BE-2026-00142",
  "resolution": "found_safe",
  "resolvedAt": "2026-01-16T10:30:00Z",
  "signature": { ... }
}
```

Resolution types:  `found_safe`, `found_deceased`, `returned_home`, `false_alarm`, `withdrawn`

### 5.3 Revocation

Issuers can revoke alerts (e.g., issued in error):

```json
{
  "@context": ["https://openamber.eu/ns/v1"],
  "type": "AlertRevocation",
  "alertId": "urn:openamber:alert:BE-2026-00142",
  "reason": "issued_in_error",
  "revokedAt": "2026-01-15T16:00:00Z",
  "signature": { ... }
}
```

---

## 6. Privacy Considerations

### 6.1 Data Minimization

Alerts MUST contain only information necessary for identification: 
- ✅ First name, age, physical description
- ✅ Last known location (general area)
- ❌ Home address, school name, family details
- ❌ Medical information (unless critical for identification)
- ❌ National ID numbers

### 6.2 Photo Handling

To prevent unauthorized photo retention:
- Alert contains only `photoHash` (SHA-256 of image)
- Actual photo retrieved from issuer's server via `photoUrl`
- Issuer can revoke photo access when alert resolves
- Relays SHOULD NOT cache photos beyond display needs

### 6.3 Automatic Expiry

- All alerts MUST have an `expires` timestamp
- Maximum expiry: 30 days from creation
- Relays MUST delete expired alerts within 24 hours
- Consumers MUST stop displaying expired alerts

### 6.4 Right to Erasure

When an alert is resolved or revoked:
- Issuers MUST publish resolution/revocation notice
- Relays MUST remove alert within 24 hours of notice
- Consumers MUST stop displaying alert immediately

---

## 7. Security Considerations

### 7.1 Key Compromise

If an issuer's private key is compromised:
1. Issuer publishes key revocation notice
2. Issuer generates new key pair
3. Relays reject alerts signed with revoked key
4. Issuer re-signs any still-active alerts with new key

### 7.2 Replay Attacks

Mitigated by:
- Unique alert `id` (relays reject duplicates)
- `created` timestamp (reject alerts too old)
- `expires` timestamp (alerts become invalid)

### 7.3 Denial of Service

Relays SHOULD implement: 
- Rate limiting per issuer
- Maximum alert size limits
- Suspicious pattern detection

### 7.4 Hoax Prevention

Cryptographic signatures prevent unauthorized alert creation.  Additionally:
- Issuer verification through web-of-trust
- Relays can require issuer pre-registration
- Public key pinning for known issuers

---

## 8. Conformance

### 8.1 Issuer Conformance

An issuer is conformant if it:
- Publishes alerts matching this specification's schema
- Signs alerts using Ed25519 as specified
- Publishes public keys at well-known URL
- Publishes resolution notices when children are found
- Respects maximum expiry limits

### 8.2 Relay Conformance

A relay is conformant if it:
- Validates all alert signatures before redistribution
- Rejects invalid or expired alerts
- Removes resolved/revoked alerts within 24 hours
- Supports at least one distribution protocol (REST API minimum)
- Respects issuer trust policies

### 8.3 Consumer Conformance

A consumer is conformant if it:
- Verifies alert signatures before display
- Does not display expired alerts
- Removes resolved/revoked alerts promptly
- Links to issuer contact information

---

## Appendix A: JSON-LD Context

See [schemas/context.jsonld](../schemas/context.jsonld)

## Appendix B: Example Alert

See [examples/example-alert.json](../examples/example-alert.json)

## Appendix C: References

- [RFC 8785: JSON Canonicalization Scheme](https://tools.ietf.org/html/rfc8785)
- [Ed25519 Signature Suite 2020](https://w3c-ccg.github.io/lds-ed25519-2020/)
- [JSON-LD 1.1](https://www.w3.org/TR/json-ld11/)
- [GDPR Article 6: Lawfulness of Processing](https://gdpr-info.eu/art-6-gdpr/)

---

*This specification is a working draft subject to revision based on community feedback, stakeholder consultation, and implementation experience.*
