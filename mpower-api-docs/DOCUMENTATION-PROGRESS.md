# API Documentation Progress

Current status of mPower Device API schema documentation.

## Progress Summary

**Last Updated**: February 11, 2026

### Completion Status

| Category | Schemas | Documented | Percentage |
|----------|---------|------------|------------|
| **High Priority** | 3 | 3 | 100% |
| **Medium Priority** | 17 | 7 | 41% |
| **Low Priority** | 33 | 0 | 0% |
| **Partial (OpenAPI)** | 7 | 7 | 100% |
| **TOTAL** | 57 | 17 | 30% |

## Completed Documentation

### ✅ Fully Documented (Detailed Schema Docs)

1. **`loraNetwork`** - [LORA-NETWORK-SCHEMA.md](LORA-NETWORK-SCHEMA.md)
   - 200+ properties documented
   - 16 major configuration sections
   - 4 operational modes
   - Complete examples

2. **`ni`** (Network Interfaces) - [NETWORK-INTERFACES-SCHEMA.md](NETWORK-INTERFACES-SCHEMA.md)
   - 34 properties documented
   - IPv4/IPv6 configuration
   - VLAN support
   - 802.1X authentication
   - 7 configuration examples

3. **`snmp`** (SNMP Agent) - [SNMP-SCHEMA.md](SNMP-SCHEMA.md)
   - SNMPv1/v2c/v3 configuration
   - Trap receivers
   - Authentication and encryption
   - 5 configuration examples

4. **`alert`** (Alerting System) - [ALERT-SCHEMA.md](ALERT-SCHEMA.md)
   - 12 alert event types
   - Email, SMS, SNMP notifications
   - Recipient groups
   - 7 configuration examples

5. **`ipsecTunnels`, `greTunnels`, `ovpnTunnels`** - [VPN-SCHEMAS.md](VPN-SCHEMAS.md)
   - IPsec (IKEv1/IKEv2) configuration
   - GRE tunnel configuration
   - OpenVPN (server/client/custom)
   - Complete examples for each type

6. **`nat`, `firewall`, `filters`, `routes`** - [NAT-FIREWALL-SCHEMAS.md](NAT-FIREWALL-SCHEMAS.md)
   - NAT rules (DNAT, SNAT, MASQUERADE)
   - Firewall settings
   - Filter rules (INPUT, OUTPUT, FORWARD)
   - Static routes
   - Port forwarding examples

### 🟡 Partially Documented (OpenAPI Only)

3. **`system`** - Basic properties in OpenAPI spec
4. **`cellular`** - Basic properties in OpenAPI spec
5. **`users`** - Basic properties in OpenAPI spec
6. **`wifi`** - Basic properties in OpenAPI spec
7. **`dns`** - Basic properties in OpenAPI spec
8. **`gps`** - Basic properties in OpenAPI spec
9. **`docker`** - Basic properties in OpenAPI spec

## In Progress

### 🔄 Currently Being Documented

None - awaiting prioritization

## Remaining High Priority

### ✅ All High Priority Schemas Completed!

## Remaining Medium Priority

### Security & VPN (14 schemas)

| Schema | Lines | Properties | Status | Estimated Effort |
|--------|-------|------------|--------|------------------|
| `nat` | 252 | 16 | ✅ Done | - |
| `ipsecTunnels` | 280 | 8 | ✅ Done | - |
| `greTunnels` | 180 | 8 | ✅ Done | - |
| `ovpnTunnels` | 320 | 6 | ✅ Done | - |
| `firewall` | 125 | 3 | ✅ Done | - |
| `filters` | 142 | 11 | ✅ Done | - |
| `routes` | 95 | 4 | ✅ Done | - |
| `secureProtocols` | 180 | 15 | ⏳ Pending | 5-7 hours |
| `remoteAccess` | 165 | 12 | ⏳ Pending | 5-7 hours |
| `radius` | 135 | 8 | ⏳ Pending | 4-6 hours |
| `trustedIp` | 65 | 10 | ⏳ Pending | 3-4 hours |
| `ddns` | 195 | 18 | ⏳ Pending | 5-7 hours |
| `mqttBroker` | 145 | 13 | ⏳ Pending | 4-6 hours |
| `scada` | 161 | 9 | ⏳ Pending | 4-6 hours |
| `remoteMgmt` | 125 | 9 | ⏳ Pending | 4-6 hours |
| `sms` | 110 | 10 | ⏳ Pending | 3-5 hours |
| `smtp` | 95 | 7 | ⏳ Pending | 3-5 hours |

**Medium Priority Completed**: 7/17 (41%)
**Remaining Medium Priority Effort**: 40-59 hours

## Remaining Low Priority

### Simple Resources (33 schemas)

These schemas have < 10 properties and can be documented more quickly or expanded inline in the OpenAPI specification.

| Schema | Lines | Properties | Priority |
|--------|-------|------------|----------|
| `firewall` | 125 | 8 | 🟢 |
| `routes` | 95 | 6 | 🟢 |
| `dhcp` | 110 | 7 | 🟢 |
| `sntp` | 85 | 10 | 🟢 |
| `trustedIp` | 65 | 10 | 🟢 |
| `ipPipes` | 145 | 8 | 🟢 |
| `notificationEventGroup` | 75 | 5 | 🟢 |
| `autoReboot` | 68 | 5 | 🟢 |
| `bluetooth` | 55 | 4 | 🟢 |
| `bluetoothLowEnergy` | 48 | 4 | 🟢 |
| `brand` | 42 | 3 | 🟢 |
| `ipPassthrough` | 45 | 2 | 🟢 |
| `extIo` | 58 | 3 | 🟢 |
| `extUsb` | 35 | 2 | 🟢 |
| `cellTimeSync` | 32 | 2 | 🟢 |
| `eventlog` | 48 | 2 | 🟢 |
| `customApps` | 55 | 3 | 🟢 |
| `customAppsConfig` | 42 | 2 | 🟢 |
| `customRoles` | 68 | 2 | 🟢 |
| `backOffTimers` | 45 | 2 | 🟢 |
| `btDevices` | 38 | 2 | 🟢 |
| `lldp` | 35 | 2 | 🟢 |
| `mdns` | 28 | 2 | 🟢 |
| `policy` | 25 | 1 | 🟢 |
| `resetButton` | 32 | 1 | 🟢 |
| `saveAndRestore` | 28 | 1 | 🟢 |
| `serial` | 85 | 2 | 🟢 |
| `syslog` | 45 | 2 | 🟢 |
| `status` | 49 | 1 | 🟢 |
| `waninfo` | 55 | 2 | 🟢 |
| `wanmngr` | 48 | 4 | 🟢 |
| `databaseInfo` | 35 | 2 | 🟢 |
| `passwordComplexityRules` | 52 | 2 | 🟢 |

**Total Low Priority Effort**: 40-60 hours

## Total Remaining Effort

| Category | Effort Estimate |
|----------|-----------------|
| High Priority (1 remaining) | 10-14 hours |
| Medium Priority (14 schemas) | 60-90 hours |
| Low Priority (33 schemas) | 40-60 hours |
| **TOTAL REMAINING** | **110-164 hours** |

## Site Infrastructure (February 2026)

### Completed

- **Site redesign** — Modern layout with fixed header, sidebar navigation, breadcrumbs, page TOC
- **Search functionality** — Client-side full-text search across all pages (SimpleJekyllSearch)
- **Version selector** — v1.0 / v2.0 dropdown in sidebar
- **Mobile responsive** — Hamburger nav, collapsible sidebar, responsive search
- **Navigation structure** — `_data/navigation.yml` with expandable sections
- **OpenAPI/Swagger** — Modular YAML files in `mpower-api-docs/openapi/`
- **Development guides** — Conduit custom app, Cursor AI dev, Dot development

### Search fix (February 11, 2026)

- Pinned CDN to `simple-jekyll-search@1.10.0` (was `@latest` on unpkg)
- Removed unused jQuery dependency
- Replaced `input` event visibility toggle with `MutationObserver`
- Added diagnostic console logging for search index load

### Pages missing YAML frontmatter

These pages render but lack `title` and `category` metadata, which affects search results and SEO:

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
3. Verify search works after CDN fix deploys

### Phase 2: Remaining Medium Priority Schemas

| Schema | Estimated Effort |
|--------|------------------|
| `secureProtocols` | 5-7 hours |
| `remoteAccess` | 5-7 hours |
| `radius` | 4-6 hours |
| `trustedIp` | 3-4 hours |
| `ddns` | 5-7 hours |
| `mqttBroker` | 4-6 hours |
| `scada` | 4-6 hours |
| `remoteMgmt` | 4-6 hours |
| `sms` | 3-5 hours |
| `smtp` | 3-5 hours |

**Medium Priority Completed**: 7/17 (41%)
**Remaining Medium Priority Effort**: 40-59 hours

### Phase 3: Complete Coverage

1. Expand OpenAPI spec with inline documentation for all low-priority schemas
2. Add configuration examples
3. Create troubleshooting guides
4. Add cross-references between related schemas

**Estimated completion**: 40-60 hours

## Documentation Quality Standards

Each schema documentation should include:

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

## Benefits Tracking

### Current Benefits (30% completion)

- **loraNetwork**: Complete LoRa configuration reference
- **ni**: Complete network interface configuration reference
- **VPN/NAT/Firewall**: Complete security and networking reference
- **SNMP/Alerting**: Complete monitoring configuration reference
- **Site redesign**: Modern, searchable, mobile-friendly documentation
- Reduced support questions for critical resources
- Faster integration for developers

### Expected Benefits (100% completion)

- Self-service documentation for all API resources
- Reduced support burden by ~60%
- Faster integration time by ~40%
- Fewer configuration errors
- Better API adoption
- Complete knowledge preservation

## Metrics

### API Schema Documentation Coverage

```
Progress: [██████████░░░░░░░░░░░░░░░░░░░░░░] 30% (17/57 schemas)
```

### Site Page Coverage

```
Pages with frontmatter: 11/18 (61%)
Pages with content:     17/18 (94%)  — basic-station.md is empty
```

### Effort Invested

- **Completed**: ~50 hours (schemas) + ~20 hours (site redesign)
- **Remaining**: ~80-125 hours (schemas)
- **Total Project**: ~150-195 hours

### Timeline

- **Started**: December 17, 2025
- **Site Redesign**: February 10, 2026
- **Search Fix**: February 11, 2026
- **Current Phase**: Site quality + Phase 2 (Medium Priority schemas)
- **Estimated Completion**: April 2026 (at 10-15 hours/week)

## How to Contribute

### For New Schema Documentation

1. Choose a schema from the "Remaining" lists above
2. Read the JSON schema file in `mtsDeviceAPI/json/permissive/`
3. Follow the template from existing schema docs
4. Include all required sections
5. Test examples with actual API
6. Submit for review

### Documentation Template

See [LORA-NETWORK-SCHEMA.md](LORA-NETWORK-SCHEMA.md) or [NETWORK-INTERFACES-SCHEMA.md](NETWORK-INTERFACES-SCHEMA.md) for the documentation template and style guide.

## Related Documents

- [Schema Coverage Analysis](SCHEMA-COVERAGE-ANALYSIS.md) - Initial analysis
- [API README](API-README.md) - Main API documentation
- [Quick Reference](API-QUICK-REFERENCE.md) - Quick command reference
- [OpenAPI Specification](api-documentation.yaml) - OpenAPI 3.0 spec

---

**Document Version**: 2.0  
**Last Updated**: February 11, 2026  
**Next Review**: Weekly
