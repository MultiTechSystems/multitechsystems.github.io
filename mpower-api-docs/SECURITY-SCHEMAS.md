# Security Configuration Schemas

Complete schema documentation for security-related resources: `secureProtocols`, `remoteAccess`, `trustedIp`, and `radius`.

---

# Secure Protocols

## Overview

The `secureProtocols` resource configures TLS/SSL cipher suites and SSH security settings for various device services (HTTPS, SSH, etc.).

## Structure

The `secureProtocols` resource is an array of protocol configurations:

```json
[
  {
    "name": "https",
    "type": "openssl",
    // OpenSSL configuration
  },
  {
    "name": "ssh",
    "type": "openssh",
    // OpenSSH configuration
  }
]
```

## Secure Protocol Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `__v` | integer | No | Schema version (2) |
| `name` | string | Yes | Component name (e.g., "https", "ssh") |
| `type` | string | Yes | Library type: "openssl" or "openssh" |

### OpenSSL Configuration (TLS/HTTPS)

| Property | Type | Description |
|----------|------|-------------|
| `protocol` | object | TLS protocol versions |
| `protocol.tls1_2` | boolean | Enable TLS 1.2 (default: true) |
| `protocol.tls1_3` | boolean | Enable TLS 1.3 (default: true) |
| `cipherSuite` | string | TLS 1.2 cipher suites (colon-separated) |
| `cipherSuiteTls13` | string | TLS 1.3 cipher suites (colon-separated) |
| `cipherSuiteAvailable` | string | Available TLS 1.2 ciphers (read-only) |
| `cipherSuiteAvailableTls13` | string | Available TLS 1.3 ciphers (read-only) |
| `cipherSuiteDeprecated` | array | Deprecated cipher suites |
| `client` | object | Client authentication settings |
| `client.verify` | boolean | Enable client certificate authentication |

### OpenSSH Configuration (SSH)

| Property | Type | Description |
|----------|------|-------------|
| `cipher` | string | SSH cipher suites (comma-separated) |
| `cipherAvailable` | string | Available SSH ciphers (read-only) |
| `cipherDeprecated` | array | Deprecated SSH ciphers |
| `mac` | string | Message authentication codes (comma-separated) |
| `macAvailable` | string | Available MACs (read-only) |
| `macDeprecated` | array | Deprecated MACs |

## Secure Protocols Examples

### Example 1: HTTPS with Strong TLS Configuration

```json
{
  "name": "https",
  "type": "openssl",
  "protocol": {
    "tls1_2": true,
    "tls1_3": true
  },
  "cipherSuite": "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256",
  "cipherSuiteTls13": "TLS_AES_256_GCM_SHA384:TLS_AES_128_GCM_SHA256",
  "cipherSuiteDeprecated": [],
  "client": {
    "verify": false
  }
}
```

### Example 2: SSH with Modern Ciphers

```json
{
  "name": "ssh",
  "type": "openssh",
  "cipher": "chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr",
  "cipherDeprecated": [],
  "mac": "hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256",
  "macDeprecated": []
}
```

### Example 3: HTTPS with Client Certificate Authentication

```json
{
  "name": "https",
  "type": "openssl",
  "protocol": {
    "tls1_2": true,
    "tls1_3": true
  },
  "cipherSuite": "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384",
  "cipherSuiteTls13": "TLS_AES_256_GCM_SHA384",
  "cipherSuiteDeprecated": [],
  "client": {
    "verify": true
  }
}
```

---

# Remote Access

## Overview

The `remoteAccess` resource configures access control for device management interfaces including HTTP, HTTPS, SSH, ICMP, SNMP, and security features.

## Structure

```json
{
  "timeoutSeconds": 300,
  "http": { /* HTTP settings */ },
  "https": { /* HTTPS settings */ },
  "ssh": { /* SSH settings */ },
  "reverseSsh": { /* Reverse SSH tunnel */ },
  "icmp": { /* ICMP/ping settings */ },
  "snmp": { /* SNMP access control */ },
  "modbusSlave": { /* Modbus settings */ },
  "dosAttackPrevention": { /* DoS protection */ },
  "pingLimit": { /* ICMP throttling */ },
  "bruteForcePrevention": { /* Login throttling */ },
  "privateWan": false
}
```

## Remote Access Properties

### Global Settings

| Property | Type | Description |
|----------|------|-------------|
| `timeoutSeconds` | integer | Session timeout (60+ seconds, default: 300) |
| `privateWan` | boolean | Drop packets with private source IPs from WAN |

### HTTP Configuration

| Property | Type | Description |
|----------|------|-------------|
| `http.enabled` | boolean | Enable HTTP access |
| `http.port` | integer | HTTP port (default: 80) |
| `http.lan` | boolean | Allow HTTP from LAN |
| `http.wan` | boolean | Allow HTTP from WAN |
| `http.redirectToHttps` | boolean | Redirect HTTP to HTTPS |

### HTTPS Configuration

| Property | Type | Description |
|----------|------|-------------|
| `https.enabled` | boolean | Enable HTTPS access |
| `https.port` | integer | HTTPS port (default: 443) |
| `https.lan` | boolean | Allow HTTPS from LAN |
| `https.wan` | boolean | Allow HTTPS from WAN |

### SSH Configuration

| Property | Type | Description |
|----------|------|-------------|
| `ssh.enabled` | boolean | Enable SSH access |
| `ssh.port` | integer | SSH port (default: 22) |
| `ssh.lan` | boolean | Allow SSH from LAN |
| `ssh.wan` | boolean | Allow SSH from WAN |
| `ssh.idleTimeoutSeconds` | integer | SSH idle timeout (60-3600 or 0=disabled) |
| `ssh.passwordAuthentication` | boolean | Allow password authentication |
| `ssh.publicKeyAuthentication` | boolean | Allow public key authentication |
| `ssh.userLockout` | object | SSH login throttling |
| `ssh.userLockout.enabled` | boolean | Enable SSH lockout |
| `ssh.userLockout.failAttempts` | integer | Failed attempts before lockout (1-99) |
| `ssh.userLockout.lockoutSeconds` | integer | Lockout duration (60-3600) |

### Reverse SSH Tunnel

| Property | Type | Description |
|----------|------|-------------|
| `reverseSsh.enabled` | boolean | Enable reverse SSH tunnel |
| `reverseSsh.serverAddress` | string | Remote SSH server IP/hostname |
| `reverseSsh.serverPort` | integer | Remote SSH port (default: 22) |
| `reverseSsh.remotePort` | integer | Tunnel remote port (default: 2222) |
| `reverseSsh.authMode` | string | "PASSWORD", "PUBLIC_KEY", "PRIVATE_KEY" |
| `reverseSsh.username` | string | Remote server username |
| `reverseSsh.password` | string | Remote server password |
| `reverseSsh.privateKey` | string | Private key for authentication |

### ICMP Configuration

| Property | Type | Description |
|----------|------|-------------|
| `icmp.enabled` | boolean | Enable ICMP responses |
| `icmp.respondToLan` | boolean | Respond to LAN pings |
| `icmp.respondToWan` | boolean | Respond to WAN pings |

### SNMP Access Control

| Property | Type | Description |
|----------|------|-------------|
| `snmp.lan` | boolean | Allow SNMP from LAN |
| `snmp.wan` | boolean | Allow SNMP from WAN |

### Modbus Slave

| Property | Type | Description |
|----------|------|-------------|
| `modbusSlave.enabled` | boolean | Enable Modbus query server |
| `modbusSlave.port` | integer | Modbus port (default: 1502) |
| `modbusSlave.lan` | boolean | Allow Modbus from LAN |

### DoS Attack Prevention

| Property | Type | Description |
|----------|------|-------------|
| `dosAttackPrevention.enabled` | boolean | Enable connection throttling |
| `dosAttackPrevention.limitPerMinute` | integer | Average connections/minute |
| `dosAttackPrevention.limitBurst` | integer | Maximum burst connections/minute |

### Ping Limit (ICMP Throttling)

| Property | Type | Description |
|----------|------|-------------|
| `pingLimit.enabled` | boolean | Enable ICMP throttling |
| `pingLimit.limitPerSecond` | integer | Average ICMP packets/second |
| `pingLimit.limitBurst` | integer | Maximum burst ICMP packets/second |

### Brute Force Prevention

| Property | Type | Description |
|----------|------|-------------|
| `bruteForcePrevention.enabled` | boolean | Enable login throttling |
| `bruteForcePrevention.lockoutSeconds` | integer | Account lockout duration |
| `bruteForcePrevention.failAttempts` | integer | Failed attempts before lockout |

## Remote Access Examples

### Example 1: Secure Remote Access (HTTPS only, SSH from LAN)

```json
{
  "timeoutSeconds": 600,
  "http": {
    "enabled": true,
    "port": 80,
    "lan": true,
    "wan": false,
    "redirectToHttps": true
  },
  "https": {
    "enabled": true,
    "port": 443,
    "lan": true,
    "wan": true
  },
  "ssh": {
    "enabled": true,
    "port": 22,
    "lan": true,
    "wan": false,
    "idleTimeoutSeconds": 300,
    "passwordAuthentication": true,
    "publicKeyAuthentication": true,
    "userLockout": {
      "enabled": true,
      "failAttempts": 3,
      "lockoutSeconds": 300
    }
  },
  "reverseSsh": {
    "enabled": false,
    "serverAddress": "",
    "serverPort": 22,
    "remotePort": 2222,
    "authMode": "PASSWORD",
    "username": "",
    "password": ""
  },
  "icmp": {
    "enabled": true,
    "respondToLan": true,
    "respondToWan": false
  },
  "snmp": {
    "lan": true,
    "wan": false
  },
  "modbusSlave": {
    "enabled": false,
    "port": 1502,
    "lan": true
  },
  "dosAttackPrevention": {
    "enabled": true,
    "limitPerMinute": 60,
    "limitBurst": 100
  },
  "pingLimit": {
    "enabled": true,
    "limitPerSecond": 10,
    "limitBurst": 30
  },
  "bruteForcePrevention": {
    "enabled": true,
    "lockoutSeconds": 300,
    "failAttempts": 3
  },
  "privateWan": true
}
```

### Example 2: Reverse SSH Tunnel for Remote Support

```json
{
  "reverseSsh": {
    "enabled": true,
    "serverAddress": "support.example.com",
    "serverPort": 22,
    "remotePort": 2222,
    "authMode": "PUBLIC_KEY",
    "username": "support",
    "password": "",
    "privateKey": "-----BEGIN OPENSSH PRIVATE KEY-----\n..."
  }
}
```

---

# Trusted IP

## Overview

The `trustedIp` resource implements IP-based access control with whitelist or blacklist modes.

## Structure

```json
{
  "enabled": false,
  "mode": "whitelist",
  "rules": [
    {
      // Trusted IP rule
    }
  ]
}
```

## Trusted IP Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `enabled` | boolean | Yes | Enable/disable trusted IP filtering |
| `mode` | string | Yes | "whitelist" or "blacklist" |
| `rules` | array | Yes | Array of IP filtering rules |

### Trusted IP Rule

| Property | Type | Description |
|----------|------|-------------|
| `name` | string | Rule name |
| `ipMode` | string | "subnet" or "range" |
| `subnetSrcAddr` | string | Source IP (for subnet mode) |
| `subnetSrcMask` | string | Subnet mask (for subnet mode) |
| `rangeSrcAddrStart` | string | Range start IP (for range mode) |
| `rangeSrcAddrEnd` | string | Range end IP (for range mode) |
| `dstPort` | string | Destination port(s) or "ANY" |
| `protocol` | string | "TCP", "UDP", "TCP/UDP", "ANY" |

## Trusted IP Modes

| Mode | Behavior |
|------|----------|
| `whitelist` | Allow traffic from listed IPs, drop all others |
| `blacklist` | Drop traffic from listed IPs, allow all others |

## Trusted IP Examples

### Example 1: Whitelist Specific Subnet

```json
{
  "enabled": true,
  "mode": "whitelist",
  "rules": [
    {
      "name": "Corporate Network",
      "ipMode": "subnet",
      "subnetSrcAddr": "10.0.0.0",
      "subnetSrcMask": "8",
      "rangeSrcAddrStart": "",
      "rangeSrcAddrEnd": "",
      "dstPort": "ANY",
      "protocol": "ANY"
    },
    {
      "name": "Admin Workstation",
      "ipMode": "subnet",
      "subnetSrcAddr": "192.168.1.100",
      "subnetSrcMask": "32",
      "rangeSrcAddrStart": "",
      "rangeSrcAddrEnd": "",
      "dstPort": "ANY",
      "protocol": "ANY"
    }
  ]
}
```

### Example 2: Blacklist IP Range

```json
{
  "enabled": true,
  "mode": "blacklist",
  "rules": [
    {
      "name": "Block Suspicious Range",
      "ipMode": "range",
      "subnetSrcAddr": "",
      "subnetSrcMask": "",
      "rangeSrcAddrStart": "203.0.113.0",
      "rangeSrcAddrEnd": "203.0.113.255",
      "dstPort": "ANY",
      "protocol": "ANY"
    }
  ]
}
```

### Example 3: Whitelist Specific Services

```json
{
  "enabled": true,
  "mode": "whitelist",
  "rules": [
    {
      "name": "Allow HTTPS from Anywhere",
      "ipMode": "subnet",
      "subnetSrcAddr": "0.0.0.0",
      "subnetSrcMask": "0",
      "rangeSrcAddrStart": "",
      "rangeSrcAddrEnd": "",
      "dstPort": "443",
      "protocol": "TCP"
    },
    {
      "name": "Allow SSH from Admin Network",
      "ipMode": "subnet",
      "subnetSrcAddr": "192.168.1.0",
      "subnetSrcMask": "24",
      "rangeSrcAddrStart": "",
      "rangeSrcAddrEnd": "",
      "dstPort": "22",
      "protocol": "TCP"
    }
  ]
}
```

---

# RADIUS Authentication

## Overview

The `radius` resource configures RADIUS authentication and accounting for device user authentication.

## Structure

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

## RADIUS Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `authEnabled` | boolean | Yes | Enable RADIUS authentication |
| `acctEnabled` | boolean | Yes | Enable RADIUS accounting |
| `host` | string | Conditional | Primary RADIUS server (required if authEnabled) |
| `authPort` | integer | Conditional | Primary auth port (1-65535, default: 1812) |
| `acctPort` | integer | Conditional | Primary accounting port (1-65535, default: 1813) |
| `secondaryHost` | string | No | Secondary RADIUS server |
| `secondaryAuthPort` | integer | Conditional | Secondary auth port |
| `secondaryAcctPort` | integer | Conditional | Secondary accounting port |
| `secret` | string | Conditional | Shared secret (required if authEnabled) |
| `authType` | string | Yes | "PAP", "EAP-TTLSv0/PAP", "EAP-PEAPv0/MSCHAPv2" |
| `timeout` | integer | Yes | Retry interval in seconds (default: 2) |
| `retries` | integer | Yes | Maximum retry attempts (default: 3) |
| `checkServer` | boolean | Yes | Validate server certificate CN |
| `anonymousEnabled` | boolean | Yes | Enable anonymous identity |
| `anonymousId` | string | Yes | Anonymous identity string |

## RADIUS Authentication Types

| Type | Description | Use Case |
|------|-------------|----------|
| `PAP` | Password Authentication Protocol | Simple, less secure |
| `EAP-TTLSv0/PAP` | EAP-TTLS with PAP | Secure tunneled authentication |
| `EAP-PEAPv0/MSCHAPv2` | PEAP with MSCHAPv2 | Microsoft-compatible secure auth |

## RADIUS Examples

### Example 1: Basic RADIUS Authentication

```json
{
  "authEnabled": true,
  "acctEnabled": false,
  "host": "radius.example.com",
  "authPort": 1812,
  "acctPort": 1813,
  "secondaryHost": "",
  "secondaryAuthPort": 1812,
  "secondaryAcctPort": 1813,
  "secret": "MyRadiusSecret123",
  "authType": "PAP",
  "timeout": 2,
  "retries": 3,
  "checkServer": true,
  "anonymousEnabled": false,
  "anonymousId": "anonymous"
}
```

### Example 2: RADIUS with Accounting and Failover

```json
{
  "authEnabled": true,
  "acctEnabled": true,
  "host": "radius1.example.com",
  "authPort": 1812,
  "acctPort": 1813,
  "secondaryHost": "radius2.example.com",
  "secondaryAuthPort": 1812,
  "secondaryAcctPort": 1813,
  "secret": "SharedSecret456",
  "authType": "EAP-PEAPv0/MSCHAPv2",
  "timeout": 3,
  "retries": 5,
  "checkServer": true,
  "anonymousEnabled": true,
  "anonymousId": "guest"
}
```

### Example 3: Secure RADIUS with EAP-TTLS

```json
{
  "authEnabled": true,
  "acctEnabled": true,
  "host": "10.1.1.50",
  "authPort": 1812,
  "acctPort": 1813,
  "secondaryHost": "10.1.1.51",
  "secondaryAuthPort": 1812,
  "secondaryAcctPort": 1813,
  "secret": "StrongSecret789!",
  "authType": "EAP-TTLSv0/PAP",
  "timeout": 2,
  "retries": 3,
  "checkServer": true,
  "anonymousEnabled": true,
  "anonymousId": "anonymous@example.com"
}
```

---

# API Operations

## Secure Protocols

### Get Secure Protocols

```bash
GET /api?fields=secureProtocols
```

### Update Secure Protocols

```bash
PUT /api/secureProtocols
Content-Type: application/json

[
  {
    "name": "https",
    "type": "openssl",
    "protocol": {
      "tls1_2": true,
      "tls1_3": true
    },
    "cipherSuite": "ECDHE-ECDSA-AES256-GCM-SHA384:...",
    "cipherSuiteTls13": "TLS_AES_256_GCM_SHA384:...",
    "client": {
      "verify": false
    }
  }
]
```

## Remote Access

### Get Remote Access Settings

```bash
GET /api?fields=remoteAccess
```

### Update Remote Access

```bash
PUT /api/remoteAccess
Content-Type: application/json

{
  "timeoutSeconds": 600,
  "http": {
    "enabled": true,
    "port": 80,
    "lan": true,
    "wan": false,
    "redirectToHttps": true
  },
  "https": {
    "enabled": true,
    "port": 443,
    "lan": true,
    "wan": true
  },
  "ssh": {
    "enabled": true,
    "port": 22,
    "lan": true,
    "wan": false,
    "idleTimeoutSeconds": 300,
    "passwordAuthentication": true,
    "publicKeyAuthentication": true,
    "userLockout": {
      "enabled": true,
      "failAttempts": 3,
      "lockoutSeconds": 300
    }
  }
}
```

## Trusted IP

### Get Trusted IP Configuration

```bash
GET /api?fields=trustedIp
```

### Update Trusted IP

```bash
PUT /api/trustedIp
Content-Type: application/json

{
  "enabled": true,
  "mode": "whitelist",
  "rules": [
    {
      "name": "Corporate Network",
      "ipMode": "subnet",
      "subnetSrcAddr": "10.0.0.0",
      "subnetSrcMask": "8",
      "rangeSrcAddrStart": "",
      "rangeSrcAddrEnd": "",
      "dstPort": "ANY",
      "protocol": "ANY"
    }
  ]
}
```

## RADIUS

### Get RADIUS Configuration

```bash
GET /api?fields=radius
```

### Update RADIUS Configuration

```bash
PUT /api/radius
Content-Type: application/json

{
  "authEnabled": true,
  "acctEnabled": false,
  "host": "radius.example.com",
  "authPort": 1812,
  "acctPort": 1813,
  "secret": "MySecret",
  "authType": "PAP",
  "timeout": 2,
  "retries": 3,
  "checkServer": true,
  "anonymousEnabled": false,
  "anonymousId": "anonymous"
}
```

---

# Security Best Practices

## TLS/SSL Configuration

1. **Disable Old Protocols**: Use only TLS 1.2 and TLS 1.3
2. **Strong Ciphers**: Prefer AEAD ciphers (GCM, ChaCha20-Poly1305)
3. **Forward Secrecy**: Use ECDHE key exchange
4. **Cipher Order**: List strongest ciphers first
5. **Regular Updates**: Review and update cipher suites quarterly

### Recommended TLS 1.2 Cipher Suite
```
ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256
```

### Recommended TLS 1.3 Cipher Suite
```
TLS_AES_256_GCM_SHA384:TLS_AES_128_GCM_SHA256
```

## SSH Configuration

1. **Modern Ciphers**: Use ChaCha20-Poly1305 or AES-GCM
2. **Strong MACs**: Use SHA2-based MACs with ETM
3. **Key-Based Auth**: Prefer public key over password
4. **Disable Root**: Never allow direct root login
5. **Rate Limiting**: Enable user lockout

### Recommended SSH Ciphers
```
chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
```

### Recommended SSH MACs
```
hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
```

## Remote Access

1. **HTTPS Only**: Disable HTTP or redirect to HTTPS
2. **WAN Access**: Minimize WAN exposure, use VPN instead
3. **SSH from LAN**: Restrict SSH to LAN unless necessary
4. **Strong Timeouts**: Use reasonable session timeouts (5-10 minutes)
5. **Brute Force Protection**: Always enable with 3-5 attempt limit

## Trusted IP

1. **Whitelist Mode**: Prefer whitelist over blacklist
2. **Specific Rules**: Be as specific as possible with IPs and ports
3. **Regular Review**: Audit trusted IP rules monthly
4. **Document Rules**: Use descriptive names
5. **Test Changes**: Verify access before saving

## RADIUS

1. **Strong Secrets**: Use 20+ character random secrets
2. **Secure Protocols**: Use EAP-TTLS or PEAP, avoid PAP
3. **Certificate Validation**: Enable checkServer
4. **Failover**: Configure secondary server
5. **Accounting**: Enable for audit trails

---

# Troubleshooting

## TLS/SSL Issues

**Problem**: Cannot connect via HTTPS

**Check**:
- TLS version compatibility
- Cipher suite compatibility
- Certificate validity
- Client supports configured ciphers

## SSH Issues

**Problem**: SSH connection refused

**Check**:
- SSH is enabled
- Port is correct
- LAN/WAN access is enabled for source
- Firewall allows SSH port
- User lockout not active

## Trusted IP Issues

**Problem**: Cannot access device after enabling trusted IP

**Solution**:
- Access device via serial console
- Disable trusted IP or add your IP
- Always test rules before saving

**Prevention**:
- Add your current IP before enabling
- Use whitelist mode carefully
- Test from multiple locations

## RADIUS Issues

**Problem**: RADIUS authentication fails

**Check**:
- RADIUS server is reachable
- Shared secret matches
- Auth type is compatible
- User exists on RADIUS server
- Firewall allows UDP 1812/1813

**Debug**:
```bash
# Check RADIUS connectivity
GET /api?fields=eventlog
# Look for RADIUS-related errors
```

---

# Related Resources

- [User Management](API-README.md#user-endpoints)
- [Firewall Configuration](NAT-FIREWALL-SCHEMAS.md)
- [Network Interfaces](NETWORK-INTERFACES-SCHEMA.md)
- [Main API Documentation](API-README.md)

---

**Last Updated**: December 17, 2025  
**Schema Versions**: secureProtocols (2), remoteAccess (1), trustedIp (1), radius (1)
