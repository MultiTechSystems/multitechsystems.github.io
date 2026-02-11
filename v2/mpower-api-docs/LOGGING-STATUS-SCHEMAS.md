---
layout: default
title: "Logging and Status Schemas Reference"
permalink: /v2/mpower-api-docs/LOGGING-STATUS-SCHEMAS/
doc_version: v2
---

# Logging and Status Schemas Reference

Complete schema documentation for logging and status resources extracted from mtsDeviceAPI source code.

**Source:** `/home/jreiss/Workspace/aep/mtsDeviceAPI/json/strict/`

---

## Table of Contents

- [Event Log](#event-log)
- [Syslog](#syslog)
- [Status](#status)
- [LLDP](#lldp)
- [mDNS](#mdns)
- [Policy](#policy)
- [External I/O](#external-io)
- [External USB](#external-usb)
- [Database Info](#database-info)

---

## Event Log

Event logging configuration.

**Source:** `eventlog.schema.json`

**Endpoints:**
- `GET /api?fields=eventlog` - Get event log configuration
- `PUT /api/eventlog` - Update event log configuration
- `GET /api?fields=eventlog/events` - Get events
- `GET /api?fields=eventlog/stats` - Get event statistics
- `GET /api?fields=eventlog/downloadcsvevents` - Download events as CSV

### Schema

```json
{
  "severityLevel": "INFORMATIONAL",
  "remote": [
    {
      "alias": "syslog-server",
      "host": "192.168.1.100",
      "port": 514,
      "severityLevel": "WARNING",
      "description": "Remote syslog server"
    }
  ]
}
```

### Severity Levels

| Level | Description |
|-------|-------------|
| `EMERGENCY` | System is unusable |
| `ALERT` | Action must be taken immediately |
| `CRITICAL` | Critical conditions |
| `ERROR` | Error conditions |
| `WARNING` | Warning conditions |
| `NOTICE` | Normal but significant |
| `INFORMATIONAL` | Informational messages |

### Remote Destination Object

| Property | Type | Description | Constraints |
|----------|------|-------------|-------------|
| `alias` | string | Unique identifier | 1-32 chars |
| `host` | string | Hostname or IP | required |
| `port` | integer | Port number | 1-65535 |
| `severityLevel` | string | Minimum severity | see above |
| `description` | string | Description | max 1024 chars |

---

## Syslog

System logging and data history configuration.

**Source:** `syslog.schema.json`

**Endpoints:**
- `GET /api?fields=syslog` - Get syslog configuration
- `PUT /api/syslog` - Update syslog configuration

### Schema

```json
{
  "enabled": false,
  "ipAddress": "",
  "port": 514,
  "protocol": "udp",
  "logLevel": 30,
  "wifiDataHistory": 30,
  "cellDataHistory": 30,
  "ethDataHistory": 30,
  "saveTimeout": 3600,
  "saveDataLimit": 10,
  "outputToFile": false
}
```

### Key Properties

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `enabled` | boolean | Enable remote syslog | false |
| `ipAddress` | string | Syslog server IP | - |
| `port` | integer | Syslog port | 514 (1-65535) |
| `protocol` | string | Transport protocol | udp |
| `logLevel` | integer | Debug level | 30 (0-100) |
| `outputToFile` | boolean | Write logs to file | false |

### Protocol Values

- `tcp` - TCP transport
- `udp` - UDP transport
- `tls` - TLS encrypted

### Log Levels

| Level | Description |
|-------|-------------|
| 20 | Warning |
| 30 | Info |
| 50 | Debug |
| 60 | Trace |
| 100 | Maximum |

### Data History Settings

| Property | Type | Description | Range |
|----------|------|-------------|-------|
| `wifiDataHistory` | integer | WiFi history (days) | 0-365 |
| `cellDataHistory` | integer | Cellular history (days) | 0-365 |
| `ethDataHistory` | integer | Ethernet history (days) | 0-365 |
| `saveTimeout` | integer | Save interval (seconds) | 60-86400 |
| `saveDataLimit` | integer | Data limit (MB) | 1-100 |

---

## Status

System status messages (read-only).

**Source:** `status.schema.json`

**Endpoints:**
- `GET /api?fields=status` - Get system status messages

### Schema

```json
[
  {
    "guid": "unique-event-id",
    "type": "INFO",
    "msg": "System started successfully",
    "timestamp": "2026-02-10T15:30:00Z"
  }
]
```

### Status Types

| Type | Description |
|------|-------------|
| `INFO` | Informational message |
| `WARNING` | Warning condition |
| `ERROR` | Error condition |

---

## LLDP

Link Layer Discovery Protocol configuration.

**Source:** `lldp.schema.json`

**Endpoints:**
- `GET /api?fields=lldp` - Get LLDP configuration
- `PUT /api/lldp` - Update LLDP configuration

### Schema

```json
{
  "enabled": false,
  "sysName": "",
  "sysDescr": "",
  "txInterval": 30,
  "txHold": 4
}
```

### Key Properties

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `enabled` | boolean | Enable LLDP | false |
| `sysName` | string | Override system name | - |
| `sysDescr` | string | Override system description | - |
| `txInterval` | integer | Transmit interval (seconds) | 30 (5-32768) |
| `txHold` | integer | TTL multiplier | 4 (2-10) |

Note: `sysName` and `sysDescr` cannot contain double quotes.

---

## mDNS

Multicast DNS configuration.

**Source:** `mdns.schema.json`

**Endpoints:**
- `GET /api?fields=mdns` - Get mDNS configuration
- `PUT /api/mdns` - Update mDNS configuration

### Schema

```json
{
  "enabled": false,
  "allowInterfaces": ["eth0", "br0"]
}
```

### Key Properties

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable mDNS service |
| `allowInterfaces` | array | Allowed network interfaces |

### Interface Name Pattern

```
^[a-zA-Z0-9_\.-]+$
```

---

## Policy

Usage policy configuration.

**Source:** `policy.schema.json`

**Endpoints:**
- `GET /api?fields=policy` - Get usage policy
- `PUT /api/policy` - Update usage policy

### Schema

```json
{
  "policy": "Acceptable Use Policy text here..."
}
```

### Key Properties

| Property | Type | Description | Constraints |
|----------|------|-------------|-------------|
| `policy` | string | Policy text | max 60000 chars |

---

## External I/O

Digital input/output configuration.

**Source:** `extIo.schema.json`

**Endpoints:**
- `GET /api?fields=extIo` - Get external I/O configuration
- `PUT /api/extIo` - Update external I/O configuration
- `GET /api?fields=stateIo/digital` - Get digital I/O state

### Schema

```json
{
  "digitalInputs": [
    {
      "id": "DI0",
      "name": "Input 1",
      "index": 0
    }
  ],
  "digitalOutputs": [
    {
      "id": "DO0",
      "name": "Output 1",
      "mode": "LEVEL",
      "activeLevel": "HIGH",
      "duration": 1000,
      "index": 0
    }
  ]
}
```

### Digital Input Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | string | GPIO identifier (read-only) |
| `name` | string | User-defined name |
| `index` | integer | Input index |

### Digital Output Properties

| Property | Type | Description | Constraints |
|----------|------|-------------|-------------|
| `id` | string | GPIO identifier (read-only) | - |
| `name` | string | User-defined name | max 10 chars, alphanumeric |
| `mode` | string | Output mode | `LEVEL`, `PULSE` |
| `activeLevel` | string | Active level (PULSE mode) | `LOW`, `HIGH` |
| `duration` | integer | Pulse duration (ms) | 1-86400000 |
| `index` | integer | Output index | - |

### Output Modes

| Mode | Description |
|------|-------------|
| `LEVEL` | Steady state output |
| `PULSE` | Pulsed output for specified duration |

---

## External USB

USB port configuration.

**Source:** `extUsb.schema.json`

**Endpoints:**
- `GET /api?fields=extUsb` - Get USB configuration
- `PUT /api/extUsb` - Update USB configuration

### Schema

```json
{
  "enabled": false
}
```

---

## Database Info

Database configuration information (read-only).

**Source:** `databaseInfo.schema.json`

**Endpoints:**
- `GET /api?fields=databaseInfo` - Get database information

### Schema

```json
{
  "location": "/path/to/database",
  "timestamp": "2026-02-10 15:30:00",
  "version": "1.0.0",
  "default": {
    "location": "/path/to/default",
    "timestamp": "2026-01-01 00:00:00",
    "version": "1.0.0"
  },
  "oem": null
}
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `location` | string | Database file path |
| `timestamp` | string | Last modified time |
| `version` | string | Database version |
| `default` | object | Default database info |
| `oem` | object | OEM database info (null if none) |

---

**Last Updated:** February 10, 2026
