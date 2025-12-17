# MTS Device API Documentation

This repository contains comprehensive API documentation for the Multi-Tech Systems (MTS) IoT device management platform (mPower Edge Intelligence).

## Overview

The MTS Device API is a RESTful API that provides complete control and monitoring capabilities for MTS IoT gateway devices. The API enables configuration and management of:

- **System Configuration**: Device information, capabilities, and system settings
- **Cellular Connectivity**: Provider profiles, SIM management, connection monitoring
- **Network Services**: WiFi, DNS, DHCP, routing, VPN tunnels
- **Security**: Firewall, certificates, user authentication and authorization
- **IoT Protocols**: LoRa network server, SCADA protocols (BACnet, Modbus)
- **Device Management**: Firmware updates, diagnostics, Docker containers
- **Location Services**: GPS configuration and tracking
- **Communication**: SMS messaging, serial port configuration

## Documentation Files

- **`api-documentation.yaml`**: Complete OpenAPI 3.0 specification
  - Can be viewed using [Swagger UI](https://swagger.io/tools/swagger-ui/)
  - Can be imported into [Postman](https://www.postman.com/)
  - Can be used with code generators like [OpenAPI Generator](https://openapi-generator.tech/)

## Quick Start

### Authentication

The API uses session-based authentication with cookies. To authenticate:

1. **Login** - POST to `/api/login` with credentials:
```bash
curl -X POST http://192.168.2.1/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "admin"}' \
  -c cookies.txt
```

2. **Use the session cookie** for subsequent requests:
```bash
curl -X GET http://192.168.2.1/api?fields=system \
  -b cookies.txt
```

3. **Logout** when done:
```bash
curl -X POST http://192.168.2.1/api/logout \
  -b cookies.txt
```

### Basic API Usage

#### Query Resources

The primary endpoint for retrieving data is `/api` with a `fields` query parameter:

```bash
# Get system information
curl -X GET "http://192.168.2.1/api?fields=system" -b cookies.txt

# Get multiple resources
curl -X GET "http://192.168.2.1/api?fields=system,cellular,users" -b cookies.txt

# Get nested fields
curl -X GET "http://192.168.2.1/api?fields=system/capabilities" -b cookies.txt
```

#### Update Configuration

Use PUT requests to update resource configurations:

```bash
# Update system settings
curl -X PUT http://192.168.2.1/api/system \
  -H "Content-Type: application/json" \
  -d '{"deviceName": "MyDevice"}' \
  -b cookies.txt

# Update cellular configuration
curl -X PUT http://192.168.2.1/api/cellular \
  -H "Content-Type: application/json" \
  -d '{"enabled": true}' \
  -b cookies.txt
```

#### Execute Commands

Use POST requests to execute device commands:

```bash
# Save configuration
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt

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

The API follows a consistent pattern for resource management:

| HTTP Method | Pattern | Description |
|-------------|---------|-------------|
| GET | `/api?fields=<resource>` | Retrieve resource data |
| PUT | `/api/<resource>` | Update entire resource |
| POST | `/api/<resource>` | Create new resource item |
| DELETE | `/api/<resource>/<id>` | Delete resource item |

### Available Resources

#### Core System Resources
- `system` - Device information and capabilities
- `status` - System status notifications
- `users` - User account management
- `customRoles` - Custom role definitions
- `permissions` - Permission management

#### Network Resources
- `cellular` - Cellular connectivity
- `wifi` - WiFi access point and client
- `dns` - DNS server configuration
- `dhcp` - DHCP server configuration
- `ni` - Network interfaces
- `routes` - Static routes
- `nat` - NAT configuration
- `firewall` - Firewall rules
- `filters` - Traffic filters

#### VPN Resources
- `ovpnTunnels` - OpenVPN tunnels
- `ipsecTunnels` - IPsec tunnels
- `greTunnels` - GRE tunnels

#### IoT Protocol Resources
- `loraNetwork` - LoRa network server
- `lora` - LoRa gateway management
- `scada` - SCADA protocol configuration
- `mqtt` - MQTT broker

#### Device Management Resources
- `docker` - Docker container management
- `customApps` - Custom application management
- `packages` - Package management
- `licensing` - License management

#### Communication Resources
- `gps` - GPS configuration and data
- `serial` - Serial port configuration
- `sms` - SMS messaging
- `bluetooth` - Bluetooth configuration
- `bluetoothLowEnergy` - BLE configuration

#### Security Resources
- `certificate` - Device certificates
- `cacertificates` - CA certificates
- `secureProtocols` - TLS/SSL protocols
- `trustedIp` - Trusted IP addresses

#### Monitoring Resources
- `stats` - Statistics and metrics
- `eventlog` - Event log
- `syslog` - Syslog configuration
- `alert` - Alert configuration
- `snmp` - SNMP configuration

### Command Endpoints

Commands are executed via POST to `/api/command/<command>`:

#### Configuration Commands
- `save` - Save configuration to persistent storage
- `revert` - Revert to last saved configuration
- `save_apply` - Save and apply configuration
- `upload_config` - Upload configuration file
- `download_config` - Download configuration file

#### System Commands
- `restart` - Restart device
- `reset_modem` - Reset cellular modem
- `reset_wifi` - Reset WiFi module
- `reset_bluetooth` - Reset Bluetooth module

#### Firmware Commands
- `firmware_upgrade` - Upgrade device firmware
- `firmware_check` - Check for firmware updates
- `radio/firmware_upgrade` - Upgrade cellular radio firmware

#### Network Commands
- `ping` - Execute ping test
- `continuous_ping_start` - Start continuous ping
- `continuous_ping_stop` - Stop continuous ping

#### User Commands
- `passwd` - Change user password

#### LoRa Commands
- `upload_lora_license` - Upload LoRa license
- `upload_lora_fota_file` - Upload LoRa FOTA firmware

#### SMS Commands
- `sms_send` - Send SMS message

## Role-Based Access Control

The API implements role-based access control with three built-in roles:

### Admin Role
- Full read/write access to all endpoints
- Can create, modify, and delete users
- Can execute all commands
- Can modify security settings

### Engineer Role (User)
- Read/write access to most configuration endpoints
- Cannot manage users (except own account)
- Can execute most commands
- Limited access to security settings

### Monitor Role (Guest)
- Read-only access to status and monitoring endpoints
- Cannot modify configuration
- Cannot execute commands
- Cannot access security settings

Each endpoint in the API has associated permissions:
- **GET** - Read access
- **EDIT** - Modify existing items
- **ADD** - Create new items
- **DELETE** - Remove items
- **VISIBLE** - Visibility in UI

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

### Common HTTP Status Codes
- `200 OK` - Request successful
- `400 Bad Request` - Invalid request data
- `401 Unauthorized` - Authentication required or failed
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Resource not found
- `409 Conflict` - Device in commissioning mode or conflict state
- `500 Internal Server Error` - Server error

## Configuration Workflow

The device uses a two-stage configuration model:

1. **Runtime Configuration** - Changes made via PUT requests update the runtime configuration
2. **Persistent Configuration** - Use `POST /api/command/save` to persist changes

### Typical Workflow

```bash
# 1. Login
curl -X POST http://192.168.2.1/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "admin"}' \
  -c cookies.txt

# 2. Get current configuration
curl -X GET "http://192.168.2.1/api?fields=cellular" -b cookies.txt

# 3. Update configuration
curl -X PUT http://192.168.2.1/api/cellular \
  -H "Content-Type: application/json" \
  -d '{"enabled": true, ...}' \
  -b cookies.txt

# 4. Check if configuration is dirty (has unsaved changes)
curl -X GET "http://192.168.2.1/api?fields=system/dbDirty" -b cookies.txt

# 5. Save configuration
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt

# 6. Optionally restart to apply changes
curl -X POST http://192.168.2.1/api/command/restart -b cookies.txt

# 7. Logout
curl -X POST http://192.168.2.1/api/logout -b cookies.txt
```

## Special Endpoints

### Device Policy Check
Check if device is available (not in commissioning mode):
```bash
curl -X GET http://192.168.2.1/api/policy
```

### Commissioning Mode
For first-time device setup:
```bash
curl -X POST http://192.168.2.1/api/commissioning \
  -H "Content-Type: application/json" \
  -d '{"password": "newpassword"}'
```

### Status Notifications
Get system status notifications:
```bash
curl -X GET "http://192.168.2.1/api?fields=status" -b cookies.txt
```

Clear a specific notification:
```bash
curl -X DELETE http://192.168.2.1/api/status/{guid} -b cookies.txt
```

## Cellular Configuration Example

Complete example of configuring cellular connectivity:

```bash
# Get current cellular configuration
curl -X GET "http://192.168.2.1/api?fields=cellular" -b cookies.txt

# Update cellular configuration
curl -X PUT http://192.168.2.1/api/cellular \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "enabled": true,
    "providerProfiles": [
      {
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
      }
    ],
    "simProfiles": [
      {
        "_id": "550e8400-e29b-41d4-a716-446655440001",
        "profileName": "My SIM",
        "providerProfileId": "550e8400-e29b-41d4-a716-446655440000",
        "simIccid": "ANY",
        "simPin": ""
      }
    ]
  }'

# Save configuration
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

## User Management Example

```bash
# List all users (admin only)
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

# Update user
curl -X PUT http://192.168.2.1/api/users/engineer1 \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "email": "newemail@example.com",
    "enabled": true
  }'

# Delete user
curl -X DELETE http://192.168.2.1/api/users/engineer1 -b cookies.txt

# Save changes
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

## LoRa Network Server Example

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

# List registered gateways
curl -X GET http://192.168.2.1/api/lora/gateways -b cookies.txt

# Restart LoRa service
curl -X POST http://192.168.2.1/api/lora/restart -b cookies.txt
```

## Viewing the Documentation

### Using Swagger UI

1. Install Swagger UI locally or use the online editor
2. Load the `api-documentation.yaml` file
3. Interact with the API directly from the UI

**Online Swagger Editor:**
Visit https://editor.swagger.io/ and paste the contents of `api-documentation.yaml`

### Using Postman

1. Open Postman
2. Click Import → Upload Files
3. Select `api-documentation.yaml`
4. The entire API collection will be imported with all endpoints

### Generating Client Code

Use OpenAPI Generator to create client libraries in various languages:

```bash
# Install OpenAPI Generator
npm install @openapitools/openapi-generator-cli -g

# Generate Python client
openapi-generator-cli generate \
  -i api-documentation.yaml \
  -g python \
  -o ./python-client

# Generate JavaScript/TypeScript client
openapi-generator-cli generate \
  -i api-documentation.yaml \
  -g typescript-axios \
  -o ./typescript-client

# Generate Java client
openapi-generator-cli generate \
  -i api-documentation.yaml \
  -g java \
  -o ./java-client
```

## API Versioning

The current API version is included in the system information:

```bash
curl -X GET "http://192.168.2.1/api?fields=system/apiVersion" -b cookies.txt
```

## Security Considerations

1. **HTTPS**: Always use HTTPS in production environments
2. **Session Management**: Sessions expire after inactivity (default 30 minutes)
3. **Password Complexity**: Enforce strong passwords via `passwordComplexityRules`
4. **Trusted IPs**: Restrict access using `trustedIp` configuration
5. **Firewall**: Configure firewall rules to limit API access
6. **Certificates**: Use valid SSL/TLS certificates for HTTPS

## Troubleshooting

### Authentication Issues

If you receive 401 errors:
- Verify credentials are correct
- Check if session cookie is being sent
- Ensure device is not in commissioning mode

### Configuration Not Persisting

If changes are lost after reboot:
- Ensure you call `/api/command/save` after making changes
- Check `system/dbDirty` to verify if there are unsaved changes

### Device Unavailable

If you receive "device unavailable" errors:
- Check `/api/policy` to verify device state
- Device may be rebooting or in commissioning mode
- Wait for device to complete startup

## Additional Resources

- **mtsDeviceAPI**: Backend API implementation (C++)
- **mtsDeviceUI**: Frontend web interface (Vue.js)
- **JSON Schemas**: Located in `mtsDeviceAPI/json/permissive/` and `mtsDeviceAPI/json/strict/`
- **Permissions**: Defined in `mtsDeviceAPI/controllers/permissions-*.json`

## Support

For technical support and additional documentation:
- Visit: https://www.multitech.com/support
- Email: support@multitech.com

## License

This API documentation is provided for use with Multi-Tech Systems devices.
Refer to your device license agreement for terms and conditions.

---

**Document Version**: 1.0.0  
**Last Updated**: December 17, 2025  
**API Version**: Based on mtsDeviceAPI and mtsDeviceUI projects
