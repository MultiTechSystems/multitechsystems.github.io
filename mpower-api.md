# mPower Edge Intelligence Device API

Complete REST API documentation for managing Multi-Tech Systems IoT gateway devices (mPower Edge Intelligence platform).

## Overview

The mPower Device API is a comprehensive RESTful API that provides complete control and monitoring capabilities for MTS IoT gateway devices including Conduit, MultiConnect, and other mPower-enabled devices.

### Key Features

- **System Configuration**: Device information, capabilities, and system settings
- **Cellular Connectivity**: Provider profiles, SIM management, connection monitoring
- **Network Services**: WiFi, DNS, DHCP, routing, VPN tunnels
- **Security**: Firewall, certificates, user authentication and authorization
- **IoT Protocols**: LoRa network server, SCADA protocols (BACnet, Modbus)
- **Device Management**: Firmware updates, diagnostics, Docker containers
- **Location Services**: GPS configuration and tracking
- **Communication**: SMS messaging, serial port configuration

## Documentation Resources

### 📘 [Complete API Guide](mpower-api-docs/API-README.md)
Comprehensive documentation with detailed explanations, authentication guide, complete endpoint reference, code examples, configuration workflows, and troubleshooting.

### 📋 [Quick Reference](mpower-api-docs/API-QUICK-REFERENCE.md)
Concise command reference with all endpoints at a glance, common usage patterns, and quick copy-paste examples.

### 📑 [Documentation Index](mpower-api-docs/API-DOCUMENTATION-INDEX.md)
Master index with overview of all documentation, how-to guides, tool integration, and quick start instructions.

### 🔧 OpenAPI/Swagger Specification
- **[YAML Format](mpower-api-docs/api-documentation.yaml)** - For Swagger UI and code generators
- **[JSON Format](mpower-api-docs/api-documentation.json)** - For programmatic access

### 📦 [Postman Collection](mpower-api-docs/MTS-Device-API.postman_collection.json)
Ready-to-import Postman collection with pre-configured requests for all major endpoints.

## Quick Start

### Authentication

The API uses session-based authentication with cookies:

```bash
# Login
curl -X POST http://192.168.2.1/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "admin"}' \
  -c cookies.txt

# Use session cookie for subsequent requests
curl -X GET "http://192.168.2.1/api?fields=system" -b cookies.txt

# Logout
curl -X POST http://192.168.2.1/api/logout -b cookies.txt
```

### Basic API Usage

#### Query Resources

```bash
# Get system information
curl -X GET "http://192.168.2.1/api?fields=system" -b cookies.txt

# Get multiple resources
curl -X GET "http://192.168.2.1/api?fields=system,cellular,users" -b cookies.txt

# Get nested fields
curl -X GET "http://192.168.2.1/api?fields=system/capabilities" -b cookies.txt
```

#### Update Configuration

```bash
# Update system settings
curl -X PUT http://192.168.2.1/api/system \
  -H "Content-Type: application/json" \
  -d '{"deviceName": "MyDevice"}' \
  -b cookies.txt

# Save configuration
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

#### Execute Commands

```bash
# Restart device
curl -X POST http://192.168.2.1/api/command/restart -b cookies.txt

# Ping test
curl -X POST http://192.168.2.1/api/command/ping \
  -H "Content-Type: application/json" \
  -d '{"host": "8.8.8.8", "count": 4}' \
  -b cookies.txt
```

## API Structure

### Resource Endpoints

| HTTP Method | Pattern | Description |
|-------------|---------|-------------|
| GET | `/api?fields=<resource>` | Retrieve resource data |
| PUT | `/api/<resource>` | Update entire resource |
| POST | `/api/<resource>` | Create new resource item |
| DELETE | `/api/<resource>/<id>` | Delete resource item |

### Available Resources

#### Core System
- `system` - Device information and capabilities
- `status` - System status notifications
- `users` - User account management
- `customRoles` - Custom role definitions

#### Network
- `cellular` - Cellular connectivity
- `wifi` - WiFi access point and client
- `dns` - DNS server configuration
- `dhcp` - DHCP server configuration
- `routes` - Static routes
- `firewall` - Firewall rules
- `nat` - NAT configuration

#### VPN
- `ovpnTunnels` - OpenVPN tunnels
- `ipsecTunnels` - IPsec tunnels
- `greTunnels` - GRE tunnels

#### IoT Protocols
- `loraNetwork` - LoRa network server
- `lora` - LoRa gateway management
- `scada` - SCADA protocol configuration
- `mqttBroker` - MQTT broker

#### Device Management
- `docker` - Docker container management
- `gps` - GPS configuration and data
- `serial` - Serial port configuration
- `sms` - SMS messaging

#### Security
- `certificate` - Device certificates
- `cacertificates` - CA certificates
- `secureProtocols` - TLS/SSL protocols

### Command Endpoints

Commands are executed via POST to `/api/command/<command>`:

#### Configuration Commands
- `save` - Save configuration to persistent storage
- `revert` - Revert to last saved configuration
- `restart` - Restart device

#### Firmware Commands
- `firmware_upgrade` - Upgrade device firmware
- `radio/firmware_upgrade` - Upgrade cellular radio firmware

#### Network Commands
- `ping` - Execute ping test
- `reset_modem` - Reset cellular modem
- `reset_wifi` - Reset WiFi module

## Interactive Documentation

### View in Swagger UI

1. Go to [Swagger Editor](https://editor.swagger.io/)
2. Click File → Import File
3. Select the [api-documentation.yaml](mpower-api-docs/api-documentation.yaml) file
4. Explore and test the API interactively

### Import to Postman

1. Open Postman
2. Click Import → Upload Files
3. Select [MTS-Device-API.postman_collection.json](mpower-api-docs/MTS-Device-API.postman_collection.json)
4. Update the `baseUrl` variable to your device IP
5. Start making requests!

## Generate Client Code

Use OpenAPI Generator to create client libraries in various languages:

```bash
# Install OpenAPI Generator
npm install @openapitools/openapi-generator-cli -g

# Generate Python client
openapi-generator-cli generate \
  -i mpower-api-docs/api-documentation.yaml \
  -g python \
  -o ./python-client

# Generate JavaScript/TypeScript client
openapi-generator-cli generate \
  -i mpower-api-docs/api-documentation.yaml \
  -g typescript-axios \
  -o ./typescript-client

# Generate Java client
openapi-generator-cli generate \
  -i mpower-api-docs/api-documentation.yaml \
  -g java \
  -o ./java-client
```

## Role-Based Access Control

The API implements role-based access control with three built-in roles:

| Role | Description | Access Level |
|------|-------------|--------------|
| **Admin** | Full system access | Read/Write/Execute all |
| **User** (Engineer) | Configuration access | Read/Write most, limited security |
| **Guest** (Monitor) | Read-only access | Read status/monitoring only |

## Response Format

All API responses follow a consistent JSON format:

### Success Response
```json
{
  "success": true,
  "result": {
    // Response data varies by endpoint
  }
}
```

### Error Response
```json
{
  "success": false,
  "error": "Error message description",
  "code": 404
}
```

## Configuration Workflow

The device uses a two-stage configuration model:

1. **Runtime Configuration** - Changes made via PUT requests update the runtime configuration
2. **Persistent Configuration** - Use `POST /api/command/save` to persist changes

### Typical Workflow

```bash
# 1. Login
curl -X POST http://192.168.2.1/api/login \
  -d '{"username":"admin","password":"admin"}' -c cookies.txt

# 2. Get current configuration
curl -X GET "http://192.168.2.1/api?fields=cellular" -b cookies.txt

# 3. Update configuration
curl -X PUT http://192.168.2.1/api/cellular \
  -d '{"enabled": true}' -b cookies.txt

# 4. Save configuration
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt

# 5. Logout
curl -X POST http://192.168.2.1/api/logout -b cookies.txt
```

## Examples

### Cellular Configuration

```bash
# Get cellular configuration
curl -X GET "http://192.168.2.1/api?fields=cellular" -b cookies.txt

# Enable cellular with provider profile
curl -X PUT http://192.168.2.1/api/cellular \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "enabled": true,
    "providerProfiles": [{
      "_id": "550e8400-e29b-41d4-a716-446655440000",
      "profileName": "My Carrier",
      "cellularMode": "4g-preferred",
      "dataCtx": {
        "apnString": "internet",
        "contextIpMode": "AUTO",
        "authentication": {
          "type": "NONE",
          "username": "",
          "password": ""
        }
      }
    }]
  }'

# Save configuration
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

### User Management

```bash
# List all users
curl -X GET "http://192.168.2.1/api?fields=users" -b cookies.txt

# Create new user
curl -X POST http://192.168.2.1/api/users \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "name": "engineer1",
    "password": "password123",
    "permission": "user",
    "firstName": "John",
    "lastName": "Doe",
    "email": "john.doe@example.com"
  }'

# Save changes
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

### LoRa Network Server

```bash
# Get LoRa configuration
curl -X GET "http://192.168.2.1/api?fields=loraNetwork" -b cookies.txt

# Update LoRa network server mode
curl -X PUT http://192.168.2.1/api/loraNetwork \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "enabled": true,
    "mode": "NETWORK_SERVER"
  }'

# Restart LoRa service
curl -X POST http://192.168.2.1/api/lora/restart -b cookies.txt
```

## Troubleshooting

### Common Issues

**401 Unauthorized**
- Solution: Login first with `/api/login`
- Check: Session cookie is being sent

**409 Conflict**
- Solution: Device may be in commissioning mode
- Check: `/api/policy` endpoint

**Changes not persisting**
- Solution: Call `/api/command/save` after changes
- Check: `system/dbDirty` status

**Device unavailable**
- Solution: Wait for device to complete startup
- Check: `/api/policy` for device state

## Support

For technical support and additional documentation:
- Visit: [Multi-Tech Support Portal](https://support.multitech.com)
- Email: support@multitech.com
- GitHub: [MultiTechSystems](https://github.com/MultiTechSystems)

## Version Information

- **API Version**: Based on mPower Edge Intelligence platform
- **Documentation Version**: 1.0.0
- **Last Updated**: December 17, 2025
- **OpenAPI Version**: 3.0.3

---

**Related Documentation:**
- [LoRaWAN App Connect](lorawan-app-connect)
- [Conduit Custom App Development](conduit-custom-app-development)
- [Conduit Cursor Development](conduit-cursor-development)
