# OpenAmber Trust Model

## Overview

OpenAmber's trust model ensures that only authorized organizations can issue alerts, while maintaining the decentralized nature of the network. This document describes how trust is established, propagated, and verified.

## Trust Hierarchy

```
┌──────────────────────────────────────────────────────────────┐
│                     ROOT ANCHORS                             │
│  (Well-known organizations with established credibility)     │
│                                                              │
│  • Missing Children Europe (MCE)                             │
│  • National 116 000 Hotline Operators                        │
│  • EU Member State Police Forces                             │
└──────────────────────────────────────────────────────────────┘
                              │
                              │ Vouches for
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   VERIFIED ISSUERS                          │
│  (Organizations vouched by root anchors)                    │
│                                                             │
│  • Regional NGOs                                            │
│  • Local Police Departments                                 │
│  • Child Protection Agencies                                │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ Publishes alerts to
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                      RELAYS                                  │
│  (Validate and redistribute alerts)                          │
│                                                              │
│  • MCE Central Relay                                         │
│  • Media Partner Relays                                      │
│  • Community Relays                                          │
└──────────────────────────────────────────────────────────────┘
                              │
                              │ Distributes to
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     CONSUMERS                               │
│  (Verify signatures independently)                          │
│                                                             │
│  • News Websites                                            │
│  • Mobile Apps                                              │
│  • Public Displays                                          │
└─────────────────────────────────────────────────────────────┘
```

## Issuer Verification

### Becoming a Verified Issuer

Organizations seeking to issue OpenAmber alerts must:

1. **Demonstrate legitimacy:**
   - Registered child protection organization, OR
   - Law enforcement agency, OR
   - Government child welfare department

2. **Obtain vouching from root anchor:**
   - Contact MCE or national 116 000 operator
   - Provide organizational credentials
   - Complete verification process

3. **Publish public key:**
   - Generate Ed25519 key pair
   - Publish public key at well-known URL
   - Register with vouching organization

### Vouching Mechanism

Root anchors vouch for issuers by signing their key registration:

```json
{
  "@context": ["https://openamber.eu/ns/v1"],
  "type": "IssuerVouch",
  "issuer": {
    "id": "https://neworg.example/. well-known/openamber",
    "name": "New Child Protection Org",
    "country": "DE"
  },
  "vouchedBy": {
    "id": "https://missingchildreneurope.eu/.well-known/openamber",
    "name": "Missing Children Europe"
  },
  "vouchedAt":  "2026-01-15T10:00:00Z",
  "validUntil": "2027-01-15T10:00:00Z",
  "signature": { ... }
}
```

### Issuer Profile

Each issuer publishes a profile at their well-known URL:

```
GET https://childfocus.be/.well-known/openamber/profile. json
```

```json
{
  "@context": ["https://openamber.eu/ns/v1"],
  "type": "IssuerProfile",
  "id": "https://childfocus.be/.well-known/openamber",
  "name": "Child Focus",
  "country": "BE",
  "type": "hotline",
  "website": "https://childfocus.be",
  "contact": "info@childfocus.be",
  "publicKey": {
    "type": "Ed25519VerificationKey2020",
    "publicKeyMultibase": "z6Mkf5rGMoatrSj1..."
  },
  "vouches": [
    "https://missingchildreneurope.eu/.well-known/openamber/vouches/childfocus.json"
  ]
}
```

## Relay Trust Policies

Relays determine which alerts to accept and redistribute based on configurable trust policies:

### Policy:  Whitelist (Recommended for Production)

```yaml
trust: 
  mode: whitelist
  issuers:
    - https://childfocus.be
    - https://anar.org
    - https://missingchildreneurope.eu
```

- Only accepts alerts from explicitly listed issuers
- Highest security, lowest risk of hoax propagation
- Requires manual issuer onboarding

### Policy: Web of Trust

```yaml
trust:
  mode: web-of-trust
  anchors:
    - https://missingchildreneurope.eu
  maxDepth: 2
```

- Accepts alerts from issuers vouched by trusted anchors
- Automatically trusts new issuers when vouched
- Balance of security and scalability

### Policy: Open (Not Recommended)

```yaml
trust:
  mode: open
```

- Accepts any validly-signed alert
- High risk of hoax propagation
- Only for testing/development

## Consumer Verification

Consumers (websites, apps) should **never trust relays blindly**.  Instead:

1. **Verify signature:** Check Ed25519 signature on every alert
2. **Check issuer:** Fetch issuer profile from origin
3. **Validate vouch:** Optionally verify issuer is vouched by trusted anchor
4. **Check expiry:** Ensure alert has not expired

### Verification Levels

| Level | Checks | Use Case |
|-------|--------|----------|
| **Basic** | Valid signature, not expired | Minimum for display |
| **Standard** | Basic + issuer profile accessible | Recommended |
| **Strict** | Standard + vouch from trusted anchor | High-security contexts |

## Key Management

### Key Generation

Issuers generate Ed25519 key pairs using standard tooling:

```bash
# Using OpenSSL
openssl genpkey -algorithm ED25519 -out private.pem
openssl pkey -in private.pem -pubout -out public.pem

# Using OpenAmber CLI
openamber keys generate --output ./keys/
```

### Key Publication

Public keys published at:
```
https://{domain}/.well-known/openamber/keys. json
```

```json
{
  "keys": [
    {
      "id": "https://childfocus.be/.well-known/openamber/keys/1",
      "type": "Ed25519VerificationKey2020",
      "controller": "https://childfocus.be/. well-known/openamber",
      "publicKeyMultibase": "z6Mkf5rGMoatrSj1.. .",
      "created": "2026-01-01T00:00:00Z",
      "revoked": null
    }
  ]
}
```

### Key Rotation

Organizations should rotate keys periodically:

1. Generate new key pair
2. Add new key to `keys.json` (keep old key)
3. Begin signing new alerts with new key
4. After transition period, mark old key as revoked
5. Old key remains in `keys.json` for historical verification

### Key Revocation

If a key is compromised: 

1. **Immediately** mark key as revoked in `keys.json`:
   ```json
   {
     "id": "https://childfocus.be/.well-known/openamber/keys/1",
     "revoked": "2026-01-15T12:00:00Z",
     "revocationReason": "key_compromise"
   }
   ```

2. Publish revocation notice to relays

3. Generate and publish new key

4. Re-sign any still-active alerts with new key

## Trust Bootstrapping

### For New Deployments

1. **Identify root anchors** for your region (MCE, national hotlines)
2. **Contact root anchors** to establish relationship
3. **Exchange public keys** through secure channel
4. **Configure relay** with anchor trust
5. **Test federation** with sample alerts

### For New Issuers

1. **Contact existing issuer** in your network
2. **Provide credentials** (organization registration, mandate)
3. **Generate key pair** and share public key
4. **Receive vouch** from established issuer
5. **Publish profile** at well-known URL
6. **Test alert creation** with relay network

## Security Considerations

### Sybil Attacks

**Threat:** Attacker creates many fake issuer identities

**Mitigation:** 
- Web-of-trust requires vouching from established organizations
- Root anchors perform real-world verification before vouching
- Whitelist mode prevents automatic trust

### Key Compromise

**Threat:** Attacker obtains issuer's private key

**Mitigation:**
- Rapid revocation protocol
- Relays can blacklist compromised keys
- Consumers check revocation status

### Relay Collusion

**Threat:** Malicious relays propagate hoax alerts

**Mitigation:**
- Consumers verify signatures independently
- Multiple relay sources reduce single-relay dependency
- Trust policies limit which issuers relays accept

## Governance

### Protocol Governance

OpenAmber protocol changes governed by: 
- Open specification process
- Community feedback via GitHub
- Stakeholder consultation (MCE, hotlines, technical community)

### Trust Anchor Governance

Root anchor status determined by:
- Established reputation in child protection
- Geographic coverage and operational capacity
- Commitment to protocol compliance
- Community consensus

### Dispute Resolution

Trust disputes (e.g., revoking issuer status) resolved through:
1. Direct communication between parties
2. Mediation by MCE or neutral anchor
3. Public documentation of resolution

---

*This trust model balances security (preventing hoax alerts) with decentralization (no single controlling authority). Implementation details may evolve based on operational experience.*
