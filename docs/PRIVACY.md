# OpenAmber Privacy & GDPR Compliance

## Overview

OpenAmber is designed with **privacy-by-design** principles to ensure GDPR compliance while enabling effective missing child alerts. This document outlines the privacy safeguards built into the protocol.

## Legal Basis for Processing

Under GDPR Article 6, OpenAmber alert processing relies on:

- **Article 6(1)(d):** Processing necessary to protect vital interests of the data subject (the missing child)
- **Article 6(1)(e):** Processing necessary for performance of a task carried out in the public interest

## Data Minimization Principles

### Required Data (Minimum Necessary)

| Field | Justification |
|-------|---------------|
| First name | Identification by public |
| Age | Identification and urgency assessment |
| Physical description | Visual identification |
| Last known location (general) | Geographic targeting of alert |
| Photo (hashed reference) | Visual identification |

### Explicitly Excluded Data

| Field | Reason for Exclusion |
|-------|---------------------|
| Home address | Not necessary; privacy risk |
| School/daycare name | Not necessary; safety risk |
| Family member names | Not necessary; privacy risk |
| Medical conditions | Only if critical for identification |
| National ID numbers | Never necessary for public alerts |
| Immigration status | Never relevant; discrimination risk |

## Photo Handling

Photos require special protection as biometric data:

### Hash-Based Reference

```json
{
  "photoHash": "sha256:a1b2c3d4.. .",
  "photoUrl": "https://issuer.example/photos/alert-123.jpg"
}
```

- Alert contains only **hash** of photo (not the image itself)
- Actual photo retrieved from **issuer's server** via secure URL
- Issuer controls photo access and can revoke at any time
- Relays **MUST NOT** cache photos beyond immediate display needs

### Photo Lifecycle

1. **Active alert:** Photo available via `photoUrl`
2. **Alert resolved:** Issuer removes photo from server
3. **Subsequent requests:** Return 404/410; consumers remove cached copies

## Automatic Data Expiry

### Alert Expiration

- All alerts **MUST** include `expires` timestamp
- Maximum expiry: **30 days** from creation
- Relays **MUST** delete expired alerts within 24 hours
- Consumers **MUST** stop displaying expired alerts immediately

### Resolution-Triggered Deletion

When child is found:
1.  Issuer publishes `AlertResolution` notice
2. Relays remove alert within **24 hours**
3. Consumers remove alert **immediately** upon receiving resolution

### Revocation

Issuers can revoke alerts (e.g., false alarm):
1. Issuer publishes `AlertRevocation` notice
2. All nodes treat alert as invalid
3. Same deletion timeline as resolution

## Right to Erasure (Article 17)

### Who Can Request Erasure? 

- Parent/guardian of the child
- The child (if sufficient age/maturity)
- Issuing organization (on behalf of family)

### Erasure Process

1. Request submitted to original issuer
2. Issuer publishes revocation notice
3. Revocation propagates through relay network
4. All nodes delete alert data within 24 hours

### Limitations

Erasure may be refused if:
- Alert is still operationally necessary (child still missing)
- Legal retention requirements apply
- Public interest override (rare)

## Data Retention Schedule

| Data Type | Retention Period | Justification |
|-----------|------------------|---------------|
| Active alerts | Until resolved/expired | Operational necessity |
| Resolved alerts | 24 hours post-resolution | Propagation buffer |
| Resolution notices | 30 days | Audit trail |
| Relay logs (anonymized) | 90 days | Security monitoring |
| Issuer public keys | Until revoked + 1 year | Signature verification |

## Cross-Border Transfers

### Within EU/EEA

- No restrictions; GDPR applies uniformly
- Relays can operate in any EU member state

### Outside EU/EEA

Alerts may only be relayed to non-EU countries if:
- Adequate protection exists (adequacy decision)
- Standard Contractual Clauses in place
- Explicit consent from issuer

### Practical Guidance

- Default configuration: EU-only relay network
- Non-EU relays: Require explicit opt-in by issuer
- Transparency:  Issuers informed which relays receive their alerts

## Children's Data (Article 8)

OpenAmber processes data **about** children (subjects of alerts), not data **from** children.  Key considerations:

- **Parental consent:** Implicit in missing child report to authorities
- **Best interests:** Alert publication serves child's vital interests
- **Proportionality:** Only minimum necessary data collected

## Privacy Impact Assessment Summary

| Risk | Likelihood | Impact | Mitigation | Residual Risk |
|------|------------|--------|------------|---------------|
| Unnecessary data collection | Low | Medium | Schema enforces minimization | Low |
| Photo misuse | Medium | High | Hash-only in alerts; issuer-controlled access | Low |
| Data persistence after resolution | Medium | Medium | Automatic expiry; deletion propagation | Low |
| Cross-border transfer issues | Low | Medium | EU-default; explicit non-EU opt-in | Low |
| Re-identification of child | Low | High | Limited data fields; expiry | Low |

## Compliance Checklist for Implementers

### Issuers

- [ ] Only collect minimum necessary data
- [ ] Obtain appropriate authorization before issuing alert
- [ ] Host photos on secure, access-controlled server
- [ ] Publish resolution notices when child is found
- [ ] Respond to erasure requests within 72 hours
- [ ] Maintain records of processing activities

### Relays

- [ ] Validate all alerts before redistribution
- [ ] Delete expired alerts within 24 hours
- [ ] Delete resolved/revoked alerts within 24 hours
- [ ] Do not cache photos beyond display needs
- [ ] Log access for security purposes only
- [ ] Implement appropriate technical security measures

### Consumers

- [ ] Verify alert signatures before display
- [ ] Do not display expired alerts
- [ ] Remove resolved/revoked alerts promptly
- [ ] Do not store alert data beyond display session
- [ ] Link to issuer for additional information

## Contact

For privacy-related questions about the OpenAmber protocol: 
- **Email:** yousef. ebrahimi. digital@gmail.com
- **GitHub Issues:** github.com/yousefebrahimi0/openamber

---

*This document provides protocol-level privacy guidance. Individual implementations must conduct their own Data Protection Impact Assessments (DPIAs) based on their specific deployment context.*
