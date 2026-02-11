---
layout: default
title: "Advanced Features Schemas Reference"
permalink: /v2/mpower-api-docs/ADVANCED-FEATURES-SCHEMAS/
doc_version: v2
---

# Advanced Features Schemas Reference

Complete schema documentation for advanced features extracted from mtsDeviceAPI source code.

**Source:** `/home/jreiss/Workspace/aep/mtsDeviceAPI/json/strict/`

---

## Table of Contents

- [Filters (Firewall Rules)](#filters-firewall-rules)
- [GRE Tunnels](#gre-tunnels)
- [SCADA](#scada)
- [MQTT Broker](#mqtt-broker)
- [RADIUS](#radius)
- [Secure Protocols](#secure-protocols)
- [Trusted IP](#trusted-ip)
- [Notification Event Groups](#notification-event-groups)

---

## Filters (Firewall Rules)

Advanced firewall filter rules.

**Source:** `filters.schema.json`

**Endpoints:**
- `GET /api?fields=filters` - Get firewall filters
- `PUT /api/filters` - Update filters
- `POST /api/filters` - Create filter
- `DELETE /api/filters/{name}` - Delete filter

### Schema

```json
{
  "name": "block-ssh-wan",
  "description": "Block SSH from WAN",
  "enabled": true,
  "chain": "INPUT",
  "target": "DROP",
  "protocol": "TCP",
  "srcInterface": "WAN",
  "srcSpecInterface": "",
  "srcAddr": "ANY",
  "srcMask": "",
  "srcPort": "ANY",
  "srcMac": "ANY",
  "dstInterface": "ANY",
  "dstSpecInterface": "",
  "dstAddr": "ANY",
  "dstMask": "",
  "dstPort": "22"
}
```

### Key Properties

| Property | Type | Description |
|----------|------|-------------|
| `name` | string | Unique rule name (required) |
| `enabled` | boolean | Enable rule (required) |
| `chain` | string | `INPUT`, `OUTPUT`, `FORWARD` (required) |
| `target` | string | `ACCEPT`, `REJECT`, `LOG`, `DROP` |
| `protocol` | string | `TCP`, `UDP`, `TCP/UDP`, `ANY` |

### Interface Values

| Value | Description |
|-------|-------------|
| `ANY` | Any interface |
| `WAN` | WAN interface |
| `LAN` | LAN interface |
| `CELLULAR` | Cellular interface |
| `WIFI` | WiFi interface |
| `WIFI_AP` | WiFi Access Point |
| `ETHERNET` | Ethernet |
| `OPENVPN` | OpenVPN tunnel |

### Address and Port Formats

**IP Address:**
- `ANY` or empty - any address
- `10.0.0.1` - specific IP

**IP Mask:**
- `255.255.255.0` - dotted notation
- `24` - CIDR notation

**Port Expression:**
- `5000` - single port
- `6000:6002` - port range
- `5000,6000-6002,8000:8010` - mixed

### Special Rules

When `srcInterface` or `dstInterface` is `OPENVPN`, the corresponding `srcSpecInterface` or `dstSpecInterface` must specify the tunnel name.

---

## GRE Tunnels

Generic Routing Encapsulation tunnel configuration.

**Source:** `greTunnels.schema.json`

**Endpoints:**
- `GET /api?fields=greTunnels` - Get GRE tunnels
- `PUT /api/greTunnels` - Update tunnels
- `POST /api/greTunnels` - Create tunnel
- `DELETE /api/greTunnels/{name}` - Delete tunnel

### Schema

```json
{
  "enabled": true,
  "name": "gre-tunnel-1",
  "description": "Remote site tunnel",
  "remoteIp": "203.0.113.10",
  "ttl": 64,
  "checkPeriod": 60,
  "tunnelIp": "10.0.0.1",
  "tunnelMask": "255.255.255.252",
  "routes": [
    {
      "ip": "192.168.100.0",
      "mask": "24"
    }
  ]
}
```

### Key Properties

| Property | Type | Description | Constraints |
|----------|------|-------------|-------------|
| `enabled` | boolean | Enable tunnel | required |
| `name` | string | Tunnel name | max 15 chars, alphanumeric/underscore/dash |
| `remoteIp` | string | Remote peer IP or hostname | required |
| `ttl` | integer | Time to live | 0-255 (0 = inherit) |
| `checkPeriod` | integer | DNS resolve interval (min) | 1-120 |
| `tunnelIp` | string | Local tunnel IP | required |
| `tunnelMask` | string | Tunnel subnet mask | required |

### Route Object

```json
{
  "ip": "192.168.100.0",
  "mask": "24"
}
```

---

## SCADA

Supervisory Control and Data Acquisition settings.

**Source:** `scada.schema.json`

**Endpoints:**
- `GET /api?fields=scada` - Get SCADA configuration
- `PUT /api/scada` - Update SCADA configuration
- Various sensor/BACnet endpoints

### Schema

```json
{
  "sensors": {
    "defaultTtl": 86400
  },
  "bacnetOut": {
    "datalink": {
      "ip": {
        "enabled": false,
        "port": 47808,
        "interface": "eth0"
      }
    },
    "vendorIdentifier": 0,
    "vendorName": "",
    "objectIdentifier": 0,
    "objectName": "",
    "modelName": "",
    "description": "",
    "location": "",
    "apduTimeout": 3000,
    "apduRetries": 3
  }
}
```

### BACnet Configuration

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `vendorIdentifier` | integer | ASHRAE vendor ID | - |
| `vendorName` | string | Manufacturer name | - |
| `objectIdentifier` | integer | Unique BACnet object ID | - |
| `objectName` | string | Unique object name | - |
| `modelName` | string | Model name (max 32) | - |
| `apduTimeout` | integer | Request timeout (ms) | 3000 |
| `apduRetries` | integer | Retry count | 3 |

### BACnet/IP Settings

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `enabled` | boolean | Enable BACnet/IP | false |
| `port` | integer | UDP port | 47808 (1-65535) |
| `interface` | string | Network interface | eth0 |

---

## MQTT Broker

MQTT broker bridge configuration.

**Source:** `mqttBroker.schema.json`

**Endpoints:**
- `GET /api?fields=mqttBroker` - Get MQTT broker configuration
- `PUT /api/mqttBroker` - Update MQTT broker configuration

### Schema

```json
{
  "bridge": {
    "enabled": false,
    "serverAddress": "mqtt.example.com",
    "serverPort": 1883,
    "secondaryAddress": "",
    "secondaryPort": 1883,
    "clientId": "device-client-id",
    "tlsEnabled": false,
    "verifyHostname": false,
    "authMethod": "noAuth",
    "username": "",
    "password": "",
    "certificate": "",
    "privateKey": "",
    "identity": "",
    "psk": "",
    "topics": [],
    "cleanSession": false,
    "notifications": true,
    "tryPrivate": true,
    "protocolVersion": "mqttv31",
    "tlsVersion": "tlsv1.2"
  }
}
```

### Authentication Methods

| Method | Required Fields |
|--------|-----------------|
| `noAuth` | None |
| `userPass` | `username`, `password` |
| `deviceCert` | Uses device certificate |
| `userCert` | `certificate`, `privateKey` |
| `psk` | `identity`, `psk` |

### Topic Configuration

```json
{
  "pattern": "sensors/#",
  "localPrefix": "local/",
  "remotePrefix": "remote/",
  "direction": "out",
  "qosLevel": 0
}
```

| Property | Type | Description |
|----------|------|-------------|
| `pattern` | string | Topic pattern to match |
| `localPrefix` | string | Prefix for local broker |
| `remotePrefix` | string | Prefix for remote broker |
| `direction` | string | `in`, `out`, or `both` |
| `qosLevel` | integer | QoS level (0, 1, or 2) |

### Protocol Versions

- `mqttv31` - MQTT 3.1
- `mqttv311` - MQTT 3.1.1

### TLS Versions

- `tlsv1.1`
- `tlsv1.2`
- `tlsv1.3`

---

## RADIUS

RADIUS authentication configuration.

**Source:** `radius.schema.json`

**Endpoints:**
- `GET /api?fields=radius` - Get RADIUS configuration
- `PUT /api/radius` - Update RADIUS configuration

### Schema

```json
{
  "authEnabled": false,
  "acctEnabled": false,
  "host": "",
  "authPort": 1812,
  "acctPort": 1813,
  "secondaryHost": "",
  "secondaryAuthPort": 1812,
  "secondaryAcctPort": 1813,
  "secret": "",
  "authType": "PAP",
  "timeout": 2,
  "retries": 3,
  "checkServer": true,
  "anonymousEnabled": false,
  "anonymousId": "anonymous"
}
```

### Key Properties

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `authEnabled` | boolean | Enable RADIUS auth | false |
| `acctEnabled` | boolean | Enable RADIUS accounting | false |
| `host` | string | Primary RADIUS server | - |
| `authPort` | integer | Authentication port | 1812 |
| `acctPort` | integer | Accounting port | 1813 |
| `secret` | string | Shared secret | - |
| `timeout` | integer | Retry timeout (seconds) | 2 |
| `retries` | integer | Max retries | 3 |

### Authentication Types

| Type | Description |
|------|-------------|
| `PAP` | Password Authentication Protocol |
| `EAP-TTLSv0/PAP` | EAP-TTLS with PAP |
| `EAP-PEAPv0/MSCHAPv2` | PEAP with MSCHAPv2 |

---

## Secure Protocols

TLS/SSL and SSH cipher configuration.

**Source:** `secureProtocols.schema.json`

**Endpoints:**
- `GET /api?fields=secureProtocols` - Get secure protocol configurations
- `PUT /api/secureProtocols` - Update secure protocols

### Schema (OpenSSL Type)

```json
{
  "name": "webserver",
  "type": "openssl",
  "protocol": {
    "tls1_2": true,
    "tls1_3": true
  },
  "cipherSuite": "ECDHE-RSA-AES256-GCM-SHA384:...",
  "cipherSuiteTls13": "TLS_AES_256_GCM_SHA384:...",
  "cipherSuiteDeprecated": [],
  "cipherSuiteAvailable": "...",
  "cipherSuiteAvailableTls13": "...",
  "client": {
    "verify": false
  }
}
```

### Schema (OpenSSH Type)

```json
{
  "name": "ssh",
  "type": "openssh",
  "cipher": "aes256-gcm@openssh.com,chacha20-poly1305@openssh.com",
  "cipherDeprecated": [],
  "cipherAvailable": "...",
  "mac": "hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com",
  "macDeprecated": [],
  "macAvailable": "..."
}
```

### Protocol Settings

At least one of `tls1_2` or `tls1_3` must be enabled.

---

## Trusted IP

IP whitelist/blacklist filtering.

**Source:** `trustedIp.schema.json`

**Endpoints:**
- `GET /api?fields=trustedIp` - Get trusted IP configuration
- `PUT /api/trustedIp` - Update trusted IP configuration

### Schema

```json
{
  "enabled": false,
  "mode": "whitelist",
  "rules": [
    {
      "name": "office-network",
      "ipMode": "subnet",
      "subnetSrcAddr": "192.168.1.0",
      "subnetSrcMask": "24",
      "rangeSrcAddrStart": "",
      "rangeSrcAddrEnd": "",
      "dstPort": "ANY",
      "protocol": "ANY"
    }
  ]
}
```

### Modes

| Mode | Description |
|------|-------------|
| `whitelist` | Allow only listed IPs |
| `blacklist` | Block listed IPs |

### Rule IP Modes

**Subnet Mode:**
```json
{
  "ipMode": "subnet",
  "subnetSrcAddr": "192.168.1.0",
  "subnetSrcMask": "24"
}
```

**Range Mode:**
```json
{
  "ipMode": "range",
  "rangeSrcAddrStart": "192.168.1.10",
  "rangeSrcAddrEnd": "192.168.1.50"
}
```

### Port and Protocol

| Property | Values |
|----------|--------|
| `dstPort` | `ANY`, single port, or range (e.g., `80`, `8000:8080`) |
| `protocol` | `TCP`, `UDP`, `TCP/UDP`, `ANY` |

Note: If `dstPort` is not `ANY`, `protocol` cannot be `ANY`.

---

## Notification Event Groups

Event notification configuration.

**Source:** `notificationEventGroup.schema.json`

**Endpoints:**
- `GET /api?fields=notificationEventGroup` - Get notification configuration
- `PUT /api/notificationEventGroup` - Update notification configuration

### Schema

```json
{
  "eventsGroups": [
    {
      "eventGroup": "Authentication",
      "enabled": true,
      "email": true,
      "sms": false,
      "snmp": true,
      "notifyGroup": "admin-group"
    }
  ],
  "groups": [
    {
      "name": "admin-group",
      "phones": [
        {
          "name": "Admin",
          "phone": "+15551234567"
        }
      ],
      "emails": [
        {
          "name": "Admin",
          "email": "admin@example.com"
        }
      ]
    }
  ]
}
```

### Event Groups

| Group | Description |
|-------|-------------|
| `Authentication` | Login/logout events |
| `Chassis` | Hardware events |
| `Configuration` | Config change events |
| `Link` | Network link events |
| `Security` | Security events |
| `Wan` | WAN connection events |

### Notification Types

| Type | Description |
|------|-------------|
| `email` | Send email notification |
| `sms` | Send SMS notification |
| `snmp` | Send SNMP trap |

---

**Last Updated:** February 10, 2026
