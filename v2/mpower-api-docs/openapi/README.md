# OpenAPI Documentation Structure

This directory contains the modular OpenAPI 3.0 specification for the MTS Device API.

## Directory Structure

```
openapi/
├── api-documentation.yaml    # Main OpenAPI spec (references other files)
├── README.md                 # This file
├── schemas/                  # Schema definitions by domain
│   ├── common.yaml           # Common types (responses, auth, etc.)
│   ├── system.yaml           # System configuration
│   ├── users.yaml            # User management
│   ├── cellular.yaml         # Cellular connectivity
│   ├── sms.yaml              # SMS messaging
│   ├── network.yaml          # Network (WiFi, DNS, DHCP, routes, WAN)
│   ├── vpn.yaml              # VPN tunnels (OpenVPN, IPsec, GRE)
│   ├── firewall.yaml         # Firewall and NAT
│   ├── lora.yaml             # LoRa network server
│   ├── services.yaml         # Network services (SNMP, NTP, SMTP, etc.)
│   ├── device-management.yaml # Device management features
│   ├── connectivity.yaml     # Bluetooth, GPS, serial, IP pipes
│   ├── advanced.yaml         # SCADA, MQTT, alerts, I/O
│   └── status.yaml           # Status and logging
├── paths/                    # API path definitions by domain
│   ├── authentication.yaml   # Login/logout/session
│   ├── system.yaml           # System endpoints
│   ├── users.yaml            # User management endpoints
│   ├── cellular.yaml         # Cellular endpoints
│   ├── sms.yaml              # SMS endpoints
│   ├── network.yaml          # Network endpoints
│   ├── vpn.yaml              # VPN tunnel endpoints
│   ├── firewall.yaml         # Firewall endpoints
│   ├── lora.yaml             # LoRa endpoints
│   ├── services.yaml         # Network services endpoints
│   ├── device-management.yaml # Device management endpoints
│   ├── connectivity.yaml     # Connectivity endpoints
│   ├── commands.yaml         # Command endpoints
│   └── advanced.yaml         # Advanced feature endpoints
└── components/               # Shared components (future use)
```

## Schema Sources

All schemas are extracted from the mtsDeviceAPI source code (`json/strict/*.schema.json`).

See the detailed Markdown documentation in `../` for comprehensive information:
- [API-SCHEMAS-REFERENCE.md](../API-SCHEMAS-REFERENCE.md) - Core schemas
- [SMS-SCHEMA.md](../SMS-SCHEMA.md) - SMS configuration
- [VPN-SCHEMAS.md](../VPN-SCHEMAS.md) - VPN tunnels
- [NETWORK-SERVICES-SCHEMAS.md](../NETWORK-SERVICES-SCHEMAS.md) - Network services
- [DEVICE-MANAGEMENT-SCHEMAS.md](../DEVICE-MANAGEMENT-SCHEMAS.md) - Device management
- [CONNECTIVITY-SCHEMAS.md](../CONNECTIVITY-SCHEMAS.md) - Connectivity features
- [ADVANCED-FEATURES-SCHEMAS.md](../ADVANCED-FEATURES-SCHEMAS.md) - Advanced features
- [LOGGING-STATUS-SCHEMAS.md](../LOGGING-STATUS-SCHEMAS.md) - Logging and status
- [LORA-NETWORK-SCHEMA.md](../LORA-NETWORK-SCHEMA.md) - LoRa configuration

## Usage

### Viewing with Swagger UI

You can use the main `api-documentation.yaml` with any OpenAPI-compatible tool:

```bash
# Using swagger-ui-express (Node.js)
npm install swagger-ui-express yaml

# Using swagger-cli to validate
npx @apidevtools/swagger-cli validate api-documentation.yaml
```

### Bundling into Single File

To bundle all files into a single OpenAPI document:

```bash
# Using swagger-cli
npx @apidevtools/swagger-cli bundle api-documentation.yaml -o bundled.yaml

# Or to JSON
npx @apidevtools/swagger-cli bundle api-documentation.yaml -o bundled.json -t json
```

### Validation

```bash
# Validate the spec
npx @apidevtools/swagger-cli validate api-documentation.yaml
```

## Version History

- **2.0.0** (February 2026): Complete restructure with modular schema files
  - Split into separate schema and path files by domain
  - All schemas extracted from source code
  - Full coverage of 58+ resources

- **1.0.0** (Previous): Original monolithic specification

## Contributing

When updating schemas:

1. First update the source `.schema.json` file in mtsDeviceAPI
2. Then update the corresponding YAML file in `schemas/`
3. Update the Markdown documentation if needed
4. Validate the spec before committing

## API Endpoints Pattern

The API follows consistent patterns:

| Operation | Endpoint Pattern |
|-----------|-----------------|
| Read | `GET /api?fields=<resource>` |
| Update | `PUT /api/<resource>` |
| Create | `POST /api/<resource>` |
| Delete | `DELETE /api/<resource>/<id>` |
| Command | `POST /api/command/<action>` |

## Authentication

All endpoints (except `/api/login`) require session authentication:

```bash
# Login
curl -c cookies.txt -X POST http://device/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin"}'

# Use session cookie for subsequent requests
curl -b cookies.txt http://device/api?fields=system
```
