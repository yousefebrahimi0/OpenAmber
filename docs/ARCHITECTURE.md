# OpenAmber Architecture Overview

## System Components

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           ISSUER LAYER                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │
│  │ Child Focus     │  │ Fundación ANAR  │  │ Police Department       │  │
│  │ (Belgium)       │  │ (Spain)         │  │ (Any EU country)        │  │
│  └────────┬────────┘  └────────┬────────┘  └────────────┬────────────┘  │
│           │                    │                        │               │
│           └────────────────────┼────────────────────────┘               │
│                                │ Signed Alerts                          │
└────────────────────────────────┼────────────────────────────────────────┘
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           RELAY LAYER                                   │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │
│  │ MCE Hub         │  │ Media Relay     │  │ Community Relay         │  │
│  │ (Primary)       │  │ (News outlets)  │  │ (NGOs, volunteers)      │  │
│  └────────┬────────┘  └────────┬────────┘  └────────────┬────────────┘  │
│           │                    │                        │               │
│           └────────────────────┼────────────────────────┘               │
│                                │ Validated Alerts                       │
└────────────────────────────────┼────────────────────────────────────────┘
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          CONSUMER LAYER                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │
│  │ News Websites   │  │ Mobile Apps     │  │ Digital Signage         │  │
│  │ (WordPress)     │  │ (iOS/Android)   │  │ (Public displays)       │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │
│  │ Social Media    │  │ Email Services  │  │ Broadcast Systems       │  │
│  │ (Fediverse)     │  │ (Newsletters)   │  │ (TV/Radio)              │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

## Reference Implementation Components

### 1. Issuer Tools

**Alert Creation CLI**
```bash
openamber create \
  --issuer "https://childfocus.be" \
  --name "Emma" \
  --age 8 \
  --description "Brown hair, blue eyes, 125cm" \
  --last-seen "Brussels Central Station" \
  --photo ./emma.jpg \
  --output alert.json
```

**Key Management**
```bash
openamber keys generate --output keys/
openamber keys publish --domain childfocus.be
openamber keys rotate --revoke-old
```

### 2. Relay Server

**Technology Stack**

| Component | Technology | Rationale |
|-----------|------------|-----------|
| Runtime | Node.js 20 LTS | Mature, async-friendly, wide deployment |
| Language | TypeScript | Type safety for protocol implementation |
| Database | PostgreSQL 15 | JSON support, reliable, proven at scale |
| Cache | Redis | Alert caching, pub/sub for real-time |
| Cryptography | libsodium | Audited Ed25519 implementation |

**API Endpoints**

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/alerts` | GET | List active alerts (paginated) |
| `/alerts` | POST | Submit new alert (issuers only) |
| `/alerts/{id}` | GET | Retrieve specific alert |
| `/alerts/{id}/verify` | GET | Verify alert signature |
| `/resolutions` | POST | Submit resolution notice |
| `/feed/rss` | GET | RSS feed of active alerts |
| `/feed/atom` | GET | Atom feed of active alerts |
| `/.well-known/openamber` | GET | Relay metadata and capabilities |

**Configuration**

```yaml
# relay-config.yaml
server:
  port: 3000
  host: 0.0.0.0

database:
  url: postgres://localhost/openamber

trust: 
  mode: whitelist  # whitelist | web-of-trust | open
  issuers:
    - https://childfocus.be
    - https://anar.org
    - https://missingchildreneurope.eu

federation:
  peers:
    - https://relay.example.org
    - https://alerts.newsoutlet.eu
  sync_interval: 60  # seconds

expiry:
  max_days: 30
  cleanup_interval: 3600  # seconds
```

### 3. Web Validator

**Features**
- Paste alert JSON → verify signature
- Enter alert URL → fetch and verify
- Display verification status with details
- Show issuer information and trust chain
- Mobile-responsive design
- No account required

**Technology**
- Vanilla JavaScript (no framework dependencies)
- WebCrypto API for client-side verification
- Progressive enhancement (works without JS for basic display)

### 4. Integration Kit

**WordPress Plugin**
- Shortcode:  `[openamber_alerts relay="https://relay.example.org" limit="5"]`
- Widget for sidebar display
- Auto-refresh via AJAX
- Customizable styling
- WCAG 2.1 AA compliant

**Embed Widget**
```html
<script src="https://openamber.eu/widget.js"></script>
<div data-openamber-widget
     data-relay="https://relay.example.org"
     data-limit="3"
     data-theme="light">
</div>
```

**JavaScript SDK**
```javascript
import { OpenAmberClient } from '@openamber/sdk';

const client = new OpenAmberClient({
  relay: 'https://relay.example.org'
});

// Fetch active alerts
const alerts = await client. getAlerts({ limit: 10 });

// Verify an alert
const result = await client.verify(alertJson);
console.log(result. valid); // true/false

// Subscribe to real-time updates
client.subscribe((alert) => {
  console.log('New alert:', alert. subject. givenName);
});
```

**Python SDK**
```python
from openamber import OpenAmberClient

client = OpenAmberClient(relay="https://relay.example.org")

# Fetch active alerts
alerts = client.get_alerts(limit=10)

# Verify an alert
result = client.verify(alert_json)
print(result.valid)  # True/False

# Create and sign an alert (issuers only)
alert = client.create_alert(
    issuer_key="path/to/private.key",
    name="Emma",
    age=8,
    description="Brown hair, blue eyes",
    last_seen="Brussels Central Station"
)
```

## Deployment Architecture

### Docker Compose (Development/Small Deployment)

```yaml
version: '3.8'
services:
  relay:
    image: openamber/relay: latest
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://postgres:password@db/openamber
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache

  validator:
    image: openamber/validator:latest
    ports: 
      - "8080:80"

  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=openamber
      - POSTGRES_PASSWORD=password

  cache:
    image: redis:7

volumes:
  pgdata: 
```

### NixOS Module (Production)

```nix
{ config, pkgs, ...  }: 

{
  services.openamber = {
    enable = true;
    
    relay = {
      enable = true;
      port = 3000;
      database = "postgres://localhost/openamber";
      trustMode = "whitelist";
      trustedIssuers = [
        "https://childfocus.be"
        "https://missingchildreneurope.eu"
      ];
    };
    
    validator = {
      enable = true;
      port = 8080;
    };
  };

  services.postgresql = {
    enable = true;
    ensureDatabases = [ "openamber" ];
  };

  services.nginx. virtualHosts."alerts.example.org" = {
    enableACME = true;
    forceSSL = true;
    locations."/" = {
      proxyPass = "http://localhost:3000";
    };
  };
}
```

## Data Flow Diagrams

### Alert Creation Flow

```
┌──────────┐    ┌──────────────┐    ┌─────────────┐    ┌──────────┐
│  Issuer  │    │ OpenAmber    │    │   Relay     │    │ Consumers│
│  (NGO)   │    │ CLI/SDK      │    │   Server    │    │          │
└────┬─────┘    └──────┬───────┘    └──────┬──────┘    └────┬─────┘
     │                 │                   │                │
     │ Create alert    │                   │                │
     │────────────────▶│                   │                │
     │                 │                   │                │
     │                 │ Generate hash     │                │
     │                 │ Sign with Ed25519 │                │
     │                 │                   │                │
     │                 │ POST /alerts      │                │
     │                 │──────────────────▶│                │
     │                 │                   │                │
     │                 │                   │ Validate sig   │
     │                 │                   │ Store alert    │
     │                 │                   │                │
     │                 │     201 Created   │                │
     │                 │◀──────────────────│                │
     │                 │                   │                │
     │                 │                   │ Broadcast      │
     │                 │                   │───────────────▶│
     │                 │                   │                │
```

### Alert Verification Flow

```
┌──────────┐    ┌──────────────┐    ┌─────────────┐    ┌──────────┐
│  User    │    │  Validator   │    │   Issuer    │    │  Result  │
│          │    │  Web UI      │    │   Server    │    │          │
└────┬─────┘    └──────┬───────┘    └──────┬──────┘    └────┬─────┘
     │                 │                   │                │
     │ Paste alert     │                   │                │
     │────────────────▶│                   │                │
     │                 │                   │                │
     │                 │ Extract issuer ID │                │
     │                 │                   │                │
     │                 │ GET /.well-known/ │                │
     │                 │ openamber/keys    │                │
     │                 │──────────────────▶│                │
     │                 │                   │                │
     │                 │    Public key     │                │
     │                 │◀──────────────────│                │
     │                 │                   │                │
     │                 │ Verify Ed25519    │                │
     │                 │ signature         │                │
     │                 │                   │                │
     │                 │ Check expiry      │                │
     │                 │                   │                │
     │  ✅ Valid /     │                   │                │
     │  ❌ Invalid     │                   │                │
     │◀────────────────│                   │                │
     │                 │                   │                │
```

## Security Architecture

### Threat Model

| Threat | Mitigation |
|--------|------------|
| Unauthorized alert creation | Ed25519 signatures; only key holders can sign |
| Alert tampering in transit | Signature covers entire alert; any change invalidates |
| Hoax alerts | Issuer verification via web-of-trust; relay whitelisting |
| Key compromise | Key revocation protocol; key rotation support |
| Replay attacks | Unique IDs; timestamp validation; expiry enforcement |
| Privacy leakage | Data minimization; photo hashing; automatic expiry |
| Relay compromise | Consumers verify signatures independently; don't trust relays |

### Defense in Depth

```
┌─────────────────────────────────────────────────────────────┐
│ Layer 1: Cryptographic Verification                         │
│ - Ed25519 signatures on all alerts                          │
│ - Consumer-side verification (don't trust relays)           │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│ Layer 2: Trust Policies                                     │
│ - Issuer whitelisting at relay level                        │
│ - Web-of-trust for new issuer onboarding                    │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│ Layer 3: Operational Security                               │
│ - Rate limiting                                             │
│ - Anomaly detection                                         │
│ - Audit logging                                             │
└─────────────────────────────────────────────────────────────┘
```

## Monitoring & Observability

### Metrics (Prometheus)

```
# Alert metrics
openamber_alerts_total{issuer, status}
openamber_alerts_active_count
openamber_alert_verification_duration_seconds

# Relay metrics
openamber_relay_sync_duration_seconds
openamber_relay_peers_connected
openamber_relay_requests_total{endpoint, status}

# Consumer metrics
openamber_widget_impressions_total
openamber_widget_clicks_total
```

### Health Checks

```
GET /health
{
  "status": "healthy",
  "checks": {
    "database": "ok",
    "cache": "ok",
    "peers": {
      "connected": 3,
      "total": 4
    }
  },
  "alerts": {
    "active": 12,
    "expired_pending_cleanup": 2
  }
}
```
