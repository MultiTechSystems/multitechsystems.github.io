# SNMP Configuration Schema

Complete schema documentation for the `snmp` resource in the mPower Device API.

## Overview

The `snmp` resource configures the SNMP (Simple Network Management Protocol) agent on the device, enabling network monitoring and management through SNMP clients and trap receivers. The device supports SNMPv1, SNMPv2c, and SNMPv3 with full authentication and encryption capabilities.

## Top-Level Structure

```json
{
  "__v": 2,
  "server": {
    // SNMP agent configuration
  },
  "traps": {
    // SNMP trap configuration
  }
}
```

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `__v` | integer | Yes | Schema version (must be 2) |
| `server` | object | Yes | SNMP agent/server configuration |
| `traps` | object | Yes | SNMP trap destinations configuration |

## SNMP Server Configuration

The `server` object configures the SNMP agent that responds to SNMP queries.

### Server Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `enabled` | boolean | Yes | Enable/disable SNMP server |
| `sysName` | string | Yes | SNMP sysName object (max 255 chars, printable ASCII) |
| `sysLocation` | string | Yes | SNMP sysLocation object (max 255 chars, printable ASCII) |
| `sysContact` | string | Yes | SNMP sysContact object (max 255 chars, printable ASCII) |
| `users` | array | Yes | Array of SNMP user configurations (max 5 users) |
| `allowedAddresses` | array | Yes | Allowed IP addresses for v1/v2c access |

### SNMP Server User Object

Each user in the `users` array:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `enabled` | boolean | Yes | Enable/disable this user |
| `label` | string | Yes | Unique configuration entry name (1-31 chars) |
| `version` | string | Yes | SNMP version: "v1", "v2c", "v1v2c", or "v3" |
| `username` | string | Yes | Username for SNMPv3 (min 5 chars for v3) |
| `authProto` | string | Yes | Authentication protocol: "", "MD5", or "SHA1" |
| `authPass` | string | Yes | Authentication password (min 8 chars if authProto set) |
| `privProto` | string | Yes | Privacy protocol: "", "DES", or "AES-128" |
| `privPass` | string | Yes | Privacy password (min 8 chars, max 128 if privProto set) |
| `community` | string | Yes | Community string for v1/v2c (1-128 chars, no quotes) |

### SNMP Version Comparison

| Version | Authentication | Encryption | Use Case |
|---------|----------------|------------|----------|
| **v1** | Community string | None | Legacy systems, simple monitoring |
| **v2c** | Community string | None | Improved v1, better error handling |
| **v1v2c** | Community string | None | Support both v1 and v2c clients |
| **v3** | Username + Auth protocol | Privacy protocol | Secure monitoring, modern systems |

### SNMPv3 Security Levels

| Auth | Priv | Security Level | Description |
|------|------|----------------|-------------|
| None | None | noAuthNoPriv | No authentication, no encryption |
| MD5/SHA1 | None | authNoPriv | Authentication only |
| MD5/SHA1 | DES/AES | authPriv | Authentication and encryption |

### Allowed Addresses Object

For SNMPv1/v2c access control:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `address` | string | Yes | IPv4 address |
| `prefix` | integer | Yes | Network prefix length (1-32) |

**Examples:**
- `{"address": "192.168.1.0", "prefix": 24}` - Allow 192.168.1.0/24 subnet
- `{"address": "10.0.0.5", "prefix": 32}` - Allow single IP 10.0.0.5

## SNMP Traps Configuration

The `traps` object configures SNMP trap (notification) destinations.

### Traps Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `enabled` | boolean | Yes | Enable/disable SNMP traps |
| `engineId` | string | Yes | Authoritative Engine ID (hex, 5-32 octets, e.g., "0x0102030405") |
| `engineIdDefault` | string | Yes | Default Engine ID (read-only, auto-generated) |
| `receivers` | array | Yes | Array of trap receiver configurations (max 5) |

### Engine ID Format

When traps are enabled, `engineId` must be:
- Hexadecimal string starting with "0x" or "0X"
- Between 10 and 64 hex characters (5-32 octets)
- Example: `"0x0102030405060708090A"`

### Trap Receiver Object

Each receiver in the `receivers` array:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `enabled` | boolean | Yes | Enable/disable this trap receiver |
| `label` | string | Yes | Unique configuration entry name (1-31 chars) |
| `address` | string | Yes | IPv4 address of trap receiver |
| `version` | string | Yes | SNMP version: "v1", "v2c", "v1v2c", or "v3" |
| `username` | string | Yes | Username for SNMPv3 traps (min 5 chars for v3) |
| `authProto` | string | Yes | Authentication protocol: "", "MD5", or "SHA1" |
| `authPass` | string | Yes | Authentication password (min 8 chars if authProto set) |
| `privProto` | string | Yes | Privacy protocol: "", "DES", or "AES-128" |
| `privPass` | string | Yes | Privacy password (min 8 chars, max 128 if privProto set) |
| `community` | string | Yes | Community string for v1/v2c (1-128 chars, no quotes) |

### Trap Event Types (MTRE devices only)

| Property | Type | Description |
|----------|------|-------------|
| `eventsAuth` | boolean | Authentication events |
| `eventsChassis` | boolean | Chassis/hardware events |
| `eventsConf` | boolean | Configuration change events |
| `eventsLink` | boolean | Link up/down events |
| `eventsSecurity` | boolean | Security events |
| `eventsWan` | boolean | WAN connectivity events |

## Configuration Examples

### Example 1: SNMPv2c Server with Community String

```json
{
  "__v": 2,
  "server": {
    "enabled": true,
    "sysName": "MyGateway",
    "sysLocation": "Building A, Floor 2",
    "sysContact": "admin@example.com",
    "users": [
      {
        "enabled": true,
        "label": "Public Access",
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
      {
        "address": "192.168.1.0",
        "prefix": 24
      },
      {
        "address": "10.0.0.100",
        "prefix": 32
      }
    ]
  },
  "traps": {
    "enabled": false,
    "engineId": "",
    "engineIdDefault": "0x80001f8880...",
    "receivers": []
  }
}
```

### Example 2: SNMPv3 Server with Authentication and Encryption

```json
{
  "__v": 2,
  "server": {
    "enabled": true,
    "sysName": "SecureGateway",
    "sysLocation": "Data Center",
    "sysContact": "netadmin@example.com",
    "users": [
      {
        "enabled": true,
        "label": "Secure Monitoring",
        "version": "v3",
        "username": "snmpuser",
        "authProto": "SHA1",
        "authPass": "authpassword123",
        "privProto": "AES-128",
        "privPass": "privpassword456",
        "community": ""
      }
    ],
    "allowedAddresses": []
  },
  "traps": {
    "enabled": false,
    "engineId": "",
    "engineIdDefault": "0x80001f8880...",
    "receivers": []
  }
}
```

### Example 3: SNMPv2c with Traps

```json
{
  "__v": 2,
  "server": {
    "enabled": true,
    "sysName": "Gateway01",
    "sysLocation": "Remote Site",
    "sysContact": "support@example.com",
    "users": [
      {
        "enabled": true,
        "label": "Monitoring",
        "version": "v2c",
        "username": "",
        "authProto": "",
        "authPass": "",
        "privProto": "",
        "privPass": "",
        "community": "monitor"
      }
    ],
    "allowedAddresses": [
      {
        "address": "10.1.1.0",
        "prefix": 24
      }
    ]
  },
  "traps": {
    "enabled": true,
    "engineId": "0x0102030405060708",
    "engineIdDefault": "0x80001f8880...",
    "receivers": [
      {
        "enabled": true,
        "label": "NMS Server",
        "address": "10.1.1.50",
        "version": "v2c",
        "username": "",
        "authProto": "",
        "authPass": "",
        "privProto": "",
        "privPass": "",
        "community": "traps"
      }
    ]
  }
}
```

### Example 4: SNMPv3 with Secure Traps

```json
{
  "__v": 2,
  "server": {
    "enabled": true,
    "sysName": "Gateway02",
    "sysLocation": "HQ",
    "sysContact": "ops@example.com",
    "users": [
      {
        "enabled": true,
        "label": "V3 User",
        "version": "v3",
        "username": "snmpv3user",
        "authProto": "SHA1",
        "authPass": "myauthpass",
        "privProto": "AES-128",
        "privPass": "myprivpass",
        "community": ""
      }
    ],
    "allowedAddresses": []
  },
  "traps": {
    "enabled": true,
    "engineId": "0x0A0B0C0D0E0F",
    "engineIdDefault": "0x80001f8880...",
    "receivers": [
      {
        "enabled": true,
        "label": "Secure NMS",
        "address": "10.2.2.100",
        "version": "v3",
        "username": "trapuser",
        "authProto": "SHA1",
        "authPass": "trapauthpass",
        "privProto": "AES-128",
        "privPass": "trapprivpass",
        "community": ""
      }
    ]
  }
}
```

### Example 5: Multiple Users and Trap Receivers

```json
{
  "__v": 2,
  "server": {
    "enabled": true,
    "sysName": "MultiUserGateway",
    "sysLocation": "Branch Office",
    "sysContact": "admin@example.com",
    "users": [
      {
        "enabled": true,
        "label": "V2c Readonly",
        "version": "v2c",
        "username": "",
        "authProto": "",
        "authPass": "",
        "privProto": "",
        "privPass": "",
        "community": "public"
      },
      {
        "enabled": true,
        "label": "V3 Secure",
        "version": "v3",
        "username": "admin",
        "authProto": "SHA1",
        "authPass": "adminauth",
        "privProto": "AES-128",
        "privPass": "adminpriv",
        "community": ""
      }
    ],
    "allowedAddresses": [
      {
        "address": "192.168.0.0",
        "prefix": 16
      }
    ]
  },
  "traps": {
    "enabled": true,
    "engineId": "0x1122334455",
    "engineIdDefault": "0x80001f8880...",
    "receivers": [
      {
        "enabled": true,
        "label": "Primary NMS",
        "address": "10.1.1.10",
        "version": "v2c",
        "username": "",
        "authProto": "",
        "authPass": "",
        "privProto": "",
        "privPass": "",
        "community": "traps"
      },
      {
        "enabled": true,
        "label": "Backup NMS",
        "address": "10.1.1.11",
        "version": "v3",
        "username": "trapuser",
        "authProto": "MD5",
        "authPass": "trapauth",
        "privProto": "DES",
        "privPass": "trappriv",
        "community": ""
      }
    ]
  }
}
```

## API Operations

### Get SNMP Configuration

```bash
GET /api?fields=snmp
```

Response:
```json
{
  "success": true,
  "result": {
    "__v": 2,
    "server": {
      // Server configuration
    },
    "traps": {
      // Traps configuration
    }
  }
}
```

### Update SNMP Configuration

```bash
PUT /api/snmp
Content-Type: application/json

{
  "__v": 2,
  "server": {
    "enabled": true,
    "sysName": "MyDevice",
    "sysLocation": "Office",
    "sysContact": "admin@example.com",
    "users": [...],
    "allowedAddresses": [...]
  },
  "traps": {
    "enabled": true,
    "engineId": "0x0102030405",
    "engineIdDefault": "0x80001f8880...",
    "receivers": [...]
  }
}
```

### Save Configuration

```bash
POST /api/command/save
```

## Configuration Workflows

### Workflow 1: Enable SNMPv2c Monitoring

```bash
# 1. Login
curl -X POST http://192.168.2.1/api/login \
  -d '{"username":"admin","password":"admin"}' -c cookies.txt

# 2. Configure SNMP
curl -X PUT http://192.168.2.1/api/snmp \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "__v": 2,
    "server": {
      "enabled": true,
      "sysName": "Gateway01",
      "sysLocation": "Building A",
      "sysContact": "admin@example.com",
      "users": [{
        "enabled": true,
        "label": "Monitoring",
        "version": "v2c",
        "username": "",
        "authProto": "",
        "authPass": "",
        "privProto": "",
        "privPass": "",
        "community": "public"
      }],
      "allowedAddresses": [{
        "address": "192.168.1.0",
        "prefix": 24
      }]
    },
    "traps": {
      "enabled": false,
      "engineId": "",
      "engineIdDefault": "",
      "receivers": []
    }
  }'

# 3. Save
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

### Workflow 2: Enable SNMPv3 with Secure Traps

```bash
# Configure SNMPv3 with authentication and encryption
curl -X PUT http://192.168.2.1/api/snmp \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "__v": 2,
    "server": {
      "enabled": true,
      "sysName": "SecureGateway",
      "sysLocation": "Data Center",
      "sysContact": "netops@example.com",
      "users": [{
        "enabled": true,
        "label": "Secure Monitor",
        "version": "v3",
        "username": "snmpuser",
        "authProto": "SHA1",
        "authPass": "authpass123",
        "privProto": "AES-128",
        "privPass": "privpass456",
        "community": ""
      }],
      "allowedAddresses": []
    },
    "traps": {
      "enabled": true,
      "engineId": "0x0A0B0C0D0E0F",
      "engineIdDefault": "",
      "receivers": [{
        "enabled": true,
        "label": "NMS Server",
        "address": "10.1.1.50",
        "version": "v3",
        "username": "trapuser",
        "authProto": "SHA1",
        "authPass": "trapauthpass",
        "privProto": "AES-128",
        "privPass": "trapprivpass",
        "community": ""
      }]
    }
  }'

# Save configuration
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

## Testing SNMP Configuration

### Test SNMPv2c

```bash
# From a Linux/Mac system with net-snmp tools
snmpwalk -v 2c -c public 192.168.2.1 system
snmpget -v 2c -c public 192.168.2.1 sysName.0
```

### Test SNMPv3

```bash
# With authentication only (authNoPriv)
snmpwalk -v 3 -l authNoPriv \
  -u snmpuser -a SHA -A authpass123 \
  192.168.2.1 system

# With authentication and encryption (authPriv)
snmpwalk -v 3 -l authPriv \
  -u snmpuser -a SHA -A authpass123 \
  -x AES -X privpass456 \
  192.168.2.1 system
```

### Test Trap Reception

```bash
# Start trap receiver (snmptrapd)
snmptrapd -f -Lo -c /etc/snmp/snmptrapd.conf

# Trigger a test trap from device
# (Traps are sent automatically on system events)
```

## Important Notes

### Version-Specific Requirements

#### SNMPv1/v2c
- **Required**: `community` string (1-128 characters, no quotes)
- **Optional**: `allowedAddresses` for access control
- **Not used**: username, authProto, authPass, privProto, privPass

#### SNMPv3
- **Required**: `username` (minimum 5 characters)
- **Optional but recommended**: Authentication (authProto + authPass)
- **Optional**: Encryption (privProto + privPass, requires authentication)
- **Not used**: community, allowedAddresses

### Password Requirements

- **Authentication password**: Minimum 8 characters (when authProto is set)
- **Privacy password**: Minimum 8 characters, maximum 128 (when privProto is set)
- **No quotes**: Passwords cannot contain quote characters (`"`)

### Engine ID Requirements

- Must be hexadecimal string with "0x" prefix
- Length: 10-64 hex characters (5-32 octets)
- Must be unique across all SNMP agents in your network
- Default value is auto-generated based on device MAC address

### Access Control

- **SNMPv1/v2c**: Use `allowedAddresses` to restrict access by IP
- **SNMPv3**: Access control is handled by username/password authentication
- Empty `allowedAddresses` array = allow from any IP (not recommended for v1/v2c)

### Limits

- **Maximum users**: 5
- **Maximum trap receivers**: 5
- **sysName/sysLocation/sysContact**: Maximum 255 printable ASCII characters

### Security Best Practices

1. **Use SNMPv3** whenever possible for secure monitoring
2. **Enable authentication** (authProto) for SNMPv3
3. **Enable encryption** (privProto) for sensitive environments
4. **Use strong passwords** (minimum 8 characters, complex)
5. **Restrict access** with allowedAddresses for v1/v2c
6. **Change default community** strings (avoid "public", "private")
7. **Use unique Engine ID** for trap receivers

## Common Use Cases

### Use Case 1: Basic Network Monitoring

Configure SNMPv2c for simple network monitoring:
- Use community string "public" or custom string
- Restrict access to monitoring subnet
- No traps needed for polling-based monitoring

### Use Case 2: Enterprise Monitoring with NMS

Configure SNMPv3 with secure traps:
- Use SHA1 authentication
- Use AES-128 encryption
- Configure trap receivers for NMS systems
- Enable relevant trap event types

### Use Case 3: Multi-Site Monitoring

Configure multiple trap receivers:
- Primary NMS at headquarters
- Backup NMS for redundancy
- Regional monitoring stations
- Use appropriate SNMP versions per site

## Troubleshooting

### SNMP Queries Not Working

**Check:**
- SNMP server is enabled: `server.enabled = true`
- User is enabled: `users[].enabled = true`
- Community string is correct (v1/v2c)
- Username/password are correct (v3)
- Client IP is in allowedAddresses (v1/v2c)
- No firewall blocking UDP port 161

**Test:**
```bash
# Test from device itself
snmpwalk -v 2c -c public localhost system
```

### SNMPv3 Authentication Fails

**Check:**
- Username is at least 5 characters
- authProto is set ("MD5" or "SHA1")
- authPass is at least 8 characters
- Passwords don't contain quotes
- Security level matches configuration

### Traps Not Received

**Check:**
- Traps are enabled: `traps.enabled = true`
- Receiver is enabled: `receivers[].enabled = true`
- Engine ID is valid (hex with 0x prefix, 10-64 chars)
- Receiver address is correct
- Community/credentials match receiver configuration
- No firewall blocking UDP port 162
- Trap receiver daemon is running

### Engine ID Issues

**Problem**: Invalid Engine ID format

**Solution**:
- Must start with "0x" or "0X"
- Must be 10-64 hex characters (5-32 octets)
- Example valid: `"0x0102030405060708"`
- Example invalid: `"0102030405"` (missing 0x prefix)

## SNMP MIB Support

The device supports standard SNMP MIBs including:
- **SNMPv2-MIB**: System group (sysName, sysLocation, sysContact, etc.)
- **IF-MIB**: Network interface statistics
- **IP-MIB**: IP statistics and routing
- **TCP-MIB**: TCP connection statistics
- **UDP-MIB**: UDP statistics
- **HOST-RESOURCES-MIB**: System resources (CPU, memory, storage)

Custom/vendor-specific MIBs may also be available depending on device model.

## Related Resources

- [Network Interfaces Configuration](NETWORK-INTERFACES-SCHEMA.md)
- [Alert Configuration](ALERT-SCHEMA.md)
- [Syslog Configuration](API-README.md#monitoring-resources)
- [Main API Documentation](API-README.md)

---

**Last Updated**: December 17, 2025  
**Schema Version**: 2
