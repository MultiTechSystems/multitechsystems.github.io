# API Documentation Progress

Current status of mPower Device API schema documentation.

## Progress Summary

**Last Updated**: February 11, 2026

### Completion Status

| Category | Schemas | Documented | Percentage |
|----------|---------|------------|------------|
| **High Priority** | 3 | 3 | 100% |
| **Medium Priority** | 17 | 17 | 100% |
| **Low Priority** | 37 | 37 | 100% |
| **TOTAL** | 57 | 57 | **100%** |
| Internal (skipped) | 1 | — | N/A |

> **Note**: `legacyDefaults` is the only schema not documented (internal use only, no external API surface).

## Schema Documentation Map

All 57 public schemas are fully documented across the following files:

### Core Resources — [API-SCHEMAS-REFERENCE.md](API-SCHEMAS-REFERENCE.md)

| Resource | Status |
|----------|--------|
| system | ✅ Complete |
| users | ✅ Complete |
| cellular | ✅ Complete |
| wifi | ✅ Complete |
| dns | ✅ Complete |
| dhcp | ✅ Complete |
| routes | ✅ Complete |
| firewall | ✅ Complete |
| nat | ✅ Complete |
| gps | ✅ Complete |
| serial | ✅ Complete |
| alert | ✅ Complete |

### LoRa Network — [LORA-NETWORK-SCHEMA.md](LORA-NETWORK-SCHEMA.md)

| Resource | Status |
|----------|--------|
| loraNetwork | ✅ Complete (200+ properties, 4 modes) |

### Network Interfaces — [NETWORK-INTERFACES-SCHEMA.md](NETWORK-INTERFACES-SCHEMA.md)

| Resource | Status |
|----------|--------|
| ni | ✅ Complete (IPv4/IPv6, VLAN, 802.1X) |

### Network Services — [NETWORK-SERVICES-SCHEMAS.md](NETWORK-SERVICES-SCHEMAS.md)

| Resource | Status |
|----------|--------|
| snmp | ✅ Complete |
| sntp | ✅ Complete |
| smtp | ✅ Complete |

### Monitoring — [SNMP-SCHEMA.md](SNMP-SCHEMA.md) / [ALERT-SCHEMA.md](ALERT-SCHEMA.md)

| Resource | Status |
|----------|--------|
| snmp | ✅ Complete (v1/v2c/v3, traps) |
| alert | ✅ Complete (12 event types) |

### SMS — [SMS-SCHEMA.md](SMS-SCHEMA.md)

| Resource | Status |
|----------|--------|
| sms | ✅ Complete (commands, inbox/outbox) |

### VPN — [VPN-SCHEMAS.md](VPN-SCHEMAS.md)

| Resource | Status |
|----------|--------|
| ipsecTunnels | ✅ Complete (IKEv1/IKEv2) |
| greTunnels | ✅ Complete |
| ovpnTunnels | ✅ Complete (server/client/custom) |

### Security — [NAT-FIREWALL-SCHEMAS.md](NAT-FIREWALL-SCHEMAS.md) / [SECURITY-SCHEMAS.md](SECURITY-SCHEMAS.md)

| Resource | Status |
|----------|--------|
| nat | ✅ Complete (DNAT, SNAT, MASQUERADE) |
| firewall | ✅ Complete |
| filters | ✅ Complete (INPUT, OUTPUT, FORWARD) |
| routes | ✅ Complete |
| secureProtocols | ✅ Complete (TLS/SSH ciphers) |
| remoteAccess | ✅ Complete (HTTP/HTTPS/SSH) |
| trustedIp | ✅ Complete (whitelist/blacklist) |
| radius | ✅ Complete |

### Device Management — [DEVICE-MANAGEMENT-SCHEMAS.md](DEVICE-MANAGEMENT-SCHEMAS.md)

| Resource | Status |
|----------|--------|
| autoReboot | ✅ Complete |
| remoteMgmt | ✅ Complete |
| remoteAccess | ✅ Complete |
| resetButton | ✅ Complete |
| saveAndRestore | ✅ Complete |
| customApps | ✅ Complete |
| customAppsConfig | ✅ Complete |
| customRoles | ✅ Complete |
| docker | ✅ Complete |
| passwordComplexityRules | ✅ Complete |
| brand | ✅ Complete (OEM/white-label) |

### Connectivity — [CONNECTIVITY-SCHEMAS.md](CONNECTIVITY-SCHEMAS.md)

| Resource | Status |
|----------|--------|
| bluetooth | ✅ Complete |
| bluetoothLowEnergy | ✅ Complete |
| btDevices | ✅ Complete |
| ipPassthrough | ✅ Complete |
| ipPipes | ✅ Complete |
| ddns | ✅ Complete |
| wanmngr | ✅ Complete |
| waninfo | ✅ Complete |
| backOffTimers | ✅ Complete |
| cellTimeSync | ✅ Complete |

### Advanced Features — [ADVANCED-FEATURES-SCHEMAS.md](ADVANCED-FEATURES-SCHEMAS.md)

| Resource | Status |
|----------|--------|
| filters | ✅ Complete |
| scada | ✅ Complete (BACnet, Modbus) |
| mqttBroker | ✅ Complete |
| radius | ✅ Complete |
| secureProtocols | ✅ Complete |
| trustedIp | ✅ Complete |
| notificationEventGroup | ✅ Complete |

### Logging & Status — [LOGGING-STATUS-SCHEMAS.md](LOGGING-STATUS-SCHEMAS.md)

| Resource | Status |
|----------|--------|
| eventlog | ✅ Complete |
| syslog | ✅ Complete |
| status | ✅ Complete |
| lldp | ✅ Complete |
| mdns | ✅ Complete |
| policy | ✅ Complete |
| extIo | ✅ Complete |
| extUsb | ✅ Complete |
| databaseInfo | ✅ Complete |

## OpenAPI / Swagger Specification

**Status**: ✅ Complete

The OpenAPI 3.0 specification has been restructured into modular files:

```
mpower-api-docs/openapi/
├── api-documentation.yaml      # Main spec with $ref imports
├── schemas/                    # 14 schema files by domain
└── paths/                      # 14 path files by domain
```

- Bundled YAML and JSON outputs validated with `swagger-cli`
- Covers 100+ endpoints across 58+ resources
- 14 domain-specific schema files, 14 path files

## Site Infrastructure (February 2026)

### Completed

- **Site redesign** — Modern layout with fixed header, sidebar navigation, breadcrumbs, page TOC
- **Search functionality** — Custom inline full-text search with relevance ranking
- **Version selector** — v1.0 / v2.0 dropdown in sidebar
- **Mobile responsive** — Hamburger nav, collapsible sidebar, responsive search
- **Navigation structure** — `_data/navigation.yml` with expandable sections
- **OpenAPI/Swagger** — Modular YAML files in `mpower-api-docs/openapi/`
- **Development guides** — Conduit custom app, Cursor AI dev, Dot development

### Search fix (February 11, 2026)

- Replaced `SimpleJekyllSearch` library with custom inline JavaScript
- Added relevance scoring (title match > content match)
- Direct `style.display` control bypasses CSS specificity issues
- Console logging confirms search index load

### Pages missing YAML frontmatter

| Page | Status |
|------|--------|
| `basic-station.md` | Empty file (0 lines) — stub |
| `dot-development-xdot.md` | Content exists, needs frontmatter |
| `dot-migrate-to-4.md` | Content exists, needs frontmatter |
| `lorawan-app-connect-mqtt-azure.md` | Content exists, needs frontmatter |
| `lorawan-app-connect-mqtt-websockets.md` | Content exists, needs frontmatter |
| `lorawan-app-connect-walkthrough.md` | Content exists, needs frontmatter |
| `xdot-daplink.md` | Content exists, needs frontmatter |

## Recommended Next Steps

### Phase 1: Site Quality (Quick wins)

1. Add YAML frontmatter to the 6 pages listed above
2. Write `basic-station.md` content (currently empty)
3. Verify all search results link correctly

### Phase 2: Polish & Maintenance

1. Review schema docs for accuracy against latest firmware
2. Add more real-world configuration examples
3. Expand troubleshooting sections based on support tickets
4. Cross-reference related schemas where missing

## Documentation Quality Standards

Each schema documentation includes:

### Required Sections

- ✅ Overview and description
- ✅ Complete property table with types and constraints
- ✅ Nested object documentation
- ✅ Configuration examples (minimum 2-3)
- ✅ API operations (GET, PUT, POST, DELETE)
- ✅ Important notes and constraints

### Optional Sections (for complex schemas)

- Configuration modes and their differences
- Common use cases
- Troubleshooting guide
- Related resources
- Migration notes

## Benefits

### Achieved (100% schema completion)

- **Complete self-service API reference** for all 57 public schemas
- **loraNetwork**: Full LoRa configuration reference (4 modes, 200+ properties)
- **VPN/NAT/Firewall/Security**: Complete networking and security reference
- **SNMP/Alerting/SMS**: Complete monitoring and notification reference
- **Device Management**: Auto-reboot, remote access, Docker, custom apps
- **Connectivity**: Bluetooth, WAN failover, IP passthrough, DDNS
- **Site redesign**: Modern, searchable, mobile-friendly documentation
- Reduced support questions for all API resources
- Faster integration for developers

## Metrics

### API Schema Documentation Coverage

```
Progress: [████████████████████████████████] 100% (57/57 schemas)
```

### Site Page Coverage

```
Pages with frontmatter: 11/18 (61%)
Pages with content:     17/18 (94%)  — basic-station.md is empty
```

### Effort Invested

- **Schema documentation**: ~70 hours
- **Site redesign & search**: ~20 hours
- **Total project**: ~90 hours

### Timeline

- **Started**: December 17, 2025
- **Phase 1 (High Priority)**: December 2025
- **Phase 2 (Medium Priority)**: December 2025 — February 2026
- **Phase 3 (Low Priority + remaining)**: February 10, 2026
- **Site Redesign**: February 10, 2026
- **Search Fix**: February 11, 2026
- **Schema Documentation Complete**: February 10, 2026

## Supporting Documents

| Document | Purpose |
|----------|---------|
| [API-DOCUMENTATION-INDEX.md](API-DOCUMENTATION-INDEX.md) | Central navigation hub |
| [API-README.md](API-README.md) | Main API documentation |
| [API-QUICK-REFERENCE.md](API-QUICK-REFERENCE.md) | Quick command reference |
| [SCHEMA-ANALYSIS-WORKING.md](SCHEMA-ANALYSIS-WORKING.md) | Schema-by-schema tracking |
| [SCHEMA-COVERAGE-ANALYSIS.md](SCHEMA-COVERAGE-ANALYSIS.md) | Initial coverage analysis |
| [PROJECT-COMPLETION-SUMMARY.md](PROJECT-COMPLETION-SUMMARY.md) | Phase 1-2 project summary |
| [FINAL-SUMMARY.md](FINAL-SUMMARY.md) | Overall project summary |

## Related Documents

- [Schema Coverage Analysis](SCHEMA-COVERAGE-ANALYSIS.md) - Initial analysis
- [API README](API-README.md) - Main API documentation
- [Quick Reference](API-QUICK-REFERENCE.md) - Quick command reference
- [OpenAPI Specification](api-documentation.yaml) - OpenAPI 3.0 spec

---

**Document Version**: 3.0
**Last Updated**: February 11, 2026
**Next Review**: As needed (schema documentation phase complete)
