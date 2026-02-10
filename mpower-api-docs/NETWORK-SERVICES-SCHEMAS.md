# Network Services Schemas Reference

Complete schema documentation for network services extracted from mtsDeviceAPI source code.

**Source:** `/home/jreiss/Workspace/aep/mtsDeviceAPI/json/strict/`

---

## Table of Contents

- [SNMP](#snmp)
- [SNTP (NTP)](#sntp-ntp)
- [SMTP](#smtp)
- [Network Interfaces (NI)](#network-interfaces-ni)

---

## SNMP

Simple Network Management Protocol configuration.

**Source:** `snmp.schema.json`

**Endpoints:**
- `GET /api?fields=snmp` - Get SNMP configuration
- `PUT /api/snmp` - Update SNMP configuration

### Schema

```json
{
  "__v": 2,
  "server": {
    "enabled": false,
    "sysName": "",
    "sysLocation": "",
    "sysContact": "",
    "users": [],
    "allowedAddresses": []
  },
  "traps": {
    "enabled": false,
    "engineId": "0x0102030405",
    "engineIdDefault": "0x...",
    "receivers": []
  }
}
```

### Server Properties

| Property | Type | Required | Constraints | Description |
|----------|------|----------|-------------|-------------|
| `server.enabled` | boolean | Yes | - | Enable SNMP server |
| `server.sysName` | string | Yes | max 255, printable ASCII | SNMP sysName object |
| `server.sysLocation` | string | Yes | max 255, printable ASCII | SNMP sysLocation object |
| `server.sysContact` | string | Yes | max 255, printable ASCII | SNMP sysContact object |
| `server.users` | array | Yes | max 5 items | SNMP users |
| `server.allowedAddresses` | array | Yes | - | Allowed IP addresses (v1/v2c) |

### SNMP User Object

```json
{
  "enabled": true,
  "label": "user1",
  "version": "v3",
  "username": "admin",
  "authProto": "SHA1",
  "authPass": "authpassword",
  "privProto": "AES-128",
  "privPass": "privpassword",
  "community": ""
}
```

| Property | Type | Constraints | Description |
|----------|------|-------------|-------------|
| `enabled` | boolean | - | Enable user |
| `label` | string | 1-31 chars | Unique configuration name |
| `version` | string | `v1`, `v2c`, `v1v2c`, `v3` | SNMP version |
| `username` | string | max 250, min 5 for v3 | SNMPv3 username |
| `authProto` | string | ``, `MD5`, `SHA1` | Authentication protocol |
| `authPass` | string | max 250, min 8 if auth enabled, no `"` | Auth password |
| `privProto` | string | ``, `DES`, `AES-128` | Encryption protocol |
| `privPass` | string | max 250, 8-128 if priv enabled, no `"` | Encryption password |
| `community` | string | max 250, 1-128 if v1/v2c, no `"` | Community string |

### Traps Configuration

| Property | Type | Required | Constraints | Description |
|----------|------|----------|-------------|-------------|
| `traps.enabled` | boolean | Yes | - | Enable SNMP traps |
| `traps.engineId` | string | Yes | hex, 5-32 octets when enabled | Authoritative Engine ID |
| `traps.engineIdDefault` | string | Yes | - | Default Engine ID (read-only) |
| `traps.receivers` | array | Yes | max 5 | Trap destinations |

### Trap Receiver Object

```json
{
  "enabled": true,
  "label": "trap1",
  "address": "192.168.2.100",
  "version": "v3",
  "username": "trapuser",
  "authProto": "SHA1",
  "authPass": "authpassword",
  "privProto": "AES-128",
  "privPass": "privpassword",
  "community": ""
}
```

### Example - Configure SNMPv3

```bash
curl -X PUT http://192.168.2.1/api/snmp \
  -H "Content-Type: application/json" \
  -d '{
    "__v": 2,
    "server": {
      "enabled": true,
      "sysName": "Gateway-01",
      "sysLocation": "Building A, Floor 2",
      "sysContact": "admin@example.com",
      "users": [
        {
          "enabled": true,
          "label": "admin-user",
          "version": "v3",
          "username": "snmpadmin",
          "authProto": "SHA1",
          "authPass": "AuthPassword123",
          "privProto": "AES-128",
          "privPass": "PrivPassword123",
          "community": ""
        }
      ],
      "allowedAddresses": []
    },
    "traps": {
      "enabled": true,
      "engineId": "0x8000000001020304",
      "receivers": [
        {
          "enabled": true,
          "label": "nms-server",
          "address": "192.168.2.200",
          "version": "v3",
          "username": "trapuser",
          "authProto": "SHA1",
          "authPass": "TrapAuth123",
          "privProto": "AES-128",
          "privPass": "TrapPriv123",
          "community": ""
        }
      ]
    }
  }' \
  -b cookies.txt
```

### Example - Configure SNMPv2c

```bash
curl -X PUT http://192.168.2.1/api/snmp \
  -H "Content-Type: application/json" \
  -d '{
    "__v": 2,
    "server": {
      "enabled": true,
      "sysName": "Gateway-01",
      "sysLocation": "Data Center",
      "sysContact": "noc@example.com",
      "users": [
        {
          "enabled": true,
          "label": "readonly",
          "version": "v2c",
          "username": "",
          "authProto": "",
          "authPass": "",
          "privProto": "",
          "privPass": "",
          "community": "public"
        }
      ],
      "allowedAddresses": [
        { "address": "192.168.2.0", "prefix": 24 }
      ]
    },
    "traps": {
      "enabled": false,
      "engineId": "",
      "receivers": []
    }
  }' \
  -b cookies.txt
```

---

## SNTP (NTP)

Network Time Protocol client configuration.

**Source:** `sntp.schema.json`

**Endpoints:**
- `GET /api?fields=sntp` - Get NTP configuration
- `PUT /api/sntp` - Update NTP configuration

### Schema

```json
{
  "enabled": false,
  "minpoll": 6,
  "maxpoll": 10,
  "pool": {
    "enabled": true,
    "server": "north-america.pool.ntp.org"
  },
  "servers": [],
  "timeZone": "UTC"
}
```

### Properties

| Property | Type | Required | Default | Constraints | Description |
|----------|------|----------|---------|-------------|-------------|
| `enabled` | boolean | Yes | `false` | - | Enable NTP client |
| `minpoll` | integer | Yes | - | 3-17 | Min polling interval (2^n seconds) |
| `maxpoll` | integer | Yes | - | 3-17 | Max polling interval (2^n seconds) |
| `pool.enabled` | boolean | Yes | - | - | Use NTP pool |
| `pool.server` | string | Yes | `north-america.pool.ntp.org` | hostname/IP, required if pool enabled | Pool server address |
| `servers` | array | Yes | - | - | List of NTP servers (used if pool disabled) |
| `timeZone` | string | Yes | `UTC` | max 250 | Timezone (zoneinfo path) |

### Polling Intervals

The polling interval is calculated as 2^n seconds:

| Value | Interval |
|-------|----------|
| 3 | 8 seconds |
| 6 | 64 seconds |
| 10 | ~17 minutes |
| 14 | ~4.5 hours |
| 17 | ~36 hours |

### Business Rules

- If `enabled` is true, either `pool.enabled` must be true OR `servers` must have at least one entry
- If `pool.enabled` is true, `pool.server` must not be empty

### Example - Configure NTP with Pool

```bash
curl -X PUT http://192.168.2.1/api/sntp \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "minpoll": 6,
    "maxpoll": 10,
    "pool": {
      "enabled": true,
      "server": "pool.ntp.org"
    },
    "servers": [],
    "timeZone": "America/Chicago"
  }' \
  -b cookies.txt
```

### Example - Configure NTP with Specific Servers

```bash
curl -X PUT http://192.168.2.1/api/sntp \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "minpoll": 6,
    "maxpoll": 10,
    "pool": {
      "enabled": false,
      "server": ""
    },
    "servers": [
      "time.google.com",
      "time.cloudflare.com",
      "time.windows.com"
    ],
    "timeZone": "America/New_York"
  }' \
  -b cookies.txt
```

---

## SMTP

Simple Mail Transfer Protocol configuration for email notifications.

**Source:** `smtp.schema.json`

**Endpoints:**
- `GET /api?fields=smtp` - Get SMTP configuration
- `PUT /api/smtp` - Update SMTP configuration

### Schema

```json
{
  "enabled": false,
  "serverAddress": "",
  "serverPort": 587,
  "tls": {
    "enabled": true,
    "startTls": true,
    "certCheck": false
  },
  "authEnabled": false,
  "username": "",
  "password": "",
  "sourceEmailAddress": "",
  "maxNumberOfEntries": 50
}
```

### Properties

| Property | Type | Required | Default | Constraints | Description |
|----------|------|----------|---------|-------------|-------------|
| `enabled` | boolean | Yes | `false` | - | Enable SMTP |
| `serverAddress` | string | Yes | - | hostname/IP, required if enabled | SMTP server address |
| `serverPort` | string/int | Yes | - | 0-65535 | SMTP server port |
| `tls.enabled` | boolean | Yes | `true` | - | Enable TLS |
| `tls.startTls` | boolean | Yes | `true` | - | Use STARTTLS |
| `tls.certCheck` | boolean | Yes | `false` | - | Verify server certificate |
| `authEnabled` | boolean | Yes | - | - | Enable authentication |
| `username` | string | Yes | - | max 250, required if authEnabled | SMTP username |
| `password` | string | Yes | - | max 250 | SMTP password |
| `sourceEmailAddress` | string | Yes | - | email format, required if enabled | From address |
| `maxNumberOfEntries` | integer | Yes | - | 10-100 | Email queue size |

### Example - Configure SMTP with TLS

```bash
curl -X PUT http://192.168.2.1/api/smtp \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "serverAddress": "smtp.gmail.com",
    "serverPort": 587,
    "tls": {
      "enabled": true,
      "startTls": true,
      "certCheck": true
    },
    "authEnabled": true,
    "username": "alerts@example.com",
    "password": "app-specific-password",
    "sourceEmailAddress": "alerts@example.com",
    "maxNumberOfEntries": 100
  }' \
  -b cookies.txt
```

### Example - Configure SMTP without TLS (Internal Server)

```bash
curl -X PUT http://192.168.2.1/api/smtp \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "serverAddress": "mail.internal.local",
    "serverPort": 25,
    "tls": {
      "enabled": false,
      "startTls": false,
      "certCheck": false
    },
    "authEnabled": false,
    "username": "",
    "password": "",
    "sourceEmailAddress": "gateway@internal.local",
    "maxNumberOfEntries": 50
  }' \
  -b cookies.txt
```

---

## Network Interfaces (NI)

Network interface configuration.

**Source:** `ni.schema.json`

**Endpoints:**
- `GET /api?fields=ni` - Get all network interfaces
- `PUT /api/ni` - Update network interfaces

### Schema

```json
{
  "__v": 3,
  "nis": [
    {
      "name": "br0",
      "type": "LAN",
      "nitype": "BRIDGE",
      "available": true,
      "bridge": "--",
      "vlanId": -1,
      "vlans": [],
      "ipv4": {
        "mode": "STATIC",
        "ip": "192.168.2.1",
        "mask": "255.255.255.0",
        "gateway": "",
        "dns1": "",
        "dns2": "",
        "wanMasquerade": true
      },
      "ipv6": {
        "enabled": false,
        "mode": "STATIC",
        "ip": [],
        "fixedIp": [],
        "linkLocalIp": [],
        "prefixDelegationEnabled": false,
        "delegatedPrefixLength": 64,
        "gateway": "",
        "dns1": "",
        "dns2": ""
      },
      "dot1x": {
        "enabled": false,
        "authType": "",
        "status": "",
        "params": {
          "identity": "",
          "password": "",
          "anonymousIdentity": "",
          "caCertificate": "",
          "clientCertificate": "",
          "privateKey": "",
          "privateKeyPassword": "",
          "domainMatch": "",
          "subjectMatch": "",
          "innerAuthType": "",
          "peapVersion": -1
        }
      }
    }
  ]
}
```

### Interface Object Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Interface name (e.g., `eth0`, `br0`, `wlan0`) |
| `type` | string | Yes | `LAN` or `WAN` |
| `nitype` | string | Yes | `ETHER`, `PPP`, `WIFI_AS_WAN`, `WIFI_AP`, `BRIDGE` |
| `available` | boolean | Yes | Interface is available |
| `bridge` | string | Yes | Bridge connection (`--` if not bridged) |
| `vlanId` | integer | No | VLAN ID (-1 if not VLAN) |
| `vlans` | array | No | VLAN configurations |
| `ipv4` | object | Yes | IPv4 configuration |
| `ipv6` | object | Yes | IPv6 configuration |
| `dot1x` | object | No | 802.1X authentication |
| `MAC` | string | No | MAC address |

### IPv4 Configuration

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `mode` | string | - | `STATIC`, `DHCP`, `DHCP-ADDRESSES-ONLY`, `PPP`, `PPP-ADDRESSES-ONLY` |
| `ip` | string | - | IP address |
| `mask` | string | - | Subnet mask |
| `gateway` | string | - | Default gateway |
| `dns1` | string | - | Primary DNS |
| `dns2` | string | - | Secondary DNS |
| `wanMasquerade` | boolean | `true` | Enable IP masquerading (WAN mode) |

### IPv6 Configuration

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `enabled` | boolean | `false` | Enable IPv6 |
| `mode` | string | - | LAN: `STATIC`, `DELEGATED`; WAN: `AUTO`, `AUTO-ADDRESSES-ONLY` |
| `ip` | array | - | Current IPv6 addresses (read-only) |
| `fixedIp` | array | - | Static IPv6 addresses |
| `linkLocalIp` | array | - | Link-local addresses (read-only) |
| `prefixDelegationEnabled` | boolean | `false` | Enable DHCPv6-PD (WAN only) |
| `delegatedPrefixLength` | integer | `64` | Prefix length (0-128) |
| `gateway` | string | - | Default IPv6 gateway |
| `dns1` | string | - | Primary IPv6 DNS |
| `dns2` | string | - | Secondary IPv6 DNS |

### 802.1X Authentication

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable 802.1X |
| `authType` | string | `EAP-PWD`, `EAP-TLS`, `EAP-TTLS`, `EAP-PEAP` |
| `status` | string | Current status (read-only) |
| `params.identity` | string | Username |
| `params.password` | string | Password (EAP-PWD, TTLS, PEAP) |
| `params.caCertificate` | string | CA certificate (max 20480) |
| `params.clientCertificate` | string | Client certificate (EAP-TLS) |
| `params.privateKey` | string | Private key (EAP-TLS) |
| `params.innerAuthType` | string | `EAP-MSCHAPV2` (TTLS), `MSCHAPV2` (PEAP) |

### Example - Configure Static LAN

```bash
curl -X PUT http://192.168.2.1/api/ni \
  -H "Content-Type: application/json" \
  -d '{
    "__v": 3,
    "nis": [
      {
        "name": "br0",
        "type": "LAN",
        "nitype": "BRIDGE",
        "available": true,
        "bridge": "--",
        "ipv4": {
          "mode": "STATIC",
          "ip": "192.168.100.1",
          "mask": "255.255.255.0",
          "gateway": "",
          "dns1": "8.8.8.8",
          "dns2": "8.8.4.4",
          "wanMasquerade": true
        },
        "ipv6": {
          "enabled": false,
          "mode": "STATIC",
          "ip": [],
          "fixedIp": [],
          "linkLocalIp": [],
          "prefixDelegationEnabled": false,
          "delegatedPrefixLength": 64,
          "gateway": "",
          "dns1": "",
          "dns2": ""
        }
      }
    ]
  }' \
  -b cookies.txt
```

### Example - Configure WAN with DHCP

```bash
curl -X PUT http://192.168.2.1/api/ni \
  -H "Content-Type: application/json" \
  -d '{
    "__v": 3,
    "nis": [
      {
        "name": "eth0",
        "type": "WAN",
        "nitype": "ETHER",
        "available": true,
        "bridge": "--",
        "ipv4": {
          "mode": "DHCP",
          "ip": "",
          "mask": "",
          "gateway": "",
          "dns1": "",
          "dns2": "",
          "wanMasquerade": true
        },
        "ipv6": {
          "enabled": true,
          "mode": "AUTO",
          "ip": [],
          "fixedIp": [],
          "linkLocalIp": [],
          "prefixDelegationEnabled": true,
          "delegatedPrefixLength": 56,
          "gateway": "",
          "dns1": "",
          "dns2": ""
        }
      }
    ]
  }' \
  -b cookies.txt
```

---

## Related Documentation

- [API Schemas Reference](API-SCHEMAS-REFERENCE.md) - Complete schema reference
- [Available Resources](AVAILABLE-RESOURCES.md) - Resource endpoints
- [VPN Schemas](VPN-SCHEMAS.md) - VPN configuration

---

**Document Version:** 1.0.0  
**Source Version:** mtsDeviceAPI (analyzed February 2026)  
**Last Updated:** February 10, 2026
