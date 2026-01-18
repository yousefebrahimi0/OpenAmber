# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in OpenAmber, please report it responsibly. 

### How to Report

**Email:** info [@] yousef [.] uk

**Subject line:** `[SECURITY] Brief description`

### What to Include

- Description of the vulnerability
- Steps to reproduce
- Potential impact assessment
- Suggested fix (if you have one)

### Response Timeline

- **Acknowledgment:** Within 48 hours
- **Initial assessment:** Within 7 days
- **Resolution timeline:** Provided after assessment

### What to Expect

1. We will acknowledge your report promptly
2. We will investigate and assess the vulnerability
3. We will work with you to understand the issue fully
4. We will develop and test a fix
5. We will release the fix and credit you (unless you prefer anonymity)

## Security Considerations

### Cryptographic Implementation

OpenAmber uses Ed25519 for digital signatures.  Key considerations:

- Use audited cryptographic libraries (libsodium recommended)
- Never implement cryptographic primitives from scratch
- Follow key management best practices

### Data Protection

- Alerts contain personal data about children
- Implement data minimization principles
- Enforce automatic expiry of alerts
- Protect photo URLs from unauthorized access

### Relay Security

- Validate all incoming alert signatures
- Implement rate limiting
- Log access for security monitoring
- Use TLS for all communications

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 0.x. x   | : white_check_mark: (development) |

## Known Limitations

As a pre-development project, OpenAmber has not yet undergone formal security audit. The protocol specification is subject to change based on security review.

Once funded, we plan to engage Radically Open Security (through NGI Zero support services) for a comprehensive security audit. 
