---
layout: default
title: "API Schemas Reference"
permalink: /v2/mpower-api-docs/API-SCHEMAS-REFERENCE/
doc_version: v2
---

# API Schemas Reference

Complete schema documentation extracted from mtsDeviceAPI source code (`json/strict/*.schema.json`).

**Source:** mtsDeviceAPI `json/strict/*.schema.json`

---

## Table of Contents

- [System Resource](#system-resource)
- [Users Resource](#users-resource)
- [Cellular Resource](#cellular-resource)
- [WiFi Resource](#wifi-resource)
- [DNS Resource](#dns-resource)
- [DHCP Resource](#dhcp-resource)
- [Routes Resource](#routes-resource)
- [Firewall Resource](#firewall-resource)
- [NAT Resource](#nat-resource)
- [GPS Resource](#gps-resource)
- [Serial Resource](#serial-resource)
- [Alert Resource](#alert-resource)
- [SMS Resource](#sms-resource)

---

## System Resource

**Source:** `system.schema.json`

**Endpoints:**
- `GET /api?fields=system` - Get system information
- `PUT /api/system` - Update writable system settings

### Schema

```json
{
  "capabilities": {
    "wifi": false,
    "bluetooth": false,
    "cell": false,
    "gps": false,
    "lora": false,
    "gpio": false,
    "adc": false,
    "din": false,
    "dout": false,
    "serial": false,
    "battery": false,
    "loraNetworkServer": false,
    "nodeRed": false,
    "ethSwitch": false,
    "docker": false,
    "tpm": false,
    "cellPpp": false,
    "cellWwan": false,
    "loraLbt": false,
    "rs232": false,
    "rs422": false,
    "rs485": false,
    "externalSerialPort": false,
    "supercap": false,
    "userDataEncryption": false,
    "extUsb": false
  },
  "accessoryCards": [],
  "apiVersion": "string",
  "datetime": "MM/DD/YYYY HH:MM:SS",
  "firstTimeSetup": true,
  "dbDirty": false,
  "defaultDeviceName": "string",
  "deviceName": "string",
  "deviceHostname": "string",
  "uuid": "string",
  "productId": "string",
  "deviceId": "string",
  "hardwareVersion": "string",
  "imei": "string",
  "imsi": "string",
  "firmware": "string",
  "firmwareRelease": "string",
  "firmwareImageId": "string",
  "firmwareDate": "string",
  "signedFirmwareValidation": true,
  "vendorId": "string",
  "oemId": "string",
  "platform": "string",
  "loraEui": "string",
  "loraProductId": "string",
  "loraHwVersion": "string",
  "loraFpgaVersion": 0,
  "macAddress": "string",
  "macAddress1": "string",
  "macWifi": "string",
  "macBluetooth": "string",
  "mode": "ROUTER",
  "webTitle": "string"
}
```

### Key Properties

| Property | Type | Writable | Description |
|----------|------|----------|-------------|
| `capabilities` | object | No | Hardware capabilities (read-only) |
| `deviceName` | string | Yes | User-defined device name (max 250) |
| `deviceHostname` | string | Yes | Device hostname (max 250) |
| `dbDirty` | boolean | No | True if unsaved changes exist |
| `signedFirmwareValidation` | boolean | Yes | Require signed firmware |
| `mode` | string | Yes | System mode: `ROUTER` or `CELL-IP-PT` |
| `webTitle` | string | Yes | Browser title (max 250) |

---

## Users Resource

**Source:** `users.schema.json`

**Endpoints:**
- `GET /api?fields=users` - Get all users
- `GET /api?fields=users/{username}` - Get specific user
- `POST /api/users` - Create user
- `PUT /api/users/{username}` - Update user
- `DELETE /api/users/{username}` - Delete user

### Schema

```json
{
  "__v": 2,
  "name": "username",
  "password": "string",
  "permission": "admin|user|guest",
  "firstName": "",
  "lastName": "",
  "title": "",
  "division": "",
  "employeeId": "",
  "address": "",
  "city": "",
  "state": "",
  "postalCode": "",
  "country": "",
  "workPhone": "",
  "mobilePhone": "",
  "email": "",
  "creationDate": 0,
  "lastLoginDate": "",
  "passwordChangedDate": ""
}
```

### Properties

| Property | Type | Required | Constraints | Description |
|----------|------|----------|-------------|-------------|
| `name` | string | Yes | 1-32 chars, pattern: `^[A-Za-z0-9._][A-Za-z0-9._-]*$` | Username/login |
| `password` | string | Yes | max 250 | Password (write-only) |
| `permission` | string | Yes | max 250 | Role: `admin`, `user`, `guest`, or custom role |
| `firstName` | string | Yes | max 250 | First name |
| `lastName` | string | Yes | max 250 | Last name |
| `email` | string | Yes | max 250 | Email address |
| `title` | string | Yes | max 250 | Job title |
| `division` | string | Yes | max 250 | Organization unit |
| `employeeId` | string | Yes | max 250 | Employee ID |

---

## Cellular Resource

**Source:** `cellular.schema.json`

**Endpoints:**
- `GET /api?fields=cellular` - Get cellular configuration
- `PUT /api/cellular` - Update cellular configuration

### Schema

```json
{
  "enabled": false,
  "providerProfiles": [
    {
      "_id": "uuid",
      "builtIn": false,
      "profileName": "string",
      "anySim": false,
      "simFilters": [],
      "cellularMode": "auto",
      "dataCtx": {
        "apnString": "",
        "contextIpMode": "AUTO",
        "authentication": {
          "type": "NONE",
          "username": "",
          "password": ""
        }
      },
      "radioMnoConfig": "auto",
      "radioConfigStrings": [],
      "lteRegCtx": {
        "separateRegApn": false,
        "apnString": "",
        "contextIpMode": "AUTO",
        "authentication": {
          "type": "NONE",
          "username": "",
          "password": ""
        }
      },
      "mtuWwan": 1500,
      "ueModeOfOperation": "AUTO"
    }
  ],
  "simProfiles": [
    {
      "_id": "uuid",
      "profileName": "string",
      "providerProfileId": "uuid|auto",
      "simIccid": "ANY",
      "simPin": ""
    }
  ],
  "dualSim": {
    "enabled": false,
    "mainSlot": "SIM1",
    "backupSimTimeout": 60
  },
  "connMonitors": {
    "failCount": { "enabled": true, "maxFail": 8 },
    "keepAlive": {
      "enabled": false,
      "pingInterval": 60,
      "hostname": "",
      "type": "ICMP",
      "icmpCount": 4,
      "packetSize": 56,
      "tcpPort": 80
    },
    "dataRecv": { "enabled": true, "timeout": 60 },
    "netreg": { "enabled": false, "timeout": 2 },
    "roamingTimeout": { "enabled": false, "timeout": 2 },
    "signal": { "enabled": true, "minRssiDbm": -113, "timeout": 10 }
  },
  "connRecovery": {
    "connReset": { "enabled": true },
    "svcReset": { "enabled": true },
    "radioReset": { "enabled": false },
    "simSwitch": { "enabled": false }
  }
}
```

### Key Properties

| Property | Type | Default | Constraints | Description |
|----------|------|---------|-------------|-------------|
| `enabled` | boolean | `false` | - | Enable cellular |
| `providerProfiles[].cellularMode` | string | `auto` | enum | Network mode preference |
| `providerProfiles[].dataCtx.apnString` | string | - | max 250 | APN string |
| `providerProfiles[].dataCtx.contextIpMode` | string | `AUTO` | enum | `AUTO`, `IP`, `IPV4V6`, `IPV6` |
| `providerProfiles[].dataCtx.authentication.type` | string | `NONE` | enum | `NONE`, `PAP`, `CHAP`, `PAP-CHAP`, `CHAP-PAP` |
| `providerProfiles[].mtuWwan` | integer | `1500` | 128-16384 | MTU for WWAN |
| `simProfiles[].simIccid` | string | `ANY` | pattern or ANY | SIM ICCID filter |
| `dualSim.mainSlot` | string | `SIM1` | enum | `SIM1` or `SIM2` |
| `dualSim.backupSimTimeout` | integer | `60` | 10-1440 | Minutes before switching SIM |
| `connMonitors.keepAlive.type` | string | `ICMP` | enum | `ICMP` or `TCP` |
| `connMonitors.signal.minRssiDbm` | integer | `-113` | -113 to -51 | Minimum signal threshold |

### Cellular Mode Options

| Value | Description |
|-------|-------------|
| `auto` | Automatic selection |
| `2g-only` | 2G only |
| `2g-preferred` | Prefer 2G |
| `3g-only` | 3G only |
| `3g-preferred` | Prefer 3G |
| `4g-only` | 4G/LTE only |
| `4g-preferred` | Prefer 4G/LTE |
| `5g-only` | 5G only |
| `5g-preferred` | Prefer 5G |

---

## WiFi Resource

**Source:** `wifi.schema.json`

**Endpoints:**
- `GET /api?fields=wifi` - Get WiFi configuration
- `PUT /api/wifi` - Update WiFi configuration

### Schema

```json
{
  "ap": {
    "enabled": false,
    "ssid": "string",
    "region": "string",
    "networkMode": "bgn",
    "channel": 6,
    "channelWidth": "20",
    "security": {
      "mode": "NONE",
      "algorithm": "TKIP+AES",
      "psk": ""
    },
    "rtsThreshold": 2347,
    "beaconInterval": 100,
    "dtimInterval": 1,
    "maxClients": 16
  },
  "client": {
    "enabled": false,
    "roaming": {
      "enabled": false,
      "mode": "BALANCED"
    },
    "savedNetworks": []
  },
  "radius": {
    "host": "",
    "port": 1812,
    "secret": "",
    "secondaryHost": "",
    "secondaryPort": 1812,
    "secondarySecret": "",
    "wpaPtkRekey": 0,
    "wpaGroupRekey": 300
  }
}
```

### Access Point Properties

| Property | Type | Default | Constraints | Description |
|----------|------|---------|-------------|-------------|
| `ap.enabled` | boolean | `false` | - | Enable AP |
| `ap.ssid` | string | - | max 32, no `?"\$[\\]+` | Network name |
| `ap.networkMode` | string | `bgn` | enum | `b`, `g`, `n`, `bg`, `ng`, `bgn`, `a`, `an`, `ac`, `ax` |
| `ap.channel` | integer | `6` | 0-14, 36-173 | WiFi channel (0=auto) |
| `ap.channelWidth` | string | `20` | enum | `20`, `40`, `80` MHz |
| `ap.security.mode` | string | `NONE` | enum | Security mode |
| `ap.security.psk` | string | - | 8-63 chars | Pre-shared key |
| `ap.maxClients` | integer | `16` | 0-64 | Max connected clients |

### Security Modes

| Value | Description |
|-------|-------------|
| `NONE` | Open network |
| `WPA-PSK` | WPA Personal |
| `WPA2-PSK` | WPA2 Personal |
| `WPA/WPA2-PSK` | WPA/WPA2 Mixed |
| `WPA3-SAE` | WPA3 Personal |
| `WPA2-PSK/WPA3-SAE` | WPA2/WPA3 Mixed |
| `WPA2-ENTERPRISE` | WPA2 Enterprise (RADIUS) |

---

## DNS Resource

**Source:** `dns.schema.json`

**Endpoints:**
- `GET /api?fields=dns` - Get DNS configuration
- `PUT /api/dns` - Update DNS configuration

### Schema

```json
{
  "enabled": true,
  "primary": "",
  "secondary": "",
  "ipv6": {
    "primary": "",
    "secondary": ""
  },
  "options": []
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `enabled` | boolean | `true` | Enable DNS server |
| `primary` | string | - | Primary DNS server (IPv4) |
| `secondary` | string | - | Secondary DNS server (IPv4) |
| `ipv6.primary` | string | - | Primary DNS server (IPv6) |
| `ipv6.secondary` | string | - | Secondary DNS server (IPv6) |

---

## DHCP Resource

**Source:** `dhcp.schema.json`

**Endpoints:**
- `GET /api?fields=dhcp` - Get DHCP configuration
- `PUT /api/dhcp` - Update DHCP configuration

### Schema

```json
{
  "dhcps": [
    {
      "interface": "br0",
      "enabled": true,
      "subnetAddress": "192.168.2.0",
      "subnetMask": "255.255.255.0",
      "defaultGateway": "192.168.2.1",
      "domain": "",
      "leaseTime": 86400,
      "leases": [],
      "maxLeases": 200,
      "fixedAddresses": [],
      "rangeStart": "192.168.2.100",
      "rangeEnd": "192.168.2.254",
      "options": []
    }
  ],
  "dhcp6s": []
}
```

### DHCP Server Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `interface` | string | - | Network interface name |
| `enabled` | boolean | `true` | Enable DHCP for interface |
| `subnetAddress` | string | `192.168.2.0` | Subnet address |
| `subnetMask` | string | `255.255.255.0` | Subnet mask |
| `defaultGateway` | string | `192.168.2.1` | Default gateway |
| `leaseTime` | integer | `86400` | Lease time in seconds |
| `maxLeases` | integer | `200` | Maximum number of leases |
| `rangeStart` | string | `192.168.2.100` | DHCP pool start |
| `rangeEnd` | string | `192.168.2.254` | DHCP pool end |

### Fixed Address Object

```json
{
  "mac": "00:11:22:33:44:55",
  "ip": "192.168.2.50"
}
```

---

## Routes Resource

**Source:** `routes.schema.json`

**Endpoints:**
- `GET /api?fields=routes` - Get static routes
- `PUT /api/routes` - Update routes
- `POST /api/routes` - Add route
- `DELETE /api/routes/{name}` - Delete route

### Schema

```json
[
  {
    "name": "route-name",
    "ip": "10.0.0.0",
    "mask": 8,
    "gateway": "192.168.2.254"
  }
]
```

### Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Unique route name (max 250) |
| `ip` | string | Yes | Destination IP address |
| `mask` | string/int | Yes | Netmask (1-32 or dotted notation) |
| `gateway` | string | Yes | Gateway IP address |

---

## Firewall Resource

**Source:** `firewall.schema.json`

**Endpoints:**
- `GET /api?fields=firewall` - Get firewall configuration
- `PUT /api/firewall` - Update firewall configuration

### Schema

```json
{
  "pptp": false,
  "l2tp": false,
  "connTrackHelpers": false
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `pptp` | boolean | `false` | Allow PPTP passthrough |
| `l2tp` | boolean | `false` | Allow L2TP passthrough |
| `connTrackHelpers` | boolean | `false` | Enable connection tracking helpers |

---

## NAT Resource

**Source:** `nat.schema.json`

**Endpoints:**
- `GET /api?fields=nat` - Get NAT rules
- `PUT /api/nat` - Update NAT rules
- `POST /api/nat` - Add NAT rule
- `DELETE /api/nat/{guid}` - Delete NAT rule

### Schema

```json
{
  "name": "rule-name",
  "description": "",
  "protocol": "TCP",
  "chain": "PREROUTING",
  "target": "DNAT",
  "guid": "auto-generated",
  "enabled": true,
  "srcInterface": "ANY",
  "srcAddr": "",
  "srcMask": "",
  "srcPort": "",
  "srcMac": "",
  "dstInterface": "ANY",
  "dstAddr": "",
  "dstMask": "",
  "dstPort": "",
  "enableNatLoopback": false,
  "toAddr": "192.168.2.100",
  "toPort": "8080"
}
```

### Properties

| Property | Type | Required | Values | Description |
|----------|------|----------|--------|-------------|
| `name` | string | Yes | max 250 | Rule name |
| `target` | string | Yes | `DNAT`, `SNAT`, `MASQUERADE` | NAT target |
| `chain` | string | No | `PREROUTING`, `POSTROUTING` | iptables chain |
| `protocol` | string | No | `TCP`, `UDP`, `TCP/UDP`, `ANY` | Protocol |
| `srcInterface` | string | No | `ANY`, `WAN`, `LAN`, `CELLULAR`, `WIFI`, `WIFI_AP`, `ETHERNET`, `OPENVPN` | Source interface |
| `dstInterface` | string | No | Same as srcInterface | Destination interface |
| `srcPort` | string | No | e.g., `5000`, `6000:6002` | Source port(s) |
| `dstPort` | string | No | Same format | Destination port(s) |
| `toAddr` | string | Conditional | IPv4 | Translated address |
| `toPort` | string | Conditional | Port expression | Translated port |
| `enableNatLoopback` | boolean | No | - | Enable NAT loopback (DNAT only) |

---

## GPS Resource

**Source:** `gps.schema.json`

**Endpoints:**
- `GET /api?fields=gps` - Get GPS configuration
- `PUT /api/gps` - Update GPS configuration

### Schema

```json
{
  "server": {
    "enabled": false,
    "port": 5445,
    "password": ""
  },
  "client": {
    "enabled": false,
    "protocol": "TCP",
    "remoteHost": "192.168.2.3",
    "port": 5445,
    "password": ""
  },
  "nmea": {
    "enabled": false,
    "interval": 10,
    "idPrefix": "",
    "id": "",
    "gga": true,
    "gsa": true,
    "gsv": true,
    "gll": true,
    "rmc": true,
    "vtg": true
  }
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `server.enabled` | boolean | `false` | Enable GPS server |
| `server.port` | integer | `5445` | Server listen port |
| `client.enabled` | boolean | `false` | Enable GPS client |
| `client.protocol` | string | `TCP` | `TCP` or `UDP` |
| `client.remoteHost` | string | - | Remote server address |
| `nmea.interval` | integer | `10` | Send interval (seconds, min 1) |
| `nmea.gga` | boolean | `true` | Send GGA sentences |
| `nmea.rmc` | boolean | `true` | Send RMC sentences |

---

## Serial Resource

**Source:** `serial.schema.json`

**Endpoints:**
- `GET /api?fields=serial` - Get serial configuration
- `PUT /api/serial` - Update serial configuration

### Schema

```json
{
  "ports": [
    {
      "id": "ap1",
      "mode": "NONE",
      "baudRate": 115200,
      "flowControl": "NONE",
      "parity": "NONE",
      "type": "RS-232",
      "dataBits": 8,
      "stopBits": 1,
      "rs4xxTermination": false
    }
  ]
}
```

### Properties

| Property | Type | Default | Values | Description |
|----------|------|---------|--------|-------------|
| `id` | string | - | - | Port identifier |
| `mode` | string | - | `NONE`, `IPPIPE`, `MODBUS`, `GPSSTREAM` | Port mode |
| `baudRate` | integer | `115200` | 300-921600 | Baud rate |
| `flowControl` | string | `NONE` | `NONE`, `RTS-CTS` | Flow control |
| `parity` | string | `NONE` | `NONE`, `ODD`, `EVEN` | Parity |
| `type` | string | `RS-232` | `RS-232`, `RS-422`, `RS-485`, `RS-485-HALF` | Serial type |
| `dataBits` | integer | `8` | `7`, `8` | Data bits |
| `stopBits` | integer | `1` | `1`, `2` | Stop bits |

---

## Alert Resource

**Source:** `alert.schema.json`

**Endpoints:**
- `GET /api?fields=alert` - Get alert configuration
- `PUT /api/alert` - Update alert configuration

### Schema

```json
{
  "alerts": [
    {
      "event": "Data Usage",
      "enabled": false,
      "email": false,
      "sms": false,
      "snmp": false,
      "notifyGroup": ""
    }
  ],
  "groups": [
    {
      "name": "Admins",
      "phones": [
        { "name": "John", "phone": "+1234567890" }
      ],
      "emails": [
        { "name": "John", "email": "john@example.com" }
      ]
    }
  ]
}
```

### Alert Events

| Event | SNMP Support | Additional Properties |
|-------|--------------|----------------------|
| `Data Usage` | Yes | `dataAllowedPerMonth`, `dataPlanType`, `alertOn` |
| `Signal Strength` | Yes | `signalStrength`, `interval`, `minWaitTimeAfterAlarm` |
| `Device Reboot` | Yes | - |
| `Ethernet Failure` | Yes | `interval`, `minWaitTimeAfterAlarm` |
| `WifiWan Failure` | Yes | `interval`, `minWaitTimeAfterAlarm` |
| `Cellular Failure` | Yes | `interval`, `minWaitTimeAfterAlarm` |
| `Ethernet Traffic` | No | `periodicInterval` |
| `Wifi Traffic` | No | `periodicInterval` |
| `Cellular Traffic` | No | `periodicInterval` |
| `WAN Failover` | Yes | `minWaitTimeAfterStart`, `failoverEvent` |
| `Ping Failure` | Yes | `pingInterval`, `netIface`, `ipAddress`, `count`, `failThreshold` |
| `Password Change` | No | - |

---

## SMS Resource

See **[SMS Schema](SMS-SCHEMA.md)** for complete SMS documentation.

---

## Common Patterns

### UUID Format
```
^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$
```

### IPv4 Address
```
format: "ipv4"
```

### IPv6 Address
```
format: "ipv6"
```

### MAC Address
```
^([0-9A-Fa-f]{2}:){5}([0-9A-Fa-f]{2})$
```

### Port Expression
```
5000           - Single port
6000:6002      - Port range
5000,6000-6002 - Multiple ports/ranges
```

---

## Related Documentation

- [SMS Schema](SMS-SCHEMA.md) - Complete SMS documentation
- [Available Resources](AVAILABLE-RESOURCES.md) - Resource endpoints
- [Command Endpoints](COMMAND-ENDPOINTS.md) - Command reference

---

**Document Version:** 1.0.0  
**Source Version:** mtsDeviceAPI (analyzed February 2026)  
**Last Updated:** February 10, 2026
