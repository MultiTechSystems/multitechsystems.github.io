---
layout: default
title: "VPN Schemas Reference"
permalink: /v2/mpower-api-docs/VPN-SCHEMAS/
doc_version: v2
---

# VPN Schemas Reference

Complete schema documentation for VPN resources extracted from mtsDeviceAPI source code.

**Source:** `/home/jreiss/Workspace/aep/mtsDeviceAPI/json/strict/`

---

## Table of Contents

- [OpenVPN Tunnels](#openvpn-tunnels)
- [IPsec Tunnels](#ipsec-tunnels)
- [GRE Tunnels](#gre-tunnels)

---

## OpenVPN Tunnels

**Source:** `ovpnTunnels.schema.json`

**Endpoints:**
- `GET /api?fields=ovpnTunnels` - Get OpenVPN tunnel configurations
- `PUT /api/ovpnTunnels` - Update OpenVPN tunnels
- `POST /api/ovpnTunnels` - Create new tunnel
- `DELETE /api/ovpnTunnels/{name}` - Delete tunnel

### Schema

```json
{
  "enabled": true,
  "name": "tunnel-name",
  "description": "Tunnel description",
  "type": "CLIENT",
  "config": "# OpenVPN configuration..."
}
```

### Properties

| Property | Type | Required | Constraints | Description |
|----------|------|----------|-------------|-------------|
| `enabled` | boolean | Yes | - | Enable/disable tunnel |
| `name` | string | Yes | max 250, no spaces | Unique tunnel name |
| `description` | string | Yes | - | Tunnel description |
| `type` | string | Yes | enum | `SERVER`, `CLIENT`, or `CUSTOM` |
| `config` | string | Yes | max 60000 | OpenVPN configuration file content |

### Tunnel Types

| Type | Description |
|------|-------------|
| `SERVER` | OpenVPN server configuration (WebUI-managed) |
| `CLIENT` | OpenVPN client configuration (WebUI-managed) |
| `CUSTOM` | User-provided configuration file |

### Example - Create OpenVPN Client Tunnel

```bash
curl -X POST http://192.168.2.1/api/ovpnTunnels \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "name": "corporate-vpn",
    "description": "Connection to corporate network",
    "type": "CLIENT",
    "config": "client\ndev tun\nproto udp\nremote vpn.example.com 1194\nresolv-retry infinite\nnobind\npersist-key\npersist-tun\nca /etc/openvpn/ca.crt\ncert /etc/openvpn/client.crt\nkey /etc/openvpn/client.key\ncipher AES-256-CBC\nauth SHA256\ncomp-lzo\nverb 3"
  }' \
  -b cookies.txt
```

### Example - Create OpenVPN Server

```bash
curl -X POST http://192.168.2.1/api/ovpnTunnels \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "name": "vpn-server",
    "description": "VPN server for remote access",
    "type": "SERVER",
    "config": "port 1194\nproto udp\ndev tun\nca ca.crt\ncert server.crt\nkey server.key\ndh dh2048.pem\nserver 10.8.0.0 255.255.255.0\nifconfig-pool-persist ipp.txt\npush \"redirect-gateway def1 bypass-dhcp\"\npush \"dhcp-option DNS 8.8.8.8\"\nkeepalive 10 120\ncipher AES-256-CBC\nauth SHA256\ncomp-lzo\npersist-key\npersist-tun\nstatus openvpn-status.log\nverb 3"
  }' \
  -b cookies.txt
```

---

## IPsec Tunnels

**Source:** `ipsecTunnels.schema.json`

**Endpoints:**
- `GET /api?fields=ipsecTunnels` - Get IPsec tunnel configurations
- `PUT /api/ipsecTunnels` - Update IPsec tunnels
- `POST /api/ipsecTunnels` - Create new tunnel
- `DELETE /api/ipsecTunnels/{name}` - Delete tunnel

### Schema

```json
{
  "enabled": true,
  "name": "ipsec-tunnel",
  "description": "IPsec tunnel description",
  "remoteWanIp": "203.0.113.1",
  "type": "IKEv2",
  "allowAllTraffic": false,
  "localSubnets": [
    { "ip": "192.168.2.0", "mask": 24 }
  ],
  "remoteSubnets": [
    { "ip": "10.0.0.0", "mask": 8 }
  ],
  "authMethod": "secret",
  "psk": "shared-secret-key",
  "caCert": "",
  "localCert": "",
  "localPrivKey": "",
  "uid": false,
  "localId": "",
  "remoteId": "",
  "encryptMethod": "aes256",
  "phase1": "",
  "phase2": "",
  "ikelife": 24,
  "keylife": 8,
  "retries": 3,
  "checkPeriod": 5,
  "compression": false,
  "aggressiveMode": false,
  "perfectForwardSecrecy": true,
  "natTraversal": true
}
```

### Properties

| Property | Type | Required | Constraints | Description |
|----------|------|----------|-------------|-------------|
| `enabled` | boolean | Yes | - | Enable/disable tunnel |
| `name` | string | Yes | max 15 | Unique tunnel name |
| `description` | string | Yes | max 250 | Tunnel description |
| `remoteWanIp` | string | Yes | IP/hostname | Remote endpoint address |
| `type` | string | Yes | enum | `IKE` or `IKEv2` |
| `allowAllTraffic` | boolean | Yes | - | Allow all traffic through tunnel |
| `localSubnets` | array | Yes | min 1 if enabled | Local network definitions |
| `remoteSubnets` | array | Yes | min 1 if enabled | Remote network definitions |
| `authMethod` | string | Yes | enum | `secret` or `rsasig` |
| `psk` | string | Conditional | max 250, required if authMethod=secret | Pre-shared key |
| `caCert` | string | Conditional | max 3000, required if authMethod=rsasig | CA certificate (PEM) |
| `localCert` | string | Conditional | max 3000, required if authMethod=rsasig | Local certificate (PEM) |
| `localPrivKey` | string | Conditional | max 3000, required if authMethod=rsasig | Local private key (PEM) |
| `uid` | boolean | Yes | - | Use endpoint identification |
| `localId` | string | Yes | max 250, no `"` or ` : ` | Local identification |
| `remoteId` | string | Yes | max 250, no `"` or ` : ` | Remote identification |
| `encryptMethod` | string | Yes | enum | Encryption method |
| `phase1` | string | No | max 250 | Phase 1 algorithms (e.g., `aes128-sha1;modp1024`) |
| `phase2` | string | No | max 250 | Phase 2 algorithms |
| `ikelife` | integer | Yes | min 0 | IKE lifetime in hours |
| `keylife` | integer | Yes | min 0 | Key lifetime in hours |
| `retries` | integer | Yes | min 0 | Connection retry attempts |
| `compression` | boolean | Yes | - | Enable IPComp compression |
| `perfectForwardSecrecy` | boolean | Yes | - | Enable PFS |

### Encryption Methods

| Value | Description |
|-------|-------------|
| `3des` | Triple DES |
| `aes` | AES (auto) |
| `aes128` | AES 128-bit |
| `aes192` | AES 192-bit |
| `aes256` | AES 256-bit |
| `` | None specified |

### Subnet Object

```json
{
  "ip": "192.168.2.0",
  "mask": 24
}
```

| Property | Type | Constraints | Description |
|----------|------|-------------|-------------|
| `ip` | string | IPv4 format | Network address |
| `mask` | integer | 1-32 | CIDR prefix length |

### Example - Create IPsec Tunnel with PSK

```bash
curl -X POST http://192.168.2.1/api/ipsecTunnels \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "name": "site-to-site",
    "description": "Site-to-site VPN to HQ",
    "remoteWanIp": "203.0.113.10",
    "type": "IKEv2",
    "allowAllTraffic": false,
    "localSubnets": [
      { "ip": "192.168.2.0", "mask": 24 }
    ],
    "remoteSubnets": [
      { "ip": "10.0.0.0", "mask": 8 }
    ],
    "authMethod": "secret",
    "psk": "MySecurePreSharedKey123!",
    "caCert": "",
    "localCert": "",
    "localPrivKey": "",
    "uid": false,
    "localId": "",
    "remoteId": "",
    "encryptMethod": "aes256",
    "ikelife": 24,
    "keylife": 8,
    "retries": 3,
    "compression": false,
    "perfectForwardSecrecy": true
  }' \
  -b cookies.txt
```

### Example - Create IPsec Tunnel with Certificates

```bash
curl -X POST http://192.168.2.1/api/ipsecTunnels \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "name": "cert-tunnel",
    "description": "Certificate-based IPsec tunnel",
    "remoteWanIp": "vpn.example.com",
    "type": "IKEv2",
    "allowAllTraffic": false,
    "localSubnets": [
      { "ip": "192.168.2.0", "mask": 24 }
    ],
    "remoteSubnets": [
      { "ip": "10.0.0.0", "mask": 24 }
    ],
    "authMethod": "rsasig",
    "psk": "",
    "caCert": "-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----",
    "localCert": "-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----",
    "localPrivKey": "-----BEGIN RSA PRIVATE KEY-----\n...\n-----END RSA PRIVATE KEY-----",
    "uid": true,
    "localId": "@gateway.example.com",
    "remoteId": "@vpn.example.com",
    "encryptMethod": "aes256",
    "phase1": "aes256-sha256;modp2048",
    "phase2": "aes256-sha256;modp2048",
    "ikelife": 24,
    "keylife": 8,
    "retries": 3,
    "compression": false,
    "perfectForwardSecrecy": true,
    "natTraversal": true
  }' \
  -b cookies.txt
```

---

## GRE Tunnels

**Source:** `greTunnels.schema.json`

**Endpoints:**
- `GET /api?fields=greTunnels` - Get GRE tunnel configurations
- `PUT /api/greTunnels` - Update GRE tunnels
- `POST /api/greTunnels` - Create new tunnel
- `DELETE /api/greTunnels/{name}` - Delete tunnel

### Schema

```json
{
  "enabled": true,
  "name": "gre-tunnel",
  "description": "GRE tunnel description",
  "localIp": "192.168.2.1",
  "remoteIp": "203.0.113.1",
  "tunnelIp": "10.255.255.1",
  "tunnelMask": 30,
  "key": 0,
  "mtu": 1476
}
```

### Properties

| Property | Type | Required | Constraints | Description |
|----------|------|----------|-------------|-------------|
| `enabled` | boolean | Yes | - | Enable/disable tunnel |
| `name` | string | Yes | max 15 | Unique tunnel name |
| `description` | string | No | max 250 | Tunnel description |
| `localIp` | string | Yes | IPv4 | Local endpoint IP |
| `remoteIp` | string | Yes | IPv4 | Remote endpoint IP |
| `tunnelIp` | string | Yes | IPv4 | Tunnel interface IP |
| `tunnelMask` | integer | Yes | 1-32 | Tunnel interface netmask |
| `key` | integer | No | min 0 | GRE key (0 = disabled) |
| `mtu` | integer | No | - | Tunnel MTU |

### Example - Create GRE Tunnel

```bash
curl -X POST http://192.168.2.1/api/greTunnels \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "name": "gre0",
    "description": "GRE tunnel to remote site",
    "localIp": "192.168.2.1",
    "remoteIp": "203.0.113.50",
    "tunnelIp": "10.255.255.1",
    "tunnelMask": 30,
    "key": 12345,
    "mtu": 1476
  }' \
  -b cookies.txt
```

---

## VPN Troubleshooting

### Common Issues

**IPsec Tunnel Not Establishing:**
1. Verify remote endpoint is reachable
2. Check PSK/certificates match on both ends
3. Ensure IKE version matches
4. Verify subnet configurations

**OpenVPN Connection Failed:**
1. Check certificate validity
2. Verify port is open (default: 1194)
3. Check configuration syntax
4. Review server/client logs

### Checking Tunnel Status

```bash
# Get IPsec tunnel status
curl -X GET "http://192.168.2.1/api?fields=ipsecTunnels" -b cookies.txt

# Get OpenVPN tunnel status
curl -X GET "http://192.168.2.1/api?fields=ovpnTunnels" -b cookies.txt

# Get interface statistics for VPN interfaces
curl -X GET "http://192.168.2.1/api?fields=stats/iface" -b cookies.txt
```

---

## Related Documentation

- [API Schemas Reference](API-SCHEMAS-REFERENCE.md) - Complete schema reference
- [Available Resources](AVAILABLE-RESOURCES.md) - Resource endpoints
- [Firewall & NAT](NAT-FIREWALL-SCHEMAS.md) - Firewall rules for VPN traffic

---

**Document Version:** 1.0.0  
**Source Version:** mtsDeviceAPI (analyzed February 2026)  
**Last Updated:** February 10, 2026
