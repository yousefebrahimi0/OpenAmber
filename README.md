# OpenAmber
**Interoperable Protocol for Verified Missing Child Alerts**

OpenAmber is an open-source protocol and reference implementation enabling cryptographically-signed missing child alerts across European organizations. It defines a privacy-preserving data schema, verification standard, and federation specification allowing NGOs, hotlines, and media outlets to publish and relay authenticated alerts without centralized platform dependency.

## Status

🚧 **Pre-development** — This project is seeking funding through the NGI Zero Commons Fund.

## The Problem

Every year, over 250,000 children are reported missing across Europe. The first hours are critical—yet alert infrastructure remains: 

- **Fragmented:** Each country operates independent systems that cannot interoperate
- **Platform-dependent:** Alerts rely on social media platforms subject to algorithms, outages, and censorship
- **Unverifiable:** No cryptographic standard exists to authenticate alerts, enabling hoaxes that undermine public trust

## The Solution

OpenAmber provides the **missing interoperability layer** for European child protection: 

- **Open Protocol:** Standardized JSON-LD schema for machine-readable alerts
- **Cryptographic Verification:** Ed25519 signatures prove alert authenticity without central authority
- **Federated Distribution:** Any organization can relay verified alerts; no single point of failure
- **Privacy-by-Design:** GDPR-compliant data minimization with automatic expiry

## How It Works

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Issuing Org    │     │  Relay Node     │     │  Consumer       │
│  (Police/NGO)   │────▶│  (Media/NGO)    │───▶│  (Website/App)  │
│                 │     │                 │     │                 │
│  Signs Alert    │     │  Validates &    │     │  Verifies &     │
│  with Ed25519   │     │  Redistributes  │     │  Displays       │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

1. **Issuer** (police, 116 000 hotline) creates alert and signs with Ed25519 key
2. **Relay nodes** (media outlets, NGOs) validate signature and redistribute
3. **Consumers** (websites, apps, citizens) verify authenticity before displaying/sharing

## Key Features

| Feature | Description |
|---------|-------------|
| **Interoperability** | Belgian Child Focus alerts automatically readable by Spanish ANAR systems |
| **Verification** | Anyone can cryptographically verify an alert's authenticity |
| **Resilience** | Federated relays ensure no single point of failure |
| **Privacy** | Data minimization, automatic expiry, photo hash protection |
| **Open Standard** | No vendor lock-in; anyone can implement |

## Planned Deliverables

- [ ] Protocol specification (JSON-LD schema, signing methodology)
- [ ] JavaScript SDK for signing and verification
- [ ] Python SDK for signing and verification
- [ ] Reference relay server (Node.js)
- [ ] Web validator tool
- [ ] WordPress plugin for displaying verified alerts
- [ ] Docker + NixOS deployment configurations
- [ ] Integration documentation for NGOs and media

## Documentation

- [Protocol Specification (Draft)](docs/SPECIFICATION.md)
- [Architecture Overview](docs/ARCHITECTURE.md)
- [Privacy & GDPR Compliance](docs/PRIVACY.md)
- [Trust Model](docs/TRUST-MODEL.md)

## Schema Examples

- [Alert Schema](schemas/alert.jsonld)
- [Example Alert](examples/example-alert.json)
- [Example Issuer Profile](examples/example-issuer. json)

## Target Stakeholders

| Organization Type | How They Use OpenAmber |
|-------------------|------------------------|
| **116 000 Hotlines** | Issue and relay verified alerts across borders |
| **Missing Children Europe** | Coordinate network-wide alert distribution |
| **Police/Authorities** | Publish alerts to federated network |
| **Media Outlets** | Display verified alerts with confidence |
| **Community Websites** | Embed verified alert widgets |
| **Citizens** | Verify alert authenticity before sharing |

## Why Not Use Existing Systems?

| System | Limitation |
|--------|------------|
| **AMBER Alert Europe** | Voluntary coordination without technical interoperability standard |
| **Social Media** | Platform-dependent, algorithmic throttling, no verification |
| **Email Lists** | No machine-readable format, no cryptographic verification |
| **National Systems** | Siloed by jurisdiction, incompatible formats |

OpenAmber provides the **protocol layer** these systems lack—enabling them to interoperate while preserving their independence.

## Alignment with EU Policy

- **EU Strategy on the Rights of the Child:** Cross-border protection coordination
- **Digital Services Act:** Alternative to platform-dependent distribution
- **GDPR:** Privacy-by-design with data minimization
- **European Digital Sovereignty:** Open standard, no US platform dependency

## License

Code:  [EUPL-1.2](LICENSE)
Documentation: CC-BY-SA 4.0

## Contributing

This project is in early specification phase. Contributions welcome: 

- **Protocol feedback:** Open an issue to discuss schema design
- **Use case documentation:** Share how your organization could use OpenAmber
- **Translation:** Help translate documentation for EU-wide adoption

## Contact

**Yousef Ebrahimi**
- Email: info [@] yousef [.] uk
- GitHub: [@yousefebrahimi0](https://github.com/yousefebrahimi0)
- LinkedIn: [yousefebrahimi0](https://linkedin.com/in/yousefebrahimi0)

## Acknowledgments

This project is proposed for funding under the NGI Zero Commons Fund, established by the European Commission through the NLnet Foundation.

## Related Organizations

- [Missing Children Europe](https://missingchildreneurope.eu/) — Network of 32 member organizations
- [116 000 Hotlines](https://missingchildreneurope.eu/116-000/) — EU-wide missing children hotline
- [AMBER Alert Europe](https://www.amberalert.eu/) — Cross-border alert coordination

---

*When a child goes missing, every minute counts. OpenAmber ensures alerts reach every possible channel—verified, interoperable, and resilient.*
