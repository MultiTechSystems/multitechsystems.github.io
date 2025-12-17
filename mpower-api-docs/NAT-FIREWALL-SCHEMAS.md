# NAT, Firewall, Filter, and Routing Schemas

Complete schema documentation for network security and routing resources: `nat`, `firewall`, `filters`, and `routes`.

---

# NAT Rules

## Overview

NAT (Network Address Translation) rules control how IP addresses and ports are translated as packets traverse the device. Supports port forwarding (DNAT), source NAT (SNAT), and masquerading.

## Structure

The `nat` resource is an array of NAT rule configurations:

```json
[
  {
    // NAT rule configuration
  }
]
```

## NAT Rule Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `__v` | integer | No | Schema version (2) |
| `name` | string | Yes | Unique rule name (max 250 chars) |
| `description` | string | No | Rule description (max 250 chars) |
| `enabled` | boolean | No | Enable/disable rule |
| `guid` | string | Yes | Rule ID (auto-generated) |
| `protocol` | string | No | Protocol: "TCP", "UDP", "TCP/UDP", "ANY" |
| `chain` | string | No | Chain: "PREROUTING" or "POSTROUTING" |
| `target` | string | Yes | NAT type: "DNAT", "SNAT", "MASQUERADE" |

### Source Matching

| Property | Type | Description |
|----------|------|-------------|
| `srcInterface` | string | Source interface: "ANY", "WAN", "LAN", "CELLULAR", "WIFI", "WIFI_AP", "ETHERNET", "OPENVPN" |
| `srcSpecInterface` | string | Specific interface name (for OPENVPN) |
| `srcAddr` | string | Source IP address or "ANY" |
| `srcMask` | string/integer | Source subnet mask (CIDR or dotted decimal) |
| `srcPort` | string | Source port(s): "80", "8000:8080", "80,443,8080" |
| `srcMac` | string | Source MAC address (not for POSTROUTING) |

### Destination Matching

| Property | Type | Description |
|----------|------|-------------|
| `dstInterface` | string | Destination interface: "ANY", "WAN", "LAN", etc. |
| `dstSpecInterface` | string | Specific interface name (for OPENVPN) |
| `dstAddr` | string | Destination IP address or "ANY" |
| `dstMask` | string/integer | Destination subnet mask |
| `dstPort` | string | Destination port(s) |

### NAT Translation

| Property | Type | Description |
|----------|------|-------------|
| `toAddr` | string | Translate to this IP address (required for DNAT/SNAT) |
| `toPort` | string | Translate to this port (optional) |
| `enableNatLoopback` | boolean | Enable NAT loopback (DNAT only) |

## NAT Types

### DNAT (Destination NAT / Port Forwarding)

Translates destination address/port - used for port forwarding:
- Chain: PREROUTING
- Requires: `toAddr` or `toPort`
- Use case: Forward external port to internal server

### SNAT (Source NAT)

Translates source address:
- Chain: POSTROUTING
- Requires: `toAddr`
- Use case: Force specific source IP for outbound traffic

### MASQUERADE

Dynamic source NAT using interface IP:
- Chain: POSTROUTING
- No `toAddr` needed (uses interface IP)
- Use case: Internet access for LAN clients

## NAT Examples

### Example 1: Port Forwarding (DNAT)

Forward external port 8080 to internal web server:

```json
{
  "__v": 2,
  "name": "Web Server Forward",
  "description": "Forward port 8080 to internal server",
  "enabled": true,
  "guid": "auto-generated",
  "protocol": "TCP",
  "chain": "PREROUTING",
  "target": "DNAT",
  "srcInterface": "WAN",
  "srcAddr": "ANY",
  "srcPort": "",
  "dstInterface": "ANY",
  "dstAddr": "",
  "dstPort": "8080",
  "toAddr": "192.168.1.100",
  "toPort": "80",
  "enableNatLoopback": true
}
```

### Example 2: Multiple Port Forwarding

Forward SSH and HTTPS:

```json
[
  {
    "name": "SSH Forward",
    "enabled": true,
    "guid": "guid-1",
    "protocol": "TCP",
    "chain": "PREROUTING",
    "target": "DNAT",
    "srcInterface": "WAN",
    "dstPort": "22",
    "toAddr": "192.168.1.50",
    "toPort": "22"
  },
  {
    "name": "HTTPS Forward",
    "enabled": true,
    "guid": "guid-2",
    "protocol": "TCP",
    "chain": "PREROUTING",
    "target": "DNAT",
    "srcInterface": "WAN",
    "dstPort": "443",
    "toAddr": "192.168.1.100",
    "toPort": "443"
  }
]
```

### Example 3: Source NAT (SNAT)

Force specific source IP for outbound traffic:

```json
{
  "name": "Force Source IP",
  "enabled": true,
  "guid": "guid-3",
  "protocol": "ANY",
  "chain": "POSTROUTING",
  "target": "SNAT",
  "srcInterface": "LAN",
  "srcAddr": "192.168.1.0",
  "srcMask": 24,
  "dstInterface": "WAN",
  "toAddr": "203.0.113.50"
}
```

### Example 4: Masquerading

Enable internet access for LAN:

```json
{
  "name": "LAN Masquerade",
  "enabled": true,
  "guid": "guid-4",
  "protocol": "ANY",
  "chain": "POSTROUTING",
  "target": "MASQUERADE",
  "srcInterface": "LAN",
  "dstInterface": "WAN",
  "toAddr": ""
}
```

---

# Firewall Settings

## Overview

The `firewall` resource configures global firewall settings and protocol helpers.

## Structure

```json
{
  "pptp": boolean,
  "l2tp": boolean,
  "connTrackHelpers": boolean
}
```

## Firewall Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `pptp` | boolean | Yes | Enable PPTP passthrough (default: false) |
| `l2tp` | boolean | Yes | Enable L2TP passthrough (default: false) |
| `connTrackHelpers` | boolean | Yes | Enable connection tracking helpers (default: false) |

### Connection Tracking Helpers

When enabled, provides connection tracking for protocols that use separate control and data flows:
- FTP (control on port 21, data on dynamic ports)
- SIP (VoIP signaling)
- H.323 (video conferencing)
- TFTP
- IRC

## Firewall Example

```json
{
  "pptp": true,
  "l2tp": true,
  "connTrackHelpers": true
}
```

---

# Filter Rules

## Overview

Filter rules control packet filtering (allow/deny/log traffic) through the device firewall.

## Structure

The `filters` resource is an array of filter rule configurations:

```json
[
  {
    // Filter rule configuration
  }
]
```

## Filter Rule Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `__v` | integer | No | Schema version (2) |
| `name` | string | Yes | Unique rule name (max 250 chars) |
| `description` | string | No | Rule description (max 250 chars) |
| `enabled` | boolean | Yes | Enable/disable rule |
| `protocol` | string | No | Protocol: "TCP", "UDP", "TCP/UDP", "ANY" |
| `chain` | string | Yes | Chain: "INPUT", "OUTPUT", "FORWARD" |
| `target` | string | No | Action: "ACCEPT", "REJECT", "LOG", "DROP" |

### Source Matching (same as NAT)

| Property | Type | Description |
|----------|------|-------------|
| `srcInterface` | string | Source interface |
| `srcSpecInterface` | string | Specific interface name (for OPENVPN) |
| `srcAddr` | string | Source IP address or "ANY" |
| `srcMask` | string/integer | Source subnet mask |
| `srcPort` | string | Source port(s) |
| `srcMac` | string | Source MAC (not for OUTPUT chain) |

### Destination Matching (same as NAT)

| Property | Type | Description |
|----------|------|-------------|
| `dstInterface` | string | Destination interface |
| `dstSpecInterface` | string | Specific interface name (for OPENVPN) |
| `dstAddr` | string | Destination IP address or "ANY" |
| `dstMask` | string/integer | Destination subnet mask |
| `dstPort` | string | Destination port(s) |

## Firewall Chains

| Chain | Description | Use Case |
|-------|-------------|----------|
| `INPUT` | Traffic destined for device itself | Allow SSH, HTTPS to device |
| `OUTPUT` | Traffic originating from device | Control device's outbound traffic |
| `FORWARD` | Traffic passing through device | Control LAN to WAN traffic |

## Filter Targets

| Target | Description | Behavior |
|--------|-------------|----------|
| `ACCEPT` | Allow traffic | Packet is allowed through |
| `REJECT` | Reject traffic | Packet is dropped, sender is notified |
| `DROP` | Drop traffic | Packet is silently dropped |
| `LOG` | Log traffic | Packet is logged, then continues to next rule |

## Filter Examples

### Example 1: Allow SSH to Device

```json
{
  "__v": 2,
  "name": "Allow SSH",
  "description": "Allow SSH access to device",
  "enabled": true,
  "protocol": "TCP",
  "chain": "INPUT",
  "target": "ACCEPT",
  "srcInterface": "LAN",
  "srcAddr": "192.168.1.0",
  "srcMask": 24,
  "dstPort": "22"
}
```

### Example 2: Block Outbound Traffic

```json
{
  "name": "Block Facebook",
  "enabled": true,
  "protocol": "ANY",
  "chain": "FORWARD",
  "target": "REJECT",
  "srcInterface": "LAN",
  "dstAddr": "157.240.0.0",
  "dstMask": 16
}
```

### Example 3: Allow Specific Service

```json
{
  "name": "Allow HTTPS",
  "enabled": true,
  "protocol": "TCP",
  "chain": "FORWARD",
  "target": "ACCEPT",
  "srcInterface": "LAN",
  "dstInterface": "WAN",
  "dstPort": "443"
}
```

### Example 4: Log Suspicious Traffic

```json
{
  "name": "Log Port Scan",
  "enabled": true,
  "protocol": "TCP",
  "chain": "INPUT",
  "target": "LOG",
  "srcInterface": "WAN",
  "dstPort": "1:1024"
}
```

---

# Static Routes

## Overview

The `routes` resource configures static routing table entries.

## Structure

The `routes` resource is an array of route configurations:

```json
[
  {
    // Route configuration
  }
]
```

## Route Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Route name (max 250 chars) |
| `ip` | string | Yes | Destination network IP address |
| `mask` | string/integer | Yes | Subnet mask (CIDR or dotted decimal, 0-32) |
| `gateway` | string | Yes | Gateway IP address |

## Route Examples

### Example 1: Route to Remote Network

```json
{
  "name": "HQ Network",
  "ip": "10.0.0.0",
  "mask": 16,
  "gateway": "192.168.1.254"
}
```

### Example 2: Multiple Static Routes

```json
[
  {
    "name": "Branch Office 1",
    "ip": "172.16.0.0",
    "mask": "255.255.0.0",
    "gateway": "192.168.1.1"
  },
  {
    "name": "Branch Office 2",
    "ip": "172.17.0.0",
    "mask": 16,
    "gateway": "192.168.1.2"
  },
  {
    "name": "Cloud Network",
    "ip": "10.100.0.0",
    "mask": 24,
    "gateway": "192.168.1.10"
  }
]
```

### Example 3: Default Route

```json
{
  "name": "Default via Cellular",
  "ip": "0.0.0.0",
  "mask": 0,
  "gateway": "10.64.64.64"
}
```

---

# API Operations

## NAT Rules

### Get NAT Rules

```bash
GET /api?fields=nat
```

### Create NAT Rule

```bash
POST /api/nat
Content-Type: application/json

{
  "name": "Port Forward",
  "enabled": true,
  "guid": "",
  "protocol": "TCP",
  "chain": "PREROUTING",
  "target": "DNAT",
  "srcInterface": "WAN",
  "dstPort": "8080",
  "toAddr": "192.168.1.100",
  "toPort": "80"
}
```

### Update NAT Rule

```bash
PUT /api/nat/{index}
```

### Delete NAT Rule

```bash
DELETE /api/nat/{index}
```

## Firewall Settings

### Get Firewall Settings

```bash
GET /api?fields=firewall
```

### Update Firewall Settings

```bash
PUT /api/firewall
Content-Type: application/json

{
  "pptp": true,
  "l2tp": true,
  "connTrackHelpers": true
}
```

## Filter Rules

### Get Filter Rules

```bash
GET /api?fields=filters
```

### Create Filter Rule

```bash
POST /api/filters
Content-Type: application/json

{
  "name": "Allow SSH",
  "enabled": true,
  "protocol": "TCP",
  "chain": "INPUT",
  "target": "ACCEPT",
  "srcInterface": "LAN",
  "dstPort": "22"
}
```

### Update Filter Rule

```bash
PUT /api/filters/{index}
```

### Delete Filter Rule

```bash
DELETE /api/filters/{index}
```

## Static Routes

### Get Routes

```bash
GET /api?fields=routes
```

### Create Route

```bash
POST /api/routes
Content-Type: application/json

{
  "name": "Remote Network",
  "ip": "10.0.0.0",
  "mask": 16,
  "gateway": "192.168.1.254"
}
```

### Update Route

```bash
PUT /api/routes/{index}
```

### Delete Route

```bash
DELETE /api/routes/{index}
```

---

# Common Use Cases

## Use Case 1: Port Forwarding for Web Server

Forward external port 80 to internal web server:

```bash
POST /api/nat
{
  "name": "HTTP Forward",
  "enabled": true,
  "guid": "",
  "protocol": "TCP",
  "chain": "PREROUTING",
  "target": "DNAT",
  "srcInterface": "WAN",
  "dstPort": "80",
  "toAddr": "192.168.1.100",
  "toPort": "80",
  "enableNatLoopback": true
}
```

## Use Case 2: Multiple Port Forwarding

Forward multiple services to different internal servers:

```bash
# Web server (HTTP/HTTPS)
POST /api/nat
{
  "name": "Web HTTP",
  "enabled": true,
  "protocol": "TCP",
  "chain": "PREROUTING",
  "target": "DNAT",
  "srcInterface": "WAN",
  "dstPort": "80",
  "toAddr": "192.168.1.100",
  "toPort": "80"
}

# Mail server (SMTP)
POST /api/nat
{
  "name": "Mail SMTP",
  "enabled": true,
  "protocol": "TCP",
  "chain": "PREROUTING",
  "target": "DNAT",
  "srcInterface": "WAN",
  "dstPort": "25",
  "toAddr": "192.168.1.101",
  "toPort": "25"
}
```

## Use Case 3: Restrict SSH Access

Allow SSH only from specific subnet:

```bash
POST /api/filters
{
  "name": "Allow SSH from Admin",
  "enabled": true,
  "protocol": "TCP",
  "chain": "INPUT",
  "target": "ACCEPT",
  "srcInterface": "LAN",
  "srcAddr": "192.168.1.0",
  "srcMask": 24,
  "dstPort": "22"
}

# Block all other SSH
POST /api/filters
{
  "name": "Block SSH Others",
  "enabled": true,
  "protocol": "TCP",
  "chain": "INPUT",
  "target": "DROP",
  "srcInterface": "ANY",
  "dstPort": "22"
}
```

## Use Case 4: Block Specific Websites

Block access to social media:

```bash
POST /api/filters
{
  "name": "Block Social Media",
  "enabled": true,
  "protocol": "ANY",
  "chain": "FORWARD",
  "target": "REJECT",
  "srcInterface": "LAN",
  "dstAddr": "157.240.0.0",
  "dstMask": 16
}
```

## Use Case 5: VPN Routing

Route specific traffic through VPN tunnel:

```bash
POST /api/routes
{
  "name": "Corporate Network",
  "ip": "10.0.0.0",
  "mask": 8,
  "gateway": "10.255.255.2"
}
```

---

# Important Notes

## Port Specification Format

Ports can be specified in multiple formats:
- **Single port**: `"80"`
- **Port range**: `"8000:8080"` or `"8000-8080"`
- **Multiple ports**: `"80,443,8080"`
- **Mixed**: `"80,443,8000:8080,9000-9100"`
- **Any**: `"ANY"` or `""`

## IP Address Formats

- **Specific IP**: `"192.168.1.100"`
- **Any IP**: `"ANY"` or `""`
- **Zero IP**: `"0.0.0.0"` (allowed)

## Subnet Mask Formats

- **CIDR notation**: `24` or `"24"`
- **Dotted decimal**: `"255.255.255.0"`
- **Any**: `""` (no mask)

## MAC Address Format

- **Standard**: `"00:11:22:33:44:55"`
- **Any**: `"ANY"` or `""`

## Interface Types

- **ANY**: Match any interface
- **WAN**: All WAN interfaces
- **LAN**: All LAN interfaces
- **CELLULAR**: Cellular interface (ppp0)
- **WIFI**: WiFi client interface
- **WIFI_AP**: WiFi access point
- **ETHERNET**: Ethernet interfaces
- **OPENVPN**: OpenVPN tunnels (requires srcSpecInterface/dstSpecInterface)

## NAT Chains

- **PREROUTING**: Before routing decision (for DNAT/port forwarding)
- **POSTROUTING**: After routing decision (for SNAT/masquerading)

## Filter Chains

- **INPUT**: Traffic to device itself
- **OUTPUT**: Traffic from device itself
- **FORWARD**: Traffic passing through device

## Rule Order

- Rules are processed in order
- First matching rule wins
- Place more specific rules before general rules
- Use LOG target to debug rule matching

## NAT Loopback

`enableNatLoopback` allows LAN clients to access forwarded services using WAN IP:
- Only applicable for DNAT rules
- Enables hairpin NAT
- Useful for accessing internal servers via public IP from LAN

---

# Configuration Workflows

## Workflow 1: Setup Port Forwarding

```bash
# 1. Create DNAT rule
curl -X POST http://192.168.2.1/api/nat \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "name": "Web Server",
    "enabled": true,
    "guid": "",
    "protocol": "TCP",
    "chain": "PREROUTING",
    "target": "DNAT",
    "srcInterface": "WAN",
    "dstPort": "80",
    "toAddr": "192.168.1.100",
    "toPort": "80",
    "enableNatLoopback": true
  }'

# 2. Create filter rule to allow forwarded traffic
curl -X POST http://192.168.2.1/api/filters \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "name": "Allow Web Forward",
    "enabled": true,
    "protocol": "TCP",
    "chain": "FORWARD",
    "target": "ACCEPT",
    "srcInterface": "WAN",
    "dstAddr": "192.168.1.100",
    "dstPort": "80"
  }'

# 3. Save
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

## Workflow 2: Setup Static Route for VPN

```bash
# 1. Create static route
curl -X POST http://192.168.2.1/api/routes \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "name": "VPN Network",
    "ip": "10.0.0.0",
    "mask": 16,
    "gateway": "10.255.255.2"
  }'

# 2. Create filter to allow VPN traffic
curl -X POST http://192.168.2.1/api/filters \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "name": "Allow VPN Traffic",
    "enabled": true,
    "protocol": "ANY",
    "chain": "FORWARD",
    "target": "ACCEPT",
    "srcInterface": "LAN",
    "dstAddr": "10.0.0.0",
    "dstMask": 16
  }'

# 3. Save
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

---

# Troubleshooting

## Port Forwarding Not Working

**Check:**
1. NAT rule is enabled
2. Protocol matches (TCP/UDP)
3. Ports are correct
4. `toAddr` points to correct internal IP
5. Filter rule allows FORWARD traffic
6. Internal server is running and listening
7. No firewall on internal server blocking traffic

**Debug:**
```bash
# Check NAT rules
GET /api?fields=nat

# Check filter rules
GET /api?fields=filters

# Test from external IP
telnet <wan-ip> <port>
```

## Filter Rules Not Working

**Check:**
1. Rule is enabled
2. Chain is correct (INPUT/OUTPUT/FORWARD)
3. Rule order (more specific before general)
4. Source/destination matching is correct
5. Protocol matches traffic type

**Debug:**
- Add LOG rule before problem rule to see if traffic matches
- Check event log for firewall logs

## Static Routes Not Working

**Check:**
1. Gateway is reachable
2. Gateway is on same subnet as interface
3. No conflicting routes
4. Routing table: `GET /api?fields=stats/routes`

## NAT Loopback Issues

**Problem**: Cannot access forwarded service from LAN using WAN IP

**Solution**:
- Enable `enableNatLoopback: true` in DNAT rule
- Ensure filter rules allow the traffic

---

# Security Best Practices

## NAT Rules

1. **Minimize exposure**: Only forward necessary ports
2. **Use specific ports**: Avoid forwarding port ranges
3. **Document rules**: Use descriptive names and descriptions
4. **Regular review**: Audit NAT rules periodically
5. **Disable unused**: Disable rather than delete for audit trail

## Filter Rules

1. **Default deny**: Block all, then allow specific traffic
2. **Specific rules first**: Place specific rules before general ones
3. **Use LOG judiciously**: Too much logging impacts performance
4. **Review regularly**: Audit filter rules quarterly
5. **Test changes**: Test in staging before production

## Routing

1. **Avoid conflicts**: Ensure routes don't overlap incorrectly
2. **Use specific masks**: Avoid overly broad routes
3. **Document purpose**: Clear names for each route
4. **Monitor reachability**: Use ping alerts for critical routes

## General

1. **Test after changes**: Verify connectivity after firewall changes
2. **Keep backup**: Save config before major changes
3. **Use VPN**: Secure remote access instead of exposing services
4. **Monitor logs**: Review firewall logs regularly
5. **Update firmware**: Keep device firmware current

---

# Related Resources

- [Network Interfaces](NETWORK-INTERFACES-SCHEMA.md) - Interface configuration
- [VPN Configuration](VPN-SCHEMAS.md) - VPN tunnels
- [Main API Documentation](API-README.md)

---

**Last Updated**: December 17, 2025  
**Schema Versions**: nat (2), firewall (1), filters (2), routes (1)
