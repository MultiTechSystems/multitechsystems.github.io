# MTS Device API Documentation - Index

This directory contains comprehensive API documentation for the Multi-Tech Systems (MTS) IoT device management platform, generated from the `mtsDeviceAPI` and `mtsDeviceUI` projects.

## Documentation Files

### 📄 Main Documentation

1. **`api-documentation.yaml`** - OpenAPI 3.0 Specification (YAML format)
   - Complete API specification in OpenAPI 3.0 format
   - Includes all endpoints, schemas, and examples
   - Can be imported into Swagger UI, Postman, or other API tools
   - Use this for code generation with OpenAPI Generator

2. **`api-documentation.json`** - OpenAPI 3.0 Specification (JSON format)
   - Same content as YAML but in JSON format
   - Better compatibility with some tools
   - Programmatically parseable

3. **`API-README.md`** - Complete API Guide
   - Comprehensive documentation with detailed explanations
   - Authentication and authorization guide
   - Complete endpoint reference
   - Code examples in curl
   - Configuration workflows
   - Troubleshooting guide
   - Best practices

4. **`API-QUICK-REFERENCE.md`** - Quick Reference Guide
   - Concise command reference
   - All endpoints at a glance
   - Common usage patterns
   - Quick copy-paste examples
   - Perfect for developers already familiar with the API

5. **`MTS-Device-API.postman_collection.json`** - Postman Collection
   - Ready-to-import Postman collection
   - Pre-configured requests for all major endpoints
   - Variables for easy configuration
   - Organized by functional area

## How to Use

### View Documentation

#### Option 1: Swagger UI (Recommended for Interactive Exploration)

**Online:**
1. Go to https://editor.swagger.io/
2. Click File → Import File
3. Select `api-documentation.yaml`
4. Explore and test the API interactively

**Local:**
```bash
# Using Docker
docker run -p 8080:8080 -e SWAGGER_JSON=/api-documentation.yaml \
  -v $(pwd):/usr/share/nginx/html swaggerapi/swagger-ui

# Then open http://localhost:8080
```

#### Option 2: Postman (Recommended for Testing)

1. Open Postman
2. Click Import → Upload Files
3. Select `MTS-Device-API.postman_collection.json`
4. Update the `baseUrl` variable to your device IP
5. Start making requests!

#### Option 3: VS Code (Recommended for Development)

1. Install the "OpenAPI (Swagger) Editor" extension
2. Open `api-documentation.yaml`
3. Right-click → "Preview Swagger"

### Generate Client Code

Use OpenAPI Generator to create client libraries:

```bash
# Install OpenAPI Generator
npm install @openapitools/openapi-generator-cli -g

# Python client
openapi-generator-cli generate \
  -i api-documentation.yaml \
  -g python \
  -o ./clients/python

# JavaScript/TypeScript client
openapi-generator-cli generate \
  -i api-documentation.yaml \
  -g typescript-axios \
  -o ./clients/typescript

# Java client
openapi-generator-cli generate \
  -i api-documentation.yaml \
  -g java \
  -o ./clients/java

# C# client
openapi-generator-cli generate \
  -i api-documentation.yaml \
  -g csharp \
  -o ./clients/csharp

# Go client
openapi-generator-cli generate \
  -i api-documentation.yaml \
  -g go \
  -o ./clients/go
```

### Quick Start

1. **Read the Overview**: Start with `API-README.md` for a comprehensive introduction

2. **Try Basic Requests**: Use the examples in `API-QUICK-REFERENCE.md` to test basic functionality

3. **Import to Postman**: Load `MTS-Device-API.postman_collection.json` for easy testing

4. **Generate Client**: Use `api-documentation.yaml` with OpenAPI Generator for your language

## API Overview

### Base URL
```
http://192.168.2.1  (default device IP)
```

### Authentication
Session-based authentication with cookies. Login required before accessing most endpoints.

### Main Endpoint Patterns

| Pattern | Description |
|---------|-------------|
| `GET /api?fields=<resource>` | Query resource data |
| `PUT /api/<resource>` | Update resource configuration |
| `POST /api/command/<action>` | Execute commands |
| `DELETE /api/<resource>/<id>` | Delete resource items |

### Key Resources

**System & Configuration:**
- `system` - Device information and capabilities
- `users` - User management
- `status` - System notifications

**Network:**
- `cellular` - Cellular connectivity
- `wifi` - WiFi configuration
- `dns`, `dhcp`, `routes` - Network services
- `firewall`, `nat` - Security

**IoT Protocols:**
- `loraNetwork` - LoRa network server
- `scada` - SCADA protocols
- `mqttBroker` - MQTT broker

**Device Management:**
- `docker` - Container management
- `gps` - Location services
- `serial` - Serial ports
- `sms` - SMS messaging

### Common Commands

```bash
# Login
POST /api/login

# Save configuration
POST /api/command/save

# Restart device
POST /api/command/restart

# Logout
POST /api/logout
```

## Documentation Structure

```
mtsDeviceAPI/
├── controllers/           # API endpoint controllers (C++)
│   ├── manifest.json     # List of all controllers
│   ├── permissions-*.json # Role-based permissions
│   └── description.txt   # Permission format description
├── json/                 # JSON Schema definitions
│   ├── permissive/       # Permissive validation schemas
│   └── strict/           # Strict validation schemas
├── src/                  # Controller implementations
└── include/              # Header files

mtsDeviceUI/
├── src/
│   ├── plugins/http.ts   # HTTP client implementation
│   ├── pages/            # UI pages (API usage examples)
│   └── types/            # TypeScript type definitions

Generated Documentation/
├── api-documentation.yaml              # OpenAPI spec (YAML)
├── api-documentation.json              # OpenAPI spec (JSON)
├── API-README.md                       # Complete guide
├── API-QUICK-REFERENCE.md              # Quick reference
├── MTS-Device-API.postman_collection.json # Postman collection
└── API-DOCUMENTATION-INDEX.md          # This file
```

## API Features

### ✅ Comprehensive Coverage
- 70+ resource endpoints
- 50+ command endpoints
- Complete schema definitions
- Role-based access control

### ✅ Well-Documented
- Detailed descriptions for all endpoints
- Request/response examples
- Error code documentation
- Authentication flows

### ✅ Standards-Compliant
- OpenAPI 3.0 specification
- RESTful design principles
- JSON request/response format
- Standard HTTP status codes

### ✅ Developer-Friendly
- Postman collection included
- Code generation support
- Interactive documentation
- Quick reference guide

## Supported Operations

### Configuration Management
- Query current configuration
- Update configuration
- Save/revert changes
- Upload/download config files

### Device Control
- Restart device
- Reset hardware modules
- Firmware updates
- Factory reset

### Network Management
- Cellular connectivity
- WiFi access point/client
- VPN tunnels (OpenVPN, IPsec, GRE)
- Firewall and routing

### IoT Protocols
- LoRa network server
- SCADA (BACnet, Modbus)
- MQTT broker
- GPS tracking

### User Management
- Create/update/delete users
- Role assignment
- Password management
- Permission control

### Monitoring
- System status
- Statistics and metrics
- Event logs
- Alerts and notifications

## Role-Based Access

| Role | Description | Access Level |
|------|-------------|--------------|
| **Admin** | Full system access | Read/Write/Execute all |
| **User** (Engineer) | Configuration access | Read/Write most, limited security |
| **Guest** (Monitor) | Read-only access | Read status/monitoring only |

## Example Workflows

### 1. Basic Configuration Change
```bash
# Login
curl -X POST http://192.168.2.1/api/login \
  -d '{"username":"admin","password":"admin"}' -c cookies.txt

# Update configuration
curl -X PUT http://192.168.2.1/api/cellular \
  -d '{"enabled":true}' -b cookies.txt

# Save changes
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt

# Logout
curl -X POST http://192.168.2.1/api/logout -b cookies.txt
```

### 2. Monitoring
```bash
# Get system status
curl -X GET "http://192.168.2.1/api?fields=system,status,stats" \
  -b cookies.txt

# Monitor without session timeout
curl -X GET "http://192.168.2.1/api?fields=status&inactivity=true" \
  -b cookies.txt
```

### 3. User Management
```bash
# Create user
curl -X POST http://192.168.2.1/api/users \
  -d '{"name":"engineer1","password":"pass","permission":"user"}' \
  -b cookies.txt

# Save
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
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

## Additional Resources

### Source Projects
- **mtsDeviceAPI**: Backend C++ API implementation
- **mtsDeviceUI**: Frontend Vue.js web interface

### JSON Schemas
All resource schemas are available in:
- `mtsDeviceAPI/json/permissive/*.schema.json`
- `mtsDeviceAPI/json/strict/*.schema.json`

### Permissions
Role permissions are defined in:
- `mtsDeviceAPI/controllers/permissions-admin.json`
- `mtsDeviceAPI/controllers/permissions-user.json`
- `mtsDeviceAPI/controllers/permissions-guest.json`

## Support

For technical support:
- Website: https://www.multitech.com/support
- Email: support@multitech.com

## Version Information

- **Documentation Version**: 1.0.0
- **Generated**: December 17, 2025
- **Based on**: mtsDeviceAPI and mtsDeviceUI projects
- **OpenAPI Version**: 3.0.3

## License

This documentation is provided for use with Multi-Tech Systems devices.
Refer to your device license agreement for terms and conditions.

---

**Quick Links:**
- [Complete Guide](API-README.md)
- [Quick Reference](API-QUICK-REFERENCE.md)
- [OpenAPI Spec (YAML)](api-documentation.yaml)
- [OpenAPI Spec (JSON)](api-documentation.json)
- [Postman Collection](MTS-Device-API.postman_collection.json)
