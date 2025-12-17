# VPN Tunnel Configuration Schemas

Complete schema documentation for VPN tunnel resources in the mPower Device API: `ipsecTunnels`, `greTunnels`, and `ovpnTunnels`.

## Overview

The mPower Device API supports three types of VPN tunnels:

1. **IPsec Tunnels** - Secure site-to-site VPN using IPsec/IKE protocols
2. **GRE Tunnels** - Generic Routing Encapsulation for IP-in-IP tunneling
3. **OpenVPN Tunnels** - Flexible SSL/TLS-based VPN solution

---

# IPsec Tunnels

## Overview

IPsec tunnels provide secure site-to-site VPN connections using Internet Key Exchange (IKE) protocols. Supports both IKEv1 and IKEv2 with pre-shared keys or certificate-based authentication.

## Structure

The `ipsecTunnels` resource is an array of tunnel configurations:

```json
[
  {
    // IPsec tunnel configuration
  }
]
```

## IPsec Tunnel Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `enabled` | boolean | Yes | Enable/disable tunnel |
| `name` | string | Yes | Tunnel label (max 15 chars, alphanumeric) |
| `description` | string | Yes | Tunnel description (max 250 chars) |
| `remoteWanIp` | string | Yes | Remote peer public IP or hostname |
| `type` | string | Yes | IKE version: "IKE" (v1) or "IKEv2" |
| `allowAllTraffic` | boolean | Yes | Allow all traffic without subnet filtering |
| `localSubnets` | array | Yes | Local networks accessible through tunnel |
| `remoteSubnets` | array | Yes | Remote networks accessible through tunnel |
| `authMethod` | string | Yes | Authentication: "secret" (PSK) or "rsasig" (certificates) |
| `psk` | string | Conditional | Pre-shared key (required if authMethod = "secret") |
| `caCert` | string | Conditional | CA certificate PEM (required if authMethod = "rsasig") |
| `localCert` | string | Conditional | Local certificate PEM (required if authMethod = "rsasig") |
| `localPrivKey` | string | Conditional | Local private key PEM (required if authMethod = "rsasig") |
| `uid` | boolean | Yes | Use unique identifiers for authentication |
| `localId` | string | Yes | Local peer identifier |
| `remoteId` | string | Yes | Remote peer identifier |
| `encryptMethod` | string | Yes | Encryption: "3des", "aes", "aes128", "aes192", "aes256", or "" |
| `phase1` | string | No | Phase 1 algorithms (e.g., "aes128-sha1;modp1024") |
| `phase2` | string | No | Phase 2 algorithms (e.g., "aes128-sha1;modp1024") |
| `ikelife` | integer | Yes | IKE SA lifetime in hours |
| `keylife` | integer | Yes | IPsec SA lifetime in hours |
| `retries` | integer | Yes | Connection retry attempts |
| `checkPeriod` | integer | No | Hostname resolution interval in minutes |
| `compression` | boolean | Yes | Enable IPComp compression |
| `aggressiveMode` | boolean | No | Enable aggressive mode (IKEv1 only, less secure) |
| `perfectForwardSecrecy` | boolean | Yes | Enable Perfect Forward Secrecy (PFS) |
| `natTraversal` | boolean | No | Enable NAT traversal (NAT-T) |

### Subnet Object

| Property | Type | Description |
|----------|------|-------------|
| `ip` | string | Network IP address |
| `mask` | integer | Subnet mask (1-32) |

## IPsec Examples

### Example 1: IKEv2 with Pre-Shared Key

```json
{
  "enabled": true,
  "name": "HQ-Branch",
  "description": "VPN to headquarters",
  "remoteWanIp": "203.0.113.50",
  "type": "IKEv2",
  "allowAllTraffic": false,
  "localSubnets": [
    {"ip": "192.168.1.0", "mask": 24}
  ],
  "remoteSubnets": [
    {"ip": "10.0.0.0", "mask": 16}
  ],
  "authMethod": "secret",
  "psk": "MySecurePreSharedKey123!",
  "caCert": "",
  "localCert": "",
  "localPrivKey": "",
  "uid": true,
  "localId": "branch@example.com",
  "remoteId": "hq@example.com",
  "encryptMethod": "aes256",
  "phase1": "",
  "phase2": "",
  "ikelife": 8,
  "keylife": 1,
  "retries": 3,
  "checkPeriod": 0,
  "compression": false,
  "perfectForwardSecrecy": true,
  "natTraversal": true
}
```

### Example 2: IKEv1 with Certificates

```json
{
  "enabled": true,
  "name": "Partner-VPN",
  "description": "VPN to partner network",
  "remoteWanIp": "198.51.100.25",
  "type": "IKE",
  "allowAllTraffic": false,
  "localSubnets": [
    {"ip": "192.168.10.0", "mask": 24}
  ],
  "remoteSubnets": [
    {"ip": "172.16.0.0", "mask": 16}
  ],
  "authMethod": "rsasig",
  "psk": "",
  "caCert": "-----BEGIN CERTIFICATE-----\nMIID...\n-----END CERTIFICATE-----",
  "localCert": "-----BEGIN CERTIFICATE-----\nMIID...\n-----END CERTIFICATE-----",
  "localPrivKey": "-----BEGIN PRIVATE KEY-----\nMIIE...\n-----END PRIVATE KEY-----",
  "uid": true,
  "localId": "C=US, O=MyCompany, CN=gateway1",
  "remoteId": "C=US, O=Partner, CN=gateway2",
  "encryptMethod": "aes128",
  "phase1": "aes128-sha256;modp2048",
  "phase2": "aes128-sha256;modp2048",
  "ikelife": 24,
  "keylife": 8,
  "retries": 5,
  "checkPeriod": 0,
  "compression": false,
  "aggressiveMode": false,
  "perfectForwardSecrecy": true,
  "natTraversal": false
}
```

---

# GRE Tunnels

## Overview

GRE (Generic Routing Encapsulation) tunnels provide IP-in-IP tunneling for routing between networks. GRE tunnels are not encrypted by default but can be combined with IPsec for security.

## Structure

The `greTunnels` resource is an array of tunnel configurations:

```json
[
  {
    // GRE tunnel configuration
  }
]
```

## GRE Tunnel Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `enabled` | boolean | Yes | Enable/disable tunnel |
| `name` | string | Yes | Tunnel name (max 15 chars, alphanumeric when enabled) |
| `description` | string | Yes | Tunnel description (max 250 chars) |
| `remoteIp` | string | Yes | Remote tunnel endpoint IP or hostname |
| `ttl` | integer | Yes | Time to live (0 = inherit, 1-255) |
| `checkPeriod` | integer | Yes | Hostname resolution interval in minutes (1-120) |
| `tunnelIp` | string | Yes | Local tunnel interface IP address |
| `tunnelMask` | string | Yes | Tunnel interface subnet mask |
| `routes` | array | Yes | Static routes through tunnel |

### Route Object

| Property | Type | Description |
|----------|------|-------------|
| `ip` | string | Destination network IP |
| `mask` | integer/string | Subnet mask (CIDR or dotted decimal) |

## GRE Examples

### Example 1: Basic GRE Tunnel

```json
{
  "enabled": true,
  "name": "gre-hq",
  "description": "GRE tunnel to headquarters",
  "remoteIp": "203.0.113.100",
  "ttl": 64,
  "checkPeriod": 5,
  "tunnelIp": "10.255.255.1",
  "tunnelMask": "255.255.255.252",
  "routes": [
    {"ip": "10.0.0.0", "mask": 16},
    {"ip": "172.16.0.0", "mask": 12}
  ]
}
```

### Example 2: GRE with Dynamic DNS

```json
{
  "enabled": true,
  "name": "gre-remote",
  "description": "GRE to remote site with DDNS",
  "remoteIp": "remote.example.com",
  "ttl": 0,
  "checkPeriod": 10,
  "tunnelIp": "10.255.255.5",
  "tunnelMask": "255.255.255.252",
  "routes": [
    {"ip": "192.168.50.0", "mask": 24}
  ]
}
```

### Example 3: GRE without Tunnel IP (Point-to-Point)

```json
{
  "enabled": true,
  "name": "gre-p2p",
  "description": "Point-to-point GRE tunnel",
  "remoteIp": "198.51.100.75",
  "ttl": 64,
  "checkPeriod": 5,
  "tunnelIp": "",
  "tunnelMask": "",
  "routes": [
    {"ip": "172.20.0.0", "mask": "255.255.0.0"}
  ]
}
```

---

# OpenVPN Tunnels

## Overview

OpenVPN tunnels provide flexible SSL/TLS-based VPN connections. The device can act as an OpenVPN server, client, or use custom configurations.

## Structure

The `ovpnTunnels` resource is an array of tunnel configurations:

```json
[
  {
    // OpenVPN tunnel configuration
  }
]
```

## OpenVPN Tunnel Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `enabled` | boolean | Yes | Enable/disable tunnel |
| `name` | string | Yes | Tunnel instance name (no spaces, max 250 chars) |
| `description` | string | Yes | Tunnel description |
| `type` | string | Yes | Tunnel type: "SERVER", "CLIENT", or "CUSTOM" |
| `config` | string | Yes | OpenVPN configuration file content (max 60000 chars) |

## OpenVPN Types

| Type | Description | Use Case |
|------|-------------|----------|
| `SERVER` | OpenVPN server mode | Accept incoming VPN connections |
| `CLIENT` | OpenVPN client mode | Connect to external OpenVPN server |
| `CUSTOM` | Custom configuration | Advanced/manual OpenVPN configuration |

## OpenVPN Examples

### Example 1: OpenVPN Server

```json
{
  "enabled": true,
  "name": "ovpn-server",
  "description": "OpenVPN server for remote access",
  "type": "SERVER",
  "config": "port 1194\nproto udp\ndev tun\nca ca.crt\ncert server.crt\nkey server.key\ndh dh2048.pem\nserver 10.8.0.0 255.255.255.0\npush \"route 192.168.1.0 255.255.255.0\"\nkeepalive 10 120\ncipher AES-256-CBC\nauth SHA256\ncomp-lzo\nuser nobody\ngroup nogroup\npersist-key\npersist-tun\nstatus /var/log/openvpn-status.log\nverb 3"
}
```

### Example 2: OpenVPN Client

```json
{
  "enabled": true,
  "name": "ovpn-client",
  "description": "Connect to corporate VPN",
  "type": "CLIENT",
  "config": "client\ndev tun\nproto udp\nremote vpn.example.com 1194\nresolv-retry infinite\nnobind\npersist-key\npersist-tun\nca ca.crt\ncert client.crt\nkey client.key\nremote-cert-tls server\ncipher AES-256-CBC\nauth SHA256\ncomp-lzo\nverb 3"
}
```

### Example 3: Custom OpenVPN Configuration

```json
{
  "enabled": true,
  "name": "ovpn-custom",
  "description": "Custom OpenVPN with specific routing",
  "type": "CUSTOM",
  "config": "mode p2p\nremote 203.0.113.50\nport 1194\nproto udp\ndev tun\nifconfig 10.9.0.1 10.9.0.2\nsecret static.key\ncipher AES-256-CBC\nauth SHA512\ncomp-lzo\nkeepalive 10 60\nverb 3\nroute 10.0.0.0 255.255.0.0\nroute 172.16.0.0 255.240.0.0"
}
```

---

# API Operations

## IPsec Tunnels

### Get IPsec Tunnels

```bash
GET /api?fields=ipsecTunnels
```

### Create IPsec Tunnel

```bash
POST /api/ipsecTunnels
Content-Type: application/json

{
  "enabled": true,
  "name": "tunnel1",
  "description": "Site-to-site VPN",
  "remoteWanIp": "203.0.113.50",
  "type": "IKEv2",
  "allowAllTraffic": false,
  "localSubnets": [{"ip": "192.168.1.0", "mask": 24}],
  "remoteSubnets": [{"ip": "10.0.0.0", "mask": 16}],
  "authMethod": "secret",
  "psk": "MySecretKey",
  "uid": true,
  "localId": "local@example.com",
  "remoteId": "remote@example.com",
  "encryptMethod": "aes256",
  "ikelife": 8,
  "keylife": 1,
  "retries": 3,
  "compression": false,
  "perfectForwardSecrecy": true
}
```

### Update IPsec Tunnel

```bash
PUT /api/ipsecTunnels/{index}
Content-Type: application/json

{
  "enabled": true,
  "description": "Updated description",
  ...
}
```

### Delete IPsec Tunnel

```bash
DELETE /api/ipsecTunnels/{index}
```

## GRE Tunnels

### Get GRE Tunnels

```bash
GET /api?fields=greTunnels
```

### Create GRE Tunnel

```bash
POST /api/greTunnels
Content-Type: application/json

{
  "enabled": true,
  "name": "gre1",
  "description": "GRE to HQ",
  "remoteIp": "203.0.113.100",
  "ttl": 64,
  "checkPeriod": 5,
  "tunnelIp": "10.255.255.1",
  "tunnelMask": "255.255.255.252",
  "routes": [
    {"ip": "10.0.0.0", "mask": 16}
  ]
}
```

### Update GRE Tunnel

```bash
PUT /api/greTunnels/{index}
```

### Delete GRE Tunnel

```bash
DELETE /api/greTunnels/{index}
```

## OpenVPN Tunnels

### Get OpenVPN Tunnels

```bash
GET /api?fields=ovpnTunnels
```

### Create OpenVPN Tunnel

```bash
POST /api/ovpnTunnels
Content-Type: application/json

{
  "enabled": true,
  "name": "ovpn1",
  "description": "OpenVPN client",
  "type": "CLIENT",
  "config": "client\ndev tun\nproto udp\n..."
}
```

### Update OpenVPN Tunnel

```bash
PUT /api/ovpnTunnels/{index}
```

### Delete OpenVPN Tunnel

```bash
DELETE /api/ovpnTunnels/{index}
```

---

# Configuration Workflows

## Workflow 1: Create IPsec Site-to-Site VPN

```bash
# 1. Login
curl -X POST http://192.168.2.1/api/login \
  -d '{"username":"admin","password":"admin"}' -c cookies.txt

# 2. Create IPsec tunnel
curl -X POST http://192.168.2.1/api/ipsecTunnels \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "enabled": true,
    "name": "site-vpn",
    "description": "Site-to-site VPN",
    "remoteWanIp": "203.0.113.50",
    "type": "IKEv2",
    "allowAllTraffic": false,
    "localSubnets": [{"ip": "192.168.1.0", "mask": 24}],
    "remoteSubnets": [{"ip": "10.0.0.0", "mask": 16}],
    "authMethod": "secret",
    "psk": "StrongPreSharedKey123!",
    "uid": true,
    "localId": "local@example.com",
    "remoteId": "remote@example.com",
    "encryptMethod": "aes256",
    "ikelife": 8,
    "keylife": 1,
    "retries": 3,
    "compression": false,
    "perfectForwardSecrecy": true,
    "natTraversal": true
  }'

# 3. Save
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

## Workflow 2: Create GRE Tunnel

```bash
# Create GRE tunnel
curl -X POST http://192.168.2.1/api/greTunnels \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "enabled": true,
    "name": "gre-hq",
    "description": "GRE tunnel to HQ",
    "remoteIp": "203.0.113.100",
    "ttl": 64,
    "checkPeriod": 5,
    "tunnelIp": "10.255.255.1",
    "tunnelMask": "255.255.255.252",
    "routes": [
      {"ip": "10.0.0.0", "mask": 16}
    ]
  }'

# Save
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

## Workflow 3: Create OpenVPN Client

```bash
# Create OpenVPN client tunnel
curl -X POST http://192.168.2.1/api/ovpnTunnels \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "enabled": true,
    "name": "corporate-vpn",
    "description": "Corporate VPN access",
    "type": "CLIENT",
    "config": "client\ndev tun\nproto udp\nremote vpn.company.com 1194\nresolv-retry infinite\nnobind\npersist-key\npersist-tun\nca ca.crt\ncert client.crt\nkey client.key\nremote-cert-tls server\ncipher AES-256-CBC\nauth SHA256\nverb 3"
  }'

# Save
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

---

# Important Notes

## IPsec Tunnels

### Authentication Methods

- **Pre-Shared Key (secret)**: Simpler, requires matching PSK on both ends
- **Certificates (rsasig)**: More secure, requires PKI infrastructure

### IKE Versions

- **IKE (IKEv1)**: Older, widely supported, has aggressive mode option
- **IKEv2**: Modern, more efficient, better for mobile connections

### Encryption Methods

- **3des**: Legacy, less secure
- **aes**: Generic AES
- **aes128**: AES 128-bit (good balance)
- **aes192**: AES 192-bit
- **aes256**: AES 256-bit (most secure, slightly slower)

### Phase 1 & Phase 2 Algorithms

Format: `cipher-hash;group`

**Common combinations:**
- `aes128-sha1;modp1024` - Basic security
- `aes256-sha256;modp2048` - Strong security
- `aes256-sha512;modp4096` - Maximum security

### Subnet Configuration

- When enabled, must have at least 1 local and 1 remote subnet
- Use `allowAllTraffic: true` to bypass subnet filtering
- Multiple subnets supported for complex networks

### Identifiers

- Can be email format: `user@domain.com`
- Can be DN format: `C=US, O=Company, CN=gateway`
- Can be IP address: `192.168.1.1`
- Must not contain quotes or ` : ` (space-colon-space)

## GRE Tunnels

### TTL Configuration

- **0**: Inherit TTL from encapsulated packet
- **1-255**: Fixed TTL value
- Recommended: 64 for most scenarios

### Tunnel IP Configuration

- Can be empty (`""`) for point-to-point without addressing
- If set, both `tunnelIp` and `tunnelMask` must be provided
- Typically use /30 subnet (255.255.255.252) for point-to-point

### Dynamic DNS Support

- Use `checkPeriod` to periodically resolve hostname
- Useful when remote peer has dynamic IP
- Set to 5-10 minutes for DDNS scenarios

### Routing

- Routes define which traffic goes through tunnel
- Can have multiple routes per tunnel
- Mask can be CIDR (24) or dotted decimal (255.255.255.0)

## OpenVPN Tunnels

### Configuration Types

- **SERVER**: Device acts as OpenVPN server
- **CLIENT**: Device connects to external OpenVPN server
- **CUSTOM**: Full control over OpenVPN configuration

### Config Field

- Contains complete OpenVPN configuration file
- Maximum 60,000 characters
- Use newlines (`\n`) to separate directives
- Include all necessary certificates inline or reference files

### Common OpenVPN Directives

**Server mode:**
```
port 1194
proto udp
dev tun
server 10.8.0.0 255.255.255.0
push "route 192.168.1.0 255.255.255.0"
```

**Client mode:**
```
client
remote vpn.example.com 1194
dev tun
proto udp
```

**Security:**
```
cipher AES-256-CBC
auth SHA256
tls-auth ta.key 0
```

---

# Common Use Cases

## Use Case 1: Site-to-Site IPsec VPN

Connect two office locations securely:
- Use IKEv2 for better performance
- Use AES-256 encryption
- Enable Perfect Forward Secrecy
- Configure both local and remote subnets
- Use pre-shared key for simplicity or certificates for security

## Use Case 2: GRE over IPsec

Combine GRE and IPsec for secure routing:
1. Create GRE tunnel for routing flexibility
2. Create IPsec tunnel with `allowAllTraffic: true`
3. Configure IPsec to protect GRE traffic
4. Provides both encryption (IPsec) and routing (GRE)

## Use Case 3: Remote Access VPN

Provide VPN access for remote users:
- Configure OpenVPN server mode
- Use TLS authentication
- Push routes to clients
- Configure user certificates

## Use Case 4: Connect to Cloud VPN

Connect device to cloud provider VPN:
- Use IPsec for AWS/Azure VPN Gateway
- Use OpenVPN client for OpenVPN Access Server
- Configure appropriate subnets and routing

## Use Case 5: Backup WAN Connection

Use VPN as backup WAN:
- Configure VPN tunnel
- Set up routing with metrics
- Use with WAN manager for failover

---

# Troubleshooting

## IPsec Tunnel Issues

### Tunnel Not Connecting

**Check:**
- Both ends have matching configuration
- PSK matches exactly (case-sensitive)
- IKE version matches on both ends
- Encryption methods are compatible
- Firewall allows UDP 500 (IKE) and UDP 4500 (NAT-T)
- Remote peer is reachable

**Debug:**
```bash
# Check IPsec status
GET /api?fields=stats/ipsec

# View logs
GET /api?fields=eventlog
```

### Authentication Failures

**Check:**
- PSK matches on both ends
- Certificates are valid and not expired
- CA certificate is correct
- Local and remote IDs match configuration

### Traffic Not Flowing

**Check:**
- Tunnel is established (check status)
- Local and remote subnets are correct
- Routing is configured properly
- Firewall rules allow tunnel traffic
- `allowAllTraffic` setting if using custom firewall rules

## GRE Tunnel Issues

### Tunnel Not Establishing

**Check:**
- Remote IP is reachable
- Firewall allows IP protocol 47 (GRE)
- Tunnel IPs don't conflict with existing networks
- Routes are configured correctly

### Hostname Not Resolving

**Check:**
- DNS is configured properly
- `checkPeriod` is set appropriately
- Hostname is valid and resolves externally

### No Connectivity Through Tunnel

**Check:**
- Routes are configured on both ends
- Tunnel IPs are in same subnet
- Firewall allows traffic
- TTL is not too low

## OpenVPN Tunnel Issues

### OpenVPN Not Starting

**Check:**
- Configuration syntax is valid
- All referenced files/certificates exist
- Port is not already in use
- Device has necessary resources

**Validate config:**
```bash
# Test OpenVPN config locally
openvpn --config /path/to/config --verb 3
```

### Client Cannot Connect (Server mode)

**Check:**
- Server is enabled and running
- Firewall allows OpenVPN port (default 1194)
- Client configuration matches server
- Certificates are valid

### Cannot Connect to Server (Client mode)

**Check:**
- Server address is correct and reachable
- Server port is correct
- Authentication credentials are valid
- Certificates match server requirements

---

# Security Best Practices

## IPsec

1. **Use IKEv2** instead of IKEv1 when possible
2. **Use strong encryption**: AES-256 preferred
3. **Enable PFS** (Perfect Forward Secrecy)
4. **Use certificates** instead of PSK for production
5. **Disable aggressive mode** (less secure)
6. **Use strong PSK**: 20+ characters, mixed case, numbers, symbols
7. **Configure specific subnets**: Avoid `allowAllTraffic: true` when possible

## GRE

1. **Use GRE over IPsec**: GRE alone is not encrypted
2. **Restrict firewall**: Allow only necessary traffic through GRE
3. **Use static IPs**: Or reliable DDNS for remote peer
4. **Monitor tunnel**: Use ping monitoring to detect failures

## OpenVPN

1. **Use TLS authentication**: Add `tls-auth` directive
2. **Use strong ciphers**: AES-256-CBC or AES-256-GCM
3. **Use strong auth**: SHA256 or SHA512
4. **Protect private keys**: Use encrypted keys with passwords
5. **Use certificates**: Avoid static key mode for production
6. **Enable compression carefully**: Can expose to VORACLE attack
7. **Limit user permissions**: Use `user nobody` and `group nogroup`

---

# Performance Considerations

## IPsec

- **AES-128 vs AES-256**: AES-128 is faster, AES-256 is more secure
- **Hardware acceleration**: Some devices have AES hardware acceleration
- **PFS overhead**: Adds computational cost but improves security
- **Compression**: Can improve throughput but adds CPU overhead

## GRE

- **No encryption overhead**: GRE alone is fast
- **MTU considerations**: GRE adds 24 bytes of overhead
- **TTL settings**: 0 (inherit) is most flexible
- **Routing**: Efficient for dynamic routing protocols

## OpenVPN

- **UDP vs TCP**: UDP is faster, TCP is more reliable
- **Compression**: Can improve throughput on slow links
- **Cipher choice**: AES-GCM is faster than AES-CBC
- **TUN vs TAP**: TUN (layer 3) is more efficient than TAP (layer 2)

---

# Related Resources

- [Network Interfaces](NETWORK-INTERFACES-SCHEMA.md) - Network configuration
- [Firewall Configuration](NAT-FIREWALL-SCHEMAS.md) - Firewall rules for VPN
- [Routing Configuration](API-README.md#network) - Static routes
- [Main API Documentation](API-README.md)

---

**Last Updated**: December 17, 2025  
**Schema Versions**: ipsecTunnels (1), greTunnels (1), ovpnTunnels (1)
