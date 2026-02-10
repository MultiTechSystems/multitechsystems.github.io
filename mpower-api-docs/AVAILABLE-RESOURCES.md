# Available Resources Reference

This document provides comprehensive documentation for all available API resources in the mPower Edge Intelligence platform.

## Overview

Resources are queried using the `GET /api?fields=<resource>` endpoint and updated via `PUT /api/<resource>`. Multiple resources can be queried in a single request using comma separation.

```bash
# Query single resource
GET /api?fields=system

# Query multiple resources
GET /api?fields=system,cellular,users

# Query nested field
GET /api?fields=system/capabilities
```

## Table of Contents

- [Core System Resources](#core-system-resources)
- [Network Resources](#network-resources)
- [Cellular Resources](#cellular-resources)
- [VPN Resources](#vpn-resources)
- [IoT Protocol Resources](#iot-protocol-resources)
- [Device Management Resources](#device-management-resources)
- [Communication Resources](#communication-resources)
- [Security Resources](#security-resources)
- [Monitoring Resources](#monitoring-resources)
- [Notification Resources](#notification-resources)

---

## Core System Resources

### `system`

Device information, capabilities, and system settings.

**Endpoints:**
- `GET /api?fields=system` - Get system information
- `PUT /api/system` - Update system settings

**Key Properties:**
| Property | Type | Description |
|----------|------|-------------|
| `deviceName` | string | User-defined device name |
| `deviceHostname` | string | Device hostname |
| `serialNumber` | string | Device serial number (read-only) |
| `firmware` | object | Firmware version information |
| `capabilities` | object | Device hardware capabilities |
| `dbDirty` | boolean | Unsaved configuration changes exist |
| `apiVersion` | string | API version string |

**Example - Get System Information:**
```bash
curl -X GET "http://192.168.2.1/api?fields=system" -b cookies.txt
```

**Response:**
```json
{
  "success": true,
  "result": {
    "system": {
      "deviceName": "My Gateway",
      "deviceHostname": "mtcdt",
      "serialNumber": "123456789",
      "productId": "MTCDT-247A",
      "firmware": {
        "version": "6.3.0",
        "date": "2025-01-15"
      },
      "capabilities": {
        "cellular": true,
        "wifi": true,
        "lora": true,
        "gps": true,
        "bluetooth": true
      },
      "dbDirty": false,
      "apiVersion": "1.0"
    }
  }
}
```

**Example - Update Device Name:**
```bash
curl -X PUT http://192.168.2.1/api/system \
  -H "Content-Type: application/json" \
  -d '{"deviceName": "Production Gateway 01"}' \
  -b cookies.txt
```

---

### `status`

System status notifications and alerts.

**Endpoints:**
- `GET /api?fields=status` - Get all status notifications
- `DELETE /api/status/{guid}` - Clear a specific notification

**Key Properties:**
| Property | Type | Description |
|----------|------|-------------|
| `guid` | string | Unique notification identifier |
| `type` | string | Notification type (info, warning, error) |
| `message` | string | Notification message |
| `timestamp` | string | When the notification was generated |

**Example - Get Status Notifications:**
```bash
curl -X GET "http://192.168.2.1/api?fields=status" -b cookies.txt
```

**Example - Clear Notification:**
```bash
curl -X DELETE http://192.168.2.1/api/status/550e8400-e29b-41d4-a716-446655440000 \
  -b cookies.txt
```

---

### `users`

User account management.

**Endpoints:**
- `GET /api?fields=users` - List all users
- `GET /api?fields=users/{username}` - Get specific user
- `POST /api/users` - Create new user
- `PUT /api/users/{username}` - Update user
- `DELETE /api/users/{username}` - Delete user

**Key Properties:**
| Property | Type | Description |
|----------|------|-------------|
| `name` | string | Username (login name) |
| `password` | string | Password (write-only) |
| `permission` | string | Role: `admin`, `user`, `guest` |
| `firstName` | string | First name |
| `lastName` | string | Last name |
| `email` | string | Email address |
| `enabled` | boolean | Account enabled status |

**Example - Create User:**
```bash
curl -X POST http://192.168.2.1/api/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "engineer1",
    "password": "SecurePass123!",
    "permission": "user",
    "firstName": "John",
    "lastName": "Doe",
    "email": "john.doe@example.com",
    "enabled": true
  }' \
  -b cookies.txt
```

**Example - Update User:**
```bash
curl -X PUT http://192.168.2.1/api/users/engineer1 \
  -H "Content-Type: application/json" \
  -d '{"email": "new.email@example.com"}' \
  -b cookies.txt
```

---

### `customRoles`

Custom role definitions for fine-grained access control.

**Endpoints:**
- `GET /api?fields=customRoles` - List custom roles
- `POST /api/customRoles` - Create custom role
- `PUT /api/customRoles/{roleId}` - Update custom role
- `DELETE /api/customRoles/{roleId}` - Delete custom role

---

### `permissions`

Permission matrix for resources and operations.

**Endpoints:**
- `GET /api?fields=permissions` - Get permission definitions

---

### `autoReboot`

Automatic reboot scheduler configuration.

**Endpoints:**
- `GET /api?fields=autoReboot` - Get auto-reboot settings
- `PUT /api/autoReboot` - Update auto-reboot settings

**Key Properties:**
| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable scheduled reboots |
| `schedule` | object | Cron-style schedule definition |
| `timezone` | string | Timezone for scheduling |

**Example:**
```bash
curl -X PUT http://192.168.2.1/api/autoReboot \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "schedule": {
      "dayOfWeek": "sunday",
      "hour": 3,
      "minute": 0
    }
  }' \
  -b cookies.txt
```

---

### `brand`

White-label branding configuration.

**Endpoints:**
- `GET /api?fields=brand` - Get branding settings
- `PUT /api/brand` - Update branding

---

### `resetButton`

Physical reset button behavior configuration.

**Endpoints:**
- `GET /api?fields=resetButton` - Get reset button settings
- `PUT /api/resetButton` - Update reset button settings

---

## Network Resources

### `cellular`

Cellular connectivity configuration.

**Endpoints:**
- `GET /api?fields=cellular` - Get cellular configuration
- `PUT /api/cellular` - Update cellular configuration

**Key Properties:**
| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable cellular connection |
| `providerProfiles` | array | Carrier/APN configurations |
| `simProfiles` | array | SIM card configurations |
| `connectionMode` | string | Connection mode (always-on, on-demand) |

**Example - Configure Cellular:**
```bash
curl -X PUT http://192.168.2.1/api/cellular \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "providerProfiles": [{
      "_id": "550e8400-e29b-41d4-a716-446655440000",
      "profileName": "AT&T",
      "cellularMode": "4g-preferred",
      "dataCtx": {
        "apnString": "broadband",
        "contextIpMode": "AUTO",
        "authentication": {
          "type": "NONE"
        }
      }
    }],
    "simProfiles": [{
      "_id": "550e8400-e29b-41d4-a716-446655440001",
      "profileName": "Primary SIM",
      "providerProfileId": "550e8400-e29b-41d4-a716-446655440000",
      "simIccid": "ANY"
    }]
  }' \
  -b cookies.txt
```

---

### `wifi`

WiFi access point and client configuration.

**Endpoints:**
- `GET /api?fields=wifi` - Get WiFi configuration
- `PUT /api/wifi` - Update WiFi configuration

**Key Properties:**
| Property | Type | Description |
|----------|------|-------------|
| `ap` | object | Access point settings |
| `client` | object | WiFi client settings |
| `enabled` | boolean | Enable WiFi |

**Example - Configure WiFi AP:**
```bash
curl -X PUT http://192.168.2.1/api/wifi \
  -H "Content-Type: application/json" \
  -d '{
    "ap": {
      "enabled": true,
      "ssid": "MyGateway-WiFi",
      "security": {
        "mode": "WPA2",
        "passphrase": "SecurePassword123"
      },
      "channel": 6
    }
  }' \
  -b cookies.txt
```

---

### `dns`

DNS server configuration.

**Endpoints:**
- `GET /api?fields=dns` - Get DNS configuration
- `PUT /api/dns` - Update DNS configuration

**Example:**
```bash
curl -X PUT http://192.168.2.1/api/dns \
  -H "Content-Type: application/json" \
  -d '{
    "primary": "8.8.8.8",
    "secondary": "8.8.4.4"
  }' \
  -b cookies.txt
```

---

### `dhcp`

DHCP server configuration.

**Endpoints:**
- `GET /api?fields=dhcp` - Get DHCP configuration
- `PUT /api/dhcp` - Update DHCP configuration

**Key Properties:**
| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable DHCP server |
| `startIp` | string | DHCP pool start address |
| `endIp` | string | DHCP pool end address |
| `leaseTime` | integer | Lease time in seconds |
| `staticLeases` | array | Static DHCP reservations |

**Example:**
```bash
curl -X PUT http://192.168.2.1/api/dhcp \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "startIp": "192.168.2.100",
    "endIp": "192.168.2.200",
    "leaseTime": 86400
  }' \
  -b cookies.txt
```

---

### `ni`

Network interfaces configuration.

**Endpoints:**
- `GET /api?fields=ni` - Get all network interfaces
- `GET /api/ni/nis/{index}` - Get specific interface by index
- `PUT /api/ni/nis/{index}` - Update specific interface

**Key Properties:**
| Property | Type | Description |
|----------|------|-------------|
| `name` | string | Interface name (eth0, br0, etc.) |
| `nitype` | string | Interface type (ETHERNET, BRIDGE, etc.) |
| `type` | string | WAN or LAN |
| `ipv4` | object | IPv4 configuration |
| `ipv6` | object | IPv6 configuration |

**Example - Update BR0 Bridge:**
```bash
curl -X PUT http://192.168.2.1/api/ni/nis/6 \
  -H "Content-Type: application/json" \
  -d '{
    "ipv4": {
      "mode": "STATIC",
      "ip": "192.168.100.1",
      "mask": "255.255.255.0",
      "gateway": "",
      "dns1": "8.8.8.8"
    }
  }' \
  -b cookies.txt
```

---

### `routes`

Static routing configuration.

**Endpoints:**
- `GET /api?fields=routes` - Get static routes
- `PUT /api/routes` - Update static routes
- `POST /api/routes` - Add static route
- `DELETE /api/routes/{id}` - Delete static route

**Example:**
```bash
curl -X POST http://192.168.2.1/api/routes \
  -H "Content-Type: application/json" \
  -d '{
    "destination": "10.0.0.0",
    "netmask": "255.0.0.0",
    "gateway": "192.168.2.254",
    "interface": "eth0",
    "metric": 100
  }' \
  -b cookies.txt
```

---

### `firewall`

Firewall rules configuration.

**Endpoints:**
- `GET /api?fields=firewall` - Get firewall rules
- `PUT /api/firewall` - Update firewall rules

See [NAT & Firewall Schemas](NAT-FIREWALL-SCHEMAS.md) for detailed documentation.

---

### `nat`

NAT and port forwarding configuration.

**Endpoints:**
- `GET /api?fields=nat` - Get NAT configuration
- `PUT /api/nat` - Update NAT configuration

See [NAT & Firewall Schemas](NAT-FIREWALL-SCHEMAS.md) for detailed documentation.

---

### `wanmngr`

WAN failover and load balancing configuration.

**Endpoints:**
- `GET /api?fields=wanmngr` - Get WAN manager configuration
- `PUT /api/wanmngr` - Update WAN manager configuration

**Key Properties:**
| Property | Type | Description |
|----------|------|-------------|
| `mode` | string | FAILOVER or LOADBALANCE |
| `wans` | array | WAN interface configurations with priorities |

See [WAN Failover Guide](WAN-FAILOVER-GUIDE.md) for detailed documentation.

**Example:**
```bash
curl -X PUT http://192.168.2.1/api/wanmngr \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "FAILOVER",
    "wans": [
      {"interface": "eth0", "priority": 1},
      {"interface": "ppp0", "priority": 2}
    ]
  }' \
  -b cookies.txt
```

---

### `waninfo`

Real-time WAN interface status (read-only).

**Endpoints:**
- `GET /api?fields=waninfo` - Get current WAN status

---

### `ipPassthrough`

IP passthrough/bridge mode configuration.

**Endpoints:**
- `GET /api?fields=ipPassthrough` - Get IP passthrough settings
- `PUT /api/ipPassthrough` - Update IP passthrough settings

---

### `ddns`

Dynamic DNS configuration.

**Endpoints:**
- `GET /api?fields=ddns` - Get DDNS configuration
- `PUT /api/ddns` - Update DDNS configuration

**Example:**
```bash
curl -X PUT http://192.168.2.1/api/ddns \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "provider": "dyndns",
    "hostname": "mygateway.dyndns.org",
    "username": "user",
    "password": "pass"
  }' \
  -b cookies.txt
```

---

### `lldp`

Link Layer Discovery Protocol configuration.

**Endpoints:**
- `GET /api?fields=lldp` - Get LLDP settings
- `PUT /api/lldp` - Update LLDP settings

---

### `mdns`

Multicast DNS (Bonjour/Avahi) configuration.

**Endpoints:**
- `GET /api?fields=mdns` - Get mDNS settings
- `PUT /api/mdns` - Update mDNS settings

---

## Cellular Resources

### `backOffTimers`

Carrier-specific cellular reconnection back-off timers.

**Endpoints:**
- `GET /api?fields=backOffTimers` - Get back-off timer configuration
- `PUT /api/backOffTimers` - Update back-off timers

---

### `cellTimeSync`

Cellular network time synchronization.

**Endpoints:**
- `GET /api?fields=cellTimeSync` - Get cellular time sync settings
- `PUT /api/cellTimeSync` - Update cellular time sync settings

---

## VPN Resources

### `ovpnTunnels`

OpenVPN tunnel configuration.

**Endpoints:**
- `GET /api?fields=ovpnTunnels` - Get OpenVPN tunnels
- `PUT /api/ovpnTunnels` - Update OpenVPN tunnels
- `POST /api/ovpnTunnels` - Create OpenVPN tunnel
- `DELETE /api/ovpnTunnels/{id}` - Delete OpenVPN tunnel

See [VPN Schemas](VPN-SCHEMAS.md) for detailed documentation.

---

### `ipsecTunnels`

IPsec tunnel configuration.

**Endpoints:**
- `GET /api?fields=ipsecTunnels` - Get IPsec tunnels
- `PUT /api/ipsecTunnels` - Update IPsec tunnels

See [VPN Schemas](VPN-SCHEMAS.md) for detailed documentation.

---

### `greTunnels`

GRE tunnel configuration.

**Endpoints:**
- `GET /api?fields=greTunnels` - Get GRE tunnels
- `PUT /api/greTunnels` - Update GRE tunnels

See [VPN Schemas](VPN-SCHEMAS.md) for detailed documentation.

---

## IoT Protocol Resources

### `loraNetwork`

LoRa network server configuration (200+ properties).

**Endpoints:**
- `GET /api?fields=loraNetwork` - Get LoRa network configuration
- `PUT /api/loraNetwork` - Update LoRa network configuration

**Key Properties:**
| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable LoRa |
| `mode` | string | NETWORK_SERVER, PACKET_FORWARDER, BASIC_STATION |
| `network` | object | Network server settings |
| `mqtt` | object | MQTT integration settings |
| `packetForwarder` | object | Packet forwarder settings |

See [LoRa Network Schema](LORA-NETWORK-SCHEMA.md) for comprehensive documentation.

**Example:**
```bash
curl -X PUT http://192.168.2.1/api/loraNetwork \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "mode": "NETWORK_SERVER",
    "network": {
      "name": "My LoRa Network"
    }
  }' \
  -b cookies.txt
```

---

### `lora`

LoRa gateway and device management.

**Endpoints:**
- `GET /api/lora/gateways` - List gateways
- `GET /api/lora/devices` - List devices
- `DELETE /api/lora/gateways/{gwEui}` - Delete gateway
- `POST /api/lora/restart` - Restart LoRa service

See [LoRa Operations Workflow](LORA-OPERATIONS-WORKFLOW.md) for FOTA and multicast operations.

---

### `scada`

SCADA protocol configuration (BACnet, Modbus).

**Endpoints:**
- `GET /api?fields=scada` - Get SCADA configuration
- `PUT /api/scada` - Update SCADA configuration

---

### `mqttBroker`

MQTT broker configuration.

**Endpoints:**
- `GET /api?fields=mqttBroker` - Get MQTT broker settings
- `PUT /api/mqttBroker` - Update MQTT broker settings

---

## Device Management Resources

### `docker`

Docker container management.

**Endpoints:**
- `GET /api?fields=docker` - Get Docker configuration
- `PUT /api/docker` - Update Docker configuration

---

### `customApps`

Custom application management.

**Endpoints:**
- `GET /api?fields=customApps` - Get custom apps
- `POST /api/customApps` - Upload custom app

---

### `customAppsConfig`

Custom application settings.

**Endpoints:**
- `GET /api?fields=customAppsConfig` - Get custom app configuration
- `PUT /api/customAppsConfig` - Update custom app configuration

---

### `packages`

System package management.

**Endpoints:**
- `GET /api?fields=packages` - Get installed packages

---

### `licensing`

License management.

**Endpoints:**
- `GET /api?fields=licensing` - Get license information

---

## Communication Resources

### `gps`

GPS configuration and location data.

**Endpoints:**
- `GET /api?fields=gps` - Get GPS configuration and data
- `PUT /api/gps` - Update GPS configuration

**Key Properties:**
| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable GPS |
| `latitude` | number | Current latitude (read-only) |
| `longitude` | number | Current longitude (read-only) |
| `altitude` | number | Current altitude (read-only) |
| `fix` | boolean | GPS fix status (read-only) |

**Example:**
```bash
curl -X GET "http://192.168.2.1/api?fields=gps" -b cookies.txt
```

---

### `serial`

Serial port configuration.

**Endpoints:**
- `GET /api?fields=serial` - Get serial port configuration
- `PUT /api/serial` - Update serial port configuration

**Key Properties:**
| Property | Type | Description |
|----------|------|-------------|
| `ports` | array | Serial port configurations |
| `baudRate` | integer | Baud rate (9600, 115200, etc.) |
| `dataBits` | integer | Data bits (7, 8) |
| `parity` | string | Parity (none, odd, even) |
| `stopBits` | integer | Stop bits (1, 2) |

---

### `sms`

SMS messaging configuration and commands.

See **[SMS Schema](SMS-SCHEMA.md)** for complete documentation.

**Endpoints:**
- `GET /api?fields=sms` - Get SMS configuration
- `PUT /api/sms` - Update SMS configuration
- `GET /api?fields=sms/inbox` - Get received messages
- `GET /api?fields=sms/outbox` - Get sent messages
- `POST /api/sms/outbox` - Send SMS message
- `DELETE /api/sms/inbox/{guid}` - Delete received message
- `DELETE /api/sms/outbox/{guid}` - Delete sent message

**Key Properties:**
| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `enabled` | boolean | `false` | Enable SMS service |
| `sentLimit` | integer | `1000` | Number of sent messages to keep (0-1000) |
| `receivedLimit` | integer | `1000` | Number of received messages to keep (0-1000) |
| `resendLimit` | integer | `0` | Retry attempts for failed messages (0-10) |
| `smsCommands` | object | - | SMS commands configuration |

**Example - Configure SMS:**
```bash
curl -X PUT http://192.168.2.1/api/sms \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "sentLimit": 500,
    "receivedLimit": 500,
    "smsCommands": {
      "enabled": true,
      "rebootEnabled": true,
      "pingEnabled": true,
      "password": {
        "enabled": true,
        "useCustomPassword": false
      },
      "whitelist": {
        "enabled": true,
        "numbers": ["+1234567890"]
      }
    }
  }' \
  -b cookies.txt
```

**Example - Send SMS:**
```bash
curl -X POST http://192.168.2.1/api/sms/outbox \
  -H "Content-Type: application/json" \
  -d '{
    "recipients": ["+1234567890"],
    "message": "Alert from gateway"
  }' \
  -b cookies.txt
```

**Example - Get Inbox:**
```bash
curl -X GET "http://192.168.2.1/api?fields=sms/inbox" -b cookies.txt
```

---

### `bluetooth`

Bluetooth configuration.

**Endpoints:**
- `GET /api?fields=bluetooth` - Get Bluetooth configuration
- `PUT /api/bluetooth` - Update Bluetooth configuration

---

### `bluetoothLowEnergy`

Bluetooth Low Energy (BLE) configuration.

**Endpoints:**
- `GET /api?fields=bluetoothLowEnergy` - Get BLE configuration
- `PUT /api/bluetoothLowEnergy` - Update BLE configuration

---

## Security Resources

### `certificate`

Device certificate management.

**Endpoints:**
- `GET /api?fields=certificate` - Get device certificates
- `POST /api/certificate` - Upload certificate
- `DELETE /api/certificate/{id}` - Delete certificate

See [Security Schemas](SECURITY-SCHEMAS.md) for detailed documentation.

---

### `cacertificates`

CA certificate management.

**Endpoints:**
- `GET /api?fields=cacertificates` - Get CA certificates
- `POST /api/cacertificates` - Upload CA certificate
- `DELETE /api/cacertificates/{id}` - Delete CA certificate

---

### `secureProtocols`

TLS/SSL protocol configuration.

**Endpoints:**
- `GET /api?fields=secureProtocols` - Get secure protocol settings
- `PUT /api/secureProtocols` - Update secure protocol settings

---

### `remoteAccess`

Remote access security configuration (HTTP, HTTPS, SSH, SNMP).

**Endpoints:**
- `GET /api?fields=remoteAccess` - Get remote access settings
- `PUT /api/remoteAccess` - Update remote access settings

---

### `radius`

RADIUS authentication configuration.

**Endpoints:**
- `GET /api?fields=radius` - Get RADIUS settings
- `PUT /api/radius` - Update RADIUS settings

---

### `trustedIp`

Trusted IP address configuration for access control.

**Endpoints:**
- `GET /api?fields=trustedIp` - Get trusted IPs
- `PUT /api/trustedIp` - Update trusted IPs

---

### `bootloader`

Bootloader security settings.

**Endpoints:**
- `GET /api?fields=bootloader` - Get bootloader settings
- `PUT /api/bootloader` - Update bootloader settings

---

### `passwordComplexityRules`

Password complexity requirements.

**Endpoints:**
- `GET /api?fields=passwordComplexityRules` - Get password rules
- `PUT /api/passwordComplexityRules` - Update password rules

---

## Monitoring Resources

### `stats`

System statistics and metrics.

**Endpoints:**
- `GET /api?fields=stats` - Get all statistics
- `GET /api?fields=stats/dashboard` - Comprehensive system overview
- `GET /api?fields=stats/cpu` - CPU usage
- `GET /api?fields=stats/memory` - Memory usage
- `GET /api?fields=stats/radio` - Cellular radio statistics
- `GET /api?fields=stats/iface` - Network interface statistics
- `GET /api?fields=stats/lora` - LoRa gateway statistics

See [Stats Endpoints Discovery](STATS-ENDPOINTS-DISCOVERY.md) for all 33+ stats categories.

**Example - Get Dashboard:**
```bash
curl -X GET "http://192.168.2.1/api?fields=stats/dashboard" -b cookies.txt
```

---

### `eventlog`

Event log entries.

**Endpoints:**
- `GET /api?fields=eventlog` - Get event log

---

### `syslog`

Syslog configuration.

**Endpoints:**
- `GET /api?fields=syslog` - Get syslog configuration
- `PUT /api/syslog` - Update syslog configuration

---

### `alert`

Alert and notification configuration.

**Endpoints:**
- `GET /api?fields=alert` - Get alert configuration
- `PUT /api/alert` - Update alert configuration

See [Alert Schema](ALERT-SCHEMA.md) for detailed documentation.

---

### `snmp`

SNMP agent and trap configuration.

**Endpoints:**
- `GET /api?fields=snmp` - Get SNMP configuration
- `PUT /api/snmp` - Update SNMP configuration

See [SNMP Schema](SNMP-SCHEMA.md) for detailed documentation.

---

## Notification Resources

### `smtp`

SMTP email configuration for notifications.

**Endpoints:**
- `GET /api?fields=smtp` - Get SMTP settings
- `PUT /api/smtp` - Update SMTP settings

**Example:**
```bash
curl -X PUT http://192.168.2.1/api/smtp \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "server": "smtp.example.com",
    "port": 587,
    "security": "STARTTLS",
    "username": "alerts@example.com",
    "password": "password",
    "fromAddress": "alerts@example.com"
  }' \
  -b cookies.txt
```

---

### `sntp`

NTP time synchronization configuration.

**Endpoints:**
- `GET /api?fields=sntp` - Get NTP settings
- `PUT /api/sntp` - Update NTP settings

---

### `notificationEventGroup`

Event notification group configuration.

**Endpoints:**
- `GET /api?fields=notificationEventGroup` - Get notification groups
- `PUT /api/notificationEventGroup` - Update notification groups

---

### `remoteMgmt`

Multi-Tech cloud management configuration.

**Endpoints:**
- `GET /api?fields=remoteMgmt` - Get cloud management settings
- `PUT /api/remoteMgmt` - Update cloud management settings

---

## Query Tips

### Batch Queries

Request multiple resources in a single call for efficiency:

```bash
GET /api?fields=system,cellular,wifi,gps
```

### Nested Field Access

Access specific nested fields to reduce response size:

```bash
GET /api?fields=system/capabilities
GET /api?fields=system/firmware/version
GET /api?fields=cellular/providerProfiles
```

### Default Values

Get factory default values for a resource:

```bash
GET /api?fields=cellular&default=true
```

### Monitoring Without Session Timeout

Use `inactivity=true` for monitoring endpoints to prevent session timeout:

```bash
GET /api?fields=stats/dashboard&inactivity=true
```

---

## Related Documentation

- [Command Endpoints](COMMAND-ENDPOINTS.md) - Device commands reference
- [API Quick Reference](API-QUICK-REFERENCE.md) - Concise command reference
- [API README](API-README.md) - Getting started guide
- [Schema Documentation](SCHEMA-COVERAGE-ANALYSIS.md) - Detailed schema documentation

---

**Document Version**: 1.0.0  
**Last Updated**: February 2026
