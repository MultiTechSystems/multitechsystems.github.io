# MTS Device API - Quick Reference Guide

## Base URL
```
http://192.168.2.1  (or your device IP)
```

## Authentication

### Login
```bash
POST /api/login
Content-Type: application/json

{
  "username": "admin",
  "password": "admin"
}
```

### Logout
```bash
POST /api/logout
```

## Common Endpoints

### System Information
```bash
# Get all system info
GET /api?fields=system

# Get specific fields
GET /api?fields=system/capabilities
GET /api?fields=system/firmware
GET /api?fields=system/dbDirty

# Update system
PUT /api/system
{
  "deviceName": "MyDevice",
  "deviceHostname": "mydevice"
}
```

### Cellular Configuration
```bash
# Get cellular config
GET /api?fields=cellular

# Update cellular
PUT /api/cellular
{
  "enabled": true,
  "providerProfiles": [...],
  "simProfiles": [...]
}
```

### User Management
```bash
# List users
GET /api?fields=users

# Get specific user
GET /api?fields=users/{username}

# Create user
POST /api/users
{
  "name": "newuser",
  "password": "password",
  "permission": "user"
}

# Update user
PUT /api/users/{username}
{
  "email": "user@example.com"
}

# Delete user
DELETE /api/users/{username}
```

### Network Configuration
```bash
# DNS
GET /api?fields=dns
PUT /api/dns

# WiFi
GET /api?fields=wifi
PUT /api/wifi

# DHCP
GET /api?fields=dhcp
PUT /api/dhcp

# Routes
GET /api?fields=routes
PUT /api/routes

# Firewall
GET /api?fields=firewall
PUT /api/firewall
```

### Status & Monitoring
```bash
# Get status notifications
GET /api?fields=status

# Clear notification
DELETE /api/status/{guid}

# Get statistics
GET /api?fields=stats

# Event log
GET /api?fields=eventlog
```

### LoRa Management
```bash
# Get LoRa config
GET /api?fields=loraNetwork

# Update LoRa config
PUT /api/loraNetwork

# List gateways
GET /api/lora/gateways

# Delete gateway
DELETE /api/lora/gateways/{gwEui}

# Restart LoRa service
POST /api/lora/restart
```

### GPS
```bash
# Get GPS data
GET /api?fields=gps

# Update GPS config
PUT /api/gps
```

### Docker
```bash
# Get Docker config
GET /api?fields=docker

# Update Docker config
PUT /api/docker
```

## Command Endpoints

### Configuration Management
```bash
# Save configuration
POST /api/command/save

# Revert configuration
POST /api/command/revert

# Save and apply
POST /api/command/save_apply

# Upload config file
POST /api/command/upload_config
(multipart/form-data)

# Download config
POST /api/command/download_config
```

### System Commands
```bash
# Restart device
POST /api/command/restart

# Reset modem
POST /api/command/reset_modem

# Reset WiFi
POST /api/command/reset_wifi

# Reset Bluetooth
POST /api/command/reset_bluetooth
```

### Firmware Management
```bash
# Check for updates
POST /api/command/firmware_check

# Upgrade firmware
POST /api/command/firmware_upgrade
(multipart/form-data with firmware file)

# Upgrade radio firmware
POST /api/command/radio/firmware_upgrade
```

### Network Testing
```bash
# Ping test
POST /api/command/ping
{
  "host": "8.8.8.8",
  "count": 4
}

# Start continuous ping
POST /api/command/continuous_ping_start
{
  "host": "8.8.8.8"
}

# Stop continuous ping
POST /api/command/continuous_ping_stop
```

### User Commands
```bash
# Change password
POST /api/command/passwd
{
  "oldPassword": "oldpass",
  "newPassword": "newpass"
}
```

### Radio Commands
```bash
# Check SIM status
POST /api/command/radio/check_sim_status

# Unlock SIM card
POST /api/command/radio/unlock_sim_card
{
  "pin": "1234"
}

# Execute AT command
POST /api/command/radio/cmd
{
  "command": "AT+CGMI"
}

# Force cellular scan
POST /api/command/radio/force_cellular_scan

# Get cellular scan results
POST /api/command/radio/get_cellular_scan
```

### SMS Commands
```bash
# Send SMS
POST /api/command/sms_send
{
  "number": "+1234567890",
  "message": "Hello from device"
}
```

## Query Parameters

### For GET /api
- `fields` - Comma-separated list of resources (required)
- `inactivity` - Don't update session timeout (optional, boolean)
- `default` - Return default values (optional, boolean)

Examples:
```bash
# Single resource
GET /api?fields=system

# Multiple resources
GET /api?fields=system,cellular,users

# Nested field
GET /api?fields=system/capabilities

# With inactivity flag
GET /api?fields=status&inactivity=true

# Get defaults
GET /api?fields=cellular&default=true
```

## Available Resources

### Core Resources
- `system` - System information
- `status` - Status notifications
- `users` - User accounts
- `customRoles` - Custom roles
- `permissions` - Permissions

### Network Resources
- `cellular` - Cellular config
- `wifi` - WiFi config
- `dns` - DNS config
- `dhcp` - DHCP config
- `ni` - Network interfaces
- `routes` - Static routes
- `nat` - NAT rules
- `firewall` - Firewall rules
- `filters` - Traffic filters
- `trustedIp` - Trusted IPs

### VPN Resources
- `ovpnTunnels` - OpenVPN
- `ipsecTunnels` - IPsec
- `greTunnels` - GRE tunnels

### IoT Resources
- `loraNetwork` - LoRa network server
- `lora` - LoRa management
- `scada` - SCADA protocols
- `mqttBroker` - MQTT broker

### Device Resources
- `docker` - Docker containers
- `customApps` - Custom apps
- `packages` - Packages
- `licensing` - Licenses

### Communication
- `gps` - GPS
- `serial` - Serial ports
- `sms` - SMS
- `bluetooth` - Bluetooth
- `bluetoothLowEnergy` - BLE

### Security
- `certificate` - Certificates
- `cacertificates` - CA certs
- `secureProtocols` - TLS/SSL
- `passwordComplexityRules` - Password rules

### Monitoring
- `stats` - Statistics
- `eventlog` - Event log
- `syslog` - Syslog config
- `alert` - Alerts
- `snmp` - SNMP config

## Response Format

### Success
```json
{
  "success": true,
  "result": {
    // Data here
  }
}
```

### Error
```json
{
  "success": false,
  "error": "Error message",
  "code": 404
}
```

## HTTP Status Codes
- `200` - Success
- `400` - Bad request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not found
- `409` - Conflict
- `500` - Server error

## User Roles

### Admin
- Full access to all endpoints
- Can manage users
- Can execute all commands

### User (Engineer)
- Read/write most config
- Cannot manage other users
- Can execute most commands

### Guest (Monitor)
- Read-only access
- Cannot modify config
- Cannot execute commands

## Typical Workflow

```bash
# 1. Login
curl -X POST http://192.168.2.1/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin"}' \
  -c cookies.txt

# 2. Get current config
curl -X GET "http://192.168.2.1/api?fields=cellular" \
  -b cookies.txt

# 3. Update config
curl -X PUT http://192.168.2.1/api/cellular \
  -H "Content-Type: application/json" \
  -d '{"enabled":true}' \
  -b cookies.txt

# 4. Check for unsaved changes
curl -X GET "http://192.168.2.1/api?fields=system/dbDirty" \
  -b cookies.txt

# 5. Save config
curl -X POST http://192.168.2.1/api/command/save \
  -b cookies.txt

# 6. Logout
curl -X POST http://192.168.2.1/api/logout \
  -b cookies.txt
```

## Tips

1. **Always save**: Configuration changes are not persistent until you call `/api/command/save`

2. **Check dbDirty**: Query `system/dbDirty` to see if there are unsaved changes

3. **Session management**: Sessions expire after inactivity. Use `inactivity=true` for monitoring queries

4. **Batch queries**: Request multiple resources in one call: `fields=system,cellular,users`

5. **Nested access**: Access nested fields directly: `fields=system/capabilities`

6. **Error handling**: Always check the `success` field in responses

7. **HTTPS**: Use HTTPS in production for security

8. **Permissions**: Check user role before attempting operations

## Common Patterns

### Check device status
```bash
GET /api?fields=system/dbDirty,status
```

### Full system snapshot
```bash
GET /api?fields=system,cellular,wifi,dns,users,status
```

### Update and save
```bash
PUT /api/cellular {...}
POST /api/command/save
```

### Monitor without timeout
```bash
GET /api?fields=status&inactivity=true
```

### Reset to defaults
```bash
GET /api?fields=cellular&default=true
```

## File Uploads

For endpoints that accept files (firmware, certificates, etc.):

```bash
curl -X POST http://192.168.2.1/api/command/firmware_upgrade \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/path/to/firmware.bin" \
  -b cookies.txt
```

## Troubleshooting

### 401 Unauthorized
- Login first with `/api/login`
- Check if session cookie is present
- Session may have expired

### 409 Conflict
- Device may be in commissioning mode
- Check `/api/policy` for device state

### Changes not persisting
- Call `/api/command/save` after changes
- Check `system/dbDirty` status

### Device unavailable
- Device may be rebooting
- Check `/api/policy`
- Wait for startup to complete

---

**Quick Reference Version**: 1.0.0  
**Last Updated**: December 17, 2025
