---
layout: default
title: "Connectivity Schemas Reference"
permalink: /v2/mpower-api-docs/CONNECTIVITY-SCHEMAS/
doc_version: v2
---

# Connectivity Schemas Reference

Complete schema documentation for connectivity and networking resources extracted from mtsDeviceAPI source code.

**Source:** mtsDeviceAPI `json/strict/*.schema.json`

---

## Table of Contents

- [Bluetooth](#bluetooth)
- [Bluetooth Low Energy](#bluetooth-low-energy)
- [IP Passthrough](#ip-passthrough)
- [IP Pipes](#ip-pipes)
- [DDNS](#ddns)
- [WAN Manager](#wan-manager)
- [WAN Info](#wan-info)
- [Backoff Timers](#backoff-timers)
- [Cell Time Sync](#cell-time-sync)

---

## Bluetooth

Classic Bluetooth configuration.

**Source:** `bluetooth.schema.json`

**Endpoints:**
- `GET /api?fields=bluetooth` - Get Bluetooth configuration
- `PUT /api/bluetooth` - Update Bluetooth configuration

### Schema

```json
{
  "enabled": false,
  "device": {
    "name": "",
    "address": ""
  },
  "devices": [],
  "savedDevices": []
}
```

### Key Properties

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable Bluetooth |
| `device.name` | string | Connected device name (max 250) |
| `device.address` | string | Connected device MAC address |
| `devices` | array | Devices found during last scan |
| `savedDevices` | array | Saved/paired devices |

### Device Object

```json
{
  "name": "Device Name",
  "address": "AA:BB:CC:DD:EE:FF",
  "signal": "-45 dBm"
}
```

### Saved Device Object

```json
{
  "name": "Device Name",
  "address": "AA:BB:CC:DD:EE:FF",
  "friendlyName": "My Device"
}
```

### MAC Address Pattern
```
^([0-9A-Fa-f]{2}:){5}([0-9A-Fa-f]{2})$
```

---

## Bluetooth Low Energy

BLE configuration and power settings.

**Source:** `bluetoothLowEnergy.schema.json`

**Endpoints:**
- `GET /api?fields=bluetoothLowEnergy` - Get BLE configuration
- `PUT /api/bluetoothLowEnergy` - Update BLE configuration

### Schema

```json
{
  "enabled": false,
  "powerRfTotal": "Medium",
  "powerRfRx": "Medium",
  "powerRfTx": "Medium",
  "device": {
    "name": "",
    "interface": "",
    "address": ""
  },
  "devices": [],
  "savedDevices": []
}
```

### Power Settings

| Property | Type | Values |
|----------|------|--------|
| `powerRfTotal` | string | `Custom`, `Low`, `Medium`, `High` |
| `powerRfRx` | string | `Low`, `Medium`, `High` |
| `powerRfTx` | string | `Low`, `Medium`, `High` |

---

## IP Passthrough

LAN interface settings for PPP IP Passthrough mode.

**Source:** `ipPassthrough.schema.json`

**Endpoints:**
- `GET /api?fields=ipPassthrough` - Get IP passthrough configuration
- `PUT /api/ipPassthrough` - Update IP passthrough configuration

### Schema

```json
{
  "ipMode": "IPv4",
  "ip": "192.168.2.1",
  "mask": "255.255.255.0",
  "gateway": "192.168.2.1",
  "publicIpMask": "255.255.255.255",
  "ipv6": {
    "ip": "",
    "prefixlen": ""
  }
}
```

### Key Properties

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `ipMode` | string | IP protocol mode | `IPv4` |
| `ip` | string | IP address | 192.168.2.1 |
| `mask` | string | Subnet mask | 255.255.255.0 |
| `gateway` | string | Default gateway | 192.168.2.1 |
| `publicIpMask` | string | Public IP mask | 255.255.255.255 |

### IP Mode Values
- `IPv4`
- `IPv6`
- `IPv4 and IPv6`

---

## IP Pipes

Serial-to-IP pipe configuration.

**Source:** `ipPipes.schema.json`

**Endpoints:**
- `GET /api?fields=ipPipes` - Get IP pipes configuration
- `PUT /api/ipPipes` - Update IP pipes
- `POST /api/ipPipes` - Create IP pipe
- `DELETE /api/ipPipes/{id}` - Delete IP pipe

### Schema

```json
{
  "enabled": true,
  "id": "pipe-1",
  "mode": "CLIENT",
  "ip": "192.168.1.100",
  "port": 5000,
  "secondaryIp": "",
  "secondaryPort": 0,
  "activation": "ALWAYS-ON",
  "termination": "ALWAYS-ON",
  "timeoutSeconds": 60,
  "sequence": "",
  "protocol": "TCP",
  "bufferTimeoutMs": 100,
  "serialReadBufSize": 1024
}
```

### Key Properties

| Property | Type | Description | Constraints |
|----------|------|-------------|-------------|
| `mode` | string | Client or server mode | `CLIENT`, `SERVER` |
| `protocol` | string | Network protocol | `UDP`, `TCP`, `TLS` |
| `port` | integer | Port number | 1-65535 |
| `activation` | string | Connection start trigger | See below |
| `termination` | string | Connection end mode | See below |
| `bufferTimeoutMs` | integer | Buffer flush timeout | 0-1000 ms |
| `serialReadBufSize` | integer | Serial buffer size | 1-65536 bytes |

### Activation Modes (CLIENT mode)

| Value | Description |
|-------|-------------|
| `ALWAYS-ON` | Always connected |
| `ON-DEMAND` | Connect on demand |
| `DTR-ASSERT` | Connect when DTR asserted |
| `CR` | Connect on carriage return |

### Termination Modes

| Value | Description |
|-------|-------------|
| `ALWAYS-ON` | Stay connected |
| `DTR-TOGGLE` | Disconnect on DTR toggle |
| `TIMEOUT` | Disconnect after inactivity |
| `SEQUENCE` | Disconnect on escape sequence |

---

## DDNS

Dynamic DNS configuration.

**Source:** `ddns.schema.json`

**Endpoints:**
- `GET /api?fields=ddns` - Get DDNS configuration
- `PUT /api/ddns` - Update DDNS configuration
- `POST /api/command/ddns_update` - Force DDNS update

### Schema

```json
{
  "enabled": false,
  "domain": "mydevice.example.com",
  "mode": "PROVIDER",
  "server": "dyn.com",
  "custom": {
    "server": "",
    "path": "",
    "port": 80,
    "ssl": false
  },
  "checkIp": {
    "enabled": true,
    "mode": "DEFAULT",
    "custom": {
      "server": "",
      "path": "/",
      "port": 80,
      "ssl": false
    }
  },
  "authentication": {
    "username": "",
    "password": ""
  },
  "updateInterval": 5,
  "checkIpPeriod": 15
}
```

### Key Properties

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `enabled` | boolean | Enable DDNS | false |
| `domain` | string | Domain name to update | - |
| `mode` | string | `PROVIDER` or `CUSTOM` | PROVIDER |
| `server` | string | DDNS provider | - |
| `updateInterval` | integer | Update interval (days) | 5 (1-30) |
| `checkIpPeriod` | integer | IP check period (minutes) | 15 (1-14400) |

### Custom Server Path Variables

For custom DDNS servers, the path supports format specifiers:
- `%u` - Username
- `%p` - Password
- `%h` - Hostname
- `%i` - IP address

Example: `/nic/update?user=%u&password=%p&domain=%h&myip=%i`

---

## WAN Manager

WAN failover and management configuration.

**Source:** `wanmngr.schema.json`

**Endpoints:**
- `GET /api?fields=wanmngr` - Get WAN manager configuration
- `PUT /api/wanmngr` - Update WAN manager configuration

### Schema

```json
{
  "mode": "FAILOVER",
  "wans": [
    {
      "interface": "eth0",
      "virtualIface": false,
      "weight": 1,
      "priority": 1,
      "monitor": {
        "mode": "ACTIVE",
        "checkInterval": 60,
        "active": {
          "hostname": "www.google.com",
          "type": "ICMP",
          "tcpPort": 80,
          "icmpCount": 10
        }
      }
    }
  ]
}
```

### WAN Manager Modes

| Mode | Description |
|------|-------------|
| `CONSTANT` | Use single WAN interface |
| `FAILOVER` | Automatic failover between WANs |

### Monitor Configuration

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `mode` | string | `ACTIVE` or `PASSIVE` | ACTIVE |
| `checkInterval` | integer | Check interval (seconds) | 60 (5-9999) |

### Active Monitor Settings

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `hostname` | string | Host to check | www.google.com |
| `type` | string | `ICMP` or `TCP` | ICMP |
| `tcpPort` | integer | TCP port (if TCP type) | 80 |
| `icmpCount` | integer | Ping count | 10 (1-200) |

---

## WAN Info

WAN interface status information (read-only).

**Source:** `waninfo.schema.json`

**Endpoints:**
- `GET /api?fields=waninfo` - Get WAN information
- `GET /api?fields=waninfo/currentWan` - Get current WAN
- `GET /api?fields=waninfo/currentWan6` - Get current IPv6 WAN

### Schema

```json
{
  "wans": [
    {
      "interface": "eth0",
      "name": "Ethernet",
      "type": "ethernet",
      "current": true,
      "available": true,
      "enabled": true,
      "ip": "192.168.1.100",
      "subnet": "255.255.255.0",
      "gateway": "192.168.1.1",
      "virtualIface": false
    }
  ]
}
```

---

## Backoff Timers

Carrier-defined backoff timers for connections.

**Source:** `backOffTimers.schema.json`

**Endpoints:**
- `GET /api?fields=backOffTimers` - Get backoff timers
- `PUT /api/backOffTimers` - Update backoff timers

### Schema

```json
{
  "enabled": true,
  "timers": [
    {
      "carrierName": "Default",
      "readOnly": false,
      "data": [30, 60, 120, 300, 600],
      "dataReset": [60, 300, 600, 1800],
      "sms": [30, 60, 120],
      "modemReset": [300, 600, 1800, 3600]
    }
  ]
}
```

### Timer Arrays

| Array | Description |
|-------|-------------|
| `data` | Seconds between cellular connection attempts |
| `dataReset` | Seconds between connection reset attempts |
| `sms` | Seconds between SMS send attempts |
| `modemReset` | Seconds between modem reset attempts |

---

## Cell Time Sync

Cellular network time synchronization.

**Source:** `cellTimeSync.schema.json`

**Endpoints:**
- `GET /api?fields=cellTimeSync` - Get cell time sync configuration
- `PUT /api/cellTimeSync` - Update cell time sync configuration

### Schema

```json
{
  "enabled": false,
  "pollingTime": 120
}
```

### Key Properties

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `enabled` | boolean | Enable cell time sync | false |
| `pollingTime` | integer | Poll interval (minutes) | 120 (5-1440) |

---

**Last Updated:** February 10, 2026
