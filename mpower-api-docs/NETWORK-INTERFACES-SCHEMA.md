# Network Interfaces (ni) Configuration Schema

Complete schema documentation for the `ni` (Network Interfaces) resource in the mPower Device API.

## Overview

The `ni` resource manages all network interfaces on the device, including Ethernet ports, PPP connections, WiFi interfaces, and bridge configurations. It supports IPv4, IPv6, VLANs, 802.1X authentication, and advanced networking features.

## Top-Level Structure

```json
{
  "__v": 3,
  "nis": [
    {
      // Network interface configuration
    }
  ]
}
```

| Property | Type | Description |
|----------|------|-------------|
| `__v` | integer | Schema version (currently 3) |
| `nis` | array | Array of network interface configurations |

## Network Interface Object

Each interface in the `nis` array contains the following properties:

### Basic Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Interface name (e.g., "eth0", "ppp0", "wlan0") |
| `type` | string | Yes | Interface role: "LAN" or "WAN" |
| `nitype` | string | Yes | Interface type: "ETHER", "PPP", "WIFI_AS_WAN", "WIFI_AP", "BRIDGE" |
| `available` | boolean | Yes | Whether the interface is available/present |
| `bridge` | string | Yes | Bridge name if connected to bridge, "--" if not bridged |
| `MAC` | string | No | MAC address of the interface |

### VLAN Configuration

| Property | Type | Description |
|----------|------|-------------|
| `vlanId` | integer | VLAN ID if this is a VLAN interface (-1 = no VLAN, 1-4094 = VLAN ID) |
| `vlans` | array | VLAN port configuration (for trunk ports) |

#### VLAN Port Object

When an interface acts as a VLAN trunk port:

```json
{
  "vlans": [
    {
      "vlanId": 100,
      "tagged": true
    },
    {
      "vlanId": 200,
      "tagged": false
    }
  ]
}
```

| Property | Type | Description |
|----------|------|-------------|
| `vlanId` | integer | VLAN ID (1-4094) |
| `tagged` | boolean | true = tagged VLAN, false = untagged (maximum 1 untagged per interface) |

**Note**: An interface can either be a VLAN interface (`vlanId` != -1) OR a VLAN trunk port (`vlans` array), but not both.

## IPv4 Configuration

The `ipv4` object configures IPv4 networking for the interface:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `mode` | string | Yes | Configuration mode (see modes below) |
| `ip` | string | Yes | IPv4 address |
| `mask` | string | Yes | Subnet mask |
| `gateway` | string | Yes | Default gateway |
| `dns1` | string | Yes | Primary DNS server |
| `dns2` | string | Yes | Secondary DNS server |
| `wanMasquerade` | boolean | Yes | Enable NAT/masquerading for WAN interfaces (default: true) |

### IPv4 Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| `STATIC` | Static IP configuration | Fixed IP address assignment |
| `DHCP` | DHCP client | Obtain IP, gateway, DNS from DHCP server |
| `DHCP-ADDRESSES-ONLY` | DHCP for addresses only | Get IP from DHCP but use static gateway/DNS |
| `PPP` | PPP connection | Cellular or dial-up connections |
| `PPP-ADDRESSES-ONLY` | PPP addresses only | PPP with static routing |

### IPv4 Configuration Examples

#### Static Configuration
```json
{
  "ipv4": {
    "mode": "STATIC",
    "ip": "192.168.1.1",
    "mask": "255.255.255.0",
    "gateway": "0.0.0.0",
    "dns1": "8.8.8.8",
    "dns2": "8.8.4.4",
    "wanMasquerade": false
  }
}
```

#### DHCP Client
```json
{
  "ipv4": {
    "mode": "DHCP",
    "ip": "0.0.0.0",
    "mask": "0.0.0.0",
    "gateway": "0.0.0.0",
    "dns1": "0.0.0.0",
    "dns2": "0.0.0.0",
    "wanMasquerade": true
  }
}
```

#### PPP Connection
```json
{
  "ipv4": {
    "mode": "PPP",
    "ip": "0.0.0.0",
    "mask": "255.255.255.255",
    "gateway": "0.0.0.0",
    "dns1": "0.0.0.0",
    "dns2": "0.0.0.0",
    "wanMasquerade": true
  }
}
```

## IPv6 Configuration

The `ipv6` object configures IPv6 networking for the interface:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `enabled` | boolean | Yes | Enable/disable IPv6 on this interface |
| `mode` | string | Yes | Configuration mode (depends on interface type) |
| `ip` | array | Yes | Current IPv6 addresses (read-only) |
| `fixedIp` | array | Yes | Static IPv6 addresses |
| `linkLocalIp` | array | Yes | Link-local IPv6 addresses (read-only) |
| `prefixDelegationEnabled` | boolean | Yes | Enable DHCPv6 Prefix Delegation (WAN only) |
| `delegatedPrefixLength` | integer | Yes | Prefix length for delegation (0-128, default: 64) |
| `gateway` | string | Yes | IPv6 default gateway |
| `dns1` | string | Yes | Primary IPv6 DNS server |
| `dns2` | string | Yes | Secondary IPv6 DNS server |

### IPv6 Modes

#### For LAN Interfaces

| Mode | Description |
|------|-------------|
| `STATIC` | Static IPv6 configuration with fixed addresses |
| `DELEGATED` | Use delegated prefix from WAN interface (DHCPv6-PD) |

#### For WAN Interfaces

| Mode | Description |
|------|-------------|
| `AUTO` | SLAAC (Stateless Address Autoconfiguration) |
| `AUTO-ADDRESSES-ONLY` | SLAAC for addresses, static routing |

### IPv6 Configuration Examples

#### Static IPv6 (LAN)
```json
{
  "ipv6": {
    "enabled": true,
    "mode": "STATIC",
    "ip": [],
    "fixedIp": ["2001:db8::1/64"],
    "linkLocalIp": [],
    "prefixDelegationEnabled": false,
    "delegatedPrefixLength": 64,
    "gateway": "",
    "dns1": "2001:4860:4860::8888",
    "dns2": "2001:4860:4860::8844"
  }
}
```

#### DHCPv6 Prefix Delegation (WAN)
```json
{
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
```

#### Delegated Prefix (LAN)
```json
{
  "ipv6": {
    "enabled": true,
    "mode": "DELEGATED",
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
```

## 802.1X Authentication

The `dot1x` object configures IEEE 802.1X port-based network access control:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `enabled` | boolean | Yes | Enable/disable 802.1X authentication |
| `authType` | string | Yes | Authentication method (see types below) |
| `status` | string | Yes | Current status: "authorized" or "unauthorized" (read-only) |
| `params` | object | Yes | Authentication parameters |

### 802.1X Authentication Types

| Type | Description | Use Case |
|------|-------------|----------|
| `EAP-PWD` | EAP-Password | Simple password-based authentication |
| `EAP-TLS` | EAP-Transport Layer Security | Certificate-based authentication |
| `EAP-TTLS` | EAP-Tunneled TLS | Tunneled authentication with inner method |
| `EAP-PEAP` | Protected EAP | Microsoft-compatible tunneled authentication |

### 802.1X Parameters

| Property | Type | Description |
|----------|------|-------------|
| `identity` | string | Username for authentication |
| `password` | string | Password (required for EAP-PWD, EAP-TTLS, EAP-PEAP) |
| `anonymousIdentity` | string | Anonymous identity for outer authentication |
| `caCertificate` | string | CA certificate (PEM format, max 20KB) |
| `clientCertificate` | string | Client certificate (PEM format, max 20KB) |
| `privateKey` | string | Private key (PEM format, max 20KB) |
| `privateKeyPassword` | string | Password to decrypt private key |
| `domainMatch` | string | Domain substring for server cert validation |
| `subjectMatch` | string | Subject substring for server cert validation |
| `innerAuthType` | string | Inner authentication: "EAP-MSCHAPV2" or "MSCHAPV2" |
| `peapVersion` | integer | PEAP version: -1 (auto), 0, or 1 |

### 802.1X Configuration Examples

#### EAP-PWD (Password)
```json
{
  "dot1x": {
    "enabled": true,
    "authType": "EAP-PWD",
    "status": "unauthorized",
    "params": {
      "identity": "user@example.com",
      "password": "secretpassword",
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
```

#### EAP-TLS (Certificate)
```json
{
  "dot1x": {
    "enabled": true,
    "authType": "EAP-TLS",
    "status": "unauthorized",
    "params": {
      "identity": "device123",
      "password": "",
      "anonymousIdentity": "",
      "caCertificate": "-----BEGIN CERTIFICATE-----\n...",
      "clientCertificate": "-----BEGIN CERTIFICATE-----\n...",
      "privateKey": "-----BEGIN PRIVATE KEY-----\n...",
      "privateKeyPassword": "keypassword",
      "domainMatch": "example.com",
      "subjectMatch": "",
      "innerAuthType": "",
      "peapVersion": -1
    }
  }
}
```

#### EAP-PEAP (Microsoft)
```json
{
  "dot1x": {
    "enabled": true,
    "authType": "EAP-PEAP",
    "status": "unauthorized",
    "params": {
      "identity": "DOMAIN\\username",
      "password": "password",
      "anonymousIdentity": "anonymous",
      "caCertificate": "-----BEGIN CERTIFICATE-----\n...",
      "clientCertificate": "",
      "privateKey": "",
      "privateKeyPassword": "",
      "domainMatch": "radius.example.com",
      "subjectMatch": "",
      "innerAuthType": "MSCHAPV2",
      "peapVersion": 0
    }
  }
}
```

**Note**: 802.1X can only be enabled on Ethernet interfaces (`nitype: "ETHER"`) that are not part of a regular bridge (must be `bridge: "--"` or `bridge: "br-vlan"`).

## Complete Interface Examples

### Example 1: LAN Ethernet Interface (Static IPv4)

```json
{
  "name": "eth0",
  "type": "LAN",
  "nitype": "ETHER",
  "available": true,
  "bridge": "--",
  "vlanId": -1,
  "ipv4": {
    "mode": "STATIC",
    "ip": "192.168.1.1",
    "mask": "255.255.255.0",
    "gateway": "0.0.0.0",
    "dns1": "8.8.8.8",
    "dns2": "8.8.4.4",
    "wanMasquerade": false
  },
  "ipv6": {
    "enabled": true,
    "mode": "STATIC",
    "ip": [],
    "fixedIp": ["fd00::1/64"],
    "linkLocalIp": [],
    "prefixDelegationEnabled": false,
    "delegatedPrefixLength": 64,
    "gateway": "",
    "dns1": "",
    "dns2": ""
  },
  "MAC": "00:11:22:33:44:55"
}
```

### Example 2: WAN Ethernet Interface (DHCP)

```json
{
  "name": "eth1",
  "type": "WAN",
  "nitype": "ETHER",
  "available": true,
  "bridge": "--",
  "vlanId": -1,
  "ipv4": {
    "mode": "DHCP",
    "ip": "0.0.0.0",
    "mask": "0.0.0.0",
    "gateway": "0.0.0.0",
    "dns1": "0.0.0.0",
    "dns2": "0.0.0.0",
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
  },
  "MAC": "00:11:22:33:44:66"
}
```

### Example 3: VLAN Interface

```json
{
  "name": "eth0.100",
  "type": "LAN",
  "nitype": "ETHER",
  "available": true,
  "bridge": "--",
  "vlanId": 100,
  "ipv4": {
    "mode": "STATIC",
    "ip": "192.168.100.1",
    "mask": "255.255.255.0",
    "gateway": "0.0.0.0",
    "dns1": "0.0.0.0",
    "dns2": "0.0.0.0",
    "wanMasquerade": false
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
```

### Example 4: VLAN Trunk Port

```json
{
  "name": "eth2",
  "type": "LAN",
  "nitype": "ETHER",
  "available": true,
  "bridge": "--",
  "vlanId": -1,
  "vlans": [
    {"vlanId": 10, "tagged": true},
    {"vlanId": 20, "tagged": true},
    {"vlanId": 30, "tagged": false}
  ],
  "ipv4": {
    "mode": "STATIC",
    "ip": "0.0.0.0",
    "mask": "0.0.0.0",
    "gateway": "0.0.0.0",
    "dns1": "0.0.0.0",
    "dns2": "0.0.0.0",
    "wanMasquerade": false
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
```

### Example 5: PPP Interface (Cellular)

```json
{
  "name": "ppp0",
  "type": "WAN",
  "nitype": "PPP",
  "available": true,
  "bridge": "--",
  "vlanId": -1,
  "ipv4": {
    "mode": "PPP",
    "ip": "0.0.0.0",
    "mask": "255.255.255.255",
    "gateway": "0.0.0.0",
    "dns1": "0.0.0.0",
    "dns2": "0.0.0.0",
    "wanMasquerade": true
  },
  "ipv6": {
    "enabled": false,
    "mode": "AUTO",
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
```

### Example 6: WiFi Access Point

```json
{
  "name": "wlan0",
  "type": "LAN",
  "nitype": "WIFI_AP",
  "available": true,
  "bridge": "br-lan",
  "vlanId": -1,
  "ipv4": {
    "mode": "STATIC",
    "ip": "0.0.0.0",
    "mask": "0.0.0.0",
    "gateway": "0.0.0.0",
    "dns1": "0.0.0.0",
    "dns2": "0.0.0.0",
    "wanMasquerade": false
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
  "MAC": "00:11:22:33:44:77"
}
```

### Example 7: Bridge Interface

```json
{
  "name": "br-lan",
  "type": "LAN",
  "nitype": "BRIDGE",
  "available": true,
  "bridge": "--",
  "vlanId": -1,
  "ipv4": {
    "mode": "STATIC",
    "ip": "192.168.2.1",
    "mask": "255.255.255.0",
    "gateway": "0.0.0.0",
    "dns1": "0.0.0.0",
    "dns2": "0.0.0.0",
    "wanMasquerade": false
  },
  "ipv6": {
    "enabled": true,
    "mode": "DELEGATED",
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
```

## API Operations

### Get All Network Interfaces

```bash
GET /api?fields=ni
```

Response:
```json
{
  "success": true,
  "result": {
    "__v": 3,
    "nis": [
      // Array of interface configurations
    ]
  }
}
```

### Get Single Network Interface by Index

```bash
GET /api/ni/nis/{index}
```

Example - Get br0 interface at index 6:
```bash
GET /api/ni/nis/6
```

Response:
```json
{
  "code": 200,
  "result": {
    "MAC": "",
    "available": true,
    "bridge": "br0",
    "ipv4": {
      "ip": "192.168.2.1",
      "mask": "255.255.255.0",
      "mode": "STATIC",
      ...
    },
    "name": "br0",
    "nitype": "BRIDGE",
    "type": "LAN"
  },
  "status": "success"
}
```

### Update Single Network Interface by Index

The recommended way to update a network interface is by index:

```bash
PUT /api/ni/nis/{index}
Content-Type: application/json
```

Example - Update br0 interface at index 6:
```bash
PUT /api/ni/nis/6
Content-Type: application/json

{
  "MAC": "",
  "available": true,
  "bridge": "br0",
  "ipv4": {
    "dns1": "8.8.8.8",
    "dns2": "",
    "gateway": "",
    "ip": "192.168.100.1",
    "mask": "255.255.255.0",
    "mode": "STATIC",
    "wanMasquerade": true
  },
  "ipv6": {
    "delegatedPrefixLength": 64,
    "dns1": "",
    "dns2": "",
    "enabled": false,
    "fixedIp": [],
    "gateway": "",
    "ip": [],
    "linkLocalIp": [],
    "mode": "DELEGATED",
    "prefixDelegationEnabled": false
  },
  "name": "br0",
  "nitype": "BRIDGE",
  "type": "LAN",
  "vlanId": -1
}
```

**Tip:** Use your browser's developer tools (F12 → Network tab) while using the mPower web UI to discover the correct index for each interface on your device.

### Update Entire Network Interfaces Object (Alternative)

You can also update the entire `ni` object, but this requires including all interfaces:

```bash
PUT /api/ni
Content-Type: application/json

{
  "__v": 3,
  "nis": [
    {
      "name": "eth0",
      "type": "LAN",
      "nitype": "ETHER",
      "available": true,
      "bridge": "--",
      "vlanId": -1,
      "ipv4": {
        "mode": "STATIC",
        "ip": "192.168.10.1",
        "mask": "255.255.255.0",
        "gateway": "0.0.0.0",
        "dns1": "8.8.8.8",
        "dns2": "8.8.4.4",
        "wanMasquerade": false
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
    // Include all other interfaces unchanged
  ]
}
```

### Save Configuration

After updating network interfaces, save the configuration:

```bash
POST /api/command/save
```

### Apply Network Changes

Network changes typically require a restart or service reload:

```bash
POST /api/command/restart
```

## Important Notes

### Interface Naming

- **Ethernet**: `eth0`, `eth1`, `eth2`, etc.
- **PPP**: `ppp0`, `ppp1`, etc.
- **WiFi**: `wlan0`, `wlan1`, etc.
- **Bridge**: `br-lan`, `br-wan`, `br-vlan`, etc.
- **VLAN**: `eth0.100`, `eth1.200`, etc. (parent.vlanid)

### Interface Types

- **LAN**: Local Area Network interfaces (typically for local devices)
- **WAN**: Wide Area Network interfaces (typically for internet connection)

### Interface NI Types

- **ETHER**: Standard Ethernet interface
- **PPP**: Point-to-Point Protocol (cellular, dial-up)
- **WIFI_AS_WAN**: WiFi client mode (connecting to external AP)
- **WIFI_AP**: WiFi Access Point mode
- **BRIDGE**: Bridge interface (combines multiple interfaces)

### VLAN Constraints

- VLAN IDs must be between 1 and 4094
- An interface can be either:
  - A VLAN interface (has `vlanId` != -1)
  - A VLAN trunk port (has `vlans` array)
  - But NOT both simultaneously
- Maximum 1 untagged VLAN per trunk port

### IPv6 Mode Constraints

- **LAN interfaces** can only use: `STATIC` or `DELEGATED`
- **WAN interfaces** can only use: `AUTO` or `AUTO-ADDRESSES-ONLY`

### 802.1X Constraints

- Only available on Ethernet interfaces (`nitype: "ETHER"`)
- Interface must not be part of a regular bridge
- Allowed bridge values: `"--"` or `"br-vlan"`

### Bridging

- Interfaces can be added to a bridge by setting the `bridge` property
- Bridge interfaces themselves have `bridge: "--"`
- Common bridges:
  - `br-lan`: LAN bridge
  - `br-wan`: WAN bridge (less common)
  - `br-vlan`: VLAN-aware bridge

### IP Address Formats

- **IPv4**: Standard dotted decimal (e.g., "192.168.1.1")
- **IPv4 Mask**: Standard dotted decimal (e.g., "255.255.255.0")
- **IPv6**: Standard colon-hexadecimal (e.g., "2001:db8::1")
- **IPv6 with Prefix**: Address/length format (e.g., "2001:db8::1/64")
- **Empty**: Use `""` or `"0.0.0.0"` for unset/dynamic values

### MAC Address Format

- Standard colon-separated format: `"00:11:22:33:44:55"`
- Special values: `""` (empty) or `"ANY"`

## Common Use Cases

### 1. Configure Static LAN Interface

```bash
PUT /api/ni
{
  "nis": [{
    "name": "eth0",
    "type": "LAN",
    "nitype": "ETHER",
    "available": true,
    "bridge": "--",
    "vlanId": -1,
    "ipv4": {
      "mode": "STATIC",
      "ip": "192.168.1.1",
      "mask": "255.255.255.0",
      "gateway": "0.0.0.0",
      "dns1": "0.0.0.0",
      "dns2": "0.0.0.0",
      "wanMasquerade": false
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
  }]
}
```

### 2. Configure DHCP WAN Interface

```bash
PUT /api/ni
{
  "nis": [{
    "name": "eth1",
    "type": "WAN",
    "nitype": "ETHER",
    "available": true,
    "bridge": "--",
    "vlanId": -1,
    "ipv4": {
      "mode": "DHCP",
      "ip": "0.0.0.0",
      "mask": "0.0.0.0",
      "gateway": "0.0.0.0",
      "dns1": "0.0.0.0",
      "dns2": "0.0.0.0",
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
  }]
}
```

### 3. Create VLAN Interface

```bash
PUT /api/ni
{
  "nis": [{
    "name": "eth0.100",
    "type": "LAN",
    "nitype": "ETHER",
    "available": true,
    "bridge": "--",
    "vlanId": 100,
    "ipv4": {
      "mode": "STATIC",
      "ip": "192.168.100.1",
      "mask": "255.255.255.0",
      "gateway": "0.0.0.0",
      "dns1": "0.0.0.0",
      "dns2": "0.0.0.0",
      "wanMasquerade": false
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
  }]
}
```

### 4. Enable 802.1X Authentication

```bash
PUT /api/ni
{
  "nis": [{
    "name": "eth0",
    "type": "WAN",
    "nitype": "ETHER",
    "available": true,
    "bridge": "--",
    "vlanId": -1,
    "dot1x": {
      "enabled": true,
      "authType": "EAP-PEAP",
      "status": "unauthorized",
      "params": {
        "identity": "user@company.com",
        "password": "password",
        "anonymousIdentity": "anonymous",
        "caCertificate": "-----BEGIN CERTIFICATE-----\n...",
        "clientCertificate": "",
        "privateKey": "",
        "privateKeyPassword": "",
        "domainMatch": "radius.company.com",
        "subjectMatch": "",
        "innerAuthType": "MSCHAPV2",
        "peapVersion": 0
      }
    },
    "ipv4": {
      "mode": "DHCP",
      "ip": "0.0.0.0",
      "mask": "0.0.0.0",
      "gateway": "0.0.0.0",
      "dns1": "0.0.0.0",
      "dns2": "0.0.0.0",
      "wanMasquerade": true
    },
    "ipv6": {
      "enabled": false,
      "mode": "AUTO",
      "ip": [],
      "fixedIp": [],
      "linkLocalIp": [],
      "prefixDelegationEnabled": false,
      "delegatedPrefixLength": 64,
      "gateway": "",
      "dns1": "",
      "dns2": ""
    }
  }]
}
```

## Troubleshooting

### Interface Not Available

If `available: false`, the physical interface may not be present or the driver may not be loaded.

### 802.1X Authentication Fails

Check:
- `status` field shows current state
- Verify certificates are valid and in PEM format
- Check `domainMatch` and `subjectMatch` against server certificate
- Ensure `innerAuthType` is correct for your RADIUS server

### VLAN Configuration Issues

- Ensure parent interface exists
- Check VLAN ID is in valid range (1-4094)
- Verify trunk port has correct tagged/untagged configuration
- Only one untagged VLAN allowed per trunk port

### IPv6 Not Working

- Verify `enabled: true`
- Check mode is appropriate for interface type (LAN vs WAN)
- For delegated prefixes, ensure WAN has `prefixDelegationEnabled: true`
- Verify ISP supports IPv6

### Bridge Not Working

- Ensure member interfaces have `bridge` set to bridge name
- Bridge interface itself should have `bridge: "--"`
- Member interfaces typically have IP `"0.0.0.0"`

## Related Resources

- [DNS Configuration](API-README.md#dns-endpoints)
- [DHCP Configuration](API-README.md#network)
- [Routing Configuration](API-README.md#network)
- [WiFi Configuration](API-README.md#wifi-endpoints)

---

**Last Updated**: December 17, 2025  
**Schema Version**: 3
